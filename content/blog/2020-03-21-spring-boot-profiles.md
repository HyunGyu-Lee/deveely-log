---
layout: post
title: "배포환경과 Spring Boot Profile 적용"
date:   2020-03-21 17:43:19
category: backend-spring-framework
tags: [Spring Boot,Spring Framework,Java]
comments: true
draft: false
---
![Image1](./images/common/spring-boot.png)

프로젝트를 개발하면, 배포 환경별로 다른 리소스를 참조하거나, 동작이 달라져야하는 등 처리가 필요합니다.
대표적인 경우 DB, 외부 API 연동, 로그레벨 등이 있을 수 있습니다.

대부분 프로젝트는 구동환경에 따라 크게 아래와 같은 단계로 나뉠수 있습니다.

- 로컬 환경 : 개발자 자신의 PC
- 개발 환경 : 테스트 등 여러 개발 목적으로 구성되어 있는 환경
- 운영 환경 : 실제 라이브로 서비스되는 환경

실제로는 QA 환경, 운영 전 베타 환경 등 세부적으로 나뉠수도 있습니다.
이번 포스트에서는 개발/운영 2가지 환경을 기준으로 간단하게 진행하겠습니다.

## Spring Boot에서 Profile
예제에 앞서 개발 / 운영 DB의 접속정보가 다음과 같다고 가정합니다.

| 환경 | url | username | password |
| ------ | --- | --- |
| 개발 | jdbc:mysql://developdb:3306/db | dbuser | dbuser123 |
| 운영 | jdbc:mysql://releasedb:3306/db | dbuser | dbuser123 |

개발단계에서는 다음과 같이 설정파일에 DB 접속정보를 설정합니다.

- application.properties
```
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://developdb:3306/db
spring.datasource.username=dbuser
spring.datasource.password=dbuser123
```

만약 위 프로젝트가 개발이 완료되고 운영환경에 배포해야한다면 어떻할까요?
단순히는 운영에 배포할 때 설정파일을 수정할 수 있지만, 매번 개발, 운영 배포 시 마다 설정파일을 변경해서 배포한다는건 사실상... 너무 문제의 여지가 많은 방법입니다.

스프링에서는 다음과 같은 방법으로 이 문제를 해결할 수 있습니다.

## 배포 환경별 설정파일 관리
프로젝트 내에서 배포 환경별로 설정파일을 관리합니다.
개발은 `develop`, 운영은 `release`로 네이밍한 후 아래와 같이 2개의 설정파일을 만듭니다.

- application-develop.properties
```
spring.datasource.url=jdbc:mysql://developdb:3306/db
spring.datasource.username=dbuser
spring.datasource.password=dbuser123
```

- application-release.properties
```
spring.datasource.url=jdbc:mysql://releasedb:3306/db
spring.datasource.username=dbuser
spring.datasource.password=dbuser123
```

이렇게 리소스를 분리한 후, application.yml엔 profile과 관련없는 공통 속성을 설정합니다.

- application.properties
```
# 공통 설정을 진행합니다.
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

그리고 프로그램 실행 시 `-Dspring.profiles.active={active profile}` 옵션으로 활성 프로필을 전달해주면
`application.properties`와 `application-{active profile}`이 사용되어 설정이 구성됩니다.

```sh
java -jar SampleApp -Dspring.profiles.active=develop
```

## 마무리
이 포스트에서 소개한 방법은 Spring Framework가 선별적으로 리소스를 사용하게 함으로써 프로필 분리를 구현한 것인데요.
이 방법외에 Maven 자체에서 빌드 시에 선별적으로 설정 (resources)를 선택하게 하는 방법도 있습니다.
이 방법은 다음 포스트에서 작성하겠습니다.

## 참고
- https://www.baeldung.com/spring-profiles