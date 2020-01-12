---
layout: post
title:  "JPA 기본적인 CRUD 해보기"
date:   2018-10-07 00:26:58
category: backend-JPA
tags: [JPA,Hibernate]
comments: true
draft: false
---
간단한 애플리케이션을 작성하여 기본적인 JPA CRUD를 수행해본다.  
<!--more-->
```java
public static void main(String[] args) {
    // EntityManagerFactory 생성
    EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpa");

    // EntityManager 생성
    EntityManager em = emf.createEntityManager();

    // 트랜잭션 취득
    EntityTransaction tx = em.getTransaction();

    try {
        // 트랜잭션 시작
        tx.begin();

        logic(em);

        // 커밋
        tx.commit();
    } catch (Exception e) {
        e.printStackTrace();

        // 오류 발생 시 롤백
        tx.rollback();
    } finally {
        // 로직 수행 후 EntityManager 종료
        em.close();
    }

    // 애플리케이션 종료 시 EntityManagerFactory 종료
    emf.close();
}

public static void logic(EmtityManager em) {
    String id = "test";

    Member member = new Member();
    member.setMemberId(id);
    member.setMemberName("사용자");
    member.setAge(20);

    // 등록 (C)
    em.persist(member);

    // 단 건 조회 (R)
    Member findMember = em.find(Member.class, id);
    System.out.println("findMember=" + findMember.getMemberName() + ", age=" + findMember.getAge());

    // 목록 조회 (R)
    List<member> members = em.createQuery("select m from Member m", Member.class).getResultList();
    System.out.println("member size=" + members.size());

    // 수정 (U)
    member.setAge(25);

    // 삭제 (D)
    em.remove(member);
}
```

위 코드에서 핵심은 크게 3가지로 볼 수 있다.  

1. 엔티티 매니저 설정
2. 트랜잭션 관리
3. 비즈니스 로직

##### 1. 앤티티 매니저 설정  
###### 1) EntityManagerFactory 생성 및 관리
JPA를 사용하기 위해서는 우선 persistance.xml을 통해 설정을 진행한 후 EntityManagerFactory를 생성해야한다.  
JPA는 클래스패스의 META/INF/persistance.xml파일을 설정파일로 자동인식한다.  

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory(persistance.xml에 설정된 persistence-unit 명);
```

위 코드 한 줄로 손쉽게 EntityManagerFactory를 생성할 수 있다.  

EntityManagerFactory가 생성될 때 JPA 동작을 위한 기반 객체를 생성하고, JPA 구현체에 따라 DB 커넥션 풀 생성 등 비용이 크다.  

때문에 EntityManagerFactory는 애플리케이션 전체적으로 1번만 생성되고 공유되어야한다.  

또한 애플리케이션 자체가 종료될 시 close 해줘야한다.  

###### 2) EntityManager 생성 및 관리
```java
EntityManager em = emf.createEntityManager();
```

EntityManagerFactory를 통해 EntityManager를 생성한다.  
JPA에서는 이 EmtityManager를 통해 DB에 등록/수정/삭제/조회하는 작업이 이루어진다.  
내부에서 DB 커넥션을 유지하고 있기 때문에 Thread간 공유하거나 재사용하면 안되며
한 비즈니스 로직에 이용된 후 반드시 close 해줘야한다.  

##### 2. 트랜잭션 설정
JPA에서 등록, 수정, 삭제는 반드시 트랜잭션 내에서 이루어져야한다.  
EntityManager를 통해 트랜잭션을 취득할 수 있다.


##### 3. 비즈니스 로직
개발자는 EntityManager를 가상의 데이터베이스로 취급하며 EntityManager에 CRUD를 행할수있다.

###### 1) 등록
```java
em.persist(entity);
```

JPA는 전달된 entity의 타입(class)를 분석하여 테이블과 매핑정보를 얻은 후
적절한 INSERT문을 생성 및 수행한다.

###### 2) 수정
수정의 경우 좀 특이하다.  
JPA는 엔티티의 변경을 추적하는 기능이 있다.  
위 소스에서 처럼 Entity객체의 setter를 통해 값을 변경하면 JPA가 이를 감지해 UPDATE문을 수행한다.  
em.update 와 같은 update메소드가 따로 없다.  

###### 3) 삭제
```java
em.remove(entity);
```

EntityManager에 삭제할 Entity객체를 넘겨줌으로써 엔티티를 삭제할 수 있다.

###### 4) 조회
- 단 건 조회

```java
em.find(Entity타입(class), ID);
```

EmtityManager의 find메소드를 통해 단 건 조회가 가능하다.

- 목록 조회

```java
TypedQuery<Entity타입> query = em.createQuery("JPQL", Entity타입(class);
query.getResultList();
```

JPA에서 목록 검색을 한다면 DB가 아닌 Entity를 대상으로 수행해야한다.  
하지만 DB의 모든 데이터를 읽어와 Entity로 변경한 후 검색을 한다는건 사실상 불가능하다.

JPA는 JPQL(Java Persistence Query Language)라는 쿼리언어로 이 문제를 해결한다.  

JPQL은 일반 SQL과 거의 비슷한 문법으로 SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 등을 사용할 수 있다.  

JPQL이 Entity 객체를 대상으로 쿼리한다. 즉 클래스 및 클래스의 필드를 대상으로 쿼리한다.
SQL은 데이터베이스 테이블을 대상으로 쿼리한다.  

얼핏보면 JPQL이 일반 데이터베이스에 쿼리하는 것으로 보일수 있으나 JPQL은 데이터베이스 테이블을 전혀 알지 못한다.  

위 소스에서 "select m from Member m" 이라는 JPQL이 사용됐는데 from Member의 Member는 MEMBER테이블이 아닌 Member 클래스를 의미한다.  

개발자가 작성한 JPQL을 JPA가 적절한 SQL로 변환하여 수행한다.  

---
>자바 ORM 표준 JPA 프로그래밍을 공부하며 정리한 내용입니다.
