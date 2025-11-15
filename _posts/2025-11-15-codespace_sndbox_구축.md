---
title: Codespace 개발 환경 완전 구축 가이드
author: M0S-quito
date: 2025-11-15 20:00:00 +0900
categories: [Tutorial, GitHub Codespace]
tags: [Codespace, GitHub, PAT, Dev Environment]
pin: True
---

# Codespace 개발 환경 완전 구축 가이드  
### 여러 레포를 안정적으로 다루기 위한 작업 환경 구성기

---

## 1. 왜 이 작업을 시작하게 되었는가

군 복무 중이라 로컬 개발 환경을 구축할 수 없는 상황에서,  
구름 IDE → 여러 클라우드 환경을 사용하다가 GitHub Codespace를 발견했다.  
Codespace는 확실히 자유도가 높고 완성도도 높았지만,  
여러 레포를 동시에 다뤄야 하는 프로젝트를 진행할 때 문제가 생겼다.

Codespace는 기본적으로 제한된 권한의 `GITHUB_TOKEN`을 제공하는데,  
이 토큰은 현재 보고 있는 리포 정도만 접근 가능하다.  
그래서 다음과 같은 문제들이 발생했다:

- 다른 레포에 push/pull 시도 시 권한 부족  
- 여러 레포 동시 작업 시 인증 충돌  
- Codespace 재생성 시 인증 정보 초기화  
- 커밋 서명(GPG/SSH) 설정이 자동으로 달라짐  

Termux에서는 Classic 토큰을 기반으로 여러 레포를 자유롭게 다뤘던 경험이 있어서  
“Codespace에서도 로컬처럼 안정적인 환경을 만들 수 없을까?”  
라는 고민에서 이번 작업을 시작하게 되었다.

---

## 2. 과정에서 새롭게 알게 된 것들

### 2-1. GitHub 토큰 방식의 차이
GitHub에는 두 가지 주요 토큰 방식이 있다.

- **Fine-grained token**  
  - 보안성이 높고 필요한 리포만 권한을 부여할 수 있다  
  - 하지만 레포가 많아질수록 설정을 반복해야 하는 번거로움이 있다  

- **Classic token**  
  - 한 번 발급하면 범용적으로 대부분의 작업을 처리할 수 있다  
  - 여러 레포를 동시에 다루는 개발 방식에 특히 적합하다  

여러 레포를 동시에 사용하는 내 개발 스타일에서는  
Classic 토큰이 더 실용적이라는 결론에 이르렀다.

### 2-2. GitHub Secrets의 존재
GitHub 내부에는 리포별로 사용할 수 있는 **Secrets**라는 기능이 있다.  
이를 통해 환경 변수를 안전하게 Codespace 내부로 전달할 수 있다는 점을 새롭게 알게 되었고,  
이를 기반으로 보안성과 편의성을 동시에 확보할 수 있었다.

---

## 3. Codespace 기본 설정의 구조적 한계

Codespace는 다음과 같은 흐름으로 초기화된다:

```
Codespace 부팅  
→ GITHUB_TOKEN 자동 발급  
→ 제한된 리포만 접근 가능  
→ 다중 레포 작업 시 권한 문제 발생  
→ 커밋 서명 및 인증 설정이 꼬일 수 있음  
→ Codespace 재생성 시 전체 초기화  
```

단일 리포를 가볍게 수정하는 목적에는 적합하지만  
여러 레포를 활용하는 프로젝트에는 한계가 있다.

---

## 4. 해결 과정: Codespace를 완전한 내 개발 머신으로 만들기

---

### 4-1. Classic PAT 발급하기

```
GitHub → Settings → Developer settings → Tokens (Classic)
```

- `repo`, `org`, `codespace` 권한만 활성화  
- 여러 레포를 사용할 경우 Classic 토큰이 가장 안정적이다  

---

### 4-2. Repo Secrets에 MY_PAT 등록하기

```
repo → Settings → Secrets and variables → Codespaces
```

여기서 PAT을 `MY_PAT`라는 이름으로 저장하면  
Codespace 내부에서 `$MY_PAT` 환경 변수로 바로 접근할 수 있다.

---

### 4-3. Codespace에서 기본 토큰 제거 + PAT 로그인

```bash
unset GITHUB_TOKEN
printf '%s' "$MY_PAT" | gh auth login --with-token
gh auth status
```

- 기본 토큰을 제거하고  
- GitHub CLI(gh)를 **내 계정 기반**으로 로그인해  
- Codespace가 로컬 환경처럼 동작하도록 만든다  

---

### 4-4. Git 인증을 GH CLI로 통일

```bash
gh auth setup-git
```

`.gitconfig`에는 다음과 같은 구문이 자동 추가된다:

```ini
[credential]
  helper = !gh auth git-credential
```

이로써 **모든 push/pull 인증이 gh를 통해 일관되게 처리**되어  
더 이상 인증이 꼬이지 않는다.

---

### 4-5. .gitignore로 Codespace 작업 환경 보호

여러 프로젝트 폴더가 루트 디렉토리에 함께 존재할 경우  
Git이 불필요한 파일을 추적할 수 있으므로 미리 제외 설정을 해둔다.

```gitignore
# projects
프로젝트이름/
```

---

# 🛠 Codespace에서 발생한 커밋 서명(Signing) 문제 해결

Codespace에서 커밋을 하려 할 때 다음 오류가 발생할 수 있다:

```
gpg: signing failed: No secret key
error: gpg failed to sign the data
```

이 문제는 Codespace 환경이 초기화되며 **GPG 비밀키가 존재하지 않는 상태에서  
Git이 GPG 서명을 강제로 시도할 때 발생한다.**

### 원인 요약

- `commit.gpgsign`이 true 상태로 레포에 설정됨  
- Codespace 내부에는 GPG 비밀키가 없음  
- GitHub의 `gh-gpgsign` 래퍼도 기본 토큰 환경에서만 정상 동작  
- 결국 “서명을 해야 하는데 사용할 키가 없다”는 오류가 발생  

---

## 🔧 해결 방법

### 1) 서명을 끄는 방법 (가장 간단)

```bash
git config commit.gpgsign false
```

Codespace 내부에서 서명 문제가 있을 때 가장 즉시 해결되는 방식이다.

---

### 2) GPG 대신 SSH 서명을 사용하도록 변경

Codespace와 GitHub CLI는 SSH 서명을 더 안정적으로 지원한다.

```bash
git config --global gpg.format ssh
gh auth setup-git
```

이렇게 하면  
- GPG 서명은 비활성화되고  
- GitHub가 자동으로 SSH 기반 커밋 서명을 처리한다  

SSH 키는 Codespace에서 자동 생성되므로 추가 설정이 필요 없다.

---

## 6. 전체 구성 요약

```
[기본 Codespace]
→ GITHUB_TOKEN 기반 제한된 권한  
→ 여러 레포 작업 불가  
→ GPG 서명 오류 발생  

[구축한 환경]
→ Classic PAT 기반 로그인  
→ 모든 인증 = gh CLI  
→ GPG 대신 SSH 서명 활성화  
→ Secrets로 안전한 PAT 전달  
→ Codespace 재생성 시에도 한 줄로 복구  

[결과]
Codespace가 완전한 내 개발 환경으로 동작  
여러 레포를 자유롭게 다룰 수 있는 안정적인 작업 환경 완성  
커밋 서명 문제도 해결됨  
```

---

## 마무리

이번 작업은 단순히 토큰 문제를 해결하는 수준을 넘어,  
GitHub Codespace를 **로컬 개발 환경처럼 안정적으로 사용할 수 있게 만드는 과정**이었다.  
여러 레포를 동시에 다루는 프로젝트에서도  
지속적이고 일관된 개발 흐름을 유지할 수 있게 되었다.
