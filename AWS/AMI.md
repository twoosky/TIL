# AMI

# AMI 생성해보기
nginx 컨테이너 환경의 인스턴스를 AMI로 만든 뒤, 해당 AMI로 인스턴스를 생성해보자.
## 1. EC2에 Docker 설치

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

```bash
$ :wq
```
2. nginx dockerfile 작성

```bash
$ vi Dockerfile
```
```docker
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

3.36.53.243:8080
```
<img src="https://github.com/twoosky/TIL/assets/50009240/1305a4aa-a9f8-4a5e-82ad-639f9643b08a" width="500" height="150">

## 3. AMI 생성

## 4. AMI로 인스턴스 생성

## 5. nginx docker 이미지 확인
