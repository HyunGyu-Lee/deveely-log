---
layout: post
title:  "Spring Framework 환경에서 Apache Commons Configuration 사용"
date:   2018-07-14 01:33:01
category: backend-spring-framework
tags: [Spring Framework,Apache Commons Configuration]
comments: true
draft: false
---
Spring Framework 환경에서 Apache Commons의 Configuration 라이브러리를 사용하는 방법을 정리합니다.
<!--more-->
###### 1. pom.xml에 Dependency 추가
```xml
<dependency>
    <groupId>commons-configuration</groupId>
    <artifactId>commons-configuration</artifactId>
    <version>1.10</version>
</dependency>
<dependency>
    <groupId>commons-collections</groupId>
    <artifactId>commons-collections</artifactId>
    <version>3.2.1</version>
</dependency>
<dependency>
    <groupId>commons-lang</groupId>
    <artifactId>commons-lang</artifactId>
    <version>2.6</version>
</dependency>
<dependency>
    <groupId>org.springmodules</groupId>
    <artifactId>spring-modules-jakarta-commons</artifactId>
    <version>0.8</version>
    <exclusions>
        <exclusion>
            <artifactId>jstl</artifactId>
            <groupId>javax.servlet</groupId>
        </exclusion>
        <exclusion>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
        </exclusion>                                
    </exclusions>
</dependency>
```

###### 2. 스프링 Bean 으로 등록
```xml
<!-- XML로 설정된 설정파일들을 SET해준다. -->
<beans:bean id="configuration" class="org.apache.commons.configuration.CompositeConfiguration">
    <beans:constructor-arg>
        <beans:list>
            <beans:bean class="org.apache.commons.configuration.XMLConfiguration">
                <beans:constructor-arg type="java.lang.String" value="config/config-customize.xml" />
            </beans:bean>
            <beans:bean class="org.apache.commons.configuration.XMLConfiguration">
                <beans:constructor-arg type="java.lang.String" value="config/config-default.xml" />
            </beans:bean>
            <beans:bean class="org.apache.commons.configuration.XMLConfiguration">
               <beans:constructor-arg type="java.lang.String" value="config/config-jdbc.xml" />
            </beans:bean>
        </beans:list>
    </beans:constructor-arg>
</beans:bean>

<!-- 다른 설정에서 사용할 수 있도록 Placeholder로 등록한다. -->
<beans:bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <beans:property name="properties">
        <beans:bean class="org.springmodules.commons.configuration.CommonsConfigurationFactoryBean">
            <beans:property name="configurations">
                <beans:list>
                    <beans:ref bean="configuration"/>
                </beans:list>
            </beans:property>
        </beans:bean>
    </beans:property>
</beans:bean>
```
