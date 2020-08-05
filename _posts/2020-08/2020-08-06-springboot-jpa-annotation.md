---
title: SpringBoot JPA Annotation
tags: [springboot, dev]
comments: true
categories: springboot
header:
  teaser: "/assets/images/springboot/springboot_logo.png"
---

본 문서에서는 객체와 DB간의 관계를 매핑시켜주는 JPA Annotation 에 대해서 소개하고 있습니다.

## 1. JPA Annotation

#### 1-1. @Entity

엔티티임을 정의

<br/>

#### 1-2. @Table 

테이블 정의 

<br/>

#### 1-3. @Id

테이블의 기본키(PK)에 매핑

<br/>

#### 1-4. @Column 

클래스의 필드를 컬럼에 매핑

<br/>

#### 1-5. @GeneratedValue 

기본키 생성전략 설정 (기본값 : AUTO)
* GenerationType.IDENTITY : 기본 키 생성을 데이터베이스에 위임. (AUTO_INCREMENT)
* GenerationType.SEQUENCE : 데이터베이스의 Sequece Object를 사용. (AUTO_INCREMENT)

<br/>


