+++
date = 2026-03-27T10:41:18+09:00
slug = 'xcode-select-and-xcrun'
draft = true
title = 'xcode-select와 xcrun에 대해서'
+++

## macOS에 개발환경 설치하기

맥을 새로 사고 `git` 명령을 쓰려고 하면, 다음과 같은 프롬프트가 떴던 것 같다.

```
The "xcode-select" command requires the command line developer tools. Would you like to install the tools now?
```

오래 되어서 문구라든가 프롬프트가 뜨는 경로는 정확히 기억이 안 나는데 macOS에는 `git`이 기본 설치되어 있지 았았던 것은 확실하다. 맥은 개발자만 쓰는 게 아니므로 `git` 같은 개발 도구는 필요한 사람만 설치하도록 되어 있다.

### 애플의 메인 IDE, Xcode

Xcode는 애플이 자랑(?)하는 IDE이다. iOS 개발을 위해서는 이게 필수품이고, 나는 iOS로 개발에 입문했기에 내가 업무에서 사용했던 첫 IDE이기도 해서 애정이 있는 편이다. 프론트엔드가 주 분야가 된 지금은 거의 안 쓰고 있지만, 설치는 항상 한다. App Store에서 Xcode를 설치하면, `git`을 비롯해서 `make`, `clang` 등 빌드 도구와 macOS, iOS 등의 애플 플랫폼용 SDK들이 앱 안에 번들링되어 같이 설치된다. Xcode와 같이 설치되는 `git`, `clang`의 경로를 살펴보면 다음과 같다.

```sh
# git의 위치
/Applications/Xcode.app/Contents/Developer/usr/bin/git
^^^^^^^^^^^^^^^^^^^^^^^^
      앱 패키지

# clang의 위치
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang
^^^^^^^^^^^^^^^^^^^^^^^^
      앱 패키지

```

여기서 `/Applications/Xcode.app`는 Xcode 앱 패키지의 위치다.
![Xcode path](/images/xcode-path.png)

{{< figure src="/images/xcode-path.png" alt="설명" width="300px" caption="캡션" >}}

iOS 개발 같은 경우에는 Xcode를 쓰지 않고 개발 환경을 잡기가 어렵지만, 프론트엔드 개발이나 서버 개발의 경우 Xcode 앱 자체는 거의 쓰지 않는다. 하지만, Xcode에서 제공하는 SDK와 각종 빌드 도구들은 개발 환경을 구성하기 위해 꼭 필요하기 때문에 애플에서는 이런 개발 도구들만 따로 패키징해서 [Command Line Tools for Xcode](https://developer.apple.com/download/all/?q=command%20line%20tools)를 제공한다.

즉, Xcode를 이용해 개발을 해야 하는 사람은 Xcode를 설치하고, Xcode가 필요 없는 사람은 Command Line Tools for Xcode를 설치하면 되는 것이다.

### Command Line Tools for Xcode 설치

Command Line Tools for Xcode(이하 CLT)는 애플에서 제공하는 [다운로드 사이트](https://developer.apple.com/download/all/?q=command%20line%20tools)에서 버전을 선택해서 설치하거나 다음 명령어로 최신 버전을 설치할 수 있다.

```sh
$ xcode-select --install
```

CLT는 다음 위치에 설치되고, 동시에 여러 버전을 설치할 수 없다.

```sh
/Library/Developer/CommandLineTools/
```



> [!TIP]+
> 이렇게 설치한 버전을 업데이트하려면 시스템 설정의 소프트웨어 업데이트에서 하면 된다. 업데이트할 게 있는 경우 이쪽에 뜬다.
> {{< figure src="/images/software-update.png" caption="시스템 설정 > 소프트웨어 업데이트" >}}


CLT에 설치된 `git`과 `clang`의 위치는 다음과 같다. (macOS 26.3.1 버전 기준)

```sh
/Library/Developer/CommandLineTools/usr/bin/git
/Library/Developer/CommandLineTools/usr/bin/clang
```

## `xcode-select`로 개발환경 선택하기

### macOS의 `/usr/bin` 에 들어있는 도구들 중 상당 부분은 shim이다.

shim은 틈을 메우거나 간격을 조절하는 얇은 조각을 뜻하는 것으로 가구의 수평을 맞추기 위해 한쪽 다리 밑에 괴어 놓는 나무조각 같은 걸 말한다고 한다. (참고: [Shim이란 무엇인가?](https://www.youtube.com/watch?v=BMaBWfsPi3c))

![영한 사전 shim](/images/dictionary-shim.png)

       소프트웨어 업계에서 shim은 보통 호환성이이나

소프트웨어 업계에서 shim은 보통 호환성이나 버전 관리를 위해 API 호출을 가로채서 다른 도구를 호출해주는 작은 코드 조각을 가리킨다.

> #### Shim (computing)
>
> In computer programming, a shim is a library that transparently intercepts API calls and changes the arguments passed, handles the operation itself or redirects the operation elsewhere.[1][2] Shims can be used to support an old API in a newer environment, or a new API in an older environment. Shims can also be used for running programs on different software platforms than they were developed for.
>
> https://en.wikipedia.org/wiki/Shim_(computing)

나는 `nvm`(Node 버전 관리 도구) `rbenv`(Ruby 버전 관리 도구), `asdf`(온갖 툴의 버전 관리 도구) 등의 버전 관리 도구에서 이 단어를 접했는데, 명령어를 대신 받아서 미리 지정한 버전의 명령을 찾아서 실행해주는 작은 스크립트들을 shim이라고 부른다. 그런데 macOS의 기본 path에 설치된 `/usr/bin/git` 같은 도구들도 이와 동일한 역할을 하는 shim이다.

`/usr/bin`에는 `git` 외에도 여러 shim들이 존재하는데, 이것들 덕분에 우리가 실제 도구의 위치를 다 입력하지 않아도 된다.

```sh
# 이렇게 명령하는 게 아니라
$ /Applications/Xcode.app/Contents/Developer/usr/bin/git commit

# 이렇게 명령할 수 있게 해준다.
$ git commit
```

이 뿐만 아니라 `/usr/bin/git` 같은 shim은 Xcode나 CLT를 설치하기 전에도 존재하는데 아직 실제 `git`이 설치되어 있지 않았을 때 `git` 명령을 시도하면, Xcode나 CLT를 설치하라고 프롬프트를 띄워주는 것도 이 shim이 하는 일이다.

### Active Developer Directory

그럼 `/usr/bin/git`은 어디에 있는 파일을 실행할까? 시스템에는 Xcode가 설치되어 있을 수도 있고, CLT가 설치되어 있을 수도 있다. 그리고 둘다 설치되어 있을 수도 있으며, Xcode가 여러 버전이 설치되어 있을 수도 있다. `/usr/bin/git`은 어디에 있는 실행파일을 실행해야 할까?

이 때 등장하는 것이 Active Developer Directory라는 개념이다. `/usr/bin/git`은 여러 개발환경이 설치된 디렉토리 중 미리 지정된 Active Developer Directory에 들어 있는 `git` 실행파일을 실행한다. 이렇게 지정된 개발환경을 찾아가는 것이 `/usr/bin/git` 같은 shim 파일들의 주요 역할이다. 그리고 Active Developer Directory를 설정하는 도구가 `xcode-select`이다. macOS에서 `xcode-select`의 manual page를 보면, Xcode 와 BSD 도구를 위한 Active Developer Directory를 관리하는 툴이라고 설명한다. `asdf`로 NodeJS 여러 버전을 설치해두고 필요에 따라 바꿔가면서 사용하는 것과 동일한 개념이라고 볼 수 있겠다.

```
NAME
       xcode-select - Manages the active developer directory for Xcode and BSD tools.

SYNOPSIS
       xcode-select [-h|--help] [-s|--switch <path>] [-p|--print-path] [-v|--version]
```

`xcode-select`에서 `-p`, `--print-path` 옵션은 현재 설정된 Active Developer Directory를 출력하는 명령인데, 내 컴퓨터에서 해 보니 다음과 같이 Xcode 앱 내부의 `Developer` 라는 디렉토리를 가리켰다. 위에서 봤던 `git` 실행파일이 있던 디렉토리다.

```sh
$ xcode-select -p
/Applications/Xcode.app/Contents/Developer
```

```sh
# git 실행파일이 있던 위치
/Applications/Xcode.app/Contents/Developer/usr/bin/git
#-----------------------------------------============
# Developer Directory                     시스템 git 경로와 동일한 모양

# clang 실행파일이 있던 위치
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang
#-----------------------------------------                                    ==============
# Developer Directorty                                             시스템 clang 경로와 동일한 모양
```

즉, `/Applications/Xcode.app/Contents/Developer`이 Active Developer Directory이고, 시스템 명령은 여기서 실행 파일을 찾아서 실행한다는 것이다. CLT만 설치한 경우, Active Developer Directory는 다음과 같이 출력된다.

```sh
$ xcode-select -p
/Library/Developer/CommandLineTools
```

개발 환경이 여러 개 설치되어 있는 경우, `xcode-select`로 이 중 하나를 Active Developer Directory로 선택하거나 기본값으로 리셋할 수 있다. 다음 명령은 모두 root 권한이 필요하다.

```sh
# CLT를 active 개발 환경으로 선택, --switch 옵션
$ sudo xcode-select -s /Library/Developer/CommandLineTools

# 별도로 다운로드한 Xcode 베타 버전을 개발 환경으로 선택
$ sudo xcode-select -s "/Users/ljw/dev/other-xcode/Xcode Beta.app/Contents/Developer"

# 기본값으로 리셋, --reset 옵션, Xcode가 있는 경우, Xcode가 선택되는 듯
$ sudo xcode-select -r
```

예전에 iOS 개발을 할 때는 실제로 새로운 Xcode 버전 배포 전후로 두 개의 버전을 유지하면서 써야 할 경우가 있었는데, 그 때 뭔가 버전 문제로 빌드가 잘 안 될 때, `xcode-select`로 개발 환경을 적절히 선택해야 해결되는 경우가 있었다.

## Active Developer Directory에 있는 도구를 `xcrun`으로 찾거나 실행하기

Xcode나 CLT에 설치된 도구들은 상당 부분 `git`처럼 `/usr/bin/` 쪽에 wrapper(shim)가 마련되어 있는 듯하다. 그러나 모든 도구가 다 그런 것은 아니고, 보통 Xcode나 CLT 설치 위치를 path 설정에 넣어놓진 않기에 wrapper가 없는 도구를 간결하게 실행하기가 어렵다. 물론 path 설정에 넣어놓는 방법도 있겠지만, 이를 도와줄 `xcrun`라는 도구가 존재한다.

`xcrun`은 개발 도구를 실행하거나 위치를 찾아주는 도구인데, `xcode-select`로 선택한 Active Developer Directory를 반영한다. Active로 지정한 개발 환경에 들어있는 도구를 실행해주는 것이다.

```
NAME
       xcrun - Run or locate development tools and properties.

SYNOPSIS
       xcrun [--sdk <SDK name>] --find <tool name>

       xcrun [--sdk <SDK name>] <tool name> ... tool arguments ...

       <tool name> ... tool arguments ...
```

나는 이 명령을 `simctl`을 사용할 때 처음 봤다. `simctl`은 iOS 시뮬레이터 관련한 조작을 할 때 사용하던 명령인데 항상 `xcrun simctl` 형태로 사용했어야 해서 처음에는 `xcrun`이라는 도구의 서브커맨드인 줄 알았다.

```sh
# 사용 가능한 시뮬레이터 목록 출력
$ xcrun simctl list

# 지정한 시뮬레이터 부팅
$ xcrun simctl boot C86A559A-1F50–40D1–8D84–954EDFBBCE18

# 부팅된 시뮬레이터의 스크린샷 찍기
$ xcrun simctl io booted screenshot screen.png
```

그런데 지금 보니, `simctl`은 별개의 도구이고, Xcode 디렉토리 안에 들어 있어서 접근이 불편한 것을 `xcrun`을 통해 찾아서 실행하는 것이었다.

`simctl` 뿐 아니라 `git`이나 `clang` 처럼 같은 다른 도구들도 `xcrun`을 통해 실행시킬 수 있다.

```sh
$ xcrun git status

$ xcrun clang --version
```

`--find` 옵션을 쓰면, 실행시키는 대신에 명령의 위치를 찾아서 출력한다. 이 위치는 `xcode-select`로 설정한 Active Developer Directory에 따라 달라진다.

```sh
$ xcrun --find simctl
/Applications/Xcode.app/Contents/Developer/usr/bin/simctl

$ xcrun --find clang
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang
```

예를 들어 Active Developer Directory를 CLT로 바꾸고 출력하면 다음과 출력 경로가 바뀐다.

```sh
$ sudo xcode-select -s /Library/Developer/CommandLineTools

# simctl은 Xcode에만 있는 도구이므로 CLT를 선택한 경우, 못 찾는다.
$ xcrun --find simctl
xcrun: error: unable to find utility "simctl", not a developer tool or in PATH

$ xcrun --find clang
/Library/Developer/CommandLineTools/usr/bin/clang
```

### `DEVELOPER_DIR` 환경변수

Active Developer Directory는 `DEVELOPER_DIR` 환경 변수로도 설정할 수 있다. 이 환경 변수가 있는 경우, `xcode-select -s`로 설정한 값보다 환경 변수의 값을 우선 사용한다고 한다. 이를 이용하면, 전체 설정을 바꾸지 않고, 특정 명령 하나만 Active Developer Directory를 바꿔서 실행하는 것도 가능하다.

```sh
# Active Developer Directory를 Xcode로 설정
$ sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
$ xcode-select -p
/Applications/Xcode.app/Contents/Developer

# 이번만 Active Developer Directory를 CLT로 바꿔서 명령
$ DEVELOPER_DIR=/Library/Developer/CommandLineTools xcode-select -p
/Library/Developer/CommandLineTools

# 여전히 시스템에는 Xcode로 설정되어 있음
$ xcode-select -p
/Applications/Xcode.app/Contents/Developer
```

### `xcode-select`는 현재 설정을 어디에 저장하고 참조할까?

예를 들어 `asdf`는 현재 버전 설정 정보를 `.tool-versions` 파일에 다음과 같은 형식으로 저장하고 참조한다.

```sh
ruby 3.3.0
nodejs 24.12.0
golang 1.26.0
bun 1.3.10
deno 2.0.6
python 3.15.3
flutter 3.19.5-stable
```

global 설정은 Home 폴더에 이 `.tool-versions` 파일을 저장한다.

이런 식으로 `xcode-select`도 `xcode-select -s`로 지정한 Active Developer Directory 값을 저장하는 곳이 있을텐데 이것에 대한 답은 못 찾았다.

이와 비슷한 질문을 하는 Stack Overflow 글은 찾았는데 [Where does xcode-select store information - Stack Overflow](https://stackoverflow.com/questions/14609738/where-does-xcode-select-store-information)이다. 여기서는 `/var/db/xcode_select_link`라는 파일을 언급했다. 실제로 `xcode-select`의 설정에 따라 이 파일이 생성되고 업데이트되긴 했는데, 이 파일을 삭제해도 `xcode-select`의 출력은 영향을 받지 않았다. 즉, 확실하진 않지만, 이 파일은 부산물이고, `xcode-select`가 참조하는 source of truth는 아닌 것으로 보였다. 아마도 내부적으로 따로 관리하는 DB가 있지 않을까? 이에 대한 정보는 찾기 어려워서 더 찾아보지는 않았다.

```sh
# 아무 설정도 하기 전에는 Xcode를 가리킨다. 이 때는 /var/db/xcode_select_link 라는 파일이 없다.
$ xcode-select -p
/Applications/Xcode.app/Contents/Developer
$ ls -al /var/db/xcode_select_link

# CLT를 가리키도록 설정한다.
$ sudo xcode-select -s /Library/Developer/CommandLineTools
/Library/Developer/CommandLineTools
$ xcode-select -p
/Library/Developer/CommandLineTools

# /var/db/xcode_select_link 라는 symlink가 생기고, CLT 디렉토리를 가리킨다.
$ ls -al /var/db/xcode_select_link
lrwxr-xr-x  1 root  wheel  35  3월 18 07:52 /var/db/xcode_select_link -> /Library/Developer/CommandLineTools

# symlink를 직접 지워본다.
$ sudo rm -f /var/db/xcode_select_link

# xcode-select는 여전히 CLT를 가리킨다. /var/db/xcode_select_link가 source of truth는 아닌 듯?
$ xcode-select -p
/Library/Developer/CommandLineTools
```

---

### 참고자료

- [Command-line tools | Apple Developer Documentation](https://developer.apple.com/documentation/xcode/command-line-tools)
- [xcrun, the version manager for the Xcode toolchain](https://mokagio.github.io/tech-journal/2015/03/10/xcode-xcrun.html)

# 🚀 절취선

Xcode를 설치한 경우, CLT에 있는 도구들을 모두 포함하고 있다. 거기에 Xcode로 개발하기 위한 `xcodebuild`나 `simctl` 같은 도구들이 추가로 들어있다. 따라서 Xcode가 있으면, CLT를 따로 설치할 필요가 없다. 하지만, CLT를 먼저 설치했거나 해서 시스템에 둘다 설치되는 경우도 있다. 또 Xcode도 베타 버전과 현재 버전, 과거 버전을 모두 유지하는 등 여러 버전을 가지고 있을 수도 있다.

iOS/macOS 등 애플 플랫폼을 타깃으로 한 프로그램을 만들지 않는 사람은 굳이 이 무거운 Xcode를 설치할 필요가 없다.

> On a fresh install of macOS, invoking any command in Xcode or the command-line tools package (such as git) from the command line prompts you to download and install the Command Line Tools for Xcode package.

`git`이 설치되어 있지 않은 것이다. Got it, got it. t

맥에서 `git`과 같은 명령ㅇ

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
