---
title: Redis cluster
tags: [devops, redis]
categories: redis
header:
  teaser: "/assets/images/redis.png"
---

## 1. Redis cluster 개요

* Redis cluster는 v3.0 이상 버전에서만 지원됩니다.

* 최대 1000 대의 노드까지 확장 할 수 있습니다.

* 클러스터가 활성화 된 이후에도 노드의 추가 및 삭제가 가능합니다.

* 특정 키값을 저장할 위치를 선정하기 위해서 CRC16 해쉬 함수를 사용합니다. 

  ```sh
  HashSlot = CRC16(Key) % 16384
  ```

* Redis cluster 환경에서 복제서버가 있는 경우, Fail over 가 자동으로 진행됩니다. (약 1분 이내 fail over가 진행됨.)

* Redis cluster 환경에서는 더 이상 센티넬이 필요하지 않습니다. 



## 2. Redis cluster 구축

#### 2-1. Redis 컴파일 및 설치

```sh
# curl -O -L http://download.redis.io/releases/redis-5.0.5.tar.gz
# tar xvfz redis-5.0.5.tar.gz
# cd deps
# make hiredis jemalloc linenoise lua
# cd ..
# make

# mkdir -p /usr/local/redis/bin \
  /usr/local/redis/logs \
  /usr/local/redis/conf \
  /usr/local/redis/pids \
  /usr/local/redis/db
  
# cp src/redis-cli /usr/local/redis/bin/ &&
  cp src/redis-server /usr/local/redis/bin/ &&
  cp src/redis-sentinel /usr/local/redis/bin/ &&
  cp redis.conf /usr/local/redis/conf/
  
# cp -rf /usr/local/redis /usr/local/redis1 &&
  cp -rf /usr/local/redis /usr/local/redis2 && 
  cp -rf /usr/local/redis /usr/local/redis3 && 
  cp -rf /usr/local/redis /usr/local/redis4 && 
  cp -rf /usr/local/redis /usr/local/redis5 && 
  cp -rf /usr/local/redis /usr/local/redis6 
```



#### 2-2. Redis1 ~ Redis6 설정

/usr/local/redis[1-6]/conf/redis.conf 파일 수정

이 때, port, pidfile, logfile, cluster-config-file, dir 설정은 각각의 redis 마다 경로를 달리 설정해 주어야 합니다.

```ini
port 7001
bind 0.0.0.0

pidfile "/usr/local/redis1/pids/redis.pid"
logfile "/usr/local/redis1/logs/redis.log"

# 클러스터 활성화
cluster-enabled yes

# 클러스터 상태 기록 파일. 자동 관리되므로 직접 편집하면 안됨.
cluster-config-file /usr/local/redis1/conf/nodes.conf

# 장애 판단 시간
# 밀리세컨드 (msec)
cluster-node-timeout 30000

# 기본 값인 yes로 설정한다면, key 스페이스의 일정 비율이 노드에 남아있지 않을 때에 클러스터가 쓰기 허용을 중지할 수 있다.
# 이 옵션을 no로 설정하면 키의 하위 집합에 대한 요청만 처리할 수 있는 경우에도 클러스터는 여전히 쿼리를 받는다.
cluster-require-full-coverage yes

# 작업 디렉토리 설정
dir "/usr/local/redis1/db"

daemonize no
```



#### 2-3. Redis1 ~ Redis6 서버 실행

```sh
# /usr/local/redis1/bin/redis-server /usr/local/redis1/conf/redis.conf
# /usr/local/redis2/bin/redis-server /usr/local/redis2/conf/redis.conf
# /usr/local/redis3/bin/redis-server /usr/local/redis3/conf/redis.conf
# /usr/local/redis4/bin/redis-server /usr/local/redis4/conf/redis.conf
# /usr/local/redis5/bin/redis-server /usr/local/redis5/conf/redis.conf
# /usr/local/redis6/bin/redis-server /usr/local/redis6/conf/redis.conf
```



#### 2-4. Redis1 ~ Redis6 클러스터 초기화

```sh
# redis-cli -c -h 192.168.56.21 -p 7001 flushall && redis-cli -c -h 192.168.56.21 -p 7001 cluster reset
# redis-cli -c -h 192.168.56.21 -p 7002 flushall && redis-cli -c -h 192.168.56.21 -p 7002 cluster reset
# redis-cli -c -h 192.168.56.21 -p 7003 flushall && redis-cli -c -h 192.168.56.21 -p 7003 cluster reset
# redis-cli -c -h 192.168.56.21 -p 7004 flushall && redis-cli -c -h 192.168.56.21 -p 7004 cluster reset
# redis-cli -c -h 192.168.56.21 -p 7005 flushall && redis-cli -c -h 192.168.56.21 -p 7005 cluster reset
# redis-cli -c -h 192.168.56.21 -p 7006 flushall && redis-cli -c -h 192.168.56.21 -p 7006 cluster reset
```



#### 2-5. Redis cluster 생성

먼저 설정된 192.168.56.21:[7001-7003] 번 서버가 Master로, 

그 이하는 Replica 로 자동으로 설정이 됩니다.

```sh
# /usr/local/redis/bin/redis-cli --cluster-replicas 1 --cluster create \
  192.168.56.21:7001 192.168.56.21:7002 192.168.56.21:7003 \
  192.168.56.21:7004 192.168.56.21:7005 192.168.56.21:7006 
  
  >>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.56.21:7005 to 192.168.56.21:7001
Adding replica 192.168.56.21:7006 to 192.168.56.21:7002
Adding replica 192.168.56.21:7004 to 192.168.56.21:7003
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: daef2da51cc05e6e34261dab970875dca7e4ddd3 192.168.56.21:7001
   slots:[0-5460] (5461 slots) master
M: a9cc4070b0bb561a2c7e2c53e32dfa9f8a4a5018 192.168.56.21:7002
   slots:[5461-10922] (5462 slots) master
M: e7ff69cf6f4bf0d24b633d304e26bca068cd6230 192.168.56.21:7003
   slots:[10923-16383] (5461 slots) master
S: 4abf283ae3138e30d31ca08291819360f3b7c1d6 192.168.56.21:7004
   replicates e7ff69cf6f4bf0d24b633d304e26bca068cd6230
S: 49699999d99d7d0d74c4b542ed520446c5f37c7d 192.168.56.21:7005
   replicates daef2da51cc05e6e34261dab970875dca7e4ddd3
S: 4312709338a49387e691ad772ecf13e3cb995a1f 192.168.56.21:7006
   replicates a9cc4070b0bb561a2c7e2c53e32dfa9f8a4a5018
Can I set the above configuration? (type 'yes' to accept): yes     <-- yes라고 입력하고 엔터

>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
....
>>> Performing Cluster Check (using node 192.168.56.21:7001)
M: daef2da51cc05e6e34261dab970875dca7e4ddd3 192.168.56.21:7001
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: a9cc4070b0bb561a2c7e2c53e32dfa9f8a4a5018 192.168.56.21:7002
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 4abf283ae3138e30d31ca08291819360f3b7c1d6 192.168.56.21:7004
   slots: (0 slots) slave
   replicates e7ff69cf6f4bf0d24b633d304e26bca068cd6230
M: e7ff69cf6f4bf0d24b633d304e26bca068cd6230 192.168.56.21:7003
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 4312709338a49387e691ad772ecf13e3cb995a1f 192.168.56.21:7006
   slots: (0 slots) slave
   replicates a9cc4070b0bb561a2c7e2c53e32dfa9f8a4a5018
S: 49699999d99d7d0d74c4b542ed520446c5f37c7d 192.168.56.21:7005
   slots: (0 slots) slave
   replicates daef2da51cc05e6e34261dab970875dca7e4ddd3
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```





## 3. Redis cluster 상태 조회

#### 3-1. Redis cluster 정보 조회

```sh
# redis-cli -h 127.0.0.1 -p 7001 cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:8
cluster_my_epoch:1
cluster_stats_messages_ping_sent:979
cluster_stats_messages_pong_sent:148
cluster_stats_messages_sent:1127
cluster_stats_messages_ping_received:143
cluster_stats_messages_pong_received:138
cluster_stats_messages_meet_received:5
cluster_stats_messages_received:286
```



#### 3-2. Redis cluster 노드 조회

```sh
# redis-cli -c -h 192.168.56.21 -p 7001 cluster nodes
a9cc4070b0bb561a2c7e2c53e32dfa9f8a4a5018 192.168.56.21:7002@17002 master - 0 1594792793000 2 connected 5461-10922
4abf283ae3138e30d31ca08291819360f3b7c1d6 192.168.56.21:7004@17004 slave e7ff69cf6f4bf0d24b633d304e26bca068cd6230 0 1594792791000 8 connected
daef2da51cc05e6e34261dab970875dca7e4ddd3 192.168.56.21:7001@17001 myself,master - 0 1594792789000 1 connected 0-5460
e7ff69cf6f4bf0d24b633d304e26bca068cd6230 192.168.56.21:7003@17003 master - 0 1594792793083 3 connected 10923-16383
4312709338a49387e691ad772ecf13e3cb995a1f 192.168.56.21:7006@17006 slave a9cc4070b0bb561a2c7e2c53e32dfa9f8a4a5018 0 1594792791000 7 connected
49699999d99d7d0d74c4b542ed520446c5f37c7d 192.168.56.21:7005@17005 slave daef2da51cc05e6e34261dab970875dca7e4ddd3 0 1594792791921 5 connected
```



## 4. Redis cluster 체크

```sh
# redis-cli --cluster check 192.168.56.21:7001
192.168.56.21:7001 (daef2da5...) -> 0 keys | 5461 slots | 1 slaves.
192.168.56.21:7002 (a9cc4070...) -> 0 keys | 5462 slots | 1 slaves.
192.168.56.21:7003 (e7ff69cf...) -> 0 keys | 5461 slots | 1 slaves.
[OK] 0 keys in 3 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.56.21:7001)
M: daef2da51cc05e6e34261dab970875dca7e4ddd3 192.168.56.21:7001
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: a9cc4070b0bb561a2c7e2c53e32dfa9f8a4a5018 192.168.56.21:7002
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 4abf283ae3138e30d31ca08291819360f3b7c1d6 192.168.56.21:7004
   slots: (0 slots) slave
   replicates e7ff69cf6f4bf0d24b633d304e26bca068cd6230
M: e7ff69cf6f4bf0d24b633d304e26bca068cd6230 192.168.56.21:7003
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 4312709338a49387e691ad772ecf13e3cb995a1f 192.168.56.21:7006
   slots: (0 slots) slave
   replicates a9cc4070b0bb561a2c7e2c53e32dfa9f8a4a5018
S: 49699999d99d7d0d74c4b542ed520446c5f37c7d 192.168.56.21:7005
   slots: (0 slots) slave
   replicates daef2da51cc05e6e34261dab970875dca7e4ddd3
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```



## 5. Redis cluster 값 입력 및 조회

```sh
# redis-cli -c -h 192.168.56.21 -p 7001 set foo bar
OK

# redis-cli -c -h 192.168.56.21 -p 7002 get foo
"bar"

# redis-cli -c -h 192.168.56.21 -p 7003 get foo
"bar"

# redis-cli -c -h 192.168.56.21 -p 7004 get foo
"bar"

# redis-cli -c -h 192.168.56.21 -p 7005 get foo
"bar"

# redis-cli -c -h 192.168.56.21 -p 7006 get foo
"bar"
```



## 6. Redis cluster failover

#### 6-1. 7003 master 노드 제거

```sh
# redis-cli -c -h 192.168.56.21 -p 7003 debug segfault
```



#### 6-2. Redis cluster 상태 조회

```sh
# redis-cli -c -h 192.168.56.21 -p 7001 cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:8
cluster_my_epoch:1
cluster_stats_messages_ping_sent:1547
cluster_stats_messages_pong_sent:688
cluster_stats_messages_sent:2235
cluster_stats_messages_ping_received:683
cluster_stats_messages_pong_received:648
cluster_stats_messages_meet_received:5
cluster_stats_messages_received:1336
```



#### 6-3. Redis cluster 노드 조회

최초 slave 였던 7004번 노드가 master 로 승격되었고, 기존 7003번 노드는 fail로 표시됨.

```sh
# redis-cli -c -h 192.168.56.21 -p 7001 cluster nodes
a9cc4070b0bb561a2c7e2c53e32dfa9f8a4a5018 192.168.56.21:7002@17002 master - 0 1594793209451 2 connected 5461-10922
4abf283ae3138e30d31ca08291819360f3b7c1d6 192.168.56.21:7004@17004 master - 0 1594793210000 9 connected 10923-16383
daef2da51cc05e6e34261dab970875dca7e4ddd3 192.168.56.21:7001@17001 myself,master - 0 1594793209000 1 connected 0-5460
e7ff69cf6f4bf0d24b633d304e26bca068cd6230 192.168.56.21:7003@17003 master,fail - 1594793150508 1594793147369 3 disconnected
4312709338a49387e691ad772ecf13e3cb995a1f 192.168.56.21:7006@17006 slave a9cc4070b0bb561a2c7e2c53e32dfa9f8a4a5018 0 1594793209000 7 connected
49699999d99d7d0d74c4b542ed520446c5f37c7d 192.168.56.21:7005@17005 slave daef2da51cc05e6e34261dab970875dca7e4ddd3 0 1594793211484 5 connected
```





#### 6-4. 7003 master 노드 재시작

```sh
# /usr/local/redis3/bin/redis-server /usr/local/redis3/conf/redis.conf
```





## 7. Redis cluster 노드 조회

7003 노드는 slave 로 다시 클러스터에 연결되었음.

```sh
# redis-cli -c -h 192.168.56.21 -p 7001 cluster nodes
a9cc4070b0bb561a2c7e2c53e32dfa9f8a4a5018 192.168.56.21:7002@17002 master - 0 1594793379000 2 connected 5461-10922
4abf283ae3138e30d31ca08291819360f3b7c1d6 192.168.56.21:7004@17004 master - 0 1594793380142 9 connected 10923-16383
daef2da51cc05e6e34261dab970875dca7e4ddd3 192.168.56.21:7001@17001 myself,master - 0 1594793378000 1 connected 0-5460
e7ff69cf6f4bf0d24b633d304e26bca068cd6230 192.168.56.21:7003@17003 slave 4abf283ae3138e30d31ca08291819360f3b7c1d6 0 1594793378045 9 connected
4312709338a49387e691ad772ecf13e3cb995a1f 192.168.56.21:7006@17006 slave a9cc4070b0bb561a2c7e2c53e32dfa9f8a4a5018 0 1594793379119 7 connected
49699999d99d7d0d74c4b542ed520446c5f37c7d 192.168.56.21:7005@17005 slave daef2da51cc05e6e34261dab970875dca7e4ddd3 0 1594793377014 5 connected
```





