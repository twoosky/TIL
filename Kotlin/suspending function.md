# suspending function
* suspend가 붙은 함수에서만 다른 suspend 함수를 호출할 수 있다.
* suspend 함수는 코루틴이 중지 되었다가 재개 ***될 수 있는*** 지점이다. (suspending point)
  * 반드시 중지되는 것은 아니고, 스레드 block이 발생하는 작업이 있는 경우 중지될 수 있는 지점을 의미한다.

### 코루틴 빌더와 suspend function
* runBlocking, launch, async에서 다른 함수를 받을 때, 해당 함수는 suspend 함수로 간주된다.
* launch 시그니처를 보면 마지막 파라미터가 suspending lambda이다. (함수 타입에 suspend가 붙은 것을 의미)
* launch 코드 블록에 정의한 코루틴 작업이 suspend 함수이기 때문에 블록 내에서 delay()와 같은 suspend 함수를 호출할 수 있었던 것이다.
```kotlin
public fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job
```

### suspend function 활용
1. delay(), await()과 같은 suspend function을 호출하기 위해 suspend function 활용
```kotlin
fun main(): Unit = runBlocking {
    launch { doWorld() }
    printWithThread("Hello")
}

suspend fun doWorld() {
    delay(1_000L)
    printWithThread("World!")
}
```
```
[DefaultDispatcher-worker-1] Hello
[DefaultDispatcher-worker-1] World!
```
2. 비동기 라이브러리 구현체와 상관없이 구현하기 위해 내부 코루틴을 메소드로 분리하는 경우
```kotlin
fun main() = runBlocking {
    val result1 = call1()
    val result2 = call2(result1)

    printWithThread(result2)
}

suspend fun call1(): Int {
    return CoroutineScope(Dispatchers.Default).async {
        Thread.sleep(1_000L)
        100
    }.await()
}

suspend fun call2(num: Int): Int {
    return CompletableFuture.supplyAsync {
        Thread.sleep(1_000L)
        num * 2
    }.await()
}
```
```
[main] 200
```
* await()은 suspend 함수이므로, 이를 호출하기 위해 call1(), call2() 모두 suspend 함수로 정의
* 메소드로 분리하지 않고, runBlocking 블록 안에서 코루틴을 생성한다면 모두 async의 반환 타입인 Deferred에 의존할 것이다.
* 따라서, 각 코루틴을 메소드로 분리하고, await()을 사용하기 위해 해당 메소드를 suspend로 정의
* 이를 통해 Java의 비동기 라이브러리인 CompletableFuture도 사용할 수 있게됨.
> 각 메소드에서 해당 비동기 라이브러리의 반환 타입(CompletableFuture, Deferred) 으로 반환하고, runBlocking 블록에서 await()하면, 굳이 메소드들을 suspend function으로 정의하지 않아도 된다.  
> (runBlocking 블록은 suspending lambda이기 때문. 그냥 위와 같이도 구현할 수 있구나~ 하고 넘어가자.)

3. 인터페이스에 suspend function 정의
```kotlin
interface AsyncCaller {
    suspend fun call()
}

// 각 구현체는 각기 다른 비동기 라이브를 사용해 구현할 수 있다.
class AsyncCallerImpl: AsyncCaller {
    override suspend fun call() {
        TODO("Not yet implemented")
    }
}
```

### Coroutine 라이브러리에서 제공하는 suspend 함수 종류
**1. coroutineScope**
* 추가적인 코루틴을 만들고, 주어진 함수 블록이 바로 실행된다.
* coroutineScope에 의해 만들어진 코루틴이 모두 완료되면 다음 코드로 넘어간다.
```kotlin
fun main() {
    runBlocking {
        launch {
            delay(1_000L)
            printWithThread("Hello1")
        }
        doSomething()
        launch {
            delay(100L)
            printWithThread("Hello2")
        }
    }
}

suspend fun doSomething() = coroutineScope{
    launch {
        delay(500L)
        printWithThread("Hello3")
    }
    delay(1_500L)
    printWithThread("Hello4")
}
```
```
[main] Hello3
[main] Hello1
[main] Hello4
[main] Hello2
```
* 마지막 launch 코루틴의 delay 시간이 가장 짧지만, coroutineScope 으로 만들어진 코루틴이 모두 완료되기 전까지 마지막 launch는 실행되지 않는다.

**1-2. runBlocking과 coroutineScope 차이점** ([참고](https://www.baeldung.com/kotlin/coroutines-runblocking-coroutinescope))
* runBlocking으로 만들어진 코루틴은 내부 코루틴이 모두 완료될 때까지 스레드를 blocked 하고 대기한다.
* corutineScope로 만들어진 코루틴은 스레드를 blocked 하지 않고, 이전 코루틴 작업을 처리할 수 있다. corutineScope 코루틴 이후 작업은 해당 코루틴이 모두 완료되기 전까지 실행되지 않는다.
```kotlin
fun main() {
    runBlocking {
        runBlocking {
            launch {
                delay(1_000L)
                printWithThread("Hello1")
            }
        }
        doSomething()
        launch {
            delay(100L)
            printWithThread("Hello2")
        }
    }
}

suspend fun doSomething() = coroutineScope{
    launch {
        delay(500L)
        printWithThread("Hello3")
    }
    delay(1_500L)
    printWithThread("Hello4")
}
```
```
[main] Hello1
[main] Hello3
[main] Hello4
[main] Hello2
```
* 1번 예시와 다른 점은 첫번째 launch 코루틴을 runBlocking으로 감쌌다.
* 이 경우, 첫번째 runBlocking 코루틴이 모두 실행될 때까지 스레드를 대기시키므로, 1번 예시의 출력값과 다르게 Hello1가 먼저 출력된다.

**2. withContext**
* context에 변화를 주는 기능이 추가적으로 있다.
* coroutineScope와 유사하다.
* withContext도 coroutineScope와 마찬가지로 내부 코루틴 작업이 모두 완료될 때까지 다음 코드를 실행하지 않는다.
```kotlin
fun main() {
    runBlocking {
        launch {
            delay(1_000L)
            printWithThread("Hello1")
        }
        doSomething()
        launch {
            delay(100L)
            printWithThread("Hello2")
        }
    }
}

suspend fun doSomething() = withContext(Dispatchers.Default) {
    launch {
        delay(500L)
        printWithThread("Hello3")
    }
    delay(1_500L)
    printWithThread("Hello4")
}
```
```
[DefaultDispatcher-worker-2] Hello3
[main] Hello1
[DefaultDispatcher-worker-2] Hello4
[main] Hello2
```
* Dispatchers.Default context를 추가해 withContext 코루틴을 생성했다.
* 따라서, withContext 내부 코루틴들은 다른 스레드에서 동작한다.

**3. withTimeout / withTimeoutOrNull**
* 주어진 시간 안에 새로 생긴 코루틴이 완료되어야 한다.
* coroutineScope와 기본적으로 유사하다.
```kotlin
fun main(): Unit = runBlocking {
    val result: Int = withTimeout(1_000L) {
        delay(1_500L)
        10 + 20
    }

    printWithThread(result)
}
```
```
Exception in thread "main" kotlinx.coroutines.TimeoutCancellationException: Timed out waiting for 1000 ms
```
```kotlin
fun main(): Unit = runBlocking {
    val result: Int? = withTimeoutOrNull(1_000L) {
        delay(1_500L)
        10 + 20
    }

    printWithThread(result ?: "NULL")
}
```
```
[main] NULL
```

