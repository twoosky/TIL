# 코틀린에서 Type 다루는 방법
## 1. 기본 타입
1. 코틀린에서는 선언된 기본값을 보고 타입을 추론한다.
```kotlin
val number1 = 3   // Int
val number2 = 3L  // Long
val number3 = 3f  // Float
val number4 = 3   // Double
```
2. 코틀린에서 기본 타입간의 변환은 명시적으로 이루어진다.  
* Java의 경우 타입간 변환이 내부적으로 이루어진다.
* Java에서는 더 큰 타입으로 자동 변환이 된다.
```java
int number1 = 4;
long number2 = number1;

// int 타입의 값이 long 타입으로 내부적으로 변환되어 연산이 처리된다.
System.out.println(number1 + number2);
```

* Kotlin에서는 내부적으로 타입을 변환해주지 않는다.
* 위와 같은 경우 Type mismatch 에러가 발생한다.
* Kotlin에서는 `to변환타입()`을 사용해 명시적으로 타입을 변환해줘야 한다.
```kotlin
val number1: Int = 4
val number2: Long = number1.toLong()

println(number1 + number2)
```
> 변수가 nullable이라면 바로 함수 호출이 불가능하므로, 타입 변환은 위해 적절한 처리가 필요하다.
> ```kotlin
> val number1: Int? = 3
> val number2: Long = number1?.toLong() ?: 0L
> ```

## 2. 타입 캐스팅
자바 코드를 코틀린 코드로 변환하며 이해해보자 


1. is, as
* `is`: 객체 타입 비교 연산자 ( = instanceof ), obj와 Person 타입이 같으면 true 아니면 false
* `as`: obj 변수를 Person 타입으로 캐스팅
```java
// Java
public static void printAgeIfPerson(Object obj) {
    if (obj instanceof Person) {
        Person person = (Person) obj;
        System.out.println(person.getAge());
    }
}
```
```kotlin
// Kotlin
fun printAgeIfPerson(obj: Any) {
    if (obj is Person) {
        val person = obj as Person
        println(person.age)
    }
}
```
* 코틀린에선 스마트 캐스트가 가능하다. 조건문에서 타입을 체크해준 경우 타입 캐스팅을 자동으로 해준다.
* if 문에서 타입 체크를 해줬으므로, as Person을 생략할 수 있다.

2. !is
* instanceof에 반대되는 연산자이다.
* 두 비교 객체 타입이 같은 경우 false, 다른 경우 true
* 자바에서는 instanceof 전체를 묶어 부정 처리(!)해야 하지만,코틀린에서는 간단히 `!is`로 구현가능하다.
```java
if (!(obj instanceof Person)) {
    Person person = (Person) obj;
    System.out.println(person.getAge());
}
```
```kotlin
if (obj !is Person) {
    val person = obj as Person
    println(person.age)
}
```

3. as?
* 변환할 객체가 null이 아니라면 타입을 변환하고, null인 경우 null을 반환한다.
```kotlin
fun main() {
    val person = Person("이하늘", 24)
    printAgeIfPerson(person)
    printAgeIfPerson(null)
}

fun printAgeIfPersonobj: Any?) {
    val person = obj as? Person
    println(person?.age)
}

// output
// 24
// null
```
* `as`를 사용해 null인 객체를 타입 변환하려는 경우 nullPointException이 발생한다.
```kotlin
fun main() {
    val person = Person("이하늘", 24)
    printAgeIfPerson(null)
}

fun printAgeIfPerson(obj: Any?) {
    val person = obj as Person
    println(person.age)
}
```
<img src="https://user-images.githubusercontent.com/50009240/210415000-a91e9108-7cf7-4ac4-88df-9afbcb55414c.png" width="780" height="100">

> 정리
> * **value `is` Type** : value가 Type이면 true, 아니면 false
> * **value `!is` Type** : value가 Type이면 false, 아니면 true
> * **value `as` Type** : value가 Type이면 Type으로 캐스팅, 아니면 ClassCastException 발생
> * **value `as?` Type** : value가 Type이면 Type으로 캐스팅, value가 null이면 null, value가 Type이 아니면 null                                                                                                                  

## 3. 특이한 타입 3가지
1. Any
* Java의 Object 역할 (모든 객체의 최상위 타입)
* 모든 Primitive Type의 최상의 타입도 Any이다.
* Any 자체로는 null을 포함할 수 없다. null을 허용하고 싶다면 Any?로 표현해야 한다.
* Any에 equals / hashCode / toString 존재

2. Unit
* Java의 void와 동일한 역할 
* void와 다르게 Unit은 그 자체로 타입 인자로 사용 가능하다.
  * Java에서는 제네릭 타입 인자로 void를 쓰려면 Void를 사용해야 하지만, Kotlin은 Unit 타입 자체로 사용 가능 

3. Nothing
* 함수가 정상적으로 끝나지 않았다는 사실을 표현하는 역할
* 무조건 예외를 반환하는 함수 / 무한 루프 함수 등 
```kotlin
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}
```                                                                                                                                                                                                                                                                      
