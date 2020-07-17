---
title: Aspect-Oriented Programming
tags: [springboot, dev]
comments: true
categories: springboot
header:
  teaser: "/assets/images/springboot_logo.png"
---
## 1. OOP와 AOP의 차이점

#### 1-1. OOP (Object Oriented Programming)

모든 데이터를 객체로 취급하여 프로그래밍하는 방법.
1. 비지니스 로직의 모듈화
2. 캡슐화(encapsulation ), 다형성(polymorphism ), 상속성(Inheritance)

<br/>

#### 1-2. AOP (Aspect-Oriented Programming)

인프라 혹은 부가 기능의 모듈화 (예 : 로깅, 트랜잭션, 보안 등)<br/>
그리고, 해당 기능을 구현하기 위해서 프록시와 위빙을 사용합니다.
* 프록시 (Proxy) : 타겟을 감싸서 타겟의 요청을 대신 받아주는 랩핑(Wrapping) 오브젝트
* 위빙 (Weaving) : 지정된 객체에 애스팩트를 적용해서 새로운 프록시 객체를 생성하는 과정

<br/>
<br/>

## 2. AOP 정리


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
#### 2-3.  Pointcut 표기예제

* JointPoint : Advice의 적용될 위치

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


