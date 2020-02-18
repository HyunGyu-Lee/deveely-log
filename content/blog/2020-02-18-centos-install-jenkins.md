---
layout: post
title: "CentOS Jenkins 설치하기"
date:   2020-02-17 11:43:19
category: ci-cd
tags: [CI-CD,Jenkins]
comments: true
draft: false
---

젠킨스 설치

설치~

나의 경우 개인적으로 사용하기 위해 하나의 머신에 여러개 프로그램이 띄워져있음(80, 8000 등)
때문에 겹치지 않게 젠킨스를 변경 (보통 9000대 포트를 사용한다고 합니다)

문제1
No Such File .. 오류 발생
Java 경로 문제 -> /etc/init.d/jenkins 에 candidate에 내가 설치한 자바 경로 추가

문제2
Java 실행 관련 Permission Denied 발생
권한문제 -> /etc/sysconfig/jenkins 에 JENKINS_USER를 root로 변경

문제3
fter installing Jenkins for integration with FishEye always get error message below trying to access Jenkins site:

AWT is not properly configured on this server. Perhaps you need to run your container with "-Djava.awt.headless=true"?

접속했더니 오류 페이지 뜸
-> OpenJDK를 사용해 라이브러리가 없어 발생하는 문제라함

아래 링크에서 OS별 설치방법 확인해서 필요한 라이브러리 설치 
https://wiki.jenkins.io/display/JENKINS/Jenkins+got+java.awt.headless+problem

centos 의 경우

sudo yum install dejavu-sans-fonts
sudo yum install fontconfig

이후 젠킨스 재시작

암호 설정 (최초 인증 단계 같음)
/var/lib/jenkins/secrets/initialAdminPassword의 값을 확인 후 입력

이후 로그인 페이지에서 admin/위 값 으로 로그인
