---
title: GitHub와 Jekyll로 블로그 구축
author: M0S-quito
date: 2024-11-14 14:10:00 +0800
categories: [Tutorial, Github 블로그 만들기]
tags: [writing]
pin: True
---

# GitHub Pages와 Jekyll로 무료 블로그 구축하기 🎉

---

GitHub Pages와 Jekyll을 사용하여 무료 블로그를 만들어보자! 다음 단계를 따라 하면 멋진 블로그를 완성할 수 있다.

---

## Step 0: 준비물
- Windows 64bit
- Git 설치
- VS Code 설치
- GitHub 계정

---

## Step 1: GitHub 세팅

### 1-1. 리포지토리 생성

1. **프로필** → **Your repositories** → **new** 클릭.
2. **Repository name**을 `username.github.io`로 설정 (예: `M0S-quito.github.io`).
    - 이 형식을 따라야 외부에서 접속 가능.
3. **Public**으로 설정.
4. **Add a README file** 체크.
5. **Create repository** 클릭.

### 1-2. GitHub Pages 설정

1. 생성된 리포지토리로 이동 후 **Settings** 클릭.
2. 왼쪽 메뉴에서 **Pages** 선택.
3. **Source**를 `Deploy from a branch`로 설정.
4. `https://username.github.io`로 접속해 사이트 연결 확인.

### 1-3. VS Code를 사용해 로컬 리포지토리 복사

1. VS Code 실행 → **F1** 키 → **git clone** 검색 후 선택.
2. 리포지토리 주소 입력 (예: `https://github.com/username/username.github.io.git`).
3. 로컬에 복사할 디렉토리 선택.

### 1-4. 로컬 변경사항 적용

1. 클론한 리포지토리를 열고 `README.md` 파일 확인.
2. `index.html` 파일 생성:

```html
<html>
    <body>
        Hello! This is the first page!
    </body>
</html>
```

3. VS Code **Source Control** 메뉴에서 변경사항 추가.
4. 커밋 메시지 입력 후 커밋 및 푸시.
5. `https://username.github.io`에서 사이트 확인.

---

## Step 2: 로컬 개발환경 구축

### 2-1. Ruby 설치

1. [Ruby 공식 사이트](https://rubyinstaller.org/downloads/)에서 최신 버전 다운로드 및 설치.
2. **Start Command Prompt with Ruby** 실행.
3. **cd** 명령어로 리포지토리 위치로 이동:

```shell
cd 리포지토리_경로
```

4. **jekyll**, **bundler**, **webrick** 설치:

```shell
gem install jekyll bundler gem install webrick
```

5. 설치 확인:

```shell
ruby -v
jekyll -v
bundler -v
```

### 2-2. Jekyll 서버 구축

1. **Jekyll 사이트 생성**:

```shell
jekyll new ./ --force
```

2. **의존성 설치**:

```shell
bundle install
```

3. **로컬 서버 실행**:

```shell
bundle exec jekyll serve
```

4. `http://127.0.0.1:4000/` 또는 `http://localhost:4000/`에서 접속 확인.
5. 변경사항을 커밋 및 푸시.

---

## Step 3: Jekyll 테마 적용

### 3-1. Chirpy 테마 적용

1. [Chirpy 테마 GitHub 페이지](https://github.com/cotes2020/jekyll-theme-chirpy)에서 테마 다운로드.
2. 다운로드한 파일을 로컬 리포지토리에 복사.
3. **의존성 설치**:

```shell
bundle install
```

4. **로컬 서버 실행**:

```shell
bundle exec jekyll serve
```

5. 로컬에서 접속 확인 후 변경사항 커밋 및 푸시.

---

## Step 4: GitHub Actions 설정

### 4-1. GitHub Actions 활성화

1. 원격 리포지토리의 **Settings** → **Pages** 이동.
2. **Source**를 `GitHub Actions`로 설정.
3. `Configure` 버튼 클릭 후 **Commit changes**.

---

## Step 5: Node.js 설치 및 테마 상세 설정

1. [Node.js 공식 페이지](https://nodejs.org/en/)에서 최신 버전 다운로드 및 설치.
2. Ruby 프롬프트에서 다음 명령어 실행:

```shell
npm install && npm run build
```

3. `_config.yml` 파일 수정:

```yaml
timezone: Asia/Seoul
url: "https://username.github.io"
github:
  username: username
```

4. 변경사항 커밋 및 푸시.

---

> "GitHub Pages와 Jekyll을 통해 무료 블로그를 쉽고 빠르게 만들어보자!" 🚀