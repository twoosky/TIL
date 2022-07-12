# OSI 7계층
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/8/8d/OSI_Model_v1.svg/870px-OSI_Model_v1.svg.png" width="400" heigh="500"/> 

### OSI 7계층이란
* ISO(국제표준기구)에서 네트워크의 통신 과정을 7단계로 나눠 표준화한 것  
* 전송 시 7계층에서 1계층으로 각각의 층마다 필요한 정보를 담은 헤더를 붙임(캡슐화)   
* 수신 시 1계층에서 7계층으로 헤더를 떼어냄(디캡슐화)
* `PDU`: 각 계층에서 전송되는 단위

### 7계층 분리 이유
통신이 일어나는 과정을 단계별로 알 수 있고, 문제 발생 시 해당 단계의 장비와 SW만 수정하면 되므로 문제 해결에 용이하다.

### 1. 물리계층: Physical Layer 
* 물리적으로 연결된 계층으로 데이터를 전기적 신호로 변환해 주고 받음  
* 데이터 송수신 기능 제공
* PDU: 비트(Bit) 
* 주요 장비: 리피터, 케이블, 허브 

### 2. 링크계층: Link Layer
* `peer to peer` 간의 신뢰성 있는 데이터 전송 보장  
* MAC주소(네트워크상에서 하드웨어를 구분하는 고유한 주소)를 통한 통신  
* 물리계층에서 발생한 오류를 검출 및 재전송 기능 존재  
* PDU: 프레임(Frame) 단위의 통신
  * 네트워크 계층에서 받아온 Datagram(Packet)을 프레임 단위로 만들고 헤더와 트레일러 추가  
  * 헤더(Header):  목적지, 출발지 MAC주소  
  * 트레일러(Trailer):  데이터 에러 감지 비트  
* 주요 장비: 브리지, 스위치
* 주요 프로토콜: 이더넷, BlueTooth, Wi-Fi, ARP

### 3. 네트워크계층: Network Layer
* `end to end` 간 데이터 전달
* `패킷 포워딩(Packet forwarding)`과 `라우팅(Routing)` 수행
  * Packet forwarding: Packet 헤더에는 출발, 목적지 IP 주소가 있어 이를 기반으로 패킷을 목적지까지 전달
  * Routing: 네트워크 간 라우터를 통해 목적지까지 최적의 경로를 선택하는 과정
* PDU: 패킷(Packet)

### 4. 전송계층: transport Layer
* `end to end` 간의 신뢰성 있는 데이터 전송 보장
* Port 번호를 통해 해당하는 application socker으로 데이터 전송
* PDU: 세그먼트(Segment)
* 주요 장비: 게이트웨이
* 주요 프로토콜: TCP, UDP
* TCP/UDP 차이점
<img src="https://user-images.githubusercontent.com/50009240/178518649-d3f2dd86-3198-41fd-a058-6489de662d6f.png" width="600" heigh="400"/>   

> Q. 2계층에서 신뢰성을 보장해주는데 4계층에서 왜 또 신뢰성 보장 해줘야 하나?  
> 1) 중간에 Router가 데이터를 제대로 받고 죽은 경우, 2계층에선 문제가 없지만 데이터 전송은 끊김  
> 2) Router 사이 loop가 발생하면 데이터 손실 발생

### 5. 세션계층: Session Layer
* 데이터가 통신하기 위한 논리적 연결을 담당
* 애플리케이션 간의 TCP/IP 세션을 구축하고 관리하며 종료시키는 계층
* 통신 장치 간 상호작용 및 동기화를 제공
* 연결 세션에서 데이터 교환과 에러 발생 시의 복구를 관리

### 6. 표현계층: Presentation Layer
* 애플리케이션의 데이터 형태와 구조를 변환(번역, 암호화, 압축)시키는 계층
* 응용계층이 Data를 이해할 수 있게 다양한 포멧으로 데이터 변환
  * 사용자 시스템에서 데이터의 형식상 차이를 다루는 부담을 응용 계층으로부터 덜어줌

### 7. 응용계층: Application Layer
* 사용자에게 실제 애플리케이션 서비스를 제공하는 계층
* 주요 프로토콜: DHCP, HTTP, DNS, FTP, SMTP

<img src="https://user-images.githubusercontent.com/50009240/178475235-31729cc5-d509-42cf-b544-2b6c90ac1688.png" width="550" heigh="800"/> 
   









