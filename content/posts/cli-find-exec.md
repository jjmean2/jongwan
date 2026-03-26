+++
date = "2026-03-26T22:37:15+09:00"
draft = true
title = 'Cli Find Exec'
+++

upnote://x-callback-url/openNote?noteId=db1156d4-b752-472c-8ea9-c73b0b97bed3

find의 장점
- `-exec`로 외부 스크립트를 필터링 조건에 이용할 수 있음.
- `-and`, `-or` 등의 연산자와 여러 primary를 이용해 다양하고 복잡한 조건을 명령 하나로 시도할 수 있음

fd의 장점
- 병렬 처리를 이용해 속도가 훨씬 빠름

복잡한 조건이 꼭 필요한 경우, `find`를 사용.
일상적이고, 기본적인 패턴 검색이나 디렉토리 파일 전체 탐색은 `fd`를 사용

