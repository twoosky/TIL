# 코틀린에서 null을 다루는 방법
## 1. Kotlin에서의 null 체크
1. Java와 Kotlin에서의 null 처리 비교
* Java 코드에선 인자 타입과 반환타입이 reference type이므로 null을 허용해준다.
* Kotlin은 *타입 뒤에 ?* 를 붙여 null 허용을 직접 명시해줘야 한다.
```kotlin
// Java
public Boolean startsWithA1(String str) {
    if (str == null) {
        return null;
    }
    return str.startsWith("A");
}
```
```kotlin
// Kotlin
fun startsWithA1(str: String?) : Boolean? {
    if (str == null) {
        return null
    }
    return str.startsWith("A")
}
```

2. null을 허용하는 타입에서 함수 호출
* 아래와 같이 null을 허용하는 타입에서 바로 함수를 호출하는 경우 에러가 발생한다.
```kotlin
fun startWithA(str: String?): Boolean {
    return str.startsWith("A")
}
```
* 방법 1)  null을 허용하지 않는 타입으로 수정 후 함수를 호출한다.
* 방법 2)  null 체크를 한 뒤 함수를 호출한다.
> Kotlin에서는 null이 가능한 타입을 완전히 다르게 취급한다.

## 2. Safe Call과 Elvis 연산자
1. Safe Call
* null이 아니면 실행하고, null이면 실행하지 않는다. (그대로 null)
* `?.`으로 함수를 호출한다.
```kotlin
val str: String? = "ABC"
str.length    // 불가능
str?.length   // 가능
```
2. Elvis 연산자
* 앞의 연산 결과가 null이면 뒤의 값을 사용한다.
* 함수 호출 뒤에 `?: 사용할 값` 을 붙인다.
```kotlin
val str: String? = "ABC"
str?.length ?: 0
```
예시 1)
```kotlin
// Before
fun startsWithA1(str: String?) : Boolean {
    if (str == null) {
        throw IllegalArgumentException("null 입니다.")
    }
    return str.startsWith("A")
}
```
```kotlin
// After
fun startsWithA1(str: String?) : Boolean {
    return str?.startsWith("A")
        ?: throw IllegalArgumentException("null 입니다.")
}
```

예시 2)
```java
// Java
public long calculate(Long number) {
    if (number == null) {
        return 0;
    }
    ...
}
```
```kotlin
// Kotlin
fun calculate(number: Long?): Long {
    number ?: return 0
    ...
}
```

## 3. null 아님 단언
* nullable 타입이지만 어떤 경우에도 null이 될 수 없는 경우 null이 아님을 명시할 수 있다.
* `!!` 을 통해 null이 아님을 명시한다.
```kotlin
fun startsWithA(str: String?) Boolean {
    return str!!.startsWith("A")
}
```
* `!!`을 명시했는데 null값이 들어온 경우 컴파일 타임에는 에러가 발생하지 않지만, 런타임에 NullPointException이 발생한다.

## 플랫폼 타입
Kotlin에서 Java 코드를 가져다 사용할 때 어떻게 처리될까?
1. getName()에 @Nullable 어노테이션을 붙인 경우
```Java
import org.jetbrains.annotations.Nullable;

public class Person {
    private final String name;

    public Person(String name) {
        this.name = name;
    }

    @Nullable
    public String getName() {
        return name;
    }
}
```

getName() 메서드가 nullable이기 때문에 person.name 부분에서 에러가 발생한다.  
<img src="https://user-images.githubusercontent.com/50009240/210384996-8775ebaa-c4b1-42f7-874f-bce393c7c12d.png" width="450" height="180">


2. getName()에 @NotNull 어노테이션을 붙인 경우
```Java
// 위 코드는 동일
@NotNull
public String getName() {
    return name;
}
```
person.name에 에러가 발생하지 않는다.  
<img src="https://user-images.githubusercontent.com/50009240/210386217-3d47e765-b8b5-426c-92f8-83ae70f0321c.png" width="450" height="180">

* null에 관련된 Annotation을 활용하면 Kotlin에서 null에 대한 타입을 인식해 활용할 수 있다.
* `플랫폼 타입`이란 코틀린이 null 관련 정보를 알 수 없는 타입이다. (null 관련 Annotation이 없는 경우)
* Kotlin에서 Java 코드를 사용할 때 플랫폼 타입을 사용하는 경우 Runtime 시 Exception이 발생할 수 있으므로 유의해야한다.
