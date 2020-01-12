---
layout: post
title:  "기본적인 JPA 설정방법"
date:   2018-10-07 00:26:58
category: backend-JPA
tags: [JPA,Hibernate]
comments: true
draft: false
---
JPA 구현체로 Hibernate를 사용하기 위해 필요한 핵심 라이브러리가 존재한다.

- hibernate-core : 하이버네이트 라이브러리
- hibernate-entitymanager : 하이버네이트가 JPA 구현체로 동작하도록 JPA  표준을 구현한 라이브러리
- hibernate-jpa-2.1-api : JPA 2.1 표준을 모아둔 라이브러리
<!--more-->
먼저 pom.xml에 다음과 같이 추가한다.

```xml
<!-- JPA, 하이버네이트 -->
<dependency>
    <groupid>org.hibernate</groupid>
    <artifactid>hibernate-entitymanager</artifactid>
    <version>4.3.10.Final</version>
</dependency>
```

entitymanager를 dependecy 해주면 관련하여 core와 jpa-2.1-api를 함께 dependency로 잡아준다.  

다음 persistence.xml을 설정한다.  

persistence.xml 파일은 JPA 사용에 필요한 설정 정보를 관리하는 파일로 보통 클래스패스의 META-INF 디렉토리 밑에 위치시키면 별도의 설정 없이 JPA가 인식한다.  

```xml
<!--?xml version="1.0" encoding="UTF-8"?-->
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.1">
    <persistence-unit name="jpabook">
        <properties>
            <!-- 데이터베이스 접속정보 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver">
            <property name="javax.persistence.jdbc.user" value="sa">
            <property name="javax.persistence.jdbc.password" value="">
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test">
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect">

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true">
            <property name="hibernate.format_sql" value="true">
            <property name="hibernate.use_sql_comments" value="true">
            <property name="hibernate.id.new_generator_mappings" value="true">
            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </property></property></property></property></property></property></property></property></property></properties>
    </persistence-unit>
</persistence>
```

DB접속정보 등은 본인의 상황에 맞게 변경하여 사용한다.

- javax.persistence.jdbc ~ 이하 설정들은 JPA 표준 속성이다.  
- hibernate.dialect는 하이버네이트에서 사용하는 속성이다.  

즉 javax.persistence 로 시작하는 속성들은 특정 JPA 구현체에 종속되지 않고, hibernate로 시작하는 속성은 하이버네이트에서만 사용가능한 속성이다.  

hibernate.dialect 속성의 경우 하이버네이트가 어떤 데이터베이스 방언(dialect)를 사용할 것인지 지정해주는 속성으로 본인이 사용하는 DB의 종류를 설정해준다.  
