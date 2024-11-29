---
title: GoatCounter를 사용한 블로그 트래픽 확인
author: M0S-quito
date: 2024-11-14 14:10:00 +0800
categories: [Github 블로그 만들기, Tutorial]
tags: [writing]
pin: True
---
## GoatCounter란?
웹 사이트의 트래픽을 분석하는 웹 분석 도구<br>
즉, 내 웹사이트에 사람들이 얼마나 많이 들어왔는지를 알 수 있다!!

## Step 1. GoatCounter사이트 세팅
- [goatcounter 공식 사이트](https://www.goatcounter.com/) 접속

#### 1-1. 회원 가입 창
1. **Code**: 내 트래픽을 확인할 goatcounter주소 이름
    - 에를 들어 **M0Squito -> https://M0Squito.goatcounter.com**으로 접속 가능
2. **Site domain**: 내 사이트 도메인, **https://username.github.io** 작성
    - 에를 들어 github닉네임이 M0S-quito라면 https://M0S-quito.github.io 작성
3. **Email address**: 개인 이메일 주소 작성, 나중에 로그인 할 때나 계정 설정 할 때 사용
4. **Password**: 내 goatcounter에 접속할 때 사용할 비밀번호 작성
5. **fill~here**: 로봇이 아닙니다 체크 같은거임 그냥 '~' 에 들어가는 숫자 넣으면 됨


## Step 2. 내 웹사이트 파일에 설정

#### 2-1. 웹사이트 head부분에  
1. goatcounter에서 주는 script구분 복사
2. _layout 이나 _head와 같은 파일 즉, 모든 html에 head에 들어가는 부분에 삽입
    - 이거 body에도 해도 되는데 어디서 읽어 오냐 차이라 head에 두는게 빨리 정보가 들어옴

#### 2-2(선택). 만약 내가 github의 root(username.github.io)이외에 다른 리포지토리로 웹사이트를 운영 한다면?
- 각각의 웹 사이트에 다 **1-2**을 적용 해야함 


