---
layout: post
title: "리눅스 SSH, MySQL 접속 지연 문제"
date:   2019-11-21 13:43:19
category: etc-unix/linux
tags: [linux,unix,ssh,mysql]
comments: true
draft: false
---
신규로 발급받은 서버에서 SSH, MySQL 접속 시 상당한 시간이 딜레이되는 현상이 있었습니다.
구글링 해본 결과 서버 측에서 접속한 클라이언트의 IP를 `resolve`하는 과정에서 발생하는 DNS lookup 등으로 발생하는 지연이 원인이었습니다.
장비, 환경 여러가지에 따라 다르겠지만 제 경우엔 Spring Boot 애플리케이션 구동하는데 DB접속하는 부분에서 20초가량 지연되는 심각한 상황이었습니다.
<!--more-->
심각한 상황과 달리 해결방법은 아주 심플하고 간단했습니다.

## MySQL 접속 지연 문제 해결
my.cnf (주로 /etc/my.cnf 에 위치함) 파일에 클라이언트 IP resolve 과정을 스킵하는 설정을 추가해줍니다.
바로 `skip-name-resolve` 옵션을 mysqld 섹션에 넣어주면 됩니다.

```
[mysqld]
...

# 추가
skip-name-resolve

...
```

설정을 추가한 후에 MySQL 서버를 재기동해줍니다.
```
service mysql restart
```

재기동 완료 후 접속해보니.. 거짓말처럼 딜레이 없이 빠르게 접속되었습니다.

## SSH 접속 지연 문제 해결
sshd_config (/etc/ssh/sshd_config) 파일에 마찬가지로 지연을 일으키는 동작을 하지 않도록 설정을 추가해줍니다.

GSSAPIAutentication 과 UseDNS를 no 로 지정해주면 됩니다.

제 경우엔 GSSAPIAutentication 설정의 경우 yes 로 되있었고 UseDNS 는 주석처리 되어있었는데요.
둘 다 명시적으로 no 옵션을 주었습니다.

```
...

# GSSAPI options
GSSAPIAuthentication no
#GSSAPIAuthentication yes
#GSSAPICleanupCredentials yes
GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

...

UseDNS no

...
```

다음 sshd 서비스를 재기동 해줍니다.
```
service sshd restart
```

마찬가지로 로컬에서 ssh 접속 시 발생하던 지연은 없고 아주 빠르게 접속되었습니다.
