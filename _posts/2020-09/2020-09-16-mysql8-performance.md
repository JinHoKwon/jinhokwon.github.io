---
title: MySQL Performance Tip
comments: true
tags: [devops, docker]
categories: devops
header:
  teaser: "/assets/images/mysql/mysql_logo.png"
---
본 문서에서는 Mysql 의 성능 개선을 위한 Tip을 설명하고 있습니다. <br/>

## 1.  innodb_flush_log_at_trx_commit

innodb의 로그중 redo log(트랜잭션 로그)가 디스크에 기록되는 방법(write, flush)을 

`innodb_flush_log_at_trx_commit` 옵션을 통해서 설정할 수 있습니다.



* **`0`** : 커밋시에 log buffer까지만 반영되고 약 1초 간격으로 버퍼의 내용을 디스크에 반영하기 때문에,

  mysqld가 비정상 종료되었을경우 약 1초간의 트랜잭션정보가 유실(복구되지 않을 수)될 수 도 있습니다.

  반면 flush의 횟수를 줄일수 있어 쿼리의 성능이 향상 됩니다.

  

* **`1 (기본값)`** : 커밋시에 디스크에 flush까지 하기 때문에, 안정성은 높지만, 

  성능면에서는 가장 느리게 동작합니다.

  

* **`2`** : 의 경우는 0과 1의 중간으로 OS의 버퍼(캐쉬)까지 반영되며 그 후 OS가 1초 간격으로 flush를 담당하게 됩니다.

  이는 mysqld가 종료되어도 안전하지만 시스템의 장애가 발생할 경우 1초간의 트랜잭션 정보가 유실될 수도 있습니다. 



안정성이(1 > 2 > 0) 높을수록 성능(0 > 2 > 1)이 저하되기 때문에, 

0보다는 2가 좋으며, 반드시 안정성이 보장되어야 하는 상황이라면, 1로 설정해야 합니다.



##### 참고 

- https://systemv.tistory.com/48
- https://engineering.linecorp.com/ko/blog/line-manga-database/







<br/>

## 2. Mysql Deadlock

### 

```
결과적으로 select ... for update 의 대상 테이블을 외부키를 정의한 순서에 맞추면 deadlock이 발생하지 않는다는 것을 알 수 있다.
그리고 slock이 문제가 되는 경우가 많은데 외부키를 지정한다면 데드락 방지를 위해 insert하기전에 xlock을 걸어두는 것도 하나의 방법인것 같다.
```



https://m.blog.naver.com/PostView.nhn?blogId=parkjy76&logNo=220639066476&proxyReferer=https:%2F%2Fwww.google.com%2F





