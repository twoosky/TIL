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

## BeanDefinition



