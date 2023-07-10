# Auto Scaling
AWS Auto Scaling은 CPU, 메모리, 디스크, 네트워크 트래픽과 같은 시스템 자원들의 메트릭(Metric)값을 모니터링하여 서버 규모를 자동으로 조절하는 서비스이다. (Scale Out)

## Scaling
* 인스턴스 혹은 컴퓨팅 파워를 늘리는 것
* **Scale Up**: 기존의 서버를 보다 높은 사양으로 업그레이드하는 것
  * 성능 상승과 가격 상승이 비례하지 않는다. 가격이 훨씬 비싸진다.
  * 스케일업은 물리적으로 한계가 존재한다. 
* **Scale Out**: 서버의 개수를 늘리는 것
  * 서버의 개수를 늘리는 방식이므로, 성능과 가격이 비례
  * 필요한만큼 확장 가능 (하드웨어 제약 X)

## Auto Scaling
**Auto Scaling 기능**
* EC2 인스턴스의 개수 보장, 그룹의 최소 및 최대 인스턴스 개수 관리
* 다양한 스케일링 정책 적용 가능 (CPU 부하에 따라 인스턴스 개수 증가 등)
* 가용 영역에 인스턴스가 골고루 분산되도록 인스턴스 분배

**Auto Scaling 구성**
* Auto Scaling Group
  * EC2 인스턴스를 조정 및 관리 목적의 논리 단위로 취급될 수 있도록 그룹으로 구성
  * 그룹을 생성할 때 EC2 인스턴스의 최소, 최대, 원하는 개수 지정, 이 범위 안에서 Scale in/out 발생
  * 인스턴스의 증감은 Auto Scaling Group 안에서 이루어진다.
  * 로드밸런서의 대상 그룹으로 Auto Scaling Group 지정 가능
* 시작 템플릿
  * Auto Scaling Group에서 인스턴스를 생성하는데 사용하는 템플릿, AMI와 비슷한 개념
  * 인스턴스 유형, 키 페어, 보안 그룹, EBS 볼륨(블록 디바이스), AMI 등 구성
  * 시작 템플릿은 수정 불가능, 버전을 올려 새로 생성해야 한다.
* Auto Scaling 조정 옵션
  * 다양한 스케일링 정책을 적용 가능
  * CloudWatch나, ELB와 연계 가능
 
## Hands-On
시작 템플릿 구성 및 Auto Scaling Group을 생성해 인스턴스 개수를 조절해보자

## 1. 시작 템플릿 생성
![Screenshot from 2023-07-02 22-09-07](https://github.com/twoosky/TIL/assets/50009240/d5a70174-d0cf-446a-8c69-9005ed3601bd)
* AMI는 기존 AMI Ubuntu 선택

![Screenshot from 2023-07-02 22-10-11](https://github.com/twoosky/TIL/assets/50009240/a318a1dd-1782-484b-9e45-7520254f746b)
![Screenshot from 2023-07-02 22-10-53](https://github.com/twoosky/TIL/assets/50009240/2189cd8e-db21-480f-93f6-13f7ae2169c9)
![Screenshot from 2023-07-02 22-11-02](https://github.com/twoosky/TIL/assets/50009240/cb6af962-1cb4-4a4a-a455-484ca0f67c51)
* 퍼블릭 IP 자동 할당 활성화, 종료 시 삭제 옵션으로 예 선택

![Screenshot from 2023-07-02 22-11-50](https://github.com/twoosky/TIL/assets/50009240/5902fe07-076f-411b-8d26-543c8920ed04)

## 2. Auto Scaling Group 생성
![Screenshot from 2023-07-02 22-13-52](https://github.com/twoosky/TIL/assets/50009240/d646153a-6324-4389-8ccf-75d202a47e36)
* Auto Scaling한 인스턴스를 배치할 가용 영역 선택

![Screenshot from 2023-07-02 22-14-12](https://github.com/twoosky/TIL/assets/50009240/a9466da1-4c76-4077-9601-f103f9abd5d1)
* 로드밸런서는 추후 로드밸런서 정리할 때 연결할 것임, 일단 로드밸런서 없음 선택

![Screenshot from 2023-07-02 22-15-04](https://github.com/twoosky/TIL/assets/50009240/5f617344-d659-4250-8e74-cd08d99be299)
* Scale in/out할 최소, 최대, 원하는 인스턴스 개수 설정

![Screenshot from 2023-07-02 22-15-19](https://github.com/twoosky/TIL/assets/50009240/1ad84118-7448-44b2-a95e-f7b7316ca0a9)

## 3. Auto Scaling 확인
* Auto Scaling 그룹 - 인스턴스 관리 확인
* 원하는 요량을 2로 선택했으므로, Auto Scaling에 의해 2개의 인스턴스가 자동 생성된 것을 확인할 수 있다.
<img width="1116" alt="Untitled (3)" src="https://github.com/twoosky/TIL/assets/50009240/e5be0dfd-bc13-48ea-b363-6b6c3ad6d9dd">
