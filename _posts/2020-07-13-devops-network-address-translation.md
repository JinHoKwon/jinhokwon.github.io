---
title: Network Address Translation
comments: true
tags: [devops, network]
categories: devops
header:
  teaser: "/assets/images/network.png"
---
네트워크 주소변환(NAT)은 
사설 IP(내부 네트워크)를 공인 IP로 변경하거나,
공인 IP로 들어오는 패킷을 사설 IP로 변환하는 것을 말합니다.

인터넷 공유기를 사용하여 여러대의 PC가 인터넷이 가능한 이유 또한
공유기에서 NAT 기능을 지원하기 때문입니다.

NAT 기능을 활용할 경우
여러대의 PC가 하나의 인터넷 회선을 공유하여 사용할 수 있다는 장점이 있는 반면에,
인터넷을 사용하는 PC의 숫자가 늘어날수록 속도저하가 발생할 수 있는 단점이 있습니다.
