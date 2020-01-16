---
layout: post
title: "HikariCP와 커넥션 누수(Connection Leak) 관련 트러블슈팅"
date:   2020-01-16 12:43:19
category: backend-spring-framework
tags: [Connection Pool,HikariCP]
comments: true
draft: false
---
# 문제 발생
운영중인 서비스에서 사용하는 DB에서 특정 테이블들을 분리하여 별도 DB로 구축하는 일이 생겼습니다. 때문에 이를 위해 Multi Datasource를 적용하였습니다.
기존 서비스는 `Tomcat connection pool`이 적용되어있었는데, 이번 작업을 하며 `HikariCP`로 변경하였습니다.
(참고로 Spring Boot 2.0 부터는 HikariCP가 기본 커넥션풀이라고 합니다.)

> In Spring Boot 1.x, Tomcat connection pool was the default connection pool but in Spring Boot 2.x HikariCP is the default connection pool.

적용을 완료하고 정상동작을 확인한 후 개발환경에 반영해둔 다음날.. 
<br>
서비스 API 호출 시 오류가 발생한다는 문의를 받고 로그를 확인해보니 아래와 같은 에러 발생 및 API 호출 불가한 현상이 확인되었습니다.

```java
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

...        

Caused by: java.net.SocketException: 파이프가 깨어짐 (Write failed)
        at java.net.SocketOutputStream.socketWrite0(Native Method)
        at java.net.SocketOutputStream.socketWrite(SocketOutputStream.java:111)
        at java.net.SocketOutputStream.write(SocketOutputStream.java:155)
        at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:82)
        at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:140)
        at com.mysql.jdbc.MysqlIO.send(MysqlIO.java:3728)
        
```

# 원인
해당 서비스가 사용하는 DBMS는 MySQL입니다.
MySQL은 기본적으로 자신에게 맺어진 커넥션 중 **일정 시간이상 사용하지 않은 커넥션을 종료**하는 프로세스가 존재합니다. (MySQL의 `wait_timeout`이라는 값을 통해 확인할 수 있고 default 8시간)

이를 위해 기존 커넥션풀은 대부분 연결을 맺은 커넥션들이 끊기는 것을 방지하기 위해 `SELECT 1` 등의 쿼리를 주기적으로 날려 이 문제를 회피하는 반면 HikariCP는 `maxLifetime` 설정값에 따라 스스로 미사용된 커넥션을 제거하고 새로 생성하는 방식으로 동작한다고 합니다.

이번에 문제는 맺어졌던 커넥션이 장시간 사용되지 않아 MySQL 서버에 의해 끊어졌고 커넥션 풀은 끊어진 커넥션인줄 모르고 계속 사용되어 발생하였습니다.

### Q. 커넥션이 장시간 사용되지 않더라도 HikariCP의 `maxLifeTime`이 MySQL의 `wait_timeout`보다 짧으면 문제될 것이 없지 않나?
위에서 언급한 것처럼 HikariCP는 `maxLifetime`에 도달한 커넥션의 연결을 끊고 새로운 커넥션을 생성하는 방식입니다. 때문에 `maxLifetime`이 `wait_timeout`보다 짧으면 MySQL 서버가 끊기 전에 스스로 연결을 끊고 새로 커넥션을 맺기 때문에 이런 문제가 발생할 여지가 없습니다.  (`maxLifetime` 기본값은 30분)

하지만 그럼에도 밤새도록 종료되지 않는 커넥션이 있었던 것은 작성된 `QueryDSL`을 적용한 Repository에서 커넥션 누수가 있었기 때문입니다. 
`maxLifetime`은 **사용하지 않는** 커넥션 한정으로 적용됩니다. 커넥션 누수로 인해 특정 커넥션이 close되지 않았고, HikariCP입장에서는 계속 `active` 상태로 인지하여 `maxLifetime` 적용되지 않았던 것입니다. 

# 문제를 해결하기 위해 시도해본 것들
사실 문제 발생 초기엔 HikariCP의 동작방식에 대한 정확한 이해가 없었고, 때문에 커넥션 누수가 있을것이라고 예상치 못했습니다.
<br>
대신 설정값에 문제가 있을 것이라고 추측했고 삽질의 길이 시작되었습니다....

### 0. HikariCP 디버깅 로그 출력 설정
우선 더 정확히 현상을 파악하기 위해 logback 설정에 HikariCP에서 출력하는 디버그용 로그를 보기 위해 아래 설정을 진행하였습니다.
```xml
...
<logger name="com.zaxxer.hikari.HikariConfig" level="DEBUG"/>
<logger name="com.zaxxer.hikari" level="TRACE"/>
...
```

### 1. 설정값 (수치)에 따른 문제?
해당 서비스의 커넥션풀을 변경하면서 풀사이즈, 최소 유휴 상태 커넥션 수 등 풀 동작과 관련된 설정값을 기존 커넥션풀에 적용되어있는 수치 그대로 적용했는데요. 
이것이 문제가 되는 것으로 알고 HikariCP를 사용하면서 문제가 없었던 타 서비스들과 동일한 설정을 사용하도록 하였습니다.

설정을 변경한 이후에도 문제가 발생했는데 그 이유는 아래와 같았습니다.
 - 타 서비스들은 주기적으로 API 클라이언트 혹은 자체 스케쥴에 의해 db 호출을 하고 있었던 상황
 - 문제가 발생한 서비스는 야간 시간대에 8시간 이상 아무런 호출이 없었음

### 2. maxLifetime 조정
HikariCP 공식 문서에 보면 `maxLifetime`값을 DB의 `wait_timeout`보다 **몇 초 정도** 짧게 설정하라고 권장하는 가이드가 있어 적용해봤습니다.
```
We strongly recommend setting this value, and it should be several seconds shorter than any database or infrastructure imposed connection time limit. 
A value of 0 indicates no maximum lifetime (infinite lifetime), subject of course to the idleTimeout setting. 

Default: 1800000 (30 minutes)
```

하지만 여전히 문제는 발생하였습니다.

### 3. autoreconnect=true
로그를 살펴보던 중, `No operations allowed after connection closed.` 라는 오류 메시지를 확인하게 됐습니다.
이를 보고 JDBC URL에 `autoreconnect`옵션을 줬지만, 효과가 없었습니다. 심지어 HikariCP 기본 작동 방식을 봤을때 넣는게 무의미한 설정이라고 합니다.
*  혼란을 주는 에러메시지
    *  아래 에러메시지도 (HikariCP라면) 믿으면 안됩니다. 
    *  에러메시지: No operations allowed after connection closed.

### 4. minimumIdle 조정
`idleTimeout`은 생성된 커넥션이 유휴 상태로 풀에 존재할 수 있는 최대시간입니다.
이 설정은 `minimumIdle` < `maximumPoolSize` 일 경우에만 적용되는 값으로 이를 테스트하기 위해 값을 조정봤습니다. (역시 문제는 해결되지않음)

### 5. HikariCP와 Hibernate 의 버전 차이
지푸라기를 잡는 심정으로 버전을 확인해봤습니다. 기존 hikariCP, hibernate 두 버전의 차이가 4년 가량 있었습니다.
때문에 버전 차이에서 오는 알려지지 않은 버그일까 라는 생각에 버전업을 진행했지만 마찬가지로 문제가 해결되지 않았습니다.

```
[Before]
hibernate.version: 5.0.12.Final
hikaricp.version: 3.1.0

[After]
hibernate.version: 5.4.10.Final
hikaricp.version: 3.4.1
```

## 6. Connection 누수 현상 확인 (실제 원인)
또 다시 오류 메시지를 보던 중 문제가 된 Connection인 `com.mysql.jdbc.JDBC4Connection@3bfbf166`이 커넥션 풀에 등록은 되었으나 해제되지 않은 부분을 발견했습니다.

```
[2020-01-14 08:21:01,071 +0900] [WARN] [c.z.h.p.ProxyConnection] GAMEDB-POOL - Connection com.mysql.jdbc.JDBC4Connection@3bfbf166 marked as broken because of SQLSTATE(08S01), ErrorCode(0)
com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: The last packet successfully received from the server was 52,889,749 milliseconds ago.  The last packet sent successfully to the server was 52,889,750 milliseconds ago. is longer than the server configured value of 'wait_timeout'. You should consider either expiring and/or testing connection validity before use in your application, increasing the server configured values for client timeouts, or using the Connector/J connection property 'autoReconnect=true' to avoid this problem.
...
```

실제로 Connection 생성 / 해제 로그를 추출해 확인해본 결과 
<br>
`maxLifetime`에 따라 생성, 해제를 수행한 내역에서 위 문제가 된 커넥션이 Add는 되었지만 Close가 된 기록은 확인할 수 없었습니다.

```
[2020-01-13 16:09:29,349 +0900] [DEBUG] [c.z.h.p.HikariPool] GAMEDB-POOL - Added connection com.mysql.jdbc.JDBC4Connection@3bfbf166
[2020-01-13 16:09:29,353 +0900] [DEBUG] [c.z.h.p.HikariPool] GAMEDB-POOL - Added connection com.mysql.jdbc.JDBC4Connection@17d7631
[2020-01-13 16:09:29,359 +0900] [DEBUG] [c.z.h.p.HikariPool] GAMEDB-POOL - Added connection com.mysql.jdbc.JDBC4Connection@161f17c2
[2020-01-13 16:34:22,082 +0900] [DEBUG] [c.z.h.p.HikariPool] GAMEDB-POOL - Added connection com.mysql.jdbc.JDBC4Connection@60cc1c45
[2020-01-13 16:34:22,089 +0900] [DEBUG] [c.z.h.p.HikariPool] GAMEDB-POOL - Added connection com.mysql.jdbc.JDBC4Connection@7e3ef6ea
[2020-01-13 16:34:22,492 +0900] [DEBUG] [c.z.h.p.HikariPool] GAMEDB-POOL - Added connection com.mysql.jdbc.JDBC4Connection@6befd804
[2020-01-13 16:35:17,046 +0900] [DEBUG] [c.z.h.p.HikariPool] GAMEDB-POOL - Added connection com.mysql.jdbc.JDBC4Connection@273141d1
[2020-01-13 16:35:17,103 +0900] [DEBUG] [c.z.h.p.HikariPool] GAMEDB-POOL - Added connection com.mysql.jdbc.JDBC4Connection@3eac312e
[2020-01-13 16:35:17,208 +0900] [DEBUG] [c.z.h.p.HikariPool] GAMEDB-POOL - Added connection com.mysql.jdbc.JDBC4Connection@66d58bc3
[2020-01-13 16:35:17,278 +0900] [DEBUG] [c.z.h.p.HikariPool] GAMEDB-POOL - Added connection com.mysql.jdbc.JDBC4Connection@7d94ee53
[2020-01-13 16:55:35,178 +0900] [DEBUG] [c.z.h.p.PoolBase] GAMEDB-POOL - Closing connection com.mysql.jdbc.JDBC4Connection@161f17c2: (connection has passed maxLifetime)
[2020-01-13 16:55:43,855 +0900] [DEBUG] [c.z.h.p.PoolBase] GAMEDB-POOL - Closing connection com.mysql.jdbc.JDBC4Connection@17d7631: (connection has passed maxLifetime)
[2020-01-13 17:20:29,614 +0900] [DEBUG] [c.z.h.p.PoolBase] GAMEDB-POOL - Closing connection com.mysql.jdbc.JDBC4Connection@60cc1c45: (connection has passed maxLifetime)
[2020-01-13 17:20:47,468 +0900] [DEBUG] [c.z.h.p.PoolBase] GAMEDB-POOL - Closing connection com.mysql.jdbc.JDBC4Connection@6befd804: (connection has passed maxLifetime)
[2020-01-13 17:20:48,773 +0900] [DEBUG] [c.z.h.p.PoolBase] GAMEDB-POOL - Closing connection com.mysql.jdbc.JDBC4Connection@7e3ef6ea: (connection has passed maxLifetime)
[2020-01-13 17:21:30,939 +0900] [DEBUG] [c.z.h.p.PoolBase] GAMEDB-POOL - Closing connection com.mysql.jdbc.JDBC4Connection@273141d1: (connection has passed maxLifetime)
[2020-01-13 17:21:47,343 +0900] [DEBUG] [c.z.h.p.PoolBase] GAMEDB-POOL - Closing connection com.mysql.jdbc.JDBC4Connection@66d58bc3: (connection has passed maxLifetime)
[2020-01-13 17:21:50,493 +0900] [DEBUG] [c.z.h.p.PoolBase] GAMEDB-POOL - Closing connection com.mysql.jdbc.JDBC4Connection@3eac312e: (connection has passed maxLifetime)
[2020-01-13 17:21:59,390 +0900] [DEBUG] [c.z.h.p.HikariPool] GAMEDB-POOL - Added connection com.mysql.jdbc.JDBC4Connection@11991843
[2020-01-13 17:21:59,394 +0900] [DEBUG] [c.z.h.p.HikariPool] GAMEDB-POOL - Added connection com.mysql.jdbc.JDBC4Connection@30201cdf
[2020-01-13 17:22:01,785 +0900] [DEBUG] [c.z.h.p.PoolBase] GAMEDB-POOL - Closing connection com.mysql.jdbc.JDBC4Connection@7d94ee53: (connection has passed maxLifetime)
[2020-01-13 17:22:29,393 +0900] [DEBUG] [c.z.h.p.HikariPool] GAMEDB-POOL - Added connection com.mysql.jdbc.JDBC4Connection@35653c28
```

이제서야 커넥션 누수가 있음을 알고, HikariCP에서 제공하는 `leakDetectionThreshold` 옵션을 설정해 로컬에서 확인해본 결과 `QueryDSL` 이 적용되어있는 특정 API 부분에서 커넥션 누수가 확인되었습니다.

#### 커넥션 누수가 발생한 원인
아래 2가지 이유가 복합적으로 연관되어 문제가 발생하였습니다.

1. EntityManager를 Bean으로 설정함
    - EntityManager는 스레드간 절대 공유하면 안됨
2. QueryDSL 구현 Resitory에서 특정 datasource의 EntityManager를 받기 위해 setEntityManager를 재정의하지 않고 별도 setting 메소드 작성

### JPA를 사용할 때 보통 EntityManager를 클래스 멤버변수로 두고 `@PersistenceContext`로 주입받아 사용하는데 이것은 어떻게 안전할까요?
바로 일반적으로 `EntityManagerFactory`를 통해 하나하나 생성되는 EntityManager가 아닌 `SharedEntityManagerBean` 라는 공유 가능하며 커넥션 생성 / 반환에 대한 관리를 지원하는 구현체가 주입되기 때문입니다.

QueryDslRepositorySupport는 아래와 같이 EntityManager를 주입받는 코드가 존재합니다.
특별한 설정이 없으면 `SharedEntityManagerBean`이 주입되어 아무 문제 없이 작동 했을것입니다.

```java
@Repository
public abstract class QueryDslRepositorySupport {
	....

	/**
	 * Setter to inject {@link EntityManager}.
	 * 
	 * @param entityManager must not be {@literal null}.
	 */
	@Autowired
	public void setEntityManager(EntityManager entityManager) {

		Assert.notNull(entityManager, "EntityManager must not be null!");
		this.querydsl = new Querydsl(entityManager, builder);
		this.entityManager = entityManager;
	}
	....
}
```

하지만 MultidataSource를 적용하는 과정에서 @Autowired 어노테이션만으로는 프레임워크가 어떤 데이터소스의 EntityManager를 넣어줘야할지 판단하지 못하기 때문에 오류가 발생했고, 이 오류를 해결한다고 아래와 같이 일반 EntityManager를 싱글톤 빈으로 잡는 잘못된 설정을 하였습니다.

```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
		entityManagerFactoryRef = "gameEntityManagerFactory",
		transactionManagerRef = ServiceTransactionManager.GAME_DATABASE_TRANSACTION_MANAGER,
		basePackages = { "com.anan.repository.game" }
)
public class GameDatabaseConfiguration {
    ...
	// 문제가 된 부분 - @Autowired 시 Persistence 기능 관련 프레임워크의 관리를 받지 못하는 EntityManager가 주입됨
	@Primary
	@Bean(name = "gameEntityManager")
	public EntityManager entityManager(@Qualifier("gameEntityManagerFactory") EntityManagerFactory entityManagerFactory) {
		return entityManagerFactory.createEntityManager();
	}
    ...
}
```

```java
public class FriendlyMatchInvitationRepositoryImpl extends QueryDslRepositorySupport implements FriendlyMatchInvitationRepositorySupport {

	private QEntityFriendlyMatchInvitation invitation = QEntityFriendlyMatchInvitation.entityFriendlyMatchInvitation;
	private QEntityRat member = QEntityRat.entityRat;

	public FriendlyMatchInvitationRepositoryImpl() {
		super(EntityFriendlyMatchInvitation.class);
	}

	// 문제가 된 부분 - EntityManager를 주입받기위해 setEntityManage 재정의하지 않음
	// @PersistenceContext에 의해 setTargetEntityManager(정상 EntityManager) -> super.setEntityManager(정상 EntityManager) 
	// 상위 메소드의 @Autowired에 의해 setEntityManager(비정상 EntityManager)로 일반 EntityManager가 사용됨
	@PersistenceContext(unitName = "game")
	public void setTargetEntityManager(EntityManager entityManager) {
		super.setEntityManager(entityManager);
	}

	@Override
	public List<EntityFriendlyMatchInvitation> findFriendlyMatchInvitations(String sno, LocalDateTime expireTimeLimit) {
		return new JPAQuery<EntityFriendlyMatchInvitation>(getEntityManager())
				.from(invitation)
				.innerJoin(invitation.inviter, member)
				.fetchJoin()
				.where(invitation.myUserId.eq(sno).and(invitation.expireDate.after(LocalDateTime.now())))
				.orderBy(invitation.id.desc())
				.fetch();
	}

}
```

때문에 해당 커넥션을 사용 후 `Connection.close`가 제대로 동작하지 않았습니다.

# 해결
해결하기 위해 우선 EntityManager를 Bean으로 설정한 부분을 제거하였습니다.
두 번째로, MultidataSource를 적용하는 과정에서 @Autowired 어노테이션만으로는 프레임워크가 어떤 데이터소스의 EntityManager를 넣어줘야할지 판단하지 못하는 오류를 해결하기 위해 @PersistenceContext + unitName attribute 지정하는 방식으로 setEntityManager를 재정의하였습니다.

```java
public class FriendlyMatchInvitationRepositoryImpl extends QueryDslRepositorySupport implements FriendlyMatchInvitationRepositorySupport {

	private QEntityFriendlyMatchInvitation invitation = QEntityFriendlyMatchInvitation.entityFriendlyMatchInvitation;
	private QEntityRat member = QEntityRat.entityRat;

	public FriendlyMatchInvitationRepositoryImpl() {
		super(EntityFriendlyMatchInvitation.class);
	}
    
	@Override
	@PersistenceContext(unitName = "game")
	public void setEntityManager(EntityManager entityManager) {
		super.setEntityManager(entityManager);
	}

	@Override
	public List<EntityFriendlyMatchInvitation> findFriendlyMatchInvitations(String sno, LocalDateTime expireTimeLimit) {
		return new JPAQuery<EntityFriendlyMatchInvitation>(getEntityManager())
				.from(invitation)
				.innerJoin(invitation.inviter, member)
				.fetchJoin()
				.where(invitation.myUserId.eq(sno).and(invitation.expireDate.after(LocalDateTime.now())))
				.orderBy(invitation.id.desc())
				.fetch();
	}

}
```

# 마치며
기본이 중요하다는 것을 다시 느꼈습니다. 분명 JPA를 공부하며 책에서 EntityManager는 스레드간 공유하면 안되는 것이라는 내용을 본 적이 있음에도 불고하고
실수를 하여 문제가 발생했고 해결하기 위해 수많은 삽질을 하였습니다. 

두번째는 자신이 사용하는 기술을 정확히 이해하고 사용하자는 것입니다. 이번에 겪은 이슈도 `JPA의 대한 이해`와 `HikariCP에 대한 이해` 부족으로 시간이 오래 걸렸습니다.
기술에 대한 이해가 있었다면 애초에 문제가 발생하지 않았거나 훨씬 빠르게 조치를 할 수 있었을 것입니다.

하지만 `퍼포먼스가 좋다`, `가장 인기있다`라는 것만 알고 있던 `HikariCP`에 대해 좀 더 자세히 알게 되어서 한편으로는 좋은 경험이었다는 생각이듭니다.

# 참고
HikariCP 공식 문서 : [https://github.com/brettwooldridge/HikariCP](https://github.com/brettwooldridge/HikariCP)