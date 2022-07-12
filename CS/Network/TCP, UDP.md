# TCP, UDP
* [TCP](#TCP)
* [UDP](#UDP)
<img src="https://github.com/InJun2/TIL/blob/main/CS-topic/network/img/TCP-UDP.png" width="600" height="300">

# TCP
TCP(Transcission Control Protocol)는 네트워크 계층 중 전송 계층에서 사용하는 프로토콜로서, **신뢰성을 보장하는 연결형 서비스**이다.   
TCP는 네트워크에 연결된 컴퓨터에 실행되는 프로그램 간에 옥텟(데이터, 메세지, 세그먼트)를 ***안정적으로, 순서대로, 에러없이 교환***할 수 있게 한다.

## TCP 특징
* 연결형 서비스 
  * 3-way handshaking 방식(SYN, ACK)
  * 4-way handshaking 방식(FIN, ACK)
* 흐름제어(Flow control)
  * 데이터 처리 속도를 조절하여 수신자의 버퍼 오버플로우 방지
  * 송신하는 곳에서 많은 데이터를 빠르게 보내는 경우 수신하는 곳에서 데이터 유실 발생
  * 따라서 수신자가 `윈도우크기(Window Size)` 값을 통해 수신량을 제어한다.
* 혼잡제어(Congestion control)
  * 네트워크 내 패킷 수가 과도하게 증가하는 현상 방지
* 신뢰성 보장
  * Sequence Number, Ack Number를 통한 신뢰성 보장
  * Dupack-based retransmission: ACK값이 중복으로 올 경우 패킷 이상을 감지하고 재전송 요청
  * Timeout-based retransmission: 일정시간동안 ACK값을 수신하지 못할 경우 재전송 요청
* 전이중(Full-Duplex), 점대점(Point to Point) 서비스
  * 전이중: 전송이 양방향으로 동시에 일어날 수 있다.
  * 점대점: 각 연결이 정확히 2개의 종단점을 가지고 있다.
  * 즉, 일대일 연결로서 멀티캐스팅이나 브로드캐스팅을 지원하지 않는다.
* UDP보다 전송속도가 느리다.
* 파일 전송과 같이 손상이 없어야하는, 신뢰성있는 전송이 필요한 경우 사용

## TCP 연결 및 해제
### 3-way handshaking 방식(SYN, ACK)
<img src="https://t1.daumcdn.net/cfile/tistory/9957DF345CD2DB0233" width="400" height="350">

1. Client에서 Server에 연결 요청을 하기위해 SYN 데이터를 보낸다.
2. Server에서 해당 포트는 LISTEN 상태에서 SYN 데이터를 받고 SYN_RCV로 상태가 변경된다.
3. 그리고 요청을 정상적으로 받았다는 대답(ACK)와 Client도 포트를 열어달라는 SYN 을 같이 보낸다.
4. Client에서는 SYN+ACK 를 받고 ESTABLISHED로 상태를 변경하고 서버에 ACK 를 전송한다.
5. ACK를 받은 서버는 상태가 ESTABLSHED로 변경된다. 


### 4-way handshaking 방식(FIN, ACK)
<img src="https://t1.daumcdn.net/cfile/tistory/997BBD395CD2E5491B" width="500" height="350">

1. 통신을 종료하고자 하는 클라이언트가 서버에게 FIN을 보낸 후, `FIN_WAIT 1`상태로 대기한다.
2. FIN을 받은 서버는 상태를 `CLOSE_WAIT`로 변경하고, 응답으로 ACK을 보낸다.
   해당 ACK을 받은 클라이언트는 상태를 `FIN_WAIT 2`로 변경한다.
   이와 동시에 서버는 해당 포트에 연결되어 있는 애플리케이션에게 CLOSE()를 요청한다.
3. CLOSE() 요청을 받은 애플리케이션은 종료 프로세스를 통해 최종적으로 close되고, 
   서버는 `FIN`을 클라이언트에게 보낸 후 상태를 `LAST_ACK`으로 변경한다.
4. 클라이언트가 FIN_WAIT 2 상태에서 서버로부터 FIN을 받으면 이에 대한 응답으로 ACK을 보낸 후 상태를 `TIME_WAIT`으로 변경한다.
5. 최종 ACK을 받은 서버는 `CLOSED`로 상태를 변경하고, 클라이언트도 일정 시간이 지나면 `CLOSED`로 상태를 변경한다.

> 반드시 서버만 `CLOSE_WAIT` 상태를 갖는 것은 아니다.  
> 서버가 먼저 종료 요청 FIN을 보낼 수 있고, 이 경우 서버가 `FIN_WAIT1` 상태가 된다.  
> 누가 먼저 `CLOSE`를 요청하느냐에 따라 상태가 달라질 수 있다.  

# UDP
UDP(User Datagram Protocol)는 전송계층의 **비연결 지향적 프로토콜**이다.   
연결과정이 없기 때문에 TCP보다는 빠른 전송을 할 수 있지만, 데이터 전달의 신뢰성은 떨어진다.     
UDP는 발신자가 데이터 패킷을 순차적으로 보내더라도 이 패킷들은 서로 다른 통신 선로를 통해 전달 될 수 있다.   
먼저 보낸 패킷이 느린 선로를 통해 전송될 경우 나중에 보낸 패킷보다 늦게 도착할 수 있으며 잘못된 선로로 전송되어 유실될 수도 있다.   
즉, UDP는 패킷의 순서를 보장하지 않고, 중간에 패킷이 유실되거나 변조가 되어도 재전송 하지 않는다.

## UDP 특징
* 비연결형 서비스로 연결 없이 통신이 가능하며 데이터그램(Datagram) 방식을 제공한다.
* 데이터 경계를 구분한다. (데이터그램 서비스)
* 정보를 주고 받을 때 정보를 보내거나 받는다는 신호절차를 거치지 않는다.
* 신뢰성 없는 데이터를 전송한다.
  * 데이터 재전송, 데이터 순서 유지를 위한 작업을 하지 않는다.
* 패킷관리가 필요하다.
* 패킷 오버헤드가 적어 네트워크 부하가 감소되는 장점이 있다.
* 상대적으로 TCP보다 전송속도가 빠르다.
* 속도가 우선시 되는 상황에서 사용, 영상 스트리밍 등에 사용

## UDP 헤더
* 송신자의 포트 번호
* 수신자의 포트 번호
* 데이터의 길이
* 에러 검출 코드(Checksum)
> 데이터 신뢰성 보장을 위한 헤더값(Sequense Number, ACK Number ..)은 없다.
