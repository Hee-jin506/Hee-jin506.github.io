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
      - DHCP 클라이언트는 마치 링크 안에 서버가 있는 것처럼 브로드캐스트로 DHCP를 요청하면 보내면 링크 안에 존재하는 agent가 unicast로 원격에 있는 DHCP 서버까지 보내준다. 그리고 서버가 이에 대한 응답으로 해당 agent로 보내주면 그것을 다시 클라이언트로 보내주는 것이다.

    ![]({{site.url}}/assets/images/142.png)

  - 정상적인 계층화

    - UDP를 트랜스포트로 사용하는 응용 계층 프로토콜
      - 서버 포트 = 67
      - 클라이언트 포트 = 68


- a (BOOTP에는 없는 것) = “LEASE” = "임대"

  - LEASE: IP 주소의 **자동 할당** 및 **회수** 메커니즘
    - **임대 기간**이 존재하며, 이 임대 기간이 지나면 **회수**됨
    - **MAC-IP 매핑을** **관리자가 수동으로 관리**해야 하는 BOOTP에서 진일보
  - 우리가 흔히 사용하는 인터넷 공유기는 DHCP 서버 역할을 한다. KT나 SK 브랜드에 가입하면 셋톱박스를 나눠주는데, 이것이 DHCP 서버를 갖고 잇어서 우리에게 IP주소를 LEASE를 해준다. 이것을 확인하고 싶다면 `ip config` 명령어를 입력해보면 된다. 무선랜 인터넷을 사용하고 있다는 정보와 함께 다음과 같은 정보를 확인할 수 있다.
    - DHCP 사용 : 예
    - 임대 시작 날짜 / 임대 만료 날짜
      - 임대 시작 날짜는 수업 당일 날짜 오전으로 교수님이 이 시간에 컴퓨터를 켰음을 짐작할 수 있다.
      - 임대의 시작 날짜와 만료 날짜를 비교해보면 DHCP 서버에서 임대 기간을 두시간으로 정해줬음을 알 수 있다.

### 10.3 DHCP 메시지 형식

![]({{site.url}}/assets/images/160.png)

- OP code / Hardware Type / Hardware Address Length : ARP  형식 메시지에서 많이 본 필드
  - Operation code : 요청(1)인지 응답(2)인지
  - Hardware Type : 어떤 데이터링크 기술인지
    - Ethernet = 1
  - Hardware Address Length: 특정 데이터링크 기술에서의 주소의 길이인데 맥주소면 6바이트니까 보통은 6이라는 값을 집어넣었었다.
    - Ethernet = 6
  - "프로토콜"은 따로 없음
    - 당연히 IPv4를 가정
- Hops : 몇개의 DHCP relay agent를 거쳤는지를 나타낸다. **16 **홉을 초과하면 Loop에 빠졌을 가능성을 고려하여 더 이상 보내지 않고, 폐기되도록 되어있다. 
- Transaction Identifier : UDP를 트랜스포트로 사용하는 스탠다드 프로토콜에는 포함되는 필드이다. UDP는 IP에 실어나르는 데이터그램 방식이므로, 요청을 여러개 날리고 그 요청들에 대한 응답을 받을 때 순서가 뒤죽박죽일 수 있다. 따라서 보낸것과 받은 것들 사이에 짝짓기를 위해서 사용되는 이 필드가 이 필드이다. 이름은 프로토콜마다 조금씩 다를 수 있다.
  - 스탠다드 프로토콜 : 최근에 짜여진 응용을 위한 응용 프로토콜들 이외에, DHCP나 DNS같이 infrastructure용으로 들어가는 응용 계층 프로토콜
  - 이 필드 값은 송신자가 보낸 그대로 수신자가 echo를 해준다. 송신자가 자기가 보낸 요청 메시지 번호 리스트 중에서 어떤 것에 해당하는지 확인할 수 있도록.
- Seconds : 처음으로 요청했을 때부터 흐른 시간으로, DHCP 서버에서 클라이언트들을 기다린 시간 순으로 줄 세울 때 사용된다. 0이면 처음 요청한 것이고, 3이면 3초가 지난 것으로 응답을 보내지 않아 클라이언트측에서 재요청을 했다는 뜻이다. 따라서 DHCP 서버는 비교적 많이 기다린 클라이언트 순으로 응답을 해준다.
- Flags : 굉장히 큰 필드인데 이 중에서 단 1이라는 한 비트만이 브로드캐스트로 현재 정의가 되어있다. 요청을 보낼 때 클라이언트에 따라서는 유니캐스트로 응답을 받을 수 없는 호스트가 있을 수 있다. 이 때 클라이언트는 Flag를 1(브로드캐스트)로 설정을 하여 브로드캐스트로 대신 응답을 해달라고 요청할 수가 있다.
- **Client Hardware** Address : Address 필드 중 가장 중요한 필드로, 위에서 언급한 Hardware Type/Length에 해당하는 값이다. 위의 필드를 기준으로 이 필드의 길이가 맞춰져 있을 것이다. 자체의 크기는 16바이트이므로, 남는 부분은 패딩을 붙이거나 할 것이다. 이게 DHCP를 통해 IP를 할당받기 위한 key가 된다. 이 필드는 Options 필드와 관계를 이루어 DHCP 서버에서 IP주소를 할당할 때 사용된다.
- **"Your" IP Address** : DHCP 서버에서 클라이언트에게 할당해준 IP 주소로 "보통은" CHAddr와 매핑된다. 가끔은 Option에서 DNS 도메인 이름 같은 것으로 매핑할 수도 있다. 원론적으로는 Options 필드와 매핑되는 것이지만 일반적인 상황에서는 Options 필드에 CHAddr와 동일한 값이 들어가기 때문에 CHAddr와 매핑된다고 할 수 있다.
- Client IP Address : IP주소를 아직 할당받지 못했을 때에는 값이 그냥 0으로 채워진다. 그런데 DHCP/BOOTP에서 IP주소 이외에 다양한 파라미터 정보를 얻을 수도 있기 때문에 그런데 클라이언트가 이미 IP주소를 받은 후에도 또 다른 정보를 목적으로 요청을 보낸 경우 자신의 IP주소를 이 필드에 적는다.
- Server IP Address : DHCP 서버 IP 주소가 아니라, RARP/BOOTP처럼 DHCP의 연원을 거슬러 올라가면 결국 부팅을 위한 과정이었는데, 서버에 접촉해서 모든 걸 해결하지 못할 경우, 부팅을 도와주는 다음 서버를 의미한다. 즉, Next Server Contact으로, 호스트가 DHCP 서버와 대화가 끝나고 나면 다음으로 접촉할 서버(ex - OS Image Server)의 IP주소가 들어간다.
  - 없으면 0.0.0.0
- Gateway IP Address : 서버에서 클라이언트에게 알려주는 relay 주소이다. 요청 메시지가 요청으로 서버까지 도달했다가 응답으로 돌아오는 경로 중에서 클라이언트와 가장 가까이 있는 relay agent의 주소가 여기에 들어간다.
- Server Name, Boot Filename : 이 두 개의 file들은 원격 부팅을 할 때 사용되는 파일들이지만 요즘에는 원격 부팅을 안하므로 거의 안 쓰인다.
  - 모두 0
- <u>**Options 필드**</u> 
  - 최대 200 개의 설정 가능한 네트워크 파라미터가 여기 실림
    1. IP 네트워킹을 위한 호스트 파라미터 (3계층)
    2. IP 네트워킹을 위한 인터페이스 파라미터 (3계층)
       - 호스트가 가진 인터페이스 각각에 대한 파라미터
       - ex) IP주소
    3. 링크 계층 파라미터 (2계층)
    4. TCP 파라미터 (4계층)
    5. 응용 및 서비스 관련 파라미터 (5계층)
    6. **DHCP 작동 자체를 위한 파라미터**

- DHCP 메시지 예

![]({{site.url}}/assets/images/161.png)

- IP - UDP - Src Port : 67 / Dst Port : 68
- Message Type => OP code
  - Message Type 이란 말보다는 Op code란 단어가 더 정확하다. 실상 DHCP 메시지 타입은 Options 필드에 나오며, 이 필드는 방향성만 제시한다.
- Bootp란 말이 메시지에서 등장할 수가 있는데(BOOTP flags), 아직 업데이트가 안 된 것이며 그만큼 BOOTP와 DHCP가 비슷함을 의미한다. LEASE라는 개념을 제외하고는 큰 차이가 없다.
- Hardware Type : Ehternet(1)
- Hardware Address Type : 6
- Hops : 0
  - Relay 없이 바로 DHCP 서버로 요청되고 응답되었음을 의미하며 내 링크 위에 DHCP 서버가 있다는 뜻이다. 이것은 인터넷 공유기를 사용한 케이스
- Transaction ID : 서버에 의해 echo된 값
- BOOTP Flags : 0
  - 유니캐스트로 응답을 보내도 클라이언트가 처리할 수 있다고 요청 메시지에 써져있었을 것
- Client IP Address : 0
  - 아직 클라이언트가 자신의 IP주소를 못받은 상태에서 보낸 요청에 대한 응답

> **클라이언트는 자신의 IP 주소도 못 받은 상태에서 어떻게 DHCP 서버에서 보낸 유니캐스트 응답 메시지가 자신에게로 온 것임을 알 수 있을까?**
>
> 호스트의 DHCP 클라이언트 구현 방식마다 다르긴 하지만, 쨌든 호스트에서 받아서 처리할 수 있도록 구현이 되어있다. 이 구현이 안된 호스트라면 어쩔 수 없이 브로드캐스트로 응답을 받아야 한다.

- Your IP Address : 172.30.1.44
  - DHCP 서버가 할당한 IP 주소
- Client MAC Address
  - DHCP가 이것을 key로 하여 IP 주소를 할당해주는데, 실은 이 값을 보고서 할당하는 것은 아니고 DB에다가 IP주소와 MAC주소의 매핑 정보를 적을 때 사용된다.
    - 매핑 정보는 LEASE 임대 기간동안 유효하다.
- Server Host Name / Boot File Name : 안 쓰임
- Option 필드들
  - DHCP Message Type : 이게 진짜 메시지 타입
  - DHCP Server Identifier : 이게 DHCP 서버의 IP 주소
  - Lease Time : 이것이 LEASE 임대기간이며 캡쳐 이미지에는 값이 명시되어있진 않음
  - Subnet Mask : 클라이언트가 사용해야할 서브넷 마스크
  - Router : 디폴트 라우터(호스트 라우팅 테이블에 들어갈 디폴트 엔트리)
  - Domain Name Server : 클라이언트가 사용해야할 DNS 서버
  - 나머지는 Padding으로 채움

### 10.4 DHCP 작동을 위한 옵션

- DHCP Message Type : 사실상 옵션이 아니라 필수임
  - 표에 보이는 것들 말고도 더 많은 옵션들이 존재함

- 50 : 클라이언트쪽에서 선호하는 IP 주소를 여기에 적어줌
  - 보통은 임대 기간의 50프로가 지나간 시점에서 클라이언트가 여태 쓰던 IP 주소를 적어보내며 이어 사용하겠다는 의향을 보일 때 사용

- 51 : 서버에서 설정하는 LEASE 임대 기간

  - DHCP 서버의 상황에 따라 이 시간은 달라질 수 있다. 예를 들어, 클라이언트는 아주 많고, 이에 비해 IP주소는 몇 개 없다면 LEASE TIME을 짧게 설정한다. 클라이언트는 IP주소를 더이상 사용하지 않을 때에 IP주소를 반납하겠다는 메시지를 보내지 않아도 임대 기간이 만료되기만 하면 알아서 회수된다. 따라서 DHCP 서버는 클라이언트 수에 비해 IP주소가 적을수록 임대를 연장하지 않는 호스트들의 IP주소를 빨리 회수하기 위해 짧게 임대기간을 설정한다.

- 53 : 필수적인 옵션

  ![]({{site.url}}/assets/images/167.png)

- 54 : DHCP 서버의 IP주소

  - =/= Server IP address
  - 클라이언트가 처음 요청할 때는 DHCP 서버가 어딨는지 모르기 때문에 브로드캐스트를 한다. 이때 DHCP 서버가 혹은 서브넷 라우터(Relay Agent)가 요청을 받아서 IP주소를 받아다줄 것이다. 그리고 응답을 클라이언트가 받고서 DHCP 서버의 IP주소를 이 필드를 통해 알게 되면, relay를 거칠 브로드캐스트 요청 없이 바로 DHCP 서버를 목적지로 유니캐스트를 보낼 수가 있게 된다. 이 때 경로가 처음 요청 메시지가 거쳐간 경로와 같을 가능성도 있지만 쨌든 목적지를 알고 유니캐스트로 보낼 수 있다는 점에서 다르다.
    - 아래 그림에서 1-5까지는 처음으로 DHCP와 대화를 나눈 것으로 목적지가 limited broadcast(255.255.255.255)로 설정되어있다.
    - 그이후로부터 10번 request의 목적지는 브로드캐스트가 아니라 DHCP 서버의 IP주소(172.30.1.254)가 직접 설정되어있다. 

  ![]({{site.url}}/assets/images/165.png)

  - Server Identifier의 활용 예 2
    - 클라이언트가 여러 DHCP 서버 중 하나를 선택할 때
      - DHCP ACK 참고

  - IP 주소를 할당받은 클라이언트는 DAD를 시행한다
    - DHCP 서버가 한 IP주소를 여러 인터페이스에 잘못 임대했을 경우를 대비해서 받았으면 DAD를 세 번 해본다. 

- 55 : 클라이언트에서 요구하는 파라미터들의 리스트

  - DHCP 서버에서 이 모든 파라미터들을 가르쳐줄 의무는 없으며, 심지어는 counter offer를 할 수도 있다.
    - counter offer : "클라이언트야. 너 이거 요청헀는데, 이거 써보지 않을래?"하며 다른 값을 역제안

![]({{site.url}}/assets/images/166.png)

- 58 : 클라이언트에서 IP주소의 임대를 다시 리뉴얼, 연장해야 하는 시점
  - 50%
- 59 : 클라이언트에서 임대를 연장해달라는 요청을 보냈지만 응답이 없을 때, 다시 재요청(rebinding)하는 시점
  - 87.5% (7/8)
- 61 : 보통은 클라이언트의 MAC 주소와 같지만, DNS 주소일 수도 있다.

![]({{site.url}}/assets/images/162.png)

- 요청 옵션 예시

  ![]({{site.url}}/assets/images/163.png)

- 응답 옵션 예시

  ![]({{site.url}}/assets/images/164.png)

> Q : relay 경로를 결정하는 기준이 있을까요?
> A : 보통은 네트워크 어드민이 설정한 경로를 따라간다. 사실 그렇게 많이 relay로 거쳐갈 필요도 별로 없어서 경로를 굳이 길게 설정하지 않는다. 

### 10.5 DHCP 프로토콜 작동

1. Discover : 서버나 Relay Agent를 찾는다

   - IP 주소를 모르므로 **limited broadcast**
   - 클라이언트에서 원하는 파라미터 값을 제시할 수 있다

   ![]({{site.url}}/assets/images/169.png)

   - 이 예시에서 보면 Options : Request IP Address가 정해져있음을 알 수 있으며 클라이언트 쪽엣 먼저 이 IP주소로 임대를 연장해주었으면 하는 의향을 보이고 있다.
   - Src IP Addr : 0.0.0.0
     - 아직 IP주소 할당받지 못한 상태
   - Dst IP Addr : 255.255.255.255
     - Limited Broadcast

2. Offer : 서버가 파라미터들을 제시한다 (응답)

   - 선택적으로 Offer할 가능성 + Counter Offer의 가능성

   - 다수의 서버가 있을 수도 있다
     - 그 중에 하나를 클라이언트가 지목해야 한다. (Server Identifier의 활용 예 2)

3. **Request : 클라이언트가 원하는 서버를 지목**

   - 동시에 파라미터 값에 대한 counter-offer 가능
     - 서버에서 온 Offer가 마음에 안들 시
   - **거절되는 서버**도 알 수 있도록 **broadcast**한다

4. Ack: 서버가 조건을 수락한다
   - 이때부터 클라이언트가 IP주소를 쓸 수 있게 된다.

5. 클라이언트는 DAD를 수행한다
   - 재량껏 클라이언트가 IP주소의 유효성을 확인해보는 것

![]({{site.url}}/assets/images/168.png)

- 양자가 **대등한 negotiation**을 수행
  - DHCP **Request를 수용**할 수 없을 때 서버는 **DHCP Nack(Negative Ack)**
  - **DAD에서 실패**할 때 클라이언트는 **DHCP Decline**
  - 위의 두가지 경우에서 클라이언트는 전에 DHCP Offer를 했던 다른 서버에게 DHCP Request 가능 (Discover - Offer 단계 생략)

> **양측의 거절 방법**
>
> - DHCP 서버 : DHCP Nack
> - 클라이언트 : DHCP Deline

- 서버도 다른 서버가 lease한 주소를 임대하지 않기 위해
  - 정상적인 상황에서는 이러한 일이 발생하면 안되지만 만에 하나 서버들끼리 IP주소 임대에 대한 조율이 덜 되었을 경우
  - IP 주소 임대 직전에 Offer할 주소에 **ping**을 해 본다
- 쓰던 **IP 주소 반환**을 위한 **DHCP Release**
  - **Ipconfig/release**로 시도해 볼 것
    - Release 없이 그냥 클라이언트가 가버려도 타임아웃이 되어 IP주소는 자동 회수된다.
    - **Ipconfig/renew**로 인터넷 연결 복구 가능: **Discover-Offer-Request-Ack** 구동
      - 만약 release 하지 않은 상태에서 renew를 하면 **Request-Ack**만 다시 하게 되므로 임대시간이 refresh될 것이다.
- **IP 주소 획득 이후** 기타 파라미터 설정을 위한 **DHCP Inform**

### 10.6 IP 주소 임대

- Linux 설정 예
  - 아래 그림은 config file 중 일부

![]({{site.url}}/assets/images/170.png)

- DHCP에 의한 IP 주소 임대는 사설 주소와 상관 없다!!
  - DHCP는 할당되는 IP주소들이 사설인지 global인지 구분을 하지 않는다.
  - 위와 같이 착각하는 이유는 대부분 사설 주소를 갖고 DHCP를 운영하기 때문임
  - 주소 종류에 관계 없이 관리해 준다

- 나는 DHCP가 준 주소를 사용하고 있는가?
  - 다음 네트워크 설정 창에서 `자동으로 IP 주소 받기`과 `자동으로 DNS 서버 주소 사용`에 체크가 되어있어야 한다.
  - 이를 체크하지 않고 DNS를 사용하지 않게 되면 사용자가 직접 아래의 값을 명시해야 한다. 이 때에는 DHCP 프로토콜과는 관계 없이 작동된다.

![]({{site.url}}/assets/images/171.png)

### 10.6 IP 주소 임대

- 임대 기간 50% 경과 시 자동으로 임대 연장 시도
  - 실패시 87.5%에 다시 한 번 시도

![]({{site.url}}/assets/images/172.png)

- 위의 예시에서 `Discover`-`Offer`-`Request`-`ACK` 과정을 통해서 IP주소 임대가 완료되었고 그 이후로부터는 `Requset`-`ACK` 과정만 반복되면서 확인만 받고 있음을 알 수 있다.
  - Transaction ID가 같은 것을 통해서 위의 네 단계와 다음 두 단계들이 각각 한 묶음임을 알 수 있다.
- 110초 시점에 `Discover`-`Offer`-`Request`-`ACK` 과정
- 1911초, 3711초, 5510초 시점에 계속해서 `Requset`-`ACK`
- => 1800초 단위로 임대 연장이 발생 중이므로 임대 기간이 1시간임을 짐작할 수 있음

### 10.7 자동 구성

- 고유 명사: autoconfiguration (=/= DHCP 사용)

  - IP 주소를 인터넷에 할당을 해주는 방법 2가지
    - 사용자가 직접 IP 주소 할당
    - DHCP를 사용
  - 둘 다 사용되지 않을 경우의 궁여지책
    - 대체 무슨 상황? 예를 들어 치과에서 몇대의 컴퓨터를 서로 연결해서 사용하고 싶으나 인터넷에 연결을 할 필요가 없다. 그치만 그렇다고 사용자가 직접 IP 주소를 할당해주는 방법도 모르고, DHCP 서버도 없는 상황

  - 인터넷 연결 불가, **동일 link에서만** **IP 네트워킹** 가능
  - Windows에서 APIPA(Automatic Private IP Addressing)로 부름
    - 169.254.0.0/16에서 무작위 선정하여 인터페이스 할당
      - 커널이 마음대로 난수를 돌려서 주소를 정한다. 
    - DAD 필수
      - 하필이면 같은 링크에 있는 다른 호스트가 같은 주소로 설정했을 (아주 낮은)가능성을 대비
      - ACD이든 GARP이든 응답이 없으면 확정하고 ip 주소를 사용한다.
  - 조금 옛날 컴퓨터라면 Ipconfig를 했을 때 아래 그림처럼 자동 구성 IPv4 주소가 명시되어있을 수도 있다. 이 주소로 같은 링크에 있는 호스트들끼리 연결이 가능하다.
    - 자동 구성 주소가 안보인다면 `ipconfig/release`를 하고 나서 `ipconfig/all`를 해보면 보일 것이다.

![]({{site.url}}/assets/images/173.png)
