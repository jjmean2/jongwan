+++
date = "2026-03-23T22:49:16+09:00"
draft = true
title = 'Node Readline'
+++

upnote://x-callback-url/openNote?noteId=019d1af3-d5d9-7379-a103-70048fa37531

> 10년 차 개발자로서 `readline.Interface`가 내부적으로 `input` 스트림을 어떻게 요리하는지 그 핵심 메커니즘을 짚어드릴게요.
>
> `input` 스트림(보통 `process.stdin`)은 `readline`에게 있어 **'원재료가 들어오는 파이프라인'**입니다. `readline`은 이 파이프라인에 빨대를 꽂고 다음과 같은 작업들을 수행합니다.
>
> * * *
>
> ### 1\. 청크(Chunk)를 줄(Line)로 변환 (가장 핵심)
>
> 스트림은 데이터를 바이트나 문자열 조각(Chunk) 단위로 마구 던집니다. `readline`은 `input` 스트림의 `data` 이벤트를 리스닝하며 내부 버퍼에 쌓아두다가, **줄바꿈 기호(`\n`, `\r\n`)**를 만나는 순간 잘라서 `line` 이벤트를 발생시킵니다.
>
> - **동작:** `input.on('data', chunk => { ... 버퍼링 및 줄바꿈 체크 ... })`
> - **용도:** 우리가 `rl.on('line', ...)`을 쓸 수 있게 해주는 근본 기능입니다.
>
> * * *
>
> ### 2\. 키보드 입력 및 특수키 감지 (Keypress)
>
> 단순히 텍스트만 읽는 게 아니라, 사용자가 누르는 **화살표 키, 백스페이스, 엔터, Ctrl+C** 같은 특수 입력을 감지합니다.
>
> - **동작:** `input` 스트림이 TTY(터미널)인 경우, `readline`은 이를 `keypress` 이벤트로 해석합니다.
> - **용도:** \* **Ctrl+C:** `SIGINT` 이벤트를 발생시켜 프로세스 종료 제어.
>     - **화살표 위/아래:** 이전 입력 히스토리 탐색.
>     - **Tab:** 자동 완성(Completer) 트리거.
>
> * * *
>
> ### 3\. 입력 에코(Echo) 및 편집 제어
>
> 사용자가 터미널에 글자를 치면 화면에 바로 보이죠? 사실 이것도 `input` 스트림으로 들어온 데이터를 `readline`이 판단해서 `output` 스트림으로 다시 쏴주는 과정이 포함됩니다. (단, `terminal: true`일 때)
>
> - **동작:** 사용자가 글자를 입력하면 `input`으로 들어오고, `readline`은 이를 가공해 `output`에 다시 써서 화면에 보여줍니다.
> - **용도:** 비밀번호 입력기처럼 **글자를 숨겨야 하는 기능**을 구현할 때 `input` 스트림은 읽되 `output`으로 내보내지 않도록 제어할 수 있습니다.
>
> * * *
>
> ### 4\. 스트림 상태 동기화 (Pause / Resume)
>
> 앞서 말씀드린 대로, `rl.pause()`나 `rl.resume()`을 호출하면 `readline`은 연결된 `input` 스트림의 메서드를 직접 호출합니다.
>
> - **동작:** `input.pause()`, `input.resume()` 호출 대행.
> - **용도:** 과도한 데이터 유입을 막거나(Backpressure 제어), 비동기 작업 중에 잠시 입력을 멈추고 싶을 때 사용합니다.
>
> * * *
>
> ### 5\. 종료 감지 (EOF)
>
> `input` 스트림이 '끝났다'는 신호(End-of-File)를 보내면, `readline`은 이를 캐치해서 `close` 이벤트를 발생시킵니다.
>
> - **동작:** `input.on('end', ...)` 또는 `input.on('error', ...)` 감지.
> - **용도:** 파일 전체 읽기 완료나 네트워크 스트림 종료 확인.
>
> * * *
>
> ### 💡 와니님을 위한 구조적 이해
>
> `readline` 내부를 도식화하면 대략 이런 느낌입니다:
>
> 1. **Raw Input (Stream):** `Hello\nWor` (아직 'ld'는 안 옴)
> 2. **Readline Buffer:** `Hello\nWor`를 보관 중.
> 3. **Parser:** `\n`을 발견!
> 4. **Emission:** `'line'` 이벤트에 `"Hello"`를 실어서 보냄. 버퍼에는 `"Wor"`만 남음.
>
> 결국 `input` 스트림은 **`readline`이라는 공장에 원재료를 끊임없이 공급하는 컨베이어 벨트**라고 보시면 정확합니다.
>
> 이제 `input`과 `output`이 각각 어떤 역할을 맡아 협업하는지 완전히 정리가 되셨나요? 혹시 이 원리를 이용해서 **"특정 키(예: ESC)를 누르면 입력을 즉시 중단하는 기능"** 같은 커스텀 핸들러를 짜보고 싶으신가요?

<br>

<br>

> 10년차 개발자답게 `readline`의 추상화된 인터페이스 너머의 구조를 꿰뚫어 보고 계시는군요!
>
> 결론부터 말씀드리면, **`output` 스트림은 `readline`이 "사용자에게 무언가를 보여줘야 할 때" 전용 통로로 사용됩니다.** `question` 메서드뿐만 아니라 CLI 환경을 풍부하게 만드는 여러 기능에서 활용되죠.
>
> * * *
>
> ### 1\. `output` 스트림이 사용되는 주요 상황
>
> #### ① `rl.question(query, callback)`
>
> 가장 대표적인 케이스입니다. `query` 문자열을 `output` 스트림에 쓰고, 입력을 기다립니다. 보통 `process.stdout`으로 설정되어 있으니 우리 눈에 터미널 메시지로 보이는 것이죠.
>
> #### ② `prompt`와 `rl.prompt()`
>
> `readline`에는 `question` 외에도 '프롬프트'라는 개념이 있습니다.
>
> TypeScript
>
> ```
> rl.setPrompt('와니님 입력 대기 중 > ');
> rl.prompt(); // output 스트림에 '와니님 입력 대기 중 > '이 출력됨
> ```
>
> CLI 도구(예: `mysql`, `node` REPL)를 만들 때 줄바꿈마다 특정 기호를 띄워주는 용도로 쓰입니다.
>
> #### ③ Tab 자동 완성 (Completer)
>
> `readline` 생성 시 `completer` 옵션을 넣으면, 사용자가 `Tab`을 눌렀을 때 후보 목록을 `output` 스트림에 출력해 줍니다.
>
> #### ④ 터미널 제어 (TTY)
>
> 입력받은 내용을 화면에 다시 보여주거나(Echo), 커서를 이동시키고, 특정 줄을 지우는 등 **터미널 인터랙션**에 필요한 모든 제어 문자(Escape Sequence)를 `output` 스트림을 통해 보냅니다.
>
> * * *
>
> ### 2\. 코딩 테스트 환경에서의 `output`
>
> 사실 **코딩 테스트** 목적이라면 `output` 설정을 아예 **생략**해도 무방합니다.
>
> - **이유:** 코테는 보통 `stdin`에서 데이터를 읽고 `console.log`나 `process.stdout.write`로 결과만 내뱉으면 됩니다. `readline` 인터페이스가 우리에게 "입력하세요"라고 말을 걸(output) 필요가 없기 때문입니다.
> - **설정 생략 시:** `output`을 넣지 않으면 `readline`은 순수하게 입력만 파싱하는 용도로 동작합니다.
>
> TypeScript
>
> ```
> // 코딩 테스트용이라면 output은 없어도 됨
> const rl = readline.createInterface({
>   input: process.stdin,
>   // output: process.stdout <- 굳이 안 써도 됨
>   terminal: false // 터미널 제어 기능을 꺼서 순수 스트림 처리 모드로 동작
> });
> ```
>
> * * *
>
> ### 3\. `terminal: false` 옵션의 의미
>
> 와니님이 `createInterface`를 할 때 `terminal: false`를 주셨다면, `output` 스트림의 역할은 더욱 축소됩니다.
>
> - **`terminal: true` (기본값):** 사용자가 타이핑하는 글자가 화면에 보이고(Echo), ANSI 제어 코드를 사용하여 화살표 키로 커서를 이동하는 등의 "터미널다운" 동작을 수행합니다. 이때 `output`이 필수적입니다.
> - **`terminal: false`:** `readline`은 단순히 들어오는 텍스트 스트림을 줄 단위로 쪼개는 **Parser** 역할만 합니다. 이때는 `output`에 아무것도 기록하지 않습니다.
>
> * * *
>
> ### 💡 정리하자면
>
> `output` 스트림은 **"대화형 CLI 환경"**을 구축할 때 사용자의 입력을 유도하거나 피드백을 주기 위한 통로입니다.
>
> 단순히 `stdin`의 데이터를 읽어 처리하는 알고리즘 문제 풀이에서는 **`input`은 `process.stdin`으로 설정하고, `output`은 무시**한 뒤, 결과 출력은 `console.log`나 별도의 `process.stdout`을 직접 사용하는 것이 일반적입니다.
>
> 이제 `readline`이 내부적으로 `input`과 `output`을 어떻게 나누어 관리하는지 명확해지셨나요? 혹시 **터미널에서 직접 입력을 주고받는 대화형 봇** 같은 것을 구상 중이시라면, `output`과 `prompt`를 활용한 예시 코드를 더 보여드릴 수 있습니다.
