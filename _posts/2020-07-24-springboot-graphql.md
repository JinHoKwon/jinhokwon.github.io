---
title: Spring Boot GraphQL
tags: [springboot, dev]
comments: true
categories: springboot
header:
  teaser: "/assets/images/springboot/springboot_logo.png"
---
## 1. GraphQL



REST 방식의 단점을 보완하고자 Facebook에서 공개한 쿼리 언어이며,<br/>

다음과 같은 특징들이 있습니다.

<br/>

#### 1-1. 필요한 필드만 요청할 수 있습니다.

가령, 다음과 같이 Box 라는 클래스가 있다고 가정을 해봅니다.

```java
class Box {
    String name;
    String color;
    String address;
    int    price;
    ...    etc        
}
```

그리고, REST 방식으로 Box 에 대한 요청을 한 경우, Box 에 대한 모든 필드값이 JSON 응답값으로 포함되지만,<br/>

GraphQL 방식으로 Box에 대한 요청을 한 경우, 

다음과 같이 필요한 필드만 요청하고, 해당 필드에 대한 정보만 응답받을 수 있습니다.

```json
# 요청
{
    box {
        name
        color
    }
}

# 응답
{
    "box" : {
        "name" : "gift",
        "color" : "green"
    }
}
```

반면에,  모든 필드를 한꺼번에 조회할 수 있는 방법은 없기 때문에, 

GraphQL 요청시 응답받아야 하는 모든 필드를 일일히 설정해 주어야 합니다.

```
Unfortunately what you'd like to do is not possible. 
GraphQL requires you to be explicit about specifying which fields you would like returned from your query.
```

https://stackoverflow.com/questions/34199982/how-to-query-all-the-graphql-type-fields-without-writing-a-long-query

<br/>

#### 1-2. 각 필드는 데이터 타입을 설정할 수 있습니다.

가령, 다음과 같이 Box 클래스에 대한 데이터 타입을 지정하여,<br/>

좀 더 명확한 통신을 가능하게 합니다.

```graphql
type Box {
    name: String!
    color: String!
    address: String
    price: Int 
}
```

이 때, 타입명 뒤에 `!`는  null을 허용하지 않겠다는 의미입니다.

<br/>

#### 1-3. GraphQL Schema 

GraphQL Schema에는 앞에서 설명했던 데이터 타입을 지정하는 것 뿐만 아니라,<br/>
몇 가지 지켜야 할 규칙들이 있습니다.<br/>

* Schema 확장자 : 모든 GraphQL Schema 의 확장자는 *.graphqls 로 설정되어야 합니다.
* Query : 읽기 연산을 정의함
* Mutation : 쓰기 연산을 정의함
* type : 응답값을 정의함
* input : 파라미터값을 정의함
* type, input 의 혼용은 불가능 함 : 2개의 값이 정확히 동일해도 각각 정의 내려야 함.
* 타입명 뒤에 `!` : null 값을 허용하지 않겠다는 의미임

<br/>



#### 1-4. GraphQL Schema 예제

```graphql
type Box {
   name: String!
   color: String!
   address: String
   price: Int
}

input BoxParam {
   name: String!
   color: String!
   address: String
   price: Int
}

type Query {
    getBox(name: String!) : Box!
    existBox(name: String!) : Boolean
    countBox: Int
}

type Mutation {
    putBox(boxParam: BoxParam) : Boolean
    delBox(name: String!) : Boolean
}
```



<br/>

<br/>







## 2. Spring Boot GraphQL

스프링 부트 환경에서도 GraphQL 기능을 지원하고 있습니다.

Query 와 관련된 기능은 `GraphQLQueryResolver` 를 통해서 제공되고,

Mutation 와 관련된 기능은 ` GraphQLMutationResolver`을 통해서 제공되고 있습니다.



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
    <artifactId>spring-boot-web-graphql</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-boot-web-graphql</name>
    <description>Spring Boot Web GraphQL Example</description>

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
            <groupId>com.graphql-java</groupId>
            <artifactId>graphql-spring-boot-starter</artifactId>
            <version>5.0.2</version>
        </dependency>
        <dependency>
            <groupId>com.graphql-java</groupId>
            <artifactId>graphql-java-tools</artifactId>
            <version>5.2.4</version>
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

#### 2-2. /src/main/resources/box.graphqls 

```
# GraphQL Schema 정의
type Box {
   name: String!
   color: String!
   address: String
   price: Int
}

 input BoxParam {
   name: String!
   color: String!
   address: String
   price: Int
}

type Query {
    getBox(name: String!) : Box!
    existBox(name: String!) : Boolean
    countBox: Int
}

type Mutation {
    putBox(boxParam: BoxParam) : Boolean
    delBox(name: String!) : Boolean
}
```

<br/>

#### 2-3. /src/main/java/com/example/app/model/Box.java

```java
package com.example.app.model;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Box {
    String name;
    String color;
    String address;
    int    price;
}
```

<br/>

#### 2-3. /src/main/java/com/example/app/SpringBootWebApplication.java

```java
package com.example.app;

import com.coxautodev.graphql.tools.GraphQLMutationResolver;
import com.coxautodev.graphql.tools.GraphQLQueryResolver;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.Banner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Repository;

import java.util.HashMap;
import java.util.Map;

@Slf4j
@SpringBootApplication
public class SpringBootWebApplication {
    @Getter
    @Setter
    @NoArgsConstructor
    @AllArgsConstructor
    public class Box {
        String name;
        String color;
        String address;
        int    price;
    }

    @Repository
    public class BoxRepository {
        Map<String, Box> boxMap = new HashMap<>();
        public Box getBox(String name) {
            return boxMap.getOrDefault(name, null);
        }
        public int countBox() {
            return boxMap.size();
        }
        public boolean existBox(String name) {
            return boxMap.get(name) != null;
        }
        public boolean putBox(Box box) {
            if (boxMap.get(box.getName()) != null)
                return false;

            this.boxMap.put(box.getName(), box);
            return true;
        }

        public boolean deleteBox(String name) {
            if (boxMap.get(name) == null)
                return false;

            boxMap.remove(name);
            return true;
        }
    }

    @Component
    public class BoxGraphQLQueryResolver implements GraphQLQueryResolver {
        public int countBox() {return boxRepository.countBox();}
        public boolean existBox(String name) {return boxRepository.existBox(name);}
        public Box getBox(String name) {return boxRepository.getBox(name);}

        @Autowired
        BoxRepository boxRepository;
    }

    @Component
    public class BoxGraphQLMutationResolver implements GraphQLMutationResolver {
        public boolean putBox(Box box) {return boxRepository.putBox(box);}
        public boolean delBox(String name) {return boxRepository.deleteBox(name);}

        @Autowired
        BoxRepository boxRepository;
    }


    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(SpringBootWebApplication.class);
        app.setBannerMode(Banner.Mode.OFF);
        app.run(args);
    }
}
```

<br/>

#### 2-4. 테스트



모든 GraphQL 요청은 POST 기반이어야 하고,<br/>

모든 GraphQL 요청의 URL은 http://127.0.0.1:8080/graphql 이어야 합니다. <br/>



##### putBox

```graphql
mutation {
    putBox(
        boxParam : {
            name: "gift"
            color : "green"
        }
    ) 
}

{
    "data": {
        "putBox": true
    }
}
```

<br/>

##### getBox

```graphql
query {
   getBox(name: "gift") {
       name
       color
   }
}

{
    "data": {
        "getBox": {
            "name": "gift",
            "color": "green"
        }
    }
}
```

<br/>

##### existBox

```graphql
query {
    existBox(name: "gift")
}

{
    "data": {
        "existBox": false
    }
}
```

<br/>

##### countBox

```graphql
query {
    countBox
}

{
    "data": {
        "countBox": 1
    }
}
```

<br/>

##### delBox

```graphql
mutation {
    delBox(name: "gift")
}

{
    "data": {
        "delBox": true
    }
}
```

<br/>

<br/>

## 3. 정리



GraphQL에서는 Schema 로 좀 더 명확한 통신이 가능하지만, 

반대로, 매번 통신 스펙이 변경될 때마다, Schema 파일까지 함께 변경해 주어야 하는 번거로움이 있습니다.

<br/>

그리고, 특정 필드를 필터링하여 응답하는 편리한 기능이 있는 반면에,

전체 필드를 한꺼번 조회할 수 있는 방법이 없기 때문에,

전체 필드를 조회할려면, 모든 필드를 모두 다 명세해주어야 하는 번거로움이 있습니다.

<br/>

모든 리소스의 Entry Point가 반드시 http://127.0.0.1:8080/graphql 이어야 하고,

모든 요청 메소드가 POST 로 되어야 하기 때문에, URL 만으로 어떤 행위가 이루어질지 모른다는 점은

GraphQL의 단점 이기도 합니다.

<br/>

참고로, JSON 환경에서 특정 필드만으로 응답하려면, 

Jackson 라이브러리에서 제공하는 JsonView 를 활용하면 됩니다. 



그리고, 자원별 Entry Point와 그에 맞는 적절한 METHOD 를 분리할 수 있다는 점은

오히려 JSON 기반의 통신방식이 더 좋다는 생각 또한 들었습니다.







