---
title: Kubernetes pod
tags: [devops, kubernetes]
comments: true
categories: [devops, kubernetes]
header:
  teaser: "/assets/images/kubernetes/kubernetes_logo.png"
---

본 문서에서는 Kubernetes의 가장 기본적인 배포 단위인 `Pod`에 대해서 설명하고 있습니다.

## 1. Pod

Kubernetes는 n개의 컨테이너로 이루어진 Pod를 배포 및 관리하며, Pod는 다음과 같은 특징을 갖고 있습니다.

* Pod에 속해있는 컨테이너가 여러개라도, Pod는 단일 노드에서만 실행됩니다. (자원 공유 목적 때문.)
* 1개의 Pod는 여러개의 컨테이너를 가질수는 있지만, 일반적으로 Pod와 컨테이너는 1:1로 매칭이 됩니다. 
* 1개의 Pod는 1개의 IP를 갖게되며, Pod안에 생성된 컨테이너와 IP를 공유하게 됩니다.
* 1개의 Pod안에서 실행되는 컨테이너들은 자원을 공유 할 수 있습니다. (예 : volume)
* Pod는 Pending, Running, Succeeded, Failed, Unknown 등과 같은 생명주기를 갖게 됩니다. 
* Pod에서 실행중인 컨테이너를 외부에서 접근하기 위해서는 Kubernetes `Service`를 사용해야 합니다.

  



![pod](/assets/images/kubernetes/pod.png)



<br/>

## 2. Pod 실습

#### 2-1. /tmp/spring-boot-rest-pod.yml 파일 생성

* **apiVersion** : Kubernetes api 버전
* **kind** : Kubernetes resource의 종류
* **metadata** : resource의 메타데이타
* **labels** : resource를 선택할 때 사용되는 검색 조건
* **spec** : resource에 대한 상세 스펙을 정의

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: spring-boot-rest-pod
  labels:
    service-name: spring-boot-rest-pod-service
spec:
  containers:
  - name: spring-boot-rest-pod-container
    image: jinhokwon/spring-boot-rest-docker
    env:
    - name: SERVER_PORT
      value: "8080"
```

<br/>

#### 2-2. Pod 생성

```sh
# kubectl create -f /tmp/spring-boot-rest-pod.yml
pod/spring-boot-rest-pod created
```

<br/>

#### 2-3. Pod 조회

```sh
# kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE    IP            NODE          NOMINATED NODE   READINESS GATES
spring-boot-rest-pod   1/1     Running   0          6m6s   10.244.2.11   k8node2.net   <none>           <none>
```

<br/>

#### 2-4. Pod 상세 조회

```sh
# kubectl describe pod spring-boot-rest-pod
Name:         spring-boot-rest-pod
Namespace:    default
Priority:     0
Node:         k8node2.net/192.168.56.152
Start Time:   Wed, 29 Jul 2020 14:05:10 +0900
Labels:       service-name=spring-boot-rest-pod-service
Annotations:  <none>
Status:       Running
IP:           10.244.2.11
IPs:
  IP:  10.244.2.11
Containers:
  spring-boot-rest-pod-container:
    Container ID:   docker://403674da15c230f61482dbee8dfc72f9082da3b520a3cb9021c232449a2aeebc
    Image:          jinhokwon/spring-boot-rest-docker
    Image ID:       docker-pullable://jinhokwon/spring-boot-rest-docker@sha256:7e17193746e06790c4b7b4c36bd9e2990fc9958be39d6de62fac405b03388328
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 29 Jul 2020 14:05:15 +0900
    Ready:          True
    Restart Count:  0
    Environment:
      SERVER_PORT:  8080
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-txjnx (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-txjnx:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-txjnx
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From                  Message
  ----    ------     ----   ----                  -------
  Normal  Scheduled  6m33s  default-scheduler     Successfully assigned default/spring-boot-rest-pod to k8node2.net
  Normal  Pulling    6m32s  kubelet, k8node2.net  Pulling image "jinhokwon/spring-boot-rest-docker"
  Normal  Pulled     6m28s  kubelet, k8node2.net  Successfully pulled image "jinhokwon/spring-boot-rest-docker"
  Normal  Created    6m28s  kubelet, k8node2.net  Created container spring-boot-rest-pod-container
  Normal  Started    6m28s  kubelet, k8node2.net  Started container spring-boot-rest-pod-container
```

<br/>

#### 2-5. Pod 로그 확인

```sh
# kubectl logs spring-boot-rest-pod
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.3.RELEASE)

2020-07-29T05:05:18,536 INFO  [main] o.s.b.StartupInfoLogger: Starting SpringBootWebApplication v0.0.1-SNAPSHOT on spring-boot-rest-pod with PID 1 (/app.jar started by root in /)
2020-07-29T05:05:18,563 DEBUG [main] o.s.b.StartupInfoLogger: Running with Spring Boot v2.1.3.RELEASE, Spring v5.1.5.RELEASE
2020-07-29T05:05:18,565 INFO  [main] o.s.b.SpringApplication: No active profile set, falling back to default profiles: default
2020-07-29T05:05:21,361 INFO  [main] o.s.b.w.e.t.TomcatWebServer: Tomcat initialized with port(s): 8080 (http)
2020-07-29T05:05:21,405 INFO  [main] o.a.j.l.DirectJDKLog: Initializing ProtocolHandler ["http-nio-8080"]
2020-07-29T05:05:21,432 INFO  [main] o.a.j.l.DirectJDKLog: Starting service [Tomcat]
2020-07-29T05:05:21,433 INFO  [main] o.a.j.l.DirectJDKLog: Starting Servlet engine: [Apache Tomcat/9.0.16]
2020-07-29T05:05:21,459 INFO  [main] o.a.j.l.DirectJDKLog: The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [/usr/lib/jvm/java-1.8-openjdk/jre/lib/amd64/server:/usr/lib/jvm/java-1.8-openjdk/jre/lib/amd64:/usr/lib/jvm/java-1.8-openjdk/jre/../lib/amd64:/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib]
2020-07-29T05:05:21,639 INFO  [main] o.a.j.l.DirectJDKLog: Initializing Spring embedded WebApplicationContext
2020-07-29T05:05:21,640 INFO  [main] o.s.b.w.s.c.ServletWebServerApplicationContext: Root WebApplicationContext: initialization completed in 2971 ms
2020-07-29T05:05:22,101 INFO  [main] o.s.s.c.ExecutorConfigurationSupport: Initializing ExecutorService 'applicationTaskExecutor'
2020-07-29T05:05:22,530 INFO  [main] o.a.j.l.DirectJDKLog: Starting ProtocolHandler ["http-nio-8080"]
2020-07-29T05:05:22,563 INFO  [main] o.s.b.w.e.t.TomcatWebServer: Tomcat started on port(s): 8080 (http) with context path ''
2020-07-29T05:05:22,571 INFO  [main] o.s.b.StartupInfoLogger: Started SpringBootWebApplication in 5.357 seconds (JVM running for 7.036)
```

<br/>

#### 2-6. Pod 테스트

Kubernetes의 Pod는 외부에서 접근할수 없기 때문에,<br/>

`spring-boot-rest-pod` 가 실행중인 k8node2.net 장비에서 테스트를 진행합니다.<br/>

만약, 외부에서 Kubernetes의 특정 Pod에 접근하려면 `Service`를 사용해야 합니다.

```sh
# curl 10.244.2.11:8080
{
  "result" : "first",
  "hostname" : "spring-boot-rest-pod"
}
```

<br/>

#### 2-7. Pod 제거

```sh
# kubectl delete -f /tmp/spring-boot-rest-pod.yml
pod "spring-boot-rest-pod" deleted
```






