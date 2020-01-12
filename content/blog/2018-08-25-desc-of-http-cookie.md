---
layout: post
title:  "HTTP 쿠키(Cookie) 란?"
date:   2018-08-25 18:44:21
category: etc-it-basic
tags: [HTTP,Cookie]
comments: true
draft: false
---
> HTTP 쿠키(웹 쿠키, 브라우저 쿠키)는 HTTP 메세지의 헤더에 담겨 서버와 브라우저간 주고받는 작은 데이터 조각이다.
쿠키를 이용하면 상태가 없는(stateless) HTTP 프로토콜에서 사용자 로그인 상태 유지 등 다양한 상태 기반 정보를 기억할 수 있다.

기본적으로 웹 서버는 다음과 같은 형태로 브라우저에 쿠키를 저장하라고 알릴 수 있다.
<!--more-->
```sh
Set-Cookie: <CookieName>=<CookieValue>


HTTP/1.0 200 OK
Content-type: text/html
Set-Cookie: test=testvalue
Set-Cookie: mycookie=good

[page content]
```

위 HTTP 응답을 받은 브라우저는 다음 해당 서버에 요청할 때 다음과 같이 보낼 것이다.

```sh
GET /sample_page.html HTTP/1.1
Host: www.example.org
Cookie: test=testvalue; mycookie=good
```

쿠키에 대한 유효기간은 서버에서 쿠기 송신 시 설정할 수 있고, 설정이 안된 경우는 브라우저를 닫는 순간 사라진다.

```sh
Set-Cookie: test=testvalue; Expires=Wed, 21 Oct 2018 07:28:00 GMT;
```
