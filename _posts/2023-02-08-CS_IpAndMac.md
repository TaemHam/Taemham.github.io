---
title: "[CS] IP & MAC: 컴퓨터에서 보낸 메시지는 길을 어떻게 찾아가지?"
author: TaemHam
date: 2023-02-08 09:00:00 +0900
categories: [CS, Network]
tags: [CS, Network, IP]
---

![](https://www.investopedia.com/thmb/Sh6RKE3ItNkCDYbjI40DcuLlaS4=/750x0/filters:no_upscale():max_bytes(150000):strip_icc():format(webp)/IPaddress-1e113dd325f74135ab2183d432ea5865.jpg)

## 개요

"컴퓨터에서 보낸 메시지는 길을 어떻게 찾아가지?"라는 질문에 대해 보통은 IP 주소를 가지고 찾아간다고 한다. 하지만 이는 엄밀히 말하면 정확한 표현 방법이 아니다. 이번 글에서는 컴퓨터끼리 통신할 때 사용되는 기술들이 어떤 것이 있으며, 그 기술이 어떻게 사용되어 통신이 이루어지는지 알아보도록 하자.

## IP

위 질문에 대답하기 위해서는, 네트워크 통신에 사용되는 프로토콜인 IP가 무엇인지 알아야 한다.

### 정의

IP(Internet Protocol)는 송신 호스트와 수신 호스트가 패킷 교환 네트워크에서 정보를 주고받는 데 사용하는 정보 위주의 규약이며, OSI 네트워크 계층에서 호스트의 주소 지정과 패킷 분할 및 조립 기능을 담당한다. 쉽게 이야기 하면, **메시지를 주고 받을 때 필요한 출발지와 도착지에 관한 정보를 어디에 어떻게 써 넣을지를 정해놓은 것이다**.

### IP 주소

여기서 이 '출발지'와 '도착지'를 표현해 놓은 것이 IP 주소이다. 정확히는, **네트워크 환경에서 컴퓨터(노드) 간 통신하기 위해 각 컴퓨터에 부여된 네트워크 상 주소**이다. 

이 IP 주소가 결정되는 방식은 개인의 집 주소를 나라에서 정해주듯이, '한국 인터넷 정보 센터(KRNIC)'라는 기관에서 인터넷 사업체에게 IP를 할당해주고, 그 IP를 다시 해당 서비스의 사용자에게 할당해 주는 것으로 정해진다.

### IP 주소 체계

![](https://www-static.cdn-one.com/cmsimages/en_41-what-is-ipv6-illustration_%20ip-07-2x.png)

IP 주소 체계는 IPv4와 IPv6로 나뉜다. 

#### IPv4

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbL627k%2FbtrcOO2icqe%2FWdt1wukdgkB6spvFQsMYRK%2Fimg.png)

둘 중 먼저 나온 것은 IPv4이다. 총 32비트 길이의 주소를 8비트씩 4부분으로 나누어 10진수로 바꾸고, `.`으로 구분해 표기한다. 최소 `0.0.0.0` 부터 최대 `255.255.255.255`가 나온다.

* 클래스 기반 할당

IP 주소 체계가 처음 나왔을 때 사용한 방식이다. 장치가 속해 있는 네트워크를 구분하기 위해 클래스를 A, B, C, D, E 총 5개로 나누었고, IP 주소의 앞 부분은 네트워크 주소, 나머지 부분은 컴퓨터에 부여하는 호스트 주소로 구성되어 있다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbZVUCs%2FbtrcXjUul4b%2Feu3dSspA3SvIlPxEFiS06k%2Fimg.png)

위 도표를 보면 알 수 있듯, 주소의 첫 비트가 0이면 클래스 A, 두번째 비트가 0 이면 클래스 B 인 방식이다. 클래스에 따라 네트워크 주소를 표현하는데 사용하는 바이트 갯수가 달라서, 클래스마다 사용할 수 있는 호스트 주소 갯수도 다르다.

#### IPv6

하지만 컴퓨터의 보급률이 늘어나고, IP 주소를 필요로 하는 장치가 많아지면서 발급해 줄 IP가 부족하게 되었는데, 이를 해소하기 위해 나온 방식 중 하나가 IPv6이다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbB9X1Z%2FbtrcXkeWFVx%2Fb1u3KHgbcxVZGudGXtlYCk%2Fimg.png)

IPv6는 총 128비트 길이의 주소를 16비트씩 8부분으로 나누어 16진수로 바꾸고, `:`으로 구분해 표기한다.

길이가 길어진 만큼 표현할 수 있는 주소의 갯수도 급수적으로 늘어났다. 2^32 와 2^128 의 차이인 것이다.

문제는 아직까지도 IPv4에서 IPv6로의 전환이 완료되지 않았다는 점이다. 아직까지도 많은 라우터들은 IPv4를 사용한다. IPv6로의 전환은 많은 시간과 비용이 들기 때문에 완료되기까지 적지 않은 세월이 걸릴 것이다. 현재는 IPv4와 IPv6를 혼용해서 사용하는데, IPv4 라우터에서는 tunneling이라는 방식을 사용해 IPv6 데이터그램을 전송한다.

#### NAT

IP 주소 부족을 해결하는 또 다른 방식으로는 NAT가 있다. 

NAT(Network Address Translation)은 패킷이 라우팅 장치를 통해 전송되는 동안 패킷의 IP 주소 정보를 수정해 IP 주소를 다른 주소로 매핑하는 방법이다. 이 방식을 사용하면 공인 IP 하나만으로 사설 IP를 여러개 두어 많은 주소를 처리할 수 있게 된다. NAT가 사용된 대표적인 장치로 인터넷 공유기가 있다.

#### DHCP

또, 클래스 기반 할당 방식은 사용하는 주소보다 버리는 주소가 많다는 단점이 있었는데, 이를 해결하기 위해 나온 것이 DHCP 이다.

DHCP(Dynamic Host Configuration Protocol)는 IP 주소, 서브넷마스크, 게이트웨이 등 TCP/IP 프로토콜의 기본 설정 값들을 자동으로 할당하기 위한 네트워크 관리 프로토콜이다. 이 기술을 통해 IP 주소를 수동으로 설정할 필요 없이 인터넷에 접속할 때마다 자동으로 IP 주소를 할당받을 수 있다.

## MAC

위에서 설명했듯, IP는 회선의 주소일 뿐 컴퓨터의 주소가 아니다. IP 주소로만 통신하는 것은, 편지를 보내는데 동호수 없이 아파트 이름만 적어 놓고 보내는 것과 비슷하다. 때문에 통신에 사용될 세부적인 주소가 필요한데, 이 때 사용되는 것이 MAC(Media Access Control) 주소이다.

### MAC 주소

MAC 주소는 하드웨어에 부여된 고유한 식별 번호이다. 이 주소는 하드웨어를 생산할 때 부여되는 것으로 임의로 바뀌는 것이 아니다. 따라서 인터넷 상의 장치들의 실제 주소를 식별하는 용도로 사용된다.

![](https://miro.medium.com/max/1200/1*FLrfO7JzkOWSBkPBYly37w.png)

MAC 주소는 48비트 길이의 주소를 8비트씩 6부분으로 나누어 16진수로 바꾸어 표기한다. 구분자는 `:`, `-`, `.` 등 다양하게 사용된다.

### ARP

그렇다면 컴퓨터는 IP만 가지고 어떻게 상대방 컴퓨터로 데이터를 전송하는 걸까? 네트워크 단에서 IP 주소로 MAC 주소를 검색할 수 있도록 해주는 프로토콜을 통해 주소를 바꾸어 통신하는데, 이 프로토콜을 ARP 라고 한다.

ARP(Address Resolution Protocol)는 IP 주소를 통해 MAC 주소를 얻을 수 있도록 다리 역할을 해주는 프로토콜이다. ARP를 통해 가상 주소인 IP 주소를 실제 주소인 MAC 주소로 변환하고, 반대로 RARP를 통해 실제 주소인 MAC 주소를 가상 주소인 IP 주소로 변환하기도 한다.

이 때 변환은 다음과 같은 방식으로 이루어진다:
1. 데이터를 전송하려는 장치는 보낼 IP에 대해 ARP Request 브로드캐스트(네트워크에 연결된 모든 호스트에 전송)를 보낸다.
2. 해당 주소에 맞는 장치는 ARP Reply 유니캐스트(정해진 호스트로 1:1 전송)를 통해 MAC 주소를 반환한다.

## 마무리

이제 처음의 질문에 대한 좀 더 정확한 대답을 해보자면, "컴퓨터는 ARP를 통해 IP 주소에서 MAC 주소를 찾고,  해당 MAC 주소를 기반으로 통신한다"고 할 수 있다.

## 참고 자료
***

* [[네트워크] IP,IP 클래스, IPv4, IPv6이란? | IP 클래스 구분](https://code-lab1.tistory.com/33)
* [https://jhnyang.tistory.com/404](https://jhnyang.tistory.com/404)