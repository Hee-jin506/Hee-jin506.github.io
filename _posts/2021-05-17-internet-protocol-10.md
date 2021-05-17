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
    - 링크 (broadcast domain)마다 RARP 서버가 하나씩 있어야 함
      - ARP처럼 요청이 ff-ff-ff-ff-ff-ff로 broadcast되기 때문 (서버가 어디 있는지 모름)
    - RARP는 계층화 원칙과 무관한 해킹에 불과
  - BOOTP의 기능
    - IP 주소 외의 많은 네트워크 파라미터를 configure할 수 있으며, 부트 이미지도 요청할 수 있음
    - BOOTP relay -> BOOTP 서버는 원격 위치에 1 대만 있어도 무방
    - 응용 계층 프로토콜로서 계층화 원칙 완벽 준수
  - DHCP의 직전 단계

따라서 원격 부팅 과정을 전반적으로 혼자 책임질 수 있는 BOOTP가 만들어졌다. RARP는 한계점 세가지

RARP가 해결해주는것은 IP주소 하나뿐, 부팅도 할줄모르고 OS image도 못가져오는. BOOTP는 굉장히 많은 파라미터를 configuration할 수 있는 프로토콜

링크 하나마다 RARP 서버가 하나씩 필요했다. 서브넷 하나마다 RARP 서버가 하나씩 박아놔야한다. BOOTP 서버는 원격 위치에 1대만 있어도 무방하다. 링크 하나마다 박아놓는 것이 아니다.

RARP 해킹- 할당해주는 작업 자체가 이더넷을 사용해서 요청하고 응답이 오면 인터페이스를 만들어서 IP를 붙이는 것이 커널에서 해야하는 일인데 OS image를 못가져온 상태이므로 해킹을 통해서 할 수 밖에 없었다. 정상적인 계층화원칙을 지키면서 allogant하게 하도록 해결해줬다. 따라서 BOOTP부터 응용계층 프로토콜이라고 할 수 있다.

BOOTP는 DHCP의 직전단계라고 할 수 있다.

DHCP의 장점

BOOTP의 장점 +a

200가지에 가까운 네트워크 파라미터 설정 가능

3 라우터 6도메인 서버 등등 굉장히 많다. 이정도로 파워풀한 configuration 프로토콜

DHCP Relay 개념 - 링크안에 DHCP 서버가 없다. relay agent가 있는데 이건 라우터이다. 이 라우터에다가 BOOTP relay라고 하는 요즘에는 DHCP relay라고 부르는 코드가 추가된다. 따라서 이일까지 해준다. 멀리 DHCP서버가 다들 멀리 있다. 미국에  DHCP 서버가 있어도 상관없다. 아주 멀리 relay가 가능하다. 각각 다른 라우터들이 사이사이에 있어도 상관없다. Relay가 있어도 좋은게 뭐냐면 DHCP 클라이언트는 마치 링크 안에 서버가 있는 것처럼 브로드 캐스트로 보내면 링크 안에 agent가  unicast로 원격에 있는 서버로 보내준다. 그리고 이에 대한 응답으로 서버도 해당 agent로 보내주면 그것을 다시 클라이언트로 보내주는 것

정상적인 계층화

BOOTP +a는 LEASE이다. 미국에는 lease라는 말을 자주 사용한다. 자동차 lease 등등 임대의 의미이다. IP주소에 해당하는 lease는 BOOTP에는 없는 거싱 IP주소의 자동 할당하고 회수하는 메커니즘이 없다. MAC-IP 매핑을 관리자가 수동으로 관리해야하는 BOOTP에서 진일보

임대주소 → 언제까지는 이건 니주소야 써. 한다음에 시간이 지나면 회수