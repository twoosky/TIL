# ELB: Elastic Load Balancer
* 외부/내부에서 들어오는 클라이언트의 요청을 ELB에 연결된 가용영역에 있는 EC2와 같은 대상에 분산하고, 등록된 대상의 상태를 검사하는 서비스
* AWS의 L4 / L7 로드밸런싱 (서버 부하분산) 서비스
* 외부/내부 트래픽은 ELB의 DNS 주소를 목적지로 하여 들어온다. (IP 고정 불가능하므로 DNS로 접근)

## ELB 구성요소
* **대상 그룹**: Target Group
  * ELB가 트래픽을 전달할 대상의 집합
  * 대상 유형에는 EC2 인스턴스, IP(private IP), Lambda, ALB, 프로토콜(HTTP, HTTPS)이 있다.
  * Target Group에 속한 대상에 대해 주기적으로 확인하는 프로세스(=Keepalive)를 통해 Health Check 수행
* **리스너**: Listener
  * 프로토콜과 포트를 기반으로 요청에 대한 대상 그룹을 지정한다.
  * 외부에서 80 port로 요청하면 지정된 대상 그룹으로 트래픽 분산

## ELB 종류
**Application Load Balancer** [[참고]](https://aws-hyoh.tistory.com/134)
* L7 로드밸런서, HTTP/HTTPS Header 정보를 기반으로 요청을 전달하는 로드밸런서
* 경로, HTTP 헤더, HTTP 요청 메서드 등의 규칙을 적용하여 적절한 대상 그룹으로 요청 전달 가능
* IP 주소가 유동적으로 변동되므로 DNS 주소로 접근해야 한다
* 보안 그룹 적용 가능
* ssl 인증서를 활용하여 ssl offload 지원 [[참고]](https://aws-hyoh.tistory.com/entry/L4-%EC%8A%A4%EC%9C%84%EC%B9%98-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-7)
* 교차 영역의 로드밸런싱 지원

**Network Load Balancer** [[참고]](https://aws-hyoh.tistory.com/135)
* L4 로드밸런서, TCP, UDP를 기반으로 요청을 전달하는 로드밸런서
* IP, Port 기반 트래픽 분산
* Elastic IP 할당을 통해 고정 IP 사용 가능
* 보안 그룹 적용 불가능
* ssl 인증서를 활용하여 ssl offload 지원
* 교차 영역 로드밸런싱 지원

## ELB 기능
**Sticky Session**
* 세션 고정 기능
* 사용자의 세션이 존재하는 특정 EC2 인스턴스로 요청 전달을 고정하는 기능
* 하나의 요청이 특정 EC2 인스턴스로 들어와 세션이 생성되었는데, 그 다음 요청은 로드밸런싱에 의해 다른 인스턴스로 전달되는 경우 세션이 없어 세션을 재생성해야 하기 때문

**교차영역 로드밸런싱**
* 가용영역이 아닌 EC2 개수를 기반으로 트래픽 분산하는 기능
* 기본적으로 ELB는 트래픽을 가용영역별로 비율을 나눠 전달한다.
  * 가용영역 2개에 연결되어 있으면 각 가용영역에 50%씩 전달
  * 가용영역 2a에는 인스턴스 1개, 2b에는 인스턴스 5개가 있는 경우 2a 가용영역에 있는 인스턴스의 부하가 심해지는 문제 발생
* 이를 방지하기 위해, 가용영역이 아닌 EC2 개수를 기반으로 트래픽 분산

**SSL offload** [[참고]](https://aws-hyoh.tistory.com/entry/L4-%EC%8A%A4%EC%9C%84%EC%B9%98-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-7)
* HTTPS는 SSL과 TLS 프로토콜을 사용하여 데이터를 암호화해 통신한다.
* HTTPS 통신을 하기 위해선 SSL 인증서를 검증하고, 데이터를 암호화하는 과정 (SSL Handshake)를 거치므로, 서버에 부담이 된다.
* 위 작업을 L4 스위치가 대신함으로써, SSL 인증서를 통해 사용자와는 암호화 통신을 하고, 서버와는 평문 통신을 하는 ssl offload를 지원한다.


> https://aws-hyoh.tistory.com/19  
> https://assu10.github.io/dev/2022/11/13/load-balancing-1/
