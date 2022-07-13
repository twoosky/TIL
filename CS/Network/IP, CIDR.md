# IP, CIDR
* [IP란](#IP란)  
* [IP 구조](#IP-구조)
* [IP 주소 클래스](#IP-주소-클래스)
* [CIDR](#CIDR)

## IP란
* IP란 인터넷 프로토콜(Internet Protocol)의 약자로 인터넷상에서 데이터를 주고 받기 위한 통신 규약이다.    
* 패킷(Packet) 단위로 데이터 통신이 이루어진다.  
* OSI7 Layer의 네트워크계층(3계층)에 해당한다.  
* 비연결성, 비신뢰성
  * 신뢰성 보장을 위해 전송계층(4계층)에 의존한다.
* 호스트 주소 지정과 패킷 분할 및 조립 기능을 담당

## IP 구조
<img src="https://user-images.githubusercontent.com/50009240/178791346-944af763-baf3-4f1b-9ddd-fb6b31b4a204.png" width="500" height="100">

* IP header: 목적지와 출발지의 IP 주소 등을 포함
* Payload(페이로드): 전송되는 데이터

### IPv4 헤더 구조
<img src="https://user-images.githubusercontent.com/50009240/178792575-452719e2-b56f-43b5-bb5c-9aca476aebfd.png" width="500" height="250">

* Version : IP의 버전, IPv4
* Header Length : 헤더의 길이
  * 4-byte 단위로 최소 5(20-byte)부터 최대 15(60-byte)의 값을 갖는다.
* Type of Service : 서비스 품질
* Total Packet Length : IP 패킷의 전체 길이를 바이트 단위로 표시한 값(최대 65,535)
* Identifier, Flag, Offset : IP Fragment 필드를 단편화와 재조합하는데 사용되는 값
  * 한번에 보내기 큰 패킷을 작은 단위로 전송되는 경우 사용
* Time to LiveTTL : IP 패킷의 수명
* Protocol ID : 데이터에 포함되는 상위 계층의 프로토콜 정보
  * (TCP : 6, UDP : 17)
* Header Checksum : 오류 검출 코드
* Source/Destination IP Address : 출발, 목직지 IP 주소
* IP Hedaer Options & Padding : 옵션, 거의 사용되지 않고 테스트 및 디버깅 용도로 사용되고 통신에는 관여하지 않는다.

> **IPv4 주소**    
> IP version 4 주소, 줄여서 IPv4 주소는 오늘날 일반적으로 사용하는 IP 주소이다. 
> 이 주소의 범위는 32비트로 보통 0~255 사이의 10진수 넷을 쓰고 .으로 구분하여 나타낸다. 
> 따라서 0.0.0.0부터 255.255.255.255까지가 된다. 
> 이론적으로 42억9496만7296개의 IP가 존재한다. 
> 중간의 일부 번호들은 특별한 용도를 위해 예약되어 있다. 
> ex) 127.0.0.1은 localhost(로컬 호스트)로 자기 자신을 가리킨다.

> **IPv6 주소**    
> 인터넷 사용자가 급등하며, IPv4의 주소가 부족해 등장
> IPv6에서는 주소 길이를 128비트로 늘려 사용가능한 주소의 갯수가 2의 128제곱개 정도 된다.(약 43억x43억x43억x43억개... )
> IPv6 주소는 보통 두 자리 16진수 여덟 개를 쓰고 각각을 : 기호로 구분한다.

## IP 주소 클래스
* IP 주소는 네트워크 크기에 따라 5개의 클래스 A, B, C, D, E로 구분
* IP 주소는 `네트워크`과 `호스트` 뷰분으로 구분된다.
  * `네트워크`는 브로드캐스트 영역
  * `호스트`는 개별 단말기 영역

<img src="https://user-images.githubusercontent.com/50009240/178797264-4749d7c2-78d1-4dce-98ef-03842303b352.png"
 width="600" height="350">

* Class A
  * Network가 가질 수 있는 Host가 가장 많다.
  * Network(1Byte), Host(3Byte)로 구성된다.
* Class B
  * Network(2Byte), Host(2Byte)로 구성
  * Network 영역의 비트가 10으로 시작한다.
* Class C
  * Network(3Byte), Host(1Byte)로 구성된다.
  * Network 영역의 비트가 110으로 시작한다. 

|클래스|첫 고정 Bit|네트워크 주소영역|호스트 주소영역|
|---|---|---|---|
|A class|	0|	8bit(2^7개)|	24bit(2^24-2개)|
|B class|	10|	16bit(2^14개)|	16bit(2^16-2개)|
|C class|	110|	24bit(2^21개)|	8bit(2^8-2개)|

* 문제점
  * 영역을 나누어서 효율적으로 관리할 수 있으나, 각 Class마다 Host개수의 차이가 너무 크다.
  * Host 영역이 과도하게 남는 경우가 발생한다.
    * C클래스를 사용하기에는 Host영역이 부족하고, B클래스를 사용하기에는 Host영역이 너무 많이 남는 경우
  * 따라서 유연한 네트워크/호스트 영역 구분을 위해 CIDR 탄생 

## CIDR
* IP Class를 대체
* Class 체계보다 더 유연하게 IP를 나눌 수 있다.
  * Network 영역과 Host 영역이 Class로 고정되어 있지 않으니, Host를 유동적으로 나눌 수 있다.
* IP와 서브넷 마스크를 함께 표기한다.
  * IP/서브넷 마스크(210.77.8.155/26)
  * 서브넷은 앞에서부터 bit 1의 개수이고, Network영역이다.
  * 나머지 bit 0의 개수는 Host 영역이다.
> CIDR 표기법  
> 1. Network 영역을 모두 1로, Host 영역을 모두 0으로 표시
> 2. IP(네트워크 주소)/(Network영역의 비트 수) 로 표기

### 서브넷(Subnet)
* 하나의 네트워크가 분할되어 나눠진 작은 네트워크  
* 클래스 단위로 분류하게 되면 적절한 네트워크의 크기로 구분할 수 없기 때문에 서브넷으로 분할하여 사용한다.  
* 네트워크를 분할하는 것을 `서브넷팅(Subneting)`이라고 하고, `서브넷 마스크(Subnet Mask)`를 통해 수행한다.

### 서브넷 마스크(Subnet Mask)
* `네트워크`영역은 bit 1, `호스트` 영역은 bit 0으로 나타내는 표기법.
```
2진수: 11111111 11111111 11111111 1111 0000 
10진수: 255.255.255.240 
```
<img src="https://velog.velcdn.com/cloudflare/redgem92/19d6535a-ca37-4b48-acb6-499644ef224b/image.png" width="550" height="150">

> **IP `8.8.8.114`, 서브넷 마스크 `255.255.255.192`의 네트워크 주소와 호스트 개수는?**
> * 네트워크 주소 -> 8.8.8.64
> * bit 0의 개수가 6개이므로 호스트 64개 할당 가능
> * `CIDR(Prefix) 표기`: 8.8.8.64/26
>   * 26은 앞서 있는 bit 1의 개수 즉, 네트워크 영역  












