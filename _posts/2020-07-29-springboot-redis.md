---
title: Spring Boot Redis
tags: [springboot, dev]
comments: true
categories: [springboot, redis]
header:
  teaser: "/assets/images/redis.png"
---
본 문서에서는 Spring boot 환경에서 `Redis 연동하는 방법`에 대해서 설명하고 있습니다.

<br/>

## 1. spring-boot-starter-data-redis

Spring boot 환경에서는 Redis 관련 라이브러리를 추상화하여 `spring-boot-starter-data-redis` 라이브러리로 제공하고 있으며,

spring-boot-starter-data-redis 를 사용하기 위해서는 pom.xml에 다음과 같이 라이브러리를 추가하면 됩니다.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

그 이후, Redis 서버와의 통신을 담당하는 `RedisConnectionFactory`와 Redis 명령을 수행하는 `RedisTemplate`을 Bean으로 생성하면 됩니다.

<br/>

<br/>

## 2. spring-boot-starter-data-redis 실습

<br/>

#### 2-1. RedisConfiguration.java

각자 환경에 맞게끔 Redis 접속 방법 (standalone, sentinel, socket, cluster) 및<br/>

Redis 서버의 IP 주소 적절히 변경합니다.

```java
package com.example.app.config;

import com.example.app.model.Box;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import org.apache.commons.pool2.impl.GenericObjectPoolConfig;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.data.redis.connection.*;
import org.springframework.data.redis.connection.lettuce.LettuceClientConfiguration;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.connection.lettuce.LettucePoolingClientConfiguration;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;

@Configuration
public class RedisConfiguration {
    @Value("${spring.redis.host:127.0.0.1}")
    private String redisHostName;

    @Value("${spring.redis.socket:/var/run/redis/redis.sock}")
    private String redisSocket;

    @Value("${spring.redis.password:}")
    private String redisPassword;

    @Value("${spring.redis.port:6379}")
    private int redisPort;

    @Value("${spring.redis.timeout:60}")
    private int  redisTimeOut;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();
        poolConfig.setMaxIdle(10);
        poolConfig.setMaxTotal(10);
        poolConfig.setMinIdle(2);

        LettuceClientConfiguration lettuceClientConfiguration =
                LettucePoolingClientConfiguration.builder().poolConfig(poolConfig)
                        .commandTimeout(Duration.ofSeconds(5)).build();

        return new LettuceConnectionFactory(redisStandaloneConfiguration(), lettuceClientConfiguration);
        //return new LettuceConnectionFactory(redisSentinelConfiguration());
        //return new LettuceConnectionFactory(redisSocketConfiguration());
        //return new LettuceConnectionFactory(redisClusterConfiguration());
    }

    private RedisSocketConfiguration redisSocketConfiguration() {
        RedisSocketConfiguration redisSocketConfiguration = new RedisSocketConfiguration(this.redisSocket);
        redisSocketConfiguration.setPassword(RedisPassword.of(redisPassword));
        return redisSocketConfiguration;
    }

    private RedisStandaloneConfiguration redisStandaloneConfiguration() {
        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration(redisHostName, redisPort);
        redisStandaloneConfiguration.setPassword(RedisPassword.of(redisPassword));
        return redisStandaloneConfiguration;
    }

    private RedisClusterConfiguration redisClusterConfiguration() {
        RedisClusterConfiguration redisClusterConfiguration = new RedisClusterConfiguration();
        redisClusterConfiguration.addClusterNode(new RedisClusterNode("192.168.56.21", 7001));
        redisClusterConfiguration.addClusterNode(new RedisClusterNode("192.168.56.21", 7002));
        redisClusterConfiguration.addClusterNode(new RedisClusterNode("192.168.56.21", 7003));
        redisClusterConfiguration.addClusterNode(new RedisClusterNode("192.168.56.21", 7004));
        redisClusterConfiguration.addClusterNode(new RedisClusterNode("192.168.56.21", 7005));
        redisClusterConfiguration.addClusterNode(new RedisClusterNode("192.168.56.21", 7006));
        return redisClusterConfiguration;
    }

    private RedisSentinelConfiguration redisSentinelConfiguration() {
        return new RedisSentinelConfiguration().master("mymaster").sentinel("redis.sentinel.net", 5000);
    }
    
    @Bean
    public RedisTemplate<String, Box> redisTemplate() {
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.registerModule(new JavaTimeModule());

        Jackson2JsonRedisSerializer<Box> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Box.class);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

        RedisTemplate<String, Box> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        return redisTemplate;
    }

    @Bean
    public StringRedisTemplate stringRedisTemplate() {
        StringRedisTemplate stringRedisTemplate = new StringRedisTemplate();
        stringRedisTemplate.setKeySerializer(new StringRedisSerializer());
        stringRedisTemplate.setValueSerializer(new StringRedisSerializer());
        stringRedisTemplate.setConnectionFactory(redisConnectionFactory());
        return stringRedisTemplate;
    }
}
```

<br/>

#### 2-2. SpringBootWebApplicationTests.java

그리고, 다음과 같은 테스트 코드를 작성하여 기능 동작 유/무를 확인합니다.

```java
package com.example.app;

import com.example.app.model.Box;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Order;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.test.context.junit.jupiter.SpringExtension;

import java.util.Arrays;
import java.util.Collection;
import java.util.List;

@Slf4j
@ExtendWith(SpringExtension.class)
@SpringBootTest
public class SpringBootWebApplicationTests {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Autowired
    private RedisTemplate<String, Box> redisTemplate;

    @Test
    @Order(1)
    public void valueCRUDTest() {
        ValueOperations<String, String> vos = this.stringRedisTemplate.opsForValue();
        String key = "myKey";
        String val = "myVal";

        vos.set(key, val);                              // create
        Assertions.assertEquals(val, vos.get(key));     // read myVal
        vos.set(key, val.toUpperCase());                // update
        Assertions.assertEquals(val.toUpperCase(), vos.get(key));       // read MYVAL
        this.stringRedisTemplate.delete(key);                           // delete
        Assertions.assertNull(vos.get(key));                            // read null
        Assertions.assertFalse(this.stringRedisTemplate.hasKey(key));   // exist false
    }

    @Test
    @Order(2)
    public void objectCRUDTest() {
        ValueOperations<String, Box> vos = this.redisTemplate.opsForValue();
        String key = "myObjectKey";

        vos.set(key, new Box("red", "001"));    // create
        Box value = (Box)vos.get(key);          // read
        Assertions.assertEquals("red", value.getName());
        Assertions.assertEquals("001", value.getCode());

        value.setName(value.getName().toUpperCase());   // update
        vos.set(key, value);

        value = (Box)vos.get(key);      // read
        Assertions.assertEquals("RED", value.getName());
        Assertions.assertEquals("001", value.getCode());

        this.redisTemplate.delete(key);       // delete
        Assertions.assertFalse(this.redisTemplate.hasKey(key));     // false
    }

    @Test
    @Order(3)
    public void multipleGetTest1() throws Exception {
        ValueOperations<String, String> vos = this.stringRedisTemplate.opsForValue();
        vos.set("11", "mbc");
        vos.set("09", "kbs");
        Collection<String> keys = Arrays.asList("11", "09");
        List<String> values = vos.multiGet(keys);
        Assertions.assertArrayEquals(values.toArray(), Arrays.asList("mbc", "kbs").toArray());
    }

    @Test
    @Order(4)
    public void multipleGetTest2() throws Exception {
        ValueOperations<String, Box> vos = this.redisTemplate.opsForValue();
        Box redBox = new Box("red", "001");
        Box greenBox = new Box("green", "002");

        vos.set(redBox.getCode(), redBox);
        vos.set(greenBox.getCode(), greenBox);

        Collection<String> keys = Arrays.asList(redBox.getCode(), greenBox.getCode());
        List<Box> values = vos.multiGet(keys);
        Assertions.assertArrayEquals(values.toArray(), Arrays.asList(redBox, greenBox).toArray());
    }
}
```



