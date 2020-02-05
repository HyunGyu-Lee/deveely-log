---
layout: post
title: "Java Bean Validation 사용"
date:   2020-02-05 11:43:19
category: backend-spring-framework
tags: [Spring Boot,Spring Framework,Java]
comments: true
draft: false
---
Java Bean Validation는 공식 사이트에서는 오브젝트 레벨의 제약 선언 및 유효성 검사 기술을 제공하는 것을 목표로 하고 있는 기술이라고 합니다.
> The technical objective of this work is to provide an object level constraint declaration and validation facility for the Java application developer, as well as a constraint metadata repository and query API.

실제로 프로그래밍을 하다보면 어떤 객체의 값이 비었는지, 공백인지, 날짜가 이전인지 등의 유효성 검사를 빈번하게 수행하게 되고 이로 인해 유효성 검사 로직들이 여러 곳에 흩어지게 되는 경우도 있는데요. Java Bean Validation을 이용하면 검사 대상 클래스에 어노테이션 기반 제약조건을 선언하여 간결하게 유효성 검사를 할 수 있습니다.

Bean Validation은 자바에서 유효성 검사를 하는 방법에 대한 명세입니다. 즉, 실제로 동작하는 코드가 아니라는 것입니다. 

때문에 실제로 동작하려면 이 명세를 구현한 구현체가 있어야하는데 대표적인게 Hibernate Validator입니다. 그리고 Spring Boot 2.0은 Bean Validation 2.0을 사용하고 있습니다.

#### \# Bean Validation 버전별 Hibernate Validator 호환성
| Version | JSR | Release | Compatible Implementation | 
| --- | --- | --- | --- |
| Bean Validation 1.0 | JSR 303 | Java EE6, 2009 | Hibernate Validator 4.3.1.Final |
| Bean Validation 1.1 | JSR 349 | Java EE7, 2013 | Hibernate Validator 5.1.1.Final |
| Bean Validation 2.0 | JSR 380 | Java EE8, 2017 | Hibernate Validator 6.0.1.Final |

그럼 Bean Validation을 실제로 사용하는 방법을 알아보겠습니다.

## 기본적인 사용 방법
#### \# 의존성 추가
Bean Validation은 `javax.validation.validation-api`로 별도 모듈이 있지만, Hibernate Validator가 해당 모듈을 포함하고 있기 때문에 pom.xml에 아래와 같이 의존성을 추가해줍니다.
```xml
<dependency>
  <groupId>org.hibernate</groupId>
  <artifactId>hibernate-validator</artifactId>
  <version>6.0.7.Final</version>
</dependency>
```

만약 실행환경이 SE라면 아래 의존성을 함께 추가해줘야함니다. (Tomcat 등의 EE 환경에서는 필요 없음)

`org.glassfish.javax.el`의 경우 JSR 341에 명시된 `Unified Expression Language`에 대한 구현체로 Hibernate Validator가 제약 조건 위반 메시지 표현 등을 처리하기 위해 필요하다고 합니다. 

`org.hibernate.hibernate-validator-cdi`는 JSR 346에 명시된 `CDI (Contexts and Dependency Injection)`의 구현체로 자세히는 모르겠지만 @Inject 등으로 Validator를 주입받고, @Valid 등으로 유효성 검사를 자동으로 수행하는 등에 사용하기 위해 필요한 의존성으로 생각됩니다.

```xml
<dependency>
  <groupId>org.glassfish</groupId>
  <artifactId>javax.el</artifactId>
  <version>3.0.0</version>
</dependency>
<dependency>
  <groupId>org.hibernate</groupId>
  <artifactId>hibernate-validator-cdi</artifactId>
  <version>6.0.7.Final</version>
</dependency>
```

#### \# 유효성 검증할 객체 생성
예시로, 아래와 같이 모델 객체를 만들어 보겠습니다.

필드 위에 선언한 `@NotBlank`, `@Positive` 어노테이션들이 바로 제약조건들입니다.

아래 예시에서는 name은 공백이지 않아야하며 age는 0보다 큰 숫자여야한다는 제약조건을 걸었습니다.
```java
public class Member {
	@NotBlank
	private String name;

	@Positive
	private int age;

	public Member(String name, int age) {
		this.name = name;
		this.age = age;
	}

	public String getName() {
		return name;
	}

	public int getAge() {
		return age;
	}
}
```

#### \# 유효성 검증 수행
먼저 유효성 검사를 수행하는 `Validator`를 생성합니다.
```java
private Validator validator = Validation.buildDefaultValidatorFactory().getValidator();
```

다음으로 Validator를 통해 Member 객체의 유효성 검사를 수행합니다.

`Validator`의 `validate` 메소드에 검사할 객체를 전달하고, 그 결과를 `Set<ConstraintViolation<?>>` (제약조건 위반 Set)으로 받습니다.

```java
Member member = new Member("", -1);

Set<ConstraintViolation<Member>> violations = validator.validate(member);
violations.forEach(e -> {
  System.out.println(String.format("검사 필드: %s, 유효하지 않은 값: [%s] 메시지: %s", e.getPropertyPath(), e.getInvalidValue(), e.getMessage()));
});
```

위 코드를 실행하면 아래와 같은 결과를 확인할 수 있습니다.
```
검사 필드: age, 유효하지 않은 값: [-1] 메시지: must be greater than 0
검사 필드: name, 유효하지 않은 값: [] 메시지: 반드시 값이 존재하고 공백 문자를 제외한 길이가 0보다 커야 합니다.
```

만약 메시지를 커스텀 메시지를 출력하고 싶다면 각 제약조건 어노테이션의 `message` 속성에 설정하면 됩니다.

```java
public class Member {
	@NotBlank(message = "이름이 비어있습니다.")
	private String name;

	@Positive(message = "나이는 0보다 큰 값이어야합니다.")
	private int age;

	...
}
```

```
검사 필드: name, 유효하지 않은 값: [] 메시지: 이름이 비어있습니다.
검사 필드: age, 유효하지 않은 값: [-1] 메시지: 나이는 0보다 큰 값이어야합니다.
```

## Built-in Constraint Definitions
Bean Validation에는 대부분의 경우에 사용가능한 제약조건 어노테이션들이 내장(Built-in) 되어있습니다.

`javax.validation.constraints` 패키지를 확인해보면 총 22개의 내장 제약조건이 있는것을 확인할 수 있습니다.

- @AssertFalse
- @AssertTrue
- @DecimalMax
- @DecimalMin
- @Digits
- @Email
- @Future
- @FutureOrPresent
- @Max
- @Min
- @Negative
- @NegativeOrZero
- @NotBlank
- @NotEmpty
- @NotNull
- @Null
- @Past
- @PastOrPresent
- @Pattern
- @Positive
- @PositiveOrZero
- @Size

거의 모든 어노테이션들이 이름만으로 어떤 제약조건인지 쉽게 유추가 가능합니다.

자세한 사항은 [공식 홈페이지](https://beanvalidation.org/2.0/spec/#builtinconstraints)의 `8. Built-in Constraint definitions
` 섹션을 참고하시는 것을 추천드립니다.

## Bean Validation 1.1 vs 2.0
Bean Validation 2.0의 주요 변경사항은 공식 사이트에서 아래와 같이 소개하고 있습니다. 
전체 변경사항은 [이 곳](https://beanvalidation.org/2.0/spec/#changelog)에서 확인할 수 있습니다.

- support for validating container elements by annotating type arguments of parameterized types e.g. List<@Positive Integer> positiveNumbers. This also includes:
  - more flexible cascaded validation of container types
  - support for java.util.Optional
  - support for the property types declared by JavaFX
  - support for custom container types
- support for the new date/time data types (JSR 310) for @Past and @Future
- new built-in constraints: @Email, @NotEmpty, @NotBlank, @Positive, @PositiveOrZero, @Negative, @NegativeOrZero, @PastOrPresent and @FutureOrPresent
- leverage the JDK 8 new features (built-in constraints are marked repeatable, parameter names are retrieved via reflection)

## 참고
아까도 언급했지만 Spring Boot 2.0 이상 버전에서는 Bean Validation 2.0이 기본적으로 사용됩니다.
하지만 이하 버전에서는 Bean Validation 1.1 버전이 사용되는데 이런 경우 다음과 같은 방법으로 버전업을 진행하면 됩니다.

```xml
<properties>
    <javax-validation.version>2.0.1.Final</javax-validation.version>
    <hibernate-validation.version>6.0.7.Final</hibernate-validation.version>
</properties>
```