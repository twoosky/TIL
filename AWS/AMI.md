# AMI
* EC2 인스턴스를 실행하기 위해 필요한 정보를 담은 이미지이다.
  * OS, 저장 용량, 연결된 EBS 볼륨(가상 하드디스크) 등
* AMI를 통해 동일한 세팅의 인스턴스를 생성할 수 있다.
* AMI를 사용하여 EC2를 복제하거나, 다른 리전으로 전달할 수 있다.
* Snapshot을 기반으로 AMI가 구성된다.

## Hands-On
nginx 컨테이너 환경의 인스턴스를 AMI로 만든 뒤, 해당 AMI로 인스턴스를 생성해보자.
## 1. EC2에 Docker 설치
* 우분투 AMI를 사용하는 기본 설정의 EC2를 생성한 뒤 EC2에 연결해 Dcoker 설치
```bash
$ sudo apt-get update -y
```

```bash
$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg
```

```bash
$ sudo mkdir -m 0755 -p /etc/apt/keyrings
```

```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

```bash
$ echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash
$ sudo apt-get update -y
```

```bash
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

**권한 추가**
* sudo 를 사용하지 않고 Docker 명령을 실행할 수 있도록 권한 추가 후 ec2 재접속
```bash
$ sudo chmod 666 /var/run/docker.sock
```
```bash
$ sudo usermod -aG docker ubuntu
```

**실행 테스트**
```bash
$ docker run hello-world

# 실행된 도커 컨테이너 확인
$ docker ps -al

# 이미지 확인
$ docker images
```

## 2. NGINX dockerfile 작성

1. index.html 작성
```bash
$ vi index.html
```
```bash
hello, world!  
```

2. nginx dockerfile 작성

```bash
$ vi Dockerfile
```
```dockerfile
FROM nginx

COPY . /usr/share/nginx/html
```

3. Docker 이미지 생성 및 컨테이너 실행

```bash
$ docker build . -t nginx
```
```bash
$ docker run -d -p 80:80 nginx
```

4. 8080 포트로 접속

```bash
{인스턴스 IP}:8080
```
<img src="https://github.com/twoosky/TIL/assets/50009240/1305a4aa-a9f8-4a5e-82ad-639f9643b08a" width="500" height="150">

## 3. AMI 생성
AMI를 생성할 인스턴스 오른쪽 마우스 클릭 -> 이미지 및 템플릿 -> 이미지 생성 클릭

![Group 110](https://github.com/twoosky/TIL/assets/50009240/499b1f65-8631-41d3-b720-a2a770d508ea)

재부팅 안함 활성화 선택, 스토리지 유형은 해당 인스턴스의 EBS 볼륨 유형을 따라간다. 크기만 조절 가능

![Group 111](https://github.com/twoosky/TIL/assets/50009240/44db0cca-e702-4fea-83f8-c94558cec9e2)

## 4. AMI로 인스턴스 생성
위에서 생성한 AMI 선택

![Group 108](https://github.com/twoosky/TIL/assets/50009240/fc565fdb-1e4a-4735-96cc-bbae6fa0907b)

보안 그룹은 ssh(22), http(80)에 대한 모든 트래픽 허용하도록 생성

![Group 107](https://github.com/twoosky/TIL/assets/50009240/e7c24c47-0220-42a9-87cc-b83e4174219a)

EBS는 기본 설정되어 있는 AMI의 EBS 사용, 인스턴스 생성 클릭

![Group 109](https://github.com/twoosky/TIL/assets/50009240/e54f7c98-e2bf-4d95-b741-5765c133c0b5)


## 5. nginx docker 이미지 확인
* EC2에 연결해 nginx 도커이미지 확인, AMI를 통해 docker가 설치된 환경, nginx 이미지가 존재하는 환경의 인스턴스를 생성했다.

![Screenshot from 2023-07-10 02-59-35](https://github.com/twoosky/TIL/assets/50009240/7d07490d-88d6-434e-9590-1a3d33c833c0)

* nginx 컨테이너 실행 후 80포트 접속해 hello, world! 확인
* AMI로 생성한 인스턴스에서는 docker 설치, nginx dockerfile 빌드와 같은 과정을 거치지 않아도 nginx 컨테이너를 실행시킬 수 있다. (성공!)
```bash
$ docker run -d -p 80:80 nginx
```
![Screenshot from 2023-07-10 02-46-31](https://github.com/twoosky/TIL/assets/50009240/51456136-5bbb-48a5-9ee9-32260bc6b84d)
