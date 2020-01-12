---
layout: post
title:  "Java 동등성과 동일성의 차이"
date:   2018-10-09 19:19:28
category: backend-java
tags: [Java]
comments: true
draft: false
---
동일성은 인스턴스가 실제로 같은 인스턴스임을 의미하는 바로 == 연산자를 통해 비교한다.

동등성은 서로 다른 인스턴스이지만 가지고 있는 값이 같음을 의미하며 equals 메소드로 비교한다.  
자바에서 equals를 따로 구현하지 않은 경우 Object클래스의 equals메도스가 호출되며, Object메소드의 equals는 아래 코드처럼 동일성을 비교한다.
<!--more-->
```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

때문에 동등성 비교를 위해서는 구현하는 클래스에 equals메소드를 Override하여 구현해야한다.

핵심은 두 비교대상이 같은 인스턴스인지 아닌지 여부이다.
