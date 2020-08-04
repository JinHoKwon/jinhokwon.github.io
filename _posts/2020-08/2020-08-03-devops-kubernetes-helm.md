---
title: Kubernetes Helm
tags: [devops, kubernetes]
comments: true
categories: [devops, kubernetes]
header:
  teaser: "/assets/images/kubernetes/kubernetes_logo.png"
---

본 문서에서 Kubernetes의 resource 관리 도구인 Helm에 대해서 소개하고 있습니다.

## 1. Helm

Helm은 Kubernetes resource의 패키징 관리를 수월하게 해주는 tool 입니다.

Helm을 사용하게 되면, 관리되어야 할 리소스가 많거나, 

한꺼번에 모든 걸 재설치해야하는 경우 도움을 받을 수 있습니다.

<br/>

#### 1-1. Helm 구성요소
* **chart** : Helm에서 사용하는 패키지 포멧
* **repository** : chart 들이 공유되는 저장소
* **templates** : resource들의 manifest 파일들이 저장되는 디렉토리
* **values** : manifest에 전달될 Param값들이 정의됨.



이 때, templates 디렉토리안에는 Kubernetes에 의해서 관리되어야 할 resources 의 manifest 파일이 모두 저장이 되기때문에,

다음과 같이 자원을 모두 삭제하거나, 다시 모두 설치하거나 하는 등의 작업을 수월하게 진행할 수 있습니다.

```sh
# helm install <name>
# helm delete <name>
```



<br/>

## 2. Helm 설치

본 섹션에서는 Centos 7 환경에서 Helm 3(최신버전)을 설치하는 방법에 대해서 설명하고 있습니다.

<br/>

#### 2-1. Helm 설치

```sh
# curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
# chmod 700 get_helm.sh
# ./get_helm.sh
```

<br/>

#### 2-2. Helm 버전확인

```sh
# helm version
version.BuildInfo{Version:"v3.2.4", GitCommit:"0ad800ef43d3b826f31a5ad8dfbb4fe05d143688", GitTreeState:"clean", GoVersion:"go1.13.12"}
```

<br/>

#### 2-3. Helm chart repository 추가

```sh
# helm repo add stable https://kubernetes-charts.storage.googleapis.com/
"stable" has been added to your repositories
```

<br/>

#### 2-4. Helm chart list

```sh
# helm search repo stable
NAME                                    CHART VERSION   APP VERSION             DESCRIPTION
stable/acs-engine-autoscaler            2.2.2           2.1.1                   DEPRECATED Scales worker nodes within agent pools
stable/aerospike                        0.3.2           v4.5.0.5                A Helm chart for Aerospike in Kubernetes
stable/airflow                          7.3.0           1.10.10                 Airflow is a platform to programmatically autho...
stable/ambassador                       5.3.2           0.86.1                  DEPRECATED A Helm chart for Datawire Ambassador
stable/anchore-engine                   1.6.9           0.7.2                   Anchore container analysis and policy evaluatio...
stable/apm-server                       2.1.5           7.0.0                   The server receives data from the Elastic APM a..
... 이하 생략 ...
```

<br/>

#### 2-5. Helm chart repository update

```sh
# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈
```

<br/>

## 3. Helm 이해

본 섹션에서는 Helm을 이해하기 위한 토이프로젝트를 진행하고 소개합니다. 

<br/>

#### 3-1. sample-helm 프로젝트 생성

```sh
# helm create sample-helm
```

<br/>

#### 3-2. 불필요한 파일 정리

```sh
# rm -rf sample-helm/templates/*
# truncate -s 0 values.yaml
```

<br/>



#### 3-3. sample-helm/templates/deployment.yml 파일 생성

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

<br/>

#### 3-4. helm install

```sh
# cd sample-helm
# helm install sample-helm-name .
NAME: sample-helm-name
LAST DEPLOYED: Mon Aug  3 15:50:04 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

<br/>

#### 3-5. helm 배포 현황 조회

```sh
# kubectl get deploy,rs,pods,services -o wide
NAME                                READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS                 IMAGES                                      SELECTOR
deployment.apps/sample-deployment   2/2     2            2           28s   spring-boot-rest-pod-app   jinhokwon/spring-boot-rest-first:v1.00.00   app=spring-boot-rest-pod-app

NAME                                           DESIRED   CURRENT   READY   AGE   CONTAINERS                 IMAGES                                      SELECTOR
replicaset.apps/sample-deployment-5bb9868dd8   2         2         2       28s   spring-boot-rest-pod-app   jinhokwon/spring-boot-rest-first:v1.00.00   app=spring-boot-rest-pod-app,pod-template-hash=5bb9868dd8

NAME                                     READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
pod/sample-deployment-5bb9868dd8-4g8ft   1/1     Running   0          28s   10.244.2.31   k8node2.net   <none>           <none>
pod/sample-deployment-5bb9868dd8-pmp4g   1/1     Running   0          28s   10.244.1.28   k8node1.net   <none>           <none>

NAME                                     TYPE           CLUSTER-IP      EXTERNAL-IP                                    PORT(S)          AGE    SELECTOR
service/kubernetes                       ClusterIP      10.96.0.1       <none>                                         443/TCP          3d5h   <none>
service/spring-boot-rest-first-service   LoadBalancer   10.96.249.202   192.168.56.150,192.168.56.151,192.168.56.152   8080:30149/TCP   28s    app=spring-boot-rest-pod-app
```

<br/>

#### 3-6. 기능 테스트

```sh
# curl -s 192.168.56.151:8080 | jq
{
  "result": "first",
  "headers": {
    "user-agent": "curl/7.29.0",
    "host": "192.168.56.151:8080",
    "accept": "*/*"
  },
  "hostname": "sample-deployment-5bb9868dd8-4g8ft",
  "address": "10.244.2.31",
  "version": "v1.00.00"
}
```

<br/>



## 4. helm upgrade

#### 4-1. deployment의 image를 v1.00.01로 변경함.

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
        image: jinhokwon/spring-boot-rest-first:v1.00.01
  	 
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

<br/>

#### 4-2. sample-helm upgrade

```sh
# cd sample-helm
# helm upgrade sample-helm-name .
Release "sample-helm-name" has been upgraded. Happy Helming!
NAME: sample-helm-name
LAST DEPLOYED: Mon Aug  3 15:53:52 2020
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None

```

<br/>

#### 4-3. 기능 테스트

```sh
# curl -s 192.168.56.151:8080 | jq
{
  "result": "first",
  "headers": {
    "user-agent": "curl/7.29.0",
    "host": "192.168.56.151:8080",
    "accept": "*/*"
  },
  "hostname": "sample-deployment-859c7c787-jg6xp",
  "address": "10.244.1.29",
  "version": "v1.00.01"
}
```

<br/>

#### 4-4. helm 배포 이력 조회

```sh
# helm history sample-helm-name
REVISION        UPDATED                         STATUS          CHART                   APP VERSION     DESCRIPTION
1               Mon Aug  3 15:50:04 2020        superseded      sample-helm-0.1.0       1.16.0          Install complete
2               Mon Aug  3 15:53:52 2020        deployed        sample-helm-0.1.0       1.16.0          Upgrade complete
```

<br/>

#### 4-5. helm 롤백

```sh
# helm rollback sample-helm-name 1
Rollback was a success! Happy Helming!
```

<br/>

#### 4-6. helm 롤백후 기능 테스트

```sh
# curl -s 192.168.56.151:8080 | jq
{
  "result": "first",
  "headers": {
    "user-agent": "curl/7.29.0",
    "host": "192.168.56.151:8080",
    "accept": "*/*"
  },
  "hostname": "sample-deployment-5bb9868dd8-n6s87",
  "address": "10.244.2.33",
  "version": "v1.00.00"
}
```

<br/>

## 5. helm values

helm 환경에서 필요한 변수값을 따로 분리하여 관리하는 용도로 `values.yaml`파일을 활용할 수 있으며,

본 문서에서는 `values.yaml`파일을 적용하는 과정에 대해서 소개하고 있습니다.

<br/>

#### 5-1. values.yaml 

```yaml
pod:
  replicas: 3
```

<br/>

#### 5-2. templates/deployment.yml

{% raw %}

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-deployment
  labels:
    app: spring-boot-rest-pod-app
spec:
  replicas: {{ .Values.pod.replicas }}
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
        image: jinhokwon/spring-boot-rest-first:v1.00.01

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

{% endraw %}

<br/>

#### 5-3. helm template  적용 미리보기

```sh
# cd sample-helm
# helm template sample-helm-name .
---
# Source: sample-helm/templates/deployment.yml
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
---
# Source: sample-helm/templates/deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-deployment
  labels:
    app: spring-boot-rest-pod-app
spec:
  replicas: 3
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
        image: jinhokwon/spring-boot-rest-first:v1.00.01

```

또는 다음과 같이 적용전 미리보기가 가능합니다.

```sh
# helm template --set pod.replicas=3 sample-helm-name .
# helm template -f values.yaml sample-helm-name .
```

<br/>



#### 5-4. helm 수정사항 반영

```sh
# helm upgrade sample-helm-name .
Release "sample-helm-name" has been upgraded. Happy Helming!
NAME: sample-helm-name
LAST DEPLOYED: Mon Aug  3 16:03:07 2020
NAMESPACE: default
STATUS: deployed
REVISION: 4
TEST SUITE: None

# kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
sample-deployment-859c7c787-8kmmj   1/1     Running   0          17s
sample-deployment-859c7c787-8rlxj   1/1     Running   0          14s
sample-deployment-859c7c787-d295h   1/1     Running   0          19s
```

<br/>

#### 5-5. Helm list

```sh
# helm list
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
sample-helm-name        default         4               2020-08-03 16:03:07.109841778 +0900 KST deployed        sample-helm-0.1.0       1.16.0
```

<br/>

## 6. Helm delete



#### 6-1. Helm delete

```sh
# helm delete sample-helm-name
release "sample-helm-name" uninstalled
```

<br/>

#### 6-2. 상태 확인

```sh
# kubectl get deploy,rs,pods,services -o wide
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE    SELECTOR
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3d5h   <none>
```

