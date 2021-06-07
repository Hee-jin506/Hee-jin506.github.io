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

  - CWR, ECN: 혼잡 제어 (congestion control) 용도
  - URG: byte stream에 긴급한 데이터가 들어왔다는 뜻
    - 1 : Urgent Pointer 값 유효
    - 0 : Urgent Pointer 값 무효
  - ACK: Acknowledgement number가 유효한가
    - 1 : Ack 번호 필드 유효
    - 0 : Ack 번호 필드 무효
  - PSH: 현재는 단지 송신 소켓 버퍼가 비게 된 것만 표현
    - 지금은 빨리 보내려고 최선을 다하므로 아무 의미 없음
    - 예전에는 응용이 급한 데이터임을 알리며 배달을 재촉할 수 있도록 요구
  - RST: Kill the connection
  - SYN: 연결 시작
  - FIN: 연결 해제

> Q : sequence number이 32비트면 40GB가 넘는 바이트스트림을 전송할 때 다시 0부터 시작하도록 하면 문제가 발생할 여지가 있지 않나요? (e.g. 앞부분 패킷이 네트워크 상에서 떠돌다가 뒤늦게 도착한 경우)
> A : 이론상으로는 가능하다. 그렇지만 현실적으로는 한바퀴 돈 숫자가 옛날 데이터와 헷갈리지 않도록 하는 옵션이 더 있으므로 거의 오류가 발생하지 않는다.

