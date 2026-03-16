+++
date = '2026-03-16T09:08:49+09:00'
draft = true
title = 'manual page 를 markdown으로 변환하기'
+++

```sh
$ pandoc -f man -t gfm $(man -w xcrun)
```
