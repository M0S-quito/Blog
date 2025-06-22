---
title: GoatCounter를 사용한 블로그 트래픽 확인
author: M0S-quito
date: 2024-11-14 14:10:00 +0800
categories: [Tutorial, Github 블로그 만들기]
tags: [Tutorial, github]
pin: True
---
# GoatCounter: 웹 트래픽 분석 도구 설정하기 📊

GoatCounter는 웹사이트의 방문자 수를 확인할 수 있는 **웹 분석 도구**이다. 간단한 설정으로 내 웹사이트에 적용해보자.

---

## Step 1: GoatCounter 사이트에서 계정 생성

1. **[GoatCounter 공식 사이트](https://www.goatcounter.com/)** 접속.
2. 회원가입 창에서 다음 정보를 입력:
   - **Code**: GoatCounter 주소 이름 (예: `M0Squito` → `https://M0Squito.goatcounter.com`).
   - **Site domain**: 내 사이트 도메인 입력 (예: `https://M0S-quito.github.io`).
   - **Email address**: 개인 이메일 주소 작성.
   - **Password**: GoatCounter 로그인에 사용할 비밀번호 설정.
   - **fill~here**: 로봇 인증 숫자 입력.
3. 가입 후 계정이 생성되며 트래픽 분석 시작 가능.

---

## Step 2: GoatCounter 내 웹사이트에 설정

### 2-1. `config.yml` 파일 수정
GoatCounter ID를 추가하여 설정.

```yaml
# Web Analytics Settings
analytics:
  google:
    id: # Google Analytics ID (사용하지 않으면 공란)
  goatcounter:
    id: m0squitoblog # GoatCounter의 코드 입력
  umami:
    id: # Umami ID
    domain: # Umami 도메인
  matomo:
    id: # Matomo ID
    domain: # Matomo 도메인
  cloudflare:
    id: # Cloudflare Web Analytics 토큰
  fathom:
    id: # Fathom Site ID
```

### 2-2. 페이지뷰 제공자 설정
GoatCounter를 페이지뷰 제공자로 지정.

```yaml
# Page views settings
pageviews:
  provider: 'goatcounter' # GoatCounter로 설정
```

---

## Step 3: `chirpy` 테마가 아닌 경우

### 3-1. GoatCounter 스크립트 삽입
1. GoatCounter에서 제공하는 **script 코드**를 복사.
2. `_layout` 또는 `_head` 파일에 추가.
   - 일반적으로 `<head>` 태그에 추가하면 빠르게 정보를 수집할 수 있음.
   - `<body>`에 추가해도 동작하나, 수집 속도 차이가 있을 수 있음.

### 3-2. GitHub Root 외 다른 리포지토리 운영 시
- 여러 웹사이트를 운영 중이라면 각각의 웹사이트에 위 설정을 개별적으로 삽입.

---

> "GoatCounter로 내 웹사이트의 방문자를 효과적으로 분석해보자!" 🚀

