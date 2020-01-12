---
layout: post
title:  "Reflection Type을 활용해 JPA 공통 AttributeConverter 구현"
date:   2019-10-30 11:43:19
category: backend-JPA
tags: [Java,JPA]
comments: true
draft: false
---

얼마전에 JPA에서 사용할 공통 AttributeConverter 를 구현하면서 고민했던 점들을 정리한다.

<!--more-->

## 공통 AttributeConverter 의 필요성
Entity와 테이블을 매핑하다보면 String, Integer, Long, LocalDateTime 외에 Json값이 저장되어있는 컬럼이나 코드값이 저장되어있는 컬럼의 경우 AttributeConverter 를 통해 Json 모델, Enum 과 매핑을 시킨다.

```java
// Entity
@Entity
public class UserEntity {

	@Converter(converter = UserStatusAttributeConverter.class)
	private Status status;

	@Converter(converter = UserOptionAttributeConverter.class)
	private UserOption userOption;

}

// 사용할 Enum 타입 매핑 Converter
public class UserStatusAttributeConverter implements AttributeConverter<Status, Integer> {

	@Override
	public Integer convertToDatabaseColumn(Status attribute) {
		return attribute.getCode();
	}

	@Override
	public Status convertToEntityAttribute(Integer value) {
		return Status.get(value);
	}
}

// 사용할 Json 타입 매핑 Converter
public class UserOptionAttributeConverter implements AttributeConverter<UserOption, String> {

	@Override
	public String convertToDatabaseColumn(UserOption attribute) {
		return JsonUtils.toJson(attribute);
	}

	@Override
	public UserOption convertToEntityAttribute(String value) {
		return JsonUtils.fromJson(value, UserOption.class);
	}

}
```

간단하게 위 예시코드처럼 필요할 때마다 Converter 를 작성하다보면 파일 갯수는 하염없이 증가한고, 중복코드도 수없이 생기게 된다.

`convertToDatabaseColumn` 메소드와 `convertToEntityAttribute` 메소드를 잘보면 처리대상이되는 Enum 타입이나 Json 모델 타입만 다르지, 매핑해주는 로직은 달라지지 않는다.

## 공통 AttributeConverter 구현
Generic을 활용하면 위에서 말한 중복코드 발생문제를 해결해 줄 수 있다. 

```java
// 공통 Json Attribute Converter
public abstract class JsonAttributeConverter<T> implements AttributeConverter<T, String> {

	// 처리할 Json 모델의 타입토큰
	private Class<T> attributeType;
 
	public JsonAttributeConverter(Class<T> attributeType) {
		this.attributeType = attributeType;
	}

	@Override
	public String convertToDatabaseColumn(T attribute) {
		return JsonUtils.toJson(attribute);
	}

	@Override
	public T convertToEntityAttribute(String value) {
		return JsonUtils.fromJson(value, attributeType);
	}

}

// 공통 컨버터 확장하여 사용
public class UserOptionAttributeConverter extends JsonAttributeConverter<UserOption> {

	UserOptionAttributeConverter() {
		// 클래스 리터럴 전달
		super(UserOption.class);
	}

}
```
위 처럼 Json 모델의 타입 토큰을 전달받아 **Json 파싱**하는 기능을 공통화하여 중복코드를 없앨수 있다.
다만 여전히 필요한 컨버터만큼 만들어줘야하는 점은 달라지지않는데, Inner 클래스를 활용해 파일수를 줄여줄 수 있겠다.

```java
public class UserOption {

	private String option;
	...

	static class UserOptionConverter extends JsonAttributeConverter<UserOption> {
		UserOptionConverter() {
			super(UserOption.class)
		}
	}
}
```

위 예시처럼 클래스 리터럴을 전달하여 알려주는 방법도 있고, Reflection Type을 이용해 리터럴을 전달받지 않고 타입을 획득하는 방법으로 처리할 수도 있다.

```java
public abstract class JsonAttributeConverter<T> implements AttributeConverter<T, String> {

	// 처리할 Json 모델의 타입토큰
	private Class<T> attributeType;
 
	public JsonAttributeConverter(Class<T> attributeType) {
		this.attributeType = detectAttributeType();
	}

	private Class<T> detectAttributeType() {
		ParameterizedType type = (ParameterizedType) getClass().getGenericSuperclass();
		return (Class<X>) type.getActualTypeArguments()[0];
	}

	@Override
	public String convertToDatabaseColumn(T attribute) {
		return JsonUtils.toJson(attribute);
	}

	@Override
	public T convertToEntityAttribute(String value) {
		return JsonUtils.fromJson(value, attributeType);
	}

}

public class UserOption {

	private String option;
	...

	static class UserOptionConverter extends JsonAttributeConverter<UserOption> {
	}
}
```