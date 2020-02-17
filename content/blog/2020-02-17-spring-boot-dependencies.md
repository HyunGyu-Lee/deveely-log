---
layout: post
title: "스프링 부트 (Spring Boot)와 의존성 관리"
date:   2020-02-17 11:43:19
category: backend-spring-framework
tags: [Spring Boot,Spring Framework,Java]
comments: true
draft: false
---
스프링 부트의 강력한 장점 중 하나는 스프링, 써드파티 라이브러리 의존성을 관리해주는 부분이라고 생각합니다. 
의존성 관련 내용에 앞서 스프링 부트가 무엇인지 간략하게 소개하겠습니다.

## 스프링 부트 (Spring Boot)란?
스프링 부트 공식 홈페이지에서는 다음과 같이 소개하고 있습니다. 
> Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can "just run".

상용 수준의 독립실행형 스프링 애플리케이션을 쉽게 만들 수 있도록 해주는 기술로 대표적으로 다음과 같은 기능을 제공합니다.
 
- 독립 실행형 애플리케이션 제작 Create stand-alone Spring applications
- 내장 WAS 지원(Tomcat, Jetty, Undertow) 
- 빌드 구성을 단순화해주는 `starter` 의존성 제공 
- 스프링 및 써드파티 라이브러리 자동 구성 (Auto Configuration)
- 메트릭, 헬스체크, 설정 외부화 등 상용 기능 제공
- XML 설정 등 필요 X


## Spring Boot 의 의존성 관리
위에서 언급한대로 스프링 부트는 `starter`라 불리는 의존성을 제공합니다.

각 `starter`는 관련있는 의존성들의 묶음으로 하나의 `starter` 의존성으로 해당 의존성과 관련된 스프링 & 써드파티 라이브러리 또는 또다른 `starter` 의존성까지도 의존성 간 버전, 호환성 걱정 없이 한번에 애플리케이션에 포함시킬 수 있습니다.

아래 링크에서 Spring Boot에서 제공하는 모든 `starter` 의존성을 확인할 수 있습니다.
https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-starter

대표적인 `spring-boot-starter-web`은 웹 관련 기능의 집합으로 기존에 `sprinb-web`, `spring-webmvc`을 포함한 여러 라이브러리를 포함시켜야했던 때와 달리 위 의존성 하나로 웹 관련 의존성을 버전 등이 최적화된 상태로 포함시킬 수 있습니다. 

- `spring-boot-starter-web`의 pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starters</artifactId>
        <version>${revision}</version>
    </parent>
    <artifactId>spring-boot-starter-web</artifactId>
    <name>Spring Boot Web Starter</name>
    <description>Starter for building web, including RESTful, applications using Spring
        MVC. Uses Tomcat as the default embedded container</description>
    <properties>
        <main.basedir>${basedir}/../../..</main.basedir>
    </properties>
    <scm>
        <url>${git.url}</url>
        <connection>${git.connection}</connection>
        <developerConnection>${git.developerConnection}</developerConnection>
    </scm>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
	        <groupId>org.springframework.boot</groupId>
	        <artifactId>spring-boot-starter-json</artifactId>
	    </dependency>
	    <dependency>
	        <groupId>org.springframework.boot</groupId>
	        <artifactId>spring-boot-starter-tomcat</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.tomcat.embed</groupId>
                    <artifactId>tomcat-embed-el</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
        </dependency>
    </dependencies>
</project>
```


## Spring Boot 의존성의 버전 관리
위 `spring-boot-starter-web`의 pom.xml을 보면 각 라이브러리의 버전정보가 적혀있지 않은데요. 그 이유는 버전정보의 경우 별도의 파일에서 관리하고 있기 때문입니다.

보통 스프링 부트를 사용해 개발하면 Maven이던 Gradle이던 `spring-boot-starter-parent`라는 의존성을 상속하게 되는데요. 이 `starter` 의존성이 상속하고 있는 `spring-boot-dependencies` 라는 의존성에 모든 의존성의 버전정보가 들어있습니다.

- `spring-boot-dependencies`의 pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    ....
    <properties>
        <main.basedir>${basedir}/../..</main.basedir>
        <!-- Dependency versions -->
        <activemq.version>5.15.11</activemq.version>
        <antlr2.version>2.7.7</antlr2.version>
        <appengine-sdk.version>1.9.77</appengine-sdk.version>
        <artemis.version>2.10.1</artemis.version>
        <aspectj.version>1.9.5</aspectj.version>
        <assertj.version>3.13.2</assertj.version>
        <atomikos.version>4.0.6</atomikos.version>
        <awaitility.version>4.0.2</awaitility.version>
        <bitronix.version>2.1.4</bitronix.version>
        <byte-buddy.version>1.10.6</byte-buddy.version>
        ...
        </properties>
        ...
</project>        
```

#### 라이브러리 버전 Override
위 처럼 `properties`에 각 라이브러리 별 버전 프로퍼티가 선언되어있고, 사용측에서 이 프로퍼티를 참조하기 때문에, 버전을 변경할 일 이 있으면 위 프로퍼티명을 참고하여 우리 프로젝트의 pom.xml에서 버전정보를 Override 할 수 있습니다.

- Project의 pom.xml
```xml
...
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>사용할 Spring Boot 버전</version>
</parent>
<properties>
    여기에서 프로퍼티 정보를 통해 기본 의존성의 버전을 변경할 수 있습니다.
</properties>
<dependencies>
    사용할 spring-boot-starter 모듈들 선언....
</dependencies>
...
```

## 참고
- [Spring Boot 공식 홈페이지](https://spring.io/projects/spring-boot)
- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)
