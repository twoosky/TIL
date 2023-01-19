# 코틀린의 scope function

## 1. scope function
람다를 사용해 일시적인 영역을 만들고, 코드를 더 간결하게 만들거나, method chaning에 활용하는 함수

**scope function 분류**

|반환 타입|it 사용|this 사용|
|---|---|---|
|람다의 결과|let|run|
|객체 그 자체|also|apply|
|||with|
* this: 생략이 가능한 대신, 다른 이름으로 쓸 수 없다.
* it: 생략이 불가능한 대신, 다른 이름으로 쓸 수 있다.
* with 외 함수는 확장 함수이다.
<br></br>
1. let, run, also, apply
```kotlin
val value1: Int = person.let {
    it.age
}

val value2: Int = person.run {
    this.age
}

val value3: Person = person.also {
    it.age
}

val value4: Person = person.apply {
    this.age
}
```
* let, also 은 파라미터로 `일반 함수`를 받고, run, apply 은 파라미터로 `확장 함수`를 받는다.
* 확장함수에서는 본인 자신을 this로 호출하고 ,생략 가능하다.
* 람다에서는 block 내 가장 마지막에 있는 expression이 return 값이 된다.

2. with
```kotlin
val person = Person("이하늘", 24)
with(person) {
    println(name)
    println(this.age)
}
```
* with(파라미터, 람다): this를 사용해 접근하고, this는 생략 가능하다.
* 확장함수가 아니므로 .with 와 같은 방식으로 호출 불가

## 2. scope function 사용
### let
1. 하나 이상의 함수를 call chain 결과로 호출할 때 사용
```kotlin
// 앞의 전체 결과에 대해 let 람다 수행
val strings = listOf("APPLE", "CAR")
strings.map { it.length }
    .filter { it > 3 }
    .let(::println)

// [5]
```
2. non-null 값에 대해서만 code block을 실행시킬 때 사용
```kotlin
val str: String? = "sky"
val length = str?.let {
    println(it.uppercase())
    it.length
}
```
3. 일회성으로 제한된 영역에 지역 변수를 만들 때 사용
```kotlin
// 일회성으로 변수를 가공해야 하는 경우 사용 
val numbers = listOf("one", "two", "three", "four")
val modifiedFirstItem = numbers.first()
    .let { firstItem ->
        if (firstItem.length >= 5) firstItem else "!$firstItem!"
    }.uppercase()
println(modifiedFirstItem)
```
### run
* 객체 초기화와 반환 값의 계산을 동시에 해야할 때 사용
```kotlin
val person = Person("이하늘", 24).run(personRepository::save)
```
```kotlin
val person = Person("이하늘", 24).run { 
    hobby = "독서"
    personRepository.save(this)  // 반환
}
```
> `TIP` 반복되는 생성 후처리는 run 대신 생성자, 프로퍼티, init block으로 넣는 것이 좋다.

### apply
* 객체 설정 시 객체를 수정하는 로직이 call chain 중간에 필요할 때 사용
```kotlin
fun createPerson(
    name: String,
    age: Int,
    hobby: String
) : Person {
    return Person(
        name = name,
        age = age,
    ).apply {
        this.hobby = hobby
    }
}
```
* Person 생성자로는 name, age만 받지만, createPerson 객체 생성 시 hobby도 set해줘야 되는 경우 위와 같이 작성 가능
* apply는 객체를 반환하기 때문에 위 코드에선 Person을 반환
* 회원 가입 후 정보를 추가해줘야 하는 등의 경우 apply 활용

### also
* 객체를 수정하는 로직이 call chain 중간에 필요한 경우 사용
```kotlin
mutableListOf("one", "two", "three")
  .also { println("four 추가 이전 지금 값: $it") }
  .add("four")
```

### with
* 특정 객체를 다른 객체로 변환해야 하는데, 모듈 간의 의존성에 의해 정적 팩토리 혹은 toClass 함수를 만들기 어려운 경우 사용
* with를 통해 this를 생략할 수 있어 필드가 많아도 코드가 간결해진다.
```kotlin
// Before
val person = Person("이하늘", 24)
return PersonDto(
    name = person.name,
    age = person.age
)
```
```kotlin
// After
val person = Person("이하늘", 24)
return with(person) {
    PersonDto(
        name = name,
        age = age
    )
}
```

## 4. scope function과 가독성
```kotlin
// 1번 코드
if (person != null && person.isAdult) {
    view.showPerson(person)
} else {
    view.showError()
}
```
```kotlin
// 2번 코드
person?.takeIf { it.isAdult }
    ?.let(view::showPerson)
    ?: view.showError()
```
* 구현 1의 디버깅이 쉽고, 읽기 쉽고, 수정이 쉽다.
* 2번 코드의 let에서 null을 반환한다면, showPerson(), showError()가 모두 호출되는 버그 발생
* scope function 사용시 복잡성이 증가될 수 있지만, 적절한 convention을 적용하면 유용하게 활용 가능
