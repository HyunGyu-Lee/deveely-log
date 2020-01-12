---
layout: post
title:  "스프링 인터셉터(interceptor) 활용"
date:   2017-01-12 18:21:11
category: backend-spring-framework
tags: [Java,Spring Framework, Interceptor]
comments: true
draft: false
---
인터셉터의 preHandle, postHandle 메소드에 보면 `Object형 handler`라는 이름의 인자가 있다.

이 인자는 `RequestMapping`으로 매핑된 하나의 메소드로 스프링 프레임워크에 의해 `org.springframework.web.method.HandlerMethod`라는 클래스로 바인드 되어 전달되는 인자다.  
<!--more-->
getMethod()를 호출하면 실제 java Reflection의 Method형 객체를 얻을수 있다.

Custom Annotation을 사용한다면 Interceptor에서 위 handler 객체를

```java
HandlerMethod method = ((HandlerMethod)handler);
MyAnnotation anno = method.getMethodAnnotation(MyAnnotation.class);
```

위와같이 자신이 선언해둔 어노테이션을 가져올 수 있다.
