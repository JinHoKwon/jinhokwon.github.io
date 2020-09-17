

본 문서에서는 서로 다른 세션에서 동시에 트랜잭션을 진행할  경우 발생 할 수 있는 데드락 상황에 대해서 설명하고 있습니다.



<br/>

## 2. Deadlock 관련 설정

<br/>

### 2-1. Shared Lock waiting timeout 설정

```sql
mysql> show variables like 'innodb_lock%';
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| innodb_lock_wait_timeout | 50    |
+--------------------------+-------+
1 row in set (0.01 sec)
```

<br/>

### 2-2. Isolation level 설정

```sql
mysql> show variables like 'tx_isolation';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| tx_isolation  | REPEATABLE-READ |
+---------------+-----------------+
1 row in set (0.00 sec)
```

<br/>

## 3. Deadlock 샘플 데이터 생성

### 3-1. 샘플 데이터베이스 생성

```sql
DROP IF EXISTS DATABASE testDB;
CREATE DATABASE testDB DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
USE testDB;
```

<br/>

### 3-2. 샘플 테이블 생성

```sql
CREATE TABLE data (id INT PRIMARY KEY) ENGINE = INNODB;

CREATE TABLE parent (
    id INT NOT NULL AUTO_INCREMENT,
    age INT NOT NULL,
    PRIMARY KEY (id)
) ENGINE=INNODB;

CREATE TABLE child (
    id INT NOT NULL AUTO_INCREMENT,
    age INT NOT NULL,
    parent_id INT NOT NULL,
    PRIMARY KEY (id),
    KEY parent_id (parent_id),
    CONSTRAINT fk_parent_id FOREIGN KEY (parent_id) REFERENCES parent (id)
) ENGINE=INNODB;
```

<br/>

### 3-3. 샘플 데이터 입력

```sql
INSERT INTO parent (id, age) VALUES (1, 50);
INSERT INTO parent (id, age) VALUES (2, 60);
INSERT INTO child (id, age, parent_id) VALUES (1, 20, 1);
INSERT INTO child (id, age, parent_id) VALUES (2, 20, 1);
```

<br/>

<br/>

## 4. Deadlock (외래키가 없는 경우)

<br/>

### 4-1. 가장 처음 Lock을 획득한 세션에서 커밋을 하는 경우

가장 처음 Lock을 획득한 세션에서만 정상적으로 데이터가 입력되고,

그 외 나머지 세션에서는 `ERROR 1062 (23000): Duplicate entry` 오류가 발생합니다. (기대했던 결과)

| No   | Session1                    | Session2                                                     | Session3                                                     |
| :--- | --------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | USE testDB                  |                                                              |                                                              |
| 2    | BEGIN                       |                                                              |                                                              |
| 3    | INSERT INTO data VALUES (1) |                                                              |                                                              |
| 4    |                             | USE testDB                                                   | USE testDB                                                   |
| 5    |                             | BEGIN                                                        | BEGIN                                                        |
| 6    |                             | INSERT INTO data VALUES (1)                                  | INSERT INTO data VALUES (1)                                  |
| 7    |                             | Shared Lock WAITING... (약 50초)                             | Shared Lock WAITING... (약 50초)                             |
| 8    | COMMIT                      |                                                              |                                                              |
| 9    |                             | ERROR 1062 (23000): Duplicate entry '1' for key 'data.PRIMARY' | ERROR 1062 (23000): Duplicate entry '1' for key 'data.PRIMARY' |
| 10   |                             |                                                              |                                                              |

<br/>

### 4-2. Deadlock 상황 재현 결과

```sql
mysql> SELECT * FROM data;
+----+
| id |
+----+
|  1 |
+----+
1 row in set (0.00 sec)
```

<br/>

### 4-3. 가장 처음 Lock을 획득한 세션에서 롤백을 하는 경우



가장 처음 Lock을 획득한 세션에서 롤백을 하는 경우, 그 외 세션에서는 Deadlock 되거나, 정상적으로 데이터가 입력이 됩니다.

> When session 1 rolls back, it releases its exclusive lock on the row and the queued shared lock requests for sessions 2 and 3 are granted. At this point, sessions 2 and 3 deadlock.

https://dev.mysql.com/doc/refman/8.0/en/innodb-locks-set.html



| No   | Session1                    | Session2                    | Session3                    |
| :--- | --------------------------- | --------------------------- | --------------------------- |
| 1 | USE testDB      |                             |                             |
| 2 | BEGIN                       |                             |                             |
| 3 | INSERT INTO data VALUES (1) |                             |                             |
| 4 |                             | USE testDB                  | USE testDB                  |
| 5 |                             | BEGIN                       | BEGIN                       |
| 6 |                             | INSERT INTO data VALUES (1) | INSERT INTO data VALUES (1) |
| 7 |  | Shared Lock WAITING... (약 50초) | Shared Lock WAITING... (약 50초) |
| 8 | ROLLBACK |  |  |
| 9 |                             | ERROR 1213 (40001): Deadlock | Query OK, 1 row affected. |
| 10 |                             |                             | COMMIT |

<br/>

### 4-4. Deadlock 상황 재현 결과

```sql
mysql> SELECT * FROM data;
+----+
| id |
+----+
|  1 |
+----+
1 row in set (0.00 sec)
```

<br/>

## 5. Deadlock (외래키가 있는 경우)

<br/>

### 5-1. 부모 테이블의 행을 UPDATE 한 상황에서 자식 테이블에 INSERT 를 하는 경우

부모 세션에서 접근한 동일한 ROW에 자식 세션에서 INSERT / UPDATE 하는 경우에는 Shared Lock Wating이 발생하지만,

부모 세션과 자식 세션이 서로 다른 ID에 접근하는 경우에는 정상적으로 커밋됩니다.

| No   | Parent session                                     | Child session                                            |
| ---- | -------------------------------------------------- | -------------------------------------------------------- |
|      | USE testDB                                         |                                                          |
|      | BEGIN                                              |                                                          |
|      | UPDATE parent SET age = age + 1 WHERE id = **`1`** |                                                          |
|      |                                                    | USE testDB                                               |
|      |                                                    | BEGIN                                                    |
|      |                                                    | INSERT INTO child (age, parent_id) VALUES (22, **`1`**); |
|      |                                                    | Shared Lock Waiting ... (약 50초)                        |
|      | COMMIT                                             |                                                          |
|      |                                                    | Query OK, 1 row affected                                 |
|      |                                                    | COMMIT                                                   |
|      |                                                    |                                                          |
|      |                                                    |                                                          |
|      |                                                    |                                                          |

<br/>

### 5-2. 실행 결과

```sql
mysql> SELECT * FROM parent p INNER JOIN child c ON p.id = c.parent_id;
+----+-----+----+-----+-----------+
| id | age | id | age | parent_id |
+----+-----+----+-----+-----------+
|  1 |  51 |  1 |  20 |         1 |
|  1 |  51 |  2 |  20 |         1 |
|  1 |  51 |  3 |  22 |         1 |
+----+-----+----+-----+-----------+
3 rows in set (0.00 sec)
```

<br/>

### 5-3.  자식행을 잠그는 경우 이에 관련된 부모행은 slock이 걸리는데 이로 인해 데드락이 발생

> One common cause for deadlocks when using InnoDB tables is from the existence of foreign key constraints and the shared locks (S-lock) they acquire on referenced rows.

| No   | Parent session                               | Child session                            |
| ---- | -------------------------------------------- | ---------------------------------------- |
|      | USE testDB                                   |                                          |
|      | BEGIN                                        |                                          |
|      | UPDATE parent SET age = age + 1 WHERE id = 2 |                                          |
|      |                                              | USE testDB                               |
|      |                                              | BEGIN                                    |
|      |                                              | UPDATE child SET parent_id=2 WHERE id=2  |
|      |                                              | Shared Lock Waiting ... (약 50초)        |
|      | UPDATE child SET parent_id=2 WHERE id=2      |                                          |
|      | ERROR 1213 (40001): Deadlock found           |                                          |
|      |                                              | Query OK, 1 row affected                 |
|      |                                              | Rows matched: 1  Changed: 1  Warnings: 0 |
|      |                                              | COMMIT                                   |
|      |                                              |                                          |

<br/>

## 6.  Deadlock과 Isolation level

<br/>

### 6-1. Insert 동작중에 UPDATE, DELETE가 동시에 진행되는 경우

##### Session 1 에서 많은 양의 데이터를 추가하는 동안

```sql
mysql> insert into activity_test_stat2
    -> select
    ->     act_type,
    ->     to_uid,
    ->     act_time,
    ->     to_user_name,
    ->     before_user_name,
    ->     count(*) cnt
    -> from activity_test
    -> group by act_type, to_uid, act_time,
    ->     to_user_name, before_user_name;
```

<br/>

##### Session 2 에서 UPDATE 또는 DELETE 를 수행하는 경우에

```sql
mysql> update activity_test set ACT_TYPE = 105 limit 10;
```

<br/>

대기 상태에 빠지거나,

```sql
mysql> show processlist\G
************************* 1. row *************************
     Id: 255867
   User: root
   Host: localhost
     db: snsfeed
Command: Query
   Time: 1
  State: Updating
   Info: update activity_test set ACT_TYPE = 105 limit 10
************************* 2. row *************************
     Id: 255962
   User: root
   Host: localhost
     db: snsfeed
Command: Query
   Time: 2
  State: Copying to tmp table
   Info: insert into activity_test_stat2 select act_type,
```

<br/>

##### 다음과 같이 데드락이 발생하게 됩니다.

```sql
mysql> delete from activity_test limit 10;
ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction
```

<br/>

그리고, 이러한 경우에는 Insert into Select 경우 Isolation Level을 READ-COMMITED나 READ-UNCOMMITED로 변경하여 해결할 수 있습니다.

<br/>



## 9. Reference

https://dev.mysql.com/doc/refman/8.0/en/innodb-locks-set.html

https://gywn.net/2012/05/mysql-transaction-isolation-level/

