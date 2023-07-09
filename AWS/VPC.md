# Network 개념
**사설 IP**
* 한정된 IP 주소를 최대한 활용하기 위해 IP주소를 분할하고자 만든 개념 (IPv4에만 적용)
* 사설망 내부 기기들은 외부 네트워크로 통신이 불가능한 사설 IP로 구성
* 외부로 통신할 때는 통신 가능한 공인 IP를 활용
* 따라서 하나의 망에는 사설 IP를 부여받은 기기들과 NAT 기능을 갖춘 Gateway로 구성

**NAT: Network Address Translation**
* 사설 IP가 공용 IP로 통신할 수 있도록 주소를 변환해주는 방법
* `Dynamic NAT`: 1개의 사설 IP를 가용 가능한 공인 IP로 연결
* `Static NAT`: 하나의 사설 IP를 하나의 공인 IP로 연결, AWS Internet Gateway가 사용하는 방식
* `PAT(Port address translation)`: 많은 사설 IP를 하나의 공인 IP로 연결
  * port로 분류해 하나의 공인 IP 사용, NAT Gateway가 사용하는 방식  
<img src="https://github.com/twoosky/TIL/assets/50009240/94a42e5e-935c-4099-978b-89976bc78d60" width="800" height="320">
<br></br>

**CIDR: Classless Inter Domain Routing**
* IP 주소의 영역을 여러 네트워크 영역으로 나누기 위한 IP 주소 할당 방법
* 여러 개의 사설망을 구축하기 위해 네트워크 영역을 나누는 방법
* ex) 192.168.2.0/24
  * IP는 총 32비트, 위 IP주소는 24비트는 네트워크 주소, 나머지 8비트는 호스트 주소를 표현한다.
  * 총 2*8 = 256개의 호스트 주소(사설IP)를 갖는 네트워크이다.
  * 192.168.2.0 ~ 192.168.2.255까지 호스트를 표현 가능
 
# VPC: virtual private cloud
* 사용자의 AWS 계정 전용 가상 네트워크
* 외부와 격리된 가상의 네트워크 환경이다.
* AWS VPC는 리전 단위이다.
* VPC는 AWS 클라우드에서 다른 가상 네트워크와 논리적으로 분리되어있다.
* `VPC 구성요소`: 서브넷, 인터넷 게이트웨이, NACL/보안그룹, 라우트 테이블, NAT Gateway, Bastion Host, VPC Endpoint

## 1. 서브넷: Subnet
* VPC의 하위 단위로 VPC에 할당된 IP를 더 작은 단위로 분할한 개념
* `하나의 서브넷은 하나의 가용영역(AZ)안에 위치`
* CIDR block range로 IP 주소 지정
* AWS 서브넷의 사용 가능한 IP 개수는 5개를 제외하고 계산
  * ex) 10.0.0.0/24
  * 10.0.0.0: 네트워크 어드레스 용도
  * 10.0.0.1: VPC Router
  * 10.0.0.2: DNS Server
  * 10.0.0.3: 미래에 사용을 위해 남겨둠
  * 10.0.0.255: 네트워크 브로드캐스트 어드레스
  * 따라서, 사용가능한 IP 개수는 2^8-5 = 251개

**서브넷 종류**
* `퍼블릭 서브넷` : 외부에서 인터넷을 통해 연결할 수 있는 서브넷
  * Internet Gateway를 통해 외부의 인터넷과 연결되어 있다.
  * 퍼블릭 서브넷 내 인스턴스에 퍼블릭 IP 부여 가능
  * 웹서버, 애플리케이션 서버 등 유저에게 노출되어야 하는 인프라
* `프라이빗 서브넷` : 외부에서 인터넷을 통해 연결할 수 없는 서브넷
  * 외부 인터넷으로의 경로도 없고, 퍼블릭 IP 부여 불가능
  * DB 등 외부에 노출될 필요가 없는 인프라

## 2. 라우팅 테이블: Route Table
* 네트워크 요청이 어디로 가야할지 알려주는 테이블
* 서브넷에서 나가는 트래픽의 destination(IP 대역), target(local, IGW, NAT ..)을 정의
* VPC 생성시 기본으로 하나 제공

<img src="https://github.com/twoosky/TIL/assets/50009240/aff58fb5-7f8c-48ea-beef-938a51b62d02" width="800" height="380">


## 3. 인터넷 게이트웨이: Internet Gateway
* VPC가 외부의 인터넷과 통신할 수 있도록 경로를 만들어주는 리소스
* 기본적으로 확장성과 고가용성이 확보되어 있다.
* IPv4, IPv6 지원 (IPv4의 경우 NAT 역할 수행 - static NAT)
* Route Table에서 경로 설정 후 접근 가능
* 외부로 나가는 Internet Gateway가 연결된 서브넷을 퍼블릭 서브넷이라 한다.
<img src="https://github.com/twoosky/TIL/assets/50009240/ab487fc9-be78-4b8d-997b-50a0c4ce6385" width="800" height="330">

## 4. NACL과 Security Group
* NACL과 Security Group은 방화벽과 같은 역할을 하며, 인바운드 및 아웃바운드 트래픽 보안정책을 설정할 수 있다.
* 인바운드: AWS 리소스로 들어오는 요청
* 아웃바운드: AWS 리소스에서 외부로 나가는 요청

**Security Group**
* `인스턴스` 단위로 적용
* Port의 허용만 가능
* Stateful: 인바운드 트래픽의 보안 정책만 설정
* 인스턴스간 서브넷이 같은 경우 Security Group만 적용

**NACL(Network Access Control List)**
* `서브넷` 단위로 적용
* Port의 허용 및 거부 가능
* 등록된 규칙의 번호순으로 트래픽 허용 및 거부
* Stateless: 인바운드/아웃바운드 트래픽의 보안 정책을 설정
* 인스턴스간 서브넷이 다른 경우 NACL 적용

<img src="https://github.com/twoosky/TIL/assets/50009240/c7d56fea-34fb-47bf-a875-ddadb17a97f9" width="800" height="380">

## 5. NAT Gateway
* VPC의 private subnet에 있는 인스턴스에서 외부의 인터넷과 통신할 수 있도록 지원하는 AWS 서비스
  * NAT Gateway가 없다면, private subnet 내 인스턴스에서 외부로부터 패키지를 다운받을 수 없음 
* public subnet에 위치
* NAT Gateway에는 탄력적 IP를 할당 (고정된 IP)
* private subnet의 route table에 0.0.0.0/0 IP 대역과 NAT Gateway 타깃을 추가

<img src="https://github.com/twoosky/TIL/assets/50009240/130c1580-d041-46d1-b2d6-d1748b9286db" width="800" height="380">

<img src="https://github.com/twoosky/TIL/assets/50009240/c9a77187-40af-49bd-a150-c03822736888" width="800" height="350">

## 6. Bastion Host
* 외부에서 사설 네트워크에 접속할 수 있도록 경로를 확보해주는 서버
* private subnet 내 인스턴스에 접근하기 위한 EC2 인스턴스
* public subnet에 위치

<img src="https://github.com/twoosky/TIL/assets/50009240/b7fbed2f-c935-453d-b7c8-a44c824734ef" width="800" height="380">
