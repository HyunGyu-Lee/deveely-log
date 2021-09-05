---
layout: post
title: "Kotlin 생성자 개념과 사용법 정리"
date:   2021-09-05 18:10:22
category: backend-kotlin
tags: [Kotlin]
comments: true
draft: false
---

코틀린에서 생성자를 정의하는 여러가지 방법에 대해 정리합니다.
코틀린 생성자는 크게 주 생성자(primary constructor)와 부 생성자(secondary constructor)로 나뉘고 각각 제약이 조금씩 다릅니다.

## 1. 주 생성자 (Primary constructor)
기본적으로 `constructor` 키워드를 통해 생성자를 정의할 수 있습니다.
`constructor` 키워드 앞에 접근 제한자를 지정할 수 있습니다.
`constructor` 키워드 자체를 생략할 수도 있습니다. 단, 이경우엔 접근 제한자는 지정할 수 없습니다.
이렇게 선언하는 생성자를 `주 생성자`라고 합니다.

```kotlin
class Person constructor(name: String, age: Int)

// 접근 제한자 지정
class Person private constructor(name: String, age: Int)

// constructor 키워드 생략 가능
class Person(name: String, age: Int)
```

하지만 이 경우엔 생성자 시그니쳐만 정의할 수 있고 초기화 로직 (내부 프로퍼티에 할당)을 작성할 수 가 없습니다.
즉 위 코드만 가지고는 객체 생성 시 전달되는 `name`, `age`를 클래스 내부에서 사용할 수 없는 것입니다.
이러한 상황에 대비해 `init`이란 키워드가 제공됩니다. 초기화 로직을 포함 추가로 필요한 로직이 있다면 호출할 수 있습니다.
혹은 프로퍼티를 정의하면서 바로 대입해줄 수 있습니다.

```kotlin
class Person(name: String, age: Int) {
    val name: String
    val age: Int

    // 아래 init 블록 통해 초기화 가능
    init {
      this.name = name
      this.age = age
    }

    // 혹은 init 블록 사용 없이 프로퍼티 선언 후 대입
    val name = name
    val age = age
}
```

하지만 매번 이런식으로 프로퍼티에 값을 할당해주는 것은 상당히 번거로운 일이 될 수 있습니다.
코틀린에서는 생성자 시그니쳐를 작성할 때 인자 선언과 동시에 프로퍼티로 할당해주는 문법을 지원합니다.
아래와 같이 인자 선언할 때 `val / var` 을 주면 됩니다.
또한 

```kotlin
class Person(val name: String, val age: Int)
```

## 2. 부 생성자 (Secondary constructor)
`주 생성자` 이외에 추가로 생성하는 생성자들은 모두 `부 생성자`가 됩니다.
부 생성자는 주 생성자에 비해 몇가지 제약이 있습니다. 
- `부 생성자`는 `주 생성자`를 반드시 상속해야합니다.
- `부 생성자`에는 인자 선언과 동시에 프로퍼티 할당을 할 수 없습니다.

```kotlin
class Person(val name: String, val age: Int) {

    constructor(grade: Int) {}  // 오류 발생

    constructor(name: String, age: Int, grade: Int): this(name, age) {  // 이렇게 주 생성자를 상속해줘야함

    }

    constructor(name: String, age: Int, val grade: Int): this(name, age) {  // 부 생성자에서 var / val 사용 불가
    }

}
```

`주 생성자`가 없는 클래스가 존재할 수도 있다. 이 경우엔 아래와 같이 부 생성자를 자유롭게 선언해둘 수 있다.
```kotlin
class Person {
    
    constructor(name: String) { 
    }

    constructor(name: String, age: Int) { 
    }

    constructor(name: String, age: Int, grade: Int) { 
    }

} 
```

다만 이렇게 할 일이 거의 없는게.. 주 / 부 생성자 상관 없이 각 인자에 기본값(default)를 정의해 줄 수 있기 때문에
이를 활용해 대부분 주 생성자 하나로 해당 클래스의 객체 생성 방법을 대부분 다 표현해낼 수 있다.

```kotlin
class Person(val name: String, val age: Int, val grade: Int = 1)

fun main(args: Array<String>) {
    val p1 = Person("p1", 20)
    val p2 = Person("p2", 21, 2)
}
```

