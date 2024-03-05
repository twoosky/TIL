# Structued Concurrency
* Structued Concurrency란 부모-자식 관계의 코루틴이 한 몸처럼 움직이는 것을 말한다.
* Structued Concurrency는 수많은 코루틴이 유실되거나 누수되지 않도록 보장한다.
* Structued Concurrency는 코드 내의 에러가 유실되지 않고 적절히 보고될 수 있도록 보장한다.

## 예외 전파에 따른 Job Life Cycle
* 아래는 `CancellationException` 이외의 예외가 발생하는 경우의 Job 생명주기이다.
* 주어진 작업이 완료된 코루틴은 바로 COMPLETED가 되는게 아니라, COMPLETING으로 처리된다.
* 그 이유는, 자식 코루틴이 있을 경우 자식 코루틴들 중 하나에서 예외가 발생하면 다른 자식 코루틴들에게도 취소 요청을 보내기 때문이다.

<img width="709" alt="image" src="https://github.com/twoosky/TIL/assets/50009240/f879fb6f-469b-4913-af3e-ce18a5c1a7de">

<br></br>

**1.예외 전파로 인해 자식 코루틴이 취소된 예시**
```kotlin
fun main(): Unit = runBlocking {
    launch {
        delay(600L)
        printWithThread("A")
    }

    launch {
        delay(500L)
        throw IllegalArgumentException("코루틴 실패!")
    }
}
```
```
Exception in thread "main" java.lang.IllegalArgumentException: 코루틴 실패!
	at org.example.Main6Kt$main$1$2.invokeSuspend(Main6.kt:15)
```
* 두번째 코루틴이 중단 상태에서 먼저 풀려나 실행된다.
* 두번째 코루틴에서 발생한 예외가 `runBlocking`에 의해 만들어진 부모 코루틴에 취소 신호를 보낸다.
* 취소 신호를 받은 부모 코루틴이 다른 자식 코루틴인 첫번째 코루틴까지 취소시키므로 첫번째 코루틴의 출력값 A는 출력되지 않는다.
> 즉, 자식 코루틴에서 발생한 예외는 부모 코루틴에 전파되고, 부모 코루틴은 모든 자식 코루틴을 취소시킨다.

**2. 예외는 전파하지만, 두 코루틴이 모두 실행된 예시**
```kotlin
fun main(): Unit = runBlocking {
    launch {
        delay(500L)
        printWithThread("A")
    }

    launch {
        delay(600L)
        throw IllegalArgumentException("코루틴 실패!")
    }
}
```
```
[main] A

Exception in thread "main" java.lang.IllegalArgumentException: 코루틴 실패!
	at org.example.Main6Kt$main$1$2.invokeSuspend(Main6.kt:15)
```
* 첫번째 코루틴이 중단 상태에서 먼저 풀려나 실행되므로 A가 출력된 뒤 exception이 발생한다.
* 이때, 두번째 코루틴에서 발생한 예외가 부모 코루틴에 전파되고, 부모 코루틴은 모든 자식 코루틴을 취소하지만, 첫번째 코루틴은 이미 실행되었으므로 출력값은 출력되는 것이다.

### 정리
* 자식 코루틴에서 예외가 발생할 경우, Structured Concurrency에 의해 부모 코루틴이 취소되고, 부모 코루틴의 다른 자식 코루틴들도 취소된다.
* 자식 코루틴에서 예외가 발생하지 않더라도, 부모 코루틴이 취소되면, 자식 코루틴들이 취소된다.
* 다만, `CancellationException`의 경우 정상적인 취소로 간주하기 때문에 부모 코루틴에 전파되지 않고, 부모 코루틴의 다른 자식 코루틴을 취소시키지도 않는다.
