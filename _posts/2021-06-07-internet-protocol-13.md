---
layout: post
title: "[김효곤 교수의 인터넷 프로토콜] 13. Transmission Control Protocol (TCP)"
categories:
  - Internet Protocol
tags:
  - 인터넷, 프로토콜, 그리고 계층화 원칙


---

# Transmission Control Protocol (TCP)

### 13.0 TCP 개관

- 인터넷 트래픽의 90% 이상을 transport
  - 웹, 이메일, 게임, 스트리밍, SNS, …
- 인터넷 프로토콜 suite 중 가장 복잡한 프로토콜!
- **Full-duplex** 프로토콜
  - 무조건 1:1 통신
  - 데이터 채널이 **2개**
    - A->B + B->A
    - **동시 사용 중**일 수 있다: 패킷이 동시에 각각의 채널에 흐르고 있을 수 있음

### 13.1 TCP 서비스 모델

- **Reliable** delivery
  - 네트워크가 살아있는 한, 응용이 믿고 맡기는 신뢰적인 배달
  - 그런데 TCP가 타고 가는 그 IP는 믿을 만한 놈이 아니다.
    - IP가 패킷을 잃어버리는 경우 **TCP는 retransmission** 
- **In-order** delivery
  - **IP가 out-of-order delivery**를 하는 경우 **TCP가 reordering**한 이후 응용에게 배달
- **Flow** control
  - 수신 응용이 **받을 수 있는 만큼만** 배달
  - 슈퍼 컴퓨터 급 웹 서버와 5만원짜리 스마트폰이 통신하는 경우
    - 수신측에서 패킷들이 오버플로우가 되지 않도록, 수신측의 프로세싱 수준에 맞춰 느리게 보내줘야 한다.
- **Byte-stream**
  - 데이터는 **바이트의 연속**
    - 기다란 가래떡처럼
  - Segmentation
    - 데이터를 segment 단위로 잘라서 하나하나를 IP 패킷으로 만듦
    - **PMTU discovery** 이용
- **Connection-oriented**
  - **“연결”**을 맺어야 데이터를 전송할 수 있음
  - **“연결”** = **데이터 전송 관리**를 위한 **bookkeeping** 준비가 끝난 상태
    - 실제로 데이터를 전달하기 전에 양측이 이 전달에 대한 같은 이해를 갖도록 준비
    - 관리 = 재전송, flow-control, reordering, segmentation

___

### 13.2 TCP 패킷 형식

- 헤더 형식
  - Port 번호들 :  multiplexing/demultiplexing을 위함
  - Socket = <IP주소, 포트번호>
    - 응용을 가리키는 **identifier**가 된다. 
  - 연결 = **Socket과 socket의 연결**
    - 어떤 프로그램들 둘 사이에 TCP 연결을 만들라는 명령이 떨어지면, 커널 안에 **Buffer 두 개**가 만들어진다. 응용에서 내려올 데이터를 받아주는 버퍼(Send Socket Buffer)와 저쪽에서 온 데이터를 받아 응용에게 줄 버퍼(Receive Socket Buffer)가 생길 것이다.
    - 이외에도 **두 응용을 연결하는데 사용되는 다양한 자료구조**들이 생긴다.
  - HDL(Header Length) : 4비트 필드로 4바이트단위 (0-15)
  - Options : 고정 헤더 20바이트이므로 최대 40바이트까지
  - Window size : receive socket buffer의 빈 공간 "내가 데이터를 받을 수 있는 공간이 얼마나 남았다!" - flow control을 위함
  - Checksum : pseudo header를 사용
  - Urgent pointer : 송신자가 수신자에게 급한 데이터가 뒤쪽에 있음과 더불어, 급한 데이터까지 몇 바이트 남았음을 알림
  - Options : 안 쓰이는 옵션이 거의 없음

![]({{site.url}}/assets/images/223.png)

> **TCP와 IP의 헤더가 유사한 이유?**
>
> TCP와 IP는 예전에 하나의 프로토콜이었기 때문이다. 따라서 헤더의 길이까지 똑같지만 필드들은 제각각이다.

- Sequence number
  - Byte stream의 **각 byte마다 붙이는 일련 번호**
    - segment 단위가 아닌 **byte 단위**
  - 데이터 유실, 순서 뒤집힘 등을 알아내기 위한 용도
  - Payload의 **첫 번째 byte**의 sequence 번호를 헤더에 적음 (대표)

- Acknowledgement number
  - **수신자**가 송신자에게 "나 **어디까지** 받았어. **다음 것** 보내줘!"
  - 연속적으로 받은 마지막 byte의 sequence number **+ 1** (를 달라는 말)
  - ACK number -1번 바이트까지 수신측 소켓 버퍼에 **“안착”**했다는 뜻
    - 재전송 염려가 없다 -> **송신측 소켓 버퍼에서 지워도 됨** (즉, Ack가 올 떄까지 송신측 소켓 버퍼에서는 데이터가 지워지지 않는다는 뜻)
    - 만약 중간 데이터가 유실되었다면, 그 유실된 데이터가 시작되는 부분부터 끝까지 다시 달라고 요청
    - 지운 공간에는 새로 송신측 응용이 데이터를 부어줄 수 있음

- 헤더 길이
  - 4 바이트 단위
  - 가변 길이 옵션 때문에 

___

- Flags

  ![]({{site.url}}/assets/images/224.png)

  - CWR, ECN: **혼잡 제어 (congestion control)** 용도
  - URG: byte stream에 긴급한 데이터가 들어왔다는 뜻
    - 1 : Urgent Pointer 값 유효
    - 0 : Urgent Pointer 값 무효
  - ACK: Acknowledgement number가 유효한가
    - 1 : Ack 번호 필드 유효
    - 0 : Ack 번호 필드 무효
  - PSH: 현재는 단지 송신 소켓 버퍼가 비게 된 것만 표현
    - 지금은 빨리 보내려고 최선을 다하므로 전송 속도 면에서는 아무 의미 없음
    - 예전에는 응용이 급한 데이터임을 알리며 배달을 재촉할 수 있도록 요구
  - RST: Kill the connection
  - SYN: 연결 시작
  - FIN: 연결 해제

> Q : sequence number이 32비트면 40GB가 넘는 바이트스트림을 전송할 때 다시 0부터 시작하도록 하면 문제가 발생할 여지가 있지 않나요? (e.g. 앞부분 패킷이 네트워크 상에서 떠돌다가 뒤늦게 도착한 경우)
> A : 이론상으로는 가능하다. 그렇지만 현실적으로는 한바퀴 돈 숫자가 옛날 데이터와 헷갈리지 않도록 하는 옵션이 더 있으므로 거의 오류가 발생하지 않는다.

> **SYN과 FIN이 중요한 이유**
>
> SYN은 Synchronization의 약자이며, 이것은 시간을 맞춘다는 뜻이다. TCP에는 시간이라는 개념이 없다. 대신 미국방부에서는 작전을 수행하기 전에 원격에서 서로의 시간을 완벽히 맞추는 과정이 필요했으며 이 개념이 TCP에 반영된 것으로 보인다. TCP는 데이터를 주고 받기 전에 연결을 맺으며, 마치 국방부에서 작전 수행 전/후 시간을 맞추는 것처럼 서로 데이터를 나누기 전/후에 무언가를 맞추는 것을 말한다. 이 역할을 수행하는 것이 SYN과 FIN이 된다.

> **FIN과 RST의 차이**
>
> FIN은 정상적인 절차에 따라 해제하는 것이고, RST는 abortion이며, 비정상적으로 급히 끊는 것을 말한다. 

___

- Window size
  - ACK number 기준 수신 소켓 버퍼의 빈 공간 크기
  - Flow control을 위함 -> ACK number부터 몇 바이트를 더 받을 수 있는가
- 체크섬
  - Pseudo-header 사용에 유의
  - TCP는 의무!
- Urgent pointer
  - Byte stream에 urgent data가 들어왔음
  - 그 위치는 뒤쪽으로 몇 바이트인지 표시 

> Q : URG포인터는 급한 segment 앞에 있는 한 패킷만 하면 되나요? 아니면 앞에 모든 패킷이 가리키나요? 
> A : 앞에 있는 모든 패킷이 가리킨다. 근데 상식적이라면 급한 것들을 제일 앞에 둬야한다. 그런데 그냥 줄 뒤에 서있으며 급한 것이 있다고 말하기만 한다.
>
> Q : 데이터가 급한지 아닌지의 여부는 송신자측에서 정하는건가요? 
> A : 그렇다. 추가적으로는 TCP는 단순히 짐을 옮기는 직원이기 때문에 TCP는 급한게 있다고 해도 아무동작을 취할 순 없으며, 그냥 수신측 응용에게 이것이 급한 데이터라고 알려주는 일 밖에 하지 않는다. 수신측 응용은 아마도 송신측에서 말하는 '급하다'의 정확한 의미를 알고 있을 것이다.

___

- 옵션: **모두 <u>필요</u>** (IP 옵션들과는 다름)

  - EOL: **잘 안 쓰임** (비슷한 역할을 NOP이 해주고 있음)
  - NOP: 4 byte align 용도
    - 아래 표에 보면 4Byte align이 안되어있는 옵션들이 있다. (IPv4의 헤더는 4바이트 배수로 딱 맞춰야한다.)
    - 안맞는 경우에는 1바이트짜리 옵션을 갖고 4바이트의 배수가 되도록  레고블록 끼우듯이 맞춰서 4바이트 배수를 맞춘다.
  - MSS : 처음에 연결을 맺을 때 특정 패킷 사이즈를 사용하자고 제안.
  - SACK : Selective ACK를 쓸 수 있다는 의미

  ![]({{site.url}}/assets/images/226.png)

- 예)

  - MSS : 1460
  - Window Scale : 윈도우 사이즈는 필드 크기가 16비트 밖에 안되기 때문에 총 64KB까지 표현이 가능하다. 그런데 이것보다 실제로 더 많은 크기의 빈공간이 남아있는 경우에는 윈도우 사이즈 필드에 있는 값에 얼마를 곱한 것(왼쪽으로 몇 칸 이동)이 실제 값이라는 의미의 수가 Option에 추가한다.
  - 이렇게 사용된 Options의 데이터 크기를 다 더하면 9바이트인데 4의 배수를 채워야 하므로 NOP과 EOL를 사용하거나 보통은 NOP만을 12바이트가 될 때까지 아무데나 끼워넣는다.
  
  ![]({{site.url}}/assets/images/225.png)

___

- Maximum Segment Size (MSS): peer에게 어떤 payload 크기를 쓸 것인지 알림
  - Path Cover Delivery 경로 중 자신이 속한 첫번째 링크의 MTU만 알고 있는 상태 : MTU – 40 bytes
    - 이때, MSS 값은 TCP의 Payload 값에 해당
    - 양방향 채널을 통해 양측이 서로(peer라고 칭함)에게 MSS 값을 알려준다.
    - 양측이 보낸 두 개의 값에서 더 작은 값을 택함으로써 두 개의 채널에서 같은 Segment Size를 사용하게 함
  - 최초 값: PMTU discovery에 의해 더 작은 값으로 추후 교정될 수 있음

- Window Extension
  - 16-bit window size 필드 -> window 크기 <= 65535
  - 64KB 보다 receive socket buffer를 크게 사용할 때 
  - 옵션 값 n = left shift by n positions = X 2^n^

- Selective Acknowledgement (SACK)는 불연속적으로 패킷을 수신할 때 미수신부분만을 재전송 요청하는데 쓰임

![]({{site.url}}/assets/images/227.png)

![]({{site.url}}/assets/images/228.png)