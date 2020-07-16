---
title: yum 그리고 yum 명령어 요약
tags: [devops, yum]
comments: true
categories: devops
header:
  teaser: "/assets/images/linux.png"
---
# 1. yum 설정

#### yum ssl 비활성화

/etc/yum.conf 파일 편집후 `sslverify=false`항목을 추가함.

```sh
[main]
cachedir=/var/cache/yum/$basearch/$releasever
keepcache=0
debuglevel=2
logfile=/var/log/yum.log
exactarch=1
obsoletes=1
gpgcheck=1
plugins=1
sslverify=false       <-- 이 부분을 추가함.
installonly_limit=5
```



# 2. yum 명령어

#### 2-1. 패키지 목록 조회

```sh
#  yum list installed 
```



#### 2-2. 특정 패키지 삭제

패키지를 삭제하기 위해서는 패키지명을 알고 있어야 하고

```sh
# yum list installed | grep node
nodejs.x86_64                         1:6.17.1-1.el7                  @epel
```



패키지명을 알고 있으면 아래와 같이 삭제할 수 있습니다. 

(-y : 삭제 확인 프롬프트 유/무 옵션)

```sh
# yum -y remove nodejs.x86_64
```







