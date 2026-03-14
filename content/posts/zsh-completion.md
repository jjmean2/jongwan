+++
date = '2026-02-09T22:42:35+09:00'
slug = 'zsh-completion-configuration'
draft = false
title = 'zsh 자동완성 스크립트 설정하기'
+++

##  CLI 도구들이 제공하는 Completion 스크립트

CLI 도구를 사용하다 보면, 자동완성 스크립트를 제공하는 도구들이 은근히 있다. 예를 들면, `pnpm`은 [이 문서](https://pnpm.io/completion)에서 보듯이 다음 명령으로 자동완성 스크립트를 얻을 수 있다.

```sh
$ pnpm completion zsh
```

Go로 작성한 정적 사이트 생성기인 [Hugo](https://gohugo.io/commands/hugo_completion_zsh/)의 경우에도 다음 명령으로 자동완성 스크립트를 얻을 수 있다.

```sh
$ hugo completion zsh
```

그런데 이 스크립트를 어떻게 하라는 걸까? 특히 [`pnpm` 쪽 문서](https://pnpm.io/completion)에는 zsh에서 스크립트를 어떻게 하라는 지시가 없어서 당황했던 기억이 있다.

대개는 Shell 초기화 파일(`~/.zshrc`)에서 이 스크립트를 `source`로 실행시키면 자동완성이 활성화되는 듯하다. `hugo`의 경우에도 다음과 같이 출력된 스크립트를 `source`하도록 가이드하고 있고,

```sh
# <(command) 는 프로세스 치환으로 command 결과가 임시 파일인 것처럼 취급해서 그 파일 경로를 인자로 전달한 것처럼 동작한다.
$ source <(hugo completion zsh)
```

pnpm에서도 `bash`의 경우에는 동일하게 출력된 스크립트를 `source` 하도록 지시한다.

```sh
$ pnpm completion bash > ~/completion-for-pnpm.bash
$ echo 'source ~/completion-for-pnpm.bash' >> ~/.bashrc
```

실제로 `pnpm completion zsh` 스크립트도 `source`하면, 기대한 대로 자동 완성이 잘 동작한다.

```sh
$ source <(pnpm completion zsh)
```

그런데 이 completion 스크립트는 어떤 일을 하길래 자동완성을 활성화해줄까? 스크립트 내용을 들여다 보았다.

`pnpm`의 자동완성 스크립트는 다음과 같이 생겼다.

```sh
#compdef pnpm
###-begin-pnpm-completion-###
if type compdef &>/dev/null; then
  _pnpm_completion () {
    local reply
    local si=$IFS

    IFS=$'\n' reply=($(COMP_CWORD="$((CURRENT-1))" COMP_LINE="$BUFFER" COMP_POINT="$CURSOR" SHELL=zsh pnpm completion-server -- "${words[@]}"))
    IFS=$si

    if [ "$reply" = "__tabtab_complete_files__" ]; then
      _files
    else
      _describe 'values' reply
    fi
  }
  # When called by the Zsh completion system, this will end with
  # "loadautofunc" when initially autoloaded and "shfunc" later on, otherwise,
  # the script was "eval"-ed so use "compdef" to register it with the
  # completion system
  if [[ $zsh_eval_context == *func ]]; then
    _pnpm_completion "$@"
  else
    compdef _pnpm_completion pnpm
  fi
fi
###-end-pnpm-completion-###
```

여기서 구체적인 함수 정의 내용을 접고 보면, 다음과 같다. (주석을 약간 정리하였다.)

```sh
#compdef pnpm 👈 1️⃣
if type compdef &>/dev/null; then
  _pnpm_completion () { # 👈 2️⃣
    #...생략
  }
  if [[ $zsh_eval_context == *func ]]; then
    _pnpm_completion "$@" # 👈 4️⃣
  # otherwise, the script was "eval"-ed so use "compdef" to register it with the completion system
  else
    compdef _pnpm_completion pnpm # 👈 3️⃣
  fi
fi
```

이번엔 `hugo`의 completion 스크립트를 보자. 실제로 보면, pnpm보다 훨씬 복잡한데, 함수 본문을 접어 놓고 보면, 다음과 같다.

```sh
#compdef hugo 👈 1️⃣
compdef _hugo hugo # 👈 3️⃣
__hugo_debug() {
  #...생략, 이 함수는 아래의 _hugo 함수에서 사용하는 함수
}

_hugo() { # 👈 2️⃣
  #...생략
}

# don't run the completion function when being source-ed or eval-ed
if [ "$funcstack[1]" = "_hugo" ]; then
    _hugo # 👈 4️⃣
fi
```

두 코드의 골격이 꽤나 비슷하다. 스크립트마다 순서가 살짝 다르긴 하지만 다음의 요소들이 존재한다.

- 1️⃣ `#comdef <command>` 라는 주석으로 시작한다.
- 2️⃣ `_`로 시작하는 함수를 정의한다.
- 3️⃣ `compdef _<func> <command>`를 실행한다.
- 4️⃣ 특정 상황에서는 정의된 `_<func>` 함수를 호출한다. (스크립트가 "eval"되는 상황이 아닐 때)

여기서 정의하는 함수의 내용이 completion 로직일 것이다. 즉, 현재까지 입력된 값을 보고, 다음에 올 값의 후보를 결정하는 내용이 들어 있다. 이 내용은 이번 노트의 관심사가 아니다. 이건 다음에 한번 공부해 보기로 하고, 여기서는 이 completion 로직을 zsh가 어떻게 가져다 쓰는 건지를 알아보려고 한다.

## compdef: 명령과 자동완성 함수를 매핑시킨다.

zsh에서 shell에 명령을 입력하고 `<TAB>` 키를 누르면, 그 명령에 해당하는 completion 함수를 호출한다. 이 때 `words`, `CURRENT` 등등의 현재 입력 상태에 대한 변수를 세팅한 후 호출하는데, completion 함수 내에서는 이런 context 정보들을 이용해서 다음에 올 만한 단어들을 선택하는 것이다. 어떤 명령에 대한 자동 완성을 활성화한다는 것은 zsh에게 그 명령에 대해 호출할 자동완성(completion) 함수가 무엇인지 알려주는 것이다.

이렇게 알려주는 함수가 `compdef` 이다. 다음과 같이 실행할 수 있다.

```sh
compdef <자동완성 함수> <명령>
```

즉, 다음과 같이 실행하면 `pnpm` 명령에 대해 자동완성을 해야 할 때 `_pnpm` 함수를 호출하도록 등록시킨다는 의미가 된다.

```sh
$ compdef _pnpm pnpm
```

실제로 이렇게 등록을 하면, `_comps`라는 Associative Array (Map, Dictionary와 같은 자료구조) 변수에 `pnpm`, `_pnpm` 키-값 쌍이 저장된다.

```sh
$ echo $_comps[pnpm]
_pnpm
```

그리고 이 상태에서 `pnpm`을 입력 후 `<TAB>` 키를 누르면 `_pnpm`를 호출한다.

```sh
$ pnpm <TAB>
# _pnpm가 호출되면서 자동완성 후보를 선택한다.
```

`pnpm`과 `hugo`의 자동완성 스크립트에서 2️⃣와 3️⃣이 하는 일은 설명이 된 셈이다.

{{<notice>}}
2️⃣ 자동완성 함수를 정의하고, 3️⃣ 명령과 매핑시킨다.\*\*
{{</notice >}}

실제로 2️⃣와 3️⃣ 스크립트만 따로 실행시켜도 자동완성이 기대한 대로 잘 동작했다. 그럼 1️⃣과 4️⃣는 어떤 역할을 하는 걸까? 찾아보니 이건 직접 `source`로 실행시키지 않고, zsh 자동 완성 시스템이 초기화될 때 알아서 읽어가게 하는 설정을 위한 요소로 보인다.

자동 완성 시스템 초기화를 알아보기 전에 zsh 설정에서 자주 보이는 autoload 기능에 대해 먼저 알아보자.

## 함수 autoload: 함수 정의를 lazy load한다.

zsh에는 autoload 라는 기능이 있다. 함수 정의를 지연 로드(lazy load)하는 것으로 다음과 같이 어떤 함수를 autoload 하면, 함수 이름만 메모리에 등록해두고, 실제 구현 본문은 처음 함수를 호출할 때 로드하는 것이다.

```sh
# 이 때는 함수 본문이 아직 로드되지 않는댜.
$ autoload some_func

# 이 때 함수 본문이 로드되고, 바로 함수가 호출된다.
$ some_func
```

autoload는 어떤 메커니즘으로 동작할까? `autoload some_func`이라고 명령하면, zsh는 다음과 같은 함수를 정의한다.

```sh
some_func() {
  builtin autoload -X
}
```

그리고 이 함수를 호출하면, `builtin autoload -X` 부분이 동작하는데, `fpath`(function path)에서 함수 파일을 찾아서 함수 본문을 파일 내용으로 교체한 후 그 함수를 호출하는 명령이다. 이 동작을 의사코드로 표현해보면 다음과 같다.

```sh
# autoload some_func 실행 시 zsh가 내부적으로 준비하는 로직 (의사 코드)
some_func() {
  # 1. $fpath에서 some_func 파일을 찾아서 내용을 읽음
  local content=$(cat "$(find_in_fpath some_func)")

  # 2. 읽어온 내용으로 자기 자신(some_func)을 덮어씀 (진짜 정의)
  eval "some_func() { $content }"

  # 3. 새로 정의된 진짜 some_func을 즉시 실행
  some_func "$@"
}
```

여기서 `fpath`는 function search path라는 의미로 command search path인 `path`와 비슷하게 경로들의 배열을 담고 있는 변수다. `fpath`가 가리키는 경로에는 함수로 해석될 파일들이 들어있는데, 파일 이름이 함수명이 되고, 파일 내용이 함수 본문이 된다. 예를 들어, `hello_world` 이라는 파일의 본문이 다음과 같다면,

```sh
local greeting="hello world"
echo $greeting
```

autoload에 의해 이 파일이 함수로 로드되면, 다음과 같은 함수가 된다.

```sh
hello_world() {
  local greeting="hello world"
  echo $greeting
}
```

이 `fpath`는 autoload 뿐 아니라 자동완성 시스템에서 자동완성 함수를 찾을 때도 사용된다.

## compinit: 자동완성 시스템 초기화

zsh에서 자동완성이 동작하려면 먼저 `compinit` 함수를 실행하여 자동완성 시스템을 활성화시켜야 한다. `compinit`은 `fpath`에 들어있는 autoload 함수이므로 다음과 같이 실행할 수 있다.

```sh
# autoload 후 바로 실행까지 한다.
autoload -Uz compinit && compinit
```

참고로 `autoload`의 `-U` 옵션은 함수가 로드될 때 alias expansion을 억제하는 옵션이고, `-z` 옵션은 zsh 스타일로 autoload하는 옵션이라고 한다. zsh에서 autoload 할 때 으레 붙이는 옵션인 듯하다.

`compinit`은 자동완성이 동작하게 하기 위해 여러 가지 일을 한다. `compdef` 함수도 `compinit`이 정의한다. 그리고 초기화 시점에 `fpath`를 돌면서 자동완성 함수를 찾아서 명령과의 매핑을 미리 작성하는 일도 하는데, 이 때 자동완성 함수로 인식되는 것은 다음 조건을 만족하는 함수 파일이다.

- `fpath` 안에 존재하고 이름이 `_`로 시작한다.
- 파일의 첫줄에 `#compdef <명령>` 같은 특수 주석이 존재한다.

`compinit`은 이런 파일을 찾으면, 다음과 같은 명령을 실행한다.

```sh
# -n는 기존에 명령에 대한 매핑이 없을 때만 매핑 추가하는 옵션
# -a 는 지정한 함수를 autoload하는 옵션
compdef -na <함수파일명> <명령>
```

즉, 이 조건에 맞춰서 자동완성 함수 파일을 작성하고, `fpath` 안에 넣어두면, 직접 `compdef`을 하지 않아도 `compinit`을 할 때 알아서 명령에 대한 자동완성을 활성해주는 것이다. `pnpm`과 `hugo`의 자동완성 스크립트에서 함수 이름을 `_`로 시작한 것과 `#compdef pnpm`, `#compdef hugo` 같은 주석(1️⃣)이 있는 이유가 설명된다. 이 스크립트들은 직접 `source`를 해도 되지만, `fpath` 안에 넣어도 되도록 설계한 것이다.

## 정의한 함수를 직접 호출하는 4️⃣ 부분은 왜 있는 걸까?

자동완성 함수는 `fpath`에 들어있으니 autoload될 수 있고, autoload된 함수는 처음 호출할 때 정의와 동시에 함수 호출까지 해주므로 autoload 함수 파일 본문에서 호출하는 코드를 직접 작성할 필요가 없다. 그러나 `pnpm`, `hugo` 등에서 제공하는 자동완성 스크립트는 `source`로도 동작하게 하기 위해서인지 일반적인 autoload 함수와는 다른 구조로 되어 있는데, 그래서 `fpath`에 배치해서 자동으로 읽어가게 설정한 경우, 4️⃣ 부분이 없으면 첫 자동완성 때 아무 반응이 없는 문제가 생길 수 있다.

예를 들어 `pnpm completion zsh`의 결과를 `fpath`에 `_pnpm_completion`이라는 파일로 작성한다고 해보자. `source`로 동작하는 것을 신경쓰지 않는다면, 원래 스크립트의 `_pnpm_completion` 본문만 가져와서 다음과 같이 작성하면 된다.

```sh
#compdef pnpm

local reply
local si=$IFS

IFS=$'\n' reply=($(COMP_CWORD="$((CURRENT-1))" COMP_LINE="$BUFFER" COMP_POINT="$CURSOR" SHELL=zsh pnpm completion-server -- "${words[@]}"))
IFS=$si

if [ "$reply" = "__tabtab_complete_files__" ]; then
  _files
else
  _describe 'values' reply
fi
```

실제로 이렇게 파일을 만들어서 `fpath`가 가리키는 경로 중 하나 안에 넣어두면, 자동 완성이 잘 동작한다. 그런데 만약, `pnpm completion zsh` 결과를 통째로 자동완성 함수 파일로 만든다고 하면 어떻게 될까? 이렇게 하는 게 가이드하기가 더 쉽기 때문에 보통은 이렇게 가이드할 것이다.

```sh
$ pnpm completion zsh > /usr/local/share/zsh/site-functions/_pnpm_completion
```

그럼 다음 스크립트는 사용자가 직접 `source`하는 스크립트가 아니라 `compinit`에서 `pnpm`의 자동완성 함수로 autoload를 걸어두는 자동완성 함수가 된다. 즉, `pnpm <TAB>`으로 처음 자동 완성을 시도할 때 다음 본문에서 정의하는 `_pnpm_completion` 함수가 실행되는 게 아니라, 다음 본문 자체가 실행된다는 것이다.

```sh
#compdef pnpm
###-begin-pnpm-completion-###
if type compdef &>/dev/null; then
  _pnpm_completion () {
    local reply
    local si=$IFS

    IFS=$'\n' reply=($(COMP_CWORD="$((CURRENT-1))" COMP_LINE="$BUFFER" COMP_POINT="$CURSOR" SHELL=zsh pnpm completion-server -- "${words[@]}"))
    IFS=$si

    if [ "$reply" = "__tabtab_complete_files__" ]; then
      _files
    else
      _describe 'values' reply
    fi
  }
  # When called by the Zsh completion system, this will end with
  # "loadautofunc" when initially autoloaded and "shfunc" later on, otherwise,
  # the script was "eval"-ed so use "compdef" to register it with the
  # completion system
  if [[ $zsh_eval_context == *func ]]; then
    _pnpm_completion "$@"
  else
    compdef _pnpm_completion pnpm
  fi
fi
###-end-pnpm-completion-###
```

이 첫 자동완성 시도시 실행 흐름을 한번 따라가 보자.

```sh
if type compdef &>/dev/null; then
```

이 부분은 `compdef` 함수가 존재하는지 검사하는 것이다. 자동완성 시스템이 초기화되어 있는지를 보는 것으로 초기화되어 있으니 이 안으로 진입한다.

```sh
  _pnpm_completion () {
    #...생략
  }
```

`_pnpm_completion` 함수를 정의한다. 현재 실행 중인 함수 이름도 autoload된 `_pnpm_completion`이다. 즉, 이 부분을 만나면, `_pnpm_completion` 함수가 이 부분에서 정의하는 진짜 pnpm 자동완성 함수로 덮어써진다. 이후 다시 자동완성을 시도할 때는 진짜 함수가 호출될 것이다. 문제는 현재의 첫 자동완성 시도이다. 이대로 끝나버리면, 처음 `<TAB>` 했을 때는 아무 일이 일어나지 않게 된다.

```sh
  # When called by the Zsh completion system, this will end with
  # "loadautofunc" when initially autoloaded and "shfunc" later on, otherwise,
  # the script was "eval"-ed so use "compdef" to register it with the
  # completion system
  if [[ $zsh_eval_context == *func ]]; then
    # 함수로서 호출된 경우, 즉, 자동완성 시도시 호출될 때 진입
    # 진짜 자동완성 함수를 호출한다.
    _pnpm_completion "$@"
  else
    # 스크립트로서 source 또는 eval된 경우, 즉, ~/.zshrc 에서 source 한 경우 진입
    # 명령과 자동완성 함수를 매핑한다.
    compdef _pnpm_completion pnpm
  fi
```

여기서 `[[ $zsh_eval_context == *func ]]` 조건은 원래의 영문 주석이 설명하듯이 스크립트가 함수로서 호출되었는지 스크립트로서 source되었는지를 구분하기 위한 조건인 것으로 보인다. `zsh_eval_context`라는 변수는 zsh가 세팅해주는 변수로 보이는데, 이 조건이 어떻게 동작하는지는 그냥 작성자를 믿는 걸로 하고, 분기의 의미를 이해해보자.

함수로 호출된 경우는 실제 자동완성 함수로서 호출된 경우를 의미한다. 위에서 살펴봤듯이 두 번째 시도부터는 `_pnpm_completion` 함수가 진짜 자동완성 함수로 교체되므로, 이건 첫 번째 시도 때에만 실행될 코드인데, 이 때 자동완성 동작이 무시되지 않게 하기 위해 `_pnpm_completion`을 한번 호출해주는 것이다. 실제로 이 코드를 주석처리해서 테스트해보면, 처음 탭했을 때는 씹히고, 두 번째 탭할 때부터 자동완성이 동작하는 것을 볼 수 있다.

그 외의 경우는 스크립트로서 `source` 또는 `eval`로 실행된 경우인데, 이건 `fpath`에 넣지 않고 `~/.zshrc`에서 직접 `source <(pnpm completion zsh)`로 설정해서 실행된 경우일 가능성이 높다. 이때는 명령과 자동완성 함수가 자동으로 매핑되지 않으므로 여기서 직접 `compdef`로 매핑을 해주는 것이다.

## 자동완성 스크립트를 설정하는 방법 정리

지금까지 알아본 바탕으로 자동완성 스크립트를 어떻게 설정하면 될지 정리해보자.

### 1. (필요하면) 커스텀 fpath 경로 추가

이건 필수 과정은 아니다. 그런데 직접 수동으로 작성하거나 배치하는 autoload 함수, 자동완성 함수는 시스템이나 다른 도구들이 설치하는 것들과 구분해두는 게 기억하고 관리하기 편할 것 같아서 전용 디렉토리를 하나 만들어서 `fpath`에 추가하였다.

```sh
fpath+=(~/.jongwan/share/zsh/site-functions)
```

### 2. compinit 초기화

자동완성이 동작하려면 `compinit` 초기화는 필수다. 다른 도구들이 제공하는 초기화 스크립트 내에서 이걸 하는 경우도 있는데, 명시적으로 하는 게 좋은 듯하다. 커스텀 `fpath`를 사용하려면, `compinit` 호출 전에 세팅되어 있어야 커스텀 경로에서도 자동완성 함수를 찾는다.

```sh
autoload -Uz compinit && compinit
```

### 3-1. 자동완성 함수를 정의하고 직접 compdef

자동완성 함수를 작성하는 법은 따로 공부해야 할 주제다. 보통은 도구 제작자가 제공하는 스크립트에서 뽑아 쓰면 된다. 직접 `compdef`을 할 거면, 함수 이름이 꼭 `_`로 시작할 필요는 없지만, 관례적으로 `_`로 시작하는 것 같다. 커맨드 앞에 `_`만 붙이면 되니 이름 짓기도 편하다. 다음 스크립트를 `~/.zshrc`에 추가하면 된다.

```sh
# ~/.zshrc에 추가
_pnpm() {
  #...자동완성 로직 생략
}
compdef _pnpm pnpm
```

제공되는 스크립트에서는 스크립트를 통째로 `source`하면 되도록 제공하는 경우가 많다. 그 경우 `~/.zshrc`에 다음을 추가하면 된다.

```sh
# ~/.zshrc에 추가
source <(pnpm completion zsh)
```

### 3-2. compinit이 읽어갈 수 있는 방식으로 작성하여 fpath에 배치

`_pnpm`와 같이 파일명을 `_`로 시작하게 하고, 본문에 자동완성 함수의 본문 로직만 담아서 `fpath` 경로에 저장한다. 이렇게 하면, lazy load 효과도 있고, `~/.zshrc` 쪽도 복잡해지지 않아서 나는 이 방식이 마음에 든다. 파일 이름은 `_`로 시작하기만 하면 아무거나 된다.

`~/.jongwan/share/zsh/site-functions/_pnpm` 파일

```sh
#compdef pnpm

# ...자동완성 로직 생략
```

제공되는 스크립트에서 스크립트를 통째로 `fpath`에 넣으면 되도록 작성된 경우가 많다. 다만 개인적으로는 자동완성 함수 본문만 추출해서 위처럼 작성하는 것이 깔끔해 보이긴 하는데, 자동완성 함수가 여러 함수로 구성되어 있거나 복잡한 경우, 추출하는 게 쉽지 않을 수 있으니 그냥 통째로 저장하는 게 나을 수 있다.

이 때는 스크립트 내에서 정의하는 진짜 자동완성 함수와 동일한 이름의 파일로 저장하는 것을 추천한다. 이 방식으로 자동완성 함수가 매핑될 때는 파일 이름이 선택되는데 이게 진짜 자동완성 함수와 이름이 다르면, 최초 실행 후에도 진짜 자동완성 함수로 교체되지 않고, 계속 스크립트 전체가 자동완성 함수로 쓰이기 때문이다.

```sh
# 다음을 한번만 실행해 두면 된다.
$ pnpm completion zsh > ~/.jongwan/share/zsh/site-functions/_pnpm_completion
```
