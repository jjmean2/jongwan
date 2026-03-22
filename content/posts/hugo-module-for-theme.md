+++
date = "2026-03-18T22:57:13+09:00"
draft = true
title = 'Hugo Module for Theme'
+++

upnote://x-callback-url/openNote?noteId=019d013c-b96d-718c-bc83-46f6392e6f11

themes와 module 은 모두 hugo 파일 시스템에 레이어를 추가하는 요소다.
사이트 레이어가 최우선이고, 이후에는 레이어를 쌓는 순으로 적용된다.

hugo module은 hugo 사이트를 모듈로 만들고 `hugo mod init <project-name>`, 외부 모듈을 import 하면 사용할 수 있다. toml 에 다음과 같이 적으면, `hugo mod get`, `hugo mod tidy`, `hugo build` 등에서 자동으로 인식하고 모듈이 없으면 다운로드 받는다.

```toml
[module]
[[module.imports]]
path = "github.com/adityatelange/hugo-PaperMod"
```
