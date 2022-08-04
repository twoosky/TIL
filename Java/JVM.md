# JVM
* 자바 가상 머신 JVM은 자바 프로그램 실행환경을 만들어 주는 소프트웨어이다.  
* JVM은 운영체제 위에서 동작하는 프로세스로 자바 코드를 컴파일해서 얻은 바이트 코드를 해당 운영체제가 이해할 수 있는   
기계어로 바꿔 실행시켜주는 역할을 한다. 
> `.class 파일`은 `바이트 코드`라고 하는데 사람이 작성한 자바 코드를 가상머신(JVM)이 이해할 수 있는 중간 코드로 컴파일한 것을 말한다.  
> 가상머신(JVM)은 이 바이트코드를 각각의 하드웨어 아키텍처에 맞는 기계어로 다시 컴파일한다.  

# JVM의 동작 방식
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcQRqku%2Fbtru0vJ6Ixx%2F9qCTW7ChXc80fGfQUrT4B0%2Fimg.png" width="500" height="400">

1. 자바로 개발된 프로그램을 실행하면 OS로부터 메모리를 할당한다.
2. 자바 컴파일러(javac)가 자바 소스코드(.java)를 자바 바이트코드(.class)로 컴파일한다.
3. Class Loader를 통해 JVM Runtime Data Area로 로딩한다.
4. Runtime Data Area에 로딩 된 .class(바이트 코드)들은 Execution Engine을 통해 해석한다.
5. 해석된 바이트 코드는 Runtime Data Area의 각 영역에 배치되어 수행하며 이 과정에서 Execution Engine에 의해  
   GC의 작동과 스레드 동기화가 이루어진다.

# JVM 구조
* `Class Loader: 클래스 로더`
  * JVM 내 Runtime Data Area에 클래스파일들 적재한다.
  > 자바에서 소스를 작성하면 .java파일이 생성된다.   
  > .java 소스를 자바 컴파일러가 컴파일하면 .class파일(바이트코드)이 생성된다.   
  > Class Loader는 이렇게 생성된 클래스파일들을 묶어서 JVM이 운영체제로부터 할당받은 메모리 영역인 `Runtime Data Area`로 적재한다.
* `Execution Engine: 실행 엔진`
  *  Class Loader에 의해 메모리에 적재된 클래스(바이트 코드)들을 기계어로 변경해 명령어 단위로 실행한다.
* `Garbage Collector`
  * Heap 메모리 영역에 생성(적재)된 객체들 중에 참조되지 않는 객체들을 탐색 후 제거한다.
  * GC가 수행되는 동안 GC를 수행하는 스레드 외 모든 스레드는 일시정지된다.
* `Runtime Data Area`
  * JVM의 메모리 영역으로 자바 애플리케이션을 실행할 때 사용되는 데이터들을 적재하는 영역이다.
  * 자바 프로그램을 실행할 때, OS로부터 할당받는다.
  * 이 영역은 크게 Method Area, Heap Area, Stack Area, PC Register, Native Method Stack로 나눌 수 있다.

# JVM 메모리 구조
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbZR97z%2FbtrvtdBl1Md%2FLbUk2NVlgDmsKMcBiQ9f4K%2Fimg.png" width="550" height="120">

* 모든 스레드가 공유해서 사용 (GC의 대상)
  * 힙 영역(Heap Area)
  * 메서드 영역(Method Area)
* 스레드마다 하나씩 생성
  * 스택 영역(Stack Area)
  * PC 레지스터(PC Register)
  * 네이티브 메서드 스택(Native Method Stack)
### Method Area
* JVM이 읽어들인 클래스와 인터페이스에 대한 Runtime Constant Pool, 멤버 변수(필드), 클래스 변수(Static 변수), 생성자와 메소드를 저장하는 공간이다. 
* Runtime Constant Pool이란?
  * 클래스와 인터페이스 상수, 메소드와 필드에 대한 모든 레퍼런스를 저장한다.
  * JVM은 Runtime Constant Pool을 통해 해당 메소드나 필드의 실제 메모리 상 주소를 찾아 참조한다.
* 모든 스레드에서 공유한다.
* JVM이 실행되면서 생성되는 영역이고, 프로그램 종료 시 소멸된다.
* `자바 바이트코드`와 `static`으로 선언된 클래스 변수는 메서드 영역에 저장된다.
### Heap Area
* JVM이 관리하는 프로그램 상에서 데이터를 저장하기 위해 런타임 시 동적으로 할당하여 사용하는 영역이다.
* New 연산자로 생성된 객체와 배열을 저장한다.
* 힙 영역에서 생성된 객체와 배열은 스택 영역의 변수나 다른 객체의 필드에서 참조한다.
* 참조하는 변수나 필드가 없다면 의미 없는 객체가 되어 GC의 대상이 된다.
  > <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcmeYY6%2FbtrveK8fwH2%2FNwdYae5gEQHih3lyV83as0%2Fimg.png" width="530" height="150">
  > 
  > Heap Area는 효율적인 GC를 위해 Young Generation, Old Generation 으로 나뉜다.   
  > Young Generation은 자바 객체가 생성되자마자 저장되고, 생긴지 얼마 안되는 객체가 저장되는 공간이다.  
  > Young Generation(Eden + Servivor)영역이 차게 되면 참조정도에 따라 Old영역으로 이동하거나 회수된다.  
  > 이렇게 Young Generation 영역에서 실행하는 GC를 `Minor GC`라고 한다.    
  > 추후 Old영역이 차게 되면 Old 영역에 있는 모든 객체들을 검사하여 참조되지 않는 객체들을 한꺼번에 삭제하는 `Major GC`가 실행된다.  
* 모든 스레드에서 공유한다.
### Stack Area
* 각 스레드마다 하나씩 존재하며, 스레드가 시작될 때 할당된다.
* 매개변수, 지역변수가 할당되는 메모리 공간이다.
* stack에 저장된 변수는 해당 변수가 선언된 메서드 종료 시 소멸된다.
* LIFO 구조로 나중에 들어온 데이터가 먼저 나간다.
* `기본 타입` 변수: 스택에 `값` 저장
* `참조 타입` 변수: 힙이나 메소드 영역 객체의 `주소` 저장
### PC Register
* 각 스레드마다 존재하며, 현재 스레드가 실행되는 부분의 주소와 명령을 저장하고 있는 영역이다.
### Native Method Stack Area
* 자바 외 언어로 작성된 네이티브 코드를 위한 스택이다.

# 변수의 종류와 메모리 구조
* 인스턴스 변수: 객체에서 사용되는 변수 - Heap 영역
* 클래스 변수: static 변수 - 메소드 영역
* 지역 변수: 메소드 내에서 선언되어 메소드 안에서만 사용할 수 있는 변수 - 스택 영역
```java
public class Test{
  private int iv; // 인스턴스 변수
  public static int cv; // 클래스 변수
  public void print(){
  	int lv; // 지역 변수
  }
}
```






