---
layout: post
title: "Kotlin 프로퍼티 (Properties)"
date:   2021-09-06 16:10:22
category: backend-kotlin
tags: [Kotlin]
comments: true
draft: false
---

보통 객체지향 프록그래밍에서 클래스를 정의할 때 객체의 상태는 멤버변수 (필드)로 행위는 메소드로 표현합니다.
그리고 보통 멤버변수가 있으면 객체의 값을 설정하고 가져오는 (setter / getter) 메소드가 동반되는 경우가 많은데요.

코틀린에서는 이런 요소를 아우르는 `프로퍼티`라는 개념이 제공됩니다.
즉 프로퍼티는 getter, setter와 같은 접근자를 포함하고있는 필드입니다.

## 프로퍼티 정의
프로퍼티를 정의하는 전체 문법은 아래와 같습니다.

```
(var/val) <propertyName>[: <PropertyType>] [= <property_initializer>]
          [<getter>]
          [<setter>]
```

- var / val : 프로퍼티 선언을 위한 예약어. var는 초기화 후 값 변경이 가능한 프로퍼티, val은 초기화 후 값 변경이 불가능한 프로퍼티
- propertyName: 프로퍼티명
- PropertyType: 프로퍼티 타입 (타입 추론이 가능한 경우 생략 가능)
- property_initializer: 프로퍼티 값 초기화 (초기화가 불필요한 경우 생략 가능)
- getter / setter : 해당 프로퍼티에 대한 getter / setter를 정의 (생략할 경우 default getter, setter 적용) `val` 프로퍼티는 setter를 가질수없음

프로퍼티와 getter, setter는 접근제한자 (public, internal, protected, private) 적용이 가능합니다.

이제 프로퍼티가 자바로 변환되면 어떻게 적용되는지 살펴보겠습니다.
아래 name, age은 생성자 인자이면서 프로퍼티입니다.
```kotlin
class Person(var age: Int, val name: String)
```

이 코드가 자바로 변환되면 아래와 같습니다.

```java
public static final class Person {
    private int age;
    @NotNull
    private final String name;

    public final int getAge() {
        return this.age;
    }

    public final void setAge(int var1) {
        this.age = var1;
    }

    @NotNull
    public final String getName() {
        return this.name;
    }

    public Person(int age, @NotNull String name) {
        Intrinsics.checkNotNullParameter(name, "name");
        super();
        this.age = age;
        this.name = name;
    }
}
```

`var`로 선언한 `age`의 경우 `getAge`, `setAge`와 같이 getter, setter가 자동으로 생성되었습니다.
반면 `val`로 선언한 `name`의 경우 값 변경이 불가능하고 setter를 정의할 수 없기 때문에 당연하게 `setter`가 생성되지 않은 것을 확인할 수 있습니다. 

보통 자바에서는 getter / setter는 필드값 반환, 세팅의 역할이 주였는데요.
코틀린에서의 getter / setter는 보다 다양하게 활용이 가능합니다.

```kotlin
class Product(var size: Int) {
    val isEmpty
        get() = this.size == 0
}
```

이렇게 작성한 코드는 자바로 다음과 같이 변환됩니다.

```java
public static final class Product {
  private int size;

  public final boolean isEmpty() {
      return this.size == 0;
  }

  public final int getSize() {
      return this.size;
  }

  public final void setSize(int var1) {
      this.size = var1;
  }

  public Product(int size) {
      this.size = size;
  }
}
```

이런식으로 다른 필드를 활용한 계산된 (computed) 속성으로도 활용이 가능합니다.

또한 커스텀 setter를 지정하여 프로퍼티 값 설정 전 유효성 검사, 혹은 다른 동작도 지정할 수 있습니다.
아래 예시에서는 1~3 값만 설정가능한 `grade` 프로퍼티를 정의했습니다.
setter 로직 중 `field`가 실제 프로퍼티의 값인데 코틀린에서는 이를 `Backing Fields`라고 합니다.
```kotlin
class Person {
    var grade = 1
        set(value) {
            if (value in 1..3) {
                field = value
            } else {
                throw IllegalArgumentException("Grade 는 1 ~ 3 사이여야합니다.")
            }
        }
}
```

