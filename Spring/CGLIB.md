# CGLIB
* `CGLIB`는 바이트코드를 조작해서 동적으로 프록시 클래스를 생성하는 기술을 제공하는 라이브러리이다.
* CGLIB는 사용하면 인터페이스가 없어도, 구체 클래스만 가지고 동적 프록시를 생성할 수 있다.
* 스프링 프레임워크 내부 소스 코드에 포함되어 있어 바로 사용 가능

## CGLIB 예제
**MethodInterceptor**
* 동적 프록시에서 공통 로직을 위해 InvocationHandler 인터페이스를 사용한 것과 같이
* CGLIB에서는 공통 로직을 위해 MethodInterceptor 인터페이스를 사용한다.
```java
public interface MethodInterceptor extends Callback {
    Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable;
}
```
* `Object`: CGLIB에 적용된 객체
* `Method`: 호출된 메서드
* `Object[]`: 메서드를 호출하면서 전달된 인수
* `MethodProxy`: 메서드 호출에 사용

**new Enhancer()**
* CGLIB는 Enhancer 인스턴스를 통해 프록시 클래스를 생성할 수 있다.
* CGLIB는 구체 클래스를 상속받아 프록시를 생성한다.
```java
@Test
void cglib() {
    ConcreteService target = new ConcreteService();

    Enhancer enhancer = new Enhancer();
    enhancer.setSuperclass(ConcreteService.class);
    enhancer.setCallback(new TimeMethodInterceptor(target));
    ConcreteService proxy = (ConcreteService) enhancer.create();
}
```
* `Enhancer`: CGLIB는 Enhancer를 사용해 프록시를 생성한다.
* `enhancer.setSuperClass()`: 상속받을 구체클래스 지정
* `enhancer.setCallback()`: 프록시에 적용할 공통 로직 할당 (MethodInterceptor 구현체)
* `enhancer.create()`: 위에서 지정한 클래스를 상속받아 프록시 생성

**예제 코드**
```java
@Slf4j
public class ConcreteService {
    public void call() {
        log.info("ConcreteService 호출");
    }
}
```
```java
@Slf4j
public class TimeMethodInterceptor implements MethodInterceptor {

    private final Object target;

    public TimeMethodInterceptor(Object target) {
        this.target = target;
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = methodProxy.invoke(target, args);

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}
```
```java
@Slf4j
public class CglibTest {

    @Test
    void cglib() {
        ConcreteService target = new ConcreteService();

        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(ConcreteService.class);
        enhancer.setCallback(new TimeMethodInterceptor(target));
        ConcreteService proxy = (ConcreteService) enhancer.create();
        log.info("targetClass={}", target.getClass());
        log.info("proxyClass={}", proxy.getClass());

        proxy.call();
    }
}
```
실행 결과
```
hello.proxy.cglib.CglibTest - targetClass=class hello.proxy.common.service.ConcreteService
hello.proxy.cglib.CglibTest - proxyClass=class hello.proxy.common.service.ConcreteService$$EnhancerByCGLIB$$25d6b0e3
hello.proxy.cglib.code.TimeMethodInterceptor - TimeProxy 실행
hello.proxy.common.service.ConcreteService - ConcreteService 호출
hello.proxy.cglib.code.TimeMethodInterceptor - TimeProxy 종료 resultTime=16
```
cglib 테스트 코드를 실행하면 target은 실제 객체가 출력이 되고, proxy는 CGLIB이 생성한 클래스가 출력된다.

<img src="https://github.com/twoosky/TIL/assets/50009240/16cef9f7-49ae-4245-a726-eb2eb6d83f2b" width="600" height="250">


## CGLIB 제약
* CGLIB는 클래스를 상속받아 프록시 객체를 생성하므로 몇가지 제약이 있다.
* CGLIB는 자식 클래스를 동적으로 생성하기 때문에 부모 클래스에 `기본 생성자`가 필요하다.
* 이외 의존관계는 setter를 사용해 주입해야 한다.
* 클래스에 final 키워드가 붙으면 상속이 불가능하다. -> CGLIB에서는 예외 발생
* 메서드에 final 키워드가 붙으면 해당 메서드를 오버라이딩 할 수 없다. -> CGLIB에서는 프록시 로직 동작 X 
