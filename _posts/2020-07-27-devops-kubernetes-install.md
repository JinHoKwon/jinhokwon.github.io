---
title: Centos 7 환경에서 Kubernetes 설치
tags: [devops, kubernetes]
comments: true
categories: devops
header:
  teaser: "/assets/images/kubernetes/kubernetes_logo.png"
---

##  1. Virtualbox



#### 1-1. Centos 7 이 설치된 총 3개의 VM 를 준비합니다.



<br/>

#### 1-2. VM 하드웨어 설정

* 네트워크 어뎁터1 : NAT 
* 네트워크 어뎁터2 : Host only network
* CPU : 2개 이상으로 설정함.



VM 네트워크 설정과 관련된 자세한 설명은 [여기](/devops/devops-virtualbox-network/)에서 확인 할 수 있습니다.

<br/>

#### 1-3. Hostname 설정

* master1.net : 192.168.56.150
* node1.net : 192.168.56.151
* node2.net : 192.168.56.152

<br/>


#### 1-4. /etc/hosts 설정

```ini
192.168.56.101 localhost
192.168.56.150 master1.net
192.168.56.151 node1.net
192.168.56.152 node2.net
```
<br/>

#### 1-5. IP 설정

##### /etc/sysconfig/network-scripts/ifcfg-enp0s3 파일 편집

```ini
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
IPADDR=192.168.56.101
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s3
UUID=b601b047-a545-47ee-b63f-2b828526fad0
DEVICE=enp0s3
ONBOOT=yes
GATEWAY=10.20.80.1
```

<br/>

#### 1-6. 모든 VM에 공통 적용

##### 시스템 업데이트

```sh
# yum -y install epel-release
# yum -y update
```



##### swap 메모리 비활성화

```sh
# swapoff -a
# sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```



##### /etc/sysctl.conf 설정

```ini
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
```

* bridge-nf-call-iptables : 컨테이너 네트워크 패킷이 호스트 iptables 설정에 따라 제어되도록 함.
* ipv4.ip_forward : ip_forward를 활성화하면 커널이 처리하는 패킷에 대해서 외부로 forwarding이 가능함.

<br/>

#### 1-7. SELinux 설정

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

#### 1-8. 방화벽 설정



`systemctl disable firewalld`명령어로 전체 방화벽을 disable 하거나,<br/>

다음과 같이 각각의 노드별로 방화벽 설정을 진행합니다.



##### master1.net 방화벽 설정

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

##### node[1-2].net 방화벽 설정

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

##### pod network add-on 방화벽 설정

| **Configuration**       | **Host(s)** | **Connection type** | **Port/protocol** |
| ----------------------- | ----------- | ------------------- | ----------------- |
| Calico networking (BGP) | All         | Bidirectional       | TCP 179           |

```sh
# firewall-cmd --permanent --zone=public --add-port=179/tcp
# firewall-cmd --reload
```



<br/>

#### 1-9. 재부팅

```sh
# reboot now
```

<br/>



## 2. Docker 



[여기](/devops/devops-docker-install/)를 참고하여 Docker와 Docker compose를 설치합니다.



<br/>

## 3. kubernetes 설치

<br/>

#### 3-1. /etc/yum.repos.d/kubernetes.repo 추가

모든 노드에 공통 적용

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

#### 3-2. kubernetes 설치

모든 노드에 공통 적용

```sh
# yum install -y kubelet kubeadm kubectl
```

<br/>

#### 3-3. kubernetes 서비스 등록

모든 노드에 공통 적용

```sh
# systemctl start kubelet && systemctl enable kubelet
```

<br/>

#### 3-4. kubernetes 버전 확인

```sh
# kubelet --version
Kubernetes v1.15.3
```



<br/>

<br/>

## 4. kubernetes master 초기화



#### 4-1. kubectl 환경변수 설정

root 계정을 이용해서 kubectl을 실행할 경우 다음 환경 변수를 설정한다.

/etc/profile 파일편집

```sh
export KUBECONFIG=/etc/kubernetes/admin.conf
```

<br/>

#### 4-2. proxy 서버 설정

필요시 proxy 서버를 설정함.

```sh
# export http_proxy=192.168.56.1:3128
# export https_proxy=192.168.56.1:3128
```

<br/>

#### 4-3. docker images 조회

```sh
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
```

<br/>



#### 4-4. master 초기화

master 노드에만 적용함.

초기화 작업은 'https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/' 페이지를 참고했습니다. 

pod network add-on의 종류에 따라서 --pod-network-cidr 설정 값을 다르게 해주어야 합니다. 

여기서는 pod network add-on로 Calico를 사용해서 

--pod-network-cidr 설정 값을 192.168.0.0/16으로 해야 하지만 

가상머신 네트워크 대역인 192.168.37.0/24와 겹치기 때문에 

172.16.0.0/16으로 변경하여 사용합니다.

<br/>

아직 실행전임.

```sh
# systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; disabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: inactive (dead)
     Docs: https://kubernetes.io/docs/
```

<br/>



```sh
# kubeadm init 또는 
# kubeadm init --kubernetes-version=v1.16.0 \
--apiserver-advertise-address=192.168.56.101 \
--pod-network-cidr=172.16.0.0/16

[init] Using Kubernetes version: v1.16.0
[preflight] Running pre-flight checks
        [WARNING HTTPProxy]: Connection to "https://192.168.56.101" uses proxy "http://192.168.56.1:3                                               128". If that is not intended, adjust your proxy settings
        [WARNING HTTPProxyCIDR]: connection to "10.96.0.0/12" uses proxy "http://192.168.56.1:3128".                                                This may lead to malfunctional cluster setup. Make sure that Pod and Services IP ranges specified cor                                               rectly as exceptions in proxy configuration
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommen                                               ded driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 1                                               9.03.2. Latest validated version: 18.09
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [default.net kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.56.101]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [default.net localhost] and IPs [192.168.56.101 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [default.net localhost] and IPs [192.168.56.101 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 24.018009 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.16" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node default.net as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node default.net as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: nwqpcm.98xtepp8s4hhendz
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.56.101:6443 --token 52fbkz.qe319zph7ae2vc2o \
    --discovery-token-ca-cert-hash sha256:9854795f543183babab7284413187e7a380d93eeca6fe5c07cebcca87233502c
```

<br/>



```sh
# systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since 토 2019-09-21 14:42:40 KST; 9min ago
     Docs: https://kubernetes.io/docs/
 Main PID: 3419 (kubelet)
   CGroup: /system.slice/kubelet.service
           └─3419 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=systemd...

 9월 21 14:52:21 localhost.localdomain kubelet[3419]: W0921 14:52:21.096430    3419 cni.go:237] Unable to update cni config: no networks found in /etc/cni/net.d
 9월 21 14:52:21 localhost.localdomain kubelet[3419]: E0921 14:52:21.549911    3419 kubelet.go:2187] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady m...ninitialized
 9월 21 14:52:22 localhost.localdomain kubelet[3419]: E0921 14:52:22.017757    3419 summary_sys_containers.go:47] Failed to get system container stats for "/system.slice/kubelet.service": failed to g...
 9월 21 14:52:26 localhost.localdomain kubelet[3419]: W0921 14:52:26.096697    3419 cni.go:237] Unable to update cni config: no networks found in /etc/cni/net.d
 9월 21 14:52:26 localhost.localdomain kubelet[3419]: E0921 14:52:26.551522    3419 kubelet.go:2187] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady m...ninitialized
 9월 21 14:52:31 localhost.localdomain kubelet[3419]: W0921 14:52:31.097252    3419 cni.go:237] Unable to update cni config: no networks found in /etc/cni/net.d
 9월 21 14:52:31 localhost.localdomain kubelet[3419]: E0921 14:52:31.553325    3419 kubelet.go:2187] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady m...ninitialized
 9월 21 14:52:32 localhost.localdomain kubelet[3419]: E0921 14:52:32.030261    3419 summary_sys_containers.go:47] Failed to get system container stats for "/system.slice/kubelet.service": failed to g...
 9월 21 14:52:36 localhost.localdomain kubelet[3419]: W0921 14:52:36.097803    3419 cni.go:237] Unable to update cni config: no networks found in /etc/cni/net.d
 9월 21 14:52:36 localhost.localdomain kubelet[3419]: E0921 14:52:36.556576    3419 kubelet.go:2187] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady m...ninitialized
Hint: Some lines were ellipsized, use -l to show in full.
```

<br/>



#### 4-5. kube config 설정

master 노드에서만 진행

```sh
# mkdir -p $HOME/.kube
# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
# sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

<br/>





#### 4-6. pod network add on 설치

pod network add-on 설치는 Master 노드에서만 합니다.<br/>

Kubernetes 환경에서 Pod들이 통신하려면 pod network add-on이 있어야 합니다. 

pod network add-on이 설치되기 전에는 CoreDNS가 시작되지 않습니다. 

<br/>

'kubeadm init' 이후에 'kubectl get pods -n kube-system' 명령어로 확인해보면 

CoreDNS가 아직 시작되지 않은 상태를 확인할 수 있습니다. 

<br/>

pod network와 호스트 네트워크가 겹칠 경우 문제가 발생할 수 있습니다. 

<br/>

그래서 앞서 'kubeadm init' 단계에서 가상머신 네트워크 대역과 겹치는 

192.168.0.0/16 네트워크를 사용하지 않고 172.16.0.0/16으로 변경하여 사용했습니다.

<br/>

kubeadm은 CNI(Container Network Interface) 기반 네트워크만 지원하며 

kubenet은 지원하지 않습니다. <br/>

CNI(Container Network Interface)는 CNCF(Cloud Native Computing Foundation)의 프로젝트 중 하나로 

컨테이너 간의 네트워킹을 제어할 수 있는 플러그인을 만들기 위한 표준 인터페이스입니다. 

CNI(Container Network Interface)를 기반으로 만들어진 pod network add-on에는 여러 종류가 있고 

'[httpsx://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network)' 페이지에서 

각 pod network add-on 마다 설치법이 소개되어 있습니다. 여기서는 Calico를 사용하려고 합니다.

<br/>

<br/>

Calico 설치 방법은 위에 소개했던 페이지에서도 확인할 수 있지만 'https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/calico' 페이지에도 소개되어 있습니다. 

Calico는 OSI(Open System Interconnection) 모델의 3계층(네트워크 계층)을 기반으로 합니다. 

BGP(Border Gateway Protocol)를 사용하여 노드 간 통신을 용이하게 하는 라우팅 테이블을 작성하고, 

Flannel 기반 네트워크 시스템보다 우수한 성능 및 네트워크 격리를 제공 할 수 있습니다.

<br/>

'kubeadm init' 단계에서 --pod-network-cidr 설정 값을 172.16.0.0/16으로 변경하여 

사용하기는 했지만 Calico YAML 파일에서도 값을 변경하여 설치해야 합니다.

```sh
# curl -O -L https://docs.projectcalico.org/v3.8/manifests/calico.yaml
# sed s/192.168.0.0\\/16/172.16.0.0\\/16/g -i calico.yaml
# kubectl apply -f calico.yaml
configmap/calico-config configured
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers configured
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers configured
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers unchanged
```



<br/>

#### 4-7. kube-apiserver 실행 확인

```sh
# docker ps | grep kube-apiserver
c551686af0db        b305571ca60a           "kube-apiserver --..."   22 minutes ago      Up 22 minutes                           k8s_kube-apiserver_kube-apiserver-localhost.localdomain_kube-system_eaef44bf2ab1a21afbd7d32e67ae5a1f_0
c8741f978bc2        k8s.gcr.io/pause:3.1   "/pause"                 22 minutes ago      Up 22 minutes                           k8s_POD_kube-apiserver-localhost.localdomain_kube-system_eaef44bf2ab1a21afbd7d32e67ae5a1f_0
```

<br/>



#### 4-8. kubelet 버전확인

```sh
# kubectl version
Client Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.0", GitCommit:"2bd9643cee5b3b3a5ecbd3af49d09018f0773c77", GitTreeState:"clean", BuildDate:"2019-09-18T14:36:53Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.0", GitCommit:"2bd9643cee5b3b3a5ecbd3af49d09018f0773c77", GitTreeState:"clean", BuildDate:"2019-09-18T14:27:17Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"linux/amd64"}
```



<br/>

#### 4-9. kube-system pods 조회

```sh
# kubectl get pods -n kube-system
NAME                                            READY   STATUS              RESTARTS   AGE
calico-kube-controllers-564b6667d7-hvccr        0/1     ContainerCreating   0          50s
calico-node-sbsx2                               0/1     PodInitializing     0          51s
coredns-5644d7b6d9-cz4g5                        0/1     ContainerCreating   0          23m
coredns-5644d7b6d9-tbxxb                        0/1     ContainerCreating   0          23m
etcd-localhost.localdomain                      1/1     Running             0          22m
kube-apiserver-localhost.localdomain            1/1     Running             0          22m
kube-controller-manager-localhost.localdomain   1/1     Running             0          22m
kube-proxy-lnq7l                                1/1     Running             0          23m
kube-scheduler-localhost.localdomain            1/1     Running             0          22m
```



<br/>

<br/>



## 5. kubernetes node join



Worker 노드가 되기 위해서는 'kubeadm join' 명령어를 실행해서 Master 노드에 등록해야 합니다. 'kubeadm join' 명령어 실행에 필요한 옵션들은 Master 노드에서 아래 명령어를 실행해서 확인할 수 있습니다.

```sh
# kubeadm token create --print-join-command
kubeadm join 192.168.56.101:6443 --token id1p6s.ye6l2dt0pqzn5ur2     --discovery-token-ca-cert-hash sha256:8d02987dbd07e86e0a2fcbed04f98b6d45864f31174fc303bdc9e3a62a0233e6
```



<br/>

#### 5-1. node1.net join

node1.net 장비에서 실행함.

```sh
# kubeadm join 192.168.56.101:6443 --token 52fbkz.qe319zph7ae2vc2o \
    --discovery-token-ca-cert-hash sha256:9854795f543183babab7284413187e7a380d93eeca6fe5c07cebcca87233502c
[preflight] Running pre-flight checks
        [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
        [WARNING HTTPProxy]: Connection to "https://192.168.56.101" uses proxy "http://192.168.56.1:3128". If that is not intended, adjust your proxy settings
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.16" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Activating the kubelet service
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

<br/>

#### 5-2. node2.net join

node2.net 장비에서 실행함

```sh
# kubeadm join 192.168.56.101:6443 --token 52fbkz.qe319zph7ae2vc2o \
    --discovery-token-ca-cert-hash sha256:9854795f543183babab7284413187e7a380d93eeca6fe5c07cebcca87233502c
[preflight] Running pre-flight checks
        [WARNING Hostname]: hostname "node3.net" could not be reached
        [WARNING Hostname]: hostname "node3.net": lookup node3.net on [::1]:53: read udp [::1]:43038->[::1]:53: read: connection refused
        [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
        [WARNING HTTPProxy]: Connection to "https://192.168.56.101" uses proxy "http://192.168.56.1:3128". If that is not intended, adjust your proxy settings
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.16" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Activating the kubelet service
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

<br/>

#### 5-3. kubernetes cluster 조회

마스터 노드에서 실행함.

```sh
# kubectl cluster-info
Kubernetes master is running at https://192.168.56.101:6443
KubeDNS is running at https://192.168.56.101:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

<br/>

#### 5-4. kubernetes 노드 상태 조회

약 2분 정도 경과후에 확인

```sh
# kubectl get nodes
NAME          STATUS   ROLES    AGE     VERSION
master1.net   Ready    master   169m    v1.16.0
node1.net     Ready    <none>   155m    v1.16.0
node2.net     Ready    <none>   2m53s   v1.16.0


# kubectl get nodes --all-namespaces
NAME          STATUS   ROLES    AGE    VERSION
master1.net   Ready    master   170m   v1.16.0
node1.net     Ready    <none>   156m   v1.16.0
node2.net     Ready    <none>   3m9s   v1.16.0
```

<br/>

<br/>

## 6. kubernetes 관련 docker 조회

#### 6-1. master 노드

```sh
# docker images
REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
k8s.gcr.io/kube-apiserver             v1.16.0             b305571ca60a        2 days ago          217 MB
k8s.gcr.io/kube-proxy                 v1.16.0             c21b0c7400f9        2 days ago          86.1 MB
k8s.gcr.io/kube-controller-manager    v1.16.0             06a629a7e51c        2 days ago          163 MB
k8s.gcr.io/kube-scheduler             v1.16.0             301ddc62b80b        2 days ago          87.3 MB
k8s.gcr.io/etcd                       3.3.15-0            b2756210eeab        2 weeks ago         247 MB
k8s.gcr.io/coredns                    1.6.2               bf261d157914        5 weeks ago         44.1 MB
docker.io/calico/node                 v3.8.2              11cd78b9e13d        5 weeks ago         189 MB
docker.io/calico/cni                  v3.8.2              c71c24a0b1a2        5 weeks ago         157 MB
docker.io/calico/kube-controllers     v3.8.2              de959d4e3638        5 weeks ago         46.8 MB
docker.io/calico/pod2daemon-flexvol   v3.8.2              96047edc008f        5 weeks ago         9.37 MB
k8s.gcr.io/pause                      3.1                 da86e6ba6ca1        21 months ago       742 kB
```

<br/>

#### 6-2. worker 노드

```sh
# docker images
REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
k8s.gcr.io/kube-proxy                 v1.16.0             c21b0c7400f9        2 days ago          86.1 MB
docker.io/calico/node                 v3.8.2              11cd78b9e13d        5 weeks ago         189 MB
docker.io/calico/cni                  v3.8.2              c71c24a0b1a2        5 weeks ago         157 MB
docker.io/calico/pod2daemon-flexvol   v3.8.2              96047edc008f        5 weeks ago         9.37 MB
k8s.gcr.io/pause                      3.1                 da86e6ba6ca1        21 months ago       742 kB
```







