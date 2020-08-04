---
title: Java PKIX path building failed 대처 
tags: [devops, java]
comments: true
categories: devops
header:
  teaser: "/assets/images/java/java_logo.png"
---
#### PKIX path building failed 개요

보통 회사 업무용 PC는 NAT 또는 PROXY 서버 뒷단에 위치하게 되는데,<br/>
이러한 환경속에서, Java를 사용하여 https 통신을 하게 될 경우, <br/>
다음과 같이 `ValidatorException` 이 발생합니다. 

```java
Exception in thread "main" javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: 
PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: 
unable to find valid certification path to requested target
```

<br/>



#### PKIX path building failed 이슈 해결 방법

만약, 사내에서 사용가능한 인증서가 있다면,<br/>
해당 인증서를 $JAVA_HOME/lib/security/cacerts (Java의 CAfile 저장소) 에 추가해 주면 됩니다.

```sh
# cd $JAVA_HOME
# bin/keytool.exe -importcert -keystore lib/security/cacerts -storepass changeit -file Company.cer
```

<br/>
사내에서 사용가능한 인증서가 없다면,
다음과 같이 코드상에서 처리할 수도 있습니다.

```java
String htmlUrl = "https://external.domain.com";

TrustManager[] trustAllCerts = new TrustManager[] { 
    new X509TrustManager() {
        public X509Certificate[] getAcceptedIssuers() {
            return null;
        }

        public void checkClientTrusted(X509Certificate[] certs, String authType) {}
        public void checkServerTrusted(X509Certificate[] certs, String authType) {}
	}
};

SSLContext sc = SSLContext.getInstance("SSL");
sc.init(null, trustAllCerts, new SecureRandom());
HttpsURLConnection.setDefaultSSLSocketFactory(sc.getSocketFactory());
```









