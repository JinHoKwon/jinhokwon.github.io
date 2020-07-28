---
title: Kubernetes node 삭제
tags: [devops, kubernetes]
comments: true
categories: devops
header:
  teaser: "/assets/images/kubernetes/kubernetes_logo.png"
---



#### master 장비에서 실행

```sh
# kubectl delete node node1.net
node "node1.net" deleted

# kubectl delete node node2.net
node "node2.net" deleted
```



#### 해당 노드에서 실행

```sh
# kubeadm reset
# systemctl start kubelet
```

```sh
# kubeadm reset
# systemctl stop kubelet
# systemctl stop docker
# rm -rf /var/lib/cni/
# rm -rf /var/lib/kubelet/*
# rm -rf /etc/cni/
# ifconfig cni0 down
# ifconfig flannel.1 down
# ifconfig docker0 down
# ip link delete cni0
# ip link delete flannel.1
```


