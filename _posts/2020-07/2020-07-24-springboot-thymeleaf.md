---
title: Spring Boot Thymeleaf
tags: [springboot, dev]
comments: true
categories: springboot
header:
  teaser: "/assets/images/springboot/thymeleaf.png"
---
## 1. Thymeleaf



Thymeleaf는 스프링 부트에서 지원하는 서버 사이드 탬플릿 엔진입니다.<br/>

HTML 태그안에 Thymeleaf 코드가 들어가는 형태라서, HTML 디자인에 영향을 주지 않기 때문에,<br/>

HTML 마크업 개발자와 공동작업을 진행할 때 서로간의 간섭없이 작업 진행이 가능합니다.<br/>

<br/>

본 문서에서는 Thymeleaf 에 대한 자세한 내용을 다루기 보다는,

Spring boot 환경에서 Thymeleaf를 연동하는 방법과, 

Thymeleaf 에서 객체와 배열 그리고 Message Source를 어떻게 처리할 수 있는지에 대해서만 설명하고 있습니다.

<br/>

## 2. Spring Boot Thymeleaf

<br/>

#### 2-1. pom.xml

`spring-boot-starter-thymeleaf`와 `spring-boot-starter-web`을 추가합니다.

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
    <artifactId>spring-boot-web-view-template-thymeleaf-message-source</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-boot-web-view-template-thymeleaf-message-source</name>
    <description>Spring Boot Web View Template Thymeleaf Message Source Example</description>

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
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
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

#### 2-2. /resources/application.yml

* prefix : 탬플릿 경로
* suffix : 탬플릿 확장자
* cache : 캐쉬 사용 유/무 (운영환경에서는 true를 권장함.)

```yaml
spring:
  thymeleaf:
    prefix: classpath:templates/
    suffix: .html
    cache: false
```

<br/>

#### 2-3. message source

Thymeleaf와 message source 를 함께 사용하기 위해서, 다음과 같이 en, ko 리소스를 정의합니다.

##### /resources/messages.properties

```properties
welcome=환영합니다. 
timestamp=시간 : {0}
```

<br/>

##### /resources/messages_en.properties

```properties
welcome=welcome
timestamp=time : {0}
```

<br/>

##### /resources/messages_ko.properties

```properties
welcome=환영합니다.
timestamp=시간 : {0}
```

<br/>

#### 2-4. Thymeleaf template 

##### /resources/templates/print.html

message source 를 출력할 때는 `#`을 사용하고, 값을 출력할 때는 `$`를 사용합니다.

```html
<html xmlns:th="http://www.thymeleaf.org">
<!DOCTYPE html>
<html>
    <head>
        <meta charset = "UTF-8" />
        <title>Spring Boot Application</title>
    </head>
    <body>
        <span th:text="${visitTime} + ' ' + #{welcome}"></span>
        <span th:text="${name}"></span>
    </body>
</html>
```

<br/>

##### /resources/templates/object.html

객체에 접근할 때는, `객체.맴버변수` 형태로 접근합니다.

```html
<html xmlns:th="http://www.thymeleaf.org">
<!DOCTYPE html>
<html>
    <head>
        <meta charset = "UTF-8" />
        <title>Spring Boot Application</title>
    </head>
    <body>
        <li th:text="${'code : ' + card.code}"></li>
        <li th:text="${'name : ' + card.name}"></li>
    </body>
</html>
```

<br/>

##### /resources/templates/array.html

배열에 접근할 때는, `th:each` 태그를 사용합니다.

```html
<html xmlns:th="http://www.thymeleaf.org">
<!DOCTYPE html>
<html>
    <head>
        <meta charset = "UTF-8" />
        <title>Spring Boot Application</title>
    </head>
    <body>
        <li th:each="name : ${names}" th:text="${name}"></li>
    </body>
</html>
```

<br/>

#### 2-5. Thymeleaf spring boot configuration

##### /app/config/WebMvcConfig.java

local change interceptor로 `lang` 파라미터를 설정하며, 기본 로케일을 KOREA로 설정합니다.

```java
package com.example.app.config;

import org.springframework.context.MessageSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.support.ReloadableResourceBundleMessageSource;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.i18n.LocaleChangeInterceptor;
import org.springframework.web.servlet.i18n.SessionLocaleResolver;

import java.util.Locale;

@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Bean
    public LocaleResolver localeResolver() {
        SessionLocaleResolver sessionLocaleResolver = new SessionLocaleResolver();
        sessionLocaleResolver.setDefaultLocale(Locale.KOREA);
        return sessionLocaleResolver;
    }

    @Bean
    public LocaleChangeInterceptor localeChangeInterceptor() {
        LocaleChangeInterceptor localeChangeInterceptor = new LocaleChangeInterceptor();
        localeChangeInterceptor.setParamName("lang");
        return localeChangeInterceptor;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(localeChangeInterceptor());
    }

    @Bean
    public MessageSource messageSource(){
        ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
        messageSource.setBasename("classpath:/messages");
        messageSource.setDefaultEncoding("UTF-8");
        messageSource.setCacheSeconds(10);
        return messageSource;
    }
}
```

<br/>

#### 2-6. Thymeleaf controller

##### /app/controller/MainController.java

```java
package com.example.app.controller;

import com.example.app.model.Card;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.MessageSource;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Arrays;
import java.util.Locale;

@Controller
@RequestMapping("/view")
public class MainController {

    @GetMapping(path = "/print")
    public ModelAndView print(Locale locale) {
        ModelAndView modelAndView = new ModelAndView();
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd hh:mm:ss");
        String params[] = new String[] {LocalDateTime.now().format(formatter)};
        
        String visitTime = this.messageSource.getMessage(
            "timestamp",
            params,
            locale
        );

        modelAndView.addObject("name", "Guest");
        modelAndView.addObject("visitTime", visitTime);
        modelAndView.setViewName("/print.html");
        return modelAndView;
    }

    @GetMapping(path = "/array")
    public String array(Model model) {
        model.addAttribute("names", Arrays.asList("jinho", "james"));
        return "/array.html";
    }

    @GetMapping(path = "/object")
    public String object(Model model) {
        model.addAttribute("card", new Card("현대카드", "001"));
        return "/object.html";
    }

    @Autowired
    MessageSource messageSource;
}
```

<br/>

##### /app/model/Card.java

```java
package com.example.app.model;

public class Card {
    public Card() {

    }

    public Card(String name, String code) {
        this.name = name;
        this.code = code;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }

    public void setCode(String code) {
        this.code = code;
    }

    public String getCode() {
        return this.code;
    }

    String name;
    String code;
}

```

<br/>

##### /app/SpringBootWebApplication.java

```java
package com.example.app;

import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.Banner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@Slf4j
@SpringBootApplication
public class SpringBootWebApplication {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(SpringBootWebApplication.class);
        app.setBannerMode(Banner.Mode.OFF);
        app.run(args);
    }
}
```

<br/>

#### 2-7. Thymeleaf test

##### message source 와 Thymeleaf  한글 테스트

```sh
$ curl http://localhost:8080/view/print?lang=ko
```

```html
<html>                                                       
<!DOCTYPE html>                                              
<html>                                                       
    <head>                                                   
        <meta charset = "UTF-8" />                           
        <title>Spring Boot Application</title>               
    </head>                                                  
    <body>                                                   
        <span>시간 : 2020-07-24 03:16:03 환영합니다.</span>         
        <span>guest</span>                                   
    </body>                                                  
</html>
```



##### message source 와 Thymeleaf  영문 테스트

```sh
$ curl http://localhost:8080/view/print?lang=en
```

```html
<html>
<!DOCTYPE html>
<html>
    <head>
        <meta charset = "UTF-8" />
        <title>Spring Boot Application</title>
    </head>
    <body>
        <span>time : 2020-07-24 03:16:24 welcome</span>
        <span>guest</span>
    </body>
</html>
```



##### Thymeleaf  오브젝트 테스트

```sh
$ curl http://localhost:8080/view/object
```

```html
<html>
<!DOCTYPE html>
<html>
   <head>
      <meta charset = "UTF-8" />
      <title>Spring Boot Application</title>
   </head>
   <body>
    <li>code : 001</li>
    <li>name : 현대카드</li>
   </body>
</html>
```



##### Thymeleaf  배열 테스트

```sh
$ curl http://localhost:8080/view/array
```

```html
<html>
<!DOCTYPE html>
<html>
   <head>
      <meta charset = "UTF-8" />
      <title>Spring Boot Application</title>
   </head>
   <body>
    <li>jinho</li>
    <li>james</li>
   </body>
</html>
```

<br/>

## 3. 정리

<br/>

과거 JSP 시절에는 서버 사이드 랜더링이 거의 대다수였지만, 

최근에는 서버에서는 데이터를 내려주고, 

클라이언트 (웹 브라우저 또는 모바일 기기등)에서 랜더링을 하는 경우가 더 많아졌고,

그 와 관련된 기술 또한 많이 보편화 되었습니다. (예 : Vue, React 등)

<br/>



그렇기 때문에, 프론트 개발자로부터 지원받을 수 있는 환경이라면,<br/>

Thymeleaf는 최소한의 데이터 조작을 위한 정보 정도만 알고 있어도 된다고 생각합니다.





