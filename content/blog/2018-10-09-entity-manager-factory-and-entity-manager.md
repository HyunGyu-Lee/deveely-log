---
layout: post
title:  "EntityManagerFactory와 EntityManager"
date:   2018-10-09 16:21:28
category: backend-JPA
tags: [JPA,Hibernate]
comments: true
draft: false
---
##### EntityManagerFactory (엔티티매니저팩토리)
일반적으로 데이터베이스를 하나만 사용하는 애플리케이션은 EntityManagerFactory를 하나만 생성한다.  
META/INF/persistence.xml 에 설정한 정보를 기반으로 다음과 같은 코드로 생성할 수 있다.
<!--more-->
```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory(설정한이름);
```

EntityManagerFactory는 EntityManager를 생성하는 Factory클래스로 최초 생성 시 커넥션풀 생성을 포함한 여러 작업들을 수행하기때문에 생성비용이 비싸고, 때문에 애플리케이션 전반적으로 하나의 인스턴스를 공유하도록 설계되어있다.  

##### EntityManager (엔티티매니저)
엔티티를 저장 / 수정 / 삭제 / 조회 하는 등 엔티티 관련된 모든 일을 처리할 때 직접적으로 사용되는 클래스이다.  
JPA 를 사용하여 개발할 땐 바로 이 EntityManager를 데이터베이스라고 생각하고 처리하면 된다.  

EntityManager는 EntityManagerFactory를 통해서 다음과 같은 코드로 생성할 수 있다.

```java
EntityManager em = emf.createEntityManager();
```

EntityManager는 DB 연결이 필요한 시점에 커넥션풀에서 커넥션을 취득하는데, 이 때문에 스레드간 절대 공유되지 않도록 개발 시 유의해야한다.
