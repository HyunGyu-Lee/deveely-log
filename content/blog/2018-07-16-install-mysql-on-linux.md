---
layout: post
title:  "리눅스 MySQL 설치"
date:   2018-07-16 23:06:25
category: etc-unix/linux
tags: [Unix,Linux,MySQL]
comments: true
draft: false
---
```sh
apt-cache search mysql-server
```

위 명령어를 통해 설치 가능한 버전을 확인한 후

```sh
apt-get install mysql-server-[version]
```

<!--more-->
위 명령어를  실행하면 바로 MySQL이 설치된다.

설치가 완료되면 MySQL서버는 실행되어있는 상태이며 기본적으로 3306 포트를 사용한다.

```sh
netstat -anp | grep 3306

---------------------------------------------------------------------------------------------
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      [PID]/mysqld
```

위와 같이 3306포트가 LISTEN상태인 것을 확인할 수 있으며

```sh
mysql -u root -p
```

를 입력한 후, 설치 시 설정한 root계정의 암호를 입력하면 mysql 서버에 정상적으로 접속된다.
