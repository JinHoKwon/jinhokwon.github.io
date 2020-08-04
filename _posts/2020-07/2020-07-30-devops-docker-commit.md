---
title: Docker commit
tags: [devops, docker]
comments: true
categories: devops
header:
  teaser: "/assets/images/docker/docker_logo.png"
---

기본적으로 컨테이너의 변경사항은 컨테이너가 소멸될 때, 함께 소멸이 됩니다.<br/>
이 때, 소멸되지 않고, 영구적으로 기록이 필요할때는 반드시 `docker commit`명령어로 <br/>
변경사항을 이미지에 기록을 해야 합니다.<br/>
<br/>

## 1. Docker commit

다음은 클라우드 환경에서 자주 사용되는 alpine 이미지에 curl을 설치하고 커밋하는 과정에 대해서 설명하고 있습니다.

<br/>



#### 1-1. alpine pull

```sh
# docker pull alpine:latest
latest: Pulling from library/alpine
df20fa9351a1: Pull complete
Digest: sha256:185518070891758909c9f839cf4ca393ee977ac378609f700f60a771a2dfe321
Status: Downloaded newer image for alpine:latest
docker.io/library/alpine:latest
```

<br/>

#### 1-2. run alpine

```sh
# docker run -i -t -d --name alpine-container alpine:latest
95e21632dfe095081d0afe626ac9f744f5f8a0f8fab3470b1e83925539170318
```

<br/>

#### 1-3. curl 설치

`docker exec` 명령어로 컨테이너에 접속합니다.

```sh
# docker exec -i -t alpine-container /bin/sh
```

<br/>

이어서 curl을 설치합니다.<br/>

```sh
# apk --no-cache add curl
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
(1/4) Installing ca-certificates (20191127-r4)
(2/4) Installing nghttp2-libs (1.41.0-r0)
(3/4) Installing libcurl (7.69.1-r0)
(4/4) Installing curl (7.69.1-r0)
Executing busybox-1.31.1-r16.trigger
Executing ca-certificates-20191127-r4.trigger
OK: 7 MiB in 18 packages

# curl --version
curl 7.69.1 (x86_64-alpine-linux-musl) libcurl/7.69.1 OpenSSL/1.1.1g zlib/1.2.11 nghttp2/1.41.0
Release-Date: 2020-03-11
Protocols: dict file ftp ftps gopher http https imap imaps pop3 pop3s rtsp smb smbs smtp smtps telnet tftp
Features: AsynchDNS HTTP2 HTTPS-proxy IPv6 Largefile libz NTLM NTLM_WB SSL TLS-SRP UnixSockets
```

여기까지 진행하고 ctrl + P 키와 ctrl + Q 키로 컨테이너를 빠져 나옵니다.

<br/>

#### 1-4. docker commit

alpine 이미지의 변경사항을 이미지에 기록하기 위해서 `docker commit` 을 수행합니다.

```sh
# docker commit alpine-container jinhokwon/alpine:latest
sha256:40ace4803866a5a816dc0cc5088b86c3b16b2bdd9027ba23b604b636c58bcc87
```

<br/>

#### 1-5. docker push

최종적으로 alpine 를 push 합니다.

```sh
# docker push jinhokwon/alpine:latest
```



