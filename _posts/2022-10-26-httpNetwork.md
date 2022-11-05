---
title: "TCP"
excerpt: "웹 개발 공부의 시작 HTTP"

categories:
  - Web
tags:
  - ["HTTP", "Network"]

date_created: 2022-10-26T00:18:22+09:00
last_modified_at: 2022-10-26T00:18:22+09:00
toc: true
toc_sticky: true
---

# IP와 TCP 프로토콜
IP는 각 컴퓨터의 주소와 같은 것이다. 패킷 단위로 데이터를 전송하며 출발지 IP, 목적지 IP 등의 정보를 전송할 데이터와 함께 패킷에 담아 전송한다.

### IP의 한계
1. 비연결성 : 패킷을 받을 대상이 없거나 서비스 불능 상태이더라도 패킷을 전송함
2. 비신뢰성 : 전송 과정 중에 패킷이 유실되거나 패킷의 전송 순서가 뒤섞을 수 있음
3. 패킷을 전송 받은 컴퓨터에서 여러 프로세스가 실행 중이라면, 어떤 프로세스의 데이터인지 구분 불가

### TCP
IP의 한계를 보완하기 위해 함께 사용되는 프로토콜. **신뢰성있는** 데이터 전송을 지원하는 **연결 지항형** 프로토콜.

- 출발지, 목적지의 IP에 추가적으로 포트, 전송 순서 등의 정보를 포함하여 전송
	- 포트 : IP가 어떤 프로세스를 위한 전송인지 구분 불가한데, 이것을 구분해줌
- 연결 지향형 : 연결 설정 시 3 way handshake, 연결 해제 시 4 way handshake로 항상 성공적인 연결을 할 수 있음. 
- 신뢰성 있음 : 데이터 전송에 있어 순차 전송을 보장한다. 
- 흐름제어, 혼잡제어, 오류제어 지원
	- 흐름제어 : 송수신자 간의 처리속도 차이로 수신자가 처리할 수  있는 데이터 용량을 초과하면 데이터가 손실될 수 있음. 따라서 수신자 측에 맞추어 전송량을 조절
	- 혼잡제어 : 하나의 라우터에 데이터가 몰릴 경우 네트워크 혼잡으로 데이터 전송이 원활하지 않을 수 있음. 송신측에서 데이터 전송 속도를 강제로 낮추어 버림.
	- 오류제어 : 데이터가 유실되거나 잘못된 데이터가 수신되었을 경우에 대처하는 방법

#### 3 Way Handshake
![3way](/assets/img/2022-10-26-httpNetwork/3-way-handshake.png)
1. 클라이언트가 연결 요청 보냄 (SYN)
2. 서버가 요청을 수신하고 수신했음을 클라이언트에 알림 (SYN + ACK)
3. 클라이언트가 서버가 수신했다는 것을 확인했다 알림 (ACK)
이렇게 세 번 통신하여 연결을 확인하기 때문에 3 way handshake!

#### 4 Way Handshake
![3way](/assets/img/2022-10-26-httpNetwork/4-way-handshake.png)
1. 클라이언트가 연결을 종료하겠다는 요청을 보냄 (FIN)
2. 서버는 요청 받았다는 것을 클라이언트에게 확인시켜줌 (ACK)
	- 서버의 ACK를 받은 클라이언트는 FIN_WAIT 상태로 대기
3. 서버가 클라이언트의 요청에 따라 모두 종료되었음을 알림 (FIN)
4. 클라이언트는 다시 확인했다 알림 (ACK)

TCP 연결을 종료할 때 4 way handshake 사용
- 그런데 만약, 모종의 이유로 서버가 FIN 패킷을 보내기 전에 전송한 데이터가 FIN 패킷보다 늦게 클라이언트에 도착한다면?
	- 클라이언트는 서버의 FIN 패킷을 받고 이미 세션을 종료시키기 때문에 데이터 유실이 일어날 수 있음
	- 따라서 클라이언트는 FIN 패킷을 수신하더라도 일정 시간 동안 세션을 종료하지 않고 잉여 패킷을 기다린다 (TIME_WAIT)



### UDP
TCP가 신뢰성 있고 연결 지향의 전송 프로토콜이라면 UDP는 상대적으로 빠른 전송을 지원하며 부하가 적다.

- 상대적으로 신뢰성은 낮지만 전송 속도가 TCP에 비해 빠르다.
- 전송 계층에서 어떤 기능을 제공하기 보단 애플리케이션 계층에서 기능을 구현하여 사용하는 것을 추구
- 수신자의 수신 여부 확인 없이 (TCP는 3 way handshake로 연결 보장) 전송함, 일대다 통신도 지원