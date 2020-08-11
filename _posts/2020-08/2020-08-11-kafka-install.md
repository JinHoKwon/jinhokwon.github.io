---
title: Kafka 설치
tags: [devops, kafka]
comments: true
categories: kafka
header:
  teaser: "/assets/images/kafka/kafka_logo.png"
---

본 문서는 Centos 7 환경에서 kafka 2.6 버전을 설치하는 과정에 대해서 설명하고 있습니다.<br/>



## 1. Kafka 설치

#### 1-1.JDK 14 설치

[여기](/devops/devops-java-install/)를 참조하여 JDK 14를 설치합니다.

<br/>


#### 1-2. Kafka 사용자 추가

```sh
# groupadd public
# useradd -s /sbin/nologin -G public kafka
```

<br/>

#### 1-3. Kafka 설치

```sh
# mkdir -p /tmp/download
# cd /tmp/download
# curl -O -L http://apache.tt.co.kr/kafka/2.6.0/kafka_2.13-2.6.0.tgz
# tar xvfz kafka_2.13-2.6.0.tgz
# mv kafka_2.13-2.6.0 /usr/local/kafka
# chown -R kafka:kafka /usr/local/kafka
```

<br/>

#### 1-4. Kafka 버전확인

```sh
# find /usr/local/kafka/libs -name kafka_*.jar | grep kafka
/usr/local/kafka/libs/kafka_2.13-2.6.0.jar
/usr/local/kafka/libs/kafka_2.13-2.6.0-sources.jar
/usr/local/kafka/libs/kafka_2.13-2.6.0-javadoc.jar
/usr/local/kafka/libs/kafka_2.13-2.6.0-test.jar
/usr/local/kafka/libs/kafka_2.13-2.6.0-test-sources.jar
/usr/local/kafka/libs/kafka_2.13-2.6.0-scaladoc.jar
```

<br/>

#### 1-5. Zookeeper 버전 확인

```sh
# find /usr/local/kafka/libs -name zoo*.jar | grep zoo
/usr/local/kafka/libs/zookeeper-3.5.8.jar
/usr/local/kafka/libs/zookeeper-jute-3.5.8.jar
```

<br/>

#### 1-6. Kafka 관련 디렉토리 생성

```sh
# mkdir -p /var/log/kafka
# mkdir -p /disk1/logs /disk2/logs /disk3/logs
# chown -R kafka:public /var/log/kafka /disk1 /disk2 /disk3
# chmod -R 2775 /var/log/kafka /disk1 /disk2 /disk3
```

<br/>

#### 1-7. Zookeeper 관련 디렉토리 생성

```sh
# mkdir -p /var/log/zk /var/zk/data/logs
# chown -R kafka:public /var/log/zk /var/zk
# chmod -R 2775 /var/log/zk /var/zk
```

`2` - GID 비트 활성화 (서브 디렉토리 생성시, 그룹 권한을 물려받고, 하위 디렉토리 또한 GID 비트를 상속함.)

`7` - 사용자 권한을 **rwx** 로 설정

`7` - 그룹 권한을 **rwx** 로 설정

`5` - Other 권한을 **rx** 로 설정

<br/>



## 2. Kafka 환경변수 설정



#### 2-1. KAFKA_HOME 환경 변수 설정

##### /etc/profile 파일 편집

```sh
export KAFKA_HOME=/usr/local/kafka
export ZOOKEEPER_HOME=/usr/local/kafka
export PATH=$PATH:$KAFKA_HOME/bin
```

<br/>

#### 2-2. 환경변수 리로드

```sh
# source /etc/profile
```

<br/>

## 3. Zookeeper 설정

#### 3-1. /usr/local/kafka/config/zookeeper.properties 설정

```properties
dataDir=/var/zk/data
dataLogDir=/var/zk/data/logs
clientPort=2181			
maxClientCnxns=0
tickTime=2000
initLimit=5
syncLimit=2
server.1=kafka1.net:2888:3888
admin.enableServer=true
admin.serverPort=8088
```
* clientPort : 클라이언트 연결포트
* dataDir : snapshot 저장 디렉토리
* dataLogDir : 트랙잭션 로그 저장 디렉토리
* tickTime : tick 단위 시간을 설정. (단위 : 밀리초)
* initLimit : follower가 zookeeper 리더에 접속해서 동기화 하는 횟수 (데이터 양이 많을수록 크게 설정)
  제한 횟수 초과시 timeout
* syncLimit : 리더를 제외한 zookeeper 노드 사이에 동기화 하는 횟수
  예를 들어 tickTime=2000이고, initLimit=5인 경우, 2초 간격으로 5회 동안 재시도를 하게됨.
* server.1 : zookeeper가 ensemble을 이루기 위한 서버의 정보
  2888은 동기화를 위한 포트, 3888은 클러스터 구성시 leader를 선출하기 위한 포트
  zookeeper를 싱글로 운영할 경우 설정하지 않아도 됨.
  만약, zookeeper ensemble을 localhost에 구축하기 위해서는 각각의 zookeeper 마다 포트 번호가 달라야 함.

  ```properties
  server.1=localhost:2888:3888
  server.2=localhost:2889:3889
  server.3=localhost:2890:3890
  ```


<br/>

#### 3-2. 각 장비별 zookeeper id 파일 생성

zookeeper 설정중, dataDir 로 설정된 경로의 하위에 zookeeper id 파일을 생성함.

이 때, zookeeper id는 server.1에 명시된 숫자와 동일해야 하며,

zookeeper를 싱글로 운영하는 경우는 설정하지 않아도 됨.

```sh
# echo 1 > /var/zk/data/myid
```

<br/>

#### 3-3. $KAFKA_HOME/bin/zookeeper-server-start.sh 파일 편집

```sh
export LOG_DIR=/var/log/zk
export GC_LOG_ENABLED=true
export KAFKA_HEAP_OPTS="-Xms6g -Xmx6g"
```

<br/>

#### 3-4. $KAFKA_HOME/bin/zookeeper-server-stop.sh 파일 편집

```sh
export LOG_DIR=/var/log/zk
```

<br/>

#### 3-5. zookeeper service 생성

##### /usr/lib/systemd/system/zookeeper.service 파일 생성

```ini
# cat /usr/lib/systemd/system/zookeeper.service
# Systemd unit file for zookeeper
[Unit]
Description=Zookeeper for Kafka
After=syslog.target network.target

[Service]
Type=forking
LimitNOFILE=1048576
LimitMEMLOCK=infinity
LimitCORE=infinity
LimitNPROC=1048576

ExecStart=/usr/local/kafka/bin/zookeeper-server-start.sh -daemon /usr/local/kafka/config/zookeeper.properties
ExecStop=/usr/local/kafka/bin/zookeeper-server-stop.sh

User=kafka
Group=kafka
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```

<br/>

#### 3-6. zookeeper 서비스 실행

```sh
# systemctl daemon-reload
# systemctl start zookeeper
```

<br/>

#### 3-7. zookeeper 서비스 로그 확인

```sh
# tail -f /var/log/zk/server.log 
[2020-08-11 14:37:43,749] INFO Started ServerConnector@3f197a46{HTTP/1.1,[http/1.1]}{0.0.0.0:8088} (org.eclipse.jetty.server.AbstractConnector)
[2020-08-11 14:37:43,749] INFO Started @1059ms (org.eclipse.jetty.server.Server)
[2020-08-11 14:37:43,749] INFO Started AdminServer on address 0.0.0.0, port 8088 and command URL /commands (org.apache.zookeeper.server.admin.JettyAdminServer)
[2020-08-11 14:37:43,755] INFO Using org.apache.zookeeper.server.NIOServerCnxnFactory as server connection factory (org.apache.zookeeper.server.ServerCnxnFactory)
[2020-08-11 14:37:43,758] INFO Configuring NIO connection handler with 10s sessionless connection timeout, 1 selector thread(s), 8 worker threads, and 64 kB direct buffers. (org.apache.zookeeper.server.NIOServerCnxnFactory)
[2020-08-11 14:37:43,760] INFO binding to port 0.0.0.0/0.0.0.0:2181 (org.apache.zookeeper.server.NIOServerCnxnFactory)
[2020-08-11 14:37:43,795] INFO zookeeper.snapshotSizeFactor = 0.33 (org.apache.zookeeper.server.ZKDatabase)
[2020-08-11 14:37:43,799] INFO Snapshotting: 0x0 to /var/zk/data/version-2/snapshot.0 (org.apache.zookeeper.server.persistence.FileTxnSnapLog)
[2020-08-11 14:37:43,802] INFO Snapshotting: 0x0 to /var/zk/data/version-2/snapshot.0 (org.apache.zookeeper.server.persistence.FileTxnSnapLog)
[2020-08-11 14:37:43,822] INFO Using checkIntervalMs=60000 maxPerMinute=10000 (org.apache.zookeeper.server.ContainerManager)
```



<br/>

#### 3-8. zookeeper 실행 여부 확인

```sh
# $KAFKA_HOME/bin/zookeeper-shell.sh 192.168.56.11:2181 ls /
Connecting to 192.168.56.11:2181

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zookeeper]
```

<br/>

## 4. Kafka 설정

#### 4-1. $KAFKA_HOME/bin/kafka-run-class.sh 파일 편집

```sh
export JAVA_HOME=/usr/local/jdk14
```

<br/>



#### 4-2. $KAFKA_HOME/bin/kafka-server-start.sh 파일 편집

```sh
export JAVA_HOME=/usr/local/jdk14
export JMX_PORT=9999
export KAFKA_HEAP_OPTS="-XX:MetaspaceSize=96m -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:G1HeapRegionSize=16M -XX:MinMetaspaceFreeRatio=50 -XX:MaxMetaspaceFreeRatio=80"
export GC_LOG_ENABLED=true
export LOG_DIR=/var/log/kafka
```

<br/>

#### 4-3. $KAFKA_HOME/bin/kafka-server-stop.sh 파일 편집

```sh
export LOG_DIR=/var/log/kafka
```

<br/>

#### 4-4. $KAFKA_HOME/config/server.properties 파일 편집

```properties
############################# Server Basics #############################
broker.id=1
############################# Socket Server Settings #############################
listeners=PLAINTEXT://0.0.0.0:9092
advertised.listeners=PLAINTEXT://192.168.56.11:9092

num.network.threads=3

# IO 를 처리하는 스레드 수로, 로그를 저장하는데 사용되는 디스크 수와 동일하게 지정하는것을 권장
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600

# IO 쓰레드가 처리하는 동안 지연된 메세지를 보관하는 큐의 크기
queued.max.requests=500

############################# Log Basics #############################

log.dirs=/disk1/logs/,/disk2/logs,/disk3/logs
num.partitions=1

# broker가 시작/중지 할 때, 다른 broker와 sync를 맞추기 위해서 log 디렉토리를 스캔하기 위해서 실행되는 스레드 수
# 값이 클수록 broker가 빨리 구동되며, 장애 복구 시간이 줄어듬.
num.recovery.threads.per.data.dir=1

auto.create.topics.enable=false
delete.topic.enable=true


############################# Internal Topic Settings  #############################
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

############################# Message Settings #############################
message.max.bytes=20485760
replica.fetch.max.bytes=20485760

############################# Log Flush Policy #############################
log.flush.interval.messages=10000
log.flush.scheduler.interval.ms=1000
log.flush.interval.ms=1000

############################# Log Retention Policy #############################
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
log.cleaner.enable=true
log.cleanup.policy=delete

############################# Zookeeper #############################
zookeeper.connect=192.168.56.11:2181


############################# Replication #############################
# leader 브로커에서 메세지를 복제할 때 사용할 스레드 수로
# 스레드 수를 늘리면 IO를 병령 처리하기 때문에 효율도 높아짐.
num.replica.fetchers=1

# auto.create.topics.enable이 true인 경우, 기본 적용되는 메시지 복제본의 갯수
default.replication.factor=1

############################# Durability and hardening #############################
# min.insync.replicas 옵션은 프로듀서가 acks=all로 설정하여 메시지를 보낼 때, 
# write를 성공하기 위한 최소 복제본의 수를 의미
# 이 때, 복제와 관련된 설정값 브로커수보다 작게 설정해야 합니다.
min.insync.replicas=1
unclean.leader.election.enable=false
```

각 장비별 나머지 설정은 동일하며, broker.id 설정값과 listeners 설정값만 장비별로 다르게 설정합니다.



<br/>

#### 4-5. kafka service 파일 생성

##### /usr/lib/systemd/system/kafka.service 파일 편집

```ini
# cat /usr/lib/systemd/system/kafka.service
# Systemd unit file for kafka
[Unit]
Description=Kafka
After=syslog.target network.target

[Service]
Type=forking

LimitNOFILE=1048576
LimitMEMLOCK=infinity
LimitCORE=infinity
LimitNPROC=1048576

ExecStart=/usr/local/kafka/bin/kafka-server-start.sh -daemon /usr/local/kafka/config/server.properties
ExecStop=/usr/local/kafka/bin/kafka-server-stop.sh

User=kafka
Group=kafka
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```

<br/>

#### 4-6. Kafka 서비스 실행

```sh
# systemctl daemon-reload
# systemctl start zookeeper
```

<br/>

#### 4-7. Kafka 실행 여부 확인

```sh
# $KAFKA_HOME/bin/zookeeper-shell.sh 192.168.56.11:2181 ls /brokers/ids
Connecting to 192.168.56.11:2181

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[1]
```

<br/>



## 5. Kafka CRUD



#### 5-1. Kafka 토픽 생성

```sh
# $KAFKA_HOME/bin/kafka-topics.sh --bootstrap-server 192.168.56.11:9092 --create  --replication-factor 1 --partitions 1 --topic testTopic
Created topic testTopic.
```

※ broker가 1개 이상인 경우에만 replication-factor의 갯수를 1개 이상으로 설정할 수 있음.

<br/>

#### 5-2. Kafka 토픽 리스트

```sh
# $KAFKA_HOME/bin/kafka-topics.sh --bootstrap-server 192.168.56.11:9092 --list
testTopic
```

<br/>

#### 5-3. Kafka 토픽 삭제

```sh
# $KAFKA_HOME/bin/kafka-topics.sh --bootstrap-server 192.168.56.11:9092 --delete --topic testTopic
```

<br/>

#### 5-4. Kafka 레코드 삭제

```json
{
    "partitions" : [
        {
            "topic" : "testTopic",
            "partition" : 0,
            "offset" : -1
        }
    ],
    "version" : 1
}
```

```sh
# $KAFKA_HOME/bin/kafka-delete-records.sh --bootstrap-server 192.168.56.11:9092 --offset-json-file ./offsetfile.json
```

<br/>

#### 5-5. Kafka Producer

```sh
# kafka-console-producer.sh --broker-list 192.168.56.11:9092 --topic testTopic
>abcd
>1234
```

<br/>

#### 5-6. Consumer

```sh
# kafka-console-consumer.sh --bootstrap-server 192.168.56.11:9092 --topic testTopic --from-beginning
abcd
1234
```

<br/>

#### 5-7. consumer offset 확인

```sh
# $KAFKA_HOME/bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list 192.168.56.11:9092 --topic testTopic --time -1
testTopic:0:2
```
<br/>

## 6. Kafka 삭제

```sh
# systemctl stop kafka
# systemctl stop zookeeper
# systemctl disable kafka
# systemctl disable zookeeper
# rm /usr/lib/systemd/system/zookeeper.service
# rm /usr/lib/systemd/system/kafka.service

# systemctl list-units --type=service | grep kafka
# systemctl list-units --type=service | grep zookeeper

# rm -rf /usr/local/kafka /var/log/zk /var/log/kafka /var/zk /disk1/logs/ /disk2/logs /disk3/logs
```






