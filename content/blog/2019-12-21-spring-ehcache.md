---
layout: post
title: "Spring Ehcache 사용 간략한 정리"
date:   2019-12-21 11:43:19
category: backend-spring-framework
tags: [Spring Framework,Ehcache]
comments: true
draft: false
---
#### 1. Spring Cache Abstraction
[https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-caching.html#boot-features-caching-provider](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-caching.html#boot-features-caching-provider)


#### 2.  적용
##### 2-1. @EnableCaching
Spring Boot Application 설정시 `@EnableCaching` 어노테이션 추가하여 Application에 캐시 기능 사용하겠다는 것을 알린다

```java
@SpringBootApplication
@ComponentScan("com.nhnent.gia")
@EntityScan(basePackages = {"com.nhnent.gia.model"}, basePackageClasses = {Application.class, Jsr310JpaConverters.class})
@EnableCaching 
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

이후 추가적이 설정이 없는 경우 기본 캐시 `ConcurrentHasnMap` 를 사용하고, 다른 캐시 라이브러리를 추가하면 Spring Boot의 `Auto Detect`기능에 따라 해당 라이브러리를 자동으로 사용하게 된다.

##### 2-2. ehcache.xml 작성
[https://www.ehcache.org/documentation/2.8/configuration/configuration.html](https://www.ehcache.org/documentation/2.8/configuration/configuration.html)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://www.ehcache.org/ehcache.xsd"
         updateCheck="true"
         moritoring="autodetect"
         dynamicConfig="true"
         maxBytesLocalHeap="4M">
    
	<cache name="recentWinHistory"
                eternal="false"
                timeToIdleSeconds="0"
                timeToLiveSeconds="300"
                overflowToDisk="false"
                diskPersistent="false"
                diskExpiryThreadIntervalSeconds="120"
                memoryStoreEvictionPolicy="LRU">
	</cache>
</ehcache>
```

##### 2-3. application.properties추가
캐시 설정파일 `ehcache.xml` 경로 설정
```properties
spring.cache.ehcache.config=classpath:ehcache.xml
```

##### 2-4. @Cacheable
`ehcache.xml`에서 정의한 캐시를 `@Cacheable("{name}")` 형태로 적용
* 기본
```java
@Cacheable("recentWinHistory")
List<WinHistory> findAllWinHistory(Integer eventCode, Integer eventNo);
```

* 파라미터별로 캐싱하고 싶은 경우
캐시가 eventCode값 단위로 캐싱된다
```java
@Cacheable("recentWinHistory", key="#eventCode")
List<WinHistory> findAllWinHistory(Integer eventCode, Integer eventNo);
```