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



#### 4-2. proxy 서버 설정

필요시 proxy 서버를 설정함.

```sh
# export http_proxy=192.168.56.1:3128
# export https_proxy=192.168.56.1:3128
```



#### 4-3. docker images 조회

```sh
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
```





#### 4-4. master 초기화

master 노드에만 적용함.

초기화 작업은 'https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/' 페이지를 참고했습니다. 

pod network add-on의 종류에 따라서 --pod-network-cidr 설정 값을 다르게 해주어야 합니다. 

여기서는 pod network add-on로 Calico를 사용해서 

--pod-network-cidr 설정 값을 192.168.0.0/16으로 해야 하지만 

가상머신 네트워크 대역인 192.168.37.0/24와 겹치기 때문에 

172.16.0.0/16으로 변경하여 사용합니다.



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





#### 4-5. kube config 설정

master 노드에서만 진행

```sh
# mkdir -p $HOME/.kube
# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
# sudo chown $(id -u):$(id -g) $HOME/.kube/config
```







#### 4-6. pod network add on 설치

pod network add-on 설치는 Master 노드에서만 합니다.

Kubernetes 환경에서 Pod들이 통신하려면 pod network add-on이 있어야 합니다. 

pod network add-on이 설치되기 전에는 CoreDNS가 시작되지 않습니다. 



'kubeadm init' 이후에 'kubectl get pods -n kube-system' 명령어로 확인해보면 

CoreDNS가 아직 시작되지 않은 상태를 확인할 수 있습니다. 



pod network와 호스트 네트워크가 겹칠 경우 문제가 발생할 수 있습니다. 



그래서 앞서 'kubeadm init' 단계에서 가상머신 네트워크 대역과 겹치는 

192.168.0.0/16 네트워크를 사용하지 않고 172.16.0.0/16으로 변경하여 사용했습니다.



kubeadm은 CNI(Container Network Interface) 기반 네트워크만 지원하며 

kubenet은 지원하지 않습니다. 

CNI(Container Network Interface)는 CNCF(Cloud Native Computing Foundation)의 프로젝트 중 하나로 

컨테이너 간의 네트워킹을 제어할 수 있는 플러그인을 만들기 위한 표준 인터페이스입니다. 

CNI(Container Network Interface)를 기반으로 만들어진 pod network add-on에는 여러 종류가 있고 

'[httpsx://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network)' 페이지에서 

각 pod network add-on 마다 설치법이 소개되어 있습니다. 여기서는 Calico를 사용하려고 합니다.



Calico 설치 방법은 위에 소개했던 페이지에서도 확인할 수 있지만 'https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/calico' 페이지에도 소개되어 있습니다. 

Calico는 OSI(Open System Interconnection) 모델의 3계층(네트워크 계층)을 기반으로 합니다. 

BGP(Border Gateway Protocol)를 사용하여 노드 간 통신을 용이하게 하는 라우팅 테이블을 작성하고, 

Flannel 기반 네트워크 시스템보다 우수한 성능 및 네트워크 격리를 제공 할 수 있습니다.



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





#### 4-7. kube-apiserver 실행 확인

```sh
# docker ps | grep kube-apiserver
c551686af0db        b305571ca60a           "kube-apiserver --..."   22 minutes ago      Up 22 minutes                           k8s_kube-apiserver_kube-apiserver-localhost.localdomain_kube-system_eaef44bf2ab1a21afbd7d32e67ae5a1f_0
c8741f978bc2        k8s.gcr.io/pause:3.1   "/pause"                 22 minutes ago      Up 22 minutes                           k8s_POD_kube-apiserver-localhost.localdomain_kube-system_eaef44bf2ab1a21afbd7d32e67ae5a1f_0
```





#### 4-8. kubelet 버전확인

```sh
# kubectl version
Client Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.0", GitCommit:"2bd9643cee5b3b3a5ecbd3af49d09018f0773c77", GitTreeState:"clean", BuildDate:"2019-09-18T14:36:53Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.0", GitCommit:"2bd9643cee5b3b3a5ecbd3af49d09018f0773c77", GitTreeState:"clean", BuildDate:"2019-09-18T14:27:17Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"linux/amd64"}
```





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









## 5. kubernetes node join



Worker 노드가 되기 위해서는 'kubeadm join' 명령어를 실행해서 Master 노드에 등록해야 합니다. 'kubeadm join' 명령어 실행에 필요한 옵션들은 Master 노드에서 아래 명령어를 실행해서 확인할 수 있습니다.

```sh
# kubeadm token create --print-join-command
kubeadm join 192.168.56.101:6443 --token id1p6s.ye6l2dt0pqzn5ur2     --discovery-token-ca-cert-hash sha256:8d02987dbd07e86e0a2fcbed04f98b6d45864f31174fc303bdc9e3a62a0233e6
```





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



#### 5-3. kubernetes cluster 조회

마스터 노드에서 실행함.

```sh
# kubectl cluster-info
Kubernetes master is running at https://192.168.56.101:6443
KubeDNS is running at https://192.168.56.101:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```



















##### 5-4. kubernetes 노드 상태 조회

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



## 7. kubernetes dashboard

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



```sh
# kubectl proxy --port=8001 --address=192.168.56.101 --accept-hosts='^*$'
```





#### 7-2. dashboard 계정 생성

```sh
# kubectl create serviceaccount cluster-admin-dashboard-sa
serviceaccount/cluster-admin-dashboard-sa created

# kubectl create clusterrolebinding cluster-admin-dashboard-sa --clusterrole=cluster-admin --serviceaccount=default:cluster-admin-dashboard-sa
clusterrolebinding.rbac.authorization.k8s.io/cluster-admin-dashboard-sa created
```



#### 7-3. dashboard 계정 토큰 확인

```sh
# kubectl get secret $(kubectl get serviceaccount cluster-admin-dashboard-sa -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
eyJhbGciOiJSUzI1NiIsImtpZCI6IjZNT1E1VkxuTlN3RFFoTHdsUDZYWTVpc1FzSEZlN2MwZUdqZFpfQVdiNEkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImNsdXN0ZXItYWRtaW4tZGFzaGJvYXJkLXNhLXRva2VuLXZjc2h6Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImNsdXN0ZXItYWRtaW4tZGFzaGJvYXJkLXNhIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiNGMyODc2MzMtNjFkMy00N2RjLTk4ZTctMGM0MGQ0OGY5ZWUxIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRlZmF1bHQ6Y2x1c3Rlci1hZG1pbi1kYXNoYm9hcmQtc2EifQ.J6xmvTqNI-a6KWzjS-JetPw25eFAaTIvO0aouVppUFrk0CqyaVNRMy5untGzLwi8ujrYS__64yAIlTQGw9wiXn7_lW4v8uyUrSUarmxa_UH5h-WDu6ans3QexLIi-Bop3zrmwUYRRGjhBC-FojduyVAgcKQR1Iwcd5cdFyDGORK7TejBcdZvICDqVlp8i5F4mweGqXx0VslptBXdGP-AjaLPnVxgcWstdXOIXpXys57w92SINZH4CEErkJzqDvOCr1v8sE9yynBkOBZLo5vT1Zjtg0yJ41vbTJvOjkUHrjPGbyHEFV1T87OEo-hpHk0Djga5zgDvO8o2-HEH6LJ_qQ
```





## 9. kubernetes hello world 실습

##### docker 로그인

```sh
# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: jinhokwon
Password:
Login Succeeded
```





##### 샘플 이미지 pull

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



```sh
# docker run -d --name spring-boot-rest-docker \
-e "SERVER_PORT=8080" \
-p 8080:8080 \
jinhokwon/spring-boot-rest-docker:latest

68f6fe53e7abbb35ed59c316ebb17f11712cb1de30ecfb067dcd0ada10426407
```



```sh
# docker ps -a | grep spring
283ad11985f3        jinhokwon/spring-boot-rest-docker:latest   "java -Djava.securit…"   10 seconds ago      Up 9 seconds                0.0.0.0:8080->8080/tcp   spring-boot-rest-docker
```





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



```sh
# curl localhost:8080
{
  "result" : "first",
  "hostname" : "283ad11985f3"
}
```



```sh
# docker stop spring-boot-rest-docker
spring-boot-rest-docker
```



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



```sh
# kubectl create -f /tmp/spring-boot-rest-pod.yaml
pod/spring-boot-rest-pod created
service/spring-boot-rest-service created
```





```sh
# kubectl get pods
NAME                   READY   STATUS              RESTARTS   AGE
spring-boot-rest-pod   0/1     ContainerCreating   0          15s
```



잠시후..

```sh
# kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
spring-boot-rest-pod   1/1     Running   0          34s
```







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



##### pod 테스트

```sh
# curl 172.16.229.131:8080
{
  "result" : "first",
  "hostname" : "spring-boot-rest-pod"
}
```





##### pod 제거

```sh
# kubectl delete -f /tmp/spring-boot-rest-pod.yaml
pod "spring-boot-rest-pod" deleted
service "spring-boot-rest-service" deleted
```



















### 8. kubernetes node 삭제

##### master 장비에서 실행

```sh
# kubectl delete node node1.net
node "node1.net" deleted

# kubectl delete node node2.net
node "node2.net" deleted
```



##### 해당 노드에서 실행

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




