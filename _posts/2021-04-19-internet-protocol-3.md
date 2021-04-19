---
layout: post
title: "[김효곤 교수의 인터넷 프로토콜] 3. Address Resolution Protocol(ARP)"
categories:
  - Internet Protocol
tags:
  - 인터넷, 프로토콜, 그리고 계층화 원칙

---

# Address Resolution Protocol(ARP)

### 3.1 ARP의 개념

- ARP는
  - 매핑을 결정해야 할 IP 주소를 담은 ARP 메시지를 공유매체 데이터링크 기술의 broadcast로 전송하고
  - 그 IP 주소가 할당된 인터페이스가 자신의 MAC 주소를 ARP 응답으로 보냄으로써 해결(resolution)된다.

### 3.2 ARP 프로토콜의 작동 개요

- ARP는 공유 매체형 링크에 연결된 모든 컴퓨터(라우터)에 구현
  - 
- ARP는 스스로 나서지 않는다.
  - 어떤 패킷을 보내야하는데, 목적지의 MAC 주소를 모를시에만 ARP에 의해 MAC 주소를 찾아나선다. 이외에 해당 IP 주소의 MAC 주소가 필요하지 않은 상황에서는 굳이 ARP가 움직이지 않는다.
- ARP는 "다음 홉" IP 주소만 해결한다.
  - 라우팅 테이블에서 보았듯이, IP는 목적지에 가기 위해 가야아하는 다음 홉만을 경로로 정의하며, ARP는 이 다음홉만 해결한다.
- 패킷을 특정 IP 주소로 전송해야 할 때
  1. 라우팅 테이블에서 해당 목적지 IP 주소로 가기 위해 거칠 다음 홉의 IP 주소를 찾는다.
  2. 이 IP 주소를 갖고 ARP 캐쉬를 뒤져 MAC 주소를 찾는다.
  3. 데이터링크 헤더 중 Destination Address 정보에 목적지 MAC 주소를 넣고, IP 데이터그램에는 목적지 IP 주소를 넣는다.

![]({{site.url}}/assets/images/102.png)

### 라우팅 테이블 및 ARP 캐쉬보기

- 라우팅 테이블
  - route -4 print
  - netstat -r
  - netsh interface ip show route
- ARP 캐쉬
  - arp -a [-v]
  - netsh interface ip show neighbor

### 3.3 ARP 패킷 형식

![]({{site.url}}/assets/images/103.png)

- Hardware = 