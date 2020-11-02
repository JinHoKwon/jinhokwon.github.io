---
title: Docker 기반의 Redis6 설치 및 실행
comments: true
tags: [devops, docker, redis]
categories: [devops, redis]
header:
  teaser: "/assets/images/redis.png"
---
본 문서에서는 Docker 환경에서 Redis 6 버전을 설치하고 실행하는 방법에 대해서 설명하고 있습니다. <br/>

## 1. Docker 설치

[여기](/devops/devops-docker-install/)를 참고하여 Docker를 설치합니다.

<br/>

## 2. Redis 6

### 2-1. Redis image pull

```sh
# docker pull redis:6.0
```

<br/>

### 2-2. Run Redis image

```sh
# docker run -d -p 16379:6379 --name redis6 redis:6.0 --appendonly yes --port 6379 --bind "0.0.0.0"
docker8e30ec332a4622337570b57c206c1fbcb5da14fd4d4846e1ccc2685a4d4a5065
```

* **`--bind`** : 바인딩할 주소 

<br/>

### 2-3. Redis container status

```sh
# docker ps
CONTAINER ID      IMAGE                 COMMAND          CREATED          STATUS      PORTS    NAMES
8e30ec332a46  redis:6.0  "docker-entrypoint.s…"   59 seconds ago   Up 59 seconds   6379/tcp   redis6
```

<br/>

### 2-4. Redis Ping

```sh
# docker exec -i -t redis6 redis-cli -h localhost -p 6379 ping
PONG
```

<br/>

```sh
# telnet 127.0.0.1 16379
telnetTrying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
ping
+PONG
quit
+OK
Connection closed by foreign host.
```



<br/>

### 2-5. Redis console  

```sh
# docker exec -i -t redis6 redis-cli -h localhost -p 6379
localhost:6379> ping
PONG
localhost:6379> quit
```

<br/>



### 2-6. Redis log

```sh
# docker logs -f redis6
1:C 21 Sep 2020 23:46:32.203 # Redis version=6.0.8, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 21 Sep 2020 23:46:32.203 # Configuration loaded
1:M 21 Sep 2020 23:46:32.204 * Running mode=standalone, port=6379.
1:M 21 Sep 2020 23:46:32.204 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
1:M 21 Sep 2020 23:46:32.204 # Server initialized
1:M 21 Sep 2020 23:46:32.204 * Ready to accept connections
```



