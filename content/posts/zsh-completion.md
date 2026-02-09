+++
date = '2026-02-09T22:42:35+09:00'
draft = true
title = 'Zsh Completion'
+++

`pnpm completion zsh`와 같이 shell completion 방법을 제공하는 도구들이 있다. 보통 이걸 실행시키면 일련의 shell script가 출력된다. 그런데 이 스크립트를 어떻게 하라는 것인지 설명이 없을 때가 있다. `pnpm completion zsh`의 출력을 어떻게 해야 Shell completion이 동작하는지 알아보자.

# autoload 함수

zsh에서 함수는 lazy load 된다. 즉, zsh shell이 시작될 때 바로 로드되는 게 아니라, 처음 호출될 때 로드가 되는 것이다.
