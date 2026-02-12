+++
date = '2026-02-09T22:42:35+09:00'
draft = true
title = 'Zsh 자동완성 스크립트 설정하기'
+++

`pnpm completion zsh`와 같이 shell completion 방법을 제공하는 도구들이 있다. 보통 이걸 실행시키면 일련의 shell script가 출력된다. 그런데 이 스크립트를 어떻게 하라는 것인지 설명이 없어서 곤란했던 경험이 있다. `pnpm completion zsh`의 출력을 어떻게 해야 Shell completion이 동작하는지 알아보자.

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
