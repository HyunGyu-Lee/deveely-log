---
layout: post
title:  "Spring Boot 설정파일"
date:   2019-01-09 01:00:19
category: backend-spring-framework
tags: [Spring Boot,Spring Framework,Java]
comments: true
draft: false
---
Spring Boot의 큰 장점 중 하나는 기존 Spring을 이용하면서 해야했던 설정들 (XML방신이던, Java방식이던)을 자동으로 진행해주는 것이라 생각합니다. (AutoConfiguration)

이를 통해 개발자는 설정파일 (application.property, yml 파일 등)에 간단한 설정정보들만 입력해주는 수준으로도 Spring의 다양한 기능들을 쉽게 이용할 수 있습니다.

이렇게 설정할 수 있는 설정의 종류는 굉장히 다양하며 예를들어 Datasource 설정의 경우에만도 아래와 같이 굉장히 많은 옵션들을 설정할 수 있습니다.

<!--more-->

```sh
# DATASOURCE (DataSourceAutoConfiguration & DataSourceProperties)
spring.datasource.continue-on-error=false # Whether to stop if an error occurs while initializing the database.
spring.datasource.data= # Data (DML) script resource references.
spring.datasource.data-username= # Username of the database to execute DML scripts (if different).
spring.datasource.data-password= # Password of the database to execute DML scripts (if different).
spring.datasource.dbcp2.*= # Commons DBCP2 specific settings
spring.datasource.driver-class-name= # Fully qualified name of the JDBC driver. Auto-detected based on the URL by default.
spring.datasource.generate-unique-name=false # Whether to generate a random datasource name.
spring.datasource.hikari.*= # Hikari specific settings
spring.datasource.initialization-mode=embedded # Initialize the datasource with available DDL and DML scripts.
spring.datasource.jmx-enabled=false # Whether to enable JMX support (if provided by the underlying pool).
spring.datasource.jndi-name= # JNDI location of the datasource. Class, url, username & password are ignored when set.
spring.datasource.name= # Name of the datasource. Default to "testdb" when using an embedded database.
spring.datasource.password= # Login password of the database.
spring.datasource.platform=all # Platform to use in the DDL or DML scripts (such as schema-${platform}.sql or data-${platform}.sql).
spring.datasource.schema= # Schema (DDL) script resource references.
spring.datasource.schema-username= # Username of the database to execute DDL scripts (if different).
spring.datasource.schema-password= # Password of the database to execute DDL scripts (if different).
spring.datasource.separator=; # Statement separator in SQL initialization scripts.
spring.datasource.sql-script-encoding= # SQL scripts encoding.
spring.datasource.tomcat.*= # Tomcat datasource specific settings
spring.datasource.type= # Fully qualified name of the connection pool implementation to use. By default, it is auto-detected from the classpath.
spring.datasource.url= # JDBC URL of the database.
spring.datasource.username= # Login username of the database.
spring.datasource.xa.data-source-class-name= # XA datasource fully qualified name.
spring.datasource.xa.properties= # Properties to pass to the XA data source.
```

[Spring 공식 홈페이지](https://docs.spring.io)에서 다양한 설정들을 가이드로 제공하고 있습니다.

위 가이드 설정파일에 주석으로도 적혀있고 당연한 얘기지만 위 설정파일의 전체내용을 복사하지 말라고 명시되어있습니다.

본인이 사용하는 기술에 대해 필요한 설정 위주로 찾아서 추가해주면 보다 더 좋은 설정을 진행할 수 있을 것 같습니다.
