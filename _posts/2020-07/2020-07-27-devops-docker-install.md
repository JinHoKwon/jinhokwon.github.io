---
title: Centos 7 환경에서 Docker 19.03.12, Docker compose 1.26.2 설치
tags: [devops, docker]
comments: true
categories: devops
header:
  teaser: "/assets/images/docker/docker_logo.png"
---
본 문서에서는 Centos 7 환경에서 Docker 와 Docker compose 를 설치하는 방법에 대해서 설명하고 있습니다.
<br/>
<br/>
## 1. Docker 

#### 1-1. Centos 버전 확인

```sh
# cat /etc/centos-release
CentOS Linux release 7.6.1810 (Core)
```

<br/>

#### 1-2. 기존 Docker 제거

```sh
# yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine
```

<br/>

#### 1-3. Docker Repository 추가

```sh
# yum install -y yum-utils
# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

<br/>

#### 1-4. Docker 설치

Docker의 최신버전은 https://docs.docker.com/compose/release-notes/ 에서 확인이 가능하며,<br/>

다른 버전을 설치하고 싶은 경우, `19.03.12` 버전 스트링을 적절히 변경하면 됩니다.

```sh
# yum install docker-ce-19.03.12 docker-ce-cli-19.03.12 containerd.io
```

<br/>

#### 1-5. Docker 버전 확인

```sh
# docker --version
Docker version 19.03.12, build 48a66213fe
```

<br/>

#### 1-6. Docker start / stop

```sh
# systemctl start docker
# systemctl stop docker
```

<br/>

<br/>

## 2. Docker compose

#### 2-1. Docker compose 설치

Docker compose 의 최신버전은 https://github.com/docker/compose/releases 에서 확인이 가능하며, <br/>

다른 버전을 설치하고 싶은 경우, `1.26.2` 버전 스트링을 적절히 변경하면 됩니다.

```sh
# curl -L https://github.com/docker/compose/releases/download/1.26.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
# chmod ugo+x /usr/local/bin/docker-compose
```

<br/>

#### 2-2. Docker compose 버전확인

```sh
# docker-compose -v
docker-compose version 1.26.2, build eefe0d31
```

