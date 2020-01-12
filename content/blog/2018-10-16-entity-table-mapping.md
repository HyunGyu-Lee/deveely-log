---
layout: post
title:  "엔티티(Entity) 테이블(Table) 매핑"
date:   2018-10-16 20:27:42
category: backend-JPA
tags: [JPA,Entity,엔티티,매핑]
comments: true
draft: false
---
### 엔티티 매핑
@Entity : JPA가 관리하는 엔티티임을 알림  
@Table name : 테이블명   
       uniqueConstraints : 유니크 제약조건   
                       @UniqueConstraint(columnNames 유니크 제약 컬럼명, name 제약명)   
@Column -> 일반 컬럼 매핑   
@Enumerate -> Enum 타입 매핑  
@Temporal -> Date 타입 매핑, default값으로 hibernate의 @UpdateTimestamp를 지정하면 시간이 자동으로 설정된다.   
@Lob -> CLOB, BLOB 타입 매핑   
<!--more-->
### 기본키 매핑
Oracle은 시퀀스오브젝트, MySQL은 AUTO_INCREMENT 등 DB마다 기본키 할당 방식은 다르다.  
JPA는 다음과 같은 방법들로 이 문제를 해결한다.

#### 1) 직접 할당
기본 키를 애플리케이션에서 직접 할당한다. 즉 엔티티매니저를 통해 엔티티가 persist 되기 전에 설정하는 것을 의미한다.

#### 2) 자동 생성
대리 키 사용 방식, @Id + @GeneratedValue 사용

###### 2-1) IDENTITY
기본 키 생성을 데이터베이스에 위임한다. MySQL, PostgreSQL, SQL Server, DB2에서 사용이 가능하다.  
@GeneratedValue의 strategy를 GenerationType.IDENTITY로 설정한다.  

###### 2-2) SEQUENCE
데이터베이스 시퀀스를 사용해 기본키를 할당한다. PostgreSQL, Oracle, DB2, H2 데이터베이스에서 사용이 가능하다.   
@GeneratedValue의 strategy를 GenerationType.SEQUENCE로 설정한다.   

###### 2-3) TABLE
키 생성 테이블을 이용한다.   
@GeneratedValue의 strategy를 GenerationType.TABLE로 설정한다.   
주로 사전에 시퀀스를 관리할 테이블을 만들어 둔후 엔티티에 @TableGenerator 를 이용해 시퀀스 관리 테이블을 매핑해주는 식으로 진행한다.   
@TableGenerator (name : generator명, table : 시퀀스 관리 테이블, pkColumnValue : 시퀀스명, allocationSize : 한번에 증가하는 시퀀스 크기)   
이렇게 진행하면 pkColumnValue에 설정한 시퀀스명이 시퀀스 관리 테이블에 값으로 추가되고, @GeneratedValue의 generator에서 사용이 가능하다.   

###### 2-4) AUTO
데이터베이스 방언 (DIALECT)에 따라 자동 선택 되도록 하는 것으로, 데이터베이스 벤더에 의존적이지 않은 장점이있다.  
스키마 자동 생성 설정을 해둔 경우 전략에 따라 시퀀스, 테이블을 자동으로 생성하며 그렇지 않은 경우 미리 만들어두어야한다.  

IDENTITY나 SEQUENCE의 경우 데이터베이스에 의존하지만 TABLE의 경우 테이블을 이용하는 방식으로 모든 데이터베이스에서 사용 가능하다.   

직접 할당은 @Id만 사용하여 할 수 있고 자동 생성은 @GeneratedValue를 추가하여 원하는 키 생성방식을 선택하면 된다.   

참고로 키 생성 전략을 사용하려면 persistence.xml에 hibernate.id.new_generator_mappings속성이 true로 설정되어있어야한다.  

### 권장하는 식별자 선택 전략
데이터베이스 기본 키는 다음 3가지 조건을 모두 만족해야한다.   

##### 1. null값은 허용되지 않는다.
##### 2. 유일해야한다.
##### 3. 변해선 안 된다.

테이블의 기본 키를 선택하는 전략은 크게 2가지가 있다.   

- 자연 키 (natural key) : 비즈니스에 의미가 있는 키 (주민등록번호, 이메일, 전화번호 등)   
- 대리 키 (surrogate key) : 비즈니스와 관련 없는 임의로 만들어진 키, 대체 키라고도 함 (시퀀스, auto_increment 등)   

자연 키보다는 대리 키를 권장한다.  
예를 들어 전화번호가 기본 키인 경우 전화번호가 없을 수도 있고 변할 수도 있다.  
주민등록번호의 경우 변하지 않고 null일 수도 없지만 정책의 변화 등 비즈니스의 변화에 영향을 받을 수 있다.  

때문에 비즈니스와 관련이 없는, 비즈니스의 변화에 영향을 받지 않는 대리 키 사용을 권장한다.   

---
>자바 ORM 표준 JPA 프로그래밍을 공부하며 정리한 내용입니다.
