---
title: MySQL SELECT FOR UPDATE 의 이해
tags: [devops, mysql]
comments: true
categories: devops
header:
  teaser: "/assets/images/mysql/mysql_logo.png"
---



## 1. SELECT FOR UPDATE 

`SELECT FOR UPDATE`쿼리는 가정 먼저 LOCK을 획득한 SESSION의 SELECT 된 ROW들이 <br/>

UPDATE 쿼리후 COMMIT 되기 이전까지 다른 SESSION들은 해당 ROW들을 수정하지 못하도록 하는 기능입니다.<br/>

<br/>

<br/>

## 2. SELECT FOR UPDATE 실습



서로 다른 SESSION에서 동시에 SELECT FOR UPDATE 쿼리를 실행하였을 때,

어떤식으로 전개가 되는지 설명하는 실습니다. 

```sql
SESSION#1> SELECT * FROM book WHERE id=1 FOR UPDATE;
SESSION#2> SELECT * FROM book WHERE id=1 FOR UPDATE;
SESSION#3> SELECT * FROM book WHERE id=2 FOR UPDATE;
```

결과를 요약하면,

동일한 ROW를 UPDATE하는 SESSION#1, SESSION#2는 

먼저 LOCK을 획득한 SESSION#1 의 COMMIT 이후에 SESSION#2의 업데이트 쿼리가 실행되고,

SESSION#3에서 선택한 ROW는 SESSION#1, SESSION#2의 겹치지지 않는 다른 ROW이기 때문에,

대기 (waiting) 없이 쿼리가 실행됩니다.

<br/>



#### 2-1. SESSION #1 

가장 먼저 LOCK을 획득한 SESSION #1이 COMMIT이 완료 되기 전까지 <br/>

다른 SESSION 들은 해당 ROW를 수정 할 수 없고 읽기는 가능합니다.

```sql
mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT * FROM book WHERE id=1 FOR UPDATE;
+----+-------+-------+
| id | title | price |
+----+-------+-------+
|  1 | Tile  | 10000 |
+----+-------+-------+
1 row in set (0.00 sec)

mysql> UPDATE book SET price=10010 WHERE id=1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> COMMIT;
Query OK, 0 rows affected (0.01 sec)
```

<br/>

#### 2-2. SESSION #2

동일한 ROW(id=1)를 서로 다른 SESSION이 접근하게 된 경우,<br/>

먼저 접근한 SESSION #1의 COMMIT 이 완료될때까지 SESSION #2는 <br/>

LOCK WAIT TIME 동안 대기하게 됩니다.<br/>

이후, SESSION #1이 COMMIT 을 하게되면, <br/>

SESSION #2의 대기가 풀리고, 이후 UPDATE 쿼리가 실행이 됩니다.

```sql
mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT * FROM book WHERE id=1 FOR UPDATE;

>>>>>>>>>>>>>>>>>>> 대기중 <<<<<<<<<<<<<<<<<<<<<
>>>>>>>>>>>>>>>>>>> 대기중 <<<<<<<<<<<<<<<<<<<<<


+----+-------+-------+
| id | title | price |
+----+-------+-------+
|  1 | Tile  | 10010 |
+----+-------+-------+
1 row in set (49.84 sec)

mysql> UPDATE book SET price=10020 WHERE id=1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> COMMIT;
Query OK, 0 rows affected (0.01 sec)
```

<br/>

#### 2-3. SESSION #3

SESSION #3이 선택한 ROW는 SESSION #1, SESSION #2와 동일한 ROW가 아니기 때문에,<br/>
대기하지 않고, 쿼리가 실행 됩니다.

```sql
mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT * FROM book WHERE id=2 FOR UPDATE;
+----+-----------+-------+
| id | title     | price |
+----+-----------+-------+
|  2 | 타이틀    | 10000 |
+----+-----------+-------+
1 row in set (0.00 sec)

mysql> UPDATE book SET price=10001 WHERE id=2;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)
```



