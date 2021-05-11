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
- 형식
  - ICMP 에러메시지의 payload에 버려지는 IP 데이터그램의 앞 부분을 잘라서 넣음
    - IP 헤더 + IP 페이로드 최소 8바이트: IP 주소 및 port numbers (= 발송 프로세스)

![]({{site.url}}/assets/images/107.png)

발송자가 원인을 제거할 수 없는 상태라면 패킷을 보내지 않는다. 그런식으로  IP데이터그램이 죽는 경우도 허다하다.

원인을 제거할 수 있는 상태에서만 패킷을 보내준다.

페이로드에 어떤것이 담기느냐 공통적으로 얘기할 수 있는 것은 버려지는 IP데이터그램의 앞부분을 잘라서 넣는다.

죽인원인을 소스에게 보고해야할 때에는 ICMP 페이로드에 IP데이터그램의 앞부분을 집어넣는것

밑의 그림에서 파란부분을 보면 IP데이터그램이 2개나오는 것처럼 보이는데, 두번째는 시체인 것이다. 시체의 헤더와 싣고가는 페이로드 8바이트를 집어넣는다. IP페이로드는 프로토콜, 출발지와 도착지의 포트번호들이 들어간다.

요 패킷을 보내도록한 응용프로그램이 발송자이다. 이 패킷의 사망의 원인이 되는 것이 이 응용프로그램이다. 범인 색출을 위한 정보가 바로 포트 번호과 프로토콜인 것이다.

ICMP 에러메시지가 발생한 경우에 취해야하는 형식이고 발생하지 않는 경우에는?

패킷이 사망했고, ICMPㅇ에러메시지가 보내지고 있는데 이 IP데이터그램이 도중에 죽은 것

이 경우에는?

이때 첫번째 에러가 발생한 곳으로 또다른 에러메시지를 만들어보내는가? 안보낸다! 보내면 계속 에러메시지가 무한으로 만들어질 가능성이 있기 때문

2번은 못들었다.

멀티캐스트나 브로드 캐스트로 보내지는 IP데이터그램

3번도 못들음

0.0.0.0 출발지 주소로부터 온 경우라면 소스가 누군지를 알지 못하므로, 이런 경우에는 에러 못보낸다.

IP  fragmentation이 일어나게 된다면 사망한 데이터그램의 앞부분은 잘라 보내야하는데, 헤더는 fragment마다 붙으므로 괜찮지만 페이로드 앞부분(포트번호나 프로토콜 정보)은 첫 fragment에만 있을 것이다.

따라서 offset=0이 아니면 안보내진다.

가장 자주보게 되는 에러 메시지 destination unreachable

가장 흔하게 일어나는 것은 0-4

0- 네트워크 언리처블 - 163.152.x.x 고려대 네트워크 코어라우터가 이 네트워크를 찾는데 없는 prefix인 것이다.네트워크 아이디가 아닌 네트워크 아이디를 목적지 주소에 달고 가는 패킷이 있어서 이런 네트워크 아이디는 없는데?하면서 죽여버릴때

1- 네트워크 아이디는 있으나, 호스트가 없는 것

2-호스트까지 모두 존재하나, 이상한 프로토콜 번호를 갖고온 경우 그 호스트에는 이 프로토콜이 안돌아간다 이런 경우

3-2번까지는 모두 괜찮은데 해당 포트에는 아무런 프로세스도 안돌아가고 있는데?

4-  DF =1로 해서 내보냈는데 MTU <IP데이터그램 사이즈  분할시키지말고 그냥 패킷을 죽인 후, 에러메시지 반송하는데 헤더 부분의 그 밑 4바이트에서는 MTU사이즈가 들어간다. 이를 발송지에서 확인하고 이에 맞게 패킷을 줄인다.

나머지는 실제로 되지 않는 에러 메시지가 잘보지 못하는 메시지가 대부분

destination unreachable 에러 (X) redirect 타입 5번 (O) → 피피티 내용 바꿔라

현실에 거의 존재하지 않는 에러

호스트가 있는데 자기 서브넷에 라우터가 한개가 아니라 두개이다.(보통은 한개이다) 아주 특수하게 두개의 라우터가 있다.

호스트들은 디폴트 라우팅을 하잖아요. route-4 print 하면 10개가 나올때 9개는 자기 가까이 있는 것이고 남은 하나가 디폴트 엔트리, 이 디폴트 엔트리에서는 일단 모르면 특정 라우터로보내는 것이다. 이것을 디폴트 라우팅이라고 하면

서브넷의 두개 라우터중 하나의 라우터를 골라서 디폴트 라우팅으로 정해놓는다면, R1으로 무조건 보내진다 근데  R1에 입장에서는 R2로 패킷을 보내는게 더 가깝다는 것을 안다. 그래서 R2로 다시 보내고, R2는 인터넷으로 패킷을 보내줄 것이다. R1 은 애초에 들어갈 필요 조차 없었던 것이다. 이 컨디션을 코드가 detact를한다. 패킷 전송에서는 exception handling을 한 것이 아니다. 아무 문제가 없다. 다만, R1에서 들어가는 패킷과 나가는 패킷이 같음을 인식하고 호스트에게 D 목적지에 관해서는 나한테 보내지말고 R2로 바꿔! 엔트리 추가하게 시키는 것

d → R2 이 엔트리에 해당하는 서브넷 마스크는 255.255.255.255일 것이다. 디폴트 엔트리로 나가지 않도록!

근데 이런 현상은 거의 없다. 말단 호스트들에 대해서 라우터를 2개를 사용할 일이 거의없다.

교수님 혹시 fragmentation 하다가 port 번호가 잘못 나눠지는 경우는 없을까요? 포트번호가 어떻게 생겼다면 잘못 나눠질 수가 없게끔 되어있다. 4바이트 얼라인으로 잘 되어있다.  포트번호까지 잘라지려면, framentation이 아주 잘잘하게 잘라져야한다. 근데 그럴 가능성은 전혀 없다.

만약 스마트폰에서 WiFi와 LTE가 동시에 연결되어서 사용되고 있는 상황이라면 이 특수한 경우라고 볼 수 있는건가요? 보통 안드로이드같은 경우네느 두개의 인터페이스가 running상태에서도 하나만 쓰도록 OS에서 컨트롤

어떤 업체에 따라서는 wifi가 값싼 기술이기 때문에 wifi와 5g를 동시에 사용하도록 하는 경우가 있다. 통상은 둘중에 하나만 쓰인다.

framentation needed unreachable 에러는 앞에서했기때문에 넘어간다.

ICMP 메시지가 있고, 헤더가 있고, 헤더에 문제가 되는MTU사이즈가 있다.

밑에는  너무 덩치가 커서 죽은 데이터그램의 앞부분이 들어간다. 위의 그림(Time exceeded 에러때 참고)은 참고하지마라

Time exceeded 에러 -11

두 종류 - 목적지 도착전  TTL=0, 목적지 도착 후 일부 frament 미도착

재조립은 중간 경로에서 일어나지 않고 최종 목적지에서만 일어난다. 조각이 다와야하는데 일부가도착을 안한 것

상당히 오랜기간동안 기다리게 한다. 15초

원하는 패킷만을 보내달라고 요구하기에도 힘들다.

따라서 그냥 다함께 죽인다.

정보 질의/응답 메시지 - 에러메시지 x

사망한 아이피데이터그램의 앞부분을 잘라서 넣는다. 이런 앞서말한 규칙들은 여기서는 다 해당되지 않는다. 에러메시지가 아니므로

에러 아닌 메시지의 대표 케이스 거의 살아남은 유일한 케이스 Ping = ICMP 에코 리퀘스트(type=8) + 에코 리플라이(type=0)

패킷 형식에서는 봐야할 것은 identifier와sequence number이다.

identifier는 프로세스별 고유값 - 윈도우즈에서는 사용하지 않는다. 핑이 여러개가 돌아가고 있으면 그럼 어케 알아? 해킹이나 뭐 그런방법을 통해서 알수 있다.

Sequence number는 보통 우리가 핑을 해보면 얼마 걸리는지도 알 수 있다. 이것을 알아내려면 짝을 잘 맞춰야한다. 1번에서 나가서 1번으로 들어가는 것끼리 빼서 시간 계산 -  이경우에 보게되면

시퀀스번호 - Big endient, Little endient  most significant bite, least significant byte 이렇게 나눠져있는데, 와이어샤크가 한 값을 갖고 친절하게 두값으로 표현해주고 있는 것

이 시퀀스 넘버가 하나씩 늘어날 것이다

데이터로는 61626364...이렇게 들어가있는데 아무 의미없는 ABCD...이런 것이다.

ICMPv6 IPv6에서의 ICMP를 말한다.

Ipv4에 와서는 ARp, IGMP의 역할을 ICMP가 한다.

ARP, IGMP가 사라지면서 그 역할을 ICMP가 하는 것이다. DHCP에게 빼앗겼던 일부 역할도 일부 회복- 에러와 정보 요청/응답

차이점은 MTU보다 큰 데이터그램 무조건 버리게 했었는데, 이름이 조금 바꼈다. Packet Too Big

dont fragment bit → IPv6에서는 라우터에서의 fragmentation자체가 없어졌다. 아예 없는게 하지말라고 할 필요가 없다. 왜 죽었냐? 니패킷 그냥 너무 커

fragmentation절대 안해준다.

멀티캐스트 경우에는 보여준다. → Ipv4에서는 무슨 경우에도 멀티캐스트에 관해서는 에러메시지를 보내지 않는다. 멀티캐스트로 다양한 갈래로 갈라져서 가는 중에 한 갈래에서 MTU가 작아도 에러메시지 보내지는 것

ICMP 정보 요청/응답을 ND에 활발히 사용

나의 디폴트라우터는 누구지? 이웃에 관하여 정보를 알아내는 것  그 중에 하나가 ARP

나의 다음홉 IP주소의 MAC 주소는 무엇일까 → ARP

ND

SNM 주소 다뤘지만, 다루지 않은 것에 대해서 조금 더 설명하자면, 구 ARP에 해당하는 케이스에 어떤 ICMP메시지를 쓰냐면 NS라는 메시지를 사용한다. 예전에ARP요청에 해당하는 부분

NS → 너의 mac주소를 달라

수신자를 Ipv6의 헤더에서 라우팅 룩업을 통해 찾아낸 nexthop의 IP주소말고, SNM 주소로 바꿔서 넣는다.

발신자는 ::으로 되어있다. Unspecified이다. 응답은 Multicast 주소로 줄 수 밖에 없다. Ipv6에는 브로드캐스트가 업삳. 따라서 all-nodes multicast로 주는 것

SNM주소가 ff03::1:ffde:fa64이것이다.

주소에 대해서 알고싶고, 이 Next hop ip주소를 갖고있는 애들만

NA - 구 ARP 응답

flags라는 필드가 있다. 3개가 있다. s라는 플래그가 가장 중요하다. 누가 요청했기 댸문에 응답을 보냈다 누가 요청하지 않았는데도 보냈다의 여부, s=0이면 요청없이 자발적으로 메시지를 보내는 것 내 핑이 바꼈으므로 이를 통보하는 것

s=1이면, NS에 대한 응답인 것 → 이 MAC주소가 뭐야? 라는 패킷이 들어왔었다는 뜻

o flag O override을 켜놔야

타겟은 끝까지 타겟으로 끝까지 남는다. 타겟이 응답을 해주고 있는 요청한자의 next hop에 해당

옵션 부분에 자신의 맥주소를 보내주고있는것을 볼수가 있다.

요청자가 unspecified 주소를 보냈기 떄문에 dst

요약하자면 ICMP는 IP와 단짝 프로토콜이며, 데이터그램을 보내는 것이 IP라면 ICMP는 에러나 상태 인식 등의 보조적인 역할 IPv4에서는 에러를 알려주는 용도로는 잘 사용했으나 두번쨰 용도가 매우 축소되는 와중이었다. Ipv6 는 다른 프로토콜의 역할을 뺏어온 것

all nodes multicast는 거의 IPv4의 broadcast와 동일하게 작동된다고 보면 되는건가요?

yes → 그치만 all nodes라고 할때 앞에 말이 붙는다. Multicast capable interface 멀티캐스트가 되는 애들만 보내준다. 이럴거면 브로드 캐스트를 왜 없앴냐? 일단 멀티캐스트로 통일해놓고 브로드캐스트로 하던것도 들어야하는 놈들만 듣도록

IPv6 애니캐스트 메시지에 대해서는 icmp를 보내주지 않는 것인가요? 애니캐스트와 유니캐스트는 IP 주소를 전혀 구별할 수 없으므로 호스트에서는 이를 구별할 수가 없어 ICMP 메시지를 보낼 것이다. 애니캐스트는 그런 구분 용은 아니므로 애초에 유니캐스트와 구분이 안된다.