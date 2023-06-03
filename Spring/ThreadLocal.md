# ThreadLocal
* 해당 쓰레드만 접근할 수 있는 특별한 저장소
* WAS는 쓰레드 풀에서 쓰레드 단위로 요청에 할당하여 처리하기 때문에 동시성 이슈가 있는 저장 항목은 쓰레드 로컬을 통해서 각각 처리해야함
* 특히 스프링 빈 처럼 싱글톤 객체의 필드나 static 같은 공용 필드를 변경하며 사용할 때 이러한 동시성 문제가 자주 발생
* 동시성 문제는 값을 읽기만 하면 발생하지 않고 어디선가 값을 변경하기 때문에 발생
* 바는 언어차원에서 쓰레드 로컬을 지원하기 위한 java.lang.ThreadLocal 클래스를 제공

**동시성 문제가 있는 코드**
```java
@Slf4j
public class FieldLogTrace implements LogTrace {

   private static final String START_PREFIX = "-->"; 
   private static final String COMPLETE_PREFIX = "<--"; 
   private static final String EX_PREFIX = "<X-";
   
   private TraceId traceIdHolder; //traceId 동기화, 동시성 이슈 발생
   
   @Override
   public TraceStatus begin(String message) {
      syncTraceId();
      TraceId traceId = traceIdHolder;
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
      if (traceIdHolder == null) {
         traceIdHolder = new TraceId();
      } else {
         traceIdHolder = traceIdHolder.createNextId(); 
      }
    }

    private void releaseTraceId() {
       if (traceIdHolder.isFirstLevel()) { 
          traceIdHolder = null; //destroy
      } else {
         traceIdHolder = traceIdHolder.createPreviousId();
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
* TraceId traceIdHolder 필드를 공통으로 사용하므로 여러 스레드에서 값을 갱신하는 경우 동시성 문제가 생김

**쓰레드 로컬이 적용된 코드**
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
