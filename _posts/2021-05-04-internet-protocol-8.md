---
layout: post
title: "[김효곤 교수의 인터넷 프로토콜] 8. Internet Control Message Protocol(ICMP)"
categories:
  - Internet Protocol
tags:
  - 인터넷, 프로토콜, 그리고 계층화 원칙

---

# Internet Control Message Protocol(ICMP)

### 8.0 ICMP 개관

- 분류
  - 기술 차이를 극복하는 데이터 배달(IP), 라우팅과 함께, 역할로는 3계층 프로토콜
    - 기능적으로는 3계층
  -  IP에 실려간다는 형식적 측면에서는 4계층 프로토콜
    - IP에 실려가는 프로토콜은 대부분 4계층
- 역할
  - IP 데이터그램 배달 실패(error)에 대한 보고: 패킷이 폐기되는 라우터에서 발송자(sender)에게 원인을 알려줌
    - IP 프로토콜 번호 = 1 (ICMP가 IP 통신에서 얼마나 중요한 역할인지 알 수 있음)
  - 진단(diagnosis) 및 정보(information) 획득 - 보조적인 역할(IPv6에서는 중요)
    - Windows에서
      Ping(상대방이 살아있는지 진단), tracert(내 패킷이 어디를 거쳐가는지에 대한 정보 획득)
    - Default 라우터 주소 등 -> 현대에는 DHCP가 역할 대체
      - ICMP의 진단 및 정보 획득 역할 축소
      - IPV6 에서는 다시 ICMP의 역할이 확장

### 8.1 ICMP 메시지 형식

- 헤더는 8byte
  - Additional header field or Unused (4byte) : Type, Code에 표시되는 패킷의 역할에 따라 바뀜
  - checksum : 익히 아는 internet checksum을 의미

- 타입 - 대분류, 코드 - 소분류 : 2단계 분류

![]({{site.url}}/assets/images/104.png)

- 에러 메시지 타입 - IP에서 프로토콜 1번
  - 대표적인 것
    - type 11 : Time exceeded(TTL = 0)
    - type 3 :  Destination unreachable (목적지 도달 x,  구체적인 종류는 코드로 나눠짐)

![]({{site.url}}/assets/images/105.png)

- 정보 요청/응답 메시지 타입
  - Ping과 관련된 타입
    - type 0 : Echo reply - 핑 요청에 대한 응답
    - type 8 : Echo (request) - 핑을 보내는 것
  - 나머지 표에 보이는 것들은 IPv4에서 DHCP로 대체된다.
    - type 9, 10 : IPv6에서 다시 ICMP에서도 역할 복구

![]({{site.url}}/assets/images/106.png)

### 8.2 ICMP 에러 메시지

- IP 데이터그램이 목적지에 도착하지 못하고 버려지는 경우, 발송자에게 원인 보고
  - 발송자가 원인을 제거하도록
    - 발송자가 다음 번에 같은 이유로 패킷을 잘못 보내지 않도록 방지
    - 만약 발송자가 원인을 제거할 수 없는 상황이라면, ICMP 에러 메시지를 발송하지 않음
    - tracert도 이 ICMP를 이용하는 것으로, TTL을 적게 설정하여 어떤 라우터에서 죽는다면, 그 라우터는 발송자가 TTL을 더 높게 설정하라고 에러 메시지를 보내주는 것이다.
- 형식
  - ICMP 에러메시지의 payload에 버려지는 IP 데이터그램의 앞 부분을 잘라서 넣음
    - IP 헤더 + IP 페이로드 최소 8바이트
    - 출발/도착지의 IP 주소
    - 프로토콜과 출발지/도착지의 port numbers (= 발송 프로세스)
      - 프로토콜과 포트번호가 필요한 이유 : 에러의 원인이 되는 발송자를 정확히 적시하기 위함이다. 여기서 발송자는 출발지 기기의 IP 인터페이스가 아니라, 이 패킷을 내보내도록 한 응용 프로그램이다.

![]({{site.url}}/assets/images/107.png)

- ICMP 에러 메시지가 발생하지 않는 경우

  1. ICMP 에러 메시지를 싣고 가던 IP 데이터그램
     - 어떤 IP 패킷이 사망했고, 이에 대해서 발송지로 에러메시지가 보내지는 와중에 또 사망한 경우에는 에러 메시지에 대한 또 다른 에러 메시지를 보내지 않는다. 에러메시지가 무한으로 발송되는 경우를 방지하기 위해 어떤 IP 데이터그램에 대한 에러 메시지는 한번만 발송하는 것으로 한다.

  2. IP 헤더의 에러 때문에 체크섬 검사를 통해 버려지는 IP 데이터그램
     - 체크섬 검사로 인해 사망한 것은 소스의 잘못은 아니다. 체크섬 검사에서 문제가 발생한 경우에는 그 직전 링크에서 문제가 있었다는 뜻이다. 그 전까지는 문제가 없었기 때문에 그 직전 링크까지는 정상적으로 도달한 것이다.

  3. IP 멀티캐스트 주소나 브로드캐스트 주소로 보내지던 IP 데이터그램
     - 목적지가 너무 많고, 어떤 브랜치에서 에러가 발생하면 보통은 IP 데이터그램 자체에 문제가 있으므로 여러개의 브랜치에서 에러가 다발적으로 발생할 것이다. 소스에서는 IP 데이터그램 한개만 보냈는데 수십, 수백 개의 에러메시지가 도착할 수도 있으므로(이것을 브로드캐스트 스톰이라고 부른다.), 이것을 방지하기 위해 멀티나 브로드캐스트 주소로 보내지던 IP 데이터그램에 대해 에러 메시지는 발송되지 않는다.

  4. 출발지 IP 주소가 0.0.0.0처럼 반송 주소로 쓰기 부적합한 경우
     - 소스를 특정할 수 없는 경우라면, 누군지 모르므로 보낼 수가 없다. 0.0.0.0 = '나'라는 대명사
  5. 분할된 IP 데이터그램 중 첫 fragment가 아닌 것
     - 헤더야 모든 fragment마다 붙어있겠지만 페이로드의 앞부분은 첫 fragment에만 있을 것이다.
     - 첫 fragment인지 여부는 offset을 통해 알 수 있다. 즉, offset=0이 아니라면 에러 메시지를 보내지 않는다.

- Destination Unreachable 에러 : type 3

  ![]({{site.url}}/assets/images/122.png)

  - code 0~4까지의 에러가 가장 자주 발생
  - 0 - Network Unreachable : core router는 전세계의 거의 모든 prefix를 알고 있는데, 이 core router가 도착지로 보내진 prefix를 모를 때, 즉, 유효한 네트워크 아이디가 아닌 IP주소를 도착지로 하고 있을 때 발생하는 에러이다.
  - 1 - Host Unreachable : 도착지의 네트워크 아이디는 유효한데, 호스트를 아무리 찾아도 라우팅 테이블에 없을 경우 발생하는 에러이다.
  - 2 - Protocol Unreachable : 네트워크 아이디부터 호스트까지 모두 유효한데, 프로토콜 넘버가 유효하지 않은 경우 발생하는 에러이다.
  - 3 - Port Unreachable : 도착지의 IP주소부터 프로토콜까지 모두 유효하지만 해당 포트 번호에 유효한 프로세스가 돌아가고 있지 않는 경우 발생하는 에러이다.
  - 4 - Fragmentation needed but don't-fragment bit set :  IP 데이터그램 분할이 일어날 때, DF=1로 설정되었고, MTU 사이즈가 IP 데이터그램 사이즈보다 작은 경우 발생하는 에러이다. 에러 메시지 페이로드에는 너무 패킷 사이즈가 커서 죽은 IP 데이터그램의 앞부분이 실려있을 것이다. 무엇보다 중요한 것은 이 때 ICMP 헤더에서 Additional header field 부분 4바이트에 MTU 사이즈가 들어간다는 점이다.
  - 그 밖에도 다양한 feature들이 있지만 11번처럼 실제로 인터넷에서 발생하지도 않는 feature들도 있고 실제로 보기 힘든 에러들이 대부분이다.

- Destination Unreachable 에러 - fragmentation needed but don't-fragment bit set

  ![]({{site.url}}/assets/images/124.png)

  ![]({{site.url}}/assets/images/125.png)

  ![]({{site.url}}/assets/images/126.png)

  ICMP 헤더에 MTU 사이즈가 1400으로 되어있는 것을 확인할 수 있다. 그 뒤로는 죽은 데이터그램의 앞부분이 따라온다.

- Redirect 에러 - type 5

  - IPv6에서는 에러가 아닌 정보(information) 메시지로 재분류

- 특수한 상황 (현실에 거의 존재하지 않는 에러)

  - 인터넷으로 연결된 라우터를 선택할 수 있는 상황
    - 에러가 나려면, 하나의 말단 호스트에 두 개의 라우터가 연결되어야 하는데, 보통 대부분의 호스트들은 하나의 라우터와만 연결되어 있다.
  - 특정 목적지("D")는 그 중 하나의 라우터("R2")가 더 가까우나 default route의 next hop은 다른 라우터("R1")인 경우
    - 일단 호스트는 R1쪽으로 패킷을 보냈으나, R1에서 라우팅 테이블 lookup을 해봤더니 D로 가려면 R2가 가까움을 알고 R2로 보낼 것이다. 그리고 R2는 정상적으로 D가 있는 쪽으로 패킷을 전달해줄 것이다.
    - 이 과정에서 R1의 입장에서는 들어온 인터페이스로 패킷이 다시 나간 것으로, 손님이 들어온 문으로 다신 나간 셈이 된다. 이러한 불필요한 컨디션을 R1은 detact를 하고 이 데이터그램을 보낸 S에게 라우팅 테이블에서 D에 관해서는 next hop을 R2로 바꾸라는 에러 메시지를 보낸다. 이 에러메시지를 받은 S는 라우팅 테이블에 D->R2 엔트리를 추가한다. 이 엔트리의 서브넷 마스크는 길이가 0.0.0.0보다는 길 것이다.
    - 사실 패킷은 폐기되지 않고 정상 전달이 되었었기 때문에 R1는 exception을 핸들링한 것이 아니다.

![]({{site.url}}/assets/images/123.png)

> Q : 혹시 fragmentation 하다가 port 번호가 잘못 나눠지는 경우는 없을까요?
> A : 포트번호는 잘못 나눠질 수가 없는 형태로, 4바이트로 잘 정렬되어있다. 포트번호까지 잘라지려면, framentation이 아주 잘게 잘라져야한다. 근데 그럴 가능성은 전혀 없다.

- Time Exceeded 에러 - Type 11
  - 목적지 도착 전 TTL=0 (code 0)
  - 목적지 도착 후 일부 fragment 미도착 (code 1)
    - 재조립은 항상 최종 목적지에서 일어나는데, 목적지에 일부 frament가 도착하지 못한 상황
    - 표준 timeout = 15초 (상당히 오랜 시간)
    - timeout이 지나고나면, 미도착한 일부 fragment만을 골라 재전송받을 수도 없고, 아예 fragment을 지원하지 않는 UDP 같은 프로토콜도 있기 때문에 이 조각을 다시 받을 수 있다고 기대를 할 수 없다.
    - 따라서 도착했던 조각들도 모두 폐기된다.

![]({{site.url}}/assets/images/127.png)

### 8.3 ICMP 정보 질의/응답 메시지

- 정보 질의/응답 메시지는 에러 메시지가 아니므로 앞에 언급했던 ICMP 규칙들은 다 여기서는 해당되지 않는다.
- Ping = ICMP Echo Request(Type8) + ICMP Echo Reply(Type0)
  - 이 외의 용도의 ICMP 정보 메시지는 IPv4에서 거의 쓰지 않음
  - DHCP가 대부분의 역할 대체
- ICMP Echo Request 패킷 형식
  - Identifier : 프로세스별 고유 값 (Windows에서 미사용)
    - Ping 프로그램을 여러개 돌리는 상황에서 이 프로그램들 중 누구에게로 갈 것이냐는 결정에 도움을 주기 위한 필드
    - Windows는 그럼 어떻게 알아보느냐? 해킹 등의 다양한 방법으로 언제든지 가능은 하다. 구체적으로 배울 필요는 없다.
  - Sequence number : 전송자에서 질의와 응답을 match 시키기 위한 값
    - `ping www.korea.ac.kr` 이런식으로 명령어를 입력하면 해당 IP주소로 가는데 얼마나 걸리는지 알아낼 수가 있다. 이 시간차를 알아내려면 짝을 잘 맞춰야 한다. 1번 메시지가 발송된 시간과 1번에 해당하는 응답이 다시 돌아오는 시간을 비교해야 한다. 1번 메시지에 대하여 2번 메시지에 대한 응답과의 시간 차를 계산하면 안된다. 이처럼 정확한 시간차 계산을 위해 헤더에서 메시지의 번호를 명시할 수 있도록 Sequence number 필드가 포함된다.
    - 와이어 샤크로도 확인이 가능하다. 시퀀스 번호가  BE(Big Endian), LE(Little Endian) 두 형태로 보여지는데, 컴퓨터의 바이트를 해석하는 방법에 따라 같은 숫자라도 두 방식으로 와이어 샤크가 친절히 해석해주고 있는 것이다.
    - 시퀀스 넘버는 패킷마다 하나씩 늘어날 것이다. 119, 120, 121...
  - Payload에는 별의미 없는 쓰레기 값이 들어가기도 한다. 와이어샤크에서 확인해보면, 61626364...이런 숫자가 보이는데, 그냥 ABCD...이런 별 의미 없는 문자들이 들어간 것이다.

![]({{site.url}}/assets/images/128.png)

### 8.4 ICMPv6

- 역할 확대
  - IPv4에서 사용하던 ARP와 IGMP의 역할 대체
    - ARP와 IGMP가 IPv6에서 사라졌으나 기능 자체는 필요하기 때문에 이것을 ICMP가 대신하는 것
  - 대체적인 틀 거의 유지
    - 에러와 정보 요청/응답 => DHCP에게 빼앗겼던 일부 역할도 일부 회복
- 주목할 만한 차이점
  - IP fragmentation이 사라지면서 MTU보다 큰 데이터그램 무조건 버림 -> Packet Too Big 에러로 이름이 바뀜
    - Don't fragment라는 말이 사라졌음 <- 라우터에서의 framentation을 IPv6에서는 지원해주지 않기 때문이다. 원래도 안하는데, 하지말라고 할 필요 조차 없는 것이다.
    - Ipv6에서는 MTU보다 패킷 사이즈가 크면 무조건 폐기하며, Fragmentation을 절대 안해준다.
    - 멀티 캐스트도 보내줌 (IPv4에서 브로드캐스트나 멀티캐스트에 대해서는 ICMP 에러 메시지를 발송하지 않는 것과 차이가 있다. - 브로드 캐스트 스톰 문제로)
      - 멀티캐스트로 보낸 패킷에 대하여 Packet Too Big 에러가 발생하면 에러를 발송해준다. (IPv4에서는 안된다.)
  - ICMP 정보 요청/응답을 Neighbor Discovery(ND)에 활발히 사용
    - ND의 예시
      - ARP의 역할 : 도착지의 IP주소에 대하여 MAC주소를 알아내기
      - 나의 디폴트 라우터는 누구인지 알아내기
    - 그중에서 ICMP는 ARP 대체 역할을 대체

- Neighbor Discovery

  - Neighbor Solicitation(NS) = (구) ARP 요청

    - 너의 MAC 주소를 달라!
    - 수신자: SNM 주소 사용 (라우팅 look up을 통해서 찾아낸 next hop의 SNM 주소 (또다른 IP멀티캐스트 주소))
    - 발신자: unspecified 주소 (`::`) -> 응답은 All-Nodes Multicast로 줄 수 밖에 없는 경우 (IPv6에는 broadcast가 없다.)

    ![]({{site.url}}/assets/images/131.png)

    여기서는 next-hop의 SNM 주소가 IP 메시지의 도착지 주소인 `ff02::1:ffde:fa64`이고, ICMP 헤더에 들어간 Target Address `fe80:b59f:89ae:13de:fa64`가 next-hop의 IP주소인 것이다. 그리고 더 나아가서는 이더넷 메시지의 도착지로 되어있는 것은 SNM 주소를 MAC주소로 변환한 것이 들어가있을 것이다.

  - Neighbor Advertisement(NA) = (구) ARP 응답

    - 그런데 ND에는 flag라는 세 개의 필드들이 존재한다.
    - S(solicited) = 1 : 이 메시지가 NS에 대한 응답임을 나타낸다.
    - S = 0 : 자발적으로 = (구) GARP
      - IP 주소가 바꼈을 시, 예전 IP주소를 갖고 있을 지도 모르는 이웃들에게 변화를 통보
      - R : 라우터가 보낸 것
      - O - override : 이 flag를 켜놔야만 캐쉬 엔트리를 update하라는 요청의 뜻이 된다.

    ![]({{site.url}}/assets/images/129.png)

  - ARP와 달리 

    - (1) target은 target 
      - 응답할 때에도 자기 자신을 sender가 아닌 target이라도 지칭
      - ICMPv6 Option에다가는 요청자가 요청했던 자신의 MAC주소를 올린다. 그림에서는 `e4:a4:71:85:17:db`
      - 또한 아까 요청할 때 sender IP 주소로 unspecified 주소를 보냈기 때문에 ANM에 해당하는 `ff02::1` 주소로 패킷을 보내고 있다.
    - (2) "응답"으로 제 3자 매핑 update

    ![]({{site.url}}/assets/images/130.png)

> Q : all nodes multicast는 거의 IPv4의 broadcast와 동일하게 작동된다고 보면 되는건가요?
> A : 그렇다. 그런데 all nodes라고 할때 앞에 Multicast capable interface란 말이 붙는다. 멀티캐스트가 되는 애들만 보내준다. 그러나 요즘에는 멀티캐스트를 지원하지 않는 애들이 따로 있지는 않다. 이럴거면 브로드 캐스트를 왜 없앴냐면, 일단 멀티캐스트로 통일을 해놓고 브로드캐스트로 하던것도 SNM주소를 기준으로 들어야할 애들의 범위를 정할 수 있게 되었기 때문이다.
>
> Q : IPv6 애니캐스트 메시지에 대해서는 ICMP를 보내주지 않는 것인가요? 
> A : 애니캐스트와 유니캐스트는 IP 주소를 전혀 구별할 수 없으므로 호스트에서는 이를 구별할 수가 없어 ICMP 메시지를 보낼 것이다. 용도상 애니캐스트를 만들었기 때문에 ICMP 메시지를 보내느냐 안보내느냐의 기준과는 상관이 없다.