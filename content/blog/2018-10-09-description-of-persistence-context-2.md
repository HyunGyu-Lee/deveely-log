---
layout: post
title:  "영속성 컨텍스트 (Persistence Context) 란 (2)"
date:   2018-10-09 19:58:22
category: backend-JPA
tags: [JPA,Hibernate,PersistenceContext]
comments: true
draft: false
---
플러시는 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영한다.

플러시가 호출되면  

1. 변경감지가 동작해서 영속성 컨텍스트 내 모든 엔티티들을 대상으로 스냅샷과 비교해 변경사항을 찾고, 수정된 엔티티들은 UPDATE문을 생성하여 SQL저장소에 등록한다.  
2. SQL저장소의 쿼리를 데이터베이스에 전송한다. (등록, 수정, 삭제 쿼리)  
<!--more-->
영속성 컨텍스트를 플러시하는 방법은 3가지가 있다.   

1. 직접 호출   
엔티티매니저의 flush 메소드를 호출하는 것으로 테스트, 다른 프레임워크와 JPA를 함께 사용할 때를 제외하곤 거의 사용하지 않는다.   

2. 트랜잭션 커밋 시   
데이터베이스에 어떤 변경도 없이 트랜잭션만 커밋해서는 아무것도 바뀌는게 없다.   
때문에 항상 데이터베이스 커밋 전엔 영속성 컨텍스트의 내용을 flush하여 변경 내용을 데이터베이스에 반영해야한다.   
JPA는 트랜잭션 커밋할 때 자동으로 flush메소드를 호출하도록 설계되어있다.  

3. JPQL 쿼리 실행 시  
JPQL은 SQL로 변환되어 데이터베이스에서 조회하는 기능으로 persist되었어도 데이터베이스에 반영이 안되있으면 조회되지 않는다.   
때문에 JPA는 JPQL이 실행되기전 영속성 컨텍스트를 flush하여 이런 문제를 예방한다.   

FlushModeType을 통해 엔티티매니저의 플러시모드를 변경할 수 있다.   

```java
// FlushModeType.AUTO   : default로 커밋 또는 JPQL 실행 시 flush 호출
// FlushModeType.COMMIT : 커밋할 때만 flush 호출
em.setFlushMode(FlushModeType.COMMIT);
```

---
>자바 ORM 표준 JPA 프로그래밍을 공부하며 정리한 내용입니다.
