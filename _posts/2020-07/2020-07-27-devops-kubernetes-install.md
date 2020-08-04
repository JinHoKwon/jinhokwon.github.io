---
title: Centos 7 환경에서 Kubernetes 설치
tags: [devops, kubernetes]
comments: true
categories: [devops, kubernetes]
header:
  teaser: "/assets/images/kubernetes/kubernetes_logo.png"
---

Kubernetes 는 가상화 및 네트워크 기술과 밀접하게 연관되어 있기 때문에,<br/>

Kubernetes에 요구하는 사전 요구사항을 만족해야만 설치 및 구동이 가능합니다. <br/>

<br/>

그런데, 불행하게도 그 요구사항을 만족하는 과정이 꽤 여러단계로 이루어져 있고,<br/>

그 중에 하나라도 누락이 되면, 더 이상 설치가 진행되지 않거나, <br/>

알수없는 오류로 Kubernetes 클러스터가 실행되지 않을수도 있습니다.<br/>

<br/>

본 문서는 (이러한 삽질을 경험한 끝에), <br/>

비교적 간단하고 필수적인 과정만을 요약하여<br/>

`Kubernetes를 Virtualbox 환경에서 설치하는 방법`에 대해서 설명하고 있습니다.<br/>

<br/>

이러한 과정이 번거롭고 단순히 Kubernetes를 온라인 환경에서 경험해 보고자 한다면,

[katacode](https://www.katacoda.com/courses/kubernetes/playground)와 같은 사이트가 Kubernetes의 명령어를 경험하는데 도움이 될 수 있습니다.

![katacode](/assets/images/kubernetes/katacode.png)

<br/>

<br/>

##  1. Virtualbox



#### 1-1. Centos 7 이 설치된 총 3개의 VM 를 준비합니다.



<br/>

#### 1-2. VM 하드웨어 설정

* 네트워크 어뎁터1 : NAT 
* 네트워크 어뎁터2 : Host only network
* CPU : master1.net은 4개, node[1-2].net은 2개로 설정
* 메모리 : master1.net 8기가, node[1-2].net은 4기가로 설정



VM 네트워크 설정과 관련된 자세한 설명은 [여기](/devops/devops-virtualbox-network/)에서 확인 할 수 있습니다.

<br/>



## 2. Centos 



앞에서 생성한 VM 이미지위에 설치된 Centos 환경에서, <br/>

다음과 같은 설정을 공통적으로 적용합니다. <br/>

<br/>

#### 2-1. Hostname 설정

* k8master1.net : 192.168.56.150
* k8node1.net : 192.168.56.151
* k8node2.net : 192.168.56.152

<br/>


#### 2-2. /etc/hosts 설정

```ini
192.168.56.101 localhost
192.168.56.150 k8master1.net
192.168.56.151 k8node1.net
192.168.56.152 k8node2.net
```
<br/>

#### 2-3. IP 설정

##### /etc/sysconfig/network-scripts/ifcfg-enp0s8 파일 편집

```ini
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s8
UUID=3bf7895a-e5b4-4238-8ce0-27dd3da8f623
DEVICE=enp0s8
ONBOOT=yes
IPADDR=192.168.56.150
GATEWAY=192.168.56.1
```

<br/>

#### 2-4. 시스템 업데이트

```sh
# yum -y install epel-release
# yum -y update
```

<br/>

#### 2-5. swap 비활성화

```sh
# swapoff -a
# sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

<br/>

#### 2-6. /etc/sysctl.conf 설정

CNI (Container network interface) 플러그인을 사용하기 위해서, 다음과 같은 설정이 필요합니다. (필수 요구사항)

```ini
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
```

* bridge-nf-call-iptables : 컨테이너 네트워크 패킷이 호스트 iptables 설정에 따라 제어되도록 함.
* ipv4.ip_forward : ip_forward를 활성화하면 커널이 처리하는 패킷에 대해서 외부로 forwarding이 가능함.

<br/>

#### 2-7. SELinux 설정

Permissive Mode로 설정

```sh
# setenforce 0
# sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   permissive
Mode from config file:          permissive
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      31
```



<br/>



## 3. Centos firewall 



`systemctl disable firewalld`명령어로 전체 방화벽을 disable 하거나,<br/>

다음과 같이 각각의 노드별로 방화벽 설정을 진행합니다.

<br/>

#### 3-1. k8master1.net 방화벽 설정

| **Protocol** | **Direction** | **Port Range** | **Purpose**             | **Userd By**         |
| ------------ | ------------- | -------------- | ----------------------- | -------------------- |
| TCP          | Inbound       | 6443           | Kubernetes API server   | All                  |
| TCP          | Inbound       | 2379~2380      | etcd server client API  | kube-apiserver, etcd |
| TCP          | Inbound       | 10250          | Kubelet API             | Self, Control plane  |
| TCP          | Inbound       | 10251          | kube-scheduler          | Self                 |
| TCP          | Inbound       | 10252          | kube-controller-manager | Self                 |

```sh
# firewall-cmd --permanent --zone=public --add-port=6443/tcp
# firewall-cmd --permanent --zone=public --add-port=2379-2380/tcp
# firewall-cmd --permanent --zone=public --add-port=10250/tcp
# firewall-cmd --permanent --zone=public --add-port=10251/tcp
# firewall-cmd --permanent --zone=public --add-port=10252/tcp
# firewall-cmd --reload
```

<br/>

#### 3-2. k8node[1-2].net 방화벽 설정

| **Protocol** | **Direction** | **Port Range** | **Purpose**       | **Userd By**        |
| ------------ | ------------- | -------------- | ----------------- | ------------------- |
| TCP          | Inbound       | 10250          | Kubelet API       | Self, Control plane |
| TCP          | Inbound       | 30000-32767    | NodePort Services | All                 |

```sh
# firewall-cmd --permanent --zone=public --add-port=10250/tcp
# firewall-cmd --permanent --zone=public --add-port=30000-32767/tcp
# firewall-cmd --reload
```



<br/>

#### 3-3. pod network add-on 방화벽 설정

| **Configuration**       | **Host(s)** | **Connection type** | **Port/protocol** |
| ----------------------- | ----------- | ------------------- | ----------------- |
| Calico networking (BGP) | All         | Bidirectional       | TCP 179           |

```sh
# firewall-cmd --permanent --zone=public --add-port=179/tcp
# firewall-cmd --reload
```



<br/>

## 4. Centos reboot

지금까지의 설정을 모두 적용하고, 모든 Centos 를 재부팅 합니다. 

```sh
# reboot now
```

<br/>

<br/>

## 5. Docker 



#### 5-1. Docker 설치

[여기](/devops/devops-docker-install/)를 참고하여 Docker와 Docker compose를 모든 노드에 설치합니다.

<br/>

#### 5-2. Docker 드라이버

/etc/docker/daemon.json 파일을 편집하여<br/>

cgroupdriver 를 kubernetes에서 권장하는 systemd로 설정합니다.

```json
{
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-opts": {
        "max-size": "30m"
    }
}
```

<br/>

#### 5-3. Docker 실행

Docker를 모든 노드에서 실행합니다.

```sh
# systemctl start docker && systemctl enable docker
```



<br/>

<br/>



> 여기까지 진행하고, 각각의 VM 장비의 `스냅샷`을 찍어놓는게 좋습니다.
>
> 왜냐하면, Kubernetes 설치하는 과정속에서 삽질을 해야하는 경우가 발생하는데,
>
> 그 경우, 바로 지금 이 시점이 다시 시작하기에 가장 적절한 시점이기 때문입니다.

<br/>

<br/>

## 6. Kubernetes

6번째 색션에서는 진행되는 모든 과정은, <br/>

k8master1.net, k8node[1-2].net 장비에서 모두 함께 적용하시면 됩니다.

<br/>

#### 6-1. /etc/yum.repos.d/kubernetes.repo 추가

```ini
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
```

<br/>

#### 6-2. kubernetes 설치

```sh
# yum install -y kubelet kubeadm kubectl
# systemctl start kubelet && systemctl enable kubelet
# kubelet --version
Kubernetes v1.18.6
```

<br/>

<br/>

## 7. kubernetes master 

다음 작업은 `k8master1.net` 노드에서만 진행합니다.<br/>

<br/>

#### 7-1. kubectl 환경변수 설정

root 계정을 이용해서 kubectl을 실행할 경우 `/etc/profile` 파일에 `KUBECONFIG` 환경 변수를 설정해야 합니다.

```sh
export KUBECONFIG=/etc/kubernetes/admin.conf
```

<br/>

#### 7-2. kubeadm init

Kubernetes master 노드를 초기화 할 때 필요한 master IP 주소와 network add on을 다음과 같이 지정하여, <br/>

마스터 초기화를 진행합니다.

* **apiserver-advertise-address** : master1.net의 IP 주소
* **pod-network-cidr** : Flannel 에서 권장하는 네트워크 대역

```sh
# kubeadm init --apiserver-advertise-address=192.168.56.150 --pod-network-cidr=10.244.0.0/16

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.56.150:6443 --token 8ucuwc.0mvfdn8quobhyktt \
    --discovery-token-ca-cert-hash sha256:51caed24374d54d02c43c6d85dc6e94aaff9f7a3ee897cab707e5f1e5ec5a317
```

정상적으로 마스터 초기화가 완료되면, Kubernetes 실행에 필요한 `사용자 설정`과,<br/>

`kubeadm join`명령어로 각각의 노드를 마스터에 join 할 수 있는 명령어가 출력됩니다.

<br/>

#### 7-3. kube config 설정

Kubernetes 실행에 필요한 사용자 설정을 진행합니다. 

```sh
# mkdir -p $HOME/.kube
# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
# chown $(id -u):$(id -g) $HOME/.kube/config
```

<br/>

#### 7-4. Pod network add on 설치

Kubernetes용 CNI(Container network interface)는 여러가지가 있지만, <br/>

그 중에서 Github 기준 가장 많은 Star 수를 기록한 Flannel로 network add on을 설치합니다.

>Flannel에서는 기본 인터페이스로 enp0s3를 사용하는데, 해당 인터페이스는 현재 VM의 NAT와 연결되어 있기 때문에,
>
>Kubernetes 노드간의 통신에 문제가 발생하니, 다운로드 받은 kube-flannel.yml 파일의 약 184번째 라인에서 `--iface=enp0s8`를 추가합니다.
>

<br/>

##### 7-4-1. kube-flannel.yml 파일 다운로드

```sh
# cd /root/download
# curl -O -L https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

<br/>

##### 7-4-2. kube-flannel.yml 파일 수정

```yaml
  containers:
  - name: kube-flannel
    image: quay.io/coreos/flannel:v0.10.0-amd64
    command:
    - /opt/bin/flanneld
    args:
    - --ip-masq
    - --kube-subnet-mgr
    - --iface=enp0s8
```

<br/>

##### 7-4-3. Flannel 설치


```sh
# kubectl apply -f /root/download/kube-flannel.yml
```

<br/>

<br/>

## 8. Kubernetes node

Kubernetes 워커 노드가 되기 위해서는 `kubeadm join` 명령어를 실행해서 Master 노드에 등록해야 합니다. <br/>

<br/>

#### 8-1. k8node1.net join

k8node1.net 장비에서 실행함.

```sh
# kubeadm join 192.168.56.150:6443 --token 8ucuwc.0mvfdn8quobhyktt \
    --discovery-token-ca-cert-hash sha256:51caed24374d54d02c43c6d85dc6e94aaff9f7a3ee897cab707e5f1e5ec5a317
```

<br/>

#### 8-2. k8node2.net join

k8node2.net 장비에서 실행함.

```sh
# kubeadm join 192.168.56.150:6443 --token 8ucuwc.0mvfdn8quobhyktt \
    --discovery-token-ca-cert-hash sha256:51caed24374d54d02c43c6d85dc6e94aaff9f7a3ee897cab707e5f1e5ec5a317
```

<br/>



#### 8-3. kubernetes 노드 상태 조회

약 2분 정도 경과후에 확인

```sh
# kubectl get nodes
NAME            STATUS   ROLES    AGE   VERSION
k8master1.net   Ready    master   59m   v1.18.6
k8node1.net     Ready    <none>   10m   v1.18.6
k8node2.net     Ready    <none>   10m   v1.18.6

# kubectl get nodes --all-namespaces
NAME            STATUS   ROLES    AGE   VERSION
k8master1.net   Ready    master   59m   v1.18.6
k8node1.net     Ready    <none>   10m   v1.18.6
k8node2.net     Ready    <none>   10m   v1.18.6
```







