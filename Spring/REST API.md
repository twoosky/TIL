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
|구성 요소|내용|표현 방법|예
|---|---|---|---|
|자원(Resource)|자원|HTTP URI|/members/{1}, /member/|
|행위(Verb)|자원에 대한 행위|HTTP Method|POST, GET, DELETE, PUT|
|표현(Representations)|자원에 대한 행위의 내용 (즉, 요청에 대한 Body)|HTTP Message Payload (JSON, XML, TEXT, RSS 등)|{  member-id:”82370”,  member-name:”홍길동“,  member-org:”10100”,  member-location:”11010” }|

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
> URI로 지정한 Resource에 대한 조작을 통일되고 한정적인 인터페이스로 수행하는 아키텍처 스타일.

## REST API
REST API란 REST의 원리를 따르는 API이다.  
아래는 REST API 설계규칙이다.
### 1. URL Rules
**1-1. URI에 명사를 사용한다.**
```
[Bad Example] http://api.test.com/Run/
[Good Example]  http://api.test.com/running/  
```
**1-2. 마지막에 슬래시(/)를 포함하지 않는다.**
```
[Bad Example] http://api.test.com/test/  
[Good Example]  http://api.test.com/test
```
**1-3. 언더바 대신 하이폰을 사용한다.**
```
[Bad Example] http://api.test.com/article_comments
[Good Example]  http://api.test.com/article-comments
```
**1-4. 파일확장자는 URI에 포함하지 않는다.**
```
[Bad Example] http://api.test.com/photo.jpg  
[Good Example]  http://api.test.com/photo  
```
**1-5. URI에 행위를 포함하지 않는다.**
```
[Bad Example] POST http://api.test.com/members/delete-article/1  
[Good Example] DELETE http://api.test.com/members/article/1  
```
자원에 대한 행위는 HTTP Method(GET, POST, PUT, DELETE 등)로 표현  

**1-6. 소문자를 사용한다.**  
대소문자에 따라 다른 리소스로 인식하기 때문에 소문자로 일관성 유지

### 2. HTTP헤더
**2-1. Content-Location**  
post 요청의 대부분은 응답 리소스의 결과가 항상 동일하지 않다.    
``` JSON
"POST /users"
{
    "name": "hak"
}
```
위와 같은 요청은 매번 다른 리소스를 반환한다.  
첫번째는 `/user/1`, 두번째는 `/user/2` ... n번째는 `/user/n`  
*따라서 요청의 응답 헤더에 새로 생성된 리소스를 식별할 수 있는 `Content-Location` 속성을 명시해야 한다.*

``` HTTP
"HTTP/1.1 200 OK"
Content-Location: /users/1
```
HATEOAS로 `content-Location`을 대체할 수 있다.

**2-2. Content-Type, Accept**   
클라이언트와 서버 모두 통신을 위해 어떤 포맷이 사용되었는지 파악하기 파악하기 위해 데이터 포맷을 HTTP Header에 명시해야 한다.  
```
* Content-Type: 응답의 형식(json, xml 등)을 명시 
  * ex)application/json
* Accept: 클라이언트/서버가 선호하는 미디어 타입 명시
```

### 3. 리소스 간의 관계 표현  
리소스 간의 연관 관계는 아래와 같이 표현한다.
```
/리소스명/리소스ID/관계가 있는 다른 리소스명
GET /users/{userId}/articles
```

### 4. 자원을 표현하는 Collection과 Document  
* `Collection`: 문서들의 집합, 객체들의 집합   
* `Document`: 문서, 하나의 객체  
**Collection은 복수**, **Document는 단수**로 표현.
```
ex1) sports라는 컬렉션과 soccer라는 도큐먼트로 표현
http://api.test.com/sports/soccer

ex2) sports, players 컬렉션과 soccer, 13(13번인 선수)를 의미하는 도큐먼트로 URI가 이루어짐.
http://api.test.com/sports/soccer/players/13
```
### 5. PATCH 사용
PUT 대신 PATCH를 사용해 REST API 완성도를 높인다.  
자원의 일부를 수정할 때는 `PATCH`를 사용하자.

PUT 요청 시 요청을 일부분만 보낸 경우 나머지는 default 값으로 수정된다.  
따라서 PUT은 다음과 같이 바뀌지 않는 속성도 보내야 한다.  
ex) `level`만 변경하고 싶은 경우
```json
"PUT /users/1"
{
    "name": "hak"
    "level": 11
}
"HTTP/1.1 200 OK"
{
    "name": "hak",
    "level": 11
}
```
PATCH를 사용하면 원래의 목적대로 `level`만 변경하는 요청을 보낸다.
```json
"PATCH /users/1"
{
    "level": 11
}
"HTTP/1.1 200 OK"
{
    "name": "hak",
    "level": 11
}
```

### 6. HTTP status
의미에 맞는 HTTP status를 리턴한다.
```json
"[Bad]"
"HTTP/1.1 200 OK"
{
    "result" : false
    "status" : 400
}

"[Good]"
"HTTP/1.1 400 Bad Request"
{
    "msg" : "check your parameter"
}
```

## 7. HATEOAS 사용
HATEOAS란 응답 객체에 해당 리소스의 상태가 전이될 수 있는 link들을 함께 제공하는 것이다.  

**7-1HATEOAS 구성 요소**
* `rel`: 변경될 리소스의 상태 관계
  * `self`: 현재 URL 자신, 예약어처럼 쓰임
* `href`: 요청 URL
* `method`: 요청 Method
```json
{
    "rel": "self",
    "href": "http://api.test.com/users/1",
    "method": "GET"
}
```
**7-2 응답 예제**
```json
201 "Created"
{
    "id": 1,
    "name": "hak",
    "createdAt": "2018-07-04 14:00:00"
    "links": [
        {
            "rel": "self",
            "href": "http://api.test.com/users/1",
            "method": "GET"
        },
        {
            "rel": "delete",
            "href": "http://api.test.com/users/1",
            "method": "DELETE"
        },
        {
            "rel": "update",
            "href": "http://api.test.com/users/1",
            "method": "PATCH",
            "more_info": "http://api.test.com/docs/user-update"
            "body": {
                "name": "{The value to be modified}"
            }
        },
        {
            "rel": "user.posts",
            "href": "http://api.test.com/users/1/posts",
            "method": "GET"
        }
    ]
}
```
* `rel`값은 `self`를 제외하고 내부 규칙을 정해 따른다. 의미만 명확히 드러나면 된다.
* `more_info`, `body`와 같이 내부 정의된 key를 사용해도 된다.
* 즉, hateoas에 명시된 값은 사용자가 직접 정의해 사용할 수 있다.

### 8. 검색, 정렬, 필터링 그리고 페이징을 위한 규칙 사용**
서버에 대한 요청은 하나의 데이터셋으로 처리되는 단순한 쿼리로 진행해야한다.  
예를 들어 GET 메소드 API로 검색, 정렬, 필터링, 페이징등의 쿼리 매개변수 추가는 다음의 예처럼 처리 권장
|상태 코드|설명|예|
|---|---|---|
|Sorting|리스트를 클라이언트의 요청에 맞게 정렬 시|GET /companies?sort=rank_asc|
|Filtering|데이터셋의 데이터를 필터링 할 때, 다양한 쿼리 파라미터를 통해 필터링 처리 가능|GET /companies?category=banking&location=india (회사의 카테고리를 은행으로 정하고 은행이 위치한 장소를 인도로 필터 처리)|
|Searching|데이터 검색 시|GET /companies?search=Digital|
|Pagination|페이징 처리 시|GET /companies?page=23|

### 9. API 버전 관리
* API 버전 관리를 반드시 필수로 하고, 버전이 다른 API는 릴리즈 하지 말 것
* 간단한 서수를 표현하고 2.5등과 같은 점 표기법은 사용하지 말 것
* 일반적으로 버전은 'v' + 숫자로 표기 
* ex) `/api/v1/board/`



## RESTful이란
* RESTful이란 REST의 원리를 따르는 시스템을 의미한다.  
* 하지만 REST를 사용했다 하여 모두가 RESTful한 것은 아니다. REST API 설계 규칙을 올바르게 지킨 시스템을 RESTful하다고 할 수 있다.  
* 부적절한 HTTP Method를 사용한 API(모든 CRUD 기능을 POST로 처리 등), URI 규칙을 올바르게 지치지 않은 API 등 REST API 설계 규칙을 올바르게 지키지 못한 시스템은 REST API를 사용했지만, RESTful하지 못한 시스템이라고 할 수 있다.  


