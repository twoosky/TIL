# Reflection
* 리플렉션은 힙 영역에 로드된 Class 타입의 객체를 통해, 원하는 클래스의 인스턴스를 생성할 수 있도록 지원하고,   
* 인스턴스의 필드와 메소드를 접근 제어자와 상관 없이 사용할 수 있도록 지원하는 API이다.
* 클래스나 메서드의 메타정보를 사용해서 동적으로 호출하는 메서드를 변경할 수 있다.
* Method 인터페이스에 정의된 invoke() 메서드를 통해 특정 오브젝트의 메서드를 실행시킬 수 있다.
* invoke() 메서드는 실행시킬 대상 (obj)와 파라미터 목록(args)을 받아서 메서드를 호출한 뒤 결과를 Object 타입으로 돌려준다.
```java
public Object invoke(Object obj, Object... args)
```
**예제 코드**
* 리플랙션 적용 전
```java
@Slf4j
public class ReflectionTest {

    @Test
    void reflection0() {
        Hello target = new Hello();

        // 공통 로직1 시작
        log.info("start");
        String result1 = target.callA();
        log.info("result={}", result1);
        // 공통 로직1 종료

        // 공통 로직2 시작
        log.info("start");
        String result2 = target.callB();
        log.info("result={}", result2);
        // 공통 로직2 종료
    }

    @Slf4j
    static class Hello {
        public String callA() {
            log.info("callA");
            return "A";
        }
        public String callB() {
            log.info("callB");
            return "B";
        }
    }
}
```
* 리플랙션 적용 후
```java
@Slf4j
public class ReflectionTest {

    @Test
    void reflection1() throws Exception {
        // 클래스 정보
        Class classHello = Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello");

        Hello target = new Hello();

        Method methodCallA = classHello.getMethod("callA");
        Object result1 = methodCallA.invoke(target);
        log.info("result={}", result1);

        Method methodCallB = classHello.getMethod("callB");
        Object result2 = methodCallB.invoke(target);
        log.info("result={}", result2);
    }
}
```
실행 결과
```
hello.proxy.jdkdynamic.ReflectionTest$Hello - callA
hello.proxy.jdkdynamic.ReflectionTest - result=A
hello.proxy.jdkdynamic.ReflectionTest$Hello - callB
hello.proxy.jdkdynamic.ReflectionTest - result=B
```
* 정적인 target.callA(), target.callB() 코드를 리플랙션을 사용해서 Method 라는 메타정보로 추상화했다.
* 이를 통해 공통 로직으로 만들 수 있다.
