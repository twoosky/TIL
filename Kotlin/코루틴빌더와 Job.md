# 코루틴빌더와 Job
* 코루틴빌더: 코루틴을 만들어주는 함수 (runBlocking, launch ..)

## 1. runBlocking
* 새로운 코루틴을 만들고 루틴 세계와 코루틴 세계를 이어준다.
* runBlocking에 의해 생성된 코루틴과, runBlocking 내부에서 생성된 코루틴이 모두 완료될 때까지 스레드를 blocking 한다.

**예시**
```kotlin
fun main() {
    runBlocking {
        printWithThread("START")
        launch {
            delay(2_000L)  // 2초간 중단하고, 다른 코루틴으로 제어권을 넘김
            printWithThread("LAUNCH END")
        }
    }

    // 다른 코루틴으로 작업을 넘겨도 여기가 실행되지 않게 된다!
    printWithThread("END")
}
```
* runBlocking에 의해 생성된 코루틴 내부에서 delay를 통해 2초간 중단하고, 다른 코루틴에게 작업을 양보했다.
* 하지만, runBlocking은 해당 코루틴이 모두 완료될 때까지 스레드를 blocking하므로, 코루틴 외부의 코드가 실행되지 않는다.

## 2. launch
* 새로운 코루틴을 생성하는 코루틴 빌더
* launch는 runBlocking 등을 통해 생성된 코루틴 내부에서만 사용 가능
* **반환값이 없는 코드를 실행**한다.
* launch는 반환값 대신 `Job`을 반환한다. Job을 통해 코루틴을 제어할 수 있다. (코루틴 시작 / 취소 / 종료시 까지 대기 등)
  * `start()`: 시작 신호
  * `cancel()`: 취소 신호
  * `join()`: 코루틴이 완료될 때까지 대기

**코루틴 시작 예시**
```kotlin
fun main(): Unit = runBlocking {
    val job = launch(start = CoroutineStart.LAZY) {
        printWithThread("Hello launch")
    }

    delay(1_000L)
    job.start()
}
```
* lauch를 통해 CoroutineStart.LAZY 옵션을 주어 코루틴이 즉시 실행되지 않도록 구현했다.
* job.start()를 직접 호출해 시작 신호를 주어야만 코루틴이 실행된다. 이처럼 코루틴의 제어가 가능하다.
* 위 예시에서는 1초 기다린 후 코루틴이 실행된다.

**코루틴 취소 예시**
```kotlin
fun main(): Unit = runBlocking {
    val job = launch {
        (1..5).forEach {
            printWithThread(it)
            delay(500)
        }
    }

    delay(1_000L)
    job.cancel()
}
```
```
[main] 1
[main] 2
```
* job.cancel()을 호출해 코루틴 실행을 취소했기 때문에 2개의 값만 출력된다.

**코루틴 종료시까지 대기 예시**
```kotlin
fun main(): Unit = runBlocking {
    val job1 = launch {
        delay(1_000L)
        printWithThread("Job 1")
    }
    
    job1.join()

    val job2 = launch {
        delay(2_000L)
        printWithThread("Job 2")
    }
}
```
```
[main] Job 1
[main] Job 2   --- 위 출력 1초 뒤 출력됨
```
* job.join()을 사용하면 해당 코루틴이 끝날 때까지 대기한 뒤 다음 코루틴을 실행할 수 있다.
* 따라서, 위와 같이 동시에 코루틴이 실행되지 않고, 코루틴1이 끝날때까지 대기 후 코루틴 2를 실행하므로, 1초 간격으로 값이 출력된다.
* join()을 호출하지 않는다면, 코루틴1이 1초를 기다리는 동안 코루틴2가 시작되어 동시에 1초를 기다리기 때문에 1.1초 정도면 두 값을 모두 출력할 수 있다.

## 3. async
* 새로운 코루틴을 생성하는 코루틴 빌더
* async는 runBlocking 등을 통해 생성된 코루틴 내부에서만 사용 가능
* 주어진 **함수의 실행 결과를 반환할 수 있다.**
* 대신 함수의 결과를 바로 반환하지 않고, `Deferred` 객체를 반환한다.
* Deferred는 Job을 상속받는 객체로 코루틴을 제어하는 기능이 존재하고, 실행된 결과를 가져오는 `await()` 함수가 추가적으로 존재한다.
* async()는 외부 자원을 동시에 호출하는 상황에서 유리하게 사용할 수 있다. (동시에 실행 가능하므로)

**3-1. await() 예시**
```kotlin
fun main(): Unit = runBlocking {
    val job = async {
        3 + 5
    }

    val result = job.await()
    printWithThread(result)
}
```
```
[main] 8
```

**3-2. await() 예시**
```kotlin
fun main(): Unit = runBlocking {
    val time = measureTimeMillis {
        val job1 = async { apiCall1() }
        val job2 = async { apiCall2() }
        printWithThread(job1.await() + job2.await())
    }

    printWithThread("소요 시간 : $time ms")
}

suspend fun apiCall1(): Int {
    delay(1_000L)
    return 5
}

suspend fun apiCall2(): Int {
    delay(1_000L)
    return 3
}
```
```
[main] 8
[main] 소요 시간 : 1025 ms
```
* async() 함수를 활용해 두 API를 동시에 호출해 대기를 동시에 함으로써 소요시간을 최소화할 수 있다.

> 동시에 대기하는 것이 어떻게 가능한가?
> * 코루틴1이 중단되면, 코루틴2가 실행되기 때문에 아래와 같이 동시에 대기할 수 있게 된다.
> * 이를 통해 여러 Job의 소요시간을 최소화할 수 있다.
> <img width="490" alt="image" src="https://github.com/twoosky/TIL/assets/50009240/187e5720-2779-4310-80a0-863d3f20ee0a">

**3-3. await() 예시: 첫번째 API의 결과가 두번째 API 실행에 필요한 경우**
```kotlin
fun main(): Unit = runBlocking {
    val time = measureTimeMillis {
        val job1 = async { apiCall1() }
        val job2 = async { apiCall2(job1.await()) }
        printWithThread(job2.await())
    }

    printWithThread("소요 시간 : $time ms")
}

suspend fun apiCall1(): Int {
    delay(1_000L)
    return 5
}

suspend fun apiCall2(num: Int): Int {
    delay(1_000L)
    return 3 + num
}
```
```
[main] 8
[main] 소요 시간 : 2028 ms
```
* job1의 결과를 await을 통해 가져와 job2에 넘겨주면된다. 위와 같이 동기식 코드로 작성할 수 있다.
* 동기 방식으로, job1의 작업이 끝나면, job2가 실행되므로 소요 시간은 2초가 걸린다.

**async 주의사항**
* CoroutineStart.LAZY 옵션을 사용하면, await() 함수를 호출했을 때 계산 결과를 계속 기다린다.
