---
layout: post
title: "[김효곤 교수의 인터넷 프로토콜] 10. Dynamic Host Configuration Protocol(DHCP)"
categories:
  - Internet Protocol
tags:
  - 인터넷, 프로토콜, 그리고 계층화 원칙



---

# Dynamic Host Configuration Protocol(DHCP)

- IPv4 속성에 가보면 `자동으로 IP 주소 받기`와 같은 항목에 선택이 거의 다 되어있을 텐데, 이것을 선택하면 DHCP를 사용하겠다는 뜻이 된다.
- 추가적으로 그 밑에는 `자동으로 DNS 서버 주소 받기` 항목이 있는 데, 도메인 이름만 알면 해당 도메인의 IP주소를 찾아주는 DNS 서버의 주소를 알아내는 것도 DHCP를 사용하는 것이다.

### 10.1 DHCP 프로토콜의 역사

- Reverse ARP (RARP) 
  - '나'의 MAC 주소 -> '나'의 IP 주소
    - 번역 방향이 ARP('남'의 IP주소 -> '남'의 MAC 주소)와 반대이다.
  - 1990년대 초 diskless workstation – no stable storage
    - PC가 비싸다보니 하드디스크가 없는 PC가 많았는데, 하드디스크가 없으니 전원을 끄면 정보가 다 사라질 수 밖에 없었다.
  - 전원을 끄면 OS image 마저 사라짐
    - 부팅을 하려면 OS image를 부트 서버에서 실어 와야 하는 상황
  - 부트 서버와의 네트워킹은?
    - PROM에 들어 있는 짧은 RARP 코드 구동하여 자기 자신의 IP 주소를 받아오고
    - 그 IP 주소에 있는 부트 서버에서 Trivial FTP (TFTP)로 OS image 받아 부팅
  - IP 주소를 받아 오는 과정
    - (burnt-in) MAC 주소를 RARP 서버에서 검색하여 IP 주소 공급
    - 브로드 캐스트를 통해 IP주소를 물으면 그것을 들은 RARP가 응답
  - ARP와 같은 프로토콜 패밀리 - 패킷 포맷 거의 동일하게 생겼다.
    - Ethertype: 0x0806 대신 0x8035
    - 요청 = 3, 응답 = 4

### 10.1 DHCP 프로토콜의 계보

- Reverse ARP (RARP) 메시지 포맷

  - déjà vu??

    ![]({{site.url}}/assets/images/139.png)

    - Frame Type에서는 0x0806대신 0x8035
    - Operation에선 3 or 4

    - ARP 요청에서는 남의 MAC 주소를 모르기 때문에 Target Mac 주소 필드가 빈칸이었다면, RARP 요청에선 나의 IP주소를 모르기 때문에 Sender IP address가 비어있다.

  - 패밀리에는 InARP 라는 프로토콜도 있다

    - ARP: your IP -> your MAC
    - RARP: my MAC -> my IP
    - InARP: your MAC -> your IP

- Bootstrap Protocol (BOOTP)
  - RARP의 성공 -> 원격 부팅 과정을 전반적으로 책임지는 프로토콜
  - RARP의 한계
    - IP 주소만을 가져오는 너무 간단한 프로토콜 -> 별도의 프로토콜 (예: TFTP)이 필요
      - TFTP : OS image를 받아오는 프로토콜
    - 링크 (broadcast domain) 하나마다 RARP 서버가 하나씩 있어야 함
      - ARP처럼 요청이 ff-ff-ff-ff-ff-ff로 broadcast되기 때문 (서버가 어디 있는지 모름)
    - RARP는 계층화 원칙과 무관한 해킹에 불과
      - IP 주소를 가져오는 작업이 이더넷을 사용해서 요청하고 응답이 오면 인터페이스를 만들어서 IP를 거기에 붙이는 작업을 커널에서 해줘야 하는데, OS image를 못가져온 상태이므로 해킹을 통해서 할 수 밖에 없었다.
  - BOOTP의 기능
    - IP 주소 외의 많은 네트워크 파라미터를 configure할 수 있으며, 부트 이미지도 요청할 수 있음
    - BOOTP relay -> BOOTP 서버는 원격 위치에 1대만 있어도 무방
    - **응용 계층 프로토콜**로서 계층화 원칙 완벽 준수
      - 해킹이 아니라 정상적인 계층화 원칙을 지키며 IP주소를 할당할 수 있게 됨
  - DHCP의 직전 단계
    - DHCP 모습에 아주 가까운 상태

### 10.2 DHCP의 장점

- DHCP의 장점 = BOOTP의 장점 + a

- BOOTP의 장점

  - 200가지에 가까운 네트워크 파라미터 설정 가능

    - https://www.iana.org/assignments/bootp-dhcp-parameters/bootp-dhcp-parameters.xhtml
    - 라우터(3), Domain Name Server (6) 등등 굉장히 많다. 파워풀한 configuration 프로토콜

  - DHCP relay (BOOTP relay)

    - 라우터에 relay 기능 구현
      - 같은 링크 안에 DHCP 서버 대신 relay agent라는 라우터이다. 이 라우터에다가 BOOTP relay(현재는 DHCP relay)라는 코드가 추가된다. 이 코드에 의해서 아주 멀리 DHCP 서버까지 relay가 가능해진다. 각각 다른 라우터들이 사이사이에 있어도 상관없다. 
      - DHCP 클라이언트는 마치 링크 안에 서버가 있는 것처럼 브로드캐스트로  DHCP를 요청하면 보내면 링크 안에 존재하는 agent가 unicast로 원격에 있는 DHCP 서버까지 보내준다. 그리고 서버가 이에 대한 응답으로 해당 agent로 보내주면 그것을 다시 클라이언트로 보내주는 것이다.

    ![]({{site.url}}/assets/images/142.png)

  - 정상적인 계층화

    - UDP를 트랜스포트로 사용하는 응용 계층 프로토콜
      - 서버 포트 = 67
      - 클라이언트 포트 = 68

- a (BOOTP에는 없는 것) = “LEASE” = "임대"

  - LEASE: IP 주소의 자동 할당 및 회수 메커니즘
    - 임대 기간이 존재하며, 이 임대 기간이 지나면 회수됨
    - MAC-IP 매핑을 관리자가 수동으로 관리해야 하는 BOOTP에서 진일보

