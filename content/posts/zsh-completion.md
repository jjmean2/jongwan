+++
date = '2026-02-09T22:42:35+09:00'
draft = true
title = '툴이 제공하는 zsh 자동완성 스크립트 설정하기'
+++

- autoload 함수에 대해서 (fpath의 역할)
- compinit 의 역할
- compdef 의 역할

`pnpm`은 CLI 도구용 자동완성 스크립트를 제공한다. 다음과 같이 `pnpm completion` 뒤에 자동완성을 설정할 shell 이름을 적으면, 자동완성을 설정하기 위한 스크립트를 출력한다.

```sh
$ pnpm completion zsh
```

그런데 이 명령은 스크립트를 출력할 뿐이라, 이 스크립트를 어디에 어떻게 저장해야 하는지를 알아야 한다. [pnpm 자동완성 관련 문서](https://pnpm.io/completion)에 Bash와 Fish에서는 설정하는 방법이 명시되어 있는데, Zsh에서 설정하는 방법이 나와 있지 않았다.

이를 설정하기 위해 관련 내용들을 찾아보았다.

# compdef


Zsh에서 자동완성을 활성화시키기 위해서는 `compinit` 함수를 실행해야 한다.
```sh
# Completion
# https://scriptingosx.com/2019/07/moving-to-zsh-part-5-completions/
autoload -Uz compinit && compinit
```

이 함수는


# pnpm completion zsh

먼저 `pnpm completion zsh`가 어떤 스크립트를 출력하는지 보자.

```zsh
$ pnpm completion zsh
```

```zsh
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

생각보다 길지 않다. 예외 처리하는 부분을 제외하고 핵심을 보면, `_pnpm_completion`이라는 함수를 정의하고, 다음을 호출하는 것이다.

```zsh
compdef _pnpm_completion pnpm
```

# zsh autoload 함수

zsh에서 함수는 lazy load 된다. 즉, zsh shell이 시작될 때 바로 로드되는 게 아니라, 처음 호출될 때 로드가 되는 것이다.
