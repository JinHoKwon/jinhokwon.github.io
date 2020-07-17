---
title: curl 그리고 curl 명령어 요약
tags: [devops, curl]
comments: true
categories: devops
header:
  teaser: "/assets/images/linux.png"
---
## 1. curl 설정
<br/>
#### 1-1. curl ssl 비활성화

curl 을 사용하여, ssl 접속시 인증서가 올바르게 설정되어 있지 않는 경우,
`-k` 또는 `--insecure` 옵션으로 ssl verify 과정을 비활성화 할 수 있습니다.

```sh
# curl https://www.google.co.kr
curl: (60) Peer's Certificate issuer is not recognized.
More details here: http://curl.haxx.se/docs/sslcerts.html

curl performs SSL certificate verification by default, using a "bundle"
 of Certificate Authority (CA) public keys (CA certs). If the default
 bundle file isn't adequate, you can specify an alternate file
 using the --cacert option.
If this HTTPS server uses a certificate signed by a CA represented in
 the bundle, the certificate verification probably failed due to a
 problem with the certificate (it might be expired, or the name might
 not match the domain name in the URL).
If you'd like to turn off curl's verification of the certificate, use
 the -k (or --insecure) option.

```


<br/>
#### 1-2. curl ssl 인증서 설치

NAT 환경 또는 PROXY 서버 환경에서는 사내에서 사용가능한 인증서를 배포해 주는데,
해당 인증서를 /etc/pki/tls/certs/ca-bundle.crt (curl 의 CAfile 저장소) 에 추가해 주면 됩니다.

```sh
# cat company.crt >> /etc/pki/tls/certs/ca-bundle.crt
```

