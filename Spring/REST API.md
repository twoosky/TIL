# REST API
* [REST란](#REST란)
* [REST 구성 요소](#REST-구성-요소)
* [REST 특징](#REST-특징)
* [REST API](#REST-API)
* [RESTful이란](#RESTful이란)
## REST란
1. HTTP URI(Uniform Resource Identifier)를 통해 자원(Resource)을 명시하고,
2. HTTP Method(POST, GET, PUT, DELETE)를 통해
3. 해당 자원(URI)에 대한 CRUD Operation을 적용하는 것을 의미한다.

## REST 구성 요소
* 자원(Resource): URI
* 행위(Verb): HTTP Method
* 표현(Representations): HTTP Message Pay Load
  * JSON 또는 XML을 통해 데이터 주고 받음

## REST 특징
**1. Server-Client: 서버 클라이언트 구조**
>  REST 서버는 API 제공, 클라이언트는 사용자 인증이나 컨텍스트(세션, 로그인 정보)등을 관리하는 구조로 각각의 역할이 확실히 구분된다. 따라서 클라이언트와 서버에서 개발해야 할 내용이 명확해지고, 서로간 의존성이 줄어들게 된다.

**2. Stateless: 무상태성**
> HTTP 프로토콜은 무상태성 프로토콜이므로 REST도 무상태성을 갖는다. 따라서 서버는 클라이언트의 상태정보를 저장하고 관리하지 않는다. 세션 정보나 쿠키 정보를 별도로 저장하고 관리하지 않기 때문에 API 서버는 들어오는 요청만을 단순히 처리하면 된다. 따라서 서비스의 자유도가 높아지고 서버에서 불필요한 정보를 관리하지 않음ㅇ로써 구현이 단순해진다.

**3. Cacheable: 캐시기능**
> REST는 웹 표준 HTTP 프로토콜을 그대로 사용하기 때문에, 웹에서 사용하는 기존 인프라를 그대로 활용할 수 있다. 따라서 HTTP가 가진 캐싱 기능이 적용 가능하다. HTTP 프로토콜 표준에서 사용하는 Last-Modified태그나 E-Tag를 이용하면 캐싱 구현이 가능하다.

**4. Layered System: 계층화**
> REST 서버는 다중 계층으로 구성될 수 있다. API 서버는 순수 비즈니스 로직을 수행하고 그 앞단에 보안, 로드밸런싱, 암호화, 사용자 인증 등을 추가하여 구조상의 유연성을 둘 수 있다. 또한 PROXY, 게이트웨이와 같은 네트워크 기반의 중간매체를 사용할 수 있다.

**5. Self-descriptiveness: 자체 표현 구조**
> REST API 메시지만 보고도 쉽게 이해할 수 있는 자체 표현 구조로 구성되어 있다.

**6. Uniform Interface: 인터페이스 일관성**
> URI로 지정한 Resource에 대한 조작을 통일되고 한정적인 인터페이스로 수행한다.

## REST API
REST API란 REST의 원리를 따르는 API이다.  
아래는 REST API 설계규칙이다.

**1. URI는 동사보다는 명사를, 대문자보다는 소문자를 사용하여야 한다.**
```
[Bad Example] http://twoosky.com/Running/
[Good Example]  http://twoosky.com/run/  
```
**2. 마지막에 슬래시(/)를 포함하지 않는다.**
```
[Bad Example] http://twoosky.com/test/  
[Good Example]  http://twoosky.com/test
```
**3. 언더바 대신 하이폰을 사용한다.**
```
[Bad Example] http://twoosky.com/test-
[Good Example]  http://twoosky.com/test-blog  
```
**4. 파일확장자는 URI에 포함하지 않는다.**
```
[Bad Example] http://twoosky.com/photo.jpg  
[Good Example]  http://twoosky.com/photo  
```
**5. 행위를 포함하지 않는다.**
```
[Bad Example] http://twoosky.com/members/delete-article/1  
[Good Example]  http://twoosky.com/members/article/1  
```
자원에 대한 행위는 HTTP Method(GET, POST, PUT, DELETE 등)로 표현
```
[Bad Example] GET members/delete-article/1  
[Good Example] DELETE members/article/1  
```

## RESTful이란
* RESTful이란 REST의 원리를 따르는 시스템을 의미한다.  
* 하지만 REST를 사용했다 하여 모두가 RESTful한 것은 아니다. REST API 설계 규칙을 올바르게 지킨 시스템을 RESTful하다고 할 수 있다.  
* 부적절한 HTTP Method를 사용한 API(모든 CRUD 기능을 POST로 처리 등), URI 규칙을 올바르게 지치지 않은 API 등 REST API 설계 규칙을 올바르게 지키지 못한 시스템은 REST API를 사용했지만, RESTful하지 못한 시스템이라고 할 수 있다.  


