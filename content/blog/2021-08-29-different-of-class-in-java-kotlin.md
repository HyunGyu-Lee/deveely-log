---
layout: post
title: "Kotlin 클래스 주요 개념 with Java class와 차이점"
date:   2021-08-29 01:37:22
category: backend-kotlin
tags: [Kotlin]
comments: true
draft: false
---
# 클래스 정의에 대한 Java와 Kotlin 차이점
이 포스트에서는 코틀린에서의 클래스가 자바와 어떻게 다른지 대략적으로 정리한 내용을 다룹니다.
공부하면서 정리한 포스트이기에 잘못된 내용이나 부족한 부분이 있을 수 있습니다.
댓글로 일러주시면 감사하겠습니다 :)

## 1. class 키워드에 대한 차이점
기본적으로 자바에서는 `class` 키워드를 통해 클래스를 정의합니다.

```java
class SomeClass {
    // 속성, 메서드 선언
}
```

코틀린에서는 이렇게 선언하면 기본적으로 클래스를 포함한 모든 멤버가 `final`로 정의됩니다. (상속 불가)
```java
class SomeClass {
    // 속성, 메서드 선언
}
```

때문에 코틀린에는 상속 및 재정의가 가능한 요소로 만들어주는 `open`이라는 키워드가 있습니다.
클래스, 속성, 메서드 모든 곳에 사용이 가능합니다.

```java
// 상속 가능
open class SomeClass {
    // final로 선언되어 상속받는 클래스에서 재정의 불가
    fun someFunction(): Int {
        return 1
    }

    // 상속받는 클래스에서 재정의 가능
    open fun otherFunction(): Int {
        return 2
    } 
}

// 이렇게 open 클래스인 SomeClass를 상속받을 수 있다.
class ExtendSomeClass(): SomeClass() {
    override fun otherFunction(): Int {
        return 4
    }
}
```

이는 추상 클래스에서도 동일하게 적용됩니다.
```java
abstract class Machine(val name: String) {    
    // 반드시 재정의해야하는 메서드
    abstract fun connect()

    // 선택적으로 재정의할 수 있는 메서드
    open fun prepare() {
        println("Prepare machine")
    }

    // 재정의 불가
    fun start() {
        connect()
        parepare()
        println("Start machine - $name")
    }
}

class LocalMachine(name: String): Machine(name) {
    override fun connect() {
        println("Connect machine - $name")
    }

    override fun prepare() {
        println("Prepare machine on local")
    }
}
```

요약해보자면
- Kotlin에서 `class` 키워드는 기본적으로 멤버들을 final로 정의된다.
- `open` 키워드를 사용하면 각 멤버 (클래스, 속성, 메서드)의 상속 및 재정의 여부를 제어할 수 있다.


## 2. data class
data class는 용어 그대로 데이터를 보관하는 목적의 클래스들을 정의할 때 사용합니다.
`data` 키워드를 클래스 선언 앞에 붙혀주면 `toString()`, `hashCode()`, `equals()`, `copy()` 메소드를 자동으로 생성해줍니다.

```java
data class Machine(val name: String, val version: Long)

fun main(args: Array<String>) {
    val aMachine = Machine("A-Machine", 1)
    println("machine info ${aMachine}")
}
```

여기서 Data Class인 Machine를 자바 코드로 변환해보면 아래와 같습니다. (Intellij Decompile 기능 사용)
Lombok의 `@Data`, `@Getter`, `@EqaulsAndHashCode`, `@ToString` 을 `data` 키워드 멋지게 해주는 것 같습니다.
```java
public final class Machine {
   @NotNull
   private final String name;
   private final long version;

   @NotNull
   public final String getName() {
      return this.name;
   }

   public final long getVersion() {
      return this.version;
   }

   public Machine(@NotNull String name, long version) {
      Intrinsics.checkNotNullParameter(name, "name");
      super();
      this.name = name;
      this.version = version;
   }

   @NotNull
   public final String component1() {
      return this.name;
   }

   public final long component2() {
      return this.version;
   }

   @NotNull
   public final Machine copy(@NotNull String name, long version) {
      Intrinsics.checkNotNullParameter(name, "name");
      return new Machine(name, version);
   }

   // $FF: synthetic method
   public static Machine copy$default(Machine var0, String var1, long var2, int var4, Object var5) {
      if ((var4 & 1) != 0) {
         var1 = var0.name;
      }

      if ((var4 & 2) != 0) {
         var2 = var0.version;
      }

      return var0.copy(var1, var2);
   }

   @NotNull
   public String toString() {
      return "Machine(name=" + this.name + ", version=" + this.version + ")";
   }

   public int hashCode() {
      String var10000 = this.name;
      return (var10000 != null ? var10000.hashCode() : 0) * 31 + Long.hashCode(this.version);
   }

   public boolean equals(@Nullable Object var1) {
      if (this != var1) {
         if (var1 instanceof Machine) {
            Machine var2 = (Machine)var1;
            if (Intrinsics.areEqual(this.name, var2.name) && this.version == var2.version) {
               return true;
            }
         }

         return false;
      } else {
         return true;
      }
   }
}
```

근데 자바 코드를 보다보면 `component1()`, `component2()` 이라는 정의한 적이 없는 메소드가 생성 되어있는데요.
이는 Kotlin의 Destructuring Declaration (구조분해할당)과 관련이 있는데요.

간단하게 말해 아래와 같이 클래스 내부 속성을 여러 변수로 나누어 할당할 수 있는 Kotlin의 기능입니다.
```java
val machine = Machine("My Machine", 1)
val (name, version) = machine
println("name: $name, version: $version")
```

이를 자바로 표현하면 아래와 같이 되겠죠
```java
Machine machine = new Machine("My Machine", 1);
String name = machine.component1();
long version = machine.component2();
System.out.println(String.format("name: %s, version: %d", name, version));
```

이렇기 때문에 `data class`에서는 component1, component2 같은 메소드를 자동으로 생성하는 것입니다.
이는 일반 클래스에서도 가능합니다만 `componentN` 메소드들을 직접 구현해주어야 합니다!

요약해보자면
- Kotlin에서는 데이터 보관 목적의 클래스를 정의하기 위한 Data Class를 지원하며 키워드는 `data class` 이다.
- Data Class는 `toString()`, `hashCode()`, `equals()`, `copy()`를 자동으로 생성해준다.
- 구조분해할당을 위한 `componentN()` 메서드를 자동으로 생성해준다.

## 3. object 클래스
Kotlin에서 `class` 대신 `object` 키워드를 통해 클래스를 정의할 수 도 있는데요.
이 키워드는 해당 클래스를 싱글톤으로 만들어주는 기능을 합니다.

```java
object MachineFactory {
    val machines = mutableListOf<Machine>()
    fun createMachine(name: String): Machine {
        val machine = Machine(name)
        machines.add(machine)
        return machine
    }
}

class Machine(val name: String)
```

이렇게 `object` 키워드만 사용했을뿐 기존 자바에서 싱글톤을 구현하기위해 했던 부수적인 코드들은 하나도 없습니다.
하지만 아래와같이 `getInstance()` 메소드는 없지만 싱글톤 클래스로 사용 / 동작하게 됩니다.
```java
fun main(args: Array<String>) {
    val aMachine = MachineFactory.createMachine("A-Machine")
    println(MachineFactory.machines.size)
    val bMachine = MachineFactory.createMachine("B-Machine")
    println(MachineFactory.machines.size)
}
```

이를 자바코드로 변환해보면 아래와 같이 `INSTANCE`라는 변수가 `static` 블록에서 한번 초기화 되며 `MachineFactory.INSTANCE`와 같이 해당 객체만 참조됨을 알 수 있습니다.
```java
public final class MachineFactory {
   @NotNull
   private static final List machines;
   @NotNull
   public static final MachineFactory INSTANCE;

   @NotNull
   public final List getMachines() {
      return machines;
   }

   @NotNull
   public final Machine createMachine(@NotNull String name) {
      Intrinsics.checkNotNullParameter(name, "name");
      Machine machine = new Machine(name);
      machines.add(machine);
      return machine;
   }

   private MachineFactory() {
   }

   static {
      MachineFactory var0 = new MachineFactory();
      INSTANCE = var0;
      boolean var1 = false;
      machines = (List)(new ArrayList());
   }
}

public static final void main(@NotNull String[] args) {
    Intrinsics.checkNotNullParameter(args, "args");
    Machine aMachine = MachineFactory.INSTANCE.createMachine("A-Machine");
    int var2 = MachineFactory.INSTANCE.getMachines().size();
    boolean var3 = false;
    System.out.println(var2);
    Machine bMachine = MachineFactory.INSTANCE.createMachine("B-Machine");
    int var6 = MachineFactory.INSTANCE.getMachines().size();
    boolean var4 = false;
    System.out.println(var6);
}
```

참고로 Kotlin에서는 `static` 키워드가 없는대신 `companion object`라는 키워드가 있는데요.
아래와 같이 클래스 내에 `companion object` 블록 내에 변수와 함수를 선언하면, 이것들이 static 멤버로 동작하게 됩니다.
```java
class Config {
    companion object {
        var PREFIX = "config."
        fun getConfig(name: String): String {
            // do something
        }
    }
}

fun main(args: Array<String>) {
    println(Config.PREFIX)
    val someConfigValue = Config.getConfig("some-config")
}
```


## 4. 자바 클래스와 코틀린 클래스의 멤버 가시성 차이 (Visibility)
코틀린과 자바는 멤버 가시성에도 차이가 있는데요.
큰 차이점으로는 
1. 기본 접근제어자(아무것도 적지 않을경우)는 `public` 이다.
2. 패키지 접근 제어성이 없는 대신 모듈에 대한 접근 제어성이 있다.
프로젝트 하위 패키지 상위 개념으로, 한꺼번에 컴파일되는 코틀린 파일들의 모임이다.

- 자바
  - public: 모든 곳에서 접근 가능
  - protected: 같은 패키지 내이거나 파생된 클래스에서 접근 가능
  - default: 같은 패키지 내에서만 접근 가능 (기본)
  - private: 같은 클래스 내에서만 접근 가능
- 코틀린
  - public: 모든 곳에서 접근 가능 (기본)
  - internal: 같은 모듈 내에서만 접근 가능
  - protected: 해당 클래스와 해당 클래스를 상속받은 클래스에서만 접근 가능. 자바와 달리 패키지가 같다고해서 접근할 수 없음
  - private: 같은 클래스 안에서만 접근 가능


## 참고
- https://velog.io/@conatuseus/Kotlin-%ED%82%A4%EC%9B%8C%EB%93%9C-%EC%A0%95%EB%A6%AC-open-internal-companion-data-class-%EC%9E%91%EC%84%B1%EC%A4%91
- https://codechacha.com/ko/data-classes-in-kotlin/
- https://codechacha.com/ko/kotlin-object-vs-class/
