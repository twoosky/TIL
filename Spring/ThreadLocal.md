# ThreadLocal
* 해당 쓰레드만 접근할 수 있는 특별한 저장소
* WAS는 쓰레드 풀에서 쓰레드 단위로 요청에 할당하여 처리하기 때문에 동시성 이슈가 있는 저장 항목은 쓰레드 로컬을 통해서 각각 처리해야함
* 특히 스프링 빈 처럼 싱글톤 객체의 필드나 static 같은 공용 필드를 변경하며 사용할 때 이러한 동시성 문제가 자주 발생
* 동시성 문제는 값을 읽기만 하면 발생하지 않고 어디선가 값을 변경하기 때문에 발생
* 바는 언어차원에서 쓰레드 로컬을 지원하기 위한 java.lang.ThreadLocal 클래스를 제공

## 동시성 문제
* 동시성 문제 : 여러 쓰레드가 동시에 같은 인스턴스의 필드 값을 변경하면서 발생하는 문제
* `싱글톤 스프링 빈`에서는 JVM에 객체 당 하나의 인스턴스만 존재한다.
* 따라서, `하나의 인스턴스 필드에 여러 쓰레드가 동시에 접근`하는 경우 동시성 문제가 발생한다.

**동시성 문제 예시**
* 여러 쓰레드가 동시에 FieldService 인스턴스의 nameStore 필드값을 변경해 동시성 문제 발생
```java
@Slf4j
public class FieldService {

    private String nameStore;

    public String logic(String name) {
        log.info("저장 name={} -> nameStore={}", name, nameStore);
        nameStore = name;
        sleep(1000);   // 1초 대기
        log.info("조회 nameStore={}", nameStore);
        return nameStore;
    }

    private void sleep(int millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
```java
@Slf4j
public class FieldServiceTest {

    private FieldService fieldService = new FieldService();

    @Test
    void field() {
        log.info("main start");
        Runnable userA = () -> {
            fieldService.logic("userA");
        };
        Runnable userB = () -> {
            fieldService.logic("userB");
        };

        Thread threadA = new Thread(userA);
        threadA.setName("thread-A");
        Thread threadB = new Thread(userB);
        threadB.setName("thread-B");

        threadA.start(); // Runnable 시작
        // sleep(2000);  // 동시성 문제 발생 X, fieldService.logic() 메서드 실행 끝날때 까지 대기
        sleep(100);  // 동시성 문제 발생 O
        threadB.start();

        sleep(3000);  // 메인 쓰레드 종료 대기
        log.info("main exit");
    }

    private void sleep(int millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

**실행 결과**
* `동시성 문제 발생 X` : sleep(2000), threadA 수행이 끝날 때까지 대기
```
[Test worker] main start
[Thread-A] 저장 name=userA -> nameStore=null
[Thread-A] 조회 nameStore=userA
[Thread-B] 저장 name=userB -> nameStore=userA
[Thread-B] 조회 nameStore=userB
[Test worker] main exit
```
* `동시성 문제 발생 O` : sleep(100), threadA 수행이 끝나기 전 threadB 동시 수행
```
[Test worker] main start
[Thread-A] 저장 name=userA -> nameStore=null
[Thread-B] 저장 name=userB -> nameStore=userA
[Thread-A] 조회 nameStore=userB
[Thread-B] 조회 nameStore=userB
[Test worker] main exit
```

## ThreadLocal 적용해 동시성 문제 해결
```java
@Slf4j
public class ThreadLocalLogTrace implements LogTrace {

   private static final String START_PREFIX = "-->"; 
   private static final String COMPLETE_PREFIX = "<--"; 
   private static final String EX_PREFIX = "<X-";

   private ThreadLocal<TraceId> traceIdHolder = new ThreadLocal<>();
   
   @Override
   public TraceStatus begin(String message) {
      syncTraceId();
      TraceId traceId = traceIdHolder.get();
      Long startTimeMs = System.currentTimeMillis();
      log.info("[{}] {}{}", traceId.getId(), addSpace(START_PREFIX, traceId.getLevel()), message);
      return new TraceStatus(traceId, startTimeMs, message);
   }
   
   @Override
   public void end(TraceStatus status) {
      complete(status, null);
   }
   
   @Override
   public void exception(TraceStatus status, Exception e) {
      complete(status, e);
   }

   private void complete(TraceStatus status, Exception e) { 
      Long stopTimeMs = System.currentTimeMillis();
      long resultTimeMs = stopTimeMs - status.getStartTimeMs(); 
      TraceId traceId = status.getTraceId();

      if (e == null) {
         log.info("[{}] {}{} time={}ms", traceId.getId(), addSpace(COMPLETE_PREFIX, traceId.getLevel()), status.getMessage(), resultTimeMs);
      } else {
         log.info("[{}] {}{} time={}ms ex={}", traceId.getId(), addSpace(EX_PREFIX, traceId.getLevel()), status.getMessage(), resultTimeMs, e.toString());
      }
      releaseTraceId();
   }

   private void syncTraceId() {
      TraceId traceId = traceIdHolder.get(); 
      if (traceId == null) {
         traceIdHolder.set(new TraceId()); 
      } else {
         traceIdHolder.set(traceId.createNextId()); 
      }
   }
   
   private void releaseTraceId() {
      TraceId traceId = traceIdHolder.get(); 
      if (traceId.isFirstLevel()) {
         traceIdHolder.remove(); //destroy 
      } else {
         traceIdHolder.set(traceId.createPreviousId()); 
      }
   }
   
   private static String addSpace(String prefix, int level) {
      StringBuilder sb = new StringBuilder();
   
      for (int i = 0; i < level; i++) {
         sb.append( (i == level - 1) ? "|" + prefix : "| "); 
      }
      
      return sb.toString(); 
   }
}
```
* 쓰레드 로컬이 적용되니 ThreadLocal traceIdHolder를 공통으로 사용하므로 여러 스레드에서 값을 갱신하는 경우 동시성 문제가 생기지 않음

**쓰레드 로컬 사용법**
* 값 저장: ThreadLocal.set(value)
* 값 조회: ThreadLocal.get()
* 값 제거: ThreadLocal.remove()
* 쓰레 로컬 주의 사항
* 쓰레드를 생성하는 비용은 비싸기 때문에 쓰레드를 제거하지 않고 보통 쓰레드 풀을 통해서 쓰레드를 재사용하는데 쓰레드 로컬에 저장된 값을 제거하지 않으면 다른 요청에 의해 쓰레드가 재사용될 수 있고 그때 쓰레드 로컬에 저장된 값이 노출될 수 있음
* 쓰레드 로컬을 모두 사용하고 나면 꼭 ThreadLocal.remove() 를 호출해서 쓰레드 로컬에 저장된 값을 제거해주어야함
