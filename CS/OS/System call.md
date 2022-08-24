# System call
* `System call`은 사용자나 응용 프로그램이 커널에서 제공하는 기능을 사용하기 위한 인터페이스이다.
* 운영체제는 커널이 제공하는 서비스를 System call을 통해 제한함으로써 컴퓨터 자원을 보호한다.
* `kernel mode`: System call을 수행하는 모드, I/O 장치를 포함해 모든 주소 영역에 접근이 가능하다.
* `user mode`: 응용프로그램이 수행되는 모드, 컴퓨터 자원에 접근할 수 없다.

### System call 수행 과정
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FALlGh%2FbtquL0XgOow%2FqaDksq1AsUKrpZKEzSu0DK%2Fimg.png" width="220" height="250">

1. 프로세스가 System call 호출
2. trap이 발생하여 Kernel Mode 진입
3. 요청 받은 System call 수행
4. return-from-trap을 발생시켜 user mode로 복귀

### System call 종류
`프로세스 제어`: Process Control
* fork(): 프로세스 생성한다.
* wait(): 자식 프로세스가 끝날 때까지 대기한다.
* exec(): 다른 프로그램을 실행한다.

`파일 조작`: File Manipulation  
* open()
* read()
* write()
* close()

`장치 조작`: Device Manipulation  
* ioctl()
* read()
* write()

`정보 유지`: Information Maintenance  
* getpid()
* alarm()
* sleep()

`통신`: Communication  
* pipe()
* shm_open()
* mmap()

`보호`: Protection  
* chmod()
* umask()
* chown()  
