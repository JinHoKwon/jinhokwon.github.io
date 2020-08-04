---
title: Spring Boot Profile
tags: [springboot, dev]
comments: true
categories: springboot
header:
  teaser: "/assets/images/springboot/springboot_logo.png"
---

properties 방식에는 서로 다른 profile을 구분하고자 할 때, 다음과 같이 사용하였습니다.

```sh
application-dev.properties   # 개발 환경인 경우
application-stage.properties # 스테이징 환경인 경우
application-real.properties  # 운영 환경인 경우
```

반면에, yaml 문서에서 제공하는 문서 나눔 구분자 `---`를 활용하면, <br/>

한개의 application.yaml 파일에서 여러개의 profiles 설정할 수 있습니다.

```yaml
spring:
  profiles:
    active: default

config:
  location: default
  server: 127.0.0.1
---
spring:
  profiles: local

config:
  location: local
  server: 127.0.0.1
---
spring:
  profiles: dev

config:
  location: dev
  server: 192.168.56.101
---
spring:
  profiles: real

config:
  location: real
  server: 20.10.2.102
```

그리고, 특정 profiles을 활성화 하기 위해서는 `-Dspring.profiles.active=dev`를<br/>

Java 실행할 때 파라미터로 전달하면 됩니다.

<br/>

<br/>

## Spring boot profile test

#### 우선 다음과 같이 간단한 샘플을 준비합니다.

```java
package com.example.app;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.Banner;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;

@Slf4j
@SpringBootApplication
public class SpringBootConsoleApplication implements CommandLineRunner {

    @Value("${config.location}")
    String location;

    @Value("${config.server}")
    String server;

    @Value("${config.unknown:default value}")
    String unknown;

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(SpringBootConsoleApplication.class);
        app.setBannerMode(Banner.Mode.OFF);
        app.run(args);
    }

    @Override
    public void run(String... args) throws Exception {
        log.info("location : {}, server : {}, unknown : {}", location, server, unknown);
    }
}
```

<br/>

#### profiles 설정없이 실행한 경우

```sh
# java -jar target\spring-boot-profile-0.0.1-SNAPSHOT.jar
The following profiles are active: default
location : default, server : 127.0.0.1, unknown : default value
```

<br/>

#### profiles=dev로 설정한 경우

```sh
# java -Dspring.profiles.active=dev -jar target\spring-boot-profile-0.0.1-SNAPSHOT.jar
The following profiles are active: dev
location : dev, server : 192.168.56.101, unknown : default value
```

<br/>

#### profiles=real로 설정한 경우

```sh
# java -Dspring.profiles.active=real -jar target\spring-boot-profile-0.0.1-SNAPSHOT.jar
The following profiles are active: real
location : real, server : 20.10.2.102, unknown : default value
```

<br/>