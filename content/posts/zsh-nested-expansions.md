+++
date = 2026-04-10T10:55:13+09:00
draft = true
title = 'Zsh Nested Expansions'
+++

upnote://x-callback-url/openNote?noteId=019d7457-4436-7526-9569-04cb99056660

# 특정 명령으로 찾은 경로의 디렉토리 경로 뽑기 (zsh에서 경로 확장시 붙일 수 있는 modifier)

* * *

<br>
<br>

### 변수를 확장할 때, 변수값이 경로라면 modifier를 붙여서 조작할 수 있다.

`${VAR:t}`, `${VAR:h}`

|     |     |     |
| --- | --- | --- |
| **수정자** | **설명** | **기능상 유사한 명령어** |
| **`:t`** | **Tail**: 경로의 마지막 부분(파일명)만 추출 | `basename` |
| **`:h`** | **Head**: 마지막 부분을 제외한 디렉토리 경로 추출 | `dirname` |
| **`:e`** | **Extension**: 파일의 확장자만 추출 | \-  |
| **`:r`** | **Root**: 확장자를 제외한 나머지 경로 추출 | \-  |
| **`:a`** | **Absolute**: 심볼릭 링크를 유지하며 절대 경로로 변환 | `readlink -f` (일부) |
| **`:A`** | **Absolute**: 심볼릭 링크까지 모두 해제하여 실제 절대 경로 추출 | `realpath` |

예를 들어, `/opt/homebrew/bin/git`를 향한 modifier를 실험해보자.

```bash
$ ls -al $(brew --prefix)/bin/git
lrwxr-xr-x@ 1 ljw  admin  28  3월 18 20:11 /opt/homebrew/bin/git -> ../Cellar/git/2.53.0/bin/git

```

`FILE` 변수에 이를 담고

```bash
$ FILE=/opt/homebrew/bin/git
```

결과는 다음과 같다. 

```bash
# 값을 그대로 확장
$ echo ${FILE}
/opt/homebrew/bin/git

# 파일명만 확장
$ echo ${FILE:t}
git

# 부모 디렉토리만 확장
$ echo ${FILE:h}
/opt/homebrew/bin

# 확장자만 확장
$ echo ${FILE:e}
# 빈 문자열

# 확장자를 제외하고 확장
$ echo ${FILE:r}
/opt/homebrew/bin/git

# 상대경로인 경우, 절대경로로 변환
# 즉, 경로가 /로 시작하지 않은 경우, cwd로부터의 상대 경로로 해석하여 절대경로로 변환
$ echo ${FILE:a}
/opt/homebrew/bin/git

# symbolic 링크를 따라가서 절대경로로 변환
$ echo ${FILE:A}
/opt/homebrew/Cellar/git/2.53.0/bin/git


```

<br>

재미있는 점은 이 변수 확장 구문은 중첩될 수 있고, 안쪽에서 확장된 내용은 바깥쪽 확장에서 익명 변수의 값처럼 취급된다는 것이다.

예를 들어 `${FILE}`은 `FILE` 변수를 확장하는 것인데 `${${FILE}}`은 `FILE` 변수를 확장하여 `${/opt/homebrew/bin/git}` 이런 의미가 되는 게 아니라 `${<익명의 변수>}`로 확장되고, 이 `<익명의 변수>`의 값이 `/opt/homebrew/bin/git`인 것처럼 동작한다. 그래서 `${${FILE}}`은 `${FILE}`은 동일한 결과인 것이다.

그리고, 파라미터 확장 뿐 아니라 커맨드 치환 같은 다른 종류의 확장도 중첩을 할 수 있고, 확장의 결과가 이를 감싼 확장에서 파라미터`name`이 아니라 값으로 해석된다는 점 때문에 파라미터 확장 기능의 modifier, flag나 Array subscript 등의 기능을 쓰기 위해서 중첩을 쓰는 유스 케이스도 가능하다. 몇 가지 예를 보자.

<br>

위에서는 modifier를 쓰기 위해 `FILE`이라는 변수를 정의했지만, 다음과 같이 변수를 미리 정의하지 않고도 파라미터 확장의 modifier를 쓸 수 있다.

```bash
# which git의 결과는 다음과 같다.
$ which git
/opt/homebrew/bin/git

# which git의 결과가 익명 변수 값인 것처럼 동작한다. 즉, ${name:A} 에서 name 변수의 값이 which git의 결과인 것처럼 동작한다.
$ echo ${$(which git):A}
/opt/homebrew/Cellar/git/2.53.0/bin/git


```

내가 직접 입력한 값에 modifier를 적용하고 싶을 때 파라미터 확장의 기본값 문법을 쓸 수 있다. `${name:-value}` 문법은 `name`의 값이 non-null이 아닌 경우, 기본값으로 확장되는 문법인데, `name`자체가 빈문자열이라도(`${:-value}`) 성립하며, 이 경우 무조건 지정한 기본값으로 확장된다.

```bash
${name-word}
${name:-word}

If name is set, or in the second form is non-null, then substitute its value; otherwise substitute word.
In the second form name may be omitted, in which case word is always substituted.

```

이를 이용하면, 내가 직접 입력하는 값을 `${:-value}`의 value에 넣어서 확장한 뒤, 그 확장한 값(`value`)를 대상으로 modifier를 적용할 수 있다.

```bash
# 다음처럼 중간변수를 두지 않고,
$ FILE=/opt/homebrew/bin/git
$ echo ${FILE:h}

# ${:-default_value} 문법을 사용하여 /opt/homebrew/bin/git의 부모 디렉토리를 바로 구할 수 있다.
$ echo ${${:-/opt/homebrew/bin/git}:h}
/opt/homebrew/bin

# 이는 다음 CLI 도구를 사용한 것과 비슷한 결과다.
$ dirname /opt/homebrew/bin/git
/opt/homebrew/bin

```

modifier 들은 대부분 외부 도구로도 동일한 결과를 얻을 수 있는 경우가 많아서 꼭 필요하진 않지만, symlink의 절대 경로를 구하는 데에는 다른 대안보다 훨씬 쉽게 가능했다. 예를 들어서 homebrew가 설치하는 실행 파일은 `/opt/homebrew/bin/`에 모이는데 이건 모두 `/opt/homebrew/Cellar/`의 각 formula 디렉토리에 있는 실행파일에 대한 symlink이다.

```bash
$ ls -al $(brew --prefix)/bin | grep -C3 -e ' git '
lrwxr-xr-x@   1 ljw  admin     37  3월 27 18:42 ginstall -> ../Cellar/coreutils/9.10/bin/ginstall
lrwxr-xr-x@   1 ljw  admin     29  4월  8 10:37 gio -> ../Cellar/glib/2.88.0/bin/gio
lrwxr-xr-x@   1 ljw  admin     42  4월  8 10:37 gio-querymodules -> ../Cellar/glib/2.88.0/bin/gio-querymodules
lrwxr-xr-x@   1 ljw  admin     28  3월 18 20:11 git -> ../Cellar/git/2.53.0/bin/git
lrwxr-xr-x@   1 ljw  admin     38  3월 18 20:11 git-cvsserver -> ../Cellar/git/2.53.0/bin/git-cvsserver
lrwxr-xr-x@   1 ljw  admin     35  4월  8 13:05 git-lfs -> ../Cellar/git-lfs/3.7.1/bin/git-lfs
lrwxr-xr-x@   1 ljw  admin     41  3월 18 20:11 git-receive-pack -> ../Cellar/git/2.53.0/bin/git-receive-pack

```

이 symlink가 가리키는 실제 실행파일의 절대 경로를 구하고 싶은데 macOS의 readlink 로는 한계가 있었다. symlink의 내용을 출력하는데, symlink 자체가 상대경로로 지정되어 있어서 다음처럼 상대 경로로 출력되는 것이다.

```bash
# symlink의 내용을 출력한다.
$ readlink $(which git)
../Cellar/git/2.53.0/bin/git
```

GNU의 `readlink` (macOS에서 coreutils로 설치 가능한 `greadlink`)는 `-f` 옵션이 있어서 이를 쉽게 처리 가능한데, macOS `readlink`에는 `-f`옵션이 없다.

<br>

```bash
# 원래는 macOS readlink 에서 -f 옵션을 지원하지 않아서 다음과 같은 에러가 났었는데, 현재 버전(Tahoe 26.4)에서는 된다! 언제부터 추가되었지?
$ readlink -f $(which git)
readlink: illegal option -- f

# GNU readlink
$ greadlink -f $(which git)
/opt/homebrew/Cellar/git/2.53.0/bin/git

```

그런데 zsh 확장의 `:A`로 이게 가능한 것이다.

```bash
# which git의 결과에 :A modifier 적용
$ echo ${$(which git):A}
/opt/homebrew/Cellar/git/2.53.0/bin/git

```

여기에 더해 `=command` 문법의 Filename extension까지 쓰면, `which git`도 없이 zsh 확장 기능으로만 할 수도 있다. command 이름 앞에 `=`을 붙이면, 그 command가 존재할 때, command의 전체 경로(full pathname)로 확장된다.

```bash
# =git은 git의 전체 경로로 확장된다. which git 과 비슷한 효과
$ echo =git
/opt/homebrew/bin/git

# ${:-=git}은 ${:-value} 문법으로 "=git" 값으로 확장된다. =git은 다시 filename expansion으로 "/opt/homebrew/bin/git"으로 확장되고, 이는 :A modifier에 의해 /opt/homebrew/Cellar/git/2.53.0/bin/git 로 확장된다.
$ echo ${${:-=git}:A}
/opt/homebrew/Cellar/git/2.53.0/bin/git

```

[https://www.unix.com/man-page/osx/1/zshexpn/#:~:text=FILENAME%20EXPANSION,-Each%20word%20is%20checked](https://www.unix.com/man-page/osx/1/zshexpn/#:~:text=FILENAME%20EXPANSION,-Each%20word%20is%20checked)

[https://www.unix.com/man-page/osx/1/zshexpn/#:~:text=%60%3D%27%20expansion,-If%20a%20word%20begins](https://www.unix.com/man-page/osx/1/zshexpn/#:~:text=%60%3D%27%20expansion,-If%20a%20word%20begins)

> `=`  expansion
> If a word begins with an unquoted `=`  and the EQUALS option is set, the remainder of the word is taken as the name of a command.  If a command exists by that name, the word is replaced by the full pathname of the command.
> <br>

<br>

참고로 macOS의`readlink`는 `-f` 옵션을 지원하지 않았었는데, 언젠가부터 지원하고 있었다. 이에 대한 질문 글도 있었다.

<br>

### command line - Which macOS version introduced "readlink -f"? - Ask Different

[https://apple.stackexchange.com/questions/464136/which-macos-version-introduced-readlink-f](https://apple.stackexchange.com/questions/464136/which-macos-version-introduced-readlink-f)
