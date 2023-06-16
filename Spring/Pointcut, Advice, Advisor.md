# 포인트컷, 어드바이스, 어드바이저
* `포인트컷: Pointcut` : 부가 기능을 적용할 대상을 판단하는 필터링 로직이다. 주로 클래스와 메서드 이름으로 필터링 한다.
* `어드바이스: Advice` : 프록시가 호출하는 부가 기능이다. 단순히 프록시가 적용할 부가 기능 로직
* `어드바이저: Advisor` : 단순하게 하나의 포인트컷과 하나의 어드바이스를 가지고 있는 것

<img src="https://github.com/twoosky/TIL/assets/50009240/2242d8ca-692a-4c4a-b680-b46fd140b99c" width="600" height="330">


## 어드바이저 예제
스프링에서 제공하는 Pointcut을 사용해 타깃의 save() 메서드에만 Advice를 적용해보자. (이를 Advisor로 생성해 프록시 팩토리에 적용)
* Advice 구현
```java
@Slf4j
public class TimeAdvice implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = invocation.proceed();

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}
```
* 프록시 팩토리에 Advisor 적용
```java
@Test
@DisplayName("스프링이 제공하는 포인트컷")
void advisorTest3() {
    ServiceImpl target = new ServiceImpl();
    ProxyFactory proxyFactory = new ProxyFactory(target);
    
    NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
    pointcut.setMappedName("save");
    
    DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(pointcut, new TimeAdvice());
    proxyFactory.addAdvisor(advisor);
    ServiceInterface proxy = (ServiceInterface) proxyFactory.getProxy();

    proxy.save();
    proxy.find();
}
```
* `new DefaultPointcutAdvisor`: Advisor 인터페이스의 가장 일반적인 구현체, 생성자로 하나의 포인트컷, 하나의 어드바이스 설정
* `new TimeAdvice()`: MethodInterceptor를 구현한 Advice
* `proxyFactory.addAdvisor(advisor)`: 프록시 팩토리에 적용할 어드바이즈 지정
* 포인트컷을 직접 구현하려면, Pointcut 인터페이스를 구현해 DefaultPointcutAdvisor의 생성자에 넣으면 된다.

```
hello.proxy.common.advice.TimeAdvice - TimeProxy 실행
hello.proxy.common.service.ServiceImpl - save 호출
hello.proxy.common.advice.TimeAdvice - TimeProxy 종료 resultTime=4
hello.proxy.common.service.ServiceImpl - find 호출
```
* save()에만 Advice의 부가 기능이 적용되었다.

## 여러 어드바이저 예제
* 스프링은 하나의 프록시에 여러 어드바이저를 적용할 수 있도록 만들어두었다.
* 어드바이저(Advisor) 적용 개수만큼 프록시가 생성되는 것이 아님! 프록시는 target 마다 하나만 생성됨
* 프록시 팩토리에 addAdvisor()를 통해 여러 어드바이저를 등록하면 된다.
* 등록하는 순서대로 advisor가 호출된다.
```java
public class MultiAdvisorTest {
    @Test
    @DisplayName("하나의 프록시")
    void multiAdvisorTest2() {
    
        // 어드바이저 생성
        DefaultPointcutAdvisor advisor1 = new DefaultPointcutAdvisor(Pointcut.TRUE, new Advice1());
        DefaultPointcutAdvisor advisor2 = new DefaultPointcutAdvisor(Pointcut.TRUE, new Advice2());
    
        // 프록시 팩토리 생성 및 여러 어드바이저 적용
        ServiceInterface target = new ServiceImpl();
        ProxyFactory proxyFactory = new ProxyFactory(target);
        proxyFactory.addAdvisor(advisor2);
        proxyFactory.addAdvisor(advisor1);
        ServiceInterface proxy = (ServiceInterface) proxyFactory.getProxy();
    
        // 실행
        proxy.save();
    }
}
```
```java
@Slf4j
static class Advice1 implements MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        log.info("advice1 호출");
        return invocation.proceed();
    }
}

@Slf4j
static class Advice2 implements MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        log.info("advice2 호출");
        return invocation.proceed();
    }
}
```
* client -> proxy(advisor2 - advisor1) -> target
```
hello.proxy.advisor.MultiAdvisorTest$Advice2 - advice2 호출
hello.proxy.advisor.MultiAdvisorTest$Advice1 - advice1 호출
hello.proxy.common.service.ServiceImpl - save 호출
```

> 스프링 AOP는 `target` 마다 하나의 프록시만 생성된다. 하나의 프록시에 여러 AOP를 적용하는 것이다.
