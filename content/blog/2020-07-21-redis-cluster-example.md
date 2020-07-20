---
layout: post
title: "Spring Boot Redis Connection Pool + Cluster 설정"
date:   2020-07-21 13:04:19
category: redis
tags: [redis,in-memory]
comments: true
draft: false
---

Spring Boot에서 Redis Cluster에 접속하는 방법을 정리한다.

## Spring Boot 설정

```yaml
spring:
    redis:
        cluster:
            nodes:
                1.1.1.1:1234
                1.1.1.2:1234
                1.1.1.3:1234    
```

```java
@Configuration
public class RedisConfiguration {
    
    @Value("${spring.redis.cluster.nodes}")
    private String[] clusterNodes;
    
    /**
     * Jedis Connection Pool
     */
    @Bean
    public JedisPoolConfig jedisPoolConfig() {
        return new JedisPoolConfig();
    }
    
    /**
     * Redis Connection Factory (커넥션풀링 + 클러스터링)
     */
    @Bean
    public RedisConnectionFactory redisConnectionFactory(JedisPoolConfig jedisPoolConfig) {
        RedisClusterConfiguration redisClusterConfigurations = new RedisClusterConfiguration(clusterNodes);
        return new JedisConnectionFactory(redisClusterConfigurations, jedisPoolConfig);
    }
    
    /**
     * 어플리케이션에서 사용할 RedisTemplate 설정
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        redisTemplate.setDefaultSerializer(new StringRedisSerializer());
        return redisTemplate;
    }

    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        return new StringRedisTemplate(redisConnectionFactory);
    }
}
```

이 후 사용은 보통 RedisTemplate 혹은 Repository 사용하듯이 사용하면 된다.

## 참고 (Redis 클러스터 모드로 접속)
-c 옵션으로 클러스터 모드로 접속
```sh
redis-cli -c -h 1.2.3.4 -p 1234
```