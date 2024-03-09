# CoroutineScope과 CoroutineContext

## CoroutineScope
* CoroutineScope은 코루틴이 탄생할 수 있는 영역이고, 주요 역할은 **CoroutineContext** 데이터를 보관하는 것이다.
* launch와 async는 coroutineScope의 확장함수이다. (시그니처 확인해보기)
* CoroutineScope를 직접 만들면, runBlocking을 사용하지 않아도 된다.
  * 대신, runBlocking은 내부 코루틴이 모두 끝날 때까지 스레드를 대기시키지만, CoroutineScope는 스레드를 대기시키지 않는다. 

**예시 1**
```kotlin
fun main() {
    CoroutineScope(Dispatchers.Default).launch {
        delay(1_000L)
        printWithThread("Job 1")
    }

    Thread.sleep(1_500L)  // Job 1이 끝나기를 기다린다.
}
```
```
[DefaultDispatcher-worker-1 @coroutine#1] JOB 1
```
* runBlocking은 내부 코루틴이 모두 실행될 때까지 스레드를 blocking 해주지만, CoroutineScope은 blocking하는 기능이 없으므로, 메인 스레드를 직접 sleep 해줘야한다.

**예시 2**
```kotlin
suspend fun main() {
    val job = CoroutineScope(Dispatchers.Default).launch {
        delay(1_000L)
        printWithThread("Job 1")
    }

    job.join()
}
```
```
[DefaultDispatcher-worker-1 @coroutine#1] JOB 1
```
* join()은 해당 job이 끝날때까지 스레드를 대기시키는 함수
* join()은 suspend 함수이기 때문에 suspend 함수 내에서만 호출 가능하다.
* 따라서, join()을 사용하기 위해선 main 함수를 suspend 함수로 만들어야 한다.

## CoroutineContext
* CoroutineContext는 코루틴과 관련된 여러가지 데이터를 갖고 있다.
  * 코루틴 이름, CoroutineExceptionHandler, 코루틴 그 자체, CoroutineDispatcher 등이 들어있을 수 있다.
* Dispatcher는 코루틴이 어떤 스레드에 배정될지를 관리하는 역할이다.

**CoroutineContext 생성 과정**
* 부모 코루틴에서 자식 코루틴을 생성하면, 같은 coroutineScope 영역에 생성된다.
* 자식 코루틴을 생성할 때 coroutineScope 영역의 context를 가져오고, 필요한 정보를 덮어 써 새로운 자식 context를 만든다.
* 새로운 context를 만드는 과정에서 부모-자식 간의 관계도 설정해준다.
* 이 원리가 바로 Structured Concurrency를 작동시킬 수 있는 기반이 된다.
<img width="559" alt="image" src="https://github.com/twoosky/TIL/assets/50009240/c0936009-f8e1-49b0-816e-46dd64dcc9e7">


* 동일한 CoroutineScope에 존재하는 코루틴들은 코루틴 영역 자체를 `cancel()`해 모든 코루틴을 종료시킬 수 있다.
```kotlin
class AsyncLogic {
    private val scope = CoroutineScope(Dispatchers.Default)

    fun doSomething() {
        scope.launch {
            delay(1_000L)
            printWithThread("Job 1")
        }

        scope.launch {
            delay(1_000L)
            printWithThread("Job 2")
        }
    }

    fun destroy() {
        scope.cancel()
    }
}
```
```kotlin
fun main() {
    val asyncLogic = AsyncLogic()
    asyncLogic.doSomething()
    asyncLogic.destroy()

    Thread.sleep(1_500L)
}
```
```
// 모든 코루틴이 종료되어 아무런 값도 출력하지 않는다.
```
* CoroutineScope는 runBlocking과 다르게 스레드를 blocking하지 않으므로, delay와 함께 두 코루틴이 모두 중단되고, cancel()이 작동된다.
* cancel()에 의해 해당 코루틴 영역의 모든 코루틴이 종료되었으므로, 출력값이 존재하지 않는다.

**CoroutineContext 구조**
* CoroutineContext는 Map + Set을 합쳐놓은 형태
  * key-value로 데이터를 저장하고, 같은 key의 데이터는 유일하게 존재한다.
* key-value 쌍을 `Element`라 부르고, `+` 기호를 사용해 각 Element를 합치거나 context에 Element를 추가할 수 있다.
```kotlin
fun main() {
    CoroutineName("나만의 코루틴") + Dispatchers.Default + SupervisorJob()
}
```
* `minusKey` 함수를 이용해 context에서 Element를 제거할 수 있다.
```kotlin
fun main() {
     CoroutineScope(Dispatchers.Default).launch {
         delay(500L)
         printWithThread("Job 1")
         coroutineContext.minusKey(CoroutineName.Key)
    }

    Thread.sleep(1_000L)
}
```

## CoroutineDispatcher
* Dispatcher는 코루틴을 스레드에 배정하는 역할을 한다.
* 각 코루틴이 서로 다른 스레드에 배정되어 동작할 수 있고, 코루틴은 중단되었다가 다른 스레드 배정될 수 있다.

**Dispathcer 종류**
* Dispatchers.Default
  * JVM의 공유 스레드풀을 사용하고, 동시 작업 가능한 최대 스레드 갯수는 CPU의 코어 수와 동일하다.
  * CPU 자원을 많이 사용할 때 권장된다. (CPU intensive 작업에 유리)
* Dispatchers.IO
  * I/O 작업에 최적화된 디스패처 (DB 조회, 네트워크 요청 등)
* Dispatchers.Main
  * 보통 UI 컴포넌트를 조작하기 위한 디스패처. 특정 의존성을 갖고 있어야 사용 가능
* Java의 스레드풀인 ExecutorService를 디스패처로 변환
  * asCoroutineDispatcher() 확장함수를 사용해 ExecutorService를 dispatcher로 전환할 수 있다.
  ```kotlin
  fun main() {
      val threadPool = Executors.newSingleThreadExecutor()
      CoroutineScope(threadPool.asCoroutineDispatcher()).launch {
          printWithThread("새로운 코루틴")
      }
  }
  ```


