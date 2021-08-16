---
layout: post
title: "Reactive Programming 1"
date:   2021-08-17 12:32:19
category: backend-reactive-programming
tags: [Reactive Programming]
comments: true
draft: false
---
이 포스트 시리즈는 Reactive Programming은 토비의 스프링 저자 이일민님의 리액티브 프로그래밍 유튜브 강좌를 공부하며 정리한 내용입니다. 

## 1. Iterable과 Observable의 차이점
##### 1.1. Iterable 개념
자바에서 연속적인 데이터 구조를 표현할 때 `List`를 주로 사용한다.
그리고 주로 아래와 같이 `for-each` 구문을 사용한다.

```java
import java.util.List;
import java.util.Arrays;

List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
for (Integer i : list) {
  System.out.println(i);
}
```

이는 `List` 인터페이스가 `Iterable` 인터페이스를 상속받고 있기 때문이다.
자바에서 `Iterable` 인터페이스를 구현한 타입은 `for-each` 루프 사용이 가능하다.

`Iterable` 인터페이스는 데이터를 꺼내올 수 있는 `iterator()`를 생성하도록 하는 메소드가 있다.
`Iterator`는 `next()` 메소드를 통해 데이터를 제공하고 `hasNext()` 메소드를 통해 데이터가 더 남아있는지 여부를 반환한다.
즉 `Iterable` > `Iterator` > `Iterator.next()` 순으로 데이터를 가져오는 것이다.


##### 1.2. Observable 개념
참고 Java9에서 deprecate 됨
- Observable: 데이터를 만들어내는 Source, Event를 발생하여 `Observer`에게 전달, 옵저버는 여려개가 될 수 있음
- Observer: Observable에서 발생하는 이벤트를 수신하여 동작하는 오브젝트

```java
import java.util.Observable;
import java.util.Observer;

class IntObservable extends Observable implements Runnable {
  @Override
    public void run() {
      for (int i = 0; i < 10; i++) {
        // Observable에 변화가 생겼음을 알리는 메소드
        setChanged();
        // Observer들에게 변화를 알림 (값 포함)
        notifyObservers(i);
      }
    }
}

class Program {
  public static void main(String[] args) {
    Observer observer = new Observer() {
      @Override
      public void update(Observable observable, Object o) {
        // 옵저버의 변화로 이벤트가 발생하면 아래 로직 수행
        System.out.println(o);
      }
    };
  }
}
```

##### 1.3. Iterable VS Observable
데이터를 전달하고 전달받는 방식이 정반대이다. (쌍대성, duality)
- Iterable은 Pull이다
  - Iterator.next()를 통해 데이터를 지속적으로 당겨옴 (pull)
  - next() 메소드는 리턴값이 있음  
- Observable은 Push다
  - Observable.notifyObservers(Object)를 통해 데이터를 밀어줌 (push)
  - notifyObservers 메소드는 리턴값이 없음

##### 1.4. Observable의 문제점
- 끝났다 라는 개념이 없다. (Complete)
  - 사용할 때 직접 이 개념을 정해놓고 써야했다.
- 에러 처리에 대한 방식이 없다.
  - 복구 가능한 예외, 복구 불가능한 예외 등 여러 예외상황에 대한 처리 방식을 제공하지 않는다.
    
## 2. Reactive Streams의 표준
https://www.reactive-streams.org/
- 정식(?)은 아닌것 같지만 리액티브 프로그래밍의 대략적인 표준을 제시하고 있다.
- Java9 부터 `java.util.concurrent.Flow` 에 포함되어 있으며, 하위 버전에서는 디펜던시를 추가하여 사용할 수 있다.   
- Processor, Publiser, Subscriber, Subscription 4가지 간단한 [API](https://www.reactive-streams.org/reactive-streams-1.0.3-javadoc/org/reactivestreams/package-summary.html) 가 있고, 
  이를 reactive streams에서 제시하는 [스펙](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.3/README.md#specification) 을 준수하여 구현하면 된다.
- 이렇게 구현된 리액티브 엔진이 ReactiveX (RxJava), Reactor 등이 있다.

##### 2.1. Publisher / Subscriber 간단 스펙
- 옵저버 패턴의 Observable 역할  
- 일련의 시퀀스를 가진 요소를 제공해야한다.
- Publisher.subscribe(Subscriber)를 통해 등록
- 아래 프로토콜을 따라야한다.
```
onSubscribe onNext* (onError | onComplete)?
- onSubscribe는 subscribe 될 시 반드시 1번 호출되야함
- onNext 0 ~ 무한대
- onError, onComplete 0~1번 (optional)
  - onError 호출 시 onComplete 호출불가
  - onComplete 호출 시 onError 호출불가 
```

##### 2.2. 간단한 Publisher & Subscriber
- 아래 리액티브 프로그래밍의 동작원리를 간단하게 구현한 코드이다.
- `Publisher`와 `Subscriber` 스펙을 모두 구현한 것은 아니지만 기본적인 동작방식은 표현되어있다.
- 옵저버 패턴과 유사하다
```java
import java.util.Arrays;
import java.util.concurrent.Flow.Subscriber;
import java.util.concurrent.Flow.Publisher;
import java.util.concurrent.Flow.Subscription;

public class IntPublisher implements Publisher<Integer> {

  private Iterator<Integer> datasource;

  public IntPublisher() {
      this.datasource = Arrays.asList(1, 2, 3, 4, 5).iterator();
  }

  @Override
  public void subscribe(Subscriber<? super Integer> subscriber) {
      Subscription subscription = new Subscription() {
          @Override
          public void request(long n) {
              try {
                  while (n-- < 0) {
                    if (datasource.hasNext()) {
                        // 데이터가 있을 경우 onNext 호출
                        subscriber.onNext(datasource.next());
                    } else {
                        // 데이터가 더이상 없을 경우 onComplete 호출
                        subscriber.onComplete();
                        break;
                    }
                }
              } catch (Exception e) {
                  // 예외 발생 시 onError 호출
                  subscriber.onError(e);
              }
          }
      };
      subscriber.onSubscribe(subscription);
  }
  
  @Override
  public void cancel() {
  }
}

public class IntSubscriber implements Subscriber<Integer> {
    private Subscription subscription;
    
    @Override
    public void onSubscribe(Subscription subscription) {
        System.out.println("onSubscribe");
    	  this.subscription = subscription;
    	  // subscribe 완료 후 데이터 1개 요청
        subscription.request(1);
    }

    @Override
    public void onNext(Integer integer) {
        // 데이터 1개 받은 후 1개 요청
        System.out.println("onNext " + integer);
        this.subscription.request(1);
    }

    @Override
    public void onComplete() {
        System.out.println("onComplete");
    }

    @Override
    public void onError(Throwable throwable) {
        System.out.println("onError " + throwable.getMessage());
    }
}
```