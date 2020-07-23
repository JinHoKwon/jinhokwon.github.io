---
title: MySQL 격리 수준(Isolation level)의 이해
tags: [devops, mysql]
comments: true
categories: mysql
header:
  teaser: "/assets/images/mysql/mysql_logo.png"
---

## 1. Isolation level

Isolation level은 동시에 여러 트랜잭션이 실행 되는 환경에서, <br/>

데이터의 접근 격리 수준 및 동시성을 제한하고자 할 때 사용이 됩니다.

<br/>

MySQL 의 Isolation level의 기본값은 `Repeatable read` 이며, 

MySQL에서 제공되는 격리 수준은 다음과 같습니다.

<br/>

| Isolation level  | Dirty read | Non-repeatable read | Phantom read |
| :--------------- | :--------: | :-----------------: | :----------: |
| Read uncommitted |     O      |          O          |      O       |
| Read committed   |     X      |          O          |      O       |
| Repeatable read  |     X      |          X          |      O       |
| Serializable     |     X      |          X          |      X       |

<br/>
그리고, 격리수준별 특징에 대한 설명은 다음과 같습니다.
<br/>

* Dirty read : 커밋되지 않은 데이터를 조회할 수 있음.
* Non-repeatable read : 동일한 트랜잭션에서 동일한 쿼리가 2번 실행되었을 때, 조회 결과가 각각 다를 수 있음.
* Phantom read : 동일한 트랜잭션 동일한 쿼리가 여러번 실행되었을 때, 이전에 없었던 결과가 추가 될 수 있음.

<br/>
<br/>

## 2. Isolation level 주의사항

모든 상황을 만족하는 Isolation level 없으니,

상황별 적절한 Isolation level을 고려하는것이 중요하며,

본 섹션에서는 보편적으로 자주 부딪히게 되는 데이터의 조회 부터 데이터 수정까지

권장되는 Isolation level을 설명하고 있습니다.

<br/>



#### 2-1. 단순히 데이터를 읽기만 하는 경우

단순히 데이터를 읽기만 하는 경우라면,

Isolation level을 **Repeatable read**로 설정하고,

대량의 데이터를 복제 또는 추가해야 하는 경우라면, **Read committed**로 설정하는것이 좋습니다. 

<br/>

#### 2-2. 대량의 데이터를 복제 또는 추가하는 경우 

Repeatable read 은 동일한 트랜잭션 내에서 동일한 쿼리의 조회 결과를 보장하기 위해서 SNAPSHOP을 사용하는데, <br/>

이 때, LOCK 이 걸리게 됩니다. <br/>

<br/>

그래서,  Isolation level이 Repeatable read 인 환경에서 다음과 같은 쿼리들이 실행 될 때,<br/>

다른 SESSION의 데이터의 수정작업은 대기 상태에 빠지게 됩니다.<br/>

<br/>

##### 즉, SESSION #1에서 테이블 복제(또는 대량의 데이터를 추가하는 등의 작업)가 이루어지는 동안

```sql
CREATE TABLE copy_table SELECT * FROM source_table
INSERT INTO copy_table(field1, field2, ...) SELECT * FROM source_table
```

<br/>

##### SESSION #2에서 UPDATE 또는 DELETE 쿼리를 수행하게 되면, 대기 또는 데드락 현상이 발생하게 됩니다.

```sql
DELETE FROM source_table WHERE ...
ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction
```

<br/>

그리고, 이러한 현상을 방지하기 위해서는 

대량의 데이터를 복제 또는 이동하는 경우, 

해당 SESSION의 Isolation Level을 **Read committed** 로 설정해야 합니다.

<br/>

#### 2-3. INSERT, UPDATE, DELETE 

Isolation level을 **Serializable**로 설정하는것이 좋습니다.

