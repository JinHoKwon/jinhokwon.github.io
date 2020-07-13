---
title: Host only network
tags: [devops, network]
categories: devops


---
Host only network는
외부와 단절된 내부 네트워크를 구축할 때 사용합니다.
즉, 내부 네트워크로 구성된 PC 들 간에는 통신은 가능하지만,
인터넷과 같은 외부 접속은 불가능합니다.

만약, VirtualBox 환경에서,
내부 네트워크와 외부 인터넷 까지 가능한 환경을 구축하고자 한다면,
NAT 어뎁터(내부 통신용)와 호스트 전용 어뎁터(외부 인터넷 접속용)를 
함께 연결해 주면 됩니다. 
