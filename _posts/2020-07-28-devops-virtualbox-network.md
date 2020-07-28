---
title: Virtualbox network
tags: [devops, virtualbox]
comments: true
categories: devops
header:
  teaser: "/assets/images/virtualbox/virtualbox_logo.png"
---



분산환경에서 HA를 지원하는 오픈소스 기반의 솔류션(예 : 쿠버네티스, 엘라스틱서치 등등)을 사용하기 위해서는 <br/>

적어도 3대 이상의 VM과 VM을 연결시켜주는 네트워크 설정이 적절히 되어 있어야 합니다.

<br>

본 문서에서는 VirtualBox를 활용하여, <br/>

분산환경을 경험하기위한, 네트워크 설정 방법에 대해서 설명하고 있습니다.<br/>

<br/>

## 1. Virtualbox network 



요약하면, 첫번째 네트워크 어뎁터는 NAT로 설정하고,

두번째 네트워크 어뎁터는 Host only network로 설정하면 됩니다.

<br/>

해당 네트워크 설정에 대한 의미는 아래 링크를 참고하시면 됩니다.

<br/>

[NAT](/devops/devops-network-address-translation/)<br/>

[Host only network](/devops/devops-host-only-network/)<br/>
<br/>

## 2. Virtualbox network 설정



#### 2-1. Host only network 구성

Virtualbox > 파일 > 호스트 네트워크 관리자 메뉴를 클릭합니다.

<br/>

![virtualbox_menu_file_hostnetwork_manager](/assets/images/virtualbox/virtualbox_menu_file_hostnetwork_manager.png)

<br/>

호스트 네트워크 관리자 화면에서, 다음 이미지와 같이 설정합니다.

참고로, 좀 더 많은 IP 대역 할당을 위해서 IPv4 서브넷 마스크를 `255.255.0.0`으로 설정하였습니다.

<br/>

![virtualbox_host_network_manager](/assets/images/virtualbox/virtualbox_host_network_manager.png)

<br/>



#### 2-2. Virtualbox VM 

VM 을 생성하고 설정 > 네트워크를 클릭한 후,

어뎁터1에는 `NAT`를 선택합니다.

<br/>

![virtualbox_vm_network_adaptor1](/assets/images/virtualbox/virtualbox_vm_network_adaptor1.png)

<br/>

그리고, 어뎁터2에는 `호스트 전용 어뎁터`를 선택합니다.

<br/>

![virtualbox_vm_network_adaptor2](/assets/images/virtualbox/virtualbox_vm_network_adaptor2.png)

<br/>

이와 같이 NAT, Host only network 와 같은 네트워크 설정을 진행하게 되면,<br/>

동일한 네트워크안에 포함된 모든 VM들은 서로 다른 VM 간의 통신 뿐만 아니라<br/>

외부 인터넷과 같은 통신 또한 가능하게 됩니다.

