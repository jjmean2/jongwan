+++
date = "2026-03-23T22:17:42+09:00"
draft = true
title = 'End of File'
+++

upnote://x-callback-url/openNote?noteId=019d1ad5-e75f-765e-8189-67df56ec15a7

NodeJS의 readFileSync가 파일의 끝을 판단하는 법

네, `fs.readFileSync`는 기본적으로 **파일의 끝(EOF, End of File)**을 만날 때까지 데이터를 메모리로 전부 읽어 들인 후, 그 전체 내용을 `Buffer`(또는 인코딩 지정 시 `string`)로 반환합니다.

하지만 말씀하신 것처럼 **일반 파일(Regular File)**이 아닌 **파이프(Pipe)**나 **가상 장치(/dev/urandom 등)**를 다룰 때는 동작 방식이 조금 더 복잡해집니다. 이를 구분하는 핵심 기제는 OS 레벨의 `stat` 시스템 콜과 `read` 시스템 콜의 반환값에 있습니다.

* * *

### 1\. 파일 전체라는 것을 어떻게 판단하나? (EOF의 원리)

Node.js(정확히는 밑단인 libuv와 OS)는 파일을 읽을 때 다음과 같은 과정을 거칩니다.

1. **크기 확인:** 먼저 `fstat`을 호출하여 파일의 크기를 확인합니다.
2. **버퍼 할당:** 확인된 크기만큼의 메모리(Buffer)를 미리 할당합니다.
3. **반복 읽기:** `read()` 시스템 콜을 호출하여 데이터를 채웁니다.
4. **EOF 감지:** `read()` 호출 시 반환값이 **`0`**이면, "더 이상 읽을 데이터가 없다(End of File)"고 판단하고 읽기를 중단합니다.

### 2\. 일반 파일 vs 파이프/가상 장치 구분

`readFileSync` 내부적으로는 대상의 성격에 따라 다르게 대응합니다.

#### **A. 일반 파일 (Regular Files)**

- `fstat`을 통해 정확한 **`st_size` (파일 크기)**를 알 수 있습니다.
- Node.js는 이 크기만큼 버퍼를 한 번에 만들고 데이터를 채웁니다.

#### **B. 파이프, 소켓, 캐릭터 장치 (FIFO, Sockets, /dev/stdin 등)**

- 이런 "특수 파일"들은 `fstat`을 해도 크기가 **`0`**으로 나옵니다. 데이터가 실시간으로 생성되거나 흘러오기 때문입니다.
- **Node.js의 전략:** 1. 크기가 0이면, 일단 **8KB(기본값)** 정도의 작은 버퍼를 할당합니다.
    2. 데이터를 읽어 버퍼에 채웁니다.
    3. 버퍼가 다 차면 버퍼 크기를 늘려가며(Dynamic Resizing) 계속 읽습니다.
    4. 송신 측에서 연결을 닫아 **`read()` 결과가 `0`**이 나올 때까지 무한히 반복합니다.

* * *

### 3\. 발생할 수 있는 문제점 (주의사항)

이 메커니즘 때문에 일반 파일이 아닌 대상을 `readFileSync`로 읽을 때 주의해야 할 점이 있습니다.

- **무한 대기 (Blocking):** `/dev/stdin` 같은 파이프를 `readFileSync`로 읽으면, 사용자가 `Ctrl+D` (EOF 전송)를 누르기 전까지 프로세스가 **완전히 멈춰(Blocking)** 버립니다.
- **무한 루프/메모리 고갈:** `/dev/zero`나 `/dev/urandom` 같은 가상 장치는 끝(EOF)이 없습니다. 여기서 `readFileSync`를 실행하면 메모리가 꽉 찰 때까지 계속 읽다가 **`RangeError [ERR_FS_FILE_TOO_LARGE]`** 또는 **`Out of Memory`** 에러를 내며 프로세스가 죽습니다.
- **소켓(Socket):** 소켓 역시 연결이 끊기기 전까지는 `read`가 `0`을 반환하지 않으므로, 데이터가 다 왔다고 생각해도 연결이 유지 중이라면 함수가 끝나지 않습니다.

* * *

### 요약: 내부 로직 (Pseudo Code)

JavaScript

```
// readFileSync의 대략적인 내부 철학
let totalBuffer = Buffer.alloc(0);
let bytesRead = 0;

while (true) {
  const chunk = Buffer.alloc(8192); // 일단 8KB씩 읽어봄
  const result = fs.readSync(fd, chunk, ...);

  if (result === 0) break; // 드디어 0(EOF)을 만나면 탈출!

  totalBuffer = Buffer.concat([totalBuffer, chunk.slice(0, result)]);
}
return totalBuffer;
```

따라서 **끝이 정해지지 않은 스트림**이나 **매우 큰 파일**을 다룰 때는 `readFileSync` 대신 반드시 **Stream API**(`fs.createReadStream`)를 사용하여 데이터를 덩어리(chunk) 단위로 처리해야 안전합니다.

혹시 특정 가상 장치나 파이프에서 데이터를 안전하게 읽어오는 코드가 필요하신가요?
