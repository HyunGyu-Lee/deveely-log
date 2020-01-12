---
layout: post
title:  "리눅스 Git 설치 및 원격 Repository clone"
date:   2018-07-15 20:41:05
category: etc-unix/linux
tags: [Unix,Linux,Git]
comments: true
draft: false
---
###### 진행 OS버전 : Ubuntu 14.04 LTS 버전
```sh
apt-get install git
```

위 명령어를 통해 git을 설치한 후

```sh
git config --global user.name "자신의 계정"
git config --global user.email "자신의 이메일"
git config --global color.ui "auto"
```

<!--more-->
을 통해 글로벌 설정을 진행한다.

이후 clone받아 로컬 repository를 구성할 디렉토리를 생성한 후 해당 디렉토리로 이동하여 clone 진행

```sh
git clone [clone할 git 주소]
```
