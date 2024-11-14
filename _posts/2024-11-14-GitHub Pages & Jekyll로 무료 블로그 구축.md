---
title: GitHub와 Jekyll로 블로그 구축
author: M0S-quito
date: 2024-11-14 14:10:00 +0800
categories: [Blogging, Tutorial]
tags: [writing]
pin: True
---
솔직히 현대 사회를 살아 가며 개인 블로그 하나 정도는 있어야 하지 않을까?<br>
GitHub Pages와 Jekyll을 사용해 간단하게 무료 블로그를 하나 구축해 보자!!!

## Step 0. 준비물
- Windows 64bit
- Git 설치
- VS Code 설치
- Github 가입

---

## Step 1. Github세팅
### 1-1. **리포지토리 생성**

1. **프로필** → **Your repositories** → **new**
2. **Repository name**을 `username.github.io`로 설정
    - 예: 사용자 이름이 `M0S-quito`라면 `M0S-quito.github.io`로 입력
    - 반드시 이 형식이어야 외부에서 접근 가능
3. **Public**으로 설정
4. **Add a README file** 체크
5. **Create repository** 클릭

### 1-2. **Github Pages 설정**

1. 생성한 리포지토리로 이동 후 **Settings** 클릭
2. 왼쪽 메뉴에서 **Pages** 클릭
3. **Source**를 `Deploy from a branch`로 설정
4. `https://username.github.io` 주소로 접속해 사이트 연결 확인

### 1-3. **Vscode 활용하여 로컬 리포지토리 복사**

1. Vscode 열기 → **F1**키 입력 → **git clone** 검색 후 선택
2. 리포지토리 주소 입력
    - 예: `https://github.com/username/username.github.io.git`
3. 로컬에 복사할 위치 선택

### 1-4. **로컬 변경사항 적용**

1. 클론한 리포지토리 열기 (`README.md` 파일 확인)
2. `index.html` 파일 생성

```html
<html>
    <body>
        Hello! This is the first page!
    </body>
</html>
```

3. 좌측 **Source Control** 메뉴 선택
4. `+` 버튼을 클릭해 변경사항 추가
5. 커밋 메시지 입력 후 커밋 & 푸시
6. 사이트 반영 확인 (예: `https://username.github.io`)

---

## Step 2. 로컬 개발환경 구축

로컬과 온라인 리포지토리가 연결된 것을 확인했다면, 이제 **로컬 개발환경**을 설정해보자.
### 2-1. Ruby 설치
1. [Ruby 공식 홈페이지](https://rubyinstaller.org/downloads/)에서 최신 버전 다운로드 및 설치 (Ruby+Devkit x.y.z-1 (x64))
2. 시작 메뉴에서 **Start Command Prompt with Ruby** 실행
3. **cd** 명령어로 리포지토리 위치로 이동

```shell
cd 리포지토리_파일경로
```

4. **jekyll**, **bundler**, **webrick** 설치

```shell
gem install jekyll bundler gem install webrick
```

5. 설치 확인

```
ruby -v 
jekyll -v 
bundler -v
```

### 2-2. jekyll 서버 구축

1. **jekyll 생성**

```shell
jekyll new ./ 
# 또는
jekyll new ./ --force
```    

2. **bundle install**

```shell
bundle install
```

3. **jekyll 서버 실행**

```shell
bundle exec jekyll serve
```

4. `http://127.0.0.1:4000/` 또는 `http://localhost:4000/` 접속 확인
5. 모든 변경사항 커밋 및 푸시

---

## Step 3. Jekyll 테마 적용

테마 중 **chirpy**를 기준으로 설명한다.

### 3-1. `chirpy` 테마 적용

1. [chirpy 테마 공식 페이지](https://github.com/cotes2020/jekyll-theme-chirpy)에서 압축 파일 다운로드
2. 압축 해제 후 모든 파일과 폴더를 로컬 리포지토리에 복사
3. **bundle install**

```shell
bundle install
```
4. **jekyll 서버 실행**

```shell
bundle exec jekyll serve
```

5. `http://127.0.0.1:4000/` 또는 `http://localhost:4000/` 접속 확인
6. 모든 변경사항 커밋 및 푸시

### Step 3-2. Github Actions 설정

1. 원격 리포지토리의 **Settings** 클릭
2. **Pages** 클릭
3. **Source**를 `Github Actions`로 설정
4. `Configure` 클릭 후 **Commit changes** 클릭
5. 로컬 리포지토리에서 **pull**

### Step 3-3. Node.js 설치

1. [Node.js 공식 홈페이지](https://nodejs.org/en/)에서 최신 버전 다운로드 및 설치
2. Ruby 프롬프트에서 아래 명령어 실행

```shell
npm install && npm run build
```

- 파일 경로에 한글이 있으면 오류 발생 가능
- JSON 파일 오류 시 아래 코드 삽입:

```json
{ "name": "my-github-blog", "version": "1.0.0", "scripts": { "build": "jekyll build", "serve": "jekyll serve" }}
```  

### Step 3-4. 테마 상세 설정

1. `.gitignore` 파일 하단 주석처리

```shell
# Misc 
# _sass/dist 
# assets/js/dist
```

2. `_config.yml` 파일 수정

```
timezone: Asia/Seoul 
url: "https://username.github.io" 
github: username: username
```

3. 모든 변경사항 커밋 및 푸시
### Step 3-5. Conventional Commits 사용

커밋 메시지는 다음 규칙을 따른다.

```shell
<type>: <subject>
```

**type** 종류:
- feat: 새로운 기능 추가
- fix: 버그 수정
- docs: 문서 수정
- style: 코드 스타일 수정 (동작에 영향 없음)
- refactor: 코드 리팩토링
- test: 테스트 코드 추가
- chore: 기타 작업

---

이제 모든 과정이 끝났으니, 사이트가 정상적으로 반영되는지 확인해봐 (`https://username.github.io`).