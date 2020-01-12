---
layout: post
title: "HikariCP Connection Pool 해제 이슈"
date:   2020-01-11 12:43:19
category: backend-spring-framework
tags: [Connection Pool,HikariCP]
comments: true
draft: false
---
몇 일 전, 개발환경에 구축되어있는 백엔드 시스템의 커넥션풀을 변경해야하는 일이 생겨 기존 Tomcat 커넥션풀 (tomcat-dbcp)에서 HikariCP로 변경하였습니다.
변경하면서 기존 설정되어있던 수치들을 HikariCP에서 제공하는 옵션에 맞추어 마이그레이션 했는데요. 완료 후 특별한 문제점은 보이지 않았습니다.
하지만 다음 날 출근 후.... 해당 시스템의 API를 호출하자 갑자기 아래와 같은 오류 메시지가 나오며 시스템이 정상적으로 동작하지 않는 상태인 것이 확인되었습니다.
<!--more-->

```
[2020-01-10 08:49:08,143 +0900] [WARN] [c.z.h.p.ProxyConnection] GAMEDB-POOL - Connection com.mysql.jdbc.JDBC4Connection@158c355 marked as broken because of SQLSTATE(08S01), ErrorCode(0)
com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: The last packet successfully received from the server was 36,305,998 milliseconds ago.  The last packet sent successfully to the server was 36,305,999 milliseconds ago. is longer than the server configured value of 'wait_timeout'. You should consider either expiring and/or testing connection validity before use in your application, increasing the server configured values for client timeouts, or using the Connector/J connection property 'autoReconnect=true' to avoid this problem.
        at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
        at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
        at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
        at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
        at com.mysql.jdbc.Util.handleNewInstance(Util.java:425)
        at com.mysql.jdbc.SQLError.createCommunicationsException(SQLError.java:989)
        at com.mysql.jdbc.MysqlIO.send(MysqlIO.java:3746)
        at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:2509)
        at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2680)
        at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2487)
        at com.mysql.jdbc.PreparedStatement.executeInternal(PreparedStatement.java:1858)
        at com.mysql.jdbc.PreparedStatement.executeQuery(PreparedStatement.java:1966)
        at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeQuery(ProxyPreparedStatement.java:52)
        at com.zaxxer.hikari.pool.HikariProxyPreparedStatement.executeQuery(HikariProxyPreparedStatement.java)
Caused by: java.net.SocketException: 파이프가 깨어짐 (Write failed)
        at java.net.SocketOutputStream.socketWrite0(Native Method)
        at java.net.SocketOutputStream.socketWrite(SocketOutputStream.java:111)
        at java.net.SocketOutputStream.write(SocketOutputStream.java:155)
        at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:82)
        at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:140)
        at com.mysql.jdbc.MysqlIO.send(MysqlIO.java:3728)        
```

## 원인
리서치를 해본 결과 핵심은 기존 사용하던 tomcat-dbcp와 HikariCP의 철학이 달라서 생긴 문제였습니다.
현재 백엔드 시스템은 MySQL을 사용중인데요. MySQL은 맺어진 커넥션이 장시간 사용되지 않으면 해당 커넥션을 해제하도록 설계되어있습니다. (기본 8시간)

#### tomcat-jdbc
`test-while-idle` 설정이 `true`로 되어있으면 IDLE 상태의 커넥션들을 대상으로 DB서버에 SELECT 1을 주기적으로 보냅니다.
이를 통해 커넥션이 지속적으로 갱신되고 서버로부터 disconnect되지 않습니다.
```
spring.datasource.tomcat.test-while-idle=true
spring.datasource.tomcat.validation-query=SELECT 1
```

#### HikariCP
공식 문서: https://github.com/brettwooldridge/HikariCP
참고로 Spring Boot 2.0을 기점으로 기본 DBCP가 Tomcat DBCP에서 HikariCP로 바뀌었습니다.
HikariCP는 tomcat-dbcp와 달리 사용하지 않는 Connection을 회수하도록 설계되었습니다.
`maxLifeTime`으로 설정된 시간이 지나면 풀에서 제거되고, 다시 생성되는 방이며 `connectionTestQuery`는 커넥션을 최초 맺을때, 풀에서 가져올 때 사용되는 옵션이라고 합니다.
또한 JDBC4를 지원하는 환경에서는 `connectionTestQuery` 를 명시하지 않는것을 권고하고 있습니다. 
```
[Do not use JDBC4]
hikaricp.opts.connectionTestQuery=SELECT 1
```

HikariCP 내부 JDBC 구현체에서는 아래와 같이 JDBC4에 대한 구현을 하고 있습니다.
```java
PoolBase.java

boolean isConnectionAlive(final Connection connection) {
  ...
  if (isUseJdbc4Validation) {
      return connection.isValid(validationSeconds);
  }
  ...
}

/**
* Execute isValid() or connection test query.
*
* @param connection a Connection to check
*/
private void checkDriverSupport(final Connection connection) throws SQLException {
    ...
    if (isUseJdbc4Validation) {
        connection.isValid(1);
    }
    ...
}
```

## 대처
우선은 `maxLifeTime` 값을 설정하고, `connectionTestQuery` 설정을 제거한 후 모니터링을 해보고자 합니다.

실제 운영환경에서는 발생하지 않았을 오류이지만, 오히려 개발환경인 덕에 발생하고, HikariCP에 대해 상세하게 리서치해 볼 수 있는 기회였습니다.
문제가 해결되면 해결된 글로 다시 작성해야겠습니다.