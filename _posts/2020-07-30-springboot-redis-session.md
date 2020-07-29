---
title: Spring Boot Redis Session
tags: [springboot, dev]
comments: true
categories: [springboot, redis]
header:
  teaser: "/assets/images/redis.png"
---
본 문서에서는 Spring boot 환경에서 `Redis Session 연동하는 방법`에 대해서 설명하고 있습니다.

<br/>

## 1. Spring boot Redis Session



[Spring Boot Redis](/springboot/redis/springboot-redis/)를 참고하여 Redis 연동을 진행합니다.

<br/>

그 후, HttpSession 저장소로 Redis를 사용하기 위해서는<br/>

 `@EnableRedisHttpSession`를 사용하거나, 다음과 같이 `session.store-type`를 설정하면 됩니다.

<br/>

#### 1-1. /resources/application.yml 

```yaml
spring:
  profiles:
    active: default

  session:
    store-type: REDIS   # HAZELCAST, JDBC, MONGODB, NONE 사용 가능

  redis:
    host: 127.0.0.1
    port: 6379
---
```



<br/>

## 2. Spring Boot Redis Session 실습

#### 2-1. MainRestController.java

```java
package com.example.app.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import javax.servlet.http.HttpSession;
import java.util.Optional;
import static java.util.AbstractMap.SimpleEntry;

@RestController
public class MainRestController {
    @GetMapping(path = "/")
    public Object home(HttpSession session) {
        Integer counter = Optional.ofNullable(session.getAttribute("counter")).map(number -> Integer.valueOf((Integer)number)).orElse(0) + 1;
        session.setAttribute("counter", counter);
        return new SimpleEntry<>("counter", counter);
    }
}
```

<br/>

#### 2-3. CURL를 이용한 Session 테스트

한 SESSION을 유지하기 위해서, 첫번째 요청후 결과값에 포함된 SESSION 값을 이용하여, 다음 요청에 활용합니다. 

```sh
$ curl -vvv localhost:8080                                                                                 
* Rebuilt URL to: localhost:8080/                                                                          
*   Trying ::1...                                                                                          
* TCP_NODELAY set                                                                                          
*   Trying 127.0.0.1...                                                                                    
* TCP_NODELAY set                                                                                          
* Connected to localhost (127.0.0.1) port 8080 (#0)                                                        
> GET / HTTP/1.1                                                                                           
> Host: localhost:8080                                                                                     
> User-Agent: curl/7.55.1                                                                                  
> Accept: */*                                                                                              
>                                                                                                          
< HTTP/1.1 200                                                                                             
< Set-Cookie: SESSION=M2ZhODI0OWEtZTAzNC00ZTk4LWI5MWItZWYwMThkYTNjOTRm; Path=/; HttpOnly; SameSite=Lax     
< Content-Type: application/json                                                                           
< Transfer-Encoding: chunked                                                                               
< Date: Wed, 29 Jul 2020 22:48:53 GMT                                                                      
<                                                                                                          
{
    "counter":1
}
```

<br/>

그리고, 다음과 같이 동일한 세션인 경우, 카운터의 값이 1씩 증가함을 확인합니다.

```sh
$ curl -H "Cookie: SESSION=M2ZhODI0OWEtZTAzNC00ZTk4LWI5MWItZWYwMThkYTNjOTRm" localhost:8080
{
    "counter":2
}

$ curl -H "Cookie: SESSION=M2ZhODI0OWEtZTAzNC00ZTk4LWI5MWItZWYwMThkYTNjOTRm" localhost:8080
{
    "counter":3
}

$ curl -H "Cookie: SESSION=M2ZhODI0OWEtZTAzNC00ZTk4LWI5MWItZWYwMThkYTNjOTRm" localhost:8080
{
    "counter":4
}
```

