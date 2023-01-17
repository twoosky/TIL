# 코틀린에서 object 키워드를 다루는 방법

## 1. static 함수와 변수
* `static`: 클래스가 인스턴스화 될 때 새로운 값이 복제되는 것이 아니라, 정적으로 인스턴스끼리의 값을 공유
* `companion object`: 클래스와 동행하는 유일한 오브젝트
* 코틀린에는 static 키워드 대신 `companion object` 블록을 사용한다.
* companion object 블록 안에 넣어둔 변수와 함수가 static과 같이 사용된다.
```kotlin
fun main() {
    Person.newBaby("늘이")
}
```
```kotlin
// Kotlin
class Person private constructor (
    var name: String,
    var age: Int
) {
    companion object {
        private const val MIN_AGE = 1  // const 변수
        fun newBaby(name: String): Person {
            return Person(name, MIN_AGE);
        }
    }
}
```
* `const`: const를 붙이면 컴파일시에 변수가 할당된다. val만 사용하는 경우 런타임시에 변수가 할당된다.
* `const`는 진짜 상수에 붙이기 위한 용도, *기본 타입과 String에 붙일* 수 있다.
* 아래는 위 예시를 Java로 구현한 것이다.
```java
public class JavaPerson {

    private static final int MIN_AGE = 1;

    public static JavaPerson newBaby(String name) {
        return new JavaPerson(name, MIN_AGE);
    }

    private String name;

    private int age;

    private JavaPerson(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

**companion object 특징**
* `companion object`, 즉 동반객체도 하나의 객체로 간주된다.
* 때문에 이름을 붙일 수도 있고, interface를 구현할 수도 있다.
```kotlin
class Person private constructor (
    var name: String,
    var age: Int
) {
    companion object Factory : Log {
        private const val MIN_AGE = 1
        fun newBaby(name: String): Person {
            return Person(name, MIN_AGE);
        }

        override fun log() {
            println("나는 Person 클래스의 동행객체 Factory 에요")
        }
    }
}
```
**Java에서 Kotlin의 static field, static 함수 사용 방법**
1. companion object 이름 여부에 따른 사용 (Companion이라는 이름이 생략되어 있는 것임)
```java
Person.Companion.newBaby("늘이");  // 이름 없는 경우
Person.Factory.newBaby("늘이");  // 이름 있는 경우
```
2. @JvmStatic 어노테이션 사용 (바로 접근 가능)
```kotlin
// kotlin
companion object {
    private const val MIN_AGE = 1

    @JvmStatic
    fun newBaby(name: String): Person {
        return Person(name, MIN_AGE);
    }
}
```
```java
// Java에서 사용
Person.newBaby("늘이");
```

## 2. 싱글톤
* `싱글톤`: 단 하나의 인스턴스만을 갖는 클래스
* 코틀린에서는 `object` 키워드만으로 싱글톤 클래스를 구현할 수 있다.
```kotlin
fun main() {
    println(Singleton.a)  // 함수, 변수에 바로 접근 가능
```
```kotlin
object Singleton {
    var a: Int = 0
}
```
* 싱글톤 클래스를 Java에서 구현하면 아래와 같다. 코틀린에선 매우 간편한데!
```java
public class JavaSingleton {
    
    private static final JavaSingleton INSTANCE = new JavaSingleton();
    
    private JavaSingleton() { }
    
    public static JavaSingleton getInstance() {
        return INSTANCE;
    }
}
```

## 3. 익명 클래스
* `익명 클래스`: 특정 인터페이스나 클래스를 상속받은 구현체를 일회성으로 사용할 때 쓰는 클래스
* 코틀린에서는 `object : 타입 {}` 으로 익명 클래스를 구현한다.
* Java에서는 `new 타입 이름() {}` 으로 인터페이스나 추상클래스 등을 구현한다.
* Java 익명 클래스 코드를 Kotlin 코드로 바꿔보자
```java
// Java
public static void main(String[] args) {
    moveSomething(new Moveable() {
        @Override
        public void move() { System.out.println("움직인다~~"); }

        @Override
        public void fly() { System.out.println("난다~~"); }
    });
}

private static void moveSomething(Moveable moveable) {
    moveable.move();
    moveable.fly();
}
```
* Java의 new 키워드 대신 코틀린의 `object : 타입` 키워드로 익명 클래스를 구현해보자
```kotlin
// Kotlin
fun main() {
    moveSomething(object : Moveable {
        override fun move() = println("움직인다~~")

        override fun fly() = println("난다~~")
    })
}
```
```kotlin
private fun moveSomething(moveable: Moveable) {
    moveable.move();
    moveable.fly();
}
```

## 4. 정리
* Kotlin에서는 `companion object`를 사용해 자바의 static 변수와 함수를 구현한다.
* `companion object`도 하나의 객체로 간주되므로 이름을 붙일 수 있고, 상속받을 수 있다.
* Kotlin에서는 싱글톤 클래스를 만들 때 `object` 키워드를 사용한다.
* Kotlin에서는 익명 클래스를 만들 때 `object : 타입`을 사용한다.
