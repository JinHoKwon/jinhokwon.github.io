---
title: Docker 기반의 Elasticsearch 설치 및 실행
comments: true
tags: [devops, elasticsearch]
categories: devops
header:
  teaser: "/assets/images/elasticsearch.png"
---
본 문서에서는 Docker 환경에서 Elasticsearch 7.9 버전을 설치하고 실행하는 방법에 대해서 설명하고 있습니다. <br/>

## 1. Docker 설치

[여기](/devops/devops-docker-install/)를 참고하여 Docker를 설치합니다.

<br/>

## 2. Elasticsearch 7.9.1 

### 2-1. Elasticsearch 설치

```sh
# docker pull docker.elastic.co/elasticsearch/elasticsearch:7.9.1
```

<br/>

### 2-2. Elasticsearch 실행

```sh
# docker run -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" --name elasticsearch7 docker.elastic.co/elasticsearch/elasticsearch:7.9.1
b5028c898638f74a5aef326899617503b1ef74ee53597cd6c23f50182e4435c6
```

<br/>

### 2-3. Elasticsearch 실행 확인

```sh
# docker ps
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                                            NAMES
0b203ff28fd2        docker.elastic.co/elasticsearch/elasticsearch:7.9.1   "/tini -- /usr/local…"   35 seconds ago      Up 34 seconds       0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   elasticsearch7
```



<br/>

## 3. Kibana 7.9.1 

### 3-1. Kibana 설치

```sh
# docker pull docker.elastic.co/kibana/kibana:7.9.1
```

<br/>

### 3-2. Kibana 실행

Kibana는 실행한 이후, 약 1분 정도 경과한 이후에 Kibana Website에 접속할 수 있습니다.

```sh
# docker run -d --link elasticsearch7:elasticsearch -p 5601:5601 --name kibana7 docker.elastic.co/kibana/kibana:7.9.1
1a9ca43a5123a30c79a36b4e8a554086b25e11ff0da3ad3b9dc9cd4159346cf5
```

<br/>

### 3-3. Kibana 실행 확인

```sh
# docker ps
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                                            NAMES
1a9ca43a5123        docker.elastic.co/kibana/kibana:7.9.1                 "/usr/local/bin/dumb…"   18 seconds ago      Up 17 seconds       0.0.0.0:5601->5601/tcp                           kibana7
b5028c898638        docker.elastic.co/elasticsearch/elasticsearch:7.9.1   "/tini -- /usr/local…"   2 minutes ago       Up 2 minutes        0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   elasticsearch7
```

<br/>

## 4. Kibana

![welcome_kibana](/assets/images/kibana/welcome_kibana.png)