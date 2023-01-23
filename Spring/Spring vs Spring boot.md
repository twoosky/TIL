# Spring vs Spring boot

## Spring
* Java 기반 WebApplication 구축을 위한 오픈소스 Framework
* 객체를 관리할 수 있는 컨테이너 제공
* IoC, DI, AOP를 지원

## Spring boot
* 스프링 기반의 애플리케이션을 간편하게 설정할 수 있는 도구
* `Starter`를 통한 의존성 자동화
* 내장 서버 `Embeded Tomcat`이 포함되어 있어 Tomcat을 따로 설치/관리할 필요가 없다.

## 차이점
**1. Dependency**
* `Spring`: 의존성 설정 파일(xml)이 매우 길고, 버전 관리도 아래와 같이 하나하나 해줘야 한다.
* `Spring Boot`: starter 의존성을 통해 의존성을 자동으로 추가하고 관리해준다.
> 1. Spring
> ```xml
> <dependency>
>     <groupId>org.springframework</groupId>
>     <artifactId>spring-web</artifactId>
>     <version>5.3.5</version>
> </dependency>
> <dependency>
>     <groupId>org.springframework</groupId>
>     <artifactId>spring-webmvc</artifactId>
>     <version>5.3.5</version>
> </dependency>
> ```
> 2. Spring Boot
> ```gradle
> implementation 'org.springframework.boot:spring-boot-starter-web'
> ```

**2. Spring boot의 AutoConfiguration**
* `Spring Boot`에는 `AutoConfiguration` 기능이 있어 Class path에 존재하는 의존성을 기반으로 Spring 애플리케이션을 자동으로 구성하고, 빈 설정과 생성을 자동으로 해준다.
* @SpringBootApplication, @EnableAutoConfiguration, 을 통해 AutoConfiguration 사용 가능

**3. 편리한 배포**
* `Spring`: war 파일을 Web Application Server에 담아 배포
* `Spring Boot`: Tomcat이나, Jetty 같은 내장 WAS를 가지고 있어 jar 파일로 간편하게 배포 가능







