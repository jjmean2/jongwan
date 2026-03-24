+++
date = "2026-03-23T18:36:20+09:00"
draft = true
title = 'File Descriptor and Redirection'
+++

upnote://x-callback-url/openNote?noteId=019d15ab-c218-75bf-a70c-357c96c052d4
upnote://x-callback-url/openNote?noteId=019d155a-a56f-77ed-999a-96943baeef29


- file descriptor
- `exec [redirection]` 문법
- zsh redirection의 다양한 문법과 실제로 일어나는 일
- 프로세스 치환의 세 가지 형태 `<()`, `=()`(zsh 전용), `>()`
  - 프로세스 치환은 PIPE를 만들어서 열어놓고 괄호로 감싼 프로세스의 표준입력(`>(command)`)이나 표준출력(`<(command)`)을 파이프와 연결한 후 그 파이프의 file descriptor로 치환해주는 명령이라고 볼 수 있다.
- `lsof -d 0,1,2 -a -p $$`로 FD 정보를 확인하는 법

```sh
$ exec > >(tee -a log.txt)
# 이후에는 모든 명령이 stdout 에도 출력되고, log.txt에도 기록됨
```


