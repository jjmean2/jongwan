+++
date = "2026-03-24T12:59:05+09:00"
draft = true
title = 'Git Log Usage Example'
+++

git log 에서 특정 path 의 변화를 제외하고 찾기 등, 여러 가지 사용 예제들을 나열하기

upnote://x-callback-url/openNote?noteId=6df81610-09f5-4a51-850b-8d86b9354e96
upnote://x-callback-url/openNote?noteId=0e0073a7-245d-4210-876e-920b5cd7fd00

```sh
$ git log -- ':(exclude)ios'
$ git log -- ':!android'
$ git log -- ':!ios' ':!android'
$ git log -- :^ios :^android
```

upnote://x-callback-url/openNote?noteId=8ec5acd5-6655-45f5-b8a6-5bf6baad036f

파일명 변경 히스토리
```sh
$ git log --name-only --follow --all -- <filename>
```
