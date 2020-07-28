---
title: Kubernetes Hello world
tags: [devops, kubernetes]
comments: true
categories: devops
header:
  teaser: "/assets/images/kubernetes/kubernetes_logo.png"
---

<br/>

#### docker 로그인

```sh
# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: jinhokwon
Password:
Login Succeeded
```



<br/>

#### 샘플 이미지 pull

master1.net 장비에서 실행

```sh
# docker pull jinhokwon/spring-boot-rest-docker:latest
Using default tag: latest
Trying to pull repository docker.io/asbubam/hello-node ...
latest: Pulling from docker.io/asbubam/hello-node
10a267c67f42: Pull complete
fb5937da9414: Pull complete
9021b2326a1e: Pull complete
dbed9b09434e: Pull complete
74bb2fc384c6: Pull complete
9b0a326fab3b: Pull complete
8089dfd0519a: Pull complete
f2be1898eb92: Pull complete
88c75a4701d0: Pull complete
f0e706fb13e8: Pull complete
a36c184b15a1: Pull complete
98fd9c1d5bd4: Pull complete
Digest: sha256:3d7137c208e1f3cccbd64a206d178751d0a1df07f6e5faf6fa9419454fa349a8
Status: Downloaded newer image for docker.io/jinhokwon/spring-boot-rest-docker:latest
```

<br/>

```sh
# docker run -d --name spring-boot-rest-docker \
-e "SERVER_PORT=8080" \
-p 8080:8080 \
jinhokwon/spring-boot-rest-docker:latest

68f6fe53e7abbb35ed59c316ebb17f11712cb1de30ecfb067dcd0ada10426407
```

<br/>

```sh
# docker ps -a | grep spring
283ad11985f3        jinhokwon/spring-boot-rest-docker:latest   "java -Djava.securit…"   10 seconds ago      Up 9 seconds                0.0.0.0:8080->8080/tcp   spring-boot-rest-docker
```

<br/>



```sh
# docker logs spring-boot-rest-docker

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.3.RELEASE)

2019-11-03T23:32:11,865 INFO  [main] o.s.b.StartupInfoLogger: Starting SpringBootWebApplication v0.0.1-SNAPSHOT on 283ad11985f3 with PID 1 (/app.jar started by root in /)
2019-11-03T23:32:11,888 DEBUG [main] o.s.b.StartupInfoLogger: Running with Spring Boot v2.1.3.RELEASE, Spring v5.1.5.RELEASE
2019-11-03T23:32:11,901 INFO  [main] o.s.b.SpringApplication: No active profile set, falling back to default profiles: default
2019-11-03T23:32:13,489 INFO  [main] o.s.b.w.e.t.TomcatWebServer: Tomcat initialized with port(s): 8080 (http)
2019-11-03T23:32:13,510 INFO  [main] o.a.j.l.DirectJDKLog: Initializing ProtocolHandler ["http-nio-8080"]
2019-11-03T23:32:13,524 INFO  [main] o.a.j.l.DirectJDKLog: Starting service [Tomcat]
2019-11-03T23:32:13,525 INFO  [main] o.a.j.l.DirectJDKLog: Starting Servlet engine: [Apache Tomcat/9.0.16]
2019-11-03T23:32:13,538 INFO  [main] o.a.j.l.DirectJDKLog: The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [/usr/lib/jvm/java-1.8-openjdk/jre/lib/amd64/server:/usr/lib/jvm/java-1.8-openjdk/jre/lib/amd64:/usr/lib/jvm/java-1.8-openjdk/jre/../lib/amd64:/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib]
2019-11-03T23:32:13,665 INFO  [main] o.a.j.l.DirectJDKLog: Initializing Spring embedded WebApplicationContext
2019-11-03T23:32:13,666 INFO  [main] o.s.b.w.s.c.ServletWebServerApplicationContext: Root WebApplicationContext: initialization completed in 1687 ms
2019-11-03T23:32:14,060 INFO  [main] o.s.s.c.ExecutorConfigurationSupport: Initializing ExecutorService 'applicationTaskExecutor'
2019-11-03T23:32:14,334 INFO  [main] o.a.j.l.DirectJDKLog: Starting ProtocolHandler ["http-nio-8080"]
2019-11-03T23:32:14,373 INFO  [main] o.s.b.w.e.t.TomcatWebServer: Tomcat started on port(s): 8080 (http) with context path ''
2019-11-03T23:32:14,376 INFO  [main] o.s.b.StartupInfoLogger: Started SpringBootWebApplication in 3.482 seconds (JVM running for 5.27)
```

<br/>

```sh
# curl localhost:8080
{
  "result" : "first",
  "hostname" : "283ad11985f3"
}
```

<br/>

```sh
# docker stop spring-boot-rest-docker
spring-boot-rest-docker
```

<br/>

#### kube yaml 파일 작성

마스터 노드에서 작성함.

/tmp/spring-boot-rest-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: spring-boot-rest-pod
  labels:
    service-name: spring-boot-rest-pod-service
spec:
  containers:
  - name: spring-boot-rest-container
    #image: asbubam/hello-node
    image: jinhokwon/spring-boot-rest-docker
    env:
    - name: SERVER_PORT
      value: "8080"
    readinessProbe:
      httpGet:
        path: /
        port: 8080
    livenessProbe:
      httpGet:
        path: /
        port: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-rest-service
spec:
  type: LoadBalancer
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    service-name: spring-boot-rest-service
```

<br/>

```sh
# kubectl create -f /tmp/spring-boot-rest-pod.yaml
pod/spring-boot-rest-pod created
service/spring-boot-rest-service created
```



<br/>

```sh
# kubectl get pods
NAME                   READY   STATUS              RESTARTS   AGE
spring-boot-rest-pod   0/1     ContainerCreating   0          15s
```

<br/>

잠시후..

```sh
# kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
spring-boot-rest-pod   1/1     Running   0          34s
```

<br/>





```sh
# kubectl describe pod spring-boot-rest-pod
Namespace:    default
Priority:     0
Node:         node2.net/192.168.56.152
Start Time:   Mon, 04 Nov 2019 08:35:41 +0900
Labels:       service-name=spring-boot-rest-pod-service
Annotations:  cni.projectcalico.org/podIP: 172.16.0.197/32
Status:       Running
IP:           172.16.0.197
IPs:
  IP:  172.16.0.197
Containers:
  spring-boot-rest-container:
    Container ID:   docker://ba6413af9b62033d981511f5387df08b5ceb953e029bdb96d0640b521427a5ab
    Image:          jinhokwon/spring-boot-rest-docker
    Image ID:       docker-pullable://jinhokwon/spring-boot-rest-docker@sha256:7e17193746e06790c4b7b4c36bd9e2990fc9958be39d6de62fac405b03388328
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 04 Nov 2019 08:35:45 +0900
    Ready:          True
    Restart Count:  0
    Liveness:       http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3
    Readiness:      http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-g9z5b (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-g9z5b:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-g9z5b
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From                Message
  ----    ------     ----       ----                -------
  Normal  Scheduled  <unknown>  default-scheduler   Successfully assigned default/spring-boot-rest-pod to node2.net
  Normal  Pulling    17s        kubelet, node2.net  Pulling image "jinhokwon/spring-boot-rest-docker"
  Normal  Pulled     14s        kubelet, node2.net  Successfully pulled image "jinhokwon/spring-boot-rest-docker"
  Normal  Created    14s        kubelet, node2.net  Created container spring-boot-rest-container
  Normal  Started    14s        kubelet, node2.net  Started container spring-boot-rest-container
```

<br/>

```sh
# kubectl logs spring-boot-rest-pod

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.3.RELEASE)

2019-11-03T23:35:48,281 INFO  [main] o.s.b.StartupInfoLogger: Starting SpringBootWebApplication v0.0.1-SNAPSHOT on spring-boot-rest-pod with PID 1 (/app.jar started by root in /)
2019-11-03T23:35:48,288 DEBUG [main] o.s.b.StartupInfoLogger: Running with Spring Boot v2.1.3.RELEASE, Spring v5.1.5.RELEASE
2019-11-03T23:35:48,289 INFO  [main] o.s.b.SpringApplication: No active profile set, falling back to default profiles: default
2019-11-03T23:35:49,683 INFO  [main] o.s.b.w.e.t.TomcatWebServer: Tomcat initialized with port(s): 8080 (http)
2019-11-03T23:35:49,702 INFO  [main] o.a.j.l.DirectJDKLog: Initializing ProtocolHandler ["http-nio-8080"]
2019-11-03T23:35:49,720 INFO  [main] o.a.j.l.DirectJDKLog: Starting service [Tomcat]
2019-11-03T23:35:49,721 INFO  [main] o.a.j.l.DirectJDKLog: Starting Servlet engine: [Apache Tomcat/9.0.16]
2019-11-03T23:35:49,732 INFO  [main] o.a.j.l.DirectJDKLog: The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [/usr/lib/jvm/java-1.8-openjdk/jre/lib/amd64/server:/usr/lib/jvm/java-1.8-openjdk/jre/lib/amd64:/usr/lib/jvm/java-1.8-openjdk/jre/../lib/amd64:/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib]
2019-11-03T23:35:49,839 INFO  [main] o.a.j.l.DirectJDKLog: Initializing Spring embedded WebApplicationContext
2019-11-03T23:35:49,839 INFO  [main] o.s.b.w.s.c.ServletWebServerApplicationContext: Root WebApplicationContext: initialization completed in 1458 ms
2019-11-03T23:35:50,095 INFO  [main] o.s.s.c.ExecutorConfigurationSupport: Initializing ExecutorService 'applicationTaskExecutor'
2019-11-03T23:35:50,348 INFO  [main] o.a.j.l.DirectJDKLog: Starting ProtocolHandler ["http-nio-8080"]
2019-11-03T23:35:50,370 INFO  [main] o.s.b.w.e.t.TomcatWebServer: Tomcat started on port(s): 8080 (http) with context path ''
2019-11-03T23:35:50,376 INFO  [main] o.s.b.StartupInfoLogger: Started SpringBootWebApplication in 3.056 seconds (JVM running for 4.896)
2019-11-03T23:35:52,954 INFO  [http-nio-8080-exec-1] o.a.j.l.DirectJDKLog: Initializing Spring DispatcherServlet 'dispatcherServlet'
2019-11-03T23:35:52,954 INFO  [http-nio-8080-exec-1] o.s.w.s.FrameworkServlet: Initializing Servlet 'dispatcherServlet'
2019-11-03T23:35:52,962 INFO  [http-nio-8080-exec-1] o.s.w.s.FrameworkServlet: Completed initialization in 8 ms
```

<br/>

##### pod 테스트

```sh
# curl 172.16.229.131:8080
{
  "result" : "first",
  "hostname" : "spring-boot-rest-pod"
}
```



<br/>

##### pod 제거

```sh
# kubectl delete -f /tmp/spring-boot-rest-pod.yaml
pod "spring-boot-rest-pod" deleted
service "spring-boot-rest-service" deleted
```






