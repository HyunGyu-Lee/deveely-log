---
layout: post
title:  "JPA의 쿼리 기술 종류"
date:   2019-10-15 13:03:19
category: backend-JPA
tags: [JPA,QueryDSL]
comments: true
draft: false
---
JPQL, Criteria, Named Query 등등 JPA는 엔티티 객체를 조회하기 위해 다양한 쿼리 기술을 지원합니다.

그동안 JPA 를 사용하면서 사용하는 쿼리 기술이 어느 기술인지도 잘 모르고 사용해온 나 자신을 위해.. 이 글에서 정리해보고자 합니다.

각 기술에 대한 자세한 설명보다는 각 기술이 어떤 것인지와 특징 등에 중점을 두고 정리하겠습니다.

<!--more-->

## JPQL 
- Java Persistence Query Language의 약자
- SQL이 데이터베이스 테이블을 대상으로 쿼리하는 거라면 JPQL은 Entity 객체를 대상으로 쿼리
- JPQL은 쿼리문장이 특정 DBMS에 종속되지 않도록 쿼리를 SQL을 추상화
- 작성한 JPQL은 프레임워크에 의해 애플리케이션이 사용하는 데이터베이스의 적합한 SQL로 변환되어 실행
- JPA 표준

## JPA Criteria Query
- JPQL을 programmatically하게, 즉 자바코드로 작성할 수 있게 도와주는 빌더
- 문자열로 작성하는 JPQL과 달리 자바코드로 쿼리를 작성하기 때문에 컴파일 단계에서 오류파악이 가능
- 쿼리가 복잡하고, 길어지는 경우 JPQL 보다 직관적
- 자바코드로 작성하기 때문에 쿼리문을 동적으로 작성 가능
- JPA 표준

## QueryDSL
- Criteria Query 와 마찬가지로 JPQL을 쉽게 작성할 수 있도록 도와주는 빌더
- 비표준
- Entity를 파싱하여 QEntity를 생성하고, 이를 쿼리작성에 이용
- Criteria의 동적인 특징과 JPQL의 표현력을 타입에 안전한 방법으로 제공
