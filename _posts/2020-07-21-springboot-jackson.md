---
title: Spring Boot Jackson
tags: [springboot, dev]
comments: true
categories: springboot
header:
  teaser: "/assets/images/springboot/springboot_logo.png"
---
## 1. Jackson

Java 기반의 대표적인 JSON 처리 라이브러리 

<br/>

#### 1-1. pom.xml

jackson 라이브러리 추가

```xml
<properties>
    <java.version>11</java.version>
    <jackson.version>2.11.0</jackson.version>
    <lombok.version>1.18.12</lombok.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>

<dependencies>
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

    <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.dataformat/jackson-dataformat-xml -->
    <dependency>
        <groupId>com.fasterxml.jackson.dataformat</groupId>
        <artifactId>jackson-dataformat-xml</artifactId>
        <version>${jackson.version}</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.datatype/jackson-datatype-jsr310 -->
    <dependency>
        <groupId>com.fasterxml.jackson.datatype</groupId>
        <artifactId>jackson-datatype-jsr310</artifactId>
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
```



<br/>

#### 1-2. 테스트 모델

Box.java

이 때, `LocalDateTime`과 같은 객체는 @JsonFormat으로 pattern을 반드시 지정해야 합니다.

```java
package com.example.app.model;

import com.fasterxml.jackson.annotation.JsonFormat;
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
import com.fasterxml.jackson.annotation.JsonInclude;
import lombok.*;
import java.time.LocalDateTime;

@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonIgnoreProperties(ignoreUnknown = true)
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PUBLIC)
@AllArgsConstructor
public class Box {
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss.SSS")
    LocalDateTime createDatetime;
}
```



#### 1-3. 시간, 배열 사용 예제

시간을 처리하기 위해서는, JavaTimeModule를 모듈로 등록한 이후에, @JsonFormat으로 pattern을 설정해야 하고,

배열을 처리하기 위해서는 TypeReference 클래스를 활용하여 Deserialize 클래스의 정보를 설정해 주어야 합니다.

```java
package com.example.app;

import com.example.app.model.Box;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit.jupiter.SpringExtension;

import java.time.LocalDateTime;
import java.time.temporal.ChronoUnit;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

@Slf4j
@ExtendWith(SpringExtension.class)
@SpringBootTest
@AutoConfigureMockMvc
public class SpringBootConsoleApplicationTests {

    @Test
    public void localDateTimeTest() throws Exception {
        LocalDateTime now = LocalDateTime.now().truncatedTo(ChronoUnit.MILLIS);
        Box box = new Box(now);
        String json = mapper.writeValueAsString(box);
        Box deserializeBox = mapper.readValue(json, Box.class);
        Assertions.assertTrue(deserializeBox.getCreateDatetime().isEqual(now));
    }

    @Test
    public void jsonArrayTest() throws Exception {
        LocalDateTime now = LocalDateTime.now().truncatedTo(ChronoUnit.MILLIS);
        List<Box> boxList = new ArrayList<>(Arrays.asList(new Box(now)));
        String json = mapper.writeValueAsString(boxList);
        List<Box> deserializeBoxList = mapper.readValue(json, new TypeReference<List<Box>>() {});
        Assertions.assertTrue(deserializeBoxList.size() == 1);
        Assertions.assertTrue(deserializeBoxList.get(0).getCreateDatetime().isEqual(now));
    }

    ObjectMapper mapper = new ObjectMapper().registerModules(
        new JavaTimeModule()    // LocalDateTime 을 파싱하기 위해서 사용
    );
}
```





