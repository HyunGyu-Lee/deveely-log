---
layout: post
title:  "Spring Framework WebApplicationContext 수동으로 참조하는 방법"
date:   2017-04-21 16:31:31
category: backend-spring-framework
tags: [Spring Framework,WebApplicationContext]
comments: true
draft: false
---
Spring Framework 환경에서 개발하는 도중 WebApplicationContext를 코드상에서 활용해야하는 상황이 생겼다.  
보통 Spring Bean의 경우 `ApplicationContext` 인터페이스를 통해 주입받아 쉽게 사용할 수 있는데,  
나는 이런 기능이 지원되지 않는 상황에서 WebApplicationContext를 사용해야하는 상황이 생겼고, ServletContext를 통해 획득하는 방법을 정리한다.  

###### 1. Root ApplicationContext
```java
WebApplicationContextUtils.getWebApplicationContext(서블릿컨텍스트객체);
```

###### 2. DispatcherServlet에 설정한 ApplicationContext
```java
WebApplicationContextUtils.getWebApplicationContext(서블릿컨텍스트객체, FrameworkServlet.SERVLET_CONTEXT_PREFIX + "설정한 서블릿 이름");
```
