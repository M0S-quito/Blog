---
title: "라즈베리파이에서 Docker 환경 구축기: 삽질 포함 풀로그"
date: 2025-11-09
categories: [Raspberry, Tutorial]
tags: [raspberrypi, docker, server, devops]
pin: True
---

## 🧭 시작 – 왜 Docker였나

라즈베리파이를 24시간 서버로 띄워두고 나서,  
환경이 점점 더러워지고 있었음.  
파이썬 테스트하다가 패키지 꼬이고,  
npm 깔았다 지웠다 하다 의존성 난장판 되고,  
결국 “이걸 매번 포맷 안 하고 격리할 방법 없나?” 싶어서  
**Docker**를 보기 시작함.

처음엔 “가벼운 VM 같은 건가?” 했는데,  
알고 보니 VM이라기보단 **프로세스 단위 격리 환경**이더라.  
OS 위에 또 OS를 얹는 게 아니라,  
커널은 공유하고 각 작업(task)만 따로 묶는 느낌.  
한마디로 “작업 하나당 작은 공간”이 생기는 셈.

---

## ⚙️ 설치 및 기본 확인

먼저 설치:
```bash
sudo apt update
sudo apt install docker.io
```

정상적으로 설치되면 `hello-world`로 확인:
```bash
sudo docker run --rm hello-world
```

출력:
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

이걸 보고 “오 됐네” 했다가,  
바로 다음 생각 — “sudo 안 치고도 쓸 수 없나?”

---

## 👾 권한 문제와 rootless 고민

Docker는 기본적으로 `root` 권한이 필요하다.  
근데 나는 서버를 장시간 켜두고 web으로도 접속할 거라,  
**rootless 모드**가 마음에 걸렸다.  

> “내가 root로 띄운 container가 만약 웹에 노출되면…  
> 그건 곧 파이 전체가 뚫린다는 거잖아?”

그래서 rootless를 켜볼까 했는데,  
Pi OS에서는 아직 완벽히 안정화되지 않은 버전이더라.  
일단 기본 root모드로만 돌려보기로 함.

---

## 🐋 컨테이너 첫 진입

테스트용으로 파이썬 컨테이너 실행:
```bash
sudo docker run -it --name python-test python bash
```

처음엔 명령어 순서 헷갈려서 이런 짓도 함:
```bash
sudo docker run it --name curl-web python bash
```
결과:
```
Unable to find image 'it:latest' locally
docker: Error response from daemon: pull access denied for it...
```

이때 깨달음:  
명령어 순서 하나 틀려도 바로 이미지가 잘못 생성됨.  
**Dangling 컨테이너**가 이런 식으로 생기는 거더라.

---

## 🧱 Dangling 이미지와 로그 관리

Docker는 실행할 때마다 이미지·컨테이너·로그 파일을 만든다.  
문제는 이게 안 지워지면 Pi 저장공간이 순식간에 녹는다.  

청소 명령어:
```bash
sudo docker image prune -f
sudo docker container prune -f
sudo docker volume prune -f
sudo docker system prune -a -f
```

로그 드라이버 확인:
```bash
sudo docker info | grep "Logging Driver"
```
결과:
```
Logging Driver: none
```

그래서 지금은 로그는 거의 안 남기도록 설정함.  
(어차피 Pi에서 dev용으로만 돌릴 거니까.)

---

## 🧩 컨테이너 종료와 상태 확인

실행 중 컨테이너 나가려면:
```bash
exit
```
그럼 자동으로 꺼진다.  
백그라운드로 계속 돌리고 싶으면:
```bash
docker run -d <image>
```

그리고 실행중인 컨테이너 목록 보기:
```bash
docker ps -a
```

---

## 🧠 오늘의 결론

라즈베리파이에서 Docker를 쓰면 확실히 편해진다.  
패키지 꼬임 걱정 줄고, 환경 초기화도 한방.  
근데 이게 리소스를 완전히 가볍게 쓰는 건 아니다 —  
이미지, 로그, dangling 잔재 관리 안 하면  
SD카드 순삭된다.  

오늘 느낀 점 한 줄 요약:  
> “Docker는 VM보다 가볍지만, 관리 안 하면 더럽다.”  

---

## 🔮 다음 목표

- rootless Docker 안정화 테스트  
- Python 크롤러 컨테이너화  
- ttyd + Docker 환경 연동  

> 다음 글: “Docker로 Python 크롤러 만들기 (마크다운 자동 수집봇)”
