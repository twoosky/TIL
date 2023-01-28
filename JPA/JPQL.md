# JPQL

## 1. JPA 쿼리 작성 방법
* JPQL
  * SQL을 추상화한 객체 지향 쿼리 언어
  * SQL 문법과 유사, SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원
  * JPQL은 엔티티 객체를 대상으로 쿼리
  * SQL은 데이터베이스 테이블을 대상으로 쿼리
* JPA Criteria
  * 문자가 아닌 자바코드로 JPQL 작성 가능
  * JPQL 빌더 역할
  * JPA 공식 기능
  * 너무 복잡하고 실용성이 없다. (QueryDSL을 사용하자)
* QueryDSL
  * 문자가 아닌 자바코드로 JPQL 작성 가능
  * JPQL 빌더 역할
  * 컴파일 시점에 문법 오류를 찾을 수 있다.
  * 동적쿼리 작성 편리
* 네이티브 SQL
  * JPA가 제공하는 SQL을 직접 사용
  * JPQL로 해결할 수 없는, 특정 데이터베이스에 의존적인 기능일 때 사용
* JDBC API 직접 사용, MyBatis, SpringJdbcTemplate 함께 사용
  * JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, 마이바티스등을 함께 사용 가능
  * 단, 영속성 컨텍스트를 적절한 시점에 강제로 flush() 해줘야 한다.

> `참고`: **flush** 는 트랜잭션 commit 시점과 JPQL과 같은 쿼리 실행 시점에 발생한다.

## 2. JPQL 문법
* 엔티티와 속성은 대소문자를 구분해야 한다. (Member, age ..)
* JPQL 키워드는 대소문자를 구분하지 않아도 된다. (SELECT, FROM, where ..)
* 엔티티 이름을 사용한다. 테이블 이름 아님
* 별칭은 필수 (as는 생략 가능)
```sql
select m from Member as m where m.age > 18
```
```
select_문 :: =
    select_절
    from_절
    [where_절]
    [groupby_절]
    [having_절]
    [orderby_절]
    
update_문 :: = update_절 [where_절]
delete_문 :: = delete_절 [where_절]
```
**집합과 정렬**
* GROUP BY, HAVING, ORDER BY .. 다 똑같이 사용
```sql
select
COUNT(m),
    SUM(m.age),  // 회원 수
    AVG(m.age),  // 나이 합
    MAX(m.age),  // 평균 나이
    MIN(m.age)   // 최소 나이
from Member m
```
**TypeQuery, Query**
* TypeQuery: 반환 타입이 명확할 때 사용
* Query: 반환 타입이 명확하지 않을 때 사용
```java
TypedQuery<Member> query = em.createQuery("select m from Member m", Member.class);
Query query2 = em.createQuery("select m.name, m.age from Member m");
```
**결과 조회 API**
1. query.getResultList(): `결과가 하나 이상`일 때 리스트 반환 (결과가 없으면 빈 리스트 반환)
```java
TypedQuery<Member> query = em.createQuery("select m from Member m", Member.class);
List<Member> resultList = query.getResultList();
```
2. query.getSingleResult(): `결과가 정확히 하나`일 때 단일 객체 반환
```java
TypedQuery<Member> query = em.createQuery("select m from Member m where m.id = 1L", Member.class);
Member result = query.getSingleResult();
```
* 결과가 없는 경우: javax.persistence.NoResultException
* 결과가 두 개 이상: javax.persistence.NonUniqueResultException
* Spring Data JPA 에서는 결과가 없는 경우 Exception이 아닌 Optional 또는 null 반환

**파라미터 바인딩**
1. 이름 기반 (권장)
```java
TypedQuery<Member> query = em.createQuery("select m from Member m where m.name = :name", Member.class);
query.setParameter("name", nameParam);
```
2. 위치 기반
```
TypeQuery<Member> query = em.createQuery("SELECT m FROM Member m where m.name=?1");
query.setParameter(1, nameParam);
```


















