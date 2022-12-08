# Stream
* Java 8부터 추가된 기술이다.
* stream은 컬렉션(배열 포함)의 요소를 하나씩 참조해서 람다식으로 처리할 수 있는 반복자이다.
* 람다식으로 요소 처리 코드를 제공한다.
  * 스트림이 제공하는 대부분의 요소 처리 메소드는 함수적 인터페이스 매개타입을 갖는다.
  * 매개값으로 람다식 또는 메소드 참조를 대입할 수 있다. 
* 내부 반복자를 사용하므로 병렬 처리가 쉽다.
  * 내부 반복자(internal iterator): 컬렉션 내부에서 요소들을 반복시키고, 개발자는 요소당 처리해야할 코드만 제공하는 코드 패턴
  * 내부 반복자 이점
    * 개발자는 요소 처리 코드에만 집중
    * 멀티코어 CPU를 최대한 활용하기 위해 요소들을 분배시켜 병렬 처리 작업을 할 수 있다.
* 병렬(parallel) 처리
  * 한가지 작업을 서브 작업으로 나누고, 서브 작업들을 분리된 스레드에서 병렬적으로 처리한 후, 서브 작업들의 결과를 최종 결합하는 방법
  * 자바는 `ForkJoinPool` 프레임워크를 이용해서 병렬 처리를 한다.  

## Stream API 특징
### 1. 원본의 데이터를 변경하지 않는다. 
* Stream API는 원본 데이터를 조회하여 원본의 데이터가 아닌 별도의 요소들로 Stream을 생성한다.  
* 정렬, 필터링 등의 작업은 별도의 Stream 요소들에서 처리가 된다.   
```java
public class stream {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(3);
        list.add(2);

        List<Integer> sortedList = list.stream().sorted().collect(Collectors.toList());

        System.out.println(list); // [1,3,2]
        System.out.println(sortedList); // [1,2,3]
    }    
}
```

### 2. Stream은 일회용이다.
* Stream API는 일회용이므로 한번 사용이 끝나면 재사용이 불가능하다. Stream이 또 필요한 경우엔 다시 생성해주어야 한다.
* 만약 닫힌 Stream을 다시 사용한다면 IllegalStateException이 발생한다.
```java
// 홀수만 추출하는 stream 생성
IntStream sumStream = list.stream().filter(number -> number % 2 == 1).mapToInt(number -> number);

int sum1 = sumStream.sum();
int sum1 = sumStream.sum();  // 스트림이 이미 사용되어 닫혔으므로 에러 발생

// IllegalStateException 발생
java.lang.IllegalStateException: stream has already been operated upon or closed
    at java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:229)
    at java.util.stream.ReferencePipeline.noneMatch(ReferencePipeline.java:459)
```

### 3. 후처리(Lazy Evaluation)
* Stream은 기본적으로 3가지 과정을 거친다.
```
1. 생성: 스트림 인스턴스 생성
2. 가공: 원하는 결과로 조작하는 중간 연산
3. 결과: 결과를 만드는 최종 연산
```
* 결과를 도출하는 함수를 호출하기 전까진 Stream이 정의가된 상태일 뿐 아무런 연산도 수행하지 않는다.
```java
// 정의만 된 상태
IntStream sumStream = list.stream().filter(number -> number % 2 == 1).mapToInt(number -> number);

// 이때 실제 정의된 작업이 수행된다.
int sum1 = sumStream.sum();
```
