# String
## 1. String interpolation
Java와 Kotlin 코드를 비교하며 이해해보자

1. 문자열에 변수를 넣는 경우
* Java에서는 문자열에 변수를 넣기위해 String.format 또는 StringBuilder와 append를 사용해야 한다.
```java
Person person = new Person("이하늘", 100);
String log = String.format("사람의 이름은 %s이고 나이는 %s세 입니다.", person.getName(), person.getAge());
```
```java
StringBuilder builder = new StringBuilder();
builder.append("사람의 이름은");
builder.append(person.getName());
builder.append("이고 나이는");
builder.append(person.getAge());
builder.append("세 입니다.");
```

* Kotlin에서는 간단히 `${변수}` 를 사용해 문자열에 변수를 넣을 수 있다. 
```kotlin
val person = Person("이하늘", 100)
val log = "사람의 이름은 ${person.name}이고 나이는 ${person.age}세 입니다."
```
* 객체 멤버 변수가 아닌 단순 변수인 경우 중괄호를 생략할 수 있다.
```kotlin
val person = Person("이하늘", 100)
val name = "이하늘'
val log = "사람의 이름은 $name이고 나이는 ${person.age}세 입니다."
```
> `Tip` 변수 이름만 사용하더라도 중괄호를 생략하지 않는 것이 가독성, 일괄 변환, 정규식 활용 측면에서 좋다.

2. 여러줄에 걸친 문자열을 작성하는 경우
* `""" """`와 `trimIndent()`을 사용해 생성할 수 있다.
```kotlin
val person = Person("이하늘", 100)
val str = """
    사람의 이름은
    ${person.name}이고 
    나이는 
    ${person.age}세 입니다
""".trimIndent()

println(str)
```
```
사람의 이름은
이하늘이고 
나이는 
100세 입니다
```

## 2. String indexing
* Java에서 문자열의 특정 문자 가져오기
```java
String str = "ABCDE";
char ch = str.charAt(1);
```
* Kotlin에서 문자열의 특정 문자 가져오기
* 배열처럼 인덱싱을 통해 특정 문자를 가져올 수 있다.
```kotlin
val str = "ABCDE"
val ch = str[1]
```
