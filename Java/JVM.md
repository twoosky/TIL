# JVM
* JVM은 Java Virtual Machine의 약자로, 자바 가상 머신이다.
* 자바 코드를 컴파일해서 얻은 바이트 코드를 JVM이 운영체제에 맞는 실행 파일로 바꿔준다.

## JVM의 동작 방식
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcQRqku%2Fbtru0vJ6Ixx%2F9qCTW7ChXc80fGfQUrT4B0%2Fimg.png" width="450" height="350">

1. 자바로 개발된 프로그램을 실행하면 OS로부터 메모리를 할당한다.
2. 자바 컴파일러(javac)는 자바 소스코드(.java)를 자바 바이트코드(.class)로 컴파일한다.
3. Class Loader를 통해 JVM Runtime Data Area로 로딩한다.
4. Runtime Data Area에 로딩 된 .class(바이트 코드)들은 Execution Engine을 통해 해석한다.
5. 해석된 바이트 코드는 Runtime Data Area의 각 영역에 배치되어 수행하며 이 과정에서 Execution Engine에 의해  
   GC의 작동과 스레드 동기화가 이루어진다.

## JVM 구조
* Class Loader(클래스 로더)
  * JVM 내로 클래스 파일(* .class)을 로드하고, 링크를 통해 배치하는 작업을 수행하는 모듈
* Execution Engine(실행 엔진)
  * 클래스 로더를 통해 JVM 내의 Runtime Data Area에 배치된 바이트 코드들을 명렁어 단위로 읽어서 실행
* Garbage Collector
  * Heap 메모리 영역에 생성된 객체들 중에 참조되지 않는 객체들을 탐색 후 제거
* Runtime Data Area
  * JVM의 메모리 영역으로 자바 애플리케이션을 실행할 때 사용되는 데이터들을 적재하는 영역
  * 자바 프로그램을 실행할 때, OS로부터 할당받는다.

## Runtime Data Area
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbZR97z%2FbtrvtdBl1Md%2FLbUk2NVlgDmsKMcBiQ9f4K%2Fimg.png" width="550" height="120">

* 모든 스레드가 공유: Method Area, Heap Area
* 스레드 별로 생성: Stack Area, PC Resisters, Native Method Stack

**1. Method Area**   
* 클래스, 인터페이스, 메소드, Static 변수 등의 바이트 코드를 보관하는 영역.
* 모든 스레드가 공유하는 메모리 영역
* 프로그램 시작 전에 로드되고 프로그램 종료 시 소멸
* Runtime Constant Pool은 클래스와 인터페이스 상수, 메서드와 필드에 대한 모든 레퍼런스를 저장한다.
* JVM은 Runtime Constant Pool을 통해 해당 메소드나 필드의 실제 메모리 상 주소를 찾아 참조한다.

**2. Heap Area**
* new 키워드로 생성된 객체와 배열을 저장하는 영역 (메소드 영역에 로드된 클래스만 생성 가능)
* Garbage Collector가 참조되지 않는 메모리를 확인하고 제거하는 영역
* 모든 스레드에서 공유하는 영역

**3. Stack Area**
* 지역변수, 파라미터, 리턴 값 등 임시로 할당되었다가, 메소드 수행이 끝나면 소멸되는 데이터를 저장하는 영역
* 각 스레드마다 하나씩 존재하며, 스레드가 시작될 때 할당된다.
* 메소드 호출 시마다 각각의 스택 프레임(그 메서드만을 위한 공간)이 생성된다. 메서드 수행이 끝나면 해당 프레임을 삭제한다.

**4. PC Register**
* 현재 스레드가 실행 중인 JVM 명령의 주소(Program Counter)를 갖고 있는 영역
* 각 스레드마다 하나씩 존재하며, 스레드가 시작될 때 생성된다.

**5. Native Method Stack Area**
* 자바 외 언어로 작성된 코드를 위한 스택 영역

## 변수의 종류와 메모리 구조
* `인스턴스 변수(Heap 영역)`: 객체에서 사용되는 static이 아닌 변수
* `클래스 변수(메소드 영역)`: static 변수
* `지역 변수(스택 영역)`: 메소드 내에서 선언되어 메소드 안에서만 사용할 수 있는 변수
```java
public class Test{
  private int iv; // 인스턴스 변수
  public static int cv; // 클래스 변수
  
  public void print(){
  	int lv; // 지역 변수
  }
}
```
