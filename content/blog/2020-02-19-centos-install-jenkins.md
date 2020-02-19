---
layout: post
title: "CentOS Jenkins 설치하기"
date:   2020-02-19 11:43:19
category: ci-cd
tags: [CI-CD,Jenkins]
comments: true
draft: false
---
Jenkins는 가장 널리 사용되는 CI(Continous Integration)툴 중 하나입니다.

현 시점에서 많이 사용되는 CI툴로 Travis CI, Jenkins, TeamCity 등이 있습니다.
이 중에서도 Jenkins는 오랜 기간 널리 사용되어 오면서 수많은 사용자와 플러그인, 레퍼런스 등이 갖춰져있습니다.

개인 프로젝트에 적용할 CI툴을 고민해봤는데, Travis CI는 무료정책도 있지만 현재 제 상황에서는 무료정책을 이용할 수 없는 상황이었고, TeamCity도 일정 기준 이상이면 무료로는 사용이 불가하다고하여 우선 무료이면서 가장 보편적인 Jenkins를 구축해보기로 했습니다.

## 설치
먼저 아래 명령어로 Jenkins를 설치합니다. 
```sh
wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
yum install jenkins
```

## 설정
#### 기본 설정
CentOS의 경우 Jenkins Home 디렉토리나 포트 등의 설정은 `/etc/sysconfig/jenkins`에서 진행할 수 있습니다.
저의 경우 머신 하나만 가지고 진행하고 있기 때문에 애플리케이션이 사용하는 포트와 겹치지 않게 하기위해 포트를 변경하였습니다. (9000대 포트를 사용한다고 합니다)
```sh
...

## Type:        integer(0:65535)
## Default:     8080
## ServiceRestart: jenkins
#
# Port Jenkins is listening on.
# Set to -1 to disable
#
JENKINS_PORT="9001"

...
```

#### JRE 설정
Jenkins는 Java 기반으로 제작된 도구이기 때문에 JRE를 필요로 하는데요.

Jenkins는 기본적으로 아래 경로들을 대상으로 JRE 자동으로 탐색합니다.

```sh
/etc/alternatives/java
/usr/lib/jvm/java-1.8.0/bin/java
/usr/lib/jvm/jre-1.8.0/bin/java
/usr/lib/jvm/java-1.7.0/bin/java
/usr/lib/jvm/jre-1.7.0/bin/java
/usr/lib/jvm/java-11.0/bin/java
/usr/lib/jvm/jre-11.0/bin/java
/usr/lib/jvm/java-11-openjdk-amd64
/usr/bin/java
```

만약 수정을 원하면 `/etc/init.d/jenkins` 파일에 `candidate`목록에 본인 환경에 맞는 경로를 추가해주면 됩니다.

위 경로에 Java가 설치되어있지 않은 경우 `No Such File...` 오류가 발생합니다.  

경로가 제대로 잡혔더라도 권한 문제로 `Permission Denied` 오류가 발생할 수도 있는데요. 
이경우엔 `/etc/sysconfig/jenkins` 에 `JENKINS_USER`값을 root로 변경해주면 됩니다.
```sh
...

## Type:        string
## Default:     "jenkins"
## ServiceRestart: jenkins
#
# Unix user account that runs the Jenkins daemon
# Be careful when you change this, as you need to update
# permissions of $JENKINS_HOME and /var/log/jenkins.
#
JENKINS_USER="root"

...
```


#### JRE 관련 추가 설정
이 설정은 OpenJDK를 사용하는 경우에만 해줘야하는 것으로 예상되는데요.

저의 경우 Jenkins 실행은 정상적으로 되지만 페이지 접속 시 AWT 관련 오류가 발생했습니다. 정확한 원인은 파악하지 못했지만 예상되는 원인으로는 사용하는 OpenJDK에 몇몇 라이브러리가 누락되어 발생한 것으로 예상했는데요.

```
after installing Jenkins for integration with FishEye always get error message below trying to access Jenkins site:
AWT is not properly configured on this server. Perhaps you need to run your container with "-Djava.awt.headless=true"?
```

아래 링크에서 OS별 설치방법 확인해서 필요한 라이브러리 설치하여 해결할 수 있었습니다. 
https://wiki.jenkins.io/display/JENKINS/Jenkins+got+java.awt.headless+problem

Centos 의 경우
```java
yum install dejavu-sans-fonts
yum install fontconfig
```

설정을 마친 후 젠킨스 서비스를 실행합니다.

## 실행
아래 명령어를 통해 젠킨스를 실행한 후 `머신주소:설정한포트`로 접속하면 드디어 젠킨스 로그인 페이지를 볼 수 있습니다.
```sh
sudo service jenkins start
```
