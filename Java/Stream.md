# Stream
* Java 8부터 추가된 기술이다.
* Stream API는 데이터를 추상화하고, 처리하는데 자주 사용되는 함수들을 정의해둔 것이다.
* 데이터를 추상화하였다는 것은 데이터의 종류에 상관 없이 같은 방식으로 데이터를 처리할 수 있다는 것을 의미한다.
```java
// 컬렉션에서 스트림 생성
Stream<Integer> stream = list.stream();
```

## Stream API 특징
1. 원본의 데이터를 변경하지 않는다. 
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

2. Stream은 일회용이다.
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

3. 후처리(Lazy Evaluation)
* Stream은 기본적으로 3가지 과정을 거친다.
```
1. 생성
2. 가공
3. 결과
```
* 결과를 도출하는 함수를 호출하기 전까진 Stream이 정의가된 상태일 뿐 아무런 연산도 수행하지 않는다.
```java
// 정의만 된 상태
IntStream sumStream = list.stream().filter(number -> number % 2 == 1).mapToInt(number -> number);

// 이때 실제 정의된 작업이 수행된다.
int sum1 = sumStream.sum();
```

4. 병렬 처리
