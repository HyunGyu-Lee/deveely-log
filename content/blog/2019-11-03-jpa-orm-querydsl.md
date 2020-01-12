---
layout: post
title: "Java ORM JPA과 객체지향 쿼리 사용(JPQL, Criteria, QueryDSL)"
date:   2019-11-03 11:43:19
category: backend-JPA
tags: [Java,JPA]
comments: true
draft: false
---
Java ORM 표준인 JPA 에 대해 설명하고 객체지향 쿼리 사용과 장단점을 정리했습니다.
<!--more-->

## 개요
예전부터 객체지향 언어인 자바와 관계형 데이터베이스간 패러다임의 불일치로 개발자들이 많은 불편함을 겪여왔다.

- 객체지향 추상화, 캡슐화, 정보은닉, 상속, 다형성 등 시스템의 복잡성 줄여주는 장치들을 제공
- 관계형 데이터베이스는 데이터 중심으로 구조화, 집합적인 사고가 필요하고 추상화, 상속, 다형성 같은 개념이 없음
- 연관관계를 표현할 때 객체는 타 객체 참조 (reference) 로, 관계형 DB 는 외래키로 표현하는 등 연관관계를 표현 
    - 객체를 테이블에 맞춰 모델링

    ```java
    class Member {
        String id;      // MEMBER_ID 컬럼 사용
        Long teamId;    // TEAM_ID FK 컬럼 사용
        String userName;
    }

    class Team {
        Long id;        // TEAM_ID PK 사용
        String name;
    }
    ```

    - 객체지향 모델링 
    
    ```java
    class Member {
        String id;
        Team team;          // 참조로 연관관계를 맺는다.
        String username;

        Team getTeam() {
            return team;
        }
    }

    class Team {
        Long id;
        String name;
    }
    ```

실제 구현해야할 비즈니스 로직 이외에 패러다임 불일치로 인해 개발자의 불필요한 수고가 많이 발생


## ORM? JPA? Hibernate?

##### JPA
- **JPA**란 **Java Persistence API**의 약자로 자바 진영의 표준 **ORM** 기술
- 자바에서 ORM 기술을 이용하기 위한 표준 API 명세와 인터페이스의 집합

##### ORM
- **ORM**이란 **Object Relational Mapping**의 약자로 **객체와 관계형 데이터베이스를 매핑하는 기술**을 의미

##### Hibernate
- **Hibernate**는 **JPA**를 구현한 ORM 프레임워크

즉 **JPA**는 <u>자바의 객체와 관계형 데이터베이스를 매핑하는 표준 기술</u> 이다.
**Hibernate**는 가장 인기있는 자바 ORM 프레임워크이다.

## 왜 사용하는가?
- **ORM**은 **객체와 관계형 데이터베이스 패러다임의 불일치**를 개발자 대신 해결해준다.
    - 상속, 연관관계, 객체 그래프 탐색, 비교 등 관계형 데이터베이스와의 패러다임 불일치로 어려워지는 문제들을 해결
    - 개발자는 비즈니스 로직에 집중할 수 있게됨
- **ORM** 상에서는 개발자가 직접 SQL문을 작성할 필요가 없다. -> 필요한 SQL을 개발자 대신 해줌으로써 지루하고 반복적인 CRUD용 SQL문을 개발자가 작성하지 않아도 된다.

## 어떻게 사용하는가?
**JPA** 에서는 SQL을 추상화한 **JPQL** 이라는 객체지향 쿼리언어를 사용한다.
**SQL**이 데이터베이스 테이블을 대상으로 사용하는 질의문이라면 **JPQL**은 객체를 대상으로 사용하는 질의문이다.

- Native SQL
```sql
select *
  from member
 where member_nm like 'lee%';
```

- JPQL

```java
// Entity (자바객체와 데이터베이스 테이블을 매핑)
@Entity
@Table(name = "MEMBER")
public class EntityMember {

    @Column("member_nm")
    private String memberName;
    ....
}


// Repository 
public interface MemberRepository JpaRepository<Memger, Long> {

    // 메소드 이름 쿼리
    List<Member> findAllByMemberNameLike(String searchName);
    
    // JPQL 직접 작성
    @Query(" select m from Member m" +
            " where m.memberName like :searchName")
    List<Member> findAllByMemberNameLikeDirectJPQL(String searchName);
}

```

위 처럼 개발자는 JPQL 을 작성하면, 프레임워크가 애플리케이션에서 사용하는 DBMS 에 맞는 SQL로 변환하여 실행시켜준다. (DB 벤더에 독립적)

##### JPQL의 단점
**JPQL**이 SQL에 비해 좋은 부분이 많으나, 결국 문자열로 적는 쿼리로 한계와 단점들이 존재한다.

- 타입 안정성을 보장 받을 수 없음
- 동적인 쿼리를 작성할 수 없음

이를 위해 **Criteria** 라는 빌더 API 를 지원하여 자바코드로 JPQL 작성을 지원한다.

- JPQL
```sql
select m from Member m
```

- Criteria 사용
```java
public List<Member> findAll() {
    CriteriaBuilder cb = em.getCriteriaBuilder();

    //Criteria 생성, 반환 타입 지정
    CriteriaQuery<Member> cq = cb.createQuery(Member.class);

    Root<Member> m = cq.from(Member.class); // FROM 절
    cq.select(m);   // SELECT 절

    TypedQuery<Member> query = em.createQuery(cq);
    return query.getResultList();
}
```

**Criteria** 는 문자열로 작성하는 **JPQL**의 한계를 일부 극복해주지만, 코드가 너무 복잡하고 직관적이지 못해 가독성이 떨어지고, 어떤 JPQL 문이 생성될 지 예측하기 어렵다.

- Native SQL
```sql
  select a.id_seq,
         b.user_id,
         ...
    from msg_friendly_match_invitation_message a inner join msg_rat b 
      on a.inviter_user_id = b.user_id 
   where a.my_user_id = ? 
     and a.expire_date > ? 
order by a.id_seq desc
```

- Criteria 사용하여 구현
```java
public List<EntityFriendlyMatchInvitation> findFriendlyMatchInvitations(String sno, LocalDateTime expireTimeLimit) {
    CriteriaBuilder builder = entityManager.getCriteriaBuilder();
    CriteriaQuery<EntityFriendlyMatchInvitation> query = builder.createQuery(EntityFriendlyMatchInvitation.class);        

    // FROM 절 (조인)
    Root<EntityFriendlyMatchInvitation> invitation = query.from(EntityFriendlyMatchInvitation.class);
    invitation.fetch("inviter", JoinType.INNER);

    // 조건절
    Predicate condition = builder.and(
        builder.equal(invitation.get("myUserId"), sno),	
        builder.greaterThan(invitation.get("expireDate"), expireTimeLimit)	
    );

    // SELECT 절
    query.select(invitation)
          .where(condition)
        .orderBy(builder.desc(invitation.get("id")));

    TypedQuery<EntityFriendlyMatchInvitation> typedQuery = entityManager.createQuery(query);
    return typedQuery.getResultList();
}
```

## QueryDSL
QueryDSL 은 이런 Criteria 의 단점을 극복해주는 JPQL 빌더 API 이다. 
Criteria 에 비해 훨씬 간결하고 코드가 JPQL 과 비슷하여 직관적이며 어떤 JPQL이 실행될 지 보다 쉽게 예측 가능하다. 

```java
// 조회에 사용할 객체 (Q 도메인)
private QEntityFriendlyMatchInvitation invitation = QEntityFriendlyMatchInvitation.entityFriendlyMatchInvitation;
private QEntityRat member = QEntityRat.entityRat;
    
public List<EntityFriendlyMatchInvitation> findFriendlyMatchInvitations(String sno, LocalDateTime expireTimeLimit) {
    return from(invitation)
        .innerJoin(invitation.inviter, member)
        .fetchJoin()
        .where(invitation.myUserId.eq(sno)
          .and(invitation.expireDate.after(expireTimeLimit)))
        .orderBy(invitation.id.desc())
        .fetch();
}
```
Entity 기반으로 자동생성된 Q도메인을 사용하여 기존 문자열로 작성하던 부분을 대체할 수 있고 타입 안정성이 보장된다.
Q도메인은 필요한데 별도 컴파일러 플러그인를 등록하면, 프로젝트 컴파일 시 생성된다.


```java
// Entity 모델 예제
@Entity
@Table(name = "msg_friendly_match_invitation_message")
public class EntityFriendlyMatchInvitation {
	@Id
	@GeneratedValue(strategy= GenerationType.IDENTITY)
	@Column(name = "id_seq")
	private Long id;

	@Column(name = "my_user_id")
	private String myUserId;

	@ManyToOne
	@JoinColumn(name = "inviter_user_id")
	private EntityRat inviter;

	@Column(name = "game_type")
	private Integer gameType;

	@Column(name = "seed_money")
	private Long seedMoney;

	@Column(name = "room_number")
	private Integer roomNumber;

	@Column(name = "room_key")
	private Long roomKey;

	@Column(name = "expire_date")
	private LocalDateTime expireDate;

	getters, setters...
}

// 자동 생성된 Q도메인
package com.nhnent.msg.entity;

import static com.querydsl.core.types.PathMetadataFactory.*;

import com.querydsl.core.types.dsl.*;

import com.querydsl.core.types.PathMetadata;
import javax.annotation.Generated;
import com.querydsl.core.types.Path;
import com.querydsl.core.types.dsl.PathInits;


/**
 * QEntityFriendlyMatchInvitation is a Querydsl query type for EntityFriendlyMatchInvitation
 */
@Generated("com.querydsl.codegen.EntitySerializer")
public class QEntityFriendlyMatchInvitation extends EntityPathBase<EntityFriendlyMatchInvitation> {

    private static final long serialVersionUID = 1449367612L;
    private static final PathInits INITS = PathInits.DIRECT2;
    public static final QEntityFriendlyMatchInvitation entityFriendlyMatchInvitation = new QEntityFriendlyMatchInvitation("entityFriendlyMatchInvitation");
    public final DateTimePath<java.time.LocalDateTime> expireDate = createDateTime("expireDate", java.time.LocalDateTime.class);

    // Java Entity 의 필드의 타입을 기반으로 자동 생성 되어 쿼리에서 사용 시 Type safe 보장
    public final NumberPath<Integer> gameType = createNumber("gameType", Integer.class);
    public final NumberPath<Long> id = createNumber("id", Long.class);
    public final QEntityRat inviter;
    public final StringPath myUserId = createString("myUserId");
    public final NumberPath<Long> roomKey = createNumber("roomKey", Long.class);
    public final NumberPath<Integer> roomNumber = createNumber("roomNumber", Integer.class);
    public final NumberPath<Long> seedMoney = createNumber("seedMoney", Long.class);

    ...

}
```
