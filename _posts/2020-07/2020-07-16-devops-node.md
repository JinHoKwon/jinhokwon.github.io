---
title: Centos 7 환경에서 nodejs v10 설치
tags: [devops, node]
comments: true
categories: devops
header:
  teaser: "/assets/images/linux.png"
---
#### 1-1. nodejs 관련 yum repository 추가

```sh
# curl -sL https://rpm.nodesource.com/setup_10.x | bash -
```


<br/>
#### 1-2. nodejs 의존성 패키지 설치

```sh
# yum clean all && sudo yum makecache fast
# yum install -y gcc-c++ make
```


<br/>
#### 1-3. nodejs 설치

```sh
# yum install -y nodejs
```


<br/>
#### 1-4. nodejs 버전 확인

```sh
# node -v
v10.21.0

# npm -v
6.14.4
```


