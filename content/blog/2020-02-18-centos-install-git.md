---
layout: post
title: "CentOS Git 설치하기"
date:   2020-02-18 11:43:19
category: git
tags: [Git]
comments: true
draft: false
---
CentOS에서 git을 설치해 사용하는 방법을 간략하게 정리합니다.

#### 1. 설치확인
설치 전에 먼저 이미 설치되어있는지 확인합니다.
```sh
git --version
```

command not found 오류가 발생하거나, git이 설치되어있지 않다는 오류 문구가 나면 설치되지 않은 것입니다.

#### 2. 설치
아래 명령어를 통해 git을 설치할 수 있습니다.
```sh
yum install git
```

위와 같이하면 yum에서 제공되는 git 버전이 자동으로 설치되는데,  만약 원하는 버전이 있다면 아래와 같이 버전을 명시하면 됩니다.
```sh
yum install git-<version>
```

#### 3. 사용자 설정
```sh
git config --global user.name "userName"
git config --global user.email "email"
```

#### 4. 정상 설치 확인
```sh
git --version
```
