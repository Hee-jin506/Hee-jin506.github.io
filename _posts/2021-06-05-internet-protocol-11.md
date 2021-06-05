---
layout: post
title: "[김효곤 교수의 인터넷 프로토콜] 11. Domain Name System (DNS)"
categories:
  - Internet Protocol
tags:
  - 인터넷, 프로토콜, 그리고 계층화 원칙


---

# Domain Name System (DNS)

### 11.0 DNS 개관

- DHCP와 더불어서 실제 사용자의 응용은 아니지만 뒤에서 infrastructure를 제공해주는 응용 계층 프로토콜 중 하나이다.
- Domain name은 URL이 아니다. URL은 URI(Uniform Resource Identifier)라고 하는 인터넷의 자원을 지칭하는 방법 중 하나이다. URI에는 URC, URN, URL이 있다.
  - URN (Uniform Resource Name) : 인터넷 자원을 이름으로 지칭
  - URC (Uniform Resource Characteristics) : 인터넷 자원을 특징의 열거로 지칭 
  - URL (Uniform Resource Location) : 인터넷 자원을 특정 위치로 지칭 - file path의 형태로 나타나며, 원하는 목적지인 호스트 안의 이 파일로 들어가면 원하는 리소스가 있다는 일련의 지칭 이라고 할 수 있다.
    - https://https://edition.cnn.com/2021/04/30/world/mars-helicopter-future-flights-rover-scn-trnd/index.html - 전체가 URL
    - 도메인 이름은 위에서 https://https://edition.cnn.com 에만 해당한다.

- 현대의 인터넷은 DNS에 크게 의존 - infrastructure라고 해도 과언 X
  - 다음 페이지 하나에 URL 323개, 네이버 페이지 하나에 URL 464개
    - 이 URL에 들어있는 Domain Name이 IP주소로 바뀌어야만 리소스에 접근이 가능
  - DNS 없이 이메일 전송 불가능

- 호스트 IP 주소를 기억하기 어려워 별칭으로 부른 것에서 시작
  - IP 주소를 대체할 수는 없음 – 인터넷의 주소는 IP 주소가 유일
  - IP는 인터페이스에, 도메인 이름은 호스트에 붙임
    - 1:1, 1:다, 다:1 모두 가능
- 1984년 시작, 1987년 표준화
  - HOSTS.TXT
    - 이 파일에다가 ip주소를 넣고 그 옆에 별명을 적었다.
  - 인터넷이 커지면서 테이블 관리가 힘들어져 자동화
    - 도메인이 ip주소를 대체할 수는 없으며, 언제까지나 ip주소 없이는 인터넷에 통신할 수가 없다. 
    - ip주소는 인터페이스에 붙지만 도메인은 호스트에 붙는다. 파일의 이름이 hosts인 이유도 그것이다. 호스트와 인터페이스의 관계는 일대다이기 때문에 한 호스트에 붙은 여러개 ip주소에 대하여 하나의 도메인이 붙을 수 있다.
  - 도메인 이름 -> IP 번역에서 시작해서 데이터베이스로 진화
    - 도메인 이름 -> IP주소를 비롯한 다양한 종류의 데이터
- 응용 프로그램 내에서 resolver 함수를 통해 호출
  - 응용 프로토콜은 응용 소프트웨어 프로그램 안에서 짜임
  - resolver는 도메인 이름을 ip주소로 바꾸는 함수

### 11.1 도메인 이름 공간

- 도메인 = 命名(naming) 도메인
  - '(법칙이나 힘이 유효한)영역'
    - 'Last name' - 우리가 고른것이 아니라 부여 받은 것
    - 'suffix' - 강제되는 규칙에 의해 부여 받은 이름
  - 아무렇게나 도메인 이름을 정할 수 없다
  - “Last name”, 즉 suffix는 정해져 있다
    - www.korea.ac.kr
      - suffix : korea.ac.kr
      - label : www - 선택할 수 있는 부분
- DNS 이름 공간의 트리 구조 (abstract)
  - kr -> ac -> korea 라는 노드 아래에서 태어났기 때문에 이것들이 모두 역순으로 suffix가 된 것
  -  label : 노드에 붙은 String

![]({{site.url}}/assets/images/174.png)

- 트리 노드에 label이 붙음
  - 영문자, 숫자, hyphen 조합 0~63자
    - 다른 언어는 **Punycode**로 변환 후 사용 (63자를 넘어가선 안됨)
    - 예) “우리나라“ = “**xn--**910bs4k7yb12w”
      - xn-- : 변환된 Punycode에 붙는 prefix
  - Root 노드의 label은 길이가 0 (유일)
    - 위의 트리 구조 제일 위에 보면 아무것도 없는 것 같지만 사실은 길이가 0인 String
  - 대소문자 구분 X
    - 예) ReSeaRCH = research = RESEARCH
    - 내부적으로 모두 대문자로 변환됨
- 주어진 노드의 도메인 이름 
  - Root 방향으로 트리를 **traverse**하면서 만나는 노드의 label을 concatenate
    - 예) cs (label) -> korea -> ac -> kr : `cs.korea.ac.kr.`
    - 서양식 명명법
    - root도 label이 있으므로 뒤에 .을 찍어야한다. 원칙적으로는 가장 뒤에 .이 있어야하고, 우리가 찍지 않아도 소프트웨어에서 찍어준다.

- Fully Qualified Domain Name (FQDN)
  - DNS 프로토콜이 인정하는 도메인 이름 형식 – unique!
    - ambiguity 발생 방지
  - 주어진 노드로부터 root 까지 label들을 “.”로 구분하며 모두 읽은 것
  - 접미사 (suffix)를 생략하면 FQDN 아님
    - Resolver 자동 완성 설정 가능
      - `고급 TCP/IP 설정`에 들어가면 `DNS 설정` 탭이 있는데, .`다음 DNS 접미사 추가`에서 접미사가 suffix이다. 즉, 워하는 도메인 네임의 label만 쳐도 알아서 suffix를 찾아 더 붙여 resolve 해주는 기능

![]({{site.url}}/assets/images/175.png)

- IP 주소의 “.”과 도메인 이름의 “.”
  - 전혀 상관 없다 – 관계 지으려 하지 말 것
  - "각 label이 각 IP주소에 해당하는 거냐?" 이런 생각은 하지마.
- 최상위 도메인 (TLD) 이름
  - TLD = root의 child 노드들
  - 세 종류의 TLD가 있다.
  - 현재는 자유화가 되어 돈만 내면 TLD를 하나 만들 수 있다.

- Generic TLD (gTLD)
  - Com, net, edu, gov, int, mil, org에서 시작해서 확대

- Country-code TLD (ccTLD)
  - kr, kp(북한), tv(투발루), ch(스위스), cn(중국) 등 각 나라 당 1개
- Addressing and Routing Protocol Area (ARPA)
  - IP 주소 à 도메인 이름 번역 용도
  - 도메인 이름 외의 다른 identifier를 resolve하는 용도

- Addressing and Routing Protocol Area (ARPA)
  - **IP 주소 -> 도메인 이름** (역변환)  번역 용도
    - 예) 157.159.10.12에 대한 도메인 이름은 12.10.159.157.in-addr.arpa. 라는 도메인 이름의 resource record로 저장

![]({{site.url}}/assets/images/176.png)

- id-addr : IPv4 주소 -> Domain Name
  - id-addr 밑에서 시작하여 모든 child node들이 0 - 255
  - 157 -> 159 -> 10 -> 12 => 이 12라는 마지막 노드에 대해 저장된 Domain Name 정보를 찾는 쿼리를 DNS 서버에 날리면 된다.
  - 근데 이렇게 쿼리를 날리려면, 이 IP 주소를 Domain Name(FQDN)의 형태로 바꿔야 한다. => `12.10.159.157.in-addr.arpa.`

> **실습**
>
> - `nslookup` 입력 : Prompt 띄우기
>
> - `set querytype=PTR`
>
> - `1.1.152.163.in-addr.arpa.` 
>
>   => Domain Name 정보 나타남

- Addressing and Routing Protocol Area (ARPA)

  - 도메인 이름 외 **다른 identifier를 resolve**하는 용도
    - 전화번호의 **도메인 이름** -> URI
    - 예) +82-2123-45678 (이렇게 생긴 전화번호의 표준을 E.164라고 한다.) => 8.7.6.5.4.3.2.1.2.2.8.**e164.arpa.**
    - arpa 밑에는 e164라는 서브도메인이 있고, 이 밑으로는 전화번호 Domain Name에 해당하는 서브트리가 매달려있다.

  - 서브 도메인들 : in-addr, ip6, e164, uri ...
    - IANA에서 어떤 서브 트리들이 있는지 확인 가능

![]({{site.url}}/assets/images/177.png)

### 11.2 데이터베이스로서의 DNS

- 데이터베이스
  - Key = 도메인 이름 (FQDN)
  - 데이터 = **Resource Records (RRs)**
    - 하나의 key에 대하여 아주 다양한 레코드들이 매달려있다.
    - 예) IP 주소

- 분산 데이터베이스
  - 특정 도메인을 책임지는 기관이 해당 도메인의 서브트리에 해당하는 모든 데이터들을 zone file에 관리
    - 각 정보들은 물리적으로 분산되어있지만 논리적으로 연결되어있다. 
  - Zone file들의 연합체
    - 도메인이름 -> RRs 파일

![]({{site.url}}/assets/images/178.png)

- Zone file은 각 기관이 책임지고 관리
  - **Authoritative** name server - zone file을 관리하고, 쿼리에 대한 응답을 해주는 권위자
  - 고려대 전산 시스템 - `korea.ac.kr.` 밑의 모든 정보들을 한 묶음으로 **zone 파일**에 관리
  - KRNIC - kr 밑으로 오는 ac을 비롯한 다양한 도메인들의 zone file들을 연결 및 관리

- **Master copy**는 다른 누구도 책임져 주지 않음
  - 1997년 com과 net 도메인의 zone file corruption
    - 4시간 동안 전세계의 com과 net 도메인 접속 장애
    - 알아야 할 점은 DNS 기능이 없다고 해서 인터넷을 접속할 수 없는 것은 아니다. 다만 IP 주소를 모르는 사람이 대부분이기 때문에 문제가 발생하는 것이다.
  - 2002년 Microsoft.com과 msnbc.com 도메인 사고
    - Authoritative name server 인터넷에서 disconnect
      - 백업 서버가 있었지만 경로가 모두 원래 서버에 몰아넣어서 전혀 사용되지 못했다.
    - 해당 도메인 2일 동안 접속 장애

- 최근에는 여러 대의 authoritative name server 운용
  - 고대도 두 대 두고 있음
  - Primary 1 + 다수의 secondary
    - secondary도 authoritative이다.
- Zone 위임 (delegation)
  - 어떤 도메인 밑으로 존재하는 데이터가 너무 커질 시에 서브트리에 해당하는 zone file을 따로 떼어내서 위임함

![]({{site.url}}/assets/images/179.png)

- 도메인 이름 등록 (registration)
  - 해당 도메인의 zone file에 도메인 이름과 그 RR들을 삽입
    - **등록비 = zone file 관리비**

> Q : 그러면 [cs.korea.ac.kr](http://cs.korea.ac.kr/) DNS query를 처리하는데 root, kr, ac, korea 이 4가지의 authoritative name server에 접근해야하는건가요? 
> A : 원칙적으로는 그렇다.

### 11.3 RR의 종류

- 자주 쓰이는 RR 타입
  - A : IPv4
  - AAAA : IPv6 - 길이가 IPv4의 네 배라서
  - NS :
  - MX :
  - PTR :
  - SOA :
  - SRV :  서비스를 담당하는 서버의 이름

![]({{site.url}}/assets/images/180.png)

- nslookup(윈도우) / dig(리눅스)

### 11.4 DNS 프로토콜

- DNS 메시지 형식
  - Header
    - OP code
      - 0: 표준
      - 1: 역질의(폐기)
      - 2:서버상태 리포트 요청
      - 3: (미사용)
      - 4: 통고 (zone file 변경 -> secondary들은 요청하라)
      - 5: 갱신 (update)
  - question : 한 개 이상의 도메인이 variable의 형태로
  - answer (response)
  - authority (response) : 본인이 resolve할 수 없는 정보일 경우, 그 정보가 있는 다른 DNS 서버의 주소를 적어줌
  - additional (response)

![]({{site.url}}/assets/images/181.png)

![]({{site.url}}/assets/images/182.png)