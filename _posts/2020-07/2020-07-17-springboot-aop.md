---
title: Aspect-Oriented Programming
tags: [springboot, dev]
comments: true
categories: springboot
header:
  teaser: "/assets/images/springboot/springboot_logo.png"
---
## 1. OOP와 AOP의 차이점

<br/>

#### 1-1. OOP (Object Oriented Programming)

모든 데이터를 객체로 취급하여 프로그래밍하는 방법.
1. 비지니스 로직의 모듈화
2. 캡슐화(encapsulation ), 다형성(polymorphism ), 상속성(Inheritance)

<br/>

#### 1-2. AOP (Aspect-Oriented Programming)

특정한 객체를 언제(Advice), 어디서(Pointcut), 어떻게 제어(상태 조회 및 변경)할지를 AOP를 통해서 구현할 수 있습니다.<br/>

<br/>

AOP 를 사용하였을 경우에 대표적인 장점은

* 객체간의 관계 (또는 의존성) 제어를 개발자가 깊이 관여하지 않아도 됨.
* 필요한 객체만 생성해주면, AOP 내부적으로 복잡한 의존성 설정을 알아서 함.
* 공통된 기능을 효율적으로 재사용 할 수 있음. 
* 중복된 코드가 제거됨으로서 생산성이 높아짐.



<br/>

그리고 이러한 AOP는 인프라 혹은 공통적인 부가 기능을 모듈화 (예 : 로깅, 트랜잭션, 보안 등) 할 때 많이 사용되며,<br/>AOP 내부적으로는 해당 기능을 구현하기 위해서 프록시와 위빙을 사용합니다.

* 프록시 (Proxy) : 타겟을 감싸서 타겟의 요청을 대신 받아주는 랩핑(Wrapping) 오브젝트
* 위빙 (Weaving) : 지정된 객체에 애스팩트를 적용해서 새로운 프록시 객체를 생성하는 과정

<br/>

## 2. AOP 정리

<br/>

#### 2-1. AOP의 주요 구성요소

AOP를 적용하기 위해서는 대상(Target)이 필요하며,<br/>
대상에 적용할 시점(Advice)과 적용될 위치(Pointcut)가 필요합니다.
* Target : 대상
* Aspect : Advice (시점) + Pointcut (위치)

<br/>

#### 2-2. Advice 
  * @Before : 대상 호출전
  * @After : 대상 호출후
  * @AfterReturning : 리턴값 참조는 가능하지만, 변경은 불가능함.
  * @AfterThrowing : 처리하는 과정에서 예외가 발생한 경우
  * @Around : 반드시 proceed() 메소드가 호출되어야 하며, 리턴값의 변경이 가능함.

<br/>
#### 2-3.  Pointcut 표기 예제

Pointcut 표기법을 사용하여,  Advice가 적용될 위치인 JointPoint를 설정할 수 있습니다.

```
@Around("execution(* com.example.app.getFoo(..))")
    |       |      |            |            |
    1       2      3            4            5

1 : Advice 지정자
2 : Pointcut 지정자
3 : 리턴 타입 (* : 모든 리턴 타입 가능)
4 : 타겟 메소드
5 : 인자 타입 (* : 모든 타입 인자 가능)
```

<br/>

public 메소드 실행

```
execution(public * *(..))
```

<br/>

이름이 set으로 시작하는 모든 메소드명 실행

```
execution(* set*(..)) 
```

<br/>

AccountService 인터페이스의 모든 메소드 실행

```
execution(* com.xyz.service.AccountService.*(..))
```

<br/>

service 패키지의 모든 메소드 실행

```
execution(* com.xyz.service.*.*(..)) 
```

<br/>

service 패키지와 하위 패키지의 모든 메소드 실행

```
execution(* com.xyz.service..*.*(..))
```

<br/>

service 패키지 내의 모든 결합점

```
within(com.xyz.service.*) 
```

<br/>

service 패키지 및 하위 패키지의 모든 결합점

```
within(com.xyz.service..*) 
```

<br/>

AccountService 인터페이스를 구현하는 프록시 개체의 모든 결합점

```
this(com.xyz.service.AccountService)
```

<br/>

AccountService 인터페이스를 구현하는 대상 객체의 모든 결합점

```
target(com.xyz.service.AccountService) 
```

<br/>

하나의 파라미터를 갖고 전달된 인자가 Serializable인 모든 결합점

```
args(java.io.Serializable)
```

<br/>

대상 객체가 @Transactional 어노테이션을 갖는 모든 결합점

```
@target(org.springframework.transaction.annotation.Transactional) 
```

<br/>

대상 객체의 선언 타입이 @Transactional 어노테이션을 갖는 모든 결합점

```
@within(org.springframework.transaction.annotation.Transactional)
```

<br/>

실행 메소드가 @Transactional 어노테이션을 갖는 모든 결합점

```
@annotation(org.springframework.transaction.annotation.Transactional)
```

<br/>

단일 파라미터를 받고, 전달된 인자 타입이 @Classified 어노테이션을 갖는 모든 결합점

```
@args(com.xyz.security.Classified)
```

<br/>

"accountRepository" 빈

```
bean(accountRepository) 
```

<br/>

"accountRepository" 빈을 제외한 모든 빈

```
!bean(accountRepository) 
```

<br/>

모든 빈

```
bean(*)
```

<br/>

이름이 'account'로 시작되는 모든 빈

```
bean(account*) 
```

<br/>

이름이 “Repository”로 끝나는 모든 빈

```
bean(*Repository)
```

<br/>

이름이 “accounting/“로 시작하는 모든 빈

```
bean(accounting/*) 
```

<br/>

이름이 "dataSource"나 "DataSource"으로 끝나는 모든 빈

```
bean(*dataSource) || bean(*DataSource) 
```

<br/>

 

## 3. AOP 예제

/monitor/ServiceMonitor.java

```java
package com.example.app.monitor;

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;
import org.springframework.util.StopWatch;

@Slf4j
@Aspect
@Component
public class ServiceMonitor {
    @Before("execution(* com.example..*Service.*(..))")
    public void onBefore(JoinPoint joinPoint) {
        log.info("onBefore : " + joinPoint);
    }

    @After(
        "execution(* com.example.app.service..hello(..)) ||" +
        "execution(* com.example.app.service..world(..)) "
    )
    public void onAfter(JoinPoint joinPoint) {
        log.info("onAfter : " + joinPoint);
    }

    @AfterReturning("execution(* com.example..*Service.*(..))")
    public void onAfterReturning(JoinPoint joinPoint) {
        log.info("onAfterReturning: " + joinPoint);
    }

    @Around("execution(* com.example..*Service.*(..))")
    public Object onAround(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("onAround " + joinPoint);
        Object obj = joinPoint.proceed();
        log.info("onAround After" + joinPoint);
        return obj;
    }

    @Around("@annotation(com.example.app.annotation.ProfileTime)")
    public Object onProfileTime(ProceedingJoinPoint joinPoint) throws Throwable {
        StopWatch watch = new StopWatch();
        watch.start();
        final Object proceed = joinPoint.proceed();
        watch.stop();
        log.info("STAT {} 실행 후, 경과시간 {}", joinPoint.getSignature(), watch.getTotalTimeSeconds());
        return proceed;
    }
}
```

<br/>

/annotation/ProfileTime.java

```java
package com.example.app.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ProfileTime {
}
```

<br/>



/service/HelloService.java

```java
package com.example.app.service;

import com.example.app.annotation.ProfileTime;
import org.springframework.stereotype.Service;

@Service
public class HelloService {

    public String hello() {
        return "Hello";
    }

    public String world() {
        return "World";
    }

    @ProfileTime
    public String longTask() {
        try {
            Thread.sleep(5_000);
        } catch (Exception e) {}

        return  "Long~~ Task~~";
    }
}
```

<br/>

/service/WorldService.java

```java
package com.example.app.service;

import com.example.app.annotation.ProfileTime;
import org.springframework.stereotype.Service;

@Service
public class WorldService {

    public String hello() {
        return "Hello";
    }

    public String world() {
        return "World";
    }

    @ProfileTime
    public String longTask() {
        try {
            Thread.sleep(5_000);
        } catch (Exception e) {}

        return  "Long~~ Task~~";
    }
}
```

<br/>

/SpringBootConsoleApplication.java

```java
package com.example.app;

import com.example.app.service.HelloService;
import com.example.app.service.WorldService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.Banner;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@Slf4j
@SpringBootApplication
public class SpringBootConsoleApplication implements CommandLineRunner {

    @Autowired
    HelloService helloService;

    @Autowired
    WorldService worldService;

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(SpringBootConsoleApplication.class);
        app.setBannerMode(Banner.Mode.OFF);
        app.run(args);
    }

    @Override
    public void run(String... args) throws Exception {
        log.info("{}", helloService.hello());
        log.info("{}", worldService.hello());
        log.info("{}", helloService.world());
        log.info("{}", worldService.world());
        log.info("{}", helloService.longTask());
        log.info("{}", worldService.longTask());
    }
}
```

<br/>

실행결과

```
2020-07-17T11:11:58,673 INFO  [main] o.s.b.StartupInfoLogger: Starting SpringBootConsoleApplication on N19074_W1 with PID 14604 (L:\tutorials\springboot\spring-boot-aop\target\classes started by 권진호 in L:\tutorials\springboot\spring-boot-aop)
2020-07-17T11:11:58,678 DEBUG [main] o.s.b.StartupInfoLogger: Running with Spring Boot v2.3.0.RELEASE, Spring v5.2.6.RELEASE
2020-07-17T11:11:58,679 INFO  [main] o.s.b.SpringApplication: No active profile set, falling back to default profiles: default
2020-07-17T11:12:00,091 INFO  [main] o.s.b.StartupInfoLogger: Started SpringBootConsoleApplication in 2.268 seconds (JVM running for 3.682)
2020-07-17T11:12:00,108 INFO  [main] c.e.a.m.ServiceMonitor: onAround execution(String com.example.app.service.HelloService.hello())
2020-07-17T11:12:00,110 INFO  [main] c.e.a.m.ServiceMonitor: onBefore : execution(String com.example.app.service.HelloService.hello())
2020-07-17T11:12:00,159 INFO  [main] c.e.a.m.ServiceMonitor: onAround Afterexecution(String com.example.app.service.HelloService.hello())
2020-07-17T11:12:00,160 INFO  [main] c.e.a.m.ServiceMonitor: onAfter : execution(String com.example.app.service.HelloService.hello())
2020-07-17T11:12:00,160 INFO  [main] c.e.a.m.ServiceMonitor: onAfterReturning: execution(String com.example.app.service.HelloService.hello())
2020-07-17T11:12:00,163 INFO  [main] c.e.a.SpringBootConsoleApplication: Hello
2020-07-17T11:12:00,163 INFO  [main] c.e.a.m.ServiceMonitor: onAround execution(String com.example.app.service.WorldService.hello())
2020-07-17T11:12:00,164 INFO  [main] c.e.a.m.ServiceMonitor: onBefore : execution(String com.example.app.service.WorldService.hello())
2020-07-17T11:12:00,169 INFO  [main] c.e.a.m.ServiceMonitor: onAround Afterexecution(String com.example.app.service.WorldService.hello())
2020-07-17T11:12:00,169 INFO  [main] c.e.a.m.ServiceMonitor: onAfter : execution(String com.example.app.service.WorldService.hello())
2020-07-17T11:12:00,170 INFO  [main] c.e.a.m.ServiceMonitor: onAfterReturning: execution(String com.example.app.service.WorldService.hello())
2020-07-17T11:12:00,170 INFO  [main] c.e.a.SpringBootConsoleApplication: Hello
2020-07-17T11:12:00,171 INFO  [main] c.e.a.m.ServiceMonitor: onAround execution(String com.example.app.service.HelloService.world())
2020-07-17T11:12:00,171 INFO  [main] c.e.a.m.ServiceMonitor: onBefore : execution(String com.example.app.service.HelloService.world())
2020-07-17T11:12:00,176 INFO  [main] c.e.a.m.ServiceMonitor: onAround Afterexecution(String com.example.app.service.HelloService.world())
2020-07-17T11:12:00,177 INFO  [main] c.e.a.m.ServiceMonitor: onAfter : execution(String com.example.app.service.HelloService.world())
2020-07-17T11:12:00,177 INFO  [main] c.e.a.m.ServiceMonitor: onAfterReturning: execution(String com.example.app.service.HelloService.world())
2020-07-17T11:12:00,177 INFO  [main] c.e.a.SpringBootConsoleApplication: World
2020-07-17T11:12:00,179 INFO  [main] c.e.a.m.ServiceMonitor: onAround execution(String com.example.app.service.WorldService.world())
2020-07-17T11:12:00,179 INFO  [main] c.e.a.m.ServiceMonitor: onBefore : execution(String com.example.app.service.WorldService.world())
2020-07-17T11:12:00,180 INFO  [main] c.e.a.m.ServiceMonitor: onAround Afterexecution(String com.example.app.service.WorldService.world())
2020-07-17T11:12:00,180 INFO  [main] c.e.a.m.ServiceMonitor: onAfter : execution(String com.example.app.service.WorldService.world())
2020-07-17T11:12:00,180 INFO  [main] c.e.a.m.ServiceMonitor: onAfterReturning: execution(String com.example.app.service.WorldService.world())
2020-07-17T11:12:00,181 INFO  [main] c.e.a.SpringBootConsoleApplication: World
2020-07-17T11:12:00,182 INFO  [main] c.e.a.m.ServiceMonitor: onAround execution(String com.example.app.service.HelloService.longTask())
2020-07-17T11:12:00,188 INFO  [main] c.e.a.m.ServiceMonitor: onBefore : execution(String com.example.app.service.HelloService.longTask())
2020-07-17T11:12:05,188 INFO  [main] c.e.a.m.ServiceMonitor: STAT String com.example.app.service.HelloService.longTask() 실행 후, 경과시간 5.0004542
2020-07-17T11:12:05,188 INFO  [main] c.e.a.m.ServiceMonitor: onAround Afterexecution(String com.example.app.service.HelloService.longTask())
2020-07-17T11:12:05,189 INFO  [main] c.e.a.m.ServiceMonitor: onAfterReturning: execution(String com.example.app.service.HelloService.longTask())
2020-07-17T11:12:05,189 INFO  [main] c.e.a.SpringBootConsoleApplication: Long~~ Task~~
2020-07-17T11:12:05,190 INFO  [main] c.e.a.m.ServiceMonitor: onAround execution(String com.example.app.service.WorldService.longTask())
2020-07-17T11:12:05,190 INFO  [main] c.e.a.m.ServiceMonitor: onBefore : execution(String com.example.app.service.WorldService.longTask())
2020-07-17T11:12:10,192 INFO  [main] c.e.a.m.ServiceMonitor: STAT String com.example.app.service.WorldService.longTask() 실행 후, 경과시간 5.0013335
2020-07-17T11:12:10,192 INFO  [main] c.e.a.m.ServiceMonitor: onAround Afterexecution(String com.example.app.service.WorldService.longTask())
2020-07-17T11:12:10,192 INFO  [main] c.e.a.m.ServiceMonitor: onAfterReturning: execution(String com.example.app.service.WorldService.longTask())
2020-07-17T11:12:10,193 INFO  [main] c.e.a.SpringBootConsoleApplication: Long~~ Task~~
2020-07-17T11:12:10,194 INFO  [main] c.e.a.SpringBootConsoleApplication: End
Disconnected from the target VM, address: '127.0.0.1:58224', transport: 'socket'

Process finished with exit code 0
```

<br/>

## 4. DI

DI는 의존성 주입(Dependency Injection)의 줄임말이며,

이러한 의존성 주입방법에는 Field Injection, Setter Injection, Constructor Inject 이 있지만,

이 중에서, Construct Injection을 권장하고 있으며,

IntelliJ 에서도 constructor based dependency injection을 권장하고 있습니다.

```
Field injection is not recommended … Always use constructor based dependency injection in your beans
```

<br/>



### 4-1. 생성자 주입의 장점

의존관계가 생성자에 정의되어 있기 때문에, 컴파일 타임에 오류를 발견할 수 있습니다.

생성자로 부터 주입받기 때문에, 주입 받는 변수를 final로 선언할 수 있습니다. (즉, 주입된 이후, 해당 주입된 객체는 변경할 수 없습니다.)

생성자 주입을 하게 될 경우, 실행시점에 순환 참조 이슈를 조기에 발견할 수 있습니다.

```
***************************
APPLICATION FAILED TO START
***************************

Description:

The dependencies of some of the beans in the application context form a cycle:
```

 



<br/>

## 5. IoC

생성된 Bean이 필요한 Container에 Spring Framework가 알아서 주입해 주는 과정으로,

보통 전통적인 개발과정에서는 의존관계의 설정을 개발자가 직접해주었다면,

Spring Framework 환경에서는 특정 기능을 사용하기 위해서 필요한 Bean을 생성해 놓으면,

Spring Framework가 알아서 의존관계를 설정해 줍니다. 그리고 이러한 과정을 요약하여 `제어의 역전`이라고

말합니다.



<br/>

