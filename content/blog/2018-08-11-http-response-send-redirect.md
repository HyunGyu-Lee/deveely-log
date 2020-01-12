---
layout: post
title:  "HTTP Redirection 응답 보내기"
date:   2018-08-11 18:22:13
category: etc-it-basic
tags: [HTTP,리다이렉트]
comments: true
draft: false
---
간단한 웹서버를 제작중인데, 기본적으로 요청한 URI에 해당하는 파일을 내려주는 기능을 하지만 미약하게나마 WAS의 역할을 할 수 있게끔 혼종스러운 느낌으로 간단하게 만들던 중...  

어플리케이션단에서 redirect응답을 주면 어떻게해야하나? 라는 고민이 생겼다.  
<!--more-->
스프링mvc에서 개발할 땐 viewName앞에 "redirect:"를 붙히면 되고, Servlet에서는 HttpServletRequest의 redirect메소드를 호출하면 되는데.. 그보다 더 로우레벨에서는 어떻게 동작하지..?  

한 번도 위 기능들이 어떻게 동작하는지 생각해보지 않아 구글링해 본 결과 간단했다.  

응답코드를 302로 지정한 후 헤더에 "Location: REDIRECT_URL" 을 추가해주면 된다.  

[예시 HTTP Response]
```sh
HTTP/1.1 302 Message for 302
Location: /main
```

위 HTTP 응답을 받은 브라우저는 알아서 REDIRECT_URL로 요청을 날린다.
