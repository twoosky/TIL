# 코루틴의 예외 처리와 Job의 상태 변화

## Root 코루틴이란
* 아래 예시에서 Root 코루틴은 runBlocking이고, 자식 코루틴은 두 개의 launch 코루틴이다.
```kotlin
fun main(): Unit = runBlocking {
    val job1 = launch {
        delay(1_000L)
        printWithThread("JOB 1")
    }

    val job2 = launch {
        delay(1_000L)
        printWithThread("JOB 2")
    }
}
```
**새로운 Root 코루틴 생성하는 방법**
* CoroutineScope 함수를 통해 새로운 영역을 만들고 해당 영역에 launch 코루틴을 만들면 된다.
* 아래 예시에서는 runBlockint, 두 개의 launch 모두 Root 코루틴이다.
```kotlin
fun main(): Unit = runBlocking {
    val job1 = CoroutineScope(Dispatchers.Default).launch {
        delay(1_000L)
        printWithThread("JOB 1")
    }

    val job2 = CoroutineScope(Dispatchers.Default).launch {
        delay(1_000L)
        printWithThread("JOB 2")
    }
}
```

## lanch와 async 예외
**1. launch**
* launch 함수는 예외가 발생하자마자 해당 예외를 출력하고 코루틴이 종료된다.
```kotlin
fun main(): Unit = runBlocking {
    val job = CoroutineScope(Dispatchers.Default).launch {
        throw IllegalArgumentException()
    }

    delay(1_000L)
}
```
```
Exception in thread "DefaultDispatcher-worker-1" java.lang.IllegalArgumentException
	at org.example.Main5Kt$main$1$job$1.invokeSuspend(Main5.kt:11)
```

**2. async**
* async 함수는 예외가 발생하더라도 예외를 출력하지 않는다.
* async 함수에서 예외를 확인하고 싶다면, await()을 사용해야 한다.
```kotlin
fun main(): Unit = runBlocking {
    val job = CoroutineScope(Dispatchers.Default).async {
        throw IllegalArgumentException()
    }

    delay(1_000L)
    job.await()
}
```
```
Exception in thread "main" java.lang.IllegalArgumentException
	at org.example.Main5Kt$main$1$job$1.invokeSuspend(Main5.kt:12)
```

**3. Root 코루틴이 아닌 경우 lanch, async 예외**
* 자식 코루틴에서 발생한 예외는 모두 부모 코루틴으로 전파된다.
* 따라서, root 코루틴이 아닌 경우 launch, async 함수 모두 예외가 발생하면 바로 예외를 출력한다.
```kotlin
fun main(): Unit = runBlocking {
    val job = async {   // launch 로 변경해도 동일
        throw IllegalArgumentException()
    }

    delay(1_000L)
    job.await()
}
```
```
Exception in thread "main" java.lang.IllegalArgumentException
	at org.example.Main5Kt$main$1$job$1.invokeSuspend(Main5.kt:12)
```

**3-1. 부모 코루틴에게 예외를 전파하지 않는 방법**
* `SupervisorJob()`을 사용하는 것이다.
* async, launch 함수에 SupervisorJob()를 넣어주면, async, launch 함수가 루트 코루틴일 때의 행동 패턴과 동일하게 예외가 발생한다.
```kotlin
fun main(): Unit = runBlocking {
    val job = async(SupervisorJob()) {
        throw IllegalArgumentException()
    }

    delay(1_000L)
    job.await()
}
```
```
Exception in thread "main" java.lang.IllegalArgumentException
	at org.example.Main5Kt$main$1$job$1.invokeSuspend(Main5.kt:13)
```

## 예외를 다루는 방법
**1. try catch finally 사용**
* try catch finally를 사용하여 발생한 예외를 잡아 코루틴이 취소되지 않도록 하거나, 적절한 처리를 한 뒤 다시 예외를 던지도록 할 수 있다.
```kotlin
fun main(): Unit = runBlocking {
    val job = launch {
        try {
            throw IllegalArgumentException()
        } catch (e: IllegalArgumentException) {
            printWithThread("정상 종료")
        }
    }
}
```
```
[main] 정상 종료
```

**2. CoroutineExceptionHandler**
* 예외가 발생한 이후 에러를 로깅하거나, 에러 메시지를 보내는 등 공통된 로직을 처리하는 경우 CoroutineExceptionHandler 객체 사용
* launch에만 적용 가능하고, 부모 코루틴이 있으면 동작하지 않는다.
* `coroutineContext`: 코틀린 구성 요소
* `throwable`: 발생한 예외
```kotlin
fun main(): Unit = runBlocking {
    val exceptionHandler = CoroutineExceptionHandler { coroutineContext, throwable ->
        printWithThread("예외")
    }

    val job = CoroutineScope(Dispatchers.Default).launch(exceptionHandler) {
        throw IllegalArgumentException()
    }

    delay(1_000L)
}
```
```
[DefaultDispatcher-worker-1] 예외
```
> try catch는 코루틴 자체에서 예외를 잡아 처리하는 것이고, coroutineExceptionHandler는 예외는 이미 발생한 상황에서 해당 예외를 처리하는 것이다.

## Job의 상태 변화
코루틴 내부에서 발생한 예외는 아래와 같이 처리한다.

**1. 발생한 예외가 `CancellationException`인 경우: 취소로 간주하고, 부모에게 전파하지 않는다.**
<img width="532" alt="Screen Shot 2024-03-05 at 11 59 48 PM" src="https://github.com/twoosky/TIL/assets/50009240/1c6f9582-8e4c-4555-b2c5-37834559541f">

**2. 다른 예외가 발생한 경우: 실패로 간주하고, 부모 코루틴에게 전파한다.**
* 코루틴은 CancellationException 포함 모든 예외에 대해 *취소됨 상태*로 간주한다.
* COMPLETING 상태가 존재하는 이유는 자식 코루틴 중 하나에서 예외가 발생하면 다른 자식 코루틴들에게도 취소 요청을 보내기 때문이다.
  * 즉, 모든 자식 코루틴이 정상적으로 작업을 끝내야 COMPLETED 상태가 된다.
<img width="683" alt="Screen Shot 2024-03-06 at 12 03 15 AM" src="https://github.com/twoosky/TIL/assets/50009240/9e9b7898-68ee-4a80-9452-856a1abd8681">

