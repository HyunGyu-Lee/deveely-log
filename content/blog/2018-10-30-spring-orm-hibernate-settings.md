---
layout: post
title:  "Spring ORM (Hibernate) 설정"
date:   2018-10-30 01:01:35
category: backend-JPA
tags: [JPA,Hibernate,Spring ORM]
comments: true
draft: false
---
보통 스프링 프레임워크로 웹 프로젝트 설정 시 Web MVC 관련 설정과 Service레이어 이하 애플리케이션 관련 설정을 분리하여 설정합니다.  
이 포스트는 Spring 프레임워크 환경에서 JPA를 사용하기 위한 애플리케이션 관련 필수 설정을 다룹니다.
<!--more-->
---
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
    <!-- 1. 어노테이션(@Transactional) 기반 트랜잭션 매니저 활성화 -->
    <tx:annotation-driven/>
    <!-- 2. 애플리케이션에서 사용되는 컴포넌트 스캔 패키지 설정 -->
    <context:component-scan base-package="my.webservice.biz">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!-- 3. 데이터소스(커넥션풀) 설정 - Hikari커넥션 풀 이용 -->
    <bean id="hikariConfig" class="com.zaxxer.hikari.HikariConfig">
        <property name="poolName" value="springHikariCP" />
        <property name="connectionTestQuery" value="SELECT 1" />
        <property name="dataSourceClassName" value="${hibernate.dataSourceClassName}" />
        <property name="maximumPoolSize" value="${hibernate.hikari.maximumPoolSize}" />
        <property name="idleTimeout" value="${hibernate.hikari.idleTimeout}" />

        <property name="dataSourceProperties">
            <props>
                <prop key="url">${dataSource.url}</prop>
                <prop key="user">${dataSource.username}</prop>
                <prop key="password">${dataSource.password}</prop>
            </props>
        </property>
    </bean>
    <bean id="dataSource" class="com.zaxxer.hikari.HikariDataSource" destroy-method="close">
        <constructor-arg ref="hikariConfig" />
    </bean>

    <!-- 4. 트랜잭션매니저 설정 -->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 5. JPA 예외 추상화 AOP 설정 -->
    <bean class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"/>

    <!-- 6. 엔티티매니저팩토리 설정 -->
    <bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="packagesToScan" value="package.for.entity"/> <!-- @Entity 탐색 시작 위치 -->
        <property name="jpaVendorAdapter">
            <!-- 하이버네이트 구현체 사용 -->
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter"/>
        </property>
        <property name="jpaProperties"> <!-- 하이버네이트 상세 설정 -->
            <props>
                <prop key="hibernate.dialect">org.hibernate.dialect.H2Dialect</prop> <!-- 방언 -->
                <prop key="hibernate.show_sql">true</prop>                   <!-- SQL 보기 -->
                <prop key="hibernate.format_sql">true</prop>                 <!-- SQL 정렬해서 보기 -->
                <prop key="hibernate.use_sql_comments">true</prop>           <!-- SQL 코멘트 보기 -->
                <prop key="hibernate.id.new_generator_mappings">true</prop>  <!-- 새 버전의 ID 생성 옵션 -->
                <prop key="hibernate.hbm2ddl.auto">create</prop>             <!-- DDL 자동 생성 -->
            </props>
        </property>
    </bean>

</beans>
```

#### 1. 어노테이션(@Transactional) 기반 트랜잭션 매니저 활성화
스프링 프레임워크에서 제공하는 어노테이션 기반 트랜잭션 매니저를 활성화합니다. @Transactional이 붙은 클래스, 메소드에 트랜잭션을 적용해줍니다.

#### 2. 애플리케이션에서 사용되는 컴포넌트 스캔 패키지 설정
보통 @Controller는 Web MVC영역 설정으로 잡히도록 하기 때문에 @Controller를 제외하고 스캔하도록 설정

#### 3. 데이터소스(커넥션풀) 설정 - Hikari커넥션 풀 이용
커넥션 풀 설정입니다. 이 포스트에서는 HikariCP를 이용했으며 다른 커넥션풀을 이용해도 무방합니다.

#### 4. 트랜잭션 매니저 설정
그동안 MyBatis 등 프레임워크을 사용할 땐 DataSourceTransactionManager를 많이 사용해왔는데, JPA를 이용할 때엔 JpaTransactionManager를 설정해줘야한다고 합니다.  
JpaTransactionManager는 DataSourceTransactionManager가 하는 일을 수행하기 때문에 JPA가 아닌 타 프레임워크(MyBatis, JdbcTemplate 등) 과도 사용이 가능합니다.

#### 5. JPA 예외 추상화 AOP 설정
JPA에서 발생한 예외 (JPA예외)를 스프링에서 추상화한 예외로 변환하는 역할을 하는 AOP를 적용합니다.

#### 6. 엔티티매니저팩토리 설정
엔티티매니저팩토리 설정은 직관적으로 이해할 수 있습니다.   
다만 Java SE 환경에서는 META-INF/persistence.xml 파일을 자동인식하여 엔티티매니저팩토리를 설정했지만  스프링 프레임워크에서는 이와같이 스프링다운 방식대로 엔티티매니저팩토리를 하나의 빈으로 설정하여 persistence.xml없이 사용할 수 있습니다.

---
>자바 ORM 표준 JPA 프로그래밍을 공부하며 정리한 내용입니다.
