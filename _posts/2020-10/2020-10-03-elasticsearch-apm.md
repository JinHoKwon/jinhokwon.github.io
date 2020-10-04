---
title: Elastic APM 구축 및 Kibana APM Dashboard
comments: true
tags: [devops, elasticsearch]
categories: [devops, elasticsearch]
header:
  teaser: "/assets/images/elasticsearch.png"
---

본 문서에서는 Elastic APM 을 구축하고, Kibana APM Dashboard 환경에서 확인하는 과정에 대해서 설명하고 있습니다.

<br/>

## 1. Docker

[여기](https://jinhokwon.github.io/devops/devops-docker-install/)를 참고하여 Docker를 설치합니다.

<br/>

## 2. Elasticsearch, Kibana

[여기](https://jinhokwon.github.io/devops/elasticsearch-docker/)를 참고하여 Elasticsearch, Kibana를 설치합니다.

<br/>

## 3. APM Server 

APM Server는 Elastic APM Agent의 수집정보를 Elasticsearch로 전달하는 역활을 수행합니다.

<br/>

### 3-1. Pull APM Server docker image

```sh
# docker pull docker.elastic.co/apm/apm-server:7.9.2
```

<br/>

### 3-2. Download APM Server configuration file

```sh
# curl -L -O https://raw.githubusercontent.com/elastic/apm-server/7.9/apm-server.docker.yml
```

<br/>

### 3-3. Run APM Server

```sh
# docker run -d \
  --link elasticsearch7:elasticsearch \
  --name=apm-server \
  --user=apm-server \
  -p 8200:8200 \
  --volume="$(pwd)/apm-server.docker.yml:/usr/share/apm-server/apm-server.yml:ro" \
  docker.elastic.co/apm/apm-server:7.9.2 \
  --strict.perms=false -e \
  -E 'output.elasticsearch.hosts=["elasticsearch:9200"]'
```

<br/>

### 3-4. Health Check APM Server

```sh
# curl -s -X GET "http://localhost:8200" | jq
{
  "build_date": "2020-09-22T22:02:35Z",
  "build_sha": "34f81a2208c862777ca587867fd3aa2373c3f669",
  "version": "7.9.2"
}
```

<br/>

## 4. APM Java Agent

Elastic APM Java Agent는 Host App의 메트릭 정보를 수집하여 Elastic APM Server에 전달하는 역활을 합니다.

<br/>

### 4-1. Download java agent

```sh
# curl -O -L https://search.maven.org/remotecontent?filepath=co/elastic/apm/elastic-apm-agent/1.18.0/elastic-apm-agent-1.18.0.jar
```

<br/>

## 5. Sample Web Service

### 5-1. SpringBootWebApplication.java

```java
package com.example.app;

import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.Banner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@Slf4j
@SpringBootApplication
@RestController
public class SpringBootWebApplication {

    @GetMapping("/normal")
    public ResponseEntity normal() {
        return ResponseEntity.ok().build();
    }

    @GetMapping("/slow")
    public ResponseEntity slow() throws Exception {
        Thread.sleep(3_000);
        return ResponseEntity.ok().build();
    }

    @GetMapping("/error")
    public ResponseEntity error() {
        throw new RuntimeException("fake error");
    }

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(SpringBootWebApplication.class);
        app.setBannerMode(Banner.Mode.OFF);
        app.run(args);
    }
}

```

<br/>

## 6. Run sample web service

### 6-1. Run sample web service with elastic-apm-agent

```sh
# java -javaagent:/root/elastic-apm-agent-1.18.0.jar \
-Delastic.apm.service_name=my-cool-service \
-Delastic.apm.application_packages=com.example,org.another.example \
-Delastic.apm.server_urls=http://localhost:8200 \
-jar spring-boot-web-apm-0.0.1-SNAPSHOT.jar \
--server.port=8080

WARNING: sun.reflect.Reflection.getCallerClass is not supported. This will impact performance.
2020-10-03 21:09:27,931 [main] INFO  co.elastic.apm.agent.util.JmxUtils - Found JVM-specific OperatingSystemMXBean interface: com.sun.management.OperatingSystemMXBean
2020-10-03 21:09:28,136 [main] INFO  co.elastic.apm.agent.configuration.StartupInfo - Starting Elastic APM 1.18.0 as my-cool-service on Java 11.0.4 Runtime version: 11.0.4+11-LTS VM version: 11.0.4+11-LTS (Azul Systems, Inc.) Linux 3.10.0-957.el7.x86_64
2020-10-03 21:09:31,596 [main] INFO  co.elastic.apm.agent.impl.ElasticApmTracer - Tracer switched to RUNNING state
2020-10-03 21:09:31,720 [elastic-apm-server-healthcheck] INFO  co.elastic.apm.agent.report.ApmServerHealthChecker - Elastic APM server is available: {  "build_date": "2020-09-22T22:02:35Z",  "build_sha": "34f81a2208c862777ca587867fd3aa2373c3f669",  "version": "7.9.2"}
2020-10-03T21:09:37,947 INFO  [main] o.s.b.StartupInfoLogger: Starting SpringBootWebApplication v0.0.1-SNAPSHOT on k8master1.net with PID 8117 (/root/spring-boot-web-apm-0.0.1-SNAPSHOT.jar started by root in /root)
2020-10-03T21:09:38,000 DEBUG [main] o.s.b.StartupInfoLogger: Running with Spring Boot v2.3.0.RELEASE, Spring v5.2.6.RELEASE
2020-10-03T21:09:38,009 INFO  [main] o.s.b.SpringApplication: No active profile set, falling back to default profiles: default
2020-10-03T21:09:43,575 INFO  [main] o.s.b.w.e.t.TomcatWebServer: Tomcat initialized with port(s): 8080 (http)
2020-10-03T21:09:43,633 INFO  [main] o.a.j.l.DirectJDKLog: Initializing ProtocolHandler ["http-nio-8080"]
2020-10-03T21:09:43,635 INFO  [main] o.a.j.l.DirectJDKLog: Starting service [Tomcat]
2020-10-03T21:09:43,649 INFO  [main] o.a.j.l.DirectJDKLog: Starting Servlet engine: [Apache Tomcat/9.0.35]
2020-10-03T21:09:43,991 INFO  [main] o.a.j.l.DirectJDKLog: Initializing Spring embedded WebApplicationContext
2020-10-03T21:09:44,005 INFO  [main] o.s.b.w.s.c.ServletWebServerApplicationContext: Root WebApplicationContext: initialization completed in 5744 ms
2020-10-03T21:09:45,725 INFO  [main] o.s.s.c.ExecutorConfigurationSupport: Initializing ExecutorService 'applicationTaskExecutor'
2020-10-03T21:09:46,562 INFO  [main] o.a.j.l.DirectJDKLog: Starting ProtocolHandler ["http-nio-8080"]
2020-10-03 21:09:46,677 [main] INFO  co.elastic.apm.agent.servlet.ServletVersionInstrumentation - Servlet container info = Apache Tomcat/9.0.35
2020-10-03T21:09:46,692 INFO  [main] o.s.b.w.e.t.TomcatWebServer: Tomcat started on port(s): 8080 (http) with context path ''
2020-10-03T21:09:46,741 INFO  [main] o.s.b.StartupInfoLogger: Started SpringBootWebApplication in 11.168 seconds (JVM running for 20.461)
```

<br/>

## 7. Kibana APM Dashboard

Kibana > Left menu > Observability > APM 클릭

<br/>

### 7-1. APM Home 

![apm_home](/assets/images/kibana/apm_home.png)

<br/>

### 7-2. APM Services

![apm_service](/assets/images/kibana/apm_service.png)

<br/>

## 9. Reference