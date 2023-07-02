# AOP
* AOP 핵심 기능과 부가기능
  * 핵심 기능: 해당 객체가 제공하는 고유의 기능
  * 부가 기능: 핵심 기능을 보조하기 위해 제공되는 기능으로 다른 객체들과 공통적으로 사용될 가능성이 크므로 변경 지점이 하나가 될 수 있도록 잘 모듈화되어야함
* Aspect
  * Aspect: 관점이라는 뜻으로 이름 그대로 애플리케이션을 바라보는 단위를 하나하나의 기능에서 횡단 관심사(cross-cutting concerns) 관점으로 보는 것
  * AOP: 애스펙트를 사용한 프로그래밍 방식
  * AspectJ 프레임워크: 자바 프로그래밍 언어에 대한 완벽한 관점 지향 프레임워크로 스프링은 대부분 AspectJ의 문법을 차용하여 필요한 기능들만 편리하게 사용할 수 있는 AOP를 제공
  * 스프링 AOP는 AspectJ의 문법을 차용하고 프록시 방식의 AOP를 제공하는데 AspectJ를 직접 사용하는 것아니며 @Aspect 애노테이션 같은 AspectJ 프레임워크가 제공하는 애노테이션을 사용
* AOP 적용 방식
  * 컴파일 시점: .java 소스 코드를 컴파일러를 사용해서 .class 를 만드는 시점에 부가 기능 로직을 추가 (실제 코드에 부가 기능 코드 포함)
  * 클래스 로딩 시점: .class 를 JVM에 저장하기 전에 조작 (실제 코드에 부가 기능 코드 포함)
  * 런타임 시점(프록시): 스프링이 사용하는 방식으로 컴파일이 다 끝나고, 클래스 로더에 클래스도 다 올라가서 이미 자바가 실행되고 난 다음인 런타임 시점에서 프록시를 통해 스프링 빈에 부가 기능을 적용
* AOP 적용 위치
  * 적용 가능 지점(조인 포인트): 생성자, 필드 값 접근, static 메서드 접근, 메서드 실행
  * 프록시를 사용하는 스프링 AOP의 조인 포인트는 메서드 실행으로 제한

**스프링 AOP 용어 정리**
* 조인 포인트(Join point)
  * 어드바이스가 적용될 수 있는 위치, 메소드 실행, 생성자 호출, 필드 값 접근, static 메서드 접근 같은 프로그램 실행 중 지점
  * AOP를 적용할 수 있는 모든 지점
  * 스프링 AOP는 프록시 방식을 사용하므로 조인 포인트는 항상 메소드 실행 지점으로 제한
* 포인트컷(Pointcut)
  * 조인 포인트 중에서 어드바이스가 적용될 위치를 선별하는 기능
  * 주로 AspectJ 표현식을 사용해서 지정
  * 프록시를 사용하는 스프링 AOP는 메서드 실행 지점만 포인트컷으로 선별 가능
* 타켓(Target)
  * 어드바이스를 받는 실제 객체, 포인트컷으로 결정됨
* 어드바이스(Advice)
  * 부가 기능
  * 특정 조인 포인트에서 Aspect에 의해 취해지는 조치
  * Around(주변), Before(전), After(후)와 같은 다양한 종류의 어드바이스가 있음
* 에스팩트
  * 어드바이스 + 포인트컷을 모듈화 한 것
  * 여러 어드바이스와 포인트 컷이 함께 존재
* 어드바이저
  * 하나의 어드바이스와 하나의 포인트 컷으로 구성
* 위빙
  * 원본 로직에 부가 기능 로직이 추가되는 것

## AOP 구현
**스프링 AOP 기본 형태 예제 코드**
```
@Slf4j
@Aspect // 애스펙트라는 표식이지 컴포넌트 스캔이 되는 것은 아니므로 @Bean, @Component, @Import 등으로 빈으로 등록해야함
public class AspectV1 {
   
   @Around("execution(* hello.aop.order..*(..))") // AspectJ 포인트컷 표현식 (hello.aop.order 패키지와 하위 패키지)
   public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable { // doLog는 어드바이스 함수
      log.info("[log] {}", joinPoint.getSignature()); //join point 시그니처
      return joinPoint.proceed(); 
   }
}
```
```java
@Import(AspectV1.class)
@SpringBootTest
public class AopTest {

   @Autowired
   OrderService orderService;
   
   @Autowired
   OrderRepository orderRepository;
   
   @Test
   void aopInfo() {
      System.out.println("isAopProxy, orderService=" + AopUtils.isAopProxy(orderService));
      System.out.println("isAopProxy, orderRepository=" + AopUtils.isAopProxy(orderRepository));
   }
   
   @Test
   void success() {
      orderService.orderItem("itemA"); 
   }

   @Test
   void exception() {
      assertThatThrownBy(() -> orderService.orderItem("ex")) 
               .isInstanceOf(IllegalStateException.class);
   }
}
```
**포인트컷 분리**
* @Around 어드바이스에서는 포인트컷을 직접 지정해도 되지만 @Pointcut 에 포인트컷 표현식을 사용하여 분리 가능
* 메서드 이름과 파라미터를 합쳐서 포인트컷 시그니처이며 메서드의 반환 타입은 void 여야함
* 포인트컷 분리 예제 코드
```java
@Slf4j
@Aspect
public class AspectV2 {

   //hello.aop.order 패키지와 하위 패키지
   @Pointcut("execution(* hello.aop.order..*(..))") //pointcut expression
   private void allOrder(){} //pointcut signature
   
   @Around("allOrder()")
   public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
      log.info("[log] {}", joinPoint.getSignature());
      return joinPoint.proceed(); 
   }
}
```
**어드바이스 추가**
* 포인트컷은 조합할 수 있는데, && (AND), || (OR), ! (NOT) 3가지 조합이 가능
* 어드바이스 추가 예제 코드
```java
@Slf4j
@Aspect
public class AspectV3 {

   //hello.aop.order 패키지와 하위 패키지 
   @Pointcut("execution(* hello.aop.order..*(..))") 
   public void allOrder(){}
   
   //클래스 이름 패턴이 *Service 
   @Pointcut("execution(* *..*Service.*(..))") 
   private void allService(){}

   @Around("allOrder()")
   public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
      log.info("[log] {}", joinPoint.getSignature());
      return joinPoint.proceed(); 
   }
   
   //hello.aop.order 패키지와 하위 패키지 이면서 클래스 이름 패턴이 *Service 
   @Around("allOrder() && allService()")
   public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
      
      try {
         log.info("[트랜잭션 시작] {}", joinPoint.getSignature()); 
         Object result = joinPoint.proceed();
         log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
         return result;
      } catch (Exception e) {
         log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
         throw e;
      } finally {
         log.info("[리소스 릴리즈] {}", joinPoint.getSignature()); 
      }
   }
}
```
**포인트컷 참조**
* 포인트컷을 공용으로 사용하기 위해 별도의 외부 클래스에 모아두어야하며 참고로 외부에서 호출할 때는 포인트컷의 접근 제어자를 public 으로 열어야함
* 포인트컷 참조 예제 코드
```java
public class Pointcuts {

   //hello.springaop.app 패키지와 하위 패키지 
   @Pointcut("execution(* hello.aop.order..*(..))") 
   public void allOrder(){}
   
   //타입 패턴이 *Service
   @Pointcut("execution(* *..*Service.*(..))") 
   public void allService(){}
   
   //allOrder && allService
   @Pointcut("allOrder() && allService()")
   public void orderAndService(){}
}
```
```java
@Slf4j
@Aspect
public class AspectV4Pointcut {

   @Around("hello.aop.order.aop.Pointcuts.allOrder()")
   public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable { 
      log.info("[log] {}", joinPoint.getSignature());
      return joinPoint.proceed();
   }

   @Around("hello.aop.order.aop.Pointcuts.orderAndService()")
   public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
      
      try {
         log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
         Object result = joinPoint.proceed(); 
         log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
         return result;
      } catch (Exception e) {
         log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
         throw e;
      } finally {
         log.info("[리소스 릴리즈] {}", joinPoint.getSignature()); 
      }
   }
}
```
**어드바이스 순서**
* 하나의 애스펙트에 여러 어드바이스가 있으면 순서를 보장 받을 수 없으므로 애스펙트를 별도의 클래스로 분리해야함
* 어드바이스 순서 예제 코드
```java
@Slf4j
public class AspectV5Order {
   
   @Aspect
   @Order(2)
   public static class LogAspect {
      @Around("hello.aop.order.aop.Pointcuts.allOrder()")
       public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable { 
          log.info("[log] {}", joinPoint.getSignature());
          return joinPoint.proceed();
   }

   @Aspect
   @Order(1)
   public static class TxAspect {
      
      @Around("hello.aop.order.aop.Pointcuts.orderAndService()")
      public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
         
         try {
            log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
            Object result = joinPoint.proceed(); 
            log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
            return result;
         } catch (Exception e) {
            log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
            throw e;
         } finally {
            log.info("[리소스 릴리즈] {}", joinPoint.getSignature()); 
         }
      }     
   }
}
```
**어드바이스 종류**
* @Around: 메서드 호출 전후에 수행, 가장 강력한 어드바이스, 조인 포인트 실행 여부 선택, 반환 값 변환, 예외 변환 등이 가능* @Before: 조인 포인트 실행 이전에 실행
* @AfterReturning: 조인 포인트가 정상 완료후 실행
* @AfterThrowing: 메서드가 예외를 던지는 경우 실행
* @After: 조인 포인트가 정상 또는 예외에 관계없이 실행(finally)
* 모든 어드바이스는 org.aspectj.lang.JoinPoint를 첫번째 파라미터에 사용하나 @Around는 JoinPoint의 하위 타입인 ProceedingJoinPoint을 사용해야함
* @Around
  * 메서드의 실행의 주변에서 실행
  * 조인 포인트 실행 여부 선택: joinPoint.proceed()
  * 호출 여부 선택 전달 값 변환: joinPoint.proceed(args[])
  * 반환 값 변환
  * 예외 변환
  * 트랜잭션 처럼 try catch finally 모두 들어가는 구문 처리 가능
  * 어드바이스의 첫 번째 파라미터는 ProceedingJoinPoint 를 사용
  * proceed() 를 통해 대상을 실행
  * @Around 하나만 있어도 모든 기능을 수행 가능
* JoinPoint 인터페이스의 주요 기능
  * getArgs(): 메서드 인수를 반환
  * getThis(): 프록시 객체를 반환
  * getTarget(): 대상 객체를 반환
  * getSignature(): 조언되는 메서드에 대한 설명을 반환
  * toString(): 조언되는 방법에 대한 유용한 설명을 인쇄
  * ProceedingJoinPoint 인터페이스의 주요 기능
  * proceed(): 다음 어드바이스나 타켓을 호출

**어드바이스 종류 예제 코드**
```java
@Slf4j
@Aspect
public class AspectV6Advice {
   
   @Around("hello.aop.order.aop.Pointcuts.orderAndService()")
   public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
      try {
         
         // @Before
         log.info("[around][트랜잭션 시작] {}", joinPoint.getSignature()); 
         Object result = joinPoint.proceed();
         
         // @AfterReturning
         log.info("[around][트랜잭션 커밋] {}", joinPoint.getSignature());
         return result;

      } catch (Exception e) {
            
         // @AfterThrowing
         log.info("[around][트랜잭션 롤백] {}", joinPoint.getSignature());
         throw e;

      } finally {
         
         // @After
         log.info("[around][리소스 릴리즈] {}", joinPoint.getSignature()); 
      }
   }

   @Before("hello.aop.order.aop.Pointcuts.orderAndService()") 
   public void doBefore(JoinPoint joinPoint) {
      log.info("[before] {}", joinPoint.getSignature()); 
   }
   
   @AfterReturning(value = "hello.aop.order.aop.Pointcuts.orderAndService()", returning = "result")public void doReturn(JoinPoint joinPoint, Object result) { 
      log.info("[return] {} return={}", joinPoint.getSignature(), result);
   }
   
   @AfterThrowing(value = "hello.aop.order.aop.Pointcuts.orderAndService()", throwing = "ex")
   public void doThrowing(JoinPoint joinPoint, Exception ex) { 
      log.info("[ex] {} message={}", joinPoint.getSignature(), ex.getMessage()); 
   }
   
   @After(value = "hello.aop.order.aop.Pointcuts.orderAndService()") 
   public void doAfter(JoinPoint joinPoint) {
      log.info("[after] {}", joinPoint.getSignature()); 
   }
}
```

## 스프링 AOP 포인트컷
* 포인트컷 지시자
  * 포인트컷 표현식은 애스펙트J가 제공하는 포인트컷 표현식을 줄여서 말하는 것으로 execution 같은 포인트컷 지시자(Pointcut Designator)로 시작
* 포인트컷 지시자의 종류
  * execution: 메소드 실행 조인 포인트를 매칭 (스프링 AOP에서 가장 많이 사용)
  * within: 특정 타입 내의 조인 포인트를 매칭
  * args: 인자가 주어진 타입의 인스턴스인 조인 포인트
  * this: 스프링 빈 객체(스프링 AOP 프록시)를 대상으로 하는 조인 포인트
  * target: Target 객체(스프링 AOP 프록시가 가르키는 실제 대상)를 대상으로 하는 조인 포인트
  * @target: 실행 객체의 클래스에 주어진 타입의 애노테이션이 있는 조인 포인트
  * @within: 주어진 애노테이션이 있는 타입 내 조인 포인트
  * @annotation: 메서드가 주어진 애노테이션을 가지고 있는 조인 포인트를 매칭
  * @args: 전달된 실제 인수의 런타임 타입이 주어진 타입의 애노테이션을 갖는 조인 포인트
  * bean: 스프링 전용 포인트컷 지시자, 빈의 이름으로 포인트컷을 지정

**execution**
* execution 문법: execution(접근제어자 반환타입 선언타입/메서드이름(파라미터) 예외)
  * 메소드 실행 조인 포인트를 매칭
  * 접근제어자, 선언타입, 예외는 생략 가능
  * * 같은 패턴 지정 가능
  * * 은 아무 값이 들어와도 된다는 뜻
  * 파라미터에서 ..은 파라미터의 타입과 파라미터 수가 상관없다는 뜻
  * 패키지에서 .은 정확하게 해당 위치의 패키지 뜻이고 ..은 해당 위치의 패키지와 그 하위 패키지까지 포함
* execution 문법 예시1: "@Around(execution(public String hello.aop.member.MemberServiceImpl.hello(String)))"
  * 접근제어자: public
  * 반환타입: String
  * 선언타입(패키지와 클래스명): hello.aop.member.MemberServiceImpl
  * 메서드이름: hello
  * 파라미터: (String)
  * 예외: 생략
* execution 문법 예시2: "@Around(execution(* *(..)))"
  * 접근제어자: 생략
  * 반환타입: *
  * 선언타입(패키지와 클래스명): 생략
  * 메서드이름: *
  * 파라미터: (..)
  * 예외: 생략
* execution 타입 매칭 예시
```java
// 타입 정보가 정확하게 일치
@Test
void typeExactMatch() {
   pointcut.setExpression("execution(* hello.aop.member.MemberServiceImpl.*(..))"); 
   assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue(); }

// 부모 타입을 선언해도 그 자식 타입은 매칭
@Test
void typeMatchSuperType() {
   pointcut.setExpression("execution(* hello.aop.member.MemberService.*(..))");
   assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
}
```
* execution 파라미터 매칭 예시
```java
// String 타입의 파라미터 허용 
// (String)
@Test
void argsMatch() { 
   pointcut.setExpression("execution(* *(String))"); 
   assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue(); 
}

// 파라미터가 없어야 함
// ()
@Test
void argsMatchNoArgs() {
   pointcut.setExpression("execution(* *())");
   assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isFalse();
}

// 정확히 하나의 파라미터 허용, 모든 타입 허용 
// (Xxx)
@Test
void argsMatchStar() {
   pointcut.setExpression("execution(* *(*))");
   assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
}

// 숫자와 무관하게 모든 파라미터, 모든 타입 허용 
// 파라미터가 없어도 됨
// (), (Xxx), (Xxx, Xxx)
@Test
void argsMatchAll() { 
   pointcut.setExpression("execution(* *(..))"); 
   assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue(); 
}

// String 타입으로 시작, 숫자와 무관하게 모든 파라미터, 모든 타입 허용 
// (String), (String, Xxx), (String, Xxx, Xxx) 허용
@Test
void argsMatchComplex() {
   pointcut.setExpression("execution(* *(String, ..))");
   assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
}
```
* execution 예시
```java
@Aspect
@Component
public class MeasureExecutionTimeAspect {
	
	// repository에 있는 모든 클래스의 메서드
	@Around("execution(* *..repository.*.*(..))")
	public Object aroundAdvice( ProceedingJoinPoint pjp) throws Throwable {
		// before advice
		StopWatch sw = new StopWatch();
		sw.start();
		
		Object result = pjp.proceed();
		
		// after advice
		sw.stop();
		Long total = sw.getTotalTimeMillis();
		
		// 어떤 클래스의 메서드인지 출력하는 정보는 pjp 객체에 있다.
		String className = pjp.getTarget().getClass().getName();
		String methodName = pjp.getSignature().getName();
		String taskName = className + "." + methodName;
		
		System.out.println("[ExecutionTime] " + taskName + " , " + total + "(ms)");
		
		return result;
	}
}
```
**@annotation**
* @annotation 문법: @annotation(선언타입/메서드이름)
* @annotation 문법 예시: @Around("@annotation(hello.aop.member.annotation.MethodAop)")
* 어노테이션의 파라미터에 값을 넣어 AOP에 전달도 가능
* @annotation 예시
```
@Retention(RetentionPolicy.RUNTIME)
public @interface TargetAnno {
   String name() default "";
   String code() default "a1000";
}
```
```java
@Aspect
@Component
public class MainAOP { 
   @Around("@annotation(org.defaultpj.sample.annotation.TargetAnno) && @ annotation(target)")
   public Object customAnnoTest(ProceedingJoinPoint joinPoint, TargetAnno target) throws Throwable {
      System.out.println("name : " + target.name());
      System.out.println("code : " + target.code()  );
      Object returnPoint = joinPoint.proceed();
      return returnPoint;
   }
}
@TargetAnno(name = "심해펭귄", code = "A-1001")
@RequestMapping(value = "/", method = RequestMethod.GET)
public String index(Model model) {
   return "test";
}
```

## 스프링 AOP 실무 주의사항
**내부 호출 문제**
* AOP를 적용하려면 항상 프록시를 통해서 대상 객체(Target)을 호출해야하며 프록시를 스프링 빈으로 등록하여 프록시 객체가 주입되기 때문에 대상 객체를 직접 호출하는 문제는 일반적으로 발생하지 않는데, 대상 객체의 내부에서 메서드 호출이 발생하면 프록시를 거치지 않고 대상 객체를 직접 호출하기에 프록시 방식의 AOP는 메서드 내부 호출에 프록시를 적용할 수 없음
* AspectJ를 사용하면 이런 문제가 발생하지 않으나 실무에서는 거의 사용하지 않음
* 내부 호출 문제 예시 코드
```
@Slf4j
@Component
public class CallServiceV0 {
   public void external() {
      log.info("call external");
      internal(); //내부 메서드 호출(this.internal())
   }

   public void internal() { 
      log.info("call internal");
   } 
}
```
```java
@Slf4j
@Aspect
public class CallLogAspect {
   
   @Before("execution(* hello.aop.internalcall..*.*(..))") 
   public void doLog(JoinPoint joinPoint) {
      log.info("aop={}", joinPoint.getSignature()); 
   }
}
```
```java
@Import(CallLogAspect.class) 
@SpringBootTest
class CallServiceV0Test {

   @Autowired
   CallServiceV0 callServiceV0;
   
   @Test
   void external() {
      callServiceV0.external(); 
   }
   
   @Test
   void internal() {
      callServiceV0.internal(); 
   }
}
```
* 내부 호출 문제 대안 1
```java
/**
* 자기 자신 주입
*/
@Slf4j
@Component
public class CallServiceV1 {
   
   private CallServiceV1 callServiceV1;
   
   @Autowired
   public void setCallServiceV1(CallServiceV1 callServiceV1) { // 생성자 주입은 순환 사이클을 만들기 때문에 실패하여 setter 주입
      this.callServiceV1 = callServiceV1; 
   }
   
   public void external() {
      log.info("call external"); 
      callServiceV1.internal(); //외부 메서드 호출
   }
   
   public void internal() { 
      log.info("call internal");
   } 
}
```
* 내부 호출 문제 대안 2
```java
/**
* ObjectProvider(Provider)나 ApplicationContext를 사용해서 지연(LAZY) 조회
*/
@Slf4j
@Component
@RequiredArgsConstructor
public class CallServiceV2 {
  
  //  private final ApplicationContext applicationContext;
   private final ObjectProvider<CallServiceV2> callServiceProvider;
   
   public void external() { 
      log.info("call external");
      
      // CallServiceV2 callServiceV2 = applicationContext.getBean(CallServiceV2.class);
      CallServiceV2 callServiceV2 = callServiceProvider.getObject(); 
      callServiceV2.internal(); //외부 메서드 호출
   }
   
   public void internal() { 
      log.info("call internal");
   } 
}
```
* 내부 호출 문제 대안 3
```java
/**
* 구조를 변경(분리)
*/
@Slf4j
@Component
@RequiredArgsConstructor
public class CallServiceV3 {
   
   private final InternalService internalService;
   
   public void external() {
      log.info("call external"); 
      internalService.internal(); //외부 메서드 호출
   } 
}

```java
@Slf4j
@Component
public class InternalService {

    public void internal() {
        log.info("call internal"); 
   }
}
```
* 프록시 기술과 한계
* JDK 동적 프록시 인터페이스 기반으로 프록시를 생성하기 때문에 구체 클래스로 타입 캐스팅이 불가능
* CGLIB 은 구체 클래스 기반으로 하기 때문에 final 키워드 클래스, 메서드 사용이 불가능
