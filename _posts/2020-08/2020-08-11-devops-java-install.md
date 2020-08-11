---
title: CentOS 환경에 JDK 14 설치
tags: [devops, java]
comments: true
categories: [devops, java]
header:
  teaser: "/assets/images/java/java_logo.png"
---
본 문서에서는 Centos 7 환경에서 JDK 14 버전을 설치하는 과정에 대해서 설명하고 있습니다.
<br/>

#### JDK 14 다운로드 및 설치

적당한 위치에 JDK 14 버전을 다운로드 하고, `/usr/local/jdk14`경로에 압축을 해제합니다.

```sh
# mkdir -p /tmp/download
# cd /tmp/download
# curl -O -L https://cdn.azul.com/zulu/bin/zulu14.29.23-ca-jdk14.0.2-linux_x64.tar.gz
# tar xvfz zulu14.29.23-ca-jdk14.0.2-linux_x64.tar.gz
# mv zulu14.29.23-ca-jdk14.0.2-linux_x64 /usr/local/jdk14
# ls -al /usr/local/jdk14 
합계 32
drwxrwxr-x. 10  500  500  175  7월 11 04:20 .
drwxr-xr-x. 16 root root  190  8월 11 11:17 ..
-rw-rw-r--.  1  500  500 2765  7월 11 04:20 DISCLAIMER
-rw-rw-r--.  1  500  500 1168  7월 11 04:20 Welcome.html
drwxrwxr-x.  2  500  500 4096  7월 11 04:22 bin
drwxrwxr-x.  5  500  500  123  7월 11 04:20 conf
drwxrwxr-x.  4  500  500   48  7월 11 04:20 demo
drwxrwxr-x.  3  500  500  132  7월 11 04:20 include
drwxrwxr-x.  2  500  500 4096  7월 11 04:20 jmods
drwxrwxr-x. 74  500  500 4096  7월 11 04:20 legal
drwxrwxr-x.  5  500  500 4096  7월 11 04:22 lib
drwxrwxr-x.  3  500  500   18  7월 11 04:20 man
-rw-rw-r--.  1  500  500  782  7월 11 04:20 readme.txt
-rw-rw-r--.  1  500  500 1291  7월 11 04:20 release
```

<br/>

#### JDK 환경 변수 설정

/etc/profile 파일을 편집하여 다음과 같이 `JAVA_HOME` 경로를 설정합니다.

```sh
export JAVA_HOME=/usr/local/jdk14
export PATH=$PATH:$JAVA_HOME/bin
```

<br/>

#### Java 명령어 등록

```sh
# alternatives --install /usr/bin/java java /usr/local/jdk14/bin/java 1 &&
  alternatives --install /usr/bin/javac javac /usr/local/jdk14/bin/javac 1 &&
  alternatives --install /usr/bin/javaws javaws /usr/local/jdk14/bin/javaws 1 &&
  alternatives --set java /usr/local/jdk14/bin/java &&
  alternatives --set javac /usr/local/jdk14/bin/javac &&
  alternatives --set javaws /usr/local/jdk14/bin/javaws
```

<br/>

#### Java 버전 확인

```sh
# java -version
openjdk version "14.0.2" 2020-07-14
OpenJDK Runtime Environment Zulu14.29+23-CA (build 14.0.2+12)
OpenJDK 64-Bit Server VM Zulu14.29+23-CA (build 14.0.2+12, mixed mode, sharing)
```


