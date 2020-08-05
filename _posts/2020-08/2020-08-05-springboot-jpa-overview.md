---
title: SpringBoot ORM, JPA, Hibernate
tags: [springboot, dev]
comments: true
categories: springboot
header:
  teaser: "/assets/images/springboot/springboot_logo.png"
---

본 문서에서는 객체와 DB간의 관계를 매핑시켜주는 ORM(Object-relational mapping) 에 대해서 소개하고 있습니다.

## 1. ORM(Object-relational mapping)

객체와 DB 간의 관계를 매핑시켜 주는 기술로,<br/>
생산성 및 유지보수가 수월하기 때문에 업계에서 많이 사용되는 표준 기술 스펙중의 하나입니다.<br/>

<br/>

![orm](/assets/images/springboot/jpa/orm.png)

<br/>

## 2. ORM 장점과 단점

#### 2-1. ORM 장점

- 객체 지향적인 코드로 직관적인 코드 작성이 가능
- 특정 DB에 대한 의존성이 줄어듬.
- 특정 객체의 CRUD와 같은 반복적인 SQL 작업을 ORM이 대신 진행해줌.
- SQL 쿼리에 익숙하지 않은 경우, 사용하기에 적합함.

<br/>

#### 2-2. ORM 단점

- 기존 SQL쿼리에 익숙할수록, ORM에 적응하는 과정이 불편하게 느낄수 있음.
- 객체간의 연결관계가 복잡할 경우, ORM에 적용하는 과정도 점점 더 난해함.
- 객체간의 연결관계를 주의 깊에 설계하지 않은 경우, 성능이 저하됨.
- DB 중심적으로 설계되어있거나, 프로시저가 많은 환경에는 ORM을 적용하기가 쉽지 않음. 
- 리포트 및 통계와 같은 대형 쿼리를 수행할 경우, ORM 만으로는 성능 튜닝의 한계가 있음. 
  이 경우, 배치성 쿼리는 ORM 대신, SQL 쿼리 구문을 직접 튜닝 및 실행.

<br/>

## 3. JPA (Java Persistence API)

Java 기반의 ORM 기술 표준 인터페이스이며, 해당 JPA를 구현한 기술이 Hibernate 입니다. 

즉, ORM > JPA > Hibernate

<br/>

## 4. JPA Annotation

#### 4-1. @Entity

엔티티임을 정의

<br/>

#### 4-2. @Table 

테이블 정의 

<br/>

#### 4-3. @Id

테이블의 기본키(PK)에 매핑

<br/>

#### 4-4. @Column 

클래스의 필드를 컬럼에 매핑

<br/>

#### 4-5. @GeneratedValue 

기본키 생성전략 설정 (기본값 : AUTO)
* GenerationType.IDENTITY : 기본 키 생성을 데이터베이스에 위임. (AUTO_INCREMENT)
* GenerationType.SEQUENCE : 데이터베이스의 Sequece Object를 사용. (AUTO_INCREMENT)

<br/>


## 5. JPA 개발환경 구축

#### 5-1. MySQL 8 준비

```sh
# mkdir -p /tmp/docker/mysql8
# rm -rf /tmp/docker/mysql8/*
# docker pull mysql:8
# docker run --name mysql8 -p 3306:3306 -v /tmp/docker/mysql5:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=1111 -d mysql:8
# docker exec -i -t mysql8 mysql -u root -p 
```

<br/>

#### 5-2. 샘플 데이터베이스 생성

```sql
DROP DATABASE IF EXISTS test;
CREATE DATABASE test CHARACTER SET utf8 COLLATE UTF8_GENERAL_CI;
```

<br/>

#### 5-3. pom.xml

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

#### 5-4. application.yml 

```yaml
spring:
  datasource:
    url: jdbc:mysql://mysql1.net:3306/test?serverTimezone=UTC&characterEncoding=UTF-8
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 1111

  jpa:
    hibernate:
      ddl-auto: create
      naming-strategy: org.hibernate.cfg.ImprovedNamingStrategy
    show-sql: true
    properties:
      hibernate:
        format_sql: true
```

`ddl-auto`로 설정 가능한 값에 대한 설명은 다음과 같습니다.
* **none** : 아무것도 실행하지 않는다 (대부분의 DB에서 기본값이다)
* **create-drop** : SessionFactory가 시작될 때 drop및 생성을 실행하고, SessionFactory가 종료될 때 drop을 실행한다 (in-memory DB의 경우 기본값이다)
* **create** : SessionFactory가 시작될 때 데이터베이스 drop을 실행하고 생성된 DDL을 실행한다
* **update** : 변경된 스키마를 적용한다
* **validate** : 변경된 스키마가 있다면 변경점을 출력하고 애플리케이션을 종료한다
* **enable_lazy_load_no_trans** : transaction이 아닌 환경에서도 lazy load가 가능하게 설정함. 

  

<br/>

## 6. JPA CRUD 

#### 6-1. Box.java

```java
package com.example.app.model;

import lombok.*;
import javax.persistence.*;

@Entity
@Table(name="box")
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PUBLIC)
@AllArgsConstructor
@ToString
public class Box {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column
    long id;

    @Column(name = "name", unique = false, length = 64)
    @NonNull
    String name;

    @Column(name = "code", unique = true, length = 64)
    @NonNull
    String code;

    @Builder
    public Box(String name, String code) {
        this.name = name;
        this.code = code;
    }
}
```

위와 같이 선언된 `Box` Entity는 다음과 같은 `CREATE TABLE`구문으로 매핑이 됩니다.

```sql
CREATE TABLE `box` (
	`id` BIGINT(19,0) NOT NULL AUTO_INCREMENT,
	`code` VARCHAR(64) NULL DEFAULT NULL COLLATE 'utf8_general_ci',
	`name` VARCHAR(64) NULL DEFAULT NULL COLLATE 'utf8_general_ci',
	PRIMARY KEY (`id`) USING BTREE,
	UNIQUE INDEX `UK_eaj689yduj53wefw3qtj6gdeb` (`code`) USING BTREE
)
COLLATE='utf8mb4_0900_ai_ci'
ENGINE=InnoDB
AUTO_INCREMENT=2
;
```

<br/>

#### 6-2. BoxRepository.java

```java
package com.example.app;

import com.example.app.model.Box;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface BoxRepository extends JpaRepository<Box, Long> {

}
```

<br/>

#### 6-3. JPA CRUD

```java
@Test
public void crudTest() throws Exception {
    Box box = new Box("red", "001");
    Box saveBox = boxRepository.save(box);
    Box readBox = boxRepository.findById(saveBox.getId()).get();
    readBox.setCode("002");
    Box updateBox = boxRepository.save(readBox);
    boxRepository.deleteById(updateBox.getId());
}

@Autowired
BoxRepository boxRepository;
```



<br/>

