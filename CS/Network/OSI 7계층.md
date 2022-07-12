# OSI 7계층
<img src="https://user-images.githubusercontent.com/50009240/178475235-31729cc5-d509-42cf-b544-2b6c90ac1688.png" width="700" heigh="1000"/>  

### OSI 7계층이란
* ISO(국제표준기구)에서 네트워크의 통신 과정을 7단계로 나눠 표준화한 것  
* 전송 시 7계층에서 1계층으로 각각의 층마다 필요한 정보를 담은 헤더를 붙임(캡슐화)   
* 수신 시 1계층에서 7계층으로 헤더를 떼어냄(디캡슐화)  

### 7계층 분리 이유
통신이 일어나는 과정을 단계별로 알 수 있고, 문제 발생 시 해당 단계의 장비와 SW만 수정하면 되므로 문제 해결에 용이하다.

### 1. Physical(Bit) 
* 물리적으로 연결된 계층으로 데이터를 전기적 신호로 변환해 주고 받음  
* 데이터 송수신 기능 제공
* Bit(0,1)로 데이터 전송  
* 주요 장비: 리피터, 케이블, 허브 

### 2. Link(Frame)
* `peer to peer` 간의 신뢰성 있는 데이터 전송 보장  
* MAC주소(네트워크상에서 하드웨어를 구분하는 고유한 주소)를 통한 통신  
* 물리계층에서 발생한 오류를 검출 및 재전송 기능 존재  
* Frame(프레임) 단위의 통신
  * 네트워크 계층에서 받아온 Datagram(Packet)을 프레임 단위로 만들고 헤더와 트레일러 추가  
  * 헤더(Header):  목적지, 출발지 MAC주소  
  * 트레일러(Trailer):  데이터 에러 감지 비트  
* 주요 장비: 브리지, 스위치
* 주요 프로토콜: 이더넷, BlueTooth, Wi-Fi, ARP

### 3. Network(Packet)
* `end to end` 간 데이터 전달
* Packet 단위의 데이터 전송
* `패킷 포워딩(Packet forwarding)`과 `라우팅(Routing)` 수행
  * Datagram forwarding: Packet 헤더에는 출발, 목적지 IP 주소가 있어 이를 기반으로 패킷을 목적지까지 전달
  * Routing: 네트워크 간 라우터를 통해 목적지까지 최적의 경로를 선택하는 과정

### 4. transport(Segment)
* `end to end` 간의 신뢰성 있는 데이터 전송 보장
* TCP -> Segment(세그먼트) 단위의 통신
* UDP -> DataGram(데이터그램) 단위의 통신
* Port 번호를 통해 해당하는 application socker으로 데이터 전송


* TCP
  * 헤더: 출발, 목적지 Port, 시퀀스 번호, 에러 검출 코드(Checksum)
  * 3-way handshaking 과정을 통해 연결을 설정
  * 4-way handshaking 을 통해 연결을 해제
  * 신뢰성 있는 데이터 전송(데이터의 재전송 존재)
  * 일대일 통신

* UDP
  *  







