---
title: Spring Boot REST
tags: [springboot, dev]
comments: true
categories: springboot
header:
  teaser: "/assets/images/springboot/springboot_logo.png"
---
## 1. REST(Representational State Transfer)

REST(Representational State Transfer)는 월드 와이드 웹과 같은 <br/>

분산 하이퍼미디어 시스템을 위한 소프트웨어 아키텍처의 한 형식이며,<br/>

이 용어는 로이 필딩(Roy Fielding)의 2000년 박사학위 논문에서 소개되었습니다.

<br/>

#### 1-1. REST의 구성요소

* 자원 (Resource) : URI
* 행위 (Verb) : Http(s) Method 
* 표현 (Representations)

<br/>

#### 1-2. REST Method


* GET : 자원 (Resource) 조회
* POST : 자원 (Resource) 생성
* PUT : 자원 (Resource) 업데이트
* DELETE : 자원 (Resource) 삭제

<br/>

#### 1-3. 응답코드

* 2xx : 클라어인트 요청이 성공적으로 수행됨
* 3xx : 클라이언트는 요청을 완료하기 위해 추가적인 행동을 취해야 함
* 4xx : 클라이언트의 잘못된 요청 또는 일시적인 서버 오류
* 5xx : 서버쪽 오류로 인한 상태코드
  

<br/>

#### 1-4. RESTful 이란?

<br/>

REST 개념이 잘 적용되었을 때, RESTful 하다라고 표현하며, <br/>
반대로, CRUD 와 같은 기능을 한 가지 HTTP METHOD로만 구현한 경우,<br/>
RESTful 하지 않다고 표현 하기도 합니다.

<br/>

<br/>

## 2. Spring boot REST CRUD

Spring boot 환경에서는 `spring-boot-starter-web` 라이브러리를 사용하면, <br/>

비교적 쉽게 REST API 개발이 가능합니다.

<br/>

#### 2-1. pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.0.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>spring-boot-rest</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-boot-rest</name>
    <description>Spring Boot REST Example</description>

    <properties>
        <java.version>11</java.version>
        <jackson.version>2.11.0</jackson.version>
        <lombok.version>1.18.12</lombok.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <!--
                junit-vintage-engine is included by default to allows easy migration from Junit 4 to Junit 5,
                by allowing both Junit 4 and Junit 5 based tests to run in parallel.
                You can exclude junit-vintage-engine if you do not have any Junit 4 based testcase in your application.
            -->
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>${jackson.version}</version>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>${jackson.version}</version>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
            <version>${jackson.version}</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.0</version>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>11</source>
                    <target>11</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```





<br/>

#### 2-2. SpringBootWebApplication.java

```java
package com.example.app;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.Setter;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.Banner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import javax.validation.constraints.NotEmpty;

@Slf4j
@RestController
@RequestMapping("label")
@SpringBootApplication
public class SpringBootWebApplication {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(SpringBootWebApplication.class);
        app.setBannerMode(Banner.Mode.OFF);
        app.run(args);
    }

    @Getter
    @Setter
    @AllArgsConstructor
    class Label {
        @NotEmpty(message = "name requires an argument.")
        String name;
    }

    @GetMapping("/")
    public ResponseEntity getLabel() {
        return (this.label == null) ? ResponseEntity.badRequest().build() : ResponseEntity.ok(this.label);
    }

    @PostMapping("/")
    public ResponseEntity createLabel(@RequestParam(required = true) @NotEmpty String name) {
        this.label = new Label(name);
        return ResponseEntity.ok().build();
    }

    @PutMapping("/")
    public ResponseEntity updateLabel(@RequestParam(required = true) @NotEmpty String name) {

        if (this.label == null) {
            return ResponseEntity.badRequest().build();
        }

        this.label.setName(name);
        return ResponseEntity.ok().build();
    }

    @DeleteMapping("/")
    public ResponseEntity deleteLabel() {
        this.label = null;
        return ResponseEntity.ok().build();
    }

    Label label = new Label("default");
}
```



<br/>

#### 2-3.  자원 읽기 (GET) 테스트

```sh
# curl -v -X GET localhost:8088/label/
{
	"name":"default"
}
```

<br/>

#### 2-4. 자원 생성 (POST) 테스트

```sh
# curl -v -X POST localhost:8088/label/?name=mylabel
< HTTP/1.1 200
< Content-Length: 0
< Date: Wed, 22 Jul 2020 04:41:36 GMT
<
* Connection #0 to host localhost left intact

# curl -v -X GET localhost:8088/label/
{
	"name":"mylabel"
}
```



<br/>

#### 2-5. 자원 수정 (PUT) 테스트

```sh
# curl -v -X PUT localhost:8088/label/?name=mylabel2
< HTTP/1.1 200
< Content-Length: 0
< Date: Wed, 22 Jul 2020 04:42:13 GMT
<
* Connection #0 to host localhost left intact

# curl -v -X GET localhost:8088/label/
{
	"name":"mylabel2"
}
```



<br/>

#### 2-6. 자원 삭제 (DELETE) 테스트

```sh
# curl -v -X DELETE localhost:8088/label/
< HTTP/1.1 200
< Content-Length: 0
< Date: Wed, 22 Jul 2020 04:42:40 GMT
<
* Connection #0 to host localhost left intact

# curl -v -X GET localhost:8088/label/
< HTTP/1.1 400
< Content-Length: 0
< Date: Wed, 22 Jul 2020 04:44:32 GMT
< Connection: close
<
* Closing connection 0
```











