+++
date = 2026-07-28T06:43:27+09:00
draft = true
title = 'Bash/Zsh의 프롬프트 변수(PS1, PROMPT) 커스터마이징하기'
summary = ''
tags = ['bash', 'zsh', 'prompt', 'shell']
categories = ['shell']
+++

# Bash & Zsh Prompt 원리 구성 및 트러블슈팅

## 1. Shell Prompt의 동작 방식과 PS1 확장 단계

Bash는 명령어를 입력받기 전 `PS1` 환경변수를 해석 및 평가하여 프롬프트 문자열을 생성함.

### PS1 평가 및 렌더링 파이프라인

```
[ 원본 PS1 문자열 ]
       │
       ▼
1단계: 백슬래시 이스케이프 해독 (Decoded)
       - \e, \u, \w, \[, \] 등 특수 이스케이프 문자를 1차 해독
       │
       ▼
2단계: promptvars 옵션 확인 및 일반 쉘 확장 (Expanded)
       - 매개변수 확장 (Parameter Expansion: $VAR)
       - 명령 치환 (Command Substitution: $(cmd))
       - 산술 연산 확장 (Arithmetic Expansion: $((1+1)))
       - 따옴표 제거 (Quote Removal)
       │
       ▼
[ 터미널로 전달 및 화면 렌더링 ]

```

### 주요 이스케이프 포맷 지정자

* `\u`: 로그인 사용자 이름
* `\h` / `\H`: 호스트 이름 (단축 / FQDN)
* `\w` / `\W`: 현재 작업 디렉터리 경로 (전체 / Basename)
* `\$`: root 사용자면 `#`, 일반 사용자면 `$`
* `\e`: ASCII Escape 문자 (`033` / `0x1B`)
* `\[`, `\]`: **화면에 출력되지 않는 문자열(Non-printing character) 시퀀스의 시작과 끝**

---

## 2. 터미널 제어와 ANSI Escape Sequence

터미널 화면의 서식(색상, 굵기, 커서 이동)을 제어하기 위해 ANSI Escape Sequence 표준 규격을 사용함.

### ANSI Escape Sequence 구조 (CSI)

대부분의 제어 명령은 CSI(Control Sequence Introducer, `ESC [`) 형태로 시작함.

$$\text{CSI} = \text{ESC} + \text{[}$$

```
\e[    31;4;1    m
└┬┘    └─┬──┘    │
CSI    매개변수  명령어 문자

```

* **`ESC`**: `\033`, `\e`, `\x1b` (ASCII Code 27)
* **매개변수**: 세미콜론(`;`)으로 구분된 숫자 조합 (예: `31` = Red, `1` = Bold, `4` = Underline, `0` = Reset)
* **명령어 문자**: `m` (스타일/색상 변경: SGR), `H` (커서 이동), `J` (화면 지우기) 등

---

## 3. 트러블슈팅: Zero-Width Sequence와 줄바꿈/커서 깨짐 현상

### 현상

프롬프트에 색상 코드(ANSI Escape Sequence)를 적용한 뒤 long command를 입력하거나 커서를 이동(`Ctrl+A`, `Ctrl+E`, Backspace 등)할 때 커서 위치가 기괴하게 어긋나거나 줄바꿈이 정상적으로 이루어지지 않음.

### 원인

Shell은 터미널 렌더링 주체가 아니므로 ANSI Escape Code 자체가 화면의 문자 너비(Width)를 차지하지 않는다는 사실을 알지 못함. Shell이 프롬프트의 시각적 길이를 오계산하여 발생함.

### 해결책: Non-printing 영역 명시

* **Bash**: `\[` 와 `\]` 로 감쌈
* **Zsh**: `%{` 와 `%}` 로 감쌈 (단, Zsh 전용 `%F{color}`, `%f` 포맷은 내부적으로 처리됨)

```bash
# Correct in Bash
PS1='\[\e[31;1m\]\u@\h \w\[\e[0m\]\$ '

```

---

## 4. 트러블슈팅: 외부 함수/변수를 통한 Escape Sequence 동적 출력 실패

### 현상

`\[` 와 `\]` 를 변수나 외부 함수의 `echo`/`printf` 출력값에 포함하여 `PS1='$(render_prompt)'` 형태로 호출했을 때, 프롬프트에 `\[\]` 문자가 그대로 노출되거나 너비 계산이 실패함.

### 원인 (Bash 내부 파이프라인 분석)

Bash의 평가 순서는 **1) 백슬래시 이스케이프 해독 (`\\[`, `\\]`) $\rightarrow$ 2) 쉘 확장 (`$(cmd)`, `$VAR`)** 순으로 수행됨.
명령 치환이나 변수 확장을 통해 **나중에 들어온 `\[` 문자는 1단계 이스케이프 해독 과정을 이미 지나쳤으므로 프롬프트 제어 문자로 인식되지 않고 단순 문자열로 처리됨.**

### 해결 가이드라인

#### 방법 A: 이스케이프 문자는 `PS1`에 직접 명시 (권장)

변수/함수는 리터럴 `ESC` 문자만 반환하고, `\[` 와 `\]` 는 `PS1` 문자열에 배치함.

```bash
# ANSI C-quoting ($'...` )을 사용하여 변수에 리터럴 ESC 저장
red=$'\e[31;1m'
reset=$'\e[0m'

# PS1 내부에서 \[ \] 처리
PS1='\[${red}\]\u@\h \w\[${reset}\] $ '

```

#### 방법 B: Bash 내부 1-byte 제어 문자 직접 송출 (Advanced/Hackish)

Bash 내부적으로 프롬프트를 평가할 때 `\[` 를 바이너리 `0x01` (SOH), `\]` 를 바이너리 `0x02` (STX)로 변환하여 처리함. 외부 바이너리/함수에서 직접 `\1`과 `\2` 바이트를 출력하면 동적 너비 제어가 가능함.

```bash
render_prompt() {
    # \1 = 0x01 (\[ 대체), \2 = 0x02 (\] 대체)
    printf '\1\033[0;35m\2$ \1\033[00m\2'
}

PS1='$(render_prompt)'

```

---

## 5. Shell Hook 및 변수 메커니즘 (Bash vs Zsh)

### 프롬프트 렌더링 직전 Hook

* **Bash (`PROMPT_COMMAND`)**: 프롬프트를 출력하기 직전에 사용자 명령줄처럼 실행되는 스크립트/함수 배열.
```bash
# 기존 값 유지를 위해 체이닝 방식 권장
PROMPT_COMMAND="${PROMPT_COMMAND:+$PROMPT_COMMAND;}my_custom_prompt_func"

```


* **Zsh (`precmd` / `add-zsh-hook`)**: 프롬프트 표현 직전 실행되는 Hook 함수.
```zsh
autoload -Uz add-zsh-hook
add-zsh-hook precmd my_custom_prompt_func

```



### Zsh 전용 격리 공간: `psvar`와 `%v`

* **목적**: 프롬프트 생성 시 전역 변수 오염을 방지하고, 변수 확장 오버헤드를 줄이기 위한 Zsh 내부 전용 배열 변수.
* **특징**: `psvar[1]="val"` 지정 후 `PROMPT`에서 `%1v`로 호출. **`psvar` 내부 데이터는 프롬프트 이스케이프 치환(Prompt Expansion)을 재거치지 않는 Raw Data로 출력**되므로, 외부 입력값(Git commit 메시지, 파일명 등)에 `%` 기호가 들어있어 프롬프트가 깨지는 현상을 방지함.

---

## 6. 고성능 Git Prompt 구축 스크립트 (최종 구현)

프롬프트 내 서브쉘 fork 비용을 최소화하고, Git Lock(`.git/index.lock`) 경합을 방지하는 `git --no-optional-locks` 기반의 파싱 알고리즘.

### Zsh 구현 (`precmd` Hook 사용)

```zsh
_ps_set_git_status() {
    _ps_branch=""
    _ps_staged_icon=""
    _ps_unstaged_icon=""

    local branch
    # Subshell fork 수 절감을 위한 Process Substitution + read 활용
    read -r branch < <(git --no-optional-locks branch --show-current 2>/dev/null)
    [[ -z "$branch" ]] && return

    _ps_branch="$branch"

    local has_staged=0
    local has_unstaged=0
    local line index_status work_status

    # git status --porcelain 파싱 (Subshell 없이 Main Shell 루프 처리)
    while IFS= read -r line; do
        [[ -z "$line" ]] && continue
        index_status="${line:0:1}" # Staged
        work_status="${line:1:1}"  # Unstaged / Untracked

        if [[ "$work_status" != " " || "$index_status" == "?" ]]; then
            has_unstaged=1
            break # Unstaged 존재 시 빠른 탈출
        fi

        if [[ "$index_status" != " " ]]; then
            has_staged=1
        fi
    done < <(git --no-optional-locks status --porcelain=v1 2>/dev/null)

    if [[ $has_unstaged -eq 1 ]]; then
        _ps_unstaged_icon="✗"
    elif [[ $has_staged -eq 1 ]]; then
        _ps_staged_icon="✓"
    fi
}

autoload -Uz add-zsh-hook
add-zsh-hook precmd _ps_set_git_status

# %(6~|%-1~/…/%3~|%~) -> 디렉토리 깊이가 6 이상일 경우 중간 경로 축약
PROMPT='%F{green}%n %(?:%f➜:%B%F{red}➜) %F{blue}%B%(6~|%-1~/…/%3~|%~)%b%f${_ps_branch:+ %B%F{cyan}(%F{red}$_ps_branch%f${_ps_unstaged_icon:+ %F{yellow}$_ps_unstaged_icon}${_ps_staged_icon:+ %F{green}$_ps_staged_icon}%F{cyan})%f%b} $ '

```

### Bash 구현 (`PROMPT_COMMAND` 사용)

```bash
_ps_set_git_status() {
    _ps_branch=""
    _ps_staged_icon=""
    _ps_unstaged_icon=""

    local branch
    read -r branch < <(git --no-optional-locks branch --show-current 2>/dev/null)
    [[ -z "$branch" ]] && return

    _ps_branch="$branch"

    local has_staged=0
    local has_unstaged=0
    local line index_status work_status

    while IFS= read -r line; do
        [[ -z "$line" ]] && continue
        index_status="${line:0:1}"
        work_status="${line:1:1}"

        if [[ "$work_status" != " " || "$index_status" == "?" ]]; then
            has_unstaged=1
            break
        fi

        if [[ "$index_status" != " " ]]; then
            has_staged=1
        fi
    done < <(git --no-optional-locks status --porcelain=v1 2>/dev/null)

    if [[ $has_unstaged -eq 1 ]]; then
        _ps_unstaged_icon="✗"
    elif [[ $has_staged -eq 1 ]]; then
        _ps_staged_icon="✓"
    fi
}

PROMPT_COMMAND="${PROMPT_COMMAND:+$PROMPT_COMMAND$'\n'}_ps_set_git_status"
PROMPT_DIRTRIM=4

PS1='\[\e[32m\]\u\[\e[0m\] $(XIT=$?; [[ $XIT != 0 ]] && printf "%s" "\[\e[31;1m\]"; printf "➜") \[\e[34;1m\]\w\[\e[0m\]${_ps_branch:+ \[\e[36;1m\](\[\e[31m\]$_ps_branch${_ps_unstaged_icon:+ \[\e[33m\]$_ps_unstaged_icon}${_ps_staged_icon:+ \[\e[32m\]$_ps_staged_icon}\[\e[36m\])\[\e[0m\]} $ '

```
