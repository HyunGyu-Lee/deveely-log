---
layout: post
title: "Reactive Programming 2"
date:   2021-08-17 12:32:19
category: backend-reactive-programming
tags: [Reactive Programming]
comments: true
draft: false
---
이 포스트 시리즈는 Reactive Programming은 토비의 스프링 저자 이일민님의 리액티브 프로그래밍 유튜브 강좌를 공부하며 정리한 내용입니다. 

## 0. Tips
Stream 인터페이스에 iterate 라는 메소드가 있다.
이 메소드는 어떠한 데이터 스트림을 쉽게 만들어 낼 수 있는 메소드이다.

```java
public List<Integer> createSampleIntegerList(int count) {
    return Stream.iterate(1, e -> e + 1).limit(count).collect(Collectors.toList());
}
```

위 코드는 1 ~ 10까지 정수를 담은 리스트를 만든다.
핵심은 `iterate(시작값, 값의 변화 함수)` 메소드인데, 위 예시에서 시작값은 1, 변화는 1씩 증가시킨다라는 뜻이다.
그리고 `limit(10)`을 통해 10개만이라는 제한을 걸어주었다. (걸지 않으면 무한하게 증가하기 때문에 collect를 할 수 없다.)

종종 활용할 곳이 있을것 같아 기록해둔다.

## 1. Operators 직접 구현해보기
- Reactive Streams의 핵심 개념은 Publisher -> Data -> Subscriber의 흐름으로 데이터가 전달된다는 것이다.
- 아래에서 `<-` 방향으로의 흐름을 업스트림(Upstream)이라 하고 `->` 방향으로의 흐름을 다운스트림(Downstream)이라 한다.
- 그리고 데이터는 업스트림에서 다운스트림 방향으로 (`->`) 흘러간다.
```
Publisher -> Data -> Subscriber
                  <- subscribe(Subscriber)
                  -> onSubscribe(Subscription)
                  -> onNext
                  -> onNext
                  -> ...
                  -> onComplete                  
```
  
- Reactive Streams에서는 이 과정에서 Publisher -> [Data1] -> Operator -> [Data2] -> Operator2 -> [Data3] -> Subscriber 이런식으로 데이터를 가공하는 Operator를 적용할 수 있다.
- 아래와 같은 Publisher와 Subscriber가 있다고 해보자. 1부터 10까지 정수 데이터가 발생하고, Subscriber는 화면에 출력하고 프로그램은 종료된다. 

```java
public class PubSub {

    public static void main(String[] args) {
        Publisher<Integer> pub = iterPub(Stream.iterate(1, e -> e + 1).limit(10).collect(Collectors.toList()));
        pub.subscribe(logSub());
    }

    /**
     * Iterator의 데이터를 발생시키는 Publisher 
     * @param iter 데이터 소스
     * @return Publisher
     */
    private static Publisher<Integer> iterPub(Iterable<Integer> iter) {
        return new Publisher<>() {
            @Override
            public void subscribe(Subscriber<? super Integer> subscriber) {
                subscriber.onSubscribe(new Subscription() {
                    @Override
                    public void request(long n) {
                        // 이 예시 코드에선 Subscriber의 요청갯수인 'n' 인자에 대한 구현은 안되어있다.
                        try {
                            iter.forEach(subscriber::onNext);
                            subscriber.onComplete();
                        } catch (Exception e) {
                            subscriber.onError(e);
                        }
                    }

                    @Override
                    public void cancel() {
                        // Subscriber가 어떠한 경우로든 Publisher에게 데이터를 그만보내라고 요청하는것
                        // 그에 대비해 Publisher는 이 메소드안에 적절한 내용을 구현해두어야함
                        // 일반적으로 flag를 두고 request쪽에서 데이터를 더 보내지 않도록 함
                    }
                });
            }
        };
    }

    /**
     * Publisher에서 수신한 데이터를 출력하는 Subscriber
     * @return Subscriber
     */
    private static Subscriber<Integer> logSub() {
        return new Subscriber<>() {
            @Override
            public void onSubscribe(Subscription subscription) {
                System.out.println("onSubscribe");
                subscription.request(Long.MAX_VALUE);
            }

            @Override
            public void onNext(Integer integer) {
                System.out.println("onNext: " + integer);
            }

            @Override
            public void onError(Throwable throwable) {
                System.out.println("onError: " + throwable);
            }

            @Override
            public void onComplete() {
                System.out.println("onComplete");
            }
        };
    }

}
```

- 여기에서, 데이터를 변환하는 과정을 넣어본다면 아래와 같이 구현할 수 있다. 
- 마치 기존 Publisher를 한 겹 감싸고, 데이터를 전달하는 부분인 onNext 메소드에 특정 작업을 심은 새로운 Publisher를 생성하는 것이다.
- 아래 `mapPub` 함수에서 스트림의 흐름을 이해하기 쉽게 방향과 대조해보면
  - 인자로 전달된 publisher: 업스트림 (현재 스트림보다 상위에 있는 스트림)
  - 새로 생성되어 return되는 Publisher: 중개 스트림 (현재 스트림)
    - 이 안에서 구현하는 subscribe함수에 인자로 전달되는 subscriber: 다운스트림 (현재 스트림보다 하위에 있는 스트림)
  - publisher.subscribe 안에 새로 생성하는 Subscriber: 중개 스트림 (현재 스트림)
- 업스트림 -> 다운스트림으로의 흐름에 무언가 중개하는 새로운 흐름이 끼어든 것이다.

```java
public class PubSub {

    public static void main(String[] args) {
        Publisher<Integer> pub = iterPub(Stream.iterate(1, e -> e + 1).limit(10).collect(Collectors.toList()));
        Publisher<Integer> mapPub = mapPub(pub, e -> e * 10); // 기존 값에 10을 곱하는 함수 전달
        mapPub.subscribe(logSub());
    }

    private static Publisher<Integer> mapPub(Publisher<Integer> publisher, Function<Integer, Integer> mappingFunc) {
        return new Publisher<Integer>() {
            @Override
            public void subscribe(Subscriber<? super Integer> subscriber) {
            	// 인자로 전달된 기존 publisher.subscribe의 인자에 새 Subscriber를 생성하여 넘겨준다.
                // 이 때, 새 Subscriber에 기존 Subscriber의 기능들을 연결시키고, onNext에 데이터를 가공하는 기능만 부가적으로 추가해준다.
                publisher.subscribe(new Subscriber<>() {
                    @Override
                    public void onSubscribe(Subscription subscription) {
                        subscriber.onSubscribe(subscription);
                    }

                    @Override
                    public void onNext(Integer integer) {
                        // 이전 퍼블리셔에서 발생된 데이터에 'mappingFunc' 적용하여 전달
                        subscriber.onNext(mappingFunc.apply(integer));
                    }

                    @Override
                    public void onError(Throwable throwable) {
                        subscriber.onError(throwable);
                    }

                    @Override
                    public void onComplete() {
                        subscriber.onComplete();
                    }
                });
            }
        };
    }
}
```

- 만약 기존 스트림(Publisher) 내 모든 값을 합한 값을 발생시키는 Publisher를 구현한다면 아래와 같이 할 수 있다.

```java
public class PubSub {

    public static void main(String[] args) {
        Publisher<Integer> pub = iterPub(Stream.iterate(1, e -> e + 1).limit(10).collect(Collectors.toList()));
        Publisher<Integer> sumPub = sumPub(pub);
        sumPub.subscribe(logSub());
    }

    private static Publisher<Integer> sumPub(Publisher<Integer> publisher, Function<Integer, Integer> mappingFunc) {
        return new Publisher<Integer>() {
            @Override
            public void subscribe(Subscriber<? super Integer> subscriber) {
                publisher.subscribe(new Subscriber<>() {
                    // 업스트림에서 전달되는 값을 합해둘 멤버변수 선언
                    private int sum = 0;

                    @Override
                    public void onSubscribe(Subscription subscription) {
                        subscriber.onSubscribe(subscription);
                    }

                    @Override
                    public void onNext(Integer integer) {
                        // 업스트림에서 데이터가 전달되면 바로 전달하지 않고 값을 더해둠
                        this.sum += integer;
                    }

                    @Override
                    public void onError(Throwable throwable) {
                        subscriber.onError(throwable);
                    }

                    @Override
                    public void onComplete() {
                    	// 업스트림에서 데이터 전달이 끝나면 더해둔 값을 다운스트림으로 보냄
                        subscriber.onNext(sum);
                        // 그리고 현재 스트림은 끝났음을 통지
                        subscriber.onComplete();
                    }
                });
            }
        };
    }
}
```

- 만약 `mapPub` 메소드에 제네릭을 적용하면 아래와 같이 작성할 수 있다.
- T타입 데이터 흐름을 R타입 데이터 흐름으로 변경한다. 

```java
private static <T, R> Publisher<R> mapPub(Publisher<T> publisher, Function<T, R> mappingFunc) {
    return new Publisher<R>() {
        @Override
        public void subscribe(Subscriber<? super R> subscriber) {
            publisher.subscribe(new Subscriber<T>() {
                @Override
                public void onSubscribe(Subscription subscription) {
                    subscriber.onSubscribe(subscription);
                }

                @Override
                public void onNext(T integer) {
                    subscriber.onNext(mappingFunc.apply(integer));
                }

                @Override
                public void onError(Throwable throwable) {
                    subscriber.onError(throwable);
                }

                @Override
                public void onComplete() {
                    subscriber.onComplete();
                }
            });
        }
    };
}
```

## 2. Reactor 맛보기
- Reactor는 reactive streams 구현체중 하나로 Spring Webflux 엔진으로도 사용된다.
- Reactive Streams 표준을 아주 쉽게 다룰수 있게 해두었다.
- 디펜던시
```groovy
implementation 'io.projectreactor:reactor-core:3.4.9'
```
- Reactor를 사용한 심플한 예제를 하나 보자
- Flux는 Publisher의 구현체 중 하나이다. (데이터를 생성하는 역할이라는 뜻)
- create 메소드엔 데이터 생성하는 로직이 들어가는데, next, complete 등 마치 Publisher 의 subscribe 메소드를 구현할 때 하는 작업을 편리하게 할 수 있다.
- map 메소드와 같이 생성 -> 구독 사이에 여러 Operators를 체이닝 방식으로 쉽게 걸어줄 수 있다.
- 위에 Publisher, Subscriber 직접 생성하고 연결하는 코드들은 모두 내부에 감춰져있고 아래와 같은 간단한 코드로 하나의 흐름을 만들수있다.
```java
Flux.<Integer>create(e -> {
    e.next(1);
    e.next(2);
    e.next(3);
    e.complete();
})
.map(e -> e * 10)
.reduce(0, (a, b) -> a + b)        
.subscribe(s -> System.out.println(s));
```