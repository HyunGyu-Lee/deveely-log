---
layout: post
title:  "JPA 객체 연관관계 설정"
date:   2018-10-23 01:06:54
category: backend-JPA
tags: [JPA,ORM,Hibernate,객체연관관계매핑]
comments: true
draft: false
---
### @ManyToOne
다대일(N:1) 관계 매핑 정보다.

###### * 주요속성  

| 속성명 | 의미 |
|---|:---:|
| `optional` | false로 설정 시 연관된 엔티티가 항상 있어야한다 (기본값 : true) |
| `fetch` | 글로벌 Fetch 전략을 설정한다 |
| `cascade` | 영속성 전이 기능을 사용한다 |
| `targetEntity` | 연관된 엔티티의 타입 정보를 설정한다. (거의 사용안함) |
<!--more-->
### @JoinColumn
외래 키를 매핑할 때 사용한다.

###### * 주요속성  

| 속성명 | 의미 |
|---|:---:|
| `name` | 매핑할 외래키 이름 (기본값 : 필드명 + _ + 참조하는 테이블의 기본 키 컬럼명) |
| `referencedColumnName` | 외래 키가 참조하는 대상 테이블의 컬럼명 (기본값 : 참조하는 테이블의 기본 키 컬럼명) |
| `foreignKey` | 테이블을 생성할 때만 사용하는 속성으로 외래 키 제약조건을 직접 지정할 수 있다 |

단방향 관계를 매핑을 할 때 다대일(@ManyToOne) 또는 일대일(@OneToOne) 중 어떤 것을 사용할 지는 반대편 관계에 달려있다.  
반대 편이 일대다(@OneToMany)면 다대일(@ManyToOne), 반대편이 일대일(@OneToOne)이면 일대일(@OneToOne)

### 엔티티 양방향 매핑 규칙
사실 테이블에서는 하나의 외래키를 통해 두 테이블의 연관관계를 관리한다.  
하지만 객체는 단방향 관계 2개를 사용하여 두 객체간 연관관계를 관리한다.  
JPA서는 연관관계의 주인을 정함으로써 이 문제를 해결한다.  

테이블의 외래키를 관리하는 엔티티를 연관관계의 주인이라 하고 외래키를 보유한 테이블이 연관관계의 주인이 되어야한다.

주인이 아닌 엔티티는 mappedBy 속성을 이용해 주인을 지정해야한다.  

연관관계의 주인만이 외래 키에 영향을 줄 수 있다. 때문에 주인이 아닌 엔티티를 통해 add등을 수행해도 INSERT 되지 않는다.  

오직 연관관계의 주인만이 연관관계 설정에 영향을 줄 수 있다.  

하지만 순수 객체의 상태에서는 이야기가 다르다.  

주인이 아닌 측에서 외래키에 영향을 줄 수 없다고 관계를 맺어두지 않으면 DB에는 잘 들어갈 지라도 객체상태에서는 사용이 불가능한 경우가 있다.  

때문에 양방향 관계가 설정된 엔티티들을 사용할땐 양 쪽 둘다 연관관계를 맺어주자.  

Team (팀) / Member (회원)  

```java
Team team = new Team();
team.setTeamName("T1");

entityManager.persist(team);

Member m = new Member();
m.setMemberName("M1");
m.setTeam(team);	// 회원 -> 팀 연관관계 설정

entityManager.persist(m);
```

위 코드에서는 회원 -> 팀으로의 방향으로만 연관관계를 설정했다.  

하지만  
```java
Team team = new Team();
team.setTeamName("T1");

entityManager.persist(team);

Member m = new Member();
m.setMemberName("M1");
m.setTeam(team);          // 회원 -> 팀 연관관계 설정
team.getMembers().add(m); // 팀 -> 회원 연관관계 설정 (실제 JPA 수행로직엔 영향 주지않음)

entityManager.persist(m);
```

위와 같이 양방향 연관관계를 모두 맺어주어 순수 객체 상태에서도 안전하게 사용가능한 상태로 만들어 주는것이 좋다.  

그리고 보통 위 처럼 양 쪽 다 신경쓰는 경우 실수로 지정하지 않는 경우가 발생할 수 있다.  

때문에 연관관계 주인 측에서 완벽하게 맺어주도록 지정하는 것이 좋다.  

```java
Member class

public void setTeam(Team team) {
    this.team = team;            // 회원 -> 팀
    team.getMembers().add(this); // 팀 -> 회원
}
```

위 처럼 연관관계 편의 메소드를 작성하여 실수를 방지할 수 있다.

하지만 위 코드에도 치명적인 문제가 있다.  
team의 members는 연관관계의 주인이 아니기 때문에 새 팀이 설정될 때 영속성 컨텍스트가 flush되지 않으면 team의 members 리스트 안에 자신이 남아 있는 문제가 발생한다.  

```java
public void setTeam(Team team) {
    // 현재 소속팀이 있으면 해당 팀의 회원목록에서 자신을 제거
    if (this.team != null) {
        this.team.getMembers().remove(this);
    }

    this.team = team;            // 회원 -> 팀
    team.getMembers().add(this); // 팀 -> 회원
}
```

위 처럼 로직을 추가하여 보다 견고하게 양방향 연관관계를 관리하도록 할 수 있다.


### 정리하면...
1. 단방향 매핑을 통해 테이블 - 객체의 연관관계 매핑은 완료된다.
2. 단방향 매핑을 양방향 매핑으로 만들면 역방향 객체 그래프 탐색 기능이 추가된다.
3. 양방향 연관관계를 매핑하려면 객체에서 양쪽 방향을 모두 관리해야한다.
4. 기본적으로 단방향 매핑을 사용하고 객체 그래프 탐색 기능이 필요할 때 양방향을 사용하도록 코드를 추가해도 무방하다.

---
>자바 ORM 표준 JPA 프로그래밍을 공부하며 정리한 내용입니다.
