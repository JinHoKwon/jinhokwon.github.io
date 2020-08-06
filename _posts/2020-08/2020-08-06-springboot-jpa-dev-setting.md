---
title: SpringBoot ORM, JPA 개발환경구축
tags: [springboot, dev]
comments: true
categories: springboot
header:
  teaser: "/assets/images/springboot/springboot_logo.png"
---

본 문서에서는 SpringBoot 환경에서 JPA 개발환경 구축 과정에 대해서 소개하고 있습니다.




## 1. JPA 개발환경 구축

#### 1-1. MySQL 8 준비

```sh
# mkdir -p /tmp/docker/mysql8
# rm -rf /tmp/docker/mysql8/*
# docker pull mysql:8
# docker run --name mysql8 -p 3306:3306 -v /tmp/docker/mysql5:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=1111 -d mysql:8
# docker exec -i -t mysql8 mysql -u root -p 
```

<br/>

#### 1-2. 샘플 데이터베이스 생성

```sql
DROP DATABASE IF EXISTS test;
CREATE DATABASE test CHARACTER SET utf8 COLLATE UTF8_GENERAL_CI;
```

<br/>

#### 1-3. pom.xml

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
    <artifactId>spring-boot-hibernate-crud</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-boot-jpa</name>
    <description>Spring Boot Hibernate CRUD Example</description>

    <properties>
        <java.version>11</java.version>
        <jackson.version>2.11.0</jackson.version>
        <lombok.version>1.18.12</lombok.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
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
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
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
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>11</source>
                    <target>11</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

<br/>

#### 1-4. application.yml 

```yaml
spring:
  datasource:
    url: jdbc:mysql://mysql1.net:3306/test?serverTimezone=UTC&characterEncoding=UTF-8
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 1111

  jpa:
    database: mysql
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
    hibernate:
      ddl-auto: create
      naming-strategy: org.hibernate.cfg.ImprovedNamingStrategy
      default_batch_fetch_size: 30
      query:
        in_clause_parameter_padding: true
    show-sql: true
    generate-ddl: false
    properties:
      hibernate:
        format_sql: true

logging:
  level:
    org:
      hibernate:
        SQL: DEBUG
        type:
          descriptor:
            sql:
              BasicBinder: TRACE
```

`ddl-auto`로 설정 가능한 값에 대한 설명은 다음과 같습니다.

* **none** : 아무것도 실행하지 않는다 (대부분의 DB에서 기본값이다)
* **create-drop** : SessionFactory가 시작될 때 drop및 생성을 실행하고, SessionFactory가 종료될 때 drop을 실행한다 (in-memory DB의 경우 기본값이다)
* **create** : SessionFactory가 시작될 때 데이터베이스 drop을 실행하고 생성된 DDL을 실행한다
* **update** : 변경된 스키마를 적용한다
* **validate** : 변경된 스키마가 있다면 변경점을 출력하고 애플리케이션을 종료한다
* **enable_lazy_load_no_trans** : transaction이 아닌 환경에서도 lazy load가 가능하게 설정함. 



`in_clause_parameter_padding` : SQL의 IN 쿼리를 효과적으로 재사용 하기 위해서 사용하는 옵션으로, 반드시 true로 설정해야 함.

false로 설정하고 IN 쿼리가 늘어나는 경우에 Hibernate의 Execution Plan Cache가 효과적으로 재사용 되지 않기 때문에 OOM 증상이 발생할 수 있음.



<br/>

