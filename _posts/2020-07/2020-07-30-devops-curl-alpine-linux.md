---
title: alpine 리눅스에 curl 설치
tags: [devops, curl]
comments: true
categories: devops
header:
  teaser: "/assets/images/linux.png"
---

cloud 환경을 고려한 alpine 리눅스 환경에서<br/>

`curl`을 설치하기 위해서는 다음과 같이 진행하면 됩니다.


```sh
# cat /etc/alpine-release
3.9.4

# apk --no-cache add curl
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/community/x86_64/APKINDEX.tar.gz
(1/4) Installing nghttp2-libs (1.35.1-r2)
(2/4) Installing libssh2 (1.9.0-r1)
(3/4) Installing libcurl (7.64.0-r3)
(4/4) Installing curl (7.64.0-r3)
Executing busybox-1.29.3-r10.trigger
OK: 104 MiB in 58 packages

# curl --version
curl 7.64.0 (x86_64-alpine-linux-musl) libcurl/7.64.0 OpenSSL/1.1.1b zlib/1.2.11 libssh2/1.9.0 nghttp2/1.35.1
Release-Date: 2019-02-06
Protocols: dict file ftp ftps gopher http https imap imaps pop3 pop3s rtsp scp sftp smb smbs smtp smtps telnet tftp
Features: AsynchDNS IPv6 Largefile NTLM NTLM_WB SSL libz TLS-SRP HTTP2 UnixSockets HTTPS-proxy
```



