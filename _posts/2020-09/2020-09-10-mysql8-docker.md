---
title: Docker 기반의 Mysql8 설치 및 실행
comments: true
tags: [devops, docker]
categories: devops
header:
  teaser: "/assets/images/mysql/mysql_logo.png"
---
본 문서에서는 Docker 환경에서 Mysql 8 버전을 설치하고 실행하는 방법에 대해서 설명하고 있습니다. <br/>



## Mysql 8

```sh
# docker pull mysql:8
```

<br/>

```sh
# docker run --name mysql8 -e MYSQL_ROOT_PASSWORD=1111 -p 3306:3306 -d mysql:8
665afd49d1d958bac99b2f414c78e62cd81ef26253f29c862bac2415360fa0fd
```

<br/>

```sh
# docker ps
CONTAINER ID        IMAGE                                   COMMAND                  CREATED             STATUS              PORTS                                                NAMES
665afd49d1d9        mysql:8                                 "docker-entrypoint.s…"   2 minutes ago       Up 2 minutes        0.0.0.0:3306->3306/tcp, 33060/tcp                    mysql8
```

<br/>

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

mysql> quit;
Bye
```

<br/>

