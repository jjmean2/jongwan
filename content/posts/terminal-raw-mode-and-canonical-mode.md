+++
date = 2026-03-31T21:42:09+09:00
draft = true
title = 'Terminal Raw Mode and Canonical Mode'
+++

터미널의 두 가지 모드 Raw mode와 Canonical mode

upnote://x-callback-url/openNote?noteId=019d43df-f43f-777a-8a34-7c6c231651d7

Canonical mode에서는 엔터를 누를 때까지 터미널 내의 버퍼에 입력을 쌓아두고, 백스페이스 같은 특수키 처리도 터미널이 직접 한다. 프로그램(프로세스)은 엔터 키를 누른 후에 그 입력을 터미널로부터 받는다.

`cat`에서 엔터키를 누르기 전까지는 `cat`으로 인한 출력이 나오지 않는 것을 생각해보자.

그러나 ZSH나 VIM처럼 입력에 대한 처리를 프로그램이 바로바로 해야 하는 Interactive 프로그램의 경우, 터미널을 Raw mode로 바꿔서 모든 입력을 날 것 그대로 받아야 한다. 이 때는 터미널이 아무 처리도 안 해주므로, 백스페이스를 비롯해서 터미널이 자동으로 하던 모든 처리도 프로그램이 직접 해야 한다.

bash는 GNU readline 이라는 라이브러리를 사용하고, Zsh는 ZLE(Zsh Line Editor)라는 자체 엔진을 사용한다.
