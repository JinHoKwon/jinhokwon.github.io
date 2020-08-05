---
title: MySQL 8 버전에서 변경된 AUTO_INCREMENT 초기화 방식
tags: [devops, mysql]
comments: true
categories: mysql
header:
  teaser: "/assets/images/mysql/mysql_logo.png"
---

MySQL 8 버전 이하까지는 AUTO_INCREMENT 카운터의 값을 메모리상에서만 보관하였고,<br/>

MySQL 서버가 재시작 되는 경우, `MAX(AUTO_INCREMENT)` 값을 기준으로 AUTO_INCREMENT 값을 관리하였습니다.<br/>

<br/>

때문에, AUTO_INCREMENT의 값이 유니크하게 증가하지 않고, <br/>

도중에 삭제된 AUTO_INCREMENT의 값이 재사용 되는 문제점이 있었습니다.<br/>

<br/>

이 문제를 해결하고자, MySQL 8 버전 이후부터는 <br/>

AUTO_INCREMENT 카운터의 값을 디스크에 기록 및 관리하는 방식으로 변경되었습니다.<br/>

<br/>



## 1. MySQL 5 AUTO_INCREMENT

#### 1-1. MySQL 5 버전의 도커 이미지 준비

```sh
# mkdir -p /tmp/docker/mysql5
# docker pull mysql:5.7
# docker run --name mysql5 -v /tmp/docker/mysql5:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=1111 -d mysql:5.7
# docker exec -i -t mysql5 mysql -u root -p 
```

<br/>

#### 1-2. 샘플 테이블 생성 및 데이터 입력

```sql
CREATE DATABASE test;
USE test;
CREATE TABLE box (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(30) NOT NULL);
INSERT INTO box (name) VALUES ('red');
INSERT INTO box (name) VALUES ('green');
INSERT INTO box (name) VALUES ('blue');
DELETE FROM box;
```

<br/>

#### 1-3. AUTO_INCREMENT 값 확인

```sql
> SELECT AUTO_INCREMENT FROM information_schema.tables WHERE table_name = 'box' AND table_schema = DATABASE();
+----------------+
| AUTO_INCREMENT |
+----------------+
|              4 |
+----------------+
1 row in set (0.00 sec)
```

<br/>

#### 1-4. MySQL 5 서버 재시작

```sh
# docker stop mysql5 
# docker start mysql5 
```

<br/>

#### 1-5.  AUTO_INCREMENT 값 확인

MySQL 8 버전 이하에서는 기존 ROW 들중 MAX 값을 구하여, AUTO_INCREMENT ID 값을 구하기 때문에,<br/>

모든 행이 삭제된 경우의 AUTO_INCREMENT의 값은 1이 됩니다.

```sh
# docker exec -i -t mysql5 mysql -u root -p 
```

```sql
USE test;
SELECT AUTO_INCREMENT FROM information_schema.tables WHERE table_name = 'box' AND table_schema = DATABASE();
+----------------+
| AUTO_INCREMENT |
+----------------+
|              1 |
+----------------+

1 row in set (0.00 sec)
=====
```

<br/>

<br/>

## 2. MySQL 8 AUTO_INCREMENT

본 색션에서는 앞에서 설명했던 동일한 과정을 MySQL 8 버전에서 그대로 진행하고, 차이점을 이해 합니다.

<br/>

#### 2-1. MySQL 8 버전의 도커 이미지 준비

```sh
# mkdir -p /tmp/docker/mysql8
# docker pull mysql:8
# docker run --name mysql8 -p 3306:3306 -v /tmp/docker/mysql5:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=1111 -d mysql:8
# docker exec -i -t mysql8 mysql -u root -p 
```

<br/>

#### 2-2. 샘플 테이블 생성 및 데이터 입력

```sql
CREATE DATABASE test;
USE test;
CREATE TABLE box (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(30) NOT NULL);
INSERT INTO box (name) VALUES ('red');
INSERT INTO box (name) VALUES ('green');
INSERT INTO box (name) VALUES ('blue');
DELETE FROM box;
```

<br/>

#### 2-3. AUTO_INCREMENT 값 확인

```sql
SELECT AUTO_INCREMENT FROM information_schema.tables WHERE table_name = 'box' AND table_schema = DATABASE();
+----------------+
| AUTO_INCREMENT |
+----------------+
|              4 |
+----------------+
1 row in set (0.01 sec)
```

<br/>

#### 2-4. MySQL 8 서버 재시작

```sh
# docker stop mysql8
# docker start mysql8 
```

<br/>

#### 2-5. AUTO_INCREMENT 값 확인

MySQL 8 버전 부터는 AUTO_INCREMENT의 값을 디스크에 기록하기 때문에,<br/>

모든 행을 삭제하고, MySQL 서버를 재시작 하더라도, 정상적인 AUTO_INCREMENT의 값이 설정됨을 확인할 수 있었습니다.

```sh
# docker exec -i -t mysql8 mysql -u root -p 
```

```sql
> USE test;
> SELECT AUTO_INCREMENT FROM information_schema.tables WHERE table_name = 'box' AND table_schema = DATABASE();
+----------------+
| AUTO_INCREMENT |
+----------------+
|              4 |
+----------------+
1 row in set (0.00 sec)
```



