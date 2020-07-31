---
title: Kubernetes manifest
tags: [devops, kubernetes]
comments: true
categories: [devops, kubernetes]
header:
  teaser: "/assets/images/kubernetes/kubernetes_logo.png"
---

Kubernetes의 Manifest 파일은 Kubernetes에서 어플리케이션을 생성하고 배포하고 관리하기 위해서 <br/>

필요한 여러가지 자원에 대한 정의(또는 스펙)가 기록된 파일이며,<br/>

해당 파일을 기준으로, Pod, Deployment, Service 와 같은 자원이 Kubernetes에 의해서 생성되고 관리가 됩니다.<br/>

<br/>

`즉, Kubernetes에 의해서 관리가 필요한 리소스에 대한 주문서와 같은 파일입니다.`

<br/>

참고로, Manifest 파일 외에 직접 명령어를 입력하여 Kubernetes 오브젝트를 생성 및 관리하는 것 또한 가능합니다.

<br/>

<br/>

## 1. Manifest format

Manifest 파일은 YAML 포멧으로 작성이 가능하며,

1개의 YAML 파일안에 여러개의 리소스가 포함이 될 경우,

페이지 나눔 구분자 (`---`)를 사용하여 리소스를 나누어서 기록하면 됩니다.

<br/>

<br/>

## 2. Manifest core fields

Manifest 파일에 포함되는 핵심 필드 정보는 다음과 같습니다.

* **apiVersion** : Kubernetes API 버전
* **kind** : Kubernetes resource의 종류
* **metadata** : 오브젝트를 유일하게 구분지어 줄 데이터 (예 : name, UUID, namespace 등)
* **spec** : resource에 대한 상세 스펙을 정의, 이 때, 오브젝트에 따라서 상세 스펙 항목 또한 가변적임.



#### 2-1. core manifest example

```yaml
apiVersion: v1
kind: ...
metadata:
  name: ...
  labels:
    app: ...
spec:
  containers:
  - name: ...
```



<br/>

<br/>

## 3. Kubernetes Object Spec

Kubernetes 의 오브젝트별 상세 스펙은 다음 링크를 참고하시면 됩니다.

<br/>

#### 3-1. Pod Spec

[공식문서 보러가기](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#podspec-v1-core)

<br/>

#### 3-2. Service Spec

[공식문서 보러가기](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#service-v1-core)

<br/>

#### 3-3. Deployment Spec

[공식문서 보러가기](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#deployment-v1-apps)

<br/>