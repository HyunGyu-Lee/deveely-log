---
title: Java Dynamic Proxy
date: 2016-11-22 17:18:21
category: backend-java
draft: false
---
#### Dynamic Proxy란
메소드 인터셉팅을 통해 부수적인 코드를 삽입할 수 있는 기술, Spring AOP에서도 사용한다.  
원래객체에 부수적인코드가 합쳐진 새로운 객체가 프록시 객체가 된다.  

###### 1. Proxy로 사용할 클래스에 대한 인터페이스를 작성한다.
```java
class interface Computer {

    public void boot();

    public void prepareGUI();

}
```

###### 2. 정의한 인터페이스를 구현
```java
class ComputerImpl implements Computer {

    @Override
    public void boot() {
        System.out.println("현재 부팅이 진행중입니다...");
    }

    @Override
    public void prepareGUI() {
        System.out.println("현재 GUI를 로딩중입니다...");
    }

}
```

###### 3. InvocationHandler를 통해 프록시 기능 구현
- 이 예시에서는 프록시 적용 대상객체의 메소드가 실행될 때 로깅을 수행하는 프록시 기능 구현

```java
class MyLogger implements InvocationHandler {
    // 실제 프록시가 적용될 대상 객체
    private Object realObject;

    public MyLogger(Object realObject) {
        this.realObject = realObject;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object result = null;

        long start = System.currentTimeMillis();

        // 아래 코드 실행 시 실제 대상객체의 메소드가 실행된다.
        result = method.invoke(realObject, args);

        long end = System.currentTimeMillis();
        System.out.println("작업명 : "+method.getName()+", 소요시간 : "+(end-start)+"ms");

        return result;
    }

}
```

###### 4. 프록시 적용
- computer는 일반 객체(프록시 적용 대상객체)이고, computerProxy는 computer에 로깅기능이 적용된 프록시 객체이다.

```java
public class Application {

    public static void main(String[] args) {
        Computer computer = new ComputerImpl();    // 프록시가 적용되지 않은 객체
        Class<?> type = ComputerImpl.class;        // 프록시가 적용될 객체의 타입
        Computer computerProxy = (Computer) Proxy.newProxyInstance(type.getClassLoader(),
                                                                   type.getInterfaces(),
                                                                   new MyLogger(new ComputerImpl()));
        computerProxy.boot();
        computerProxy.prepareGUI();
    }

}
```

위와같이 하면 boot, prepareGUI 메소드 실행 시 MyLogger에 구현한 대로 소요시간이 찍히는 것을 확인할 수 있다.

주의할 점은 프록시를 적용할 객체, 즉 ComputerImpl과 같은 realObject 역할의 객체 내부에서

자신의 메소드를 호출하는 것은 프록시로 잡히지 않는다.

예를들어

```java
computerProxy.boot(); => 프록시 적용  
computerProxy.prepareGUI(); => 프록시 적용
```

하지만

```java
public void boot() {
    System.out.println("현재 부팅이 진행중입니다...");
    prepareGUI();    => 프록시 미적용
}
```

이유는 아직까지 모르겠다. 알면 적어야지..
