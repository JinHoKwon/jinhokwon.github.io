---
title: Kafka docker-compose
comments: true
tags: [devops, kafka]
categories: [kafka]
header:
  teaser: "/assets/images/kafka.png"
---

본 문서에서는 docker 기반의 Kafka 를 설치하는 과정에 대해서 소개하고 있습니다.

<br/>

## 1. 사전 준비 과정

### 1-1. 디렉토리 초기화

Kafka의 메세지가 저장될 경로의 디렉토리를 다음과 같이 초기화 합니다.

```sh
# rm -rf /tmp/kafka1 /tmp/kafka2
# mkdir -p /tmp/kafka1 /tmp/kafka2
```

<br/>

### 1-2. docker-compose.yml

```yaml
version: '2' 
services:
  zookeeper:
    hostname: zookeeper
    container_name: zookeeper
    image: zookeeper:3.5.8
    ports:
      - "2181:2181"
  kafka1:
    hostname: kafka1
    container_name: kafka1
    image: wurstmeister/kafka:2.12-2.3.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      #KAFKA_ADVERTISED_HOST_NAME: kafka1
      #KAFKA_ADVERTISED_PORT: 9092
      KAFKA_BROKER_ID: 1
      KAFKA_LISTENERS: INSIDE://:19092,OUTSIDE://kafka1:9092
      KAFKA_ADVERTISED_LISTENERS: INSIDE://:19092,OUTSIDE://kafka1:9092    
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INSIDE
      KAFKA_LOG_DIRS: /kafka1
      KAFKA_CREATE_TOPICS: "test_topic:1:1" # Topic명:Partition개수:Replica개수
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181/kafka
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /tmp/kafka1:/kafka1
  kafka2:
    hostname: kafka2
    container_name: kafka2
    image: wurstmeister/kafka:2.12-2.3.0
    depends_on:
      - zookeeper
      - kafka1
    ports:
      - "9093:9092"
    environment:
      #KAFKA_ADVERTISED_HOST_NAME: kafka2
      #KAFKA_ADVERTISED_PORT: 9092
      KAFKA_BROKER_ID: 2
      KAFKA_LISTENERS: INSIDE://:19093,OUTSIDE://kafka2:9092
      KAFKA_ADVERTISED_LISTENERS: INSIDE://:19093,OUTSIDE://kafka2:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INSIDE
      KAFKA_LOG_DIRS: /kafka2
      KAFKA_CREATE_TOPICS: "test_topic:1:1" # Topic명:Partition개수:Replica개수
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181/kafka
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /tmp/kafka2:/kafka2
  kafka-manager:
  	hostname: kafka-manager
  	container_name: kafka-manager
    image: hlebalbau/kafka-manager:stable
    depends_on:
      - zookeeper
      - kafka1
      - kafka2
    ports:
      - "9000:9000"
    environment:
      ZK_HOSTS: "zookeeper:2181"
      APPLICATION_SECRET: "random-secret"
     #KAFKA_MANAGER_AUTH_ENABLED: "true"		# 인증이
     #KAFKA_MANAGER_USERNAME: username			# 필요한 경우에
     #KAFKA_MANAGER_PASSWORD: password 			# 설정합니다.
```

* KAFKA_CREATE_TOPICS 이 올바르게 적용되기 위해서, 내부 포트번호를 9092로 설정합니다.
* kafka1, kafka2 컨테이너는 KAFKA_ZOOKEEPER_CONNECT 에 설정된  zookeeper:2181/kafka를 공유하기 때문에,   동일한 클러스터안에 포함되게 됩니다. 
만약, kafka1, kafka2를 서로 다른 클러스터로 만드려면, KAFKA_ZOOKEEPER_CONNECT를 다음과 같이 설정하면 됩니다.
```yaml
kafka1:
  KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181/kafka1
kafka2:
  KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181/kafka2
```

<br/>
<br/>

## 2. Kafka

### 2-1. docker 기반의 kafka 실행

```sh
# docker stop kafka1 kafka2 local-zookeeper
# docker rm -f kafka1 kafka2 local-zookeeper
# docker-compose -f ./docker-compose.yaml up -d
```

<br/>

### 2-2. kafka cluster 에 속한 broker 조회 

```sh
# docker exec -i -t kafka1 /opt/kafka/bin/zookeeper-shell.sh zookeeper:2181 ls /
# docker exec -i -t kafka1 /opt/kafka/bin/zookeeper-shell.sh zookeeper:2181 ls /kafka1/brokers/ids
```

<br/>

### 2-3. kafka topic 생성

```sh
$ docker exec -i -t kafka1 /opt/kafka/bin/kafka-topics.sh \
--bootstrap-server kafka1:9092 \
--create --topic testTopic \
--replication-factor 1 \
--partitions 3
```

<br/>

### 2-4. kafka topic 조회

```sh
$ docker exec -i -t kafka1 /opt/kafka/bin/kafka-topics.sh \
--bootstrap-server kafka1:9092 \
--list
```

<br/>

### 2-5. kafka log 조회

```sh
$ docker exec -i -t kafka1 \
tail -f /opt/kafka_2.12-2.3.0/logs/server.log
```

<br/>

<br/>

## 3. Kafka Manager

### 3-1. Add Cluster

* Cluster Name : myKafkaCluster
* Cluster Zookeeper Hosts : zookeeper:2181/kafka
* Kafka Version : 2.3.0
* Enable JMX Polling (Set JMX_PORT env variable before starting kafka server) : Check
* JMX with SSL : Check
* Poll consumer information (Not recommended for large # of consumers if ZK is used for offsets tracking on older Kafka versions) : Check
* Enable Active OffsetCache (Not recommended for large # of consumers) : Check
* Display Broker and Topic Size (only works after applying [this patch](https://issues.apache.org/jira/browse/KAFKA-1614)) : Check
* brokerViewThreadPoolSize : 2
* offsetCacheThreadPoolSize : 2
* kafkaAdminClientThreadPoolSize : 2
<br/>

### 3-2. Add Partitions

Cluster > myKafkaCluster > Topics > test_topic > Add Partitions
* Topic : test_topic
* Partitions : 3
* Brokers : kafka1, kafka2
Add Partitons 버튼 클릭

<br/>

### 3-3. Manual Partition Assignments

Cluster > myKafkaCluster > Topics > test_topic > Manual Partition Assignments

* Partition 0 : broker 2
* Partition 1 : broker 2
* Partition 2 : broker 1

Save Partition Assignments 버튼 클릭

그리고

Cluster > myKafkaCluster > Topics > test_topic > Run Reassign Partitions 버튼을 클릭하면 

Topic > Partitions by Broker 화면에 변경된 Partition 정보가 표시됩니다.



