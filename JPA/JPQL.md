# JPQL

**JPA 쿼리 작성 방법**
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

## 1. JPQL 기본 문법
* 엔티티와 속성은 대소문자를 구분해야 한다. (Member, age ..)
* JPQL 키워드는 대소문자를 구분하지 않아도 된다. (SELECT, FROM, where ..)
* 엔티티 이름을 사용한다. 테이블 이름 아님
* 별칭은 필수 (as는 생략 가능)
```
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
```java
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
```java
TypeQuery<Member> query = em.createQuery("SELECT m FROM Member m where m.name=?1");
query.setParameter(1, nameParam);
```

## 2. 프로젝션
* SELECT 절에 조회할 대상을 지정하는 것
* 프로젝션 대상: 엔티티, 임베디드 타입, 스칼라 타입(숫자, 문자 등 기본 데이터 타입)
* 엔티티 프로젝션 쿼리에서 반환된 엔티티는 모두 영속성 컨텍스트에 의해 관리된다.

**예시**
1. 엔티티 프로젝션: `SELECT m FROM Member m`
2. 엔티티 프로젝션: `SELECT t FROM Member m JOIN m.team t` (연관된 엔티티 조회 시 join 사용 권장)
3. 임베디드 타입 프로젝션: `SELECT m.address FROM Member m`
4. 스칼라 타입 프로젝션: `SELECT m.name, m.age FROM Member m`
5. `DISTINCT`로 주복 제거: `SELECT DISTINCT ~ FROM ~`

**프로젝션 여러값 조회**
1. Query 타입으로 조회
2. Object[] 타입으로 조회
```java
List resultList = em.createQuery("select m.name, m.age from Member m")
                    .getResultList();

Object o = resultList;
Object[] result = (Object[]) o;
System.out.pringln("name = " + result.get(0));
System.out.println("age = " + result.get(1));
```
3. new 명령어로 조회 (DTO 사용)
```java
String jpql = "select new jpql.MemberDTO(m.name, m.age) from Member m"
List<MemberDTO> resultList = em.createQuery(jpql, MemberDTO.class);
```

## 3. 페이징
* JPA는 페이징을 다음 두 API로 추상화
* setFirstResult(int startPosition): 조회 시작 위치 (0부터 시작)
* setMaxResults(int maxResult): 조회할 데이터 수
```java
//페이징 쿼리
String jpql = "select m from Member m order by m.age desc";
List<Member> resultList = em.createQuery(jpql, Member.class)
      .setFirstResult(1)  // 조회할 페이지
      .setMaxResults(10)  // 페이지 내 데이터 수
      .getResultList();
```

## 4. 조인
* 내부 조인: `SELECT m FROM Member m INNER JOIN m.team t`
* 외부 조인: `SELECT m FROM Member m LEFT OUTER JOIN m.team t`
* 세타 조인: `SELECT count(m) FROM Member m, Team t WHERE m.name = t.name`
  * 연관관계가 없는 테이블 join

**조인 - ON 절** (JPA 2.1부터 지원)
1. 조인 대상 필터링
```
SELECT m FROM MEmber m LEFT JOIN m.team t ON t.name = 'a'
```
3. 연관관계 없는 엔티티 외부 조인 (하이버네이트 5.1부터) 
```
SELECT m, t FROM Member m LEFT JOIN Team t on m.name = t.name
```

## 5. 서브 쿼리
* 쿼리 내에서 또 다른 쿼리를 생성하는 것
* JPA는 `WHERE`, `HAVING` 절에서만 서브 쿼리 지원
* Hibernate에서는 `SELECT`절 서브 쿼리까지 지원
* FROM 절은 서브 쿼리 미지원 (JOIN 으로 풀어 해결하자)
  * Application에서 조립, 쿼리 2개로 풀어 사용 등

**jpql 서브 쿼리 예시**
1. 나이가 평균보다 많은 회원
```java
SELECT m FROM Member m
WHERE m.age > (SELECT avg(m2.age) FROM Member m2)
```
2. 한 건이라도 주문한 고객
```java
SELECT m FROM Member m
WHERE (SELECT count(o) FROM Order o WHERE m = o.member) > 0
```

**서브 쿼리 지원 함수**
1. [NOT] EXISTS (subquery): 서브 쿼리에 결과가 존재하면 참
```java
// 팀 A 소속인 회원 존재 여부
select m from Member m
where EXISTS (select t from m.team t where t.name = 'A')
```
2. ALL (subquery): 모두 만족하면 참
```java
// 전체 상품 각각의 재고보다 주문량이 많은 주문들
select o from Order o
where o.orderAmount > ALL (select p.stockAmount from Product p)
```
3. {ANY | SOME} (subquery): 조건을 하나라도 만족하면 참
```java
// 어떤 팀이든 팀에 소속된 회원
select m from Member m
where m.team = ANY (select t from Team t)
```
4. [NOT] IN (subquery): 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참

## 6. JPQL 타입 표현과 기타식
**JPQL 타입 표현**
* 문자(''안에 넣어주기): 'hello','She''s'
* 숫자: 10L(Long), 10D(Double), 10F(Float)
* Boolean: TRUE, FALSE
* ENUM: jpabook.MemberType.Admin (패키지명 명시)
  * 파라미터 바인딩을 사용하면 편리 
* 엔티티 타입: TYPE(m) = Member (상속 관계에서 사용)
  * `SELECT i FROM Item i WHERE type(i) = Book`

**JPQL 기타**
* SQL 문법과 거의 유사
* EXISTS, IN
* AND, OR, NOT
* =, >, >=, <, <=, <>
* BETWEEN, LIKE, IS NULL

## 7. 조건식 - CASE 식
1. 기본 CASE 식
```java
SELECT
  CASE WHEN m.age <= 10 then '학생요금'
       WHEN m.age >= 60 then '경로요금'
       ELSE '일반요금'
  END
FROM Member m
```
2. 단순 CASE 식
```java
SELECT
  CASE t.name
      WHEN '팀A' then '인센티브110%'
      WHEN '팀B' then '인센티브120%'
      ELSE '인센티브105%'
  END
FROM Team t
```
3. COALESCE: 하나씩 조회해서 null이 아니면 첫번째 값 반환, null이면 두번째 값 반환
```java
// 사용자 이름이 없으면 이름 없는 회원을 반환
SELECT COALESCE(m.name, '이름 없는 회원') FROM Member m
```
5. NULLIF: 두 값이 같으면 null 반환, 다르면 첫번째 값 반환
```java
// 사용자 이름이 ‘관리자’면 null을 반환하고 나머지는 본인의 이름을 반환
SELECT NULLIF(m.name, '관리자') FROM Member m
```

## 8. JPQL 기본 함수
* CONCAT: 문자열 더하는 함수
* SUBSTRING: 문자열 자르는 함수
* TRIM: 공백 제거
* LOWER, UPPER
* LENGTH: 문자열 길이
* LOCATE: 문자열에서 해당 문자의 위치 반환
* ABS, SQRT, MOD
* SIZE, INDEX(JPA 용도)
* `사용자 정의 함수`: 사용하는 DB의 Dialect를 상속받고 생성자에 registerFunction() 을 구현해 정의

## 9. 엔티티 직접 사용
JPQL에서 엔티티를 직접 사용하면 SQL에서 해당 엔티티의 기본 키 값을 사용한다.

1. 기본 키 값
```java
// 1. 엔티티를 파라미터로 전달
select m from Member m where m = :member

// 2. 식별자를 직접 전달
select m from Member m where m.id = :memberId
```
```sql
-- 실행된 SQL
SELECT m.* FROM member m where m.id=?
```

2. 외래 키 값
```java
1. 엔티티를 파라미터로 전달
select m from Member m where m = :member

// 2. 식별자를 직접 전달
select m from Member m where m.team.id = :teamId”
```
```sql
-- 실행된 SQL
SELECT m.* FROM member m WHERE m.team_id=?
```

## 10. Named 쿼리 - 정적 쿼리
* 미리 정의해서 이름을 부여해두고 사용하는 JPQL
* 정적 쿼리, 어노테이션, XML에 정의
* 애플리케이션 로딩 시점에 초기화 후 재사용
  * 애플리케이션 로딩 시점에 Named 쿼리를 SQL로 파싱해 캐시
* 애플리케이션 `로딩 시점에 쿼리를 검증`
  * Named 쿼리 오류 시 QuerySyntaxException 발생

```java
@Entity
@NamedQuery(
        name = "Member.findByName",
        query = "select m from Memberqq m where m.name = :name"
)
public class Member {
```
```java
List<Member> result = em.createNamedQuery("Member.findByName", Member.class)
                .setParameter("name", "회원1")
                .getResultList();
```

* Spring Data Jpa에서의 Named 쿼리
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    
    // Named 쿼리
    @Query("select m from Member m where m.name = :name")
    Member findByName(String name);
}
```















