+++
date = '2026-02-12T22:06:19+09:00'
draft = true
slug = 'git-submodule'
title = 'git submodule, 저장소 안에 다른 저장소 두기'
+++

upnote://x-callback-url/openNote?noteId=b5f6c82f-26dd-4af1-b717-7ee32525b90b

Hugo로 블로그를 만들기 위해 Hugo의 [Getting Started](https://gohugo.io/getting-started/quick-start/) 문서를 따라하는데 테마를 설치하기 위해 Git 서브모듈(Submodule)을 사용하고 있었다.

> Clone the [Ananke](https://github.com/theNewDynamic/gohugo-theme-ananke) theme into the `themes` directory, adding it to your project as a [Git submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules).
>
> ```sh
> git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
> ```

Git 서브모듈은 저장소 안에 다른 저장소를 두고 관리하는 기능이다. 프로젝트에서 여러 모듈을 사용하고, 각 모듈마다 독립된 저장소가 있을 경우에 유용해 보이는 기능이다. 이 기능을 처음 접했을 때, 내가 작업하는 환경에서 적용해보고 싶었는데, 실제로 프로젝트 내에서 개발하는 여러 모듈들을 굳이 독립된 저장소로 분리할 일이 별로 없어서 써보지는 못했다. 서브모듈 저장소를 수정하면, 서브모듈과 상위 저장소를 둘다 커밋해야 해서 독립된 저장소가 꼭 필요한 경우가 아니면, 그냥 모노리포로 두는 것이 더 편하게 느껴졌다. 그리고 다른 사람이 개발하는 외부 모듈이라면, npm 등 보통 개발환경에서 제공하는 패키지 관리 시스템이 있으므로 굳이 Git 서브모듈을 쓸 필요가 없었다.

Hugo의 경우, 테마를 사용하려면 `<Hugo_프로젝트_루트>/themes/{테마_이름}/` 디렉토리에 테마 파일들을 위치시켜야 하는데, 다른 사람이 만든 테마를 다운로드 받기 위해 Git을 이용하면, 이후 버전 업데이트도 쉬워지기 때문에 Git 서브모듈을 쓰는 방법을 소개하는 것 같다.
```sh

```







https://themes.gohugo.io/

---

서브모듈 설정 없이, 상위 저장소와 하위 저장소가 있고, 하위 저장소에서 새로운 커밋을 한 뒤에 상위 저장소에서 git add 를 하는 경우, 다음과 같은 메시지가 나온다.
```sh
$ git add .
warning: 내장 깃 저장소 추가: otherdir
hint: You've added another git repository inside your current repository.
hint: Clones of the outer repository will not contain the contents of
hint: the embedded repository and will not know how to obtain it.
hint: If you meant to add a submodule, use:
hint:
hint: 	git submodule add <url> otherdir
hint:
hint: If you added this path by mistake, you can remove it from the
hint: index with:
hint:
hint: 	git rm --cached otherdir
hint:
hint: See "git help submodule" for more information.
hint: Disable this message with "git config set advice.addEmbeddedRepo false"
```

그리고 상위 디렉토리에서 add 한 내용을 확인하면 다음과 같은데, 이는 서브모듈을 추가했을 때와 동일하다.
```sh
$ git diff --cached
diff --git a/otherdir b/otherdir
new file mode 160000
index 0000000..bda5ffb
--- /dev/null
+++ b/otherdir
@@ -0,0 +1 @@
+Subproject commit bda5ffb2820a9427883a3fc460c54eedf55b3c39
```

```sh
$ git ls-files -s otherdir
160000 bda5ffb2820a9427883a3fc460c54eedf55b3c39 0	otherdir
```

---

##

## 정리할 주제

- git submodule을 새로 추가하는 법
- 다른 저장소에서 추가된 git submodule을 반영하는 법
- git submodule을 제거하는 법

## git submodule 관련 정보들

- .gitmodules 파일: 외부 저장소의 URL과 경로 정보
- .git/config 파일: 로컬 환경의 서브모듈 설정
- .git/modules/ 디렉토리: 서브모듈의 실제 .git 데이터(히스토리)
- 인덱스(Staging Area): 메인 저장소가 기억하는 서브모듈의 커밋 해시

## git submodule deinit

서브모듈의 로컬 설정을 해제한다. 저장소에 저장된 정보(`.gitmodules`)를 건드리거나 제거하는 것은 아니다.
로컬의 워크트리만 수정하는 것이므로 비교적 안전하고 가역적인 명령이다.

- `.git/config` 설정 삭제
- 워킹 디렉토리 파일 삭제
- `.gitmodules` 유지

## git submodule을 제거할 때 git rm을 쓰자.

### 1. 서브모듈 설정 해제 및 삭제 `git rm`

```sh
git rm -f path/to/submodule
```

submodule 디렉토리에는 특수한 파일모드가 설정되어 있다. gitlink?
따라서 그냥 `rm`으로 제거하면, 이 파일모드가 끊어지지 않아서 서브모듈이 있어야 하는데 없다는 메시지를 뱉는다.
`git rm`은 이것까지 끊어준다.

`git rm`은 파일 삭제 + Staging Area에 반영하는 명령이다.

#### 참고로

`git submodule deinit`은 서브모듈 내의 파일들은 제거해주지만, 서브모듈 디렉토리 자체는 그대로 놔둔다.

`rm`으로 디렉토리를 제거해도 git은 서브모듈이 있다고 인지한다. `.gitmodules`의 내용을 직접 지워도 마찬가지였다. `git rm`으로 제거하면, 서브모듈 디렉토리와 파일, `.gitmodules`도 정리해주고, Staging 되어 있던 서브모듈이 있다는 링크도 제거해주는 것 같다.

> 따라서 `git rm`을 하지 않고 그냥 윈도우 탐색기나 `rm -rf`로 폴더만 지우면 어떻게 될까요?
>
> 1. 폴더(파일)는 사라집니다.
> 2. 하지만 Git의 인덱스(명단)에는 여전히 **"여기에 160000 모드로 연결된 커밋 정보가 있어야 해"**라는 기록이 남아있습니다.
> 3. 결국 Git은 "파일은 없는데 명단엔 있네? 이거 오류 아니야?"라며 계속 경고를 띄우거나, 나중에 다시 서브모듈을 설치할 때 꼬이게 만듭니다.

```
$ git ls-files -s
100644 97e006367bacf2a511c92b186a3d14b6cd59d332 0	layouts/render-blockquote.html
100644 8dd651f914d3939512be19294c12f061f29cdbc5 0	static/images/software-update.png
100644 32d6940aa2a84a5fd4ed8f82449819c3884cb315 0	static/images/xcode-path.png
160000 10d3dcc0e05cee0aaca58a1305a9d824b2cf9a2a 0	themes/PaperMod
100644 25b67521d3eacd6f5b65eaa1ae7f371a0c34ae93 0	themes/QuietWanderer/archetypes/default.md
100644 166ade9245232d7735f4f92a1a5d78bfce45ddd2 0	themes/QuietWanderer/assets/css/main.css
100644 e2aac5275102bfdb6e752c4b94b6ee629fcdf30a 0	themes/QuietWanderer/assets/js/main.js
```

### git rm 후 .git 디렉토리에 남은 흔적 정리하기

`.git/modules` 디렉토리는 서브 모듈의 .git 디렉토리 같은 것이다. 서브 모듈의 히스토리(커밋/브랜치 등)가 저장되어 있는데, 이건 `git rm`으로 제거되지 않으므로 직접 수동으로 지워야 한다.

| **단계**   | **명령어**                   | **역할**                        |
| ---------- | ---------------------------- | ------------------------------- |
| **Step 1** | `git rm -f [경로]`           | 파일 삭제 및 `.gitmodules` 반영 |
| **Step 2** | `rm -rf .git/modules/[경로]` | 내부 히스토리 캐시 완전 삭제    |
| **Step 3** | `git commit`                 | 제거 상태를 저장소에 영구 반영  |

```sh
# 1. 인덱스에서 제거 (설정 파일과 워킹 트리에서 자동 삭제 시도)
git rm -f [서브모듈_경로]

# 2. 남은 내부 캐시 삭제 (데이터 유실 방지를 위해 Git이 자동으로 안 지워줌)
rm -rf .git/modules/[서브모듈_경로]
```

> ## 3\. 가장 깔끔한 제거 시나리오
>
> 만약 서브모듈을 **완전히(영구히)** 제거하고 싶다면, 보통 아래 순서가 가장 정석입니다.
>
> 1. **`git submodule deinit [path]`**: 내 로컬 설정을 먼저 깨끗하게 비웁니다.
> 2. **`git rm [path]`**: 프로젝트 공통 설정과 인덱스 포인터를 지웁니다.
> 3. **`rm -rf .git/modules/[path]`**: 혹시 모를 내부 캐시까지 지워 용량을 확보합니다.

> 혹시 지금 `git rm`을 먼저 해버려서 `.git/config`에 정보가 남아있는 상황인가요? 그렇다면 `git config --local --remove-section submodule.[이름]` 명령어로 수동으로 지울 수 있습니다. 필요하시면 구체적인 방법을 알려드릴게요!
