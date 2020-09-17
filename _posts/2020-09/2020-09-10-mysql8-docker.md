---
title: Docker 기반의 Mysql8 설치 및 실행
comments: true
tags: [devops, docker]
categories: devops
header:
  teaser: "/assets/images/mysql/mysql_logo.png"
---
본 문서에서는 Docker 환경에서 Mysql 8 버전을 설치하고 실행하는 방법에 대해서 설명하고 있습니다. <br/>

## 1. Docker 설치

[여기](/devops/devops-docker-install/)를 참고하여 Docker를 설치합니다.

<br/>

## 2. Mysql 8

### 2-1. mysql image pull

```sh
# docker pull mysql:8
```

<br/>

### 2-2. run mysql image

```sh
# docker run --name mysql8 -e MYSQL_ROOT_PASSWORD=1111 -p 3306:3306 -d mysql:8
665afd49d1d958bac99b2f414c78e62cd81ef26253f29c862bac2415360fa0fd
```

<br/>

### 2-3. mysql container status

```sh
# docker ps
CONTAINER ID        IMAGE                                   COMMAND                  CREATED             STATUS              PORTS                                                NAMES
665afd49d1d9        mysql:8                                 "docker-entrypoint.s…"   2 minutes ago       Up 2 minutes        0.0.0.0:3306->3306/tcp, 33060/tcp                    mysql8
```

<br/>

### 2-4. mysql query log 

##### mysql query 로그 활성화 및 query 로그 경로 확인

```sh
# docker exec -i -t mysql8 mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.21 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> set global general_log=1;
mysql> set global innodb_print_all_deadlocks=1;
mysql> show variables like 'general_log_file';
+------------------+---------------------------------+
| Variable_name    | Value                           |
+------------------+---------------------------------+
| general_log_file | /var/lib/mysql/665afd49d1d9.log |
+------------------+---------------------------------+
1 row in set (0.01 sec)

mysql> ctrl + p + q esacpe docker
```

##### mysql query 로그 확인

```sh
# docker exec -i -t mysql8 tail -f /var/lib/mysql/665afd49d1d9.log
```



<br/>

### 2-5. mysql deadlock log

##### mysql container의 /etc/mysql/my.cnf 파일을 호스트에 복사

```sh
# docker cp mysql8:/etc/mysql/my.cnf .
```

<br/>

##### 복사된 my.cnf 파일 편집

```sh
# vi my.cnf
```

```ini
#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL
log_error       = /var/lib/mysql/error.log

# Custom config should go here
!includedir /etc/mysql/conf.d/
```

<br/>

##### 편집된 my.cnf 파일을 mysql container로 복사

```sh
# docker cp my.cnf mysql8:/etc/mysql/my.cnf
```

<br/>

##### mysql error log 활성화 및 error 로그 경로확인

```sh
# docker exec -i -t mysql8 mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.21 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> set global innodb_print_all_deadlocks=1;
mysql> show variables like 'log_error';
+---------------+--------------------------+
| Variable_name | Value                    |
+---------------+--------------------------+
| log_error     | /var/lib/mysql/error.log |
+---------------+--------------------------+
1 row in set (0.01 sec)

mysql> ctrl + p + q esacpe docker

# docker exec -i -t mysql8 tail -f /var/lib/mysql/error.log
```

<br/>

## 3. Mysql deadlock

### 3-1. 샘플 데이터베이스 생성

```mysql
DROP IF EXISTS DATABASE testDB;
CREATE DATABASE testDB DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
USE testDB;
```

<br/>

### 3-2. 샘플 테이블 생성

```mysql
CREATE TABLE data (id INT PRIMARY KEY) ENGINE = INNODB;
```

<br/>

### 3-3. 샘플 데이터 입력

```sql
INSERT INTO data VALUES (1);
INSERT INTO data VALUES (2);
INSERT INTO data VALUES (3);
```

<br/>

### 3-3. Deadlock 재현

| No   | Session 1                                        | Session 2                                        |
| ---- | ------------------------------------------------ | ------------------------------------------------ |
|      | USE testDB                                       |                                                  |
|      | BEGIN                                            |                                                  |
|      | SELECT * FROM data WHERE ID=1 LOCK IN SHARE MODE |                                                  |
|      |                                                  | USE testDB                                       |
|      |                                                  | BEGIN                                            |
|      |                                                  | SELECT * FROM data WHERE ID=2 LOCK IN SHARE MODE |
|      | DELETE FROM data WHERE id=2                      |                                                  |
|      | WAITING ...                                      | DELETE FROM data WHERE id=1                      |
|      |                                                  | WAITING...                                       |
|      | Query OK, 1 row affected                         |                                                  |
|      |                                                  | ERROR 1213 (40001): Deadlock                     |
|      | COMMIT                                           |                                                  |
|      |                                                  | COMMIT                                           |
|      |                                                  |                                                  |

<br/>

##### 실행결과

```sql
mysql> SELECT * FROM data;
+----+
| id |
+----+
|  1 |
|  3 |
+----+
2 rows in set (0.00 sec)
```

<br/>

##### mysql error 로그 확인

```sh
# docker exec -i -t mysql8 tail -n 100 -f /var/lib/mysql/error.log

TRANSACTION 4129, ACTIVE 24 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1136, 2 row lock(s)
MySQL thread id 9, OS thread handle 140266854479616, query id 47 localhost root updating
DELETE FROM data WHERE id=2
RECORD LOCKS space id 26 page no 4 n bits 72 index PRIMARY of table `testDB`.`data` trx id 4129 lock mode S locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 80000001; asc     ;;
 1: len 6; hex 000000001012; asc       ;;
 2: len 7; hex 81000001050110; asc        ;;

RECORD LOCKS space id 26 page no 4 n bits 72 index PRIMARY of table `testDB`.`data` trx id 4129 lock_mode X locks rec but not gap waiting
Record lock, heap no 3 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 80000002; asc     ;;
 1: len 6; hex 000000001020; asc       ;;
 2: len 7; hex 820000010c0110; asc        ;;

TRANSACTION 4130, ACTIVE 11 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1136, 2 row lock(s)
MySQL thread id 11, OS thread handle 140266854184704, query id 48 localhost root updating
DELETE FROM data WHERE id=1
RECORD LOCKS space id 26 page no 4 n bits 72 index PRIMARY of table `testDB`.`data` trx id 4130 lock mode S locks rec but not gap
Record lock, heap no 3 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 80000002; asc     ;;
 1: len 6; hex 000000001020; asc       ;;
 2: len 7; hex 820000010c0110; asc        ;;

RECORD LOCKS space id 26 page no 4 n bits 72 index PRIMARY of table `testDB`.`data` trx id 4130 lock_mode X locks rec but not gap waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 80000001; asc     ;;
 1: len 6; hex 000000001012; asc       ;;
 2: len 7; hex 81000001050110; asc        ;;
```



