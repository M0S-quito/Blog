---
title: "🛠️ Termux에서 tmux로 제어하는 임시 SSH 서버 열기"
date: 2026-01-20 15:00:00 +0900
categories: [Termux, Tutorial]
tags: [Termux, ssh, tmux]
pin: True
---


---

1️⃣ 🧭 내가 이 작업을 시작한 이유

나는 pi-server에 연동과 접근성 그리고 보안을 위해 내 폰이 일종에 중가간에 허브처럼 작동하길 바랬다.
가장 가까이 있으면서 가장 내 기기들과의 소통이 많은 기기가 스마트폰 아라 생각해 이런 생각이 들었다.
그래서 Termux를 사용하면서, 필요할 때만 잠깐 쓸 수 있는 SSH 서버가 필요했다.
항상 켜져 있는 서버가 아니라, 같은 Wi-Fi(LAN) 환경에서만 접속하고 사용이 끝나면 바로 끌 수 있는 구조를 원했다.


---

2️⃣ 🛠️ 작업한거 정리한 튜토리얼


---

✅ 1. Termux에 SSH 서버와 tmux를 설치한다

pkg update
pkg install openssh tmux

openssh → SSH 서버(sshd)

tmux → sshd를 관리하기 위한 관리 콘솔

이 시점부터 전제는 하나다.

> tmux ≠ sshd
sshd ≠ tmux



서로 생명주기 공유 안 한다.
이걸 섞어서 생각하는 순간부터 헷갈림이 시작된다.


---

🔐 2. SSH 비밀번호 인증을 사용할 수 있게 설정한다

Termux의 SSH 서버는 기본 사용자(termux) 기준으로 동작한다.
비밀번호를 한 번도 설정하지 않으면 로그인 자체가 불가능하다.

passwd

이 단계는 초기 1회 필수다.


---

⚙️ 3. sshd의 생존 주기(Lifecycle)를 정확히 이해하기

이 구성이 헷갈리는 이유의 90%는
sshd의 생존 주기를 터미널 프로그램처럼 착각하기 때문이다.

여기서 그 착각을 완전히 끊어내야 한다.


---

🔥 sshd는 “명령어”가 아니라 “데몬”이다

sshd

이걸 실행하는 순간:

sshd는 백그라운드 데몬 프로세스로 전환된다

실행한 터미널과의 연결이 끊긴다

부모 쉘이 사라져도 살아남는다


즉,

상황	sshd 상태

터미널 종료	❌ 안 꺼짐
tmux 세션 종료	❌ 안 꺼짐
앱 화면 닫기	❌ 안 꺼짐
폰 화면 꺼짐	❌ 안 꺼짐


명시적으로 죽이지 않는 한 계속 살아 있다.


---

🧟 “나 분명 tmux 닫았는데 SSH 접속이 되네?”의 정체

이 상황은 버그도 아니고, Termux 문제도 아니다.
정상 동작이다.

tmux new -s test
sshd
exit

이렇게 해도 sshd는:

pgrep -a sshd

→ 멀쩡히 살아 있다.

👉 이유는 간단하다.
sshd는 tmux의 자식 프로세스가 아니기 때문이다.


---

🔍 sshd의 상태 확인은 “프로세스 기준”으로만 한다

터미널 상태 ❌
tmux 세션 상태 ❌
Shortcut 실행 여부 ❌

오직 프로세스만 본다.

pgrep -a sshd

결과 있음 → 서버 살아 있음

결과 없음 → 서버 꺼짐

이 기준 하나만 믿으면 된다.


---

☠️ sshd 종료는 반드시 직접 한다

pkill sshd

또는 PID 잡아서:

kill <PID>

이걸 안 하면:

LAN 안에서 계속 열려 있음

포트 계속 리슨 중

“임시 서버”라는 전제가 무너짐


그래서 이 글 전체 구조가
“자동 종료”가 아니라 “명시적 종료”로 설계된 이유다.


---

🧩 4. tmux의 역할을 다시 정확히 정의한다

tmux는 딱 하나다.

> sshd를 켜고 / 상태를 확인하기 위한 관리용 터미널



그 이상도 이하도 아니다.


---

tmux에서 할 수 있는 것

tmux new -s ssh-admin
sshd
ip addr show

sshd 실행

IP 확인

로그 확인



---

tmux에서 기대하면 안 되는 것

sshd 생명주기 관리 ❌

자동 종료 ❌

상태 판단 ❌


Ctrl + b → d

이건 tmux에서 나가는 것이지
sshd를 끄는 행위가 아니다.


---

🚫 5. 왜 자동 종료 구조를 버렸는가

tmux hook 같은 걸 쓰면 이런 생각이 든다.

> “세션 닫히면 알아서 sshd도 꺼지게 하면 되지 않나?”



이론상은 가능하다.
현실은 다르다.


---

tmux hook이 위험한 이유

pane 종료 ≠ session 종료

detached 상태에서는 훅 안 타는 경우 있음

SSH 접속 중인 세션이 있어도 훅이 먼저 실행될 수 있음

언제 꺼질지 사람이 예측하기 어려움


결국 이런 상황이 나온다.

“왜 안 꺼졌지?”

“왜 갑자기 끊겼지?”

“지금 열려 있는 거야, 아닌 거야?”


👉 임시 서버에서 이건 치명적이다.


---

📱 6. Termux Shortcut 기반으로 열기 / 닫기 스크립트 분리

그래서 최종적으로 선택한 구조는 이거다.

> 사람이 직접 연다
사람이 직접 닫는다
자동 판단 없음




---

📂 스크립트 위치

~/.shortcuts/


---

▶️ SSH 서버 열기 (ssh-open)

#!/data/data/com.termux/files/usr/bin/sh

SESSION="sshd"

# create session if not exists
tmux has-session -t "$SESSION" 2>/dev/null
if [ $? -ne 0 ]; then
  tmux new-session -d -s "$SESSION"
fi

# run sshd inside that tmux session (no output is normal)
tmux send-keys -t "$SESSION" "sshd" C-m
tmux send-keys -t "$SESSION" "echo 'IP:'; ifconfig | awk '/inet / && \$2 != \"127.0.0.1\" {print \" \", \$2}'" C-m

# (optional) auto-attach when you run the shortcut
tmux attach -t "$SESSION"


---

⏹️ SSH 서버 닫기 (ssh-close)

#!/data/data/com.termux/files/usr/bin/sh

SESSION="sshd"

echo "🔒 SSHD 종료 시퀀스 시작"

# 1) tmux 세션 종료 (있으면)
if tmux has-session -t "$SESSION" 2>/dev/null; then
  echo "🧨 tmux 세션 '$SESSION' 종료"
  tmux kill-session -t "$SESSION"
else
  echo "ℹ️ tmux 세션 '$SESSION' 없음"
fi

# 2) sshd 종료 (남아있으면)
if pgrep -f sshd >/dev/null; then
  echo "🛑 sshd 프로세스 종료"
  pkill sshd
else
  echo "ℹ️ sshd 이미 꺼짐"
fi

echo "✅ 종료 완료"


---

🔁 7. 실제 사용 흐름 (생존 주기 기준)

1. 📱 ssh-open 실행


2. sshd 생성됨 (데몬)


3. IP 확인


4. 💻 PC에서 접속



ssh termux@192.168.x.x

5. 작업 종료


6. 📱 ssh-close 실행


7. pgrep sshd → 결과 없음 = 완전 종료




---

📝 3️⃣ 작업 후기

이번 작업으로 Termux의 shortcut을 통해 버튼 딸깍으로 sshd를 열어 같은 네트워크 안에 기기들이 접속할 수 있게 되었다.
솔직히 언젠가는 이것도 cloudflared기반으로 열고 닫는것도 만들것 같다.(군대라 같은 네트워크가 쉽지아늠;;)
