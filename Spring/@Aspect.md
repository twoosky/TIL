# @Aspect AOP
* @Aspect 는 관점 지향 프로그래밍(AOP)을 가능하게 하는 AspectJ 프로젝트에서 제공하는 애노테이션
* @Aspect 애노테이션으로 매우 편리하게 포인트컷과 어드바이스로 구성되어 있는 어드바이저 생성 기능을 지원
* 스프링은 이것을 차용해서 프록시를 통한 AOP를 가능하게함

**@Aspect 예제 코드**
```java
@Slf4j
@Aspect // 애노테이션 기반 프록시를 적용
public class LogTraceAspect {

   private final LogTrace logTrace;

   public LogTraceAspect(LogTrace logTrace) { 
      this.logTrace = logTrace;
   }

   @Around("execution(* hello.proxy.app..*(..))") // @Around는 포인트 컷
   public Object execute(ProceedingJoinPoint joinPoint) throws Throwable { // excute는 어드바이스

      TraceStatus status = null;

      try {

         String message = joinPoint.getSignature().toShortString(); // joinPoint 안에 내부에 실제 호출 대상, 전달 인자, 그리고 어떤 객체와 어떤 메서드가 호출되었는지 정보가 포함되어 있음
         status = logTrace.begin(message);
         
         //실제 로직 호출
         Object result = joinPoint.proceed();
         
         logTrace.end(status);
         return result;
         
      } catch (Exception e) {
         logTrace.exception(status, e);
         throw e; 
      }
   }
}
```
```java
@Configuration
@Import({AppV1Config.class, AppV2Config.class}) // 수동 빈 등록
public class AopConfig {

   @Bean // @Aspect 가 있어도 스프링 빈으로 등록해야함
   public LogTraceAspect logTraceAspect(LogTrace logTrace) {
         return new LogTraceAspect(logTrace);
   }

}
```
```java
@Import(AopConfig.class) 
@SpringBootApplication(scanBasePackages = "hello.proxy.app") 
public class ProxyApplication {
   public static void main(String[] args) { 
      SpringApplication.run(ProxyApplication.class, args);
   }
   
   @Bean
   public LogTrace logTrace() {
         return new ThreadLocalLogTrace();
   }
}
```

**@Aspect를 어드바이저로 변환해서 저장하는 과정**
* 실행: 스프링 애플리케이션 로딩 시점에 자동 프록시 생성기를 호출한다.
* 모든 @Aspect 빈 조회: 자동 프록시 생성기는 스프링 컨테이너에서 @Aspect 애노테이션이 붙은 스프링 빈을 모두 조회한다.
* 어드바이저 생성: @Aspect 어드바이저 빌더를 통해 @Aspect 애노테이션 정보를 기반으로 어드바이저를 생성한다.
* @Aspect 기반 어드바이저 저장: 생성한 어드바이저를 @Aspect 어드바이저 빌더 내부에 저장한다.

**자동 프록시 생성기의 작동 과정**
* 생성: 스프링 빈 대상이 되는 객체를 생성한다. ( @Bean , 컴포넌트 스캔 모두 포함)
* 전달: 생성된 객체를 빈 저장소에 등록하기 직전에 빈 후처리기에 전달한다.
* Advisor 빈 조회: 스프링 컨테이너에서 Advisor 빈을 모두 조회한다.
* @Aspect Advisor 조회: @Aspect 어드바이저 빌더 내부에 저장된 Advisor 를 모두 조회한다.
* 프록시 적용 대상 체크: 앞서 조회한 Advisor들에 포함되어 있는 포인트컷을 사용해서 해당 객체가 프록시를 적용할 대상인지 아닌지 판단한다. 이때 객체의 클래스 정보는 물론이고, 해당 객체의 모든 메서드를 포인트컷에 하나하나 모두 매칭해본다. 그래서 조건이 하나라도 만족하면 프록시 적용 대상이 된다.
* 프록시 생성: 프록시 적용 대상이면 프록시를 생성하고 프록시를 반환한다. 그래서 프록시를 스프링 빈으로 등록한다. 만약 프록시 적용 대상이 아니라면 원본 객체를 반환해서 원본 객체를 스프링 빈으로 등록한다.
* 빈 등록: 반환된 객체는 스프링 빈으로 등록
