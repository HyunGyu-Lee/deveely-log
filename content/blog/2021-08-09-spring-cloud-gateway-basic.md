---
layout: post
title: "Spring Cloud Gateway 기본 활용법"
date:   2021-08-09 23:27:19
category: backend-spring-framework
tags: [Spring Boot,Spring Framework,Spring Cloud]
comments: true
draft: false
---

## 0. Spring Cloud Gateway 란?
Netflix OSS의 API Gateway 컴포넌트인 `Zuul`을 Spring 진영에서 직접 만든 API Gateway 입니다.
Zuul은 기본적으로 블록킹 방식으로 동작했었는데요. (Zuul 1.x) 이를 개선하기 위해 Zuul 2.x에서 논블록킹 방식을 도입했습니다.

Spring 진영에서는 Zuul의 동기방식이었던 단점을 보완하며 Spring 생태계에 더 적합한 형태의 비동기 API Gateway를 만든것이 바로 `Spring Cloud Gateway` 입니다.

```
This project provides an API Gateway built on top of the Spring Ecosystem, including: Spring 5, Spring Boot 2 and Project Reactor. Spring Cloud Gateway aims to provide a simple, yet effective way to route to APIs and provide cross cutting concerns to them such as: security, monitoring/metrics, and resiliency.
```

- Spring Boot 2.4.x 이상부터 Zuul을 사용할 수 없습니다.
- spring-webmvc 위에서 동작하던 Zuul과 달리 Spring Cloud Gateway는 spring-webflux + Reactor 기반으로 동작합니다.
  - 때문에 spring-webmvc, spring-data, spring-security 등 동기방식 기반 프로젝트들과 함께 실행하면 문제가 발생할 수 있습니다.

## 1. Spring Cloud Gateway 의존성 추가
- build.gradle
```groovy
plugins {
	id 'org.springframework.boot' version '2.5.2'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
}

group = 'com.sample.project'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

repositories {
	mavenCentral()
}

// Spring Cloud 의존성 관리 
dependencyManagement {
	imports {
		mavenBom "org.springframework.cloud:spring-cloud-dependencies:2020.0.3"
	}
}

// Spring Cloud Gateway 의존성 추가
dependencies {
	implementation 'org.springframework.cloud:spring-cloud-starter-gateway' 
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

## 2. 라우팅 설정 추가
- application.yml
- 기본적으로 라우팅 정보, 라우팅 대상 지정 (predicates), 라우팅 시 동작 추가 (filters)를 할 수 있습니다.
    - predicates를 통해 게이트웨이로 들어온 요청 중 어떤 요청들을 해당 라우트에서 처리할 지 조건을 기술합니다.
        - 시간, 쿠키, 헤더, 호스트,  메소드, 쿼리 스트링 등 사용할 수 있는 조건이 다양합니다.
            - https://cloud.spring.io/spring-cloud-gateway/reference/html/#gateway-request-predicates-factories
    - filters를 통해 게이트웨이로 들어온 요청이 해당 라우트를 거치며 수행될 추가 동작들을 선언할 수 있습니다.
        - 헤더 추가, 쿼리 스트링 추가, MSA 환경의 경우 써킷 브레이커 추가, Path 재설정 등등 다양합니다.
            - https://cloud.spring.io/spring-cloud-gateway/reference/html/#gatewayfilter-factories
- 아래 설정은 게이트웨이로 들어오는 요청을 아래와 같이 처리합니다.
    - `/api/monitoring` 로 시작하는 요청만을 대상으로 하며
    - Path를 `/api/monitoring/*` => `/monitor/*`로 변경하여
    - uri에 설정된 서버로 라우팅

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: 서비스 아이디
          uri: 라우팅할 서비스 URI
          predicates:
            - Path=/api/monitoring/**
          filters:
            - RewritePath=/api/monitoring, /monitor
```