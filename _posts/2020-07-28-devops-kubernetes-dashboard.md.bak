---
title: Kubernetes dashboard
tags: [devops, kubernetes]
comments: true
categories: devops
header:
  teaser: "/assets/images/kubernetes/kubernetes_logo.png"
---





## 1. kubernetes dashboard

#### 7-1. dashboard 설치

```sh
# kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta1/aio/deploy/recommended.yaml
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/kubernetes-metrics-scraper created
```

<br/>

```sh
# kubectl get pods --all-namespaces
NAMESPACE              NAME                                          READY   STATUS    RESTARTS   AGE
kube-system            calico-kube-controllers-564b6667d7-45ffl      1/1     Running   1          3h1m
kube-system            calico-node-llzlt                             1/1     Running   1          3h1m
kube-system            calico-node-tg7lg                             1/1     Running   0          25m
kube-system            calico-node-vlnr7                             1/1     Running   1          178m
kube-system            coredns-5644d7b6d9-29xw2                      1/1     Running   1          3h12m
kube-system            coredns-5644d7b6d9-9nrbg                      1/1     Running   1          3h12m
kube-system            etcd-master1.net                              1/1     Running   1          3h11m
kube-system            kube-apiserver-master1.net                    1/1     Running   1          3h11m
kube-system            kube-controller-manager-master1.net           1/1     Running   1          3h11m
kube-system            kube-proxy-2gncf                              1/1     Running   1          178m
kube-system            kube-proxy-5jnvr                              1/1     Running   0          25m
kube-system            kube-proxy-bg45j                              1/1     Running   1          3h12m
kube-system            kube-scheduler-master1.net                    1/1     Running   1          3h11m
kubernetes-dashboard   kubernetes-dashboard-bf855c94d-x2dkk          1/1     Running   0          48s
kubernetes-dashboard   kubernetes-metrics-scraper-6b97c6d857-4c5r4   1/1     Running   0          48s
```

<br/>

```sh
# kubectl proxy --port=8001 --address=192.168.56.101 --accept-hosts='^*$'
```

<br/>



#### 1-2. dashboard 계정 생성

```sh
# kubectl create serviceaccount cluster-admin-dashboard-sa
serviceaccount/cluster-admin-dashboard-sa created

# kubectl create clusterrolebinding cluster-admin-dashboard-sa --clusterrole=cluster-admin --serviceaccount=default:cluster-admin-dashboard-sa
clusterrolebinding.rbac.authorization.k8s.io/cluster-admin-dashboard-sa created
```

<br/>

#### 1-3. dashboard 계정 토큰 확인

```sh
# kubectl get secret $(kubectl get serviceaccount cluster-admin-dashboard-sa -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
eyJhbGciOiJSUzI1NiIsImtpZCI6IjZNT1E1VkxuTlN3RFFoTHdsUDZYWTVpc1FzSEZlN2MwZUdqZFpfQVdiNEkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImNsdXN0ZXItYWRtaW4tZGFzaGJvYXJkLXNhLXRva2VuLXZjc2h6Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImNsdXN0ZXItYWRtaW4tZGFzaGJvYXJkLXNhIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiNGMyODc2MzMtNjFkMy00N2RjLTk4ZTctMGM0MGQ0OGY5ZWUxIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRlZmF1bHQ6Y2x1c3Rlci1hZG1pbi1kYXNoYm9hcmQtc2EifQ.J6xmvTqNI-a6KWzjS-JetPw25eFAaTIvO0aouVppUFrk0CqyaVNRMy5untGzLwi8ujrYS__64yAIlTQGw9wiXn7_lW4v8uyUrSUarmxa_UH5h-WDu6ans3QexLIi-Bop3zrmwUYRRGjhBC-FojduyVAgcKQR1Iwcd5cdFyDGORK7TejBcdZvICDqVlp8i5F4mweGqXx0VslptBXdGP-AjaLPnVxgcWstdXOIXpXys57w92SINZH4CEErkJzqDvOCr1v8sE9yynBkOBZLo5vT1Zjtg0yJ41vbTJvOjkUHrjPGbyHEFV1T87OEo-hpHk0Djga5zgDvO8o2-HEH6LJ_qQ
```



