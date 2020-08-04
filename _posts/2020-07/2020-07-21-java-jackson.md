---
title: Jackson
tags: [java, dev]
comments: true
categories: java
header:
  teaser: "/assets/images/java/java_logo.png"
---
## 1. Jackson

Java 기반의 대표적인 JSON 라이브러리이며,

REST API 가 대세인 요즘 Jackson 은 Log4j2 라이브러리 만큼이나 

기본적으로 포함이 되는 필수 라이브러리 이기도 합니다.

<br/>

#### 1-1. pom.xml

jackson 을 사용하기 위해서 필요한 최소 라이브러리는 다음과 같습니다.
* jackson 라이브러리 추가
* jackson-core : 코어 라이브러리
* jackson-databind : 데이터 바인딩에 사용
* jackson-dataformat-xml : XML 지원 (선택)
* jackson-datatype-jsr310 : Java 8 Date & Time API 지원 
* jackson-annotations : 데이터 바인딩에 필요한 어노테이션 지원 
* lombok : 어노테이션 기반의 Getter / Setter 를 자동으로 만들어 주는 라이브러리 (선택)

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

이해를 돕기위한 `Box.java` VO 객체를 다음과 같이 생성합니다.

* @JsonInclude : 포함시킬 필드
* @JsonIgnoreProperties : 제외시킬 필드
* @JsonFormat : 날짜, 시간을 포멧에 맞게 처리 하기 위해서 사용
* @JsonProperty : 필드명을 명시적으로 boxName으로 설정
* @JsonIgnore : Json 제외처리
* @Getter, @Setter, @ToString, @NoArgsConstructor, @AllArgsConstructor 는 lombox 에서 사용함.

만약에 lombok 라이브러리를 사용하지 않았더라면, 각각의 맴버변수마다 Getter와 Setter 메소드를 설정해 주어야 합니다.

```java
import com.fasterxml.jackson.annotation.*;
import lombok.*;
import java.time.LocalDateTime;

@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonIgnoreProperties(ignoreUnknown = true)
@Getter
@Setter
@ToString
@NoArgsConstructor(access = AccessLevel.PUBLIC)
@AllArgsConstructor
public class Box {

    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'")
    LocalDateTime createDatetime;

    @JsonProperty("boxName")
    String name;

    @JsonIgnore
    String unlockPassword;
}
```

<br/>

#### 1-3. 시간 

시간을 처리하기 위해서는, JavaTimeModule를 모듈로 등록한 이후에, 

@JsonFormat으로 pattern을 설정해야 올바르게 처리됩니다.

<br/>

##### LocalDateTime의 deserialize 예제

```java
ObjectMapper mapper = new ObjectMapper().registerModules(
    new JavaTimeModule()
);

String json = "{" +
    "\"createDatetime\" : \"2020-02-20T11:22:33.007Z\", " +
    "\"boxName\" : \"giftBox\"" +
    "}";
Box deserializeBox = mapper.readValue(json, Box.class);
log.info("{}", deserializeBox);
```

<br/>

##### LocalDateTime의 밀리초 이하를 truncate하고, serialize ~ deserialize한 결과값이 같은지 비교하는 예제

```java
LocalDateTime now = LocalDateTime.now().truncatedTo(ChronoUnit.MILLIS);
Box box = new Box(now);
String json = mapper.writeValueAsString(box);
Box deserializeBox = mapper.readValue(json, Box.class);
Assertions.assertTrue(deserializeBox.getCreateDatetime().isEqual(now));
```



<br/>

#### 1-4. 배열 

배열을 처리하기 위해서는 TypeReference 클래스를 활용하여 deserialize 클래스의 정보를 설정해 주어야 합니다.

<br/>

##### Jackson으로 List를 처리하는 예제

```java
ObjectMapper mapper = new ObjectMapper().registerModules(
    new JavaTimeModule()    // LocalDateTime 을 파싱하기 위해서 사용
);

LocalDateTime now = LocalDateTime.now().truncatedTo(ChronoUnit.MILLIS);
List<Box> boxList = new ArrayList<>(Arrays.asList(new Box(now)));
String json = mapper.writeValueAsString(boxList);
List<Box> deserializeBoxList = mapper.readValue(json, new TypeReference<List<Box>>() {});
Assertions.assertTrue(deserializeBoxList.size() == 1);
Assertions.assertTrue(deserializeBoxList.get(0).getCreateDatetime().isEqual(now));
```







