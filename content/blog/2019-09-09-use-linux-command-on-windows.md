---
layout: post
title:  "Windows에서 Linux 명령어 사용환경 구성하기"
date:   2019-09-09 11:12:19
category: backend-unix/linux
tags: [Unix,Linux]
comments: true
draft: false
---
개발자 특성상 Windows, Linux 등 다양한 환경의 PC를 이용하게 되는데요. 저는 주로 일반 사무, 업무를 볼 때는 Windows, 개발할 땐 Linux 환경을 이용합니다.
다만 직업 특성상 Linux 환경에 노출되는 시간이 많고, 자연스레 Linux 명령어에 익숙해지게 되는데요. 이 때문에 가끔 Windows의 CMD 명령어들이 Linux와 달라 불편함을 종종 느낄때가 있습니다.
이 때문에 이번에 개인적으로 사용하는 PC (Windows)에 [Cygwin](https://cygwin.com/) 을 설치해봤습니다.

<!--more-->

## Cygwin 이란
* 공식 홈페이지 : https://cygwin.com/
* 설치 : https://cygwin.com/install.html

POSIX 기반 소프트웨어를 Windows에서 구동 및 개발이 가능하도록 환경을 제공해주는 역할을 합니다.
가상머신 등을 통해 유닉스용 어셈블리 코드를 에뮬레이션 하여 진짜 유닉스/리눅스 등의 운영체제를 사용하는건 아니고, `Cygwin1.dll`이라는 런타임 라이브러리를 통해 Windows에 동적으로 링크하여
실제 유닉스처럼 동작하도록 해줍니다.

## 설치
[설치페이지](https://cygwin.com/install.html)에 접속하여 32/64 bit에 맞는 setup파일을 받아 쉽게 설치할 수 있습니다.

## 사용
Cygwin을 실행하여 `pwd`를 실행해보면 /home/{자신의 userName} 이 출력됩니다.
이는 실제 경로는 아니며 Cygwin 설치경로 이하의 경로입니다. 즉 Cygwin 터미널은 자신의 설치경로가 root인데요.
우리가 원래 사용하고 있던 Windows의 C:, D: 등을 접속하려면 `/cygdrive/{Drive 볼륨명}` 경로로 이동하면 됩니다.
다만 매번 위 경로를 통해 접근하려면 귀찮으니 저같은 경우는 링크를 생성하여 사용하고 있습니다.