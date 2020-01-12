---
layout: post
title:  "Spring AOP 동작 방식, 원리"
date:   2019-10-29 13:03:19
category: backend-spring-framework
tags: [Java,Spring Framework,AOP]
comments: true
draft: false
---

스프링은 기본적으로 프록시 기반 AOP 를 제공한다.
스프링에서는 Java Dynamic Proxy를 사용하거나 Cglib을 사용하여 프록시 기반 AOP를 구현했다.

이 글에서는 어떻게 Spring 에서 AOP를 사용할 수 있는 방식의 차이와 프레임워크가 AOP를 지원하기 위해 어떤 처리를 해주는지 개인적으로 분석한 내용을 정리한다.
<!--more-->
1. AopProxy 라는 Delegator 인터페이스로 표현되며 Dynamic Proxy 기반은 JdkDynamicAopProxy 클래스, Cglib 기반은 CglibAopProxy 클래스이다.

2. Dynamic Proxy 기반과 Cglib 차이는 어떻게 프록시 객체를 생성하는지 방식 차이이다.

3. Dynamic Proxy 는 프록시 객체 생성을 위해 인터페이스를 필수로 구현해야하며, Cglib 은 인터페이스를 구현하지 않은 일반 클래스에 런타임 시 코드 조작으로 프록시 객체를 생성해준다.

4. DefaultAopProxyFactory 클래스의 createAopProxy 메소드를 통해 AopProxy 의 실 구현체가 생성된다.

	4.1. AOP 적용 대상이 인터페이스거나 인터페이스를 구현(implement)하고 있으면 JdkDynamicAopProxy 가 생성된다.

	4.2. 위 경우가 아니면 ObjenesisCglibAopProxy 기반 실 객체가 생성된다.

	4.3. Objenesis 는 과거 Cglib 의 몇 가지 단점을 해결해주는 라이브러리

5. Transaction 대상의 경우 기본이 Cglib 으로 생성하게 설정되어있다.

	5.1. 성능상 Cglib 이 이점이 많고, 예외발생 확률도 적다고 한다. 

	5.2. Cglib 이 가지고 있던 몇 가지 문제점들을 Objenesis 라이브러리를 통해 해결했다. (생성자 중복 호출, default 생성자 필요 문제 등) 

6. 기본 Spring AOP 이외에 스프링에서 사용할 수 있는 방법으로는 AspectJ 가 있다.

7.  Spring AOP 가 제공하는 프록시 기반 방식은 런타임 위빙 (RTW) 이라는 방식으로 프로그램 구동중에 위빙이 일어난다. 반면 AspectJ 는 컴파일 단계 또는 로드타임에 코드를 삽입하여(위빙) RTW 보다 성능면에서 우세하다. (CTW : 컴파일 타임 위빙, LTW : 로드 타임 위빙). 또한 AspectJ 는 기본 Spring AOP 보다 보다 다양한 포인트컷을 지원한다.

8. 일반적인 경우 Spring AOP 에서 지원하는 방식으로 요구사항들이 충분히 해결되는 경우가 많고, 성능 또한 @AspectJ 갯수에 따라 달라지는데 일반적인 경우에서는 크게 체감하기 힘들다는 평이 있다.

9. AspectJ 를 사용하기 위해서는 AJC 등 별도의 컴파일러 설정 등 추가 설정이 필요한데, Spring AOP 는 그렇지 않다.

