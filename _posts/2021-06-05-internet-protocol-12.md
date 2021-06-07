---
layout: post
title: "[김효곤 교수의 인터넷 프로토콜] 12. IP 멀티캐스트"
categories:
  - Internet Protocol
tags:
  - 인터넷, 프로토콜, 그리고 계층화 원칙

---

# IP 멀티캐스트

### 12.0 IP 멀티캐스트 개관

- 다수의 대상 인터페이스에 **multiple unicast**가 아닌 IP 멀티캐스트 주소를 사용한 효율적 전송 가능

  - multiple unicast : 여러 대상 인터페이스에 대해 각각의 unicast를 보내는 것
    - 송신 host에서 필요한 통신선 용량이 큼
    - 송신 host에서 필요한 processing 용량이 큼
  - Computation(processing 용량) 및 bandwidth(통신선 용량) 절약: “Green technology”
    - 송신자가 하나의 패킷을 보내면 중간 라우터에서 사용자 개수만큼 복사
    - 라우터에게도 자신이 서브 호스트 개수만큼만 복사하기 때문에 부담스럽지 않다.
    - 아주 효율적인 **분배 트리** 구조

  ![]({{site.url}}/assets/images/183.png)

- Killer app -> IPTV
  - 1990년대 중반즈음에 IP멀티캐스트 연구가 이뤄졌지만 결과가 좋지 않았던 이유는 글로벌 멀티캐스트를 위해서는 모든 AS들이 비협조적인 태도를 보였기 때문이다. 그래서 OSPF처럼 라우터들끼리 대화하는 것 이외의 용도로는 거의 쓰이지 않았다. 현재는 한 개의 망을 단위로 하여 멀티캐스트를 한다.
  - IPTV가 IP 멀티캐스트 기술의 부활을 가져옴
    - IPTV 전용망

___

- 2 단계 구현
  - End host – router : 가입/탈퇴 정보 수집
    - **그룹 관리 프로토콜 (IGMP)** 사용
      - host에서 작동하는 응용들이 멀티캐스트 그룹에 가입 탈퇴하는 정보를 수집
- Router – router : 분배 트리 형성
  - **멀티캐스트 라우팅 프로토콜** 사용
    - 어떻게 효과적으로 가입자들에게 패킷을 분배할 것인가
    - OSPF인가 BGP인가? 둘다 아니다. 전용 프로토콜이 따로 존재

___

### 12.1 Internet Group Management Protocol

- IGMP
  - 호스트 컴퓨터가 **특정 IP 멀티캐스트 그룹 (Class D IP 멀티캐스트 주소)** 가입/탈퇴를 **라우터에게 알리기 위한 프로토콜**
    - 멀티캐스트 그룹 하나 당 Class D IP 멀티캐스트 주소 하나 매핑
  - 호스트는 인터페이스에 **자신이 가입한 그룹(Class D 주소)의 리스트**를 기록
    - 해당 호스트의 링크 계층에서도 MAC 주소가 설치되어 **32:1 MAC 주소**를 통해 자신에게로 온 패킷임을 인식할 수 있도록 설정
      - 32:1 MAC : IP 멀티 캐스트 주소 32개가 MAC 주소 1개에 매핑
    - 멀티캐스트 트래픽을 받을 인터페이스에 여러 응용 프로그램이 같은 그룹에 가입한 경우: **참조(reference) 카운트 증가** 
      - 가입/탈퇴의 주체는 응용이다.
      - reference 카운트가 0이 되면 인터페이스에 가입정보를 지우고 **탈퇴했다는 메시지를 즉시 라우터에게 전송**
  - 라우터는 자신이 담당한 **서브넷의 인터페이스들이 가입한 그룹 리스트** 유지
    - **주기적(대략 2분마다)/비주기적** 멥버쉽 체크
    - 상태 변화 (가입/탈퇴)의 경우 멀티캐스트 라우팅 프로토콜을 사용, **분배 트리에 반영** (상위 라우터와 대화)

> **한 응용 프로그램이 특정 멀티캐스트 그룹에 가입했을 경우**
>
> 1. 호스트의 커널이 이 가입 정보를 받아서 인터페이스에다가 기록
> 2. 하드웨어에 번역된 맥주소 매핑
> 3. 가입 정보를 IGMP 메시지를 라우터에게 전송

___

- IGMP
  - IGMP를 보내기 전에 호스트는 인터페이스에 **가입한 그룹의 리스트**를 기록/관리
    - Netsh interface ip show joins
    - 두번째 컬럼에서 `참조`에 해당하는 것이 reference count로 몇 개의 프로세스가 가입되어있는지를 나타낸다.
    - 224.0.0.251에 대해서 어떤 종류의 프로세스가 가입하는지는 인터넷에 검색해보면 금방 알 수 있다.
      https://stackoverflow.com/questions/12483717/what-is-the-multicast-doing-on-224-0-0-251

![]({{site.url}}/assets/images/184.png)

___

- IGMP **v1** 메시지 형식
  - Version = 1
  - Type
    - 0x11: 멤버쉽 질의
      - 라우터가 주기적/비주기적으로 호스트에게 가입 정보를 묻는 것
    - 0x12: 멤버쉽 리포트 (질의 없이 가능)
      - 호스트가 자신의 가입 정보를 보고하는 것
      - 라우터가 묻지 않아도 새로 가입하게 되는 순간 자발적으로 리포트를 함
      - 질의에 대해서는 1개의 인터페이스만 응답하면 됨
        - 왜? 라우터 입장에서는 가입자 한명만 있으면 멀티캐스트 패킷을 받아다가 브로드캐스트할 것이기 때문
        - 왜 브로드캐스트? 멀티캐스트가 개발되던 시절의 상황을 반영.  이더넷이니까 가장 말단 네트워크는 브로드캐스트라고 가정했던 개발자의 철학
      - 질의를 하면 **10초** 내에 random(호스트에서 10 아래의 난수를 정함)하게 응답하게 하고, 누군가 먼저 응답하면 추가 응답은 없음
        - 리포트는 도착지 주소를 그룹 ip주소로 설정하여 라우터 뿐만 아니라 그 그룹에 가입된 다른 호스트도 들을 수 있도록 한다. 이를 통해 누군가가 먼저 응답하면 다른 가입자들은 이를 듣고 응답을 하지 않을 수가 있다.
  - Group address = 멀티캐스트 그룹 주소
    - 멤버쉽 질의 중 General Query에서는 0.0.0.0
      - "가입한 그룹은 다 줘봐"

![]({{site.url}}/assets/images/185.png)

---

- IGMP v2 메시지 형식 (IPTV 등장 전 & IPTV 등장 직후 지배적인 프로토콜 버전)
  - Type : version 필드 내포 (“1”)
    - 0x11: 멤버쉽 질의
    - 0x12: v1 멤버쉽 리포트
      - 하위 호환성을 위해 유지
    - 0x16: v2 멤버쉽 리포트
    - **0x17: 그룹 탈퇴 (Leave)**
      - 라우터에서 질의를 할 때, IGMP는 IP위에 실려가므로 한번에 잘 가리라는 보장이 없다. 따라서 두 번을 보내본다.
      - General Query 주기 = 125초, timeout= 125*2+10 = 260초
      - 260초 동안 발생할 트래픽을 절약하기 위해서는 260초가 만료되기 전에 지금 끊어버리고 싶다고 탈퇴 메시지를 보내게 할 수가 있다.
- Max response time
  - 응답을 재촉할 수 있게 되었다. 
  - v1에서는 쿼리 주기가 딱 정해져있었던 것은 아니다. 
  - 0x11에 대한 응답이 요구되는 시간 (0.1초 단위)

![]({{site.url}}/assets/images/186.png)

> **v1과 v2의 차이**
>
> - version -> type  (0x01 => 0x1~)
>   - 첫 니블이 0이냐 1이냐에 따라 버전 구분 가능
> - type -> max-response time
> - unused 삭제
> - Checksum 길이 길어짐

---

- IGMPv2 메시지 예
  - IGMP 메시지 자체는 멀티캐스트 메시지가 아니다. IPTV 영상을 싣고 오는 멀티 캐스트의 화견을 조성하기 위한 **control 메시지**인 것이다.
  - dst : 224.0.0.1
    - 224.0.0.0/24 => IP 멀티캐스트 주소
    - 그중에서 224.0.0.1은 "all interfaces"로 모든 멀티캐스트 capable한 호스트의 인터페이스들을 타겟으로 하고 있음을 의미
- IP 헤더 Option : route alert
  - 라우터가 IP 메시지를 열어보게 하는 옵션
    - 예전에는 라우터가 모든 그룹에 조인되어있다고 생각하여 모든 멀티캐스트의 트래픽은 다 받아보도록 했다가 이 옵션이 있는메시지만 (자신이 필요한 그룹의 IGMP 메시지) 라우터가 열어보도록 축소.
- 회색 띠 밑으로 IGMP 메시지이다.
  - version 2
  - max response time - 10초 (통상 10초이지만 바꿀 수 있음)
  - multicast address - 0.0.0.0 (제너럴 쿼리)

![]({{site.url}}/assets/images/187.png)

---

- General query
  - 125초마다 제너럴 쿼리가 보내지고 있다.
  - 제너럴 쿼리 전송 후 10초 안에 호스트들이 자신이 어떤 그룹에 가입되어있는지 보고 중
    - IP 목적지 주소에 자신이 가입할 그룹 주소를 넣음으로써 라우터뿐만 아니라 이미 해당 그룹에 가입된 호스트들에게도 가게 함

![]({{site.url}}/assets/images/188.png)

![]({{site.url}}/assets/images/189.png)

---

- Group-specific query
  - Leave 메시지 전송 후, 해당 그룹에 대해서 다른 가입자가 남아있는지 확인하기 위한 메시지
  - 1초 간격으로 두번 전송
    - 응답이 없으면 끊는다.

- 아래의 예시
  - 어떤 호스트가 224.0.0.2로 라우터에게만 슬쩍 leave 메시지를 보냄
    - 224.0.0.2 : "all routers" - "멀티캐스트 라우터들만 들어라."
  - Group-specific query를 통해서 남은 가입자들이 있는지 1초 간격으로 두 번 확인
    - group ip 주소를 목적지로 하여 전송

![]({{site.url}}/assets/images/190.png)

___

- IGMPv2 -> IGMPv3 를 야기한 멀티캐스트 모델의 변화
  - IPTV를 위해 등장
  - **Any-Source Multicast (ASM)** -> **Source-Specific Multicast (SSM)**
    - Any-Source Multicast
    - Source-Specific Multicast : 목적지가 자신이 속한 그룹인지 뿐만 아니라, 적법한 source에서 온 패킷인지 확인
      - 철저히 IPTV를 위한 변화

- ASM: 원래의 IP 멀티캐스트 모델
  - 수신자는 송신자가 누구냐에 상관 없이 자신이 목적지 주소가 자신이 **가입한 그룹이면 무조건 수신**
    - 송신자 또한 수신자를 몰라도 된다: IP 멀티캐스트 설계 철학
      - 송신자가 **멤버 관리(분배 트리)를 할 필요가 없다** -> **그룹 크기 scalability** 
  - IPTV: 아무나 그룹(“채널“)에 패킷을 보내도 되나?
    - 해킹 가능
    - IPTV 셋톱박스들이 원하는 패킷 외에 다른 패킷들도 받을 수 있는 불안정한 네트워크 환경이 조성됨

- SSM: IPTV를 위한 **전송자 체크** 및 **거부** 가능 모델
  - IGMP와 PIM을 모두 변화시킴
  - IP source address = **정해진 전송자** (예: 방송국 콘텐츠 서버)인 것만 수신
    - 나머지는 응용계층까지 올라가지 않는다.
  - IGMPv2 메시지에는 source IP 주소를 적는 field가 없다
    - 당연 – ASM 모델이므로
  - 새로운 버전 필요
    - 소스 ip주소를 명시하는 필드가 필요
    - SSM : IGMPv3 (IPv4) - ICMP의 MLD(Multi Cast Listener Discovery)v2 (IPv6) 
      - ASM :  IGMPv2 - MLDv1

> **IP 헤더에 소스 주소가 있을 텐데, 꼭 IGMP 메시지에 소스 주소를 적어줄 필요가 있나요?**
>
> PIM을 배우면 알게 되겠지만 IP 헤더의 소스 주소에는 해당 멀티캐스트 주소가 아닌 것이 들어가는 경우가 있으므로 IGMP 메시지에 별도로 명시가 필요하다.

___

- IGMP v3 메시지 형식

  - 소스 주소를 적는 필드(Source Address)가 여러개 생김
  - 소스에 대해 describe해주는 필드(Reserved, S, QRV, QQIC, Number of Sources)들도 생김
  - IGMPv2와 첫 두줄은 거의 유사함
    - max-response `time` => max-response `code` :  값의 포맷이 time 하나뿐이었던 것에서 code를 써넣을 수 있게끔 포맷이 확장되었기 때문이다.

  ![]({{site.url}}/assets/images/191.png)

  - Type : 하위호환성을 위해 첫 니블을 1로 설정
    - 0x11: 멤버쉽 질의
    - *<u>0x22: 멤버쉽 리포트</u>* : 이것만 v3에서 새로 추가된 것
      - 가입과 **탈퇴**를 포함
    - 0x12: **v1** 멤버쉽 리포트 (하위호환성)
    - 0x16: **v2** 멤버쉽 리포트 (하위호환성)
    - 0x17: **v2** 그룹 탈퇴 (하위호환성)

  > 하위호환성
  >
  > 같은 서브넷에 존재하는 라우터의 버전들이 다를 경우에도 문제 없이 IGMP가 작동되도록 하기 위해 하위호환성을 맞춰야 한다.

  - Reserved : 안 쓰임
  - Number of Sources : 뒤에 오는 소스 주소 개수
  - Querier’s Robustness Variable (QRV)
    - 질의가 잃어버려졌을 때 몇 번까지 전송할 것인가
      - v2에서는 2번이라고 정해졌지만, 이숫자가 어떤 케이스에서도 효율적이고 동시에 신뢰적이지는 않기 때문에링크의 신뢰성과 같은 네트워크 환경에 따라 전송 횟수를 직접 정할 수 있게 함
  - Querier’s Query Interval Code (QQIV)
    - 질의 간 간격 (단위 = 초)
      - 마찬가지로 몇 초 간격으로 질의를 보낼 것인지도 직접 정할 수 있도록 함

___

- IGMP v3 메시지 형식
  - 질의의 3 가지 종류
    - \+) Group-and-source-specific Query : A 소스로부터 오는 B 그룹의 메시지를 받아보고 있는 가입자를 찾는 질의
      - src 주소와 dst 주소는 모두 Group-specific address

![]({{site.url}}/assets/images/192.png)

- IGMPv3 질의 예시
  - QRV : 2번
  - QQIC : 60초

![]({{site.url}}/assets/images/193.png)

___

- IGMP v3 멤버쉽 리포트
  - join&leave를 둘 다 표현
  - 그룹 레코드 각각은 그룹에 대해 송신자별 수신 여부 표현

![]({{site.url}}/assets/images/194.png)

- Type = 0x22 : 리포트

- Reserved : max-reponse code 필드가 사라짐

- Group records : 이 그룹을 join하거나 leave하고 싶다는 의미

  - 구성

    ![]({{site.url}}/assets/images/195.png)

  - 주소만 적힌 것이 아니라, leave를 표현하기 위해 복잡하게 구성됨

    - Group Address
    - Source Address : "`이 소스`에서 온 이 그룹에 가입하고 싶다"에서 `이 소스`
    - Record Type : join? leave?

___

- IGMP v3 멤버쉽 리포트 예
  - Version : 3
  - Type : report (0x22)
  - Num Gourp Records : 1
  - Group Record
    - Record Type : Change to Exclude Mode
    - num source : 0 - IPTV에서 패킷을 따온 것이 아니기 때문에 굳이 소스 주소 체크할 필요 x

![]({{site.url}}/assets/images/196.png)

___

- Record type: 필터링 모드 = INCLUDE vs. EXCLUDE
  - Solicited Report (요청된 리포트) - 현재 상태 보고
    - MODE_IS_INCLUDE (1)
    - MODE_IS_EXCLUDE (2)
  - Unsolicited Report (자발적인 리포트) - 변화 보고
    - CHANGE_TO_INCLUDE_MODE (3)
      - 특히, **CHANGE_TO_INCLUDE MODE { } (공집합) : 탈퇴!**
    - CHANGE_TO_EXCLUDE_MODE (4)
      - CHANGE_TO_EXCLUDE_MODE {} : 모든 소스로부터 오는 패킷을 받겠다.
  - Source를 list에서 더하거나 빼는 경우 - Group 멤버쉽은 유지
    - ALLOW_NEW_SOURCES (5)
    - BLOCK_OLD_SOURCES (6)
- 필터링 모드 설정 예
  - INCLUDE {a,b,c} 상태를 INCLUDE {b,c,d}로 바꾸기
    - ALLOW_NEW_SOURCES(d)
    - BLOCK_OLD_SOURCES(a)
  - **EXCLUDE {a,b,c} 상태를 EXCLUDE {b,c,d}로 바꾸기** - 배제하하는 소스 리스트 변경
    - ALLOW_NEW_SOURCES(a)
    - BLOCK_OLD_SOURCES(d)

> Q : IGMP v1, v2는 첫 니블로 version을 구분할 수 있다고 하셨는데, v2와 v3는 이런 구분 방법이 없는 건가요?
> A : 첫 니블이 1이냐 2냐로 구분이 가능하며 v3의 질의는 아주 유사한 형식에서 뒤의 몇 개의 필드만 추가된 것이므로 굳이 구분하지 않는다.

> SSM으로 만들어놓기는 했지만 실제로 사용하는 곳은 IPTV밖에 없다. IGMPv3로 한다 하더라도 뭐하러 특정 소스를 배제할 필요가 있겠는가? 늘상 ASM이다. 그렇지만 이미 업그레이드가 되었으니 IPTV가 아닌 호스트들은 그냥 어쩔 수 없이 Record Type에서 3,4만 쓰게 된다.

- 여러 응용이 관계된 때의 필터링 모드 설정 예 
  - **대원칙 - "어느 한 응용이라도 수용하는 소스들은 모두 배제하지 않고 포함하겠다."**
  - 여러 응용이 INCLUDE 모드만을 사용 
    - Source list = 모든 source의 합집합 
  - 한 응용이라도 EXCLUDE 모드를 사용 
    - INCLUDE {} 모드를 -> EXCLUDE ~{}로 변환
    - Source list = 모든 source의 교집합(을 거부)
  - 예) 그룹 G에 한 인터페이스를 통해 3개의 응용이 가입
    - INCLUDE {a,b,c}, {b,c,d}, {e,f} -> (합집합) -> INCLUDE {a,b,c,d,e,f}
    - EXCLUDE {a,b,c}, {b,c,d} + INCLUDE {d,e,f} -> EXCLUDE {a,b,c}, {b,c,d} + **EXCLUDE ~{d,e,f}** -> (교집합) -> EXCLUDE {b,c}

___

- 가입/탈퇴에 관한 IGMPv2와 IGMPv3 메시지 차이
  - Type : 리포트임은 같으나 Type 번호는 다름
  - Destination IP : IGMPv3는 IGMPv3가 가능한 라우터들을 타겟으로 하는 224.0.0.22로 설정
    - LEAVE와 통일하기 위함
  - Address to join is in : 둘 다 Group Address에 적는다.

![]({{site.url}}/assets/images/197.png)

- 라우터에서의 그룹별 상태
  - <그룹 주소, 타이머, 필터 모드, [발신자 주소, 발신자 타이머]>
    - 예) 발신자 주소 = { }, 필터 모드 = EXCLUDE : 그룹 가입 상태
  - 그룹 타이머는 EXCLUDE 모드에서만 사용
    - 타임 아웃 -> INCLUDE로 변환

___

### 12.2 IPTV

- 채널 변경시 발생하는 IGMPv2 패킷

  - KT의 **GLOP address** 사용 : SBS -> KBS로 채널 변경
    - SBS에게서 메시지를 받고 있었음 : SBS 서버 `125.159.0.53`
    - SBS 그룹 LEAVE
    - KBS 그룹 JOIN
      - src : `118.48.34.143` - 교수님 셋톱박스
    - 그 후에 아직 끊기기전에 전송된 SBS 패킷이 마지막으로 도착
    - SBS에서 Group-specific query
      - src : `61.72.223.167` - 교수님 셋톱박스를 담당하는 라우터
    - KBS에서 패킷을 흘려보냄 : KBS 서버 `125.159.1.170`
  
  ![]({{site.url}}/assets/images/198.png)

> **GLOP (Global Private) address**
>
> ip global multicast 223.0.0.0/24 : 자신의 AS를 띄워놓으면 마치 자기 자신의 주소인 것처럼 사용 가능

- IGMP Proxying

  - 교수님 셋톱박스와 교수님 셋톱박스에게 Group-specific query를 보낸 서버가 같은 링크에 있지 않다.
    - 이것을 가능하게 해주는 기법은 IGMP Proxying으로, 중간의 셋톱박스와 같은 링크에 있는 라우터가 투명인간처럼 IGMPv3 capable한 멀티캐스트 라우터 사이에서 대리로 전달해주고 있는 것
    - 링크마다 IGMP capable 멀티캐스트 라우터를 둘 필요가 없게 됨

  ![]({{site.url}}/assets/images/199.png)

___

### 12.3 IPv6에서의 멤버쉽 관리

- IGMP 없음: **ICMPv6**를 사용한 **Multicast Listener Discovery (MLD)**
  - MLDv1 = IGMPv2 (ASM)
  - MLDv2 = IGMPv3 (SSM)
  - **Link-local source IP**, **TTL=1**, IP Router Alert Option
    - 보통 기존의 IGMP가 미치던 범위는 **한 홉**이다. (IGMP Proxing 예외)
    - 원래에는 라우터는 모든 멀티캐스트 그룹의 가입원이다라는 가정 하에 모든 멀티캐스트 그룹을 열어보도록 했지만 지금은 그럴 필요없이 IP Router Alert Option만 필요할 때 설정한다.

- MLDv1 형식
  - Type
    - 130 = receiver query
    - 131 = receiver report (자발적/비자발적)
    - **132 = receiver done (탈퇴)**
      - IGMPv2이므로 탈퇴가 명시되어 있음
  - Checksum: **pseudo-header** 사용
    - **IP 주소 2개 + payload length** + next header(ICMPv6 헤더)
    - IP 레벨에서 한번더 체크해주는 역할 포함
  - Max Response Delay (단위ms)

![]({{site.url}}/assets/images/200.png)

- MLDv2 메시지 - SSM을 위한 필드 추가
  - General Query는 ff02::1로
    - IPv4에서는 224.0.0.1
  - Multicast-Address-Specific Query (구 Group-Specific Query)는 ip source 주소가 그룹 주소로 설정됨
  - Report
    - MLDv1: 해당 그룹 주소로
    - MLDv2: **ff02::16**으로 
      - IPv4에서는 224.0.0.2/224.0.0.22
      - 모든 **MLDv2-capable 라우터 그룹**
  - **Done 메시지(탈퇴)**는 **ff02::2**로
    - **모든 라우터 그룹**

![]({{site.url}}/assets/images/201.png)

- MLDv2 메시지 예
  - IPv6 Hop by Hop Options - Router alert
  - IP dst Addr : `ff02::16` - MLDv2 capable 라우터들 => **REPORT**
  - IP src Addr : `fe80::..` - Link Local Addr
  - Report Message v2
  - Multicast Address Record "Change to Include"
    - **<u>Number of Sources : 0</u>** => 탈퇴!!

![]({{site.url}}/assets/images/202.png)

___

### 12.4 멀티캐스트 라우팅 프로토콜

- 멀티캐스트 그룹의 소스로부터 가입자들까지 멀티캐스트 메시지를 효과적으로 전달하기 위한 분배트리를 구성하기 위해 만들어진 프로토콜
  - 분배 트리를 구성하기 위한 **라우터들끼리의 대화**
  - **Protocol-Independent Multicast (PIM)**이 표준
    - PIM은 OSPF와 같이 같은 링크 안에서만 대화하기 위한 용도로는 필요가 없다. 원격의 서버에서부터 시작하여 수많은 가입자들까지 메시지를 분배하기 위한 **트리 구성을 위한 용도**이다.
    - 이름의 유래 : 멀티캐스트 라우팅 테이블을 만들때 유니캐스트 라우팅 테이블을 조금 사용한다. 이 때 어떤 프로토콜이 유니캐스트 라우팅 테이블을 만들었는지 상관하지 않고 만들어진 결과만 사용하기 때문에 (유니캐스트) 프로토콜의 종류에 상관없이 테이블을 참조하여 테이블을 만든다는 의미에서 이름이 지어졌다.
    - IP 멀티캐스트가 ASM이었을 당시에는 PIM-DM과 PIM-SM을 사용했으며, IGMP와 더불어서 PIM도 SSM 모델을 지원하기 위해 **PIM-SSM**이 새로 생겨났다.

![]({{site.url}}/assets/images/203.png)

- PIM-DM(dense mode)은 **flood-and-prune** 방식
  - 멤버쉽 변화에 맞춰 **분배 트리를 수정**하기 위해, 모든 멀티캐스트 라우터들은 주기적으로 source로부터의 트래픽 **flooding**을 허용
    - Multicast 패킷이 들어온 인터페이스 외의 모든 인터페이스로 복사해서 내보냄
    - **주기 예) 2분** : 그 사이에 멤버십 변화가 생길 경우를 대비해서
  - Reverse Path Forwarding (RPF) 검사
    - 불필요한 flooding을 제거하기 위한 검사
    - 만일 source 쪽으로 unicast 트래픽을 보낸다면 사용할 인터페이스로 들어온 multicast traffic은 forward; 나머지 인터페이스로 들어온 multicast traffic은 무시 
  - 멀티캐스트 라우터들은 **수신 여부를 상류 (upstream) 라우터에게 통고**
    - 하류에 수신자가 있는 경우: no action
    - 하류에 수신자가 **없는** 경우: PRUNE 메시지

> **PIM-DM은 좀 옛날 방식**
>
> 사실상 IPTV 망에서는 잘 사용되지 않는다. 거의 모든 가입자가 flooding해주는 메시지를 받아볼 의향이 있는 경우(dense한 경우)가 아니라면, 이것은 비효율적인 방법이다. 현재는 점점 더 공중파 시청률이 떨어지고 있으므로 더욱 더 DM은 효율적인 방법이 될 수가 없는 것이다.
>
> 또한 2분마다 주기가 있다는 것은 그 2분 사이에 채널을 바꾸면 바로 원하는 영상이 나오지 않는다는 치명적인 단점이 있다. 그렇다고 주기를 짧게 하면 할수록 더 효율이 안좋아지는 딜레마에 있는 것이다.

___

- PIM-DM은 flood-and-prune 방식

  - Flood : 오로지 라우터들 사이에서 발생

    - 서로 주고 받고 난리가 남

    ![]({{site.url}}/assets/images/204.png)

  - Reverse Path Forwarding (RPF) check

    - 똑같은 패킷이 한 라우터에게 여러번 보내지거나 루프를 발생시키는 비효율적인 flood를 차단
    - 방법 : 각각의 라우터에서 소스에서부터 최단 경로로 도착할 수 있는 패킷만 받고 나머지는 차단
      - 최단 경로를 구하는 방법 : 소스를 목적지로 하는 경로를 유니캐스트 라우팅 테이블에서 lookup하여 해당 엔트리의 next hop 인터페이스로 온 패킷만을 받음
    - 이를 통해 일방향으로 downstream으로만 흘러가도록 제어

    ![]({{site.url}}/assets/images/205.png)

    > **최단 경로를 판단하기 위해 TTL을 사용하면 안될까?**
    >
    > - TTL을 통해 경로를 구하는 것은 지금은 사용되지 않는 RARP를 사용하는 방법이다.
    >
    > - 또한 이제는 OSPF를 통해 최단 경로를 구하게 되는데, 이 때, 단순히 홉 수 뿐만 아니라, 한 링크를 건너가는데 발생하는 비용을 고려하기 때문에 TTL보다는 라우팅 테이블 lookup을 통해 최단 경로를 판단하는 것이 더 정확하다. 
    > - Protocol Independant Protocol이라는 점에서도, 어떤 protocol을 사용했는지 여부와 상관없이 만들어진 라우팅 테이블만을 사용한다는 설계 철학이 반영되었다.

  - Prune

    - Prune을 해야되는 라우터라면 자신이 RPF를 통해 무시했던 수용했든 관계없이 자신에게 패킷을 보낸 모든 인터페이스에게 Prune 메시지를 보낸다.
    - RPF를 통해 패킷을 수용한 라우터를 자신의 상위 라우터라고 간주하고, 자신이 패킷을 보내준 모든 나머지 라우터들에게서 Prune을 모두 받은 경우에만 한하여 상위 라우터에게 Prune을 보낸다.

    ![]({{site.url}}/assets/images/206.png)

___

- PIM-DM 멀티캐스트 라우터 상태
  - Prune은 논리적인 flag를 통해서 기록된다.
    - InPort : 메시지가 들어온 포트(상위)
    - Outports : 메시지가 나가는 포트(하위)
    - 모든 Outports에서 p라는 flag가 기록되면 Inport에도 Pr이라는 flag를 마크해놓음
  - 멀티캐스트 라우팅 테이블 =/= 유니캐스트 라우팅 테이블
  - Prune 기록은 다음 flooding 때까지만 유효
    - 주기가 지나면 flag 기록이 모두 지워지면서 다시 경로가 뚫림

![]({{site.url}}/assets/images/207.png)

___

- Protocol-Independent Multicast (PIM) 프로토콜
  - 사실상 표준
  - RPF 계산 시 unicast 라우팅 테이블 참고
    - 어떤 unicast 라우팅 프로토콜이든지 상관 없다 (protocol-independent)

- PIM 모드
  - DM
    - Flood-and-prune
    - **대부분의 멀티캐스트 라우터가 수신**할 때 유의미 (cf. IPTV???)
  - Sparse Mode (SM)
  - Source-Specific Multicast (SSM)
  - Bi-directional PIM

- PIM-SM
  - 트리 가지는 **JOIN 요청이 있어야** 만들어짐
    - 새로운 멤버가 생기면 트리에 덧붙여줌
  - **Core (Rendezvous Point; RP)**라는 특별한 라우터 사용

- PIM-SSM
  - **SSM을 지원**하기 위해서는 IGMP 뿐 아니라 PIM도 바뀌어야 했다
  - 소스 유효성 반영

- Bi-directional PIM
  - 그룹에 **발신자가 많을 때**
  - 링크에서 RP로 지정 전달자 (Designated Fowarder; DF) 존재

___

- PIM 메시지 형식
  - PIM = **IP 프로토콜** **103**
  - 트랜스포트를 사용하지 않고 IP 위에 바로 실려감

![]({{site.url}}/assets/images/208.png)

- Type

  ![]({{site.url}}/assets/images/209.png)

- PIM 메시지 형식

  - Hello: 각 인터페이스가 주기적으로 **ALL-PIM-ROUTERS** 그룹에 전송

    - `224.0.0.13`, `ff02::d`
    - **인접 라우터** 파악
    - Options 내용
      - **Hold time**: Hello 메시지 없이 **이 시간이 지나면 연결 불가능**으로 간주
      - **Generation ID**: 바뀌면 **이웃 라우터의 상태가 바뀐 것**으로 간주
      - DR 우선순위

    ![]({{site.url}}/assets/images/210.png)

  - Register

    - **PIM-SM**에서만 쓰임
    - Source를 담당한 라우터가 **RP**에 트래픽을 등록하기 위해 멀티캐스트 패킷을 tunneling
      - 등록 : 송신자가 수신자를 모르는 상태에서 트래픽을 일단 RP에게 보내주는 것
      - 등록은 패킷을 하나하나 포장하는 과정이며 더군다나 최종 목적지도 아니기 때문에 비용이 꽤 되는 작업이다.
    - 등록 중지 (Register-Stop)은 **최단 경로 트리**를 통한 배달로 전환하도록 RP가 Source 담당 라우터로 전송
      - 서로의 IP주소를 알게 되면, 비용이 드는 등록을 멈추고 다이렉트하게 보내는 것을 의미한다.

    ![]({{site.url}}/assets/images/211.png)

    - 소스 호스트가 자신을 담당하는 라우터에게 패킷을 보내주면 해당 라우터는 RP의 주소를 이미 알고 있기 때문에 RP로 유니캐스트를 보낸다.

> **RP는 중매쟁이**
>
> IP 멀티캐스트의 가장 중요한 설계 철학은 송신자가 수신자를 모른다는 점이다. 수신자도 마찬가지로 처음에는 송신자가 누구인지 모르며 그룹 주소만 알고 있다. RP는 이러한 양측에게 있어서 만남의 광장이다. 송신자의 트래픽도 RP로 가며, 수신자의 Request 메시지도 RP로 간다. 기본적인 전제는 송신자도, 수신자도 RP의 위치는 안다는 점이다.

> **PIM은 라우터들끼리만 이뤄지는 프로토콜**
>
> 멀티캐스트 센더가 되는 호스트는 그냥 자신을 담당하는 라우터에게 패킷만 단순히 보내줄 뿐, PIM 대화에 끼지 않는다. 대화에 관여하는 것은 패킷을 받은 라우터뿐이다.

> **IPTV 망은 IP멀티캐스트만을 위해 셋업된 망**
>
> 따라서 이 망에 있는 모든 라우터는 RP의 주소를 알고 있도록 설정이 되어있다. 수동으로 설정할 수도 있고, 자동으로 설정하는 프로토콜의 피쳐를 사용할 수도 있다.

