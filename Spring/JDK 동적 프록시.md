# JDK 동적 프록시
* 프록시 객체를 동적으로 런타임에 개발자 대신 만들어주는 기술이다.
* 그리고, 동적 프록시에 원하는 실행 로직을 지정할 수 있다.
* JDK 동적 프록시는 인터페이스가 필수이다.

**프록시 구현의 문제점**
* 프록시 패턴, 데코레이터 패턴에서는 프록시를 생성하기 위해 매번 타깃 인터페이스의 모든 메서드를 구현하고,   
  실제 객체의 메서드를 호출하는 코드를 넣어야 했다.
* 타깃 객체 개수만큼 프록시 객체를 생성해야 했다.
* 부가기능 코드가 중복될 가능성이 많다. (각 메서드마다 부가기능 중복 작성)
* 위 문제들을 해결하기 위해 `JDK 동적 프록시` 기술을 사용한다.

## 동적 프록시 예제

**newProxyInstance()**
* Java에서 제공해주는 reflection API의 newProxyInstance() 메서드를 통해 런타임 시점에 프록시 클래스 생성
* 따라서, 대상 클래스 수만큼 프록시 클래스를 만들어야 하는 단점을 보완해준다.
```java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
```
* `ClassLoader`: 동적 프록시 클래스의 로딩에 사용될 클래스 로더  
* `Class`: 프록시 클래스가 구현할 인터페이스 목록 (배열)  
* `InvocationHandler`: 메서드가 호출되었을 때 실행될 부가기능과 위임 코드를 담은 핸들러

**InvocationHandler**
* invoke() 메서드는 런타임 시점에 생성된 동적 프록시의 메서드가 호출되었을 때 실행되는 메서드
* 해당 인터페이스를 구현해 JDK 동적 프록시에 적용할 공통 로직을 개발할 수 있다.
```java
public interface InvocationHandler {
    Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
}
```
* `Object`: 프록시 객체
* `Method`: 호출한 메서드 정보
* `Object[]`: 메서드에 전달된 파라미터

**예제 코드**
```java
public interface AInterface {
    String call();
}
```
```java
@Slf4j
public class AImpl implements AInterface {

    @Override
    public String call() {
        log.info("A 호출");
        return "A";
    }
}
```
* InvocationHandler 구현체
```java
@Slf4j
public class TimeInvocationHandler implements InvocationHandler {

    private final Object target;  // 프록시가 호출할 실제 객체

    public TimeInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = method.invoke(target, args);

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}
```
* 동적 프록시 사용 테스트
```java
@Slf4j
public class JdkDynamicProxyTest {

    @Test
    void dynamicA() {
        AInterface target = new AImpl();
        TimeInvocationHandler handler = new TimeInvocationHandler(target);  // 동적 프록시에 적용할 핸들러 로직

        AInterface proxy = (AInterface) Proxy.newProxyInstance(AInterface.class.getClassLoader(), new Class[]{AInterface.class}, handler);
        proxy.call();

        log.info("targetClass={}", target.getClass());
        log.info("proxyClass={}", proxy.getClass());
    }
}
```
* 실행 결과
```
hello.proxy.jdkdynamic.code.TimeInvocationHandler - TimeProxy 실행
hello.proxy.jdkdynamic.code.AImpl - A 호출
hello.proxy.jdkdynamic.code.TimeInvocationHandler - TimeProxy 종료 resultTime=0
hello.proxy.jdkdynamic.JdkDynamicProxyTest - targetClass=class hello.proxy.jdkdynamic.code.AImpl
hello.proxy.jdkdynamic.JdkDynamicProxyTest - proxyClass=class com.sun.proxy.$Proxy12
```

**실행 과정**
1. 클라이언트는 JDK 동적 프록시 생성
2. JDK 동적 프록시의 call() 실행
3. JDK 동적 프록시는 InvocationHandler 구현체인 TimeInvocationHandler의 invoke() 호출
4. invoke() 내부 공통 로직을 수행하고, method.invoke()를 통해 target인 실제 객체(AImpl)를 호출
5. AImpl 인스턴스의 call() 실행
6. TimeInvocationHandler에서 시간로그 출력 후 결과 반환


<img src="https://github.com/twoosky/TIL/assets/50009240/1d6f01c2-3c20-4779-bc12-2e4347550886" width="600" height="250">

## 정리
* AImpl의 프록시 객체를 생성하지 않았다.
* JDK 동적 프록시를 사용해 프록시를 동적으로 만들고, TimeInvocationHandler를 공통으로 사용했다.
* 결과적으로, 동적 프록시를 사용함으로써 대상 클래스 수만큼 프록시 클래스를 만들지 않고,
* 부가 기능 로직도 InvocationHandler 구현 클래스에 모아서 단일 책임 원칙(SRP)을 지킬 수 있게 된다.
