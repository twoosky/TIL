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
