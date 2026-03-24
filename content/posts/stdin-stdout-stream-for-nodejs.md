+++
date = "2026-03-22T17:01:31+09:00"
draft = true
title = 'Stdin Stdout Stream for Nodejs'
+++

upnote://x-callback-url/openNote?noteId=019d1425-21a3-7129-b4d1-24700a7d4772
upnote://x-callback-url/openNote?noteId=019d1313-2bdc-716c-ba6b-7a525d743855


# 표준 입출력 에 대해서
- OS 프로세스에서의 표준 입출력의 역할
- Stream 인가?
- NodeJS stream vs web stream
- Push 방식 vs Pull 방식
- Cold stream vs Hot stream
- stream 과 file 의 관계
- Stream 과 async iterator 의 관계


## NodeJS에서 파일을 다루는 여러 가지 방법에 대한 예제를 보자.

### ① `open`, `read`, `close` (Low-level)

파일 식별자(fd)를 직접 제어하며, 미리 할당한 **Buffer**에 데이터를 채우는 방식입니다.

```
const fs = require('fs');

fs.open('example.txt', 'r', (err, fd) => {
  if (err) throw err;

  // 1024바이트 크기의 버퍼 공간을 미리 확보
  const buffer = Buffer.alloc(1024);

  // fd를 통해 0번 포지션부터 1024바이트를 읽어 buffer에 채움
  fs.read(fd, buffer, 0, 1024, 0, (err, bytesRead, buf) => {
    if (err) throw err;
    console.log('Read:', buf.slice(0, bytesRead).toString());

    // 사용 후 반드시 수동으로 닫아야 함
    fs.close(fd, () => {});
  });
});

```

### ② `readFile` (High-level)

가장 간편하며, 파일 전체를 하나의 **Buffer**에 다 담은 후 콜백을 실행합니다.

```
const fs = require('fs');

fs.readFile('example.txt', (err, data) => {
  if (err) throw err;
  // data는 파일 전체 내용이 담긴 Buffer 객체
  console.log(data.toString());
});

```

### ③ Node.js Stream

파일을 조각(Chunk) 단위로 읽어 흐름(Stream)을 만듭니다. 대용량 파일에 효율적입니다.

```
const fs = require('fs');

const readStream = fs.createReadStream('example.txt', { highWaterMark: 64 * 1024 });

readStream.on('data', (chunk) => {
  // chunk는 Buffer 객체 (데이터 조각)
  console.log('Received chunk:', chunk.toString());
});

readStream.on('end', () => console.log('Finished reading'));

```

NodeJS Stream은 데이터를 소스에서 Stream 소비자에게 전달을 하는데 이때 직접 전달할 수도 있지만, 만약 소비자가 아직 ‘data’ 이벤트 리스너를 달지 않았거나 pause를 해서 Paused Mode 인 경우, 데이털르 전달할 수 없으므로 내부 버퍼에 쌓는다. 이렇게 내부 버퍼가 있으므로, 소스에서 데이터를 읽는 것과 Stream 소비자가 데이터를 읽어가는 것이 독립적으로 일어날 수 있다. Stream은 소스로부터 데이터를 읽어서 내부 버퍼에 쌓고, Stream 소비자는 내부 버퍼로부터 데이터를 읽어가며, 소비자가 데이터를 읽는 중에 동시에 Stream이 데이터를 쌓을 수도 있는 것이다.

버퍼는 선입선출(FIFO) 이므로, 소비자가 버퍼의 앞에서 읽는 동안, Stream은 버퍼의 뒤에 데이터를 쌓는 것이다. 어디까지 읽었는지에 대한 offset과 어디까지 썼는지에 대한 offset을 따로 관리하면 된다.

Stream은 생성되면, 소비자가 데이터를 달라고 하지 않아도 먼저 소스에서 데이터를 꺼내서 내부 버퍼에 쌓는다. 이는 pre-fetching 동작으로 사용자가 달라고 할 때 바로 줄 수 있도록 메모리에 준비시키는 것이다. 다만, 무한정 읽지는 않고 `highWaterMark` 라는 최대값을 정해놓고, 내부 버퍼의 양이 이를 넘어가면 pre-fetching 을 멈춘다. 소비자가 버퍼를 읽어가면서 쌓아둔 데이터 양이 `highWaterMark` 이하로 떨어지면 다시 pre-fetching을 한다.

Stream에는 Paused 모드와 Flowing 모드가 있는데 이는 소비에게 데이터가 흐르느냐를 가리키는 모드로 pre-fetching 동작과는 관련이 없다. Paused 모드라도 pre-fetching을 일어난다. 다만, 곧 `highWaterMark` 이상 버퍼가 찰 것이므로 결국에는 소스로부터 읽는 동작도 멈추게 되긴 할 것이다. 이 두 가지 모드는 ReadStream에만 있는 모드인데, 기본적으로 생성되면 Paused 모드로 시작하고, `on('data')` 이벤트 리스너를 붙이면 Flowing 모드가 되면서 데이터가 흐르기 시작한다. 이벤트 리스너가 붙은 상태에서도 `pause()`, `resume()`으로 이 모드를 조절할 수 있다. `pause()`를 하면 내부 버퍼에 데이터가 있어도 `'data'`이벤트가 발생하지 않는다. 이는 백프레셔를 구현하기 위해 필요하다.

WriteStream에도 내부 버퍼가 있다. WriteStream에 쓰면(`write`) 내부적으로 정의된 `_write` 메소드가 실행되면서 최종 목적지에 쓰기 작업을 진행하는데, 이 메소드에는 쓰기 작업의 완료를 표시하기 위한 콜백을 인자로 받는다. 즉, 쓰기를 시작했으나 이 콜백을 호출하지 않아서 완료되지 않은 상태로 인식된 상태에서 또 쓰려고(`write`) 하면, 이 때 내부 버퍼에 데이터를 쌓는다. 현재 진행 중인 쓰기가 없고, 버퍼가 비어 있으면, 내부 버퍼를 거치지 않고 바로 write 작업을 한다고 한다.

```
class MyWritable extends Writable {
  _write(chunk, encoding, callback) {
    // 1. 데이터를 처리함 (예: 외부 API 전송)
    doHeavyTask(chunk, () => {
      // 2. 처리가 완전히 끝났을 때 이 callback을 호출함!
      // ★ 이 callback을 호출해야 Node.js가 "아, 한 조각 비워졌구나"라고 인식함
      callback();
    });
  }
}

```

이렇게 내부 버퍼가 차기 시작해서 `highWaterMark` 이상으로 차면, WriteStream은 더 쓰기 불가능한 상태가 된다. WriteStream의 `writeStream.write(chunk)`함수는 boolean 값을 반환하는데 이게 더 쓸 수 있느냐를 가리키는 값이다. 예를 들어 이 함수가 `false`를 반환한다는 것은 방금 쓰기 작업으로 내부 버퍼가 지정된 용량(`highWaterMark`) 이상으로 차게 되어 더 쓸 수 없게 되었다는 의미다.

```
const readStream = fs.createReadStream('source.txt');
const writeStream = fs.createWriteStream('dest.txt');

readStream.on('data', (chunk) => {
  const canContinue = writeStream.write(chunk);
  if (!canContinue) {
    // 쓰기 스트림이 꽉 찼으니 잠시 멈춤 (수동 배압 제어)
    readStream.pause();
  }
});

writeStream.on('drain', () => {
  // 쓰기 스트림이 비워졌으니 다시 시작
  readStream.resume();
});

```

만약 ReadStream과 WriteStream을 위와 같이 수동으로 연결 중이라면, WriteStream의 내부 버퍼가 꽉 찬 경우, ReadStream을 `pause()` 하여 데이터를 잠시 받지 않아야 한다. 그리고 WriteStream의 쓰기 작업이 끝나고 내부 버퍼를 소비하면서 내부 버퍼가 쓸 수 있는 상태가 되면 WriteStream에서 `'drain'` 이벤트가 발생하는데 이 때 ReadStream을 `resume()` 하여 데이터를 다시 받는 식으로 구현하는데 이게 백프레셔를 구현하는 것이다.

ReadStream에는 `.pipe(writeStream)` 이라는 메소드가 있어서, 이를 쓰면, 위와 같은 백프레셔 로직을 알아서 구현해준다고 한다.

```
const readStream = fs.createReadStream('source.txt');
const writeStream = fs.createWriteStream('dest.txt');

// 위 수십 줄의 코드를 단 한 줄로 해결 (배압 제어 포함)
readStream.pipe(writeStream);

```

NodeJS의 readStream은 `read(size)` 메소드도 사용 가능하다. 이는 지정한 크기만큼 데이터를 읽는 것으로 Pause 모드에서도 동작한다고 한다.

<br>
