---
title: "🛠️ Pi-Server만들기 ssh연결편: cloudflared기반 SSH열기"
date: 2025-11-07 15:00:00 +0900
categories: [Raspberry, Tutorial]
tags: [raspberrypi, server]
pin: True
---

## 🧠 title: Cloudflare Tunnel + Access로 SSH 열기 (실패 로그 포함 실전 튜토리얼)

## 1️⃣ 내가 이 작업을 시작한 이유

집에 있는 서버(라즈베리파이)에 ttyd로 접속 하는 방식에 한계를 느꼈다...
군 부대에서 사용하기 위해 ssh은 힘들꺼 같으니(참고로 싸지방 컴은 cmd도 안열린다...) ttyd를 사용 했던 건데
아무튼 버벅인다(엄청;;) 그래서 그냥 휴대폰 기반으로 하려 한다. Termux에서 작업 하는게 훨씬 스트레스를 덜 받을꺼 같아 미뤄왔던 ssh연결 추가를 이번 휴가에 하려한다.
하지만 또 여전히 포트포워딩으로 22번 포트를 열어두는 방식이나 공인 IP를 그대로 노출하는 구조는 쓰고 싶지 않았다.
거기에 이미 이전 작업에서 했던 cloudflared 터널도 이미 있겠다. 그냥 기존에 있는 cludflared tunnel을 그대로 사용 하기러 했다.

그래서 Cloudflare Tunnel과 Access를 사용해서,

* 서버는 내부망 그대로 두고
* 외부에서는 인증된 나만 접속 가능하게
* PC, 노트북, Termux 환경 어디서든

아래 한 줄로 접속되는 상태를 만드는 것이 목표였다.

```bash
ssh m0s
````

---

## 2️⃣ 작업한거 정리한 튜토리얼

### ✅ 1. 서버에서 SSH 서비스가 정상 동작하는 상태를 먼저 만든다

Cloudflare Tunnel이나 Access는 **기존 서비스를 “대체”하지 않는다**.
어디까지나 *이미 정상 동작 중인 서비스 위에 얹는 프록시 레이어*다.
그래서 가장 먼저 해야 할 일은, **서버 자체에서 SSH가 완전히 정상인지 확인하는 것**이다.

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
systemctl status ssh
```

이 상태에서 **같은 서버 내부에서 직접 SSH 접속 테스트**를 진행했다.

```bash
ssh m0squito@localhost
```

이 단계가 중요한 이유는,
이후 문제가 생겼을 때 **“SSH 자체 문제”와 “Tunnel/Access 문제”를 분리해서 판단**할 수 있기 때문이다.
여기서 이미 실패하면 이후 설정은 전부 의미가 없다.

---

### ⚙️ 2. systemd에서 cloudflared가 config.yml을 읽도록 설정한다

기존에 사용 중이던 cloudflared Tunnel은 웹 서비스용으로만 구성돼 있었고,
SSH ingress를 추가하려면 **config 파일 기반 실행이 필수**였다.

cloudflared는 실행 방식에 따라 동작이 완전히 달라지기 때문에,
systemd 서비스에서 **명시적으로 `--config` 옵션을 지정**했다.

```bash
sudo nano /etc/systemd/system/cloudflared.service
```

```ini
[Service]
ExecStart=/usr/local/bin/cloudflared --config /home/m0squito/.cloudflared/config.yml tunnel run
User=m0squito
Restart=always
```

이 설정의 핵심은 두 가지다.

* `--url` 기반 실행을 완전히 배제
* ingress / Access / SSH 설정을 **전부 config.yml 기준으로 통합**

수정 후에는 반드시 데몬을 다시 로드하고 서비스를 재시작한다.

```bash
sudo systemctl daemon-reload
sudo systemctl restart cloudflared
systemctl status cloudflared
```

이 시점부터 cloudflared는 **설정 파일을 “진짜로” 읽는 상태**가 된다.

---

### 📄 3. cloudflared 설정 파일에 SSH ingress를 추가한다

기존에 사용 중이던 `~/.cloudflared/config.yml`에
SSH 접속을 위한 ingress 규칙을 추가했다.

```yaml
tunnel: m0s-tunnel
credentials-file: /home/m0squito/.cloudflared/XXXX.json

ingress:
  - hostname: tunnel.m0squito.dev
    service: http://localhost:3000

  - hostname: ssh.m0squito.dev
    service: ssh://localhost:22

  - service: http_status:404
```

여기서 중요한 포인트는:

* SSH는 **http가 아니라 `ssh://` 스킴**을 사용
* 포트 포워딩 없이 로컬 22번 포트를 그대로 사용
* ingress는 **위에서 아래로 매칭**되므로 순서 유지

이 설정으로
`ssh.m0squito.dev` → cloudflared → 로컬 SSH 데몬
이라는 경로가 완성된다.

---

### 🌐 4. Tunnel과 SSH 도메인을 DNS로 연결한다

Cloudflare DNS 대시보드에서 직접 레코드를 추가하지 않고,
cloudflared 명령을 사용해 **터널 라우팅을 명시적으로 등록**했다.

```bash
cloudflared tunnel route dns m0s-tunnel ssh.m0squito.dev
```

이 명령을 실행하면:

* `ssh.m0squito.dev`에 대한 CNAME 레코드가 자동 생성
* 해당 도메인은 **오직 해당 Tunnel로만 연결**

즉, 공인 IP나 포트는 **끝까지 노출되지 않는다**.

---

### 🔐 5. Cloudflare Access에서 SSH 전용 Application을 만든다

Zero Trust 대시보드에서 **SSH 전용 Access Application**을 생성했다.

설정 요약:

* 유형: 자체 호스팅
* Application type: **SSH**
* 공개 호스트 이름: `ssh.m0squito.dev`
* 정책: Allow + 본인 이메일

Application type을 SSH로 지정하지 않으면,
Access가 트래픽을 HTTP로 오인해 **302 리다이렉트나 인증 루프**가 발생할 수 있다.

이 설정으로 Access는
“이 트래픽은 웹이 아니라 SSH다”
라고 정확히 인식하게 된다.

---

### 🧩 6. 클라이언트 SSH config에 ProxyCommand를 추가한다

매번 cloudflared 명령을 직접 입력하는 대신,
클라이언트 쪽 SSH 설정 파일에 ProxyCommand를 등록했다.

```bash
nano ~/.ssh/config
```

```sshconfig
Host m0s
  HostName ssh.m0squito.dev
  User m0squito
  ProxyCommand cloudflared access ssh --hostname %h
```

이 설정의 효과는:

* SSH 접속 시 cloudflared가 자동으로 Access 인증 처리
* 토큰/브라우저 인증은 Access가 담당
* SSH 키는 기존 방식 그대로 사용 가능

이후에는 아래 명령 하나로 접속이 가능해진다.

```bash
ssh m0s
```

---

## 🧱 3) 작업 하면서 생갔던 문제들

### ❌ cloudflared가 config.yml을 읽지 않음

* 문제 발생: ingress를 추가해도 SSH가 전혀 열리지 않았다.
* 분석: systemd 서비스가 CLI 모드로 실행 중이라 설정 파일을 무시하고 있었다.
* 해결: `ExecStart`에 `--config` 옵션을 명시해 config.yml을 강제로 사용하도록 수정했다.

---

### ⛔ DNS에서 A 레코드를 추가하려고 시도함

* 문제 발생: Cloudflare DNS에서 A 레코드에 IP를 넣으려 했다.
* 분석: Tunnel 구조에서는 IP 기반 연결이 아니다.
* 해결: `cloudflared tunnel route dns`로 CNAME을 생성했다.

---

### ⛔ 일반 SSH로 바로 접속 시도

* 문제 발생: `ssh user@ssh.domain`으로 접속 시 무한 로딩 또는 UNKNOWN port 오류가 발생했다.
* 분석: Access 인증을 통과하지 않은 트래픽이었다.
* 해결: Access를 거치지 않은 SSH는 정상 동작하지 않는 구조임을 확인했다.

---

### ⛔ Access 앱을 SSH 타입으로 만들지 않음

* 문제 발생: Access Application을 만들었는데 접속이 되지 않았다.
* 분석: Application type이 SSH가 아니었다.
* 해결: SSH 타입으로 다시 생성하고 공개 호스트 이름을 정확히 지정했다.

---

### ⛔ `cloudflared access ssh`만 실행하고 끝냄

* 문제 발생: 이메일 인증은 되는데 셸이 열리지 않았다.
* 분석: 해당 명령은 인증 세션만 생성한다.
* 해결: SSH config에서 ProxyCommand로 연결했다.

---

## 📝 4) 작업 후기 및 다음 작업 예정
이번 작업은 원래 전역 한 이후에 내 기기들 사이의 자동화를 위해 하려 했던건데... 뭐 지금 해 버렸으니 그냥 지금부터 자동화 해 보려 한다.
최근에 또 `Tasker`라는 앱도 찾아서 Termux랑 연계해 다양한 걸 실험해 볼 생각이다.

