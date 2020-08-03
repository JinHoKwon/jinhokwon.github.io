---
title: Kubernetes Deployment
tags: [devops, kubernetes]
comments: true
categories: [devops, kubernetes]
header:
  teaser: "/assets/images/kubernetes/kubernetes_logo.png"
---

본 문서에서는 Kubernetes 환경에서 배포에 특화된 Deployment controller 를 소개하고 있습니다.<br/>


## Deployment

Deployment는 배포에 특화된 Controller로서, 다음과 같은 특징이 있습니다.

* 배포시에 롤링 업데이트
* Revision 기반의 롤백 
* 실행시켜야 할 Pod의 갯수를 유지 (replicas)

<br/>

## 2. Deployment 테스트

#### 2-1. sample-deployment.yml 파일생성

Deployment를 테스트 하기 위해서 다음과 같이 sample-deployment 문서를 작성합니다.


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-deployment
  labels:
  	app: spring-boot-rest-pod-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: spring-boot-rest-pod-app
  template:
    metadata:
      labels:
        app: spring-boot-rest-pod-app
    spec:
      containers:
      - name: spring-boot-rest-pod-app
        image: jinhokwon/spring-boot-rest-first:v1.00.00
  	 
---
kind: Service
apiVersion: v1
metadata:
  name: spring-boot-rest-first-service
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

* **spec.template** : Pod 에 대한 스펙 정의
* **spec.replicas** : Pod의 replica를 정의 

<br/>

#### 2-2. sample-deployment 생성 및 조회

```sh
# kubectl create -f sample-deployment.yml
deployment.apps/sample-deployment created
service/spring-boot-rest-first-service created

# kubectl get deployment sample-deployment
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
sample-deployment   2/2     2            2           8s

# kubectl get deploy,rs,pods
NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/sample-deployment   2/2     2            2           16s

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/sample-deployment-5bb9868dd8   2         2         2       16s

NAME                                     READY   STATUS    RESTARTS   AGE
pod/sample-deployment-5bb9868dd8-h5hqw   1/1     Running   0          16s
pod/sample-deployment-5bb9868dd8-wtnsc   1/1     Running   0          16s
```

<br/>

#### 2-3. sample-deployment의 image 조회

```sh
# kubectl get deploy -o jsonpath='{.items[0].spec.template.spec.containers[0].image}{"\n"}'
jinhokwon/spring-boot-rest-first:v1.00.00
```

<br/>

#### 2-4. sample-deployment 설명 조회

```sh
# kubectl describe deployment sample-deployment
Name:                   sample-deployment
Namespace:              default
CreationTimestamp:      Mon, 03 Aug 2020 09:29:33 +0900
Labels:                 app=spring-boot-rest-pod-app
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=spring-boot-rest-pod-app
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=spring-boot-rest-pod-app
  Containers:
   spring-boot-rest-pod-app:
    Image:        jinhokwon/spring-boot-rest-first:v1.00.00
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   sample-deployment-5bb9868dd8 (2/2 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  2m1s  deployment-controller  Scaled up replica set sample-deployment-5bb9868dd8 to 2

```

<br/>

#### 2-5. sample-deployment로 배포된 Pod 테스트

```sh
# curl -s 192.168.56.150:8080 | jq
{
  "result": "first",
  "headers": {
    "user-agent": "curl/7.29.0",
    "host": "192.168.56.150:8080",
    "accept": "*/*"
  },
  "hostname": "sample-deployment-5bb9868dd8-wtnsc",
  "address": "10.244.2.16",
  "version": "v1.00.00"
}

# curl -s 192.168.56.151:8080 | jq
{
  "result": "first",
  "headers": {
    "user-agent": "curl/7.29.0",
    "host": "192.168.56.151:8080",
    "accept": "*/*"
  },
  "hostname": "sample-deployment-5bb9868dd8-h5hqw",
  "address": "10.244.1.13",
  "version": "v1.00.00"
}

# curl -s 192.168.56.152:8080 | jq
{
  "result": "first",
  "headers": {
    "user-agent": "curl/7.29.0",
    "host": "192.168.56.152:8080",
    "accept": "*/*"
  },
  "hostname": "sample-deployment-5bb9868dd8-wtnsc",
  "address": "10.244.2.16",
  "version": "v1.00.00"
}

```

<br/>



## 3. Deployment scale



Deployment의 scale 변경은 `kubectl scale deploy` 명령어 또는 <br/>

deployment.yml 파일을 수정한 후 `kubectl apply -f ` 명령어로 적용이 가능합니다.

<br/>



#### 3-1. sample-deployment scale 변경

```sh
# kubectl scale deploy sample-deployment --replicas=5
deployment.apps/sample-deployment scaled
```

<br/>

#### 3-2. deployment 설명 조회

방금전 scale을 변경하였기 때문에, Scaled up replica set 이벤트에 대한 메세지가 표시됨.

```sh
#  kubectl describe deployment sample-deployment
Name:                   sample-deployment
Namespace:              default
CreationTimestamp:      Mon, 03 Aug 2020 09:29:33 +0900
Labels:                 app=spring-boot-rest-pod-app
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=spring-boot-rest-pod-app
Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=spring-boot-rest-pod-app
  Containers:
   spring-boot-rest-pod-app:
    Image:        jinhokwon/spring-boot-rest-first:v1.00.00
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   sample-deployment-5bb9868dd8 (5/5 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  2m58s  deployment-controller  Scaled up replica set sample-deployment-5bb9868dd8 to 2
  Normal  ScalingReplicaSet  18s    deployment-controller  Scaled up replica set sample-deployment-5bb9868dd8 to 5
```

<br/>

<br/>



## 4. Deployment 배포

Deployment의 배포는 `kubectl set image` 명령어 또는 <br/>

deployment.yml 파일을 수정한 후 `kubectl apply -f ` 명령어로 적용이 가능합니다.

<br/>

#### 4-1. sample-deployment 배포

다음 예제는 spring-boot-rest-pod-app의 컨테이너 이미지를 `jinhokwon/spring-boot-rest-first:v1.00.01`로 배포합니다.

```sh
# kubectl set image deployment/sample-deployment spring-boot-rest-pod-app=jinhokwon/spring-boot-rest-first:v1.00.01 --record=true
deployment.apps/sample-deployment image updated
```
* **--record** : 리소스 변경사항을 저장

또는 sample-deployment.yml 파일을 수정한 이후에, `kubectl apply` 명령어로 업데이트가 가능합니다.

```sh
# kubectl apply -f sample-deployment.yml
```

<br/>

#### 4-2. deployment update 확인

```sh
# kubectl get deploy -o jsonpath='{.items[0].spec.template.spec.containers[0].image}{"\n"}'
jinhokwon/spring-boot-rest-first:v1.00.01
```

<br/>

#### 4-3. deployment history 조회

```sh
# kubectl rollout history deploy sample-deployment
deployment.apps/sample-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment/sample-deployment spring-boot-rest-pod-app=jinhokwon/spring-boot-rest-first:v1.00.01 --record=true
```

<br/>

#### 4-4. deployment history 중 특정 revision 조회

```sh
# kubectl rollout history deploy sample-deployment --revision=2
deployment.apps/sample-deployment with revision #2
Pod Template:
  Labels:       app=spring-boot-rest-pod-app
        pod-template-hash=859c7c787
  Annotations:  kubernetes.io/change-cause:
          kubectl set image deployment/sample-deployment spring-boot-rest-pod-app=jinhokwon/spring-boot-rest-first:v1.00.01 --record=true
  Containers:
   spring-boot-rest-pod-app:
    Image:      jinhokwon/spring-boot-rest-first:v1.00.01
    Port:       <none>
    Host Port:  <none>
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
```



<br/>

#### 4-5. spring-boot-rest-first:v1.00.01 deployment 확인

```sh
# curl -s 192.168.56.150:8080 | jq
{
  "result": "first",
  "headers": {
    "user-agent": "curl/7.29.0",
    "host": "192.168.56.150:8080",
    "accept": "*/*"
  },
  "hostname": "sample-deployment-859c7c787-4bpgb",
  "address": "10.244.2.20",
  "version": "v1.00.01"
}

# curl -s 192.168.56.151:8080 | jq
{
  "result": "first",
  "headers": {
    "user-agent": "curl/7.29.0",
    "host": "192.168.56.151:8080",
    "accept": "*/*"
  },
  "hostname": "sample-deployment-859c7c787-gz45q",
  "address": "10.244.2.19",
  "version": "v1.00.01"
}

# curl -s 192.168.56.152:8080 | jq
{
  "result": "first",
  "headers": {
    "user-agent": "curl/7.29.0",
    "host": "192.168.56.152:8080",
    "accept": "*/*"
  },
  "hostname": "sample-deployment-859c7c787-rvxcw",
  "address": "10.244.1.17",
  "version": "v1.00.01"
}
```





<br/>

## 5. Deployment rollback

Deployment의 rollback은 `kubectl rollout undo` 명령어로 가능하며, 

상세 설명은 다음과 같습니다.

<br/>

#### 5-1. deployment rollback

```sh
# kubectl rollout undo deploy sample-deployment --record
deployment.apps/sample-deployment rolled back
```

<br/>

#### 5-2. 특정 revision으로 rollback

```sh
# kubectl rollout undo deploy sample-deployment --to-revision=1
deployment.apps/sample-deployment rolled back
```

rollback 진행후, 기능 테스트를 진행합니다.

```sh
# curl 192.168.56.151:8080
{
  "result": "first",
  "headers": {
    "user-agent": "curl/7.29.0",
    "host": "192.168.56.151:8080",
    "accept": "*/*"
  },
  "hostname": "sample-deployment-5bb9868dd8-kxbhs",
  "address": "10.244.1.25",
  "version": "v1.00.00"
}
```

정상적으로 `v1.00.00`으로 롤백되었음을 확인합니다.

<br/>

#### 5-3. deployment status 조회

```sh
# kubectl rollout status deploy sample-deployment 
deployment "sample-deployment" successfully rolled out
```

<br/>

## 6. Deployment 제거

#### 6-1. deployment 제거

```sh
# kubectl delete -f sample-deployment.yml
deployment.apps "sample-deployment" deleted
service "spring-boot-rest-first-service" deleted
```







































