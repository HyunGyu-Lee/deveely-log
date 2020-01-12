---
layout: post
title:  "Java Stream API 소개"
date:   2018-08-15 01:55:23
category: backend-java
tags: [Java8,StreamAPI,Java Collections]
comments: true
draft: false
---
#### Stream API의 특징 및 기존 Collection과의 차이점
1. 개발자가 직접 반복문을 작성하는 방식의 컬렉션과 달리 내부 반복(internal iteration)을 통해 작업한다.
2. 재사용이 가능한 컬렉션과 달리 재사용이 불가능하다.
3. 스트림은 원본 데이터를 변경하지 않는다.
4. 스트림의 연산은 filter-map 기반의 API를 사용하여 지연(lazy)연산을 통해 성능을 최적화한다.
5. 스트림은 parallelStream() 메소드를 통해 병렬 처리를 쉽게 할 수 있다.
<!--more-->
#### Stream API 사용 메커니즘
1. 스트림 생성 : 스트림을 생성한다.
2. 중개 연산 : 생성된 스트림을 다른 스트림으로 변환하는 중간 연산으로, 하나 이상의 연산을 지정할 수 있다.
3. 최종 연산 : 중개 연산이 종료된 스트림에서 결과를 어떤 방식으로 산출할 지에 대한 연산을 지정한다.

```java
someCollection.스트림생성().중개연산().최종연산();
```

#### 1. 스트림 생성
스트림을 생성하는 방법은 정말 다양하다.  
스트림API는 주로 컬렉션 또는 배열에 담긴 데이터를 처리하는데 사용되며 아래 예시 코드에서는 배열, 컬렉션에서 스트림을 얻는 방법을 소개한다.  

```java
// 가변인자 배열을 받아 스트림 생성
Stream<String> stream = Stream.of("Java", "C", "Javascript");

// 컬렉션에서 메소드 호출을 통한 스트림 생성
List<String> list = getSomeList();
Stream<String> stream = list.stream();
```

#### 2. 중개 연산
중개연산 메소드는 함수형 인터페이스로 람다 표현식을 통해 간결하게 작성이 가능하다.  
아래 예시 코드에서 유용하게 사용하는 중개연산 메소드를 소개한다.  

```java
// 데이터 필터링
Stream<String> filteredStream = stream.filter(e -> e.contains("Java"));

// 데이터 변환
Stream<String> processedStream = stream.map(e -> e.toUpperCase());

// 중복 값 제거
 Stream<String> uniqueStream = stream.distinct();

// 첫 번째 요소 조회
Stream<String> first = stream.findFirst();

// 정렬
Stream<String> sortedStream = stream.sorted();
Stream<String> sortedStream = stream.sorted(Comparator.comparing(String::length));
```

다음과 같이 여러 중개연산을 조합하여 데이터를 손쉽게 처리할 수 있다.

```java
// 데이터 필터링 후 첫 번째 요소 조회
Stream<String> filteredFirst = stream.filter(조건).findFirst();

// 중복 값을 제거한 후 정렬
Stream<String> uniqueSorted = stream.distinct().sorted(정렬조건);
```

#### 3. 최종연산 (결과 취합)
중개연산을 통해 가공된 스트림을 원하는 결과로 추출하는 방법을 제공한다.

```java
// List로 변환
List<String> processedList = stream.collect(Collectors.toList());

// Joining
String joined = stream.collect(Collectors.joining());
String joined = stream.collect(Collectors.joining(","));

// 그룹핑 (분류함수에 따라 분류됨)
Map<String, List<String>> groups = stream.collect(Collectors.groupingBy(분류함수));

// 그룹핑 응용 (분류를 수행한 후 각 그룹별 건수를 집계한 Map으로 변환)
Map<String, Long> groups = stream.collect(Collectors.groupingBy(분류함수, Collectors.counting()));
```

----
### 이 포스트에서 소개한 방법들은 Stream API에서 제공하는 수많은 연산에 비하면 극히일부분입니다.  
기존 for, while 등 반복문, if문 등을 통해 각 종 컬렉션을 처리하던 방법에 비해 코드량을 상당량 줄여주고, 간결하게 만들어 줄 수 있는 방법들에 대해 알아봤습니다.  

하지만 Stream API 또한 개발자의 성향, 습관 등에 따라 오히려 가독성이 떨어질 가능성이 있으며 성능에 대해서도 전통적인 Collection보다 떨어지는 케이스도 존재한다고 합니다.  

Collection, Stream 각각의 특징에 따라 알맞게 사용하여 두 방식이 제공하는 강점들을 챙길수 있는 코딩을 하는 것이 가장 Best 하다는 생각이 듭니다.  
