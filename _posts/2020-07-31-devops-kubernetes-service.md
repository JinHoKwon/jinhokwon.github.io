---
title: Kubernetes Service
tags: [devops, kubernetes]
comments: true
categories: [devops, kubernetes]
header:
  teaser: "/assets/images/kubernetes/kubernetes_logo.png"
---

Pod는 배포가 이루어질때마다 생성과 소멸을 반복하고, 그 시점마다 매번 IP가 변경이 됩니다.<br/>

이렇게 동적으로 IP가 변경되는 Pod에 고정적으로 접근하기 위해서 Kubernetes의 `Service`를 사용하게 됩니다.<br/>

<br/>

본 문서는 Pod 가 외부 통신을 하기 위해서 필요한 Kubernetes `Service`에 대해서 설명하고 있습니다.<br/>

<br/>

## 1. Service Type

Kubernetes 사용 가능한 Service Type은 다음과 같습니다.

* **ClusterIP** : 서비스를 클러스터 내부 IP에 노출하며, 외부에서는 접근이 불가능. (type 생략시 기본값으로 적용됨.)
* **NodePort** : 고정 포트로 각 노드의 IP에 노출하며, 외부에서 내부의 모든 노드에 동일한 포트로 접근이 가능함.
* **LoadBalancer** :  서비스를 외부에 노출시킬 때 가장 많이 사용되는 방법이며, 로드 밸런서가 라우팅되는 NodePort와 ClusterIP 서비스가 자동 생성됨. 
* **ExternalName** : 값과 함께 CNAME 레코드를 리턴하여, 서비스를 `externalName` 필드의 콘텐츠 (예:`foo.bar.example.com`)에 매핑함.

그 밖에, 동일한 IP 주소로 여러 서비스를 노출시킬수 있는 `Ingress` 를 사용하여 서비스를 노출 시킬수도 있습니다.

<br/>

<br/>



## 2. Test Pod 

Service Type에서 공통적으로 사용되는 Pod를 다음과 같이 생성합니다.

<br/>

#### 2-1. sample-pod.yml

```yml
apiVersion: v1
kind: Pod
metadata:
  name: spring-boot-rest-pod-app
  labels:
    app: spring-boot-rest-pod-app
spec:
  containers:
  - name: spring-boot-rest-pod-container
    image: jinhokwon/spring-boot-rest-first
    env:
    - name: SERVER_PORT
      value: "8080"
```

<br/>

#### 2-2. Sample Pod 생성

```sh
# kubectl create -f sample-pod.yml
pod/spring-boot-rest-pod-app created
```

<br/>

#### 2-3. Sample Pod 조회

```sh
# kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
spring-boot-rest-pod-app   1/1     Running   0          6s    10.244.1.10   k8node1.net   <none>           <none>

# kubectl describe pod spring-boot-rest-pod-app
kubectl describe pod spring-boot-rest-pod-app
Name:         spring-boot-rest-pod-app
Namespace:    default
Priority:     0
Node:         k8node1.net/192.168.56.151
Start Time:   Fri, 31 Jul 2020 13:58:13 +0900
Labels:       app=spring-boot-rest-pod-app
Annotations:  <none>
Status:       Running
IP:           10.244.1.10
IPs:
  IP:  10.244.1.10
Containers:
  spring-boot-rest-pod-container:
    Container ID:   docker://095613dcc21eae90cea66b8316bebc242e0178eb5636250a6f356ecffe2f21b8
    Image:          jinhokwon/spring-boot-rest-first
    Image ID:       docker-pullable://jinhokwon/spring-boot-rest-first@sha256:ccd352c2cfc6d06a51b38b381d613610cbc575fcb42dd4b89320d9549af9456b
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 31 Jul 2020 13:58:18 +0900
    Ready:          True
    Restart Count:  0
    Environment:
      SERVER_PORT:  8080
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-sw8x7 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-sw8x7:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-sw8x7
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From                  Message
  ----    ------     ----  ----                  -------
  Normal  Scheduled  90s   default-scheduler     Successfully assigned default/spring-boot-rest-pod-app to k8node1.net
  Normal  Pulling    89s   kubelet, k8node1.net  Pulling image "jinhokwon/spring-boot-rest-first"
  Normal  Pulled     85s   kubelet, k8node1.net  Successfully pulled image "jinhokwon/spring-boot-rest-first"
  Normal  Created    85s   kubelet, k8node1.net  Created container spring-boot-rest-pod-container
  Normal  Started    85s   kubelet, k8node1.net  Started container spring-boot-rest-pod-container
```

<br/>

#### 2-4. Sample Pod 테스트

```sh
# kubectl exec -i -t spring-boot-rest-pod-app -- curl localhost:8080
{
  "result" : "first",
  "headers" : {
    "host" : "localhost:8080",
    "user-agent" : "curl/7.64.0",
    "accept" : "*/*"
  },
  "hostname" : "spring-boot-rest-pod-app",
  "address" : "10.244.1.10"
}
```

<br/>

<br/>

## 3. ClusterIP

서비스를 클러스터 내부 IP에 노출하며, 외부에서는 접근이 불가능 한 서비스 방식이며,

type 생략시 기본값으로 적용이 됩니다. <br/>



#### 3-1. clusterip-sample-service.yml 파일 생성

```yaml
kind: Service
apiVersion: v1
metadata:
  name: clusterip-sample-service
spec:
  type: ClusterIP
  selector:
    app: spring-boot-rest-pod-app
  ports:
    - name: http
      protocol: TCP
      port: 8080
      targetPort: 8080
```

<br/>

#### 3-2.  clusterip-sample-service 생성 및 조회

`spring-boot-rest-pod-app`이 배포된 IP가 `10.244.1.10` 임을 주의깊에 확인합니다.

```sh
# kubectl create -f clusterip-sample-service.yml 
service/clusterip-sample-service created

# kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
spring-boot-rest-pod-app   1/1     Running   0          6m43s   10.244.1.10   k8node1.net   <none>           <none>

# kubectl get services -o wide
NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE     SELECTOR
clusterip-sample-service   ClusterIP   10.102.126.204   <none>        8080/TCP   22s     app=spring-boot-rest-pod-app
kubernetes                 ClusterIP   10.96.0.1        <none>        443/TCP    3h40m   <none>
```

<br/>

#### 3-3. clusterip-sample-service 테스트

현재 노출된 IP는 `10.244.1.10`이기 때문에, Kubernetes Cluster 환경에서 Pod를 테스트 하기 위해서는 `10.244.1.10`로 테스트를 진행해야 합니다.

##### k8master1.net 장비에서 테스트

```sh
# curl 10.244.1.10:8080
{
  "result" : "first",
  "headers" : {
    "user-agent" : "curl/7.29.0",
    "host" : "10.244.1.10:8080",
    "accept" : "*/*"
  },
  "hostname" : "spring-boot-rest-pod-app",
  "address" : "10.244.1.10"
}                              
```

<br/>

##### k8node1.net 장비에서 테스트

```sh
# curl 10.244.1.10:8080
{
  "result" : "first",
  "headers" : {
    "user-agent" : "curl/7.29.0",
    "host" : "10.244.1.10:8080",
    "accept" : "*/*"
  },
  "hostname" : "spring-boot-rest-pod-app",
  "address" : "10.244.1.10"
}
```



<br/>

##### k8node2.net 장비에서 테스트

```sh
# curl 10.244.1.10:8080
{
  "result" : "first",
  "headers" : {
    "user-agent" : "curl/7.29.0",
    "host" : "10.244.1.10:8080",
    "accept" : "*/*"
  },
  "hostname" : "spring-boot-rest-pod-app",
  "address" : "10.244.1.10"
}
```

<br/>

#### 3-4. clusterip-sample-service 제거

```sh
# kubectl delete -f clusterip-sample-service.yml 
service "clusterip-sample-service" deleted

# kubectl get services -o wide
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE     SELECTOR
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3h55m   <none>
```



<br/>

## 4. NodePort

고정 포트로 각 노드의 IP에 노출하며, 외부에서 내부의 모든 노드에 동일한 포트로 접근이 가능한 특징을 갖고 있습니다.

즉, NodePort: 8080이고, Kubernetes node가 k8node1.net, k8node2.net 인 경우, 다음과 같이 서비스와 통신 할 수 있습니다.

```sh
# curl k8node1.net:8080
# curl k8node2.net:8080
```

<br/>

이 때, NodePort 사용시 주의사항은 다음과 같습니다.

* 포트당 한 서비스만 할당할 수 있다.
* 30000-32767 사이의 포트만 사용할 수 있다.
* Node나 VM의 IP 주소가 바뀌면, 이를 반영해야 한다.

<br/>

#### 4-1. nodeport-sample-service.yml 파일 생성

```yaml
kind: Service
apiVersion: v1
metadata:
  name: nodeport-sample-service
spec:
  type: NodePort
  externalIPs:
  - 192.168.56.150
  - 192.168.56.151
  - 192.168.56.152
  selector:
    app: spring-boot-rest-pod-app
  ports:
    - name: http
      protocol: TCP
      port: 8080
      targetPort: 8080
      nodePort: 30303
```



#### 4-2. nodeport-sample-service.yml 생성 및 조회

NodePort 타입은 외부에서 통신이 가능하며, 이 때 통신이 가능한 IP 정보는 `EXTERNAL-IP` 에 표시가 됩니다.

```sh
# kubectl create -f nodeport-sample-service.yml
service/nodeport-sample-service created

# kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
spring-boot-rest-pod-app   1/1     Running   0          22m   10.244.1.10   k8node1.net   <none>           <none>

# kubectl get services -o wide
NAME                      TYPE        CLUSTER-IP     EXTERNAL-IP                                    PORT(S)          AGE     SELECTOR
kubernetes                ClusterIP   10.96.0.1      <none>                                         443/TCP          3h56m   <none>
nodeport-sample-service   NodePort    10.108.39.32   192.168.56.150,192.168.56.151,192.168.56.152   8080:30303/TCP   25s     app=spring-boot-rest-pod-app
```

<br/>

#### 4-3. nodeport-sample-service 테스트

현재 제가 Kubernetes를 테스트하고 있는 VM 환경의 호스트 파일은 다음과 같습니다.

```ini
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.56.150 k8master1.net
192.168.56.151 k8node1.net
192.168.56.152 k8node2.net
```

즉, `EXTERNAL-IP`는 각각의 node명과 매핑이 되어 있기 때문에, 다음과 같이 테스트를 진행 할 수 있습니다.

```sh
# curl k8node1.net:8080
{
  "result" : "first",
  "headers" : {
    "user-agent" : "curl/7.29.0",
    "host" : "k8node1.net:8080",
    "accept" : "*/*"
  },
  "hostname" : "spring-boot-rest-pod-app",
  "address" : "10.244.1.10"
}

# curl k8node2.net:8080
{
  "result" : "first",
  "headers" : {
    "user-agent" : "curl/7.29.0",
    "host" : "k8node2.net:8080",
    "accept" : "*/*"
  },
  "hostname" : "spring-boot-rest-pod-app",
  "address" : "10.244.1.10"
}
```

<br/>

#### 4-4. nodeport-sample-service 제거

```sh
# kubectl delete -f nodeport-sample-service.yml
service "nodeport-sample-service" deleted

# kubectl get services -o wide
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE    SELECTOR
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4h2m   <none>
```

<br/>

<br/>



## 5. LoadBalancer

서비스를 외부에 노출시킬 때 가장 많이 사용되는 방법이며, <br/>

Service Type으로 LoadBalancer를 사용하는 경우,<br/>

LoadBalancer 가 라우팅되는 NodePort와 ClusterIP 서비스가 자동으로 생성이 됩니다.<br/>

<br/>

즉, 특정 Pod가 특정 Node에서만 실행중이어도, <br/>

LoadBalancer가 알아서 해당 Pod로 라우팅 해줍니다.

<br/>



#### 5-1. loadbalancer-sample-service.yml 파일 생성

```yaml
kind: Service
apiVersion: v1
metadata:
  name: loadbalancer-sample-service
spec:
  type: LoadBalancer
  externalIPs:
  - 192.168.56.150
  - 192.168.56.151
  - 192.168.56.152
  selector:
    app: spring-boot-rest-pod-app
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
```



<br/>

#### 5-2. loadbalancer-sample-service 생성 및 조회

```sh
# kubectl create -f loadbalancer-sample-service.yml
service/loadbalancer-sample-service created

# kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
spring-boot-rest-pod-app   1/1     Running   0          31m   10.244.1.10   k8node1.net   <none>           <none>

# kubectl get services -o wide
NAME                          TYPE           CLUSTER-IP      EXTERNAL-IP                                    PORT(S)          AGE    SELECTOR
kubernetes                    ClusterIP      10.96.0.1       <none>                                         443/TCP          4h5m   <none>
loadbalancer-sample-service   LoadBalancer   10.107.175.33   192.168.56.150,192.168.56.151,192.168.56.152   8080:31973/TCP   19s    app=spring-boot-rest-pod-app 
```

<br/>



#### 5-3. loadbalancer-sample-service 테스트

```sh
# curl k8node1.net:8080
{
  "result" : "first",
  "headers" : {
    "user-agent" : "curl/7.29.0",
    "host" : "k8node1.net:8080",
    "accept" : "*/*"
  },
  "hostname" : "spring-boot-rest-pod-app",
  "address" : "10.244.1.10"
}

# curl k8node2.net:8080
{
  "result" : "first",
  "headers" : {
    "user-agent" : "curl/7.29.0",
    "host" : "k8node2.net:8080",
    "accept" : "*/*"
  },
  "hostname" : "spring-boot-rest-pod-app",
  "address" : "10.244.1.10"
}
```

<br/>

#### 5-4. loadbalancer-sample-service 제거

```sh
# kubectl delete -f loadbalancer-sample-service.yml
service "loadbalancer-sample-service" deleted

# kubectl get services -o wide
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE    SELECTOR
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4h9m   <none>
```

<br/>

## 6. 마무리



지금까지 Kubernetes의 Service 객체에 대해서 설명하고, <br/>

각각의 Service Type별로 Pod와 매핑하여 테스트까지 진행해 보았습니다.<br/>

<br/>

그런데, 실제로는 Service 객체를 Pod와 매칭하여 운영하는 경우는 매우 드물며,<br/>

대개의 경우는 롤백기능과 복제(Replica)기능이 제공되는 Deployment 객체와 매핑하여 <br/>

Service 객체를 활용하는 경우가 더 많습니다. <br/>

<br/>

설명의 편의상 Pod와 Service 객체에 한정해서 설명해 드렸으며,<br/>

다음 시간에는 Kubernetes가 나오게 된 배경을 이해 할 수 있는 Deployment 객체에 대해서 설명해 보겠습니다.






