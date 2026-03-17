+++
date = '2026-03-16T22:52:16+09:00'
draft = true
title = 'macOS의 개발도구 관리 도구, xcode-select와 xcrun'
+++

macOS에서 개발을 하다 보면 생각보다 자주 마주치는 메시지가 있다.

```bash
xcrun: error: invalid active developer path
```

혹은 어떤 머신에서는 `clang`이 바로 되는데, 어떤 머신에서는 먼저 Xcode를 설치하라는 안내가 나온다. 처음에는 그냥 "애플 개발하려면 Xcode가 필요한가 보다" 정도로 넘기기 쉽지만, 조금만 들여다보면 macOS의 개발환경은 생각보다 층이 나뉘어 있다.

나는 iOS 개발자로 커리어를 시작하다 보니 예전에는 Xcode를 기본으로 설치했지만, 프론트엔드 쪽 업무를 주로 하면서부터는 Xcode 대신 VS Code를 주로 사용하게 되었다. 터미널 도구도 `npm` 생태계의 Node.js 도구들이 대부분이고, `git`도 최신 버전을 쓰기 위해 Homebrew로 직접 설치해서 쓴다. `python`, `go`, `ruby` 등을 가볍게 써볼 때도 `asdf`나 Homebrew 같은 패키지 관리 도구를 더 자주 쓴다. 그래서 한동안은 Xcode가 제공하는 개발환경이 이제는 별로 필요 없는 것 아닌가 생각했는데, 실제로는 그렇지 않았다.

이 글에서는 다음 네 가지를 정리해보려 한다.

- Xcode IDE를 설치하는 것과 Command Line Tools for Xcode를 설치하는 것은 무엇이 다른가
- macOS에서 개발도구는 어떤 층위로 나뉘어 있는가
- `xcode-select`는 무엇을 선택하는 도구인가
- `xcrun`은 무엇을 기준으로 어떤 실행 파일을 찾아주는가

## macOS의 개발도구를 나눠서 보기

터미널을 사용하다 보면 다양한 명령을 쓴다. 파일을 탐색하고 조작하기 위한 `ls`, `cd`, `cat`, `vi`, `cp`, `mv` 같은 기본 명령도 있고, 개발할 때 수시로 사용하는 `git`, `make`, `clang` 같은 도구도 있다. 그리고 Apple 플랫폼 개발을 할 때만 필요한 `xcodebuild`, `simctl`, `actool` 같은 도구도 있다.

macOS에서 사용하는 CLI 도구를 굳이 나누면 대략 다음 세 부류로 볼 수 있다.

- 시스템 기본 도구: `ls`, `cat`, `ps`, `vi` 등
- 범용 개발 도구: `git`, `make`, `clang` 등
- Apple 플랫폼용 개발 도구: `xcodebuild`, `simctl`, SDK, Simulator 관련 도구 등

여기서 첫 번째는 유닉스 계열 시스템에 기본적으로 포함된 도구들이라 별도 설치 없이 쓸 수 있다. 반면 뒤의 두 부류는 개발을 위한 도구들이고, 특히 Apple 플랫폼용 도구들은 Xcode 생태계 안에서 관리된다.

이 지점에서 중요한 사실이 하나 있다. macOS에서 "애플 개발도구를 설치한다"는 말은 사실 두 가지를 가리킬 수 있다.

- Xcode IDE 전체를 설치하는 것
- Command Line Tools for Xcode만 설치하는 것

이 둘은 겹치는 부분도 있지만, 목적과 포함 범위가 다르다.

## Xcode IDE와 Command Line Tools의 차이

가장 단순하게 말하면 Xcode IDE는 풀패키지이고, Command Line Tools는 최소한의 개발 툴체인이다.

### Xcode IDE

App Store에서 설치하는 Xcode는 GUI IDE를 포함한 전체 개발환경이다. 보통 다음과 같은 것들이 함께 들어 있다.

- Xcode 앱 자체
- iOS, macOS, watchOS, tvOS용 SDK
- `xcodebuild`, `simctl` 같은 Apple 플랫폼 전용 도구
- 시뮬레이터 및 디버깅 도구
- `clang`, `swiftc`, `git`, `make` 등 여러 커맨드라인 도구

즉 iPhone 앱을 빌드하고 시뮬레이터로 실행하고, 아카이브를 만들고, 서명하고, 디버깅하는 데 필요한 것들이 거의 다 포함된다.

### Command Line Tools for Xcode

반면 Command Line Tools는 IDE 없이도 터미널에서 개발 작업을 할 수 있도록 최소 구성만 제공한다. 보통 다음과 같은 상황에서 설치된다.

```bash
xcode-select --install
```

이 패키지에는 주로 이런 것들이 포함된다.

- `clang`, `clang++`, `make` 같은 빌드 도구
- 일부 SDK 헤더와 시스템 개발용 파일
- 애플이 제공하는 기본 `git` 같은 커맨드라인 도구

즉 C 계열 컴파일러가 필요하거나, 어떤 패키지가 소스 빌드를 요구하거나, 터미널에서 기본 개발도구를 써야 할 때는 CLT만으로도 충분한 경우가 많다.

다만 CLT는 어디까지나 최소 세트다. iOS 시뮬레이터를 돌리거나, `xcodebuild`로 앱을 아카이브하거나, 특정 플랫폼 SDK 전체가 필요한 작업은 결국 Xcode IDE 쪽이 필요하다.

### 어떤 걸 설치해야 할까

대략 이렇게 생각하면 편하다.

- 웹 개발, 백엔드 개발, CLI 기반 개발만 한다면: 우선 CLT로 충분한 경우가 많다.
- iOS, macOS 앱 개발을 하거나 시뮬레이터가 필요하다면: Xcode IDE가 필요하다.
- 둘 다 할 수 있느냐: 가능하다. 보통 Xcode를 설치하면 더 큰 범위의 개발도구가 제공되고, 필요에 따라 CLT 경로나 Xcode 경로를 선택해 쓴다.

## xcode-select는 무엇을 선택하는가

`xcode-select`는 이름만 보면 Xcode 앱을 선택하는 도구처럼 보이지만, 정확히는 "현재 사용할 active developer directory"를 정하는 도구다.

이 표현이 중요한 이유는 macOS가 Apple 개발도구를 찾을 때 단순히 `/Applications/Xcode.app`만 보는 것이 아니기 때문이다. 실제로는 다음 같은 개발자 디렉터리를 기준으로 본다.

- Xcode IDE의 경우: `/Applications/Xcode.app/Contents/Developer`
- Command Line Tools의 경우: `/Library/Developer/CommandLineTools`

즉 `xcode-select`가 하는 일은 "앞으로 애플 개발도구를 어디 기준으로 찾을지"를 정하는 것이다.

가장 자주 쓰는 명령은 다음과 같다.

```bash
xcode-select -p
```

현재 선택된 developer directory를 출력한다.

```bash
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
```

앞으로는 Xcode IDE 쪽 도구를 기준으로 사용하겠다는 뜻이다.

```bash
sudo xcode-select -s /Library/Developer/CommandLineTools
```

앞으로는 Command Line Tools를 기준으로 사용하겠다는 뜻이다.

```bash
sudo xcode-select --reset
```

수동 설정을 초기화해서 시스템 기본 상태로 되돌린다.

### 왜 이런 선택이 필요한가

이 구조는 특히 여러 버전의 Xcode를 함께 설치했을 때 중요해진다. 예를 들어 안정판 Xcode와 베타 Xcode를 같이 깔아두고 상황에 따라 전환할 수 있다.

```bash
sudo xcode-select -s /Applications/Xcode-beta.app/Contents/Developer
```

그러면 이후 `xcrun`, `xcodebuild` 등이 베타 Xcode에 들어 있는 SDK와 도구를 기준으로 동작하게 된다.

## xcrun은 무엇을 하는가

`xcrun`은 현재 선택된 developer directory를 기준으로 실제 도구의 경로를 찾아서 실행해주는 도구다.

즉 비유하자면 이렇다.

- `xcode-select`: 어느 작업장을 기준으로 할지 정한다.
- `xcrun`: 그 작업장에서 실제 공구를 찾아 꺼내 쓴다.

예를 들어 `clang`이나 `simctl`의 정확한 경로를 매번 외울 필요 없이 `xcrun`에게 물어볼 수 있다.

```bash
xcrun --find clang
xcrun --find xcodebuild
xcrun --find simctl
```

특정 SDK 경로를 확인할 때도 자주 사용한다.

```bash
xcrun --show-sdk-path
xcrun --sdk macosx --show-sdk-path
```

아예 실행까지 맡길 수도 있다.

```bash
xcrun clang --version
xcrun simctl list devices
```

여기서 중요한 점은 `xcrun`이 임의로 도구를 찾는 것이 아니라, 현재 `xcode-select`로 지정된 developer directory를 기준으로 찾는다는 것이다. 그래서 둘은 거의 항상 함께 이해해야 한다.

## 둘의 관계를 한 번에 이해하기

둘의 관계를 한 문장으로 줄이면 이렇다.

> `xcode-select`가 기준 개발환경을 정하고, `xcrun`이 그 기준 아래에서 필요한 실행 파일과 SDK를 찾는다.

예를 들어 현재 선택된 경로를 확인해보면:

```bash
xcode-select -p
```

그 다음 그 환경에서 `clang`이 어디에 있는지 확인할 수 있다.

```bash
xcrun --find clang
```

또 같은 방식으로 macOS SDK 경로도 확인할 수 있다.

```bash
xcrun --sdk macosx --show-sdk-path
```

즉 어떤 머신에서 빌드 스크립트가 실패할 때는 종종 단순히 "도구가 없는가"만 볼 게 아니라, "지금 어떤 developer directory가 선택되어 있는가"까지 같이 봐야 한다.

## 설치와 확인은 이렇게 하면 된다

처음 macOS 개발환경을 잡을 때는 보통 아래 순서로 확인하면 된다.

### 1. Command Line Tools 설치 여부 확인

```bash
xcode-select -p
```

정상 경로가 나오면 일단 Apple 개발도구의 기준 경로는 잡혀 있는 상태다. 아무것도 설치되어 있지 않다면 설치 안내가 나오거나 잘못된 경로 오류가 날 수 있다.

필요하면 다음으로 설치한다.

```bash
xcode-select --install
```

### 2. Xcode IDE가 필요한지 판단

다음 작업이 필요하면 Xcode IDE를 설치하는 편이 낫다.

- iOS 앱 빌드
- 시뮬레이터 실행
- `xcodebuild` 기반 아카이브/서명
- 특정 Apple 플랫폼 SDK 사용

### 3. 현재 어떤 도구가 잡히는지 확인

```bash
xcrun --find clang
xcrun --find xcodebuild
xcrun --sdk macosx --show-sdk-path
```

이렇게 보면 현재 선택된 환경에서 어떤 도구를 실제로 쓸 수 있는지 확인할 수 있다.

## 자주 만나는 오류

이 주제를 이해해야 하는 가장 현실적인 이유는 오류 메시지 때문이다. 특히 많이 보는 것은 다음 두 가지다.

### `invalid active developer path`

보통 예전에 설치되어 있던 Xcode나 CLT가 삭제되었는데, `xcode-select`가 여전히 그 옛 경로를 가리킬 때 발생한다.

이럴 때는 다음 순서로 보면 된다.

```bash
xcode-select -p
sudo xcode-select --reset
```

그래도 안 되면 CLT를 다시 설치하거나, Xcode를 설치한 뒤 명시적으로 경로를 지정한다.

```bash
xcode-select --install
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
```

### 어떤 머신에서는 되는데 다른 머신에서는 안 되는 경우

이런 경우는 PATH 차이만 볼 게 아니라, 다음도 같이 확인해야 한다.

- CLT만 설치되어 있는가, Xcode까지 설치되어 있는가
- 현재 `xcode-select`가 어느 경로를 가리키는가
- 해당 경로 아래에 필요한 SDK나 도구가 실제로 있는가

CI 환경이나 회사 지급 장비처럼 여러 머신을 오갈 때는 이 차이가 꽤 자주 문제를 만든다.

## 내 기준에서 정리하면

정리해보면 macOS의 개발환경에서 Xcode와 관련된 요소는 이렇게 이해하면 편하다.

- Xcode IDE는 애플 플랫폼 개발을 위한 전체 작업장이다.
- Command Line Tools는 터미널 개발을 위한 최소한의 공구 세트다.
- `xcode-select`는 지금 어떤 작업장을 기준으로 할지 정한다.
- `xcrun`은 그 작업장에서 실제 공구와 SDK를 찾아 실행한다.

웹 개발이나 일반적인 CLI 작업만 할 때는 Homebrew나 asdf로 대부분의 도구를 따로 설치해 쓸 수 있다. 그래도 어떤 순간에는 결국 Apple이 관리하는 개발도구 체계와 맞닿게 된다. 특히 소스 빌드가 필요한 패키지를 설치하거나, `clang`이 필요하거나, 어떤 스크립트가 내부적으로 `xcrun`을 호출할 때가 그렇다.

그래서 macOS에서 개발을 오래 할수록 "Xcode를 설치했는가"보다 더 중요한 질문은 이것에 가까워진다.

"지금 이 머신은 어떤 Apple 개발환경을 기준으로 삼고 있는가?"

그 질문에 답하는 도구가 `xcode-select`이고, 그 환경에서 실제 실행 파일을 찾아 쓰는 도구가 `xcrun`이다.

이 둘을 같이 이해해두면, macOS 개발환경에서 만나는 꽤 많은 문제를 훨씬 덜 막막하게 볼 수 있다.
