---
layout: post
title: "Spring Boot에서 Redis 활용"
date:   2020-07-17 13:04:19
category: redis
tags: [redis,in-memory]
comments: true
draft: false
---

## 설정

* 의존성 추가

``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

* 설정 클래스 작성
아래와 같이 설정하면 default 설정에 의해 localhost:6379 로 연결합니다.
변경하려면 설정파일에 `spring.redis.host`, `spring.redis.port`로 지정할 수 있습니다.
Key / Value Serializer를 설정해주는 이유는 RedisTemplate에서 Spring \~ Redis간 데이터 직, 역직렬화 시 사용하는 방식이 Jdk 직렬화 방식이기 때문입니다. 동작에는 문제가 없지만 redis-cli를 통해 직접 데이터를 보려고 할 때 알아볼수 없는 형태로 출력되기 때문에 Serializer를 변경해준 것입니다. [참고 링크](https://stackoverflow.com/questions/31608394/get-set-value-from-redis-using-redistemplate)

한글의 경우 이 설정으로도 깨질수가 있는데, cli 접속 시 `redis-cli --raw` 이렇게 옵션을 주고 접속하면 정상적으로 볼 수 있습니다.

```java
@Configuration
public class RedisConfiguration {

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        // 사용하는 Redis Client 팩토리 지정
        // Pool 사용 시 이곳에서 설정 가능
        return new JedisConnectionFactory(); 
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());
        // Hash Operation 사용 시
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(new StringRedisSerializer());
        
        // 혹은 아래 설정으로 모든 Key / Value Serialization을 변경할 수 있음
        redisTemplate.setDefaultSerializer(new StringRedisSerializer());
        
        return redisTemplate;
    }
}
```

## 사용
### 1. RedisTemplate 사용한 방법
```java
@Service
public RedisTestService {
    
    // 기본 
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * String Operation의 set 커맨드
     * @param key Redis Key
     * @param value key에 저장될 value
     */
    public void setStringValue(String key, String value) {
        redisTemplate.opsForValue().set(key, value);
    }
    
    /**
     * String Operation의 get 커맨드
     * @param key Redis Key
     * @param value key에 저장될 value
     */
    public String getStringValue(String key) {
        return (String) redisTemplate.opsForValue().get(key);
    } 
}
```

RedisTemplate의 `opsFor*` 메소드들은 특정 컬렉션의 커맨드(Operation)을 호출할 수 있는 기능을 모아둔 
`*Operations` 인터페이스를 반환합니다.


| 메소드명 | 반환 오퍼레이션 | 관련 Redis 자료구조 |
|:---:|:---:|:---:|
| opsForValue() | ValueOperations | String |
| opsForList() | ListOperations | List |
| opsForSet() | ListOperations | List |
| opsForList() | SetOperations | Set |
| opsForZSet() | ZSetOperations | Sorted Set |
| opsForHash() | HashOperations | Hash |

여기서 이 `*Operations`는 아래와 같은 방식으로도 주입받을 수 있습니다.
이런 경우 프레임워크에서 자동으로 판단하여 주입해준다고 합니다.
> For cases where a certain template view is needed, declare the view as a dependency and inject the template: the container will automatically perform the conversion eliminating the opsFor[X] calls:
```java
@Service
public RedisTestService {
    
    // 기본 
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    // 특정 오퍼레이션 직접 주입
    @Resource(name ="redisTemplate")
    private ValueOperations<String, Object> valueOps;
}
```

또한, Key Bound Operation을 제공하는데, 이건 특정 Redis Key 한정으로 오퍼레이션을 수행할 수 있는 인터페이스입니다. 즉, `redisTemplate.opsForValue().set("key~~", "string value")` 이렇게 할 것을 `redisTemplate.boundValueOps("key~~").set("string value")` 이렇게 할 수 있습니다.

추가로, `StringRedisTemplate` 라는 구현체도 있습니다.
Serializer가 String으로 설정되어있는 `RedisTemplate<String, String>`입니다.

이렇게 여러가지 방식이 있으니 필요에 따라서 사용하면 좋을듯 합니다.

### 2. Spring Data CrudRepository 를 통한 방법
JPA 사용해 RDB와 통신할 때 `org.springframework.data.repository` 밑에 있는 Repository 인터페이스를 상속받으면 findAll, findById, save 등 자동으로 여러 기능들을 바로 사용할 수 있듯이 Redis 또한 이 방식을 사용할 수 있습니다. (Redis의 Hash 자료구조 한정)

1. 엔티티(모델) 클래스 작성
- `@RedisHash` 어노테이션의 **"testModel"** 을 Redis 키 prefix로 사용합니다.
- `@Id`는 JPA와 동일한 역할을 수행합니다. **"testModel:{id}"** 의 id 위치에 자동 generate 값이 들어갑니다.
```java
@RedisHash("testModels")
public class TestModel {
    @Id
    private Long id;
    
    private String name;
    
    private int age;
    
    // getter, setters, ...
}
```

2. Repository 작성
```java
public interface TestModelRedisRepository extends CrudRepository<TestModel, Long> {
}
```

3. 사용
간단합니다. JPA에서 save를 호출하면 insert 문이 호출되듯, hmset, hset 등이 자동으로 호출됩니다.
```java
@Service
public class RedisTestService {

    @Autowired
    private TestModelRedisRepository testModelRedisRepository;

    public void addModel(String name, int age) {
        TestModel testModel = createModel(name, age);
        testModelRedisRepository.save(testModel);
    }

}
```

redis-cli에서 `keys *`로 확인해보면 추가된 것이 확인됩니다.
```
> keys *
$ testModels
$ testModels:1564644222311766174
```

<br>
`hgetall testModels:1564644222311766174`로 값이 잘 들어갔는지 확인해보면
```sh
_class
com.testprj.entity.redis.TestModel
id
1564644222311766174
name
이현규
age._class
java.lang.Integer
age
99
```

위와같이 우리가 저장하고자 하는 실제 필드 (id, name, age)와 Java Type (Redis - Spring Boot간 매핑에 사용되는 듯한)이 저장되있음을 확인할 수 있습니다. 

이외에 `findOne`, `findAll` 또한 모두 잘 동작합니다.

## 참고
https://stackoverflow.com/questions/31608394/get-set-value-from-redis-using-redistemplate
https://yookeun.github.io/java/2017/05/21/spring-redis/