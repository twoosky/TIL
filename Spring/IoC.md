# Spring IoC

### IoC란
* IoC란 Inversion of Control의 줄임말이며, 제어의 역전이라고 한다.
* 스프링에서는 객체(빈)의 생성과 의존 관계 설정, 사용, 제거 등의 작업을 애플리케이션 코드 대신 **스프링 컨테이너가 담당한다.**
* 스프링 컨테이너가 코드 대신 객체에 대한 제어권을 갖고 있다고 해서 `IoC`라고 한다.
* 따라서, 스프링 컨테이너를 IoC 컨테이너 라고도 부른다.

### IoC 컨테이너란
* 객체(빈)를 생성하고 관리하며, 의존 관계를 연결해준다.
* 스프링에서는 IoC를 담당하는 컨테이너를 `ApplicationContext` 라고 한다.

## BeanFactory와 ApplicationContext
BeanFactory
* 스프링 컨테이너의 최상위 인터페이스
* 스프링 빈을 관리하고 조회하는 역할을 한다.
* getBean() 을 제공한다.

ApplicationContext
* ApplicationContext는 BeanFactory에 부가기능을 추가한 것이다. 
* BeanFactory 기능을 모두 상속받아 제공하고, 여러 인터페이스도 상속받는다.
  * 국제화 기능, 환경 변수 처리, 애플리케이션 이벤트, 리소스 조회
```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
		                      MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
```

* ApplicationContext의 구현체에는 여러가지가 있다.
* 구현체에 따라 스프링 컨테이너를 XML로 구성할 수도 있고, Java 클래스로 구성할 수도 있다.
* 다양한 형태로 스프링 컨테이너 구성이 가능한 이유는 스프링은 다양항 형태의 설정 정보를 `BeanDefinition`으로 추상화해서 사용하기 때문이다.

## BeanDefinition
* 빈 설정 메타정보이다. `@Bean` 당 각각 하나씩 메타 정보가 생성된다.
* 스프링 컨테이너는 이 메타정보를 기반으로 스프링 컨테이너를 생성한다.
* 스프링은 다양한 형태의 설정 정보를 `BeanDefinition`으로 추상화해서 사용한다.

<img src="https://user-images.githubusercontent.com/50009240/232467320-812e183f-6515-4cdf-85c3-94b979b62249.png" width="600" height="350">

* AnnotationConfigApplicationContext는 AnnotatedBeanDefinitionReader를 사용해서 AppConfig.class를 읽고 BeanDefinition을 생성한다.

## Java로 스프링 컨테이너 생성하기
1. 빈 구성 클래스 작성
```java
@Configuration
public class AppConfig {
    
    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }
}
```
* `@Configuration`: 1개 이상의 비을 제공하는 클래스의 경우 반드시 명시해야 한다.
* `@Bean`: 클래스를 빈으로 등록할 때 사용한다.
* @Bean이 붙은 메소드 이름은 빈 이름으로, 메소드 반환값은 빈 객체로 스프링 컨테이너에 등록

2. 스프링 컨테이너 생성
```java
ApplicationContext ac = new AnnotaionConfigApplicationContext(AppConfig.class);
```
<br></br>
**Reference**
* https://steady-coding.tistory.com/600
* https://velog.io/@max9106/Spring-ApplicationContext
