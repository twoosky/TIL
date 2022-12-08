# Stream API

## 1. Stream 생성하기
* 컬렉션
```java
public class StreamTest {
    public static void main(String[] args) {
        List<String> list = List.of("a", "b", "C");
        Stream<String> listStream = list.stream();
    }
}
```
* 배열  
```java
public class StreamTest {
    public static void main(String[] args) {
        Integer[] arr = {1, 2, 3, 4, 5};
        Stream<Integer> arrStream1 = Arrays.stream(arr);
        Stream<Integer> stream = Arrays.stream(arr, 0, 3);
        Stream<String> arrstream2 = Stream.of("a", "b", "c");
    }
}

// 배열로 Stream을 생성하기 위해서는 Stream의 of 메소드 또는 Arrays의 stream 메소드를 사용하면 된다.
```
* 원시
```java
public class StreamTest {
    public static void main(String[] args) {
        IntStream stream = IntStream.range(0, 10);
    }
}
```
* 빈 스트림
```java
public class StreamTest {
    public static void main(String[] args) {
        Stream<Integer> emptyStream1 = Stream.empty();
        Stream<String> emptyStream2 = Stream.empty();
    }
}
```

## 2. Stream 가공하기 : 중간 연산
1. filter: 데이터 정제
```java
// 함수 인자
Stream<T> filter(Predicate<? super T> predicate);

// 사용
public class StreamTest {
    public static void main(String[] args) {
        List<Integer> data = List.of(1, 2, 3, 4, 5);
        Stream<Integer> stream = data.stream().filter(it -> it % 2 == 0);
        List<Integer> list = stream.collect(Collectors.toList());  
        // [2, 4]
    }
}
```
2. map: 데이터 변환
```java
// 함수 인자
<R> Stream<R> map(Function<? super T, ? extends R> mapper);

// 사용
public class StreamTest {
    public static void main(String[] args) {
        List<Integer> data = List.of(1, 2, 3, 4, 5);
        Stream<Integer> stream = data.stream().map(it -> it + 1);
        List<Integer> list = stream.collect(Collectors.toList());
        // [2, 3, 4, 5, 6]
    }
}
```
* 위의 map 함수의 람다식은 메소드 참조를 이용해 변경이 가능하다.
* `Object명::메소드명` 으로 static 메소드 참조 가능
```java
public class StreamTest {
    public static void main(String[] args) {
        List<Integer> data = List.of(1, 2, 3, 4, 5);
        Stream<Integer> stream = data.stream().map(StreamTest::plusOne);
        List<Integer> list = stream.collect(Collectors.toList());
        // [2, 3, 4, 5, 6]
        System.out.println(list);
    }
    
    public static int plusOne(int number) {
        return number + 1;
    }
}
```
3. sorted: 정렬
```java


