# Fetch Join
* JPQL에서 `성능 최적화`를 위해 제공하는 기능
* Fetch Join을 통해 연관된 엔티티나 컬렉션을 한 번에 함께 조회 (즉시 로딩)
* @ManyToOne 연관관계를 갖는 엔티티에서 사용
* Fetch Join은 연관된 엔티티들을 SQL 한번으로 조회하는 개념

```java
SELECT m FROM Member m JOIN FETCH m.team
```
```sql
-- 위 Fetch Join JPQL에 대한 실제 SQL
-- Member와 Team을 함께 조회
SELECT m.*, t.* FROM member m
INNER JOIN team t ON m.team_id=t.id
```
          
## 1. N+1 문제
* 연관 관계가 설정된 엔티티를 조회할 경우 조회된 데이터 개수(N)만큼 연관 엔티티 조회 쿼리가 추가로 발생하는 문제 
* `부모 조회(1) + 부모와 연관된 자식 조회(N)`

**N+1 문제 예시**
* 지연로딩에 의해 Member 조회 시 연관된 엔티티는 프록시 객체로 가져온다.
* `회원1`: 팀A 객체는 영속성 컨텍스트에 없으므로, SQL을 날려 가져온다.
* `회원2`: 팀A 객체가 영속성 컨텍스트에 존재하므로, 1차 캐시에서 가져온다.
* `회원3`: 팀B 객체는 영속성 컨텍스트에 없으므로, SQL을 날려 가져온다.
* 만약 회원 100명이 모두 다른 팀이라면? -> `N+1` 개의 쿼리가 실행될 것이다.
  * 1: 회원(Member)를 가져오기 위한 SELECT 쿼리
  * N: 각 회원에 대한 연관 엔티티(Team)를 가져오기 위한 SELECT 쿼리
```java
// (회원1 - 팀A), (회원2 - 팀A), (회원3 - 팀B)
String query = "select m from Member m";
List<Member> result = em.createQuery(query, Member.class)
                .getResultList();

for (Member member : result) {
    System.out.println("team class = " + member.getTeam().getClass());
    System.out.println("resultMember = " + member.getName() + ", " + member.getTeam().getName());
}
```
<details>
<summary>실행 결과</summary>
<div>

```sql
-- jpql 쿼리 실행 (Member 조회)
Hibernate: 
    /* select
        m 
    from
        Member m */ select
            member0_.id as id1_0_,
            member0_.age as age2_0_,
            member0_.name as name3_0_,
            member0_.team_id as team_id4_0_ 
        from
            Member member0_

-- 회원1에 대한 Team 조회
team class = class jpql.Team$HibernateProxy$GN2jo7dM
Hibernate: 
    select
        team0_.id as id1_3_0_,
        team0_.name as name2_3_0_ 
    from
        Team team0_ 
    where
        team0_.id=?

resultMember = 회원1, 팀A

-- 회원2에 대한 Team 조회
team class = class jpql.Team$HibernateProxy$GN2jo7dM
resultMember = 회원2, 팀A

-- 회원3에 대한 Team 조회
team class = class jpql.Team$HibernateProxy$GN2jo7dM
Hibernate: 
    select
        team0_.id as id1_3_0_,
        team0_.name as name2_3_0_ 
    from
        Team team0_ 
    where
        team0_.id=?
resultMember = 회원3, 팀B
```

</div>
</details>

**Fetch Join 사용해 해결**
* Fetch Join으로 부모 조회 시 연관된 자식 엔티티를 함께 가져온다. (즉시 로딩과 같은 효과)
* 프록시 객체가 아닌 실제 엔티티를 가져온다.
* SQL 쿼리 하나로 Member, Team 정보를 가져오므로 N+1 문제 해결 가능
```java
String query = "select m from Member m join fetch m.team";
// ... 아래는 위와 동일
```
<details>
<summary>실행 결과</summary>
<div>

```sql
Hibernate: 
    /* select
        m 
    from
        Member m 
    join
        fetch m.team */ select
            member0_.id as id1_0_0_,
            team1_.id as id1_3_1_,
            member0_.age as age2_0_0_,
            member0_.name as name3_0_0_,
            member0_.team_id as team_id4_0_0_,
            team1_.name as name2_3_1_ 
        from
            Member member0_ 
        inner join
            Team team1_ 
                on member0_.team_id=team1_.id

team class = class jpql.Team
resultMember = 회원1, 팀A
team class = class jpql.Team
resultMember = 회원2, 팀A
team class = class jpql.Team
resultMember = 회원3, 팀B
```

</div>
</details>

## 2. 컬렉션 Fetch Join
* @OneToMany 연관관계를 갖는 엔티티에서 사용
```java
SELECT t FROM Team t JOIN FETCH t.members
WHERE t.name = '팀A'
```
```sql
-- 위 JPQL에 대한 실제 SQL
SELECT t.*, m.*
FROM team t
INNER JOIN member m ON t.id=m.team_id
WHERE t.name='팀A'
```

**문제점**
* 일대다 Fetch Join은 두 테이블이 Join하는 과정에서 데이터가 뻥튀기 된다.
* Join에 의해 각 Team에 해당하는 Member 수 만큼 row가 늘어난다.
* 따라서 팀A에 해당하는 Member가 2개이므로, Join시 팀A에 대한 row가 2개가 되어 팀A가 2번 출력된다.
```java
String query = "select t From Team t join fetch t.members";
List<Team> result2 = em.createQuery(query, Team.class).getResultList();

for (Team team : result2) {
    System.out.println("team = " + team.getName() +" | members = " + team.getMembers().size());
}
```
```
team = 팀A | members = 2
team = 팀A | members = 2
team = 팀B | members = 1
```

<img src="https://user-images.githubusercontent.com/50009240/215422743-56e3708e-b264-4c62-97cd-052e767f17d2.png" width="600" height="250">


> `참고` 일대다 JOIN은 데이터가 뻥튀기 된다. 다대일에선 JOIN해도 데이터 뻥튀기 X

## 3. DISTINCT
* SQL의 DISTINCT는 중복된 결과를 제거하는 명령
* JPQL의 DISTINCT는 2가지 기능 제공
1. SQL에 DISTINCT 추가
2. 애플리케이션에서 엔티티 중복 제거 

**예시**
* SQL에 DISTINCT를 추가하지만 데이터가 다르므로 SQL 결과에서 중복제거 실패
* 애플리케이션에서 같은 식별자를 가진 Team 엔티티를 제거한다.
```java
SELECT DISTINCT t FROM Team t JOIN FETCH t.members
```
```
team = 팀A | members = 2
team = 팀B | members = 1
```

## 4. Fetch Join과 일반 Join 차이
**1. 일반 Join**: 연관된 엔티티를 함께 조회하지 않는다. 
* select 절에서 Team만 가져온다.
* member는 프록시 객체로 가져와 실제 사용할 때 초기화 된다. 
```java
SELECT t FROM Team t JOIN t.members m
```
<details>
<summary>실행 결과</summary>
<div>

```sql
Hibernate: 
    /* select
        t 
    From
        Team t 
    join
        t.members */ select
            team0_.id as id1_3_,
            team0_.name as name2_3_ 
        from
            Team team0_ 
        inner join
            Member members1_ 
                on team0_.id=members1_.team_id
```

</div>
</details>

**2. Fetch Join**: 연관된 엔티티를 함께 조회한다.
* select 절에서 Team, Member를 모두 가져온다. 
```java
SELECT t FROM Team t JOIN FATCH t.members
```
<details>
<summary>실행 결과</summary>
<div>

```sql
Hibernate: 
    /* select
        t 
    From
        Team t 
    join
        fetch t.members */ select
            team0_.id as id1_3_0_,
            members1_.id as id1_0_1_,
            team0_.name as name2_3_0_,
            members1_.age as age2_0_1_,
            members1_.name as name3_0_1_,
            members1_.team_id as team_id4_0_1_,
            members1_.team_id as team_id4_0_0__,
            members1_.id as id1_0_0__ 
        from
            Team team0_ 
        inner join
            Member members1_ 
                on team0_.id=members1_.team_id
```

</div>
</details>

## 5. Fetch Join 한계
1. Fetch 조인 대상에는 별칭을 줄 수 없다.
2. 둘 이상의 Collection을 Fetch Join 할 수 없다.
    * `org.hibernate.loader.MultipleBagFetchException` 발생
    * 일대다 관계에서는 컬렉션의 카테시안 곱이 만들어지므로, 너무 많은 row가 생성되어 에러 발생
    * `ToMany` 관계에서는 컬렉션 하나만 Fetch Join 가능
    * `ToOne` 관계에서는 컬렉션 여러개 Fetch Join 가능
3. 컬렉션을 페치 조인하면 페이징 API를 사용할 수 없다.
    * `ToOne` 관계에서는 Fetch Join해도 페이징 가능 (Join해도 row 수가 늘어나지 않으므로)
    * `ToMany` 관계에서는 Join 대상에 따라 Row 수가 늘어나므로, 페이징이 중간에 짤릴 수 있다.
      * Hibernate는 경로 로그를 남기고 메모리에서 페이징 (매우 위험)
      * SQL의 limit문을 사용해 페이징하는 것이 아닌, 테이블을 FULL SCAN해 애플리케이션 메모리에 올려 페이징

## 6. 일대다 FetchJoin 페이징
**1. 쿼리의 방향을 `@OneToMany`가 아닌, `@ManyToOne` 방향으로 바꾸자.**
```java
// Before
SELECT t FROM Team t JOIN FETCH t.members

// After
SELECT m FROM Member m JOIN FETCH t.team
```
**2. @BatchSize 사용**
* `BatchSize`는 여러 개의 프록시 객체를 조회할 때 WHERE 조건절을 하나의 `IN` 절로 바꾸어 쿼리 실행
* size 속성은 IN 절에 들어갈 요소의 최대 개수
* 만약 조회할 프록시 객체가 설정한 배치사이즈 크기보다 더 많다면 `IN` 쿼리가 추가로 날아간다.
  * size는 100으로 설정, 조회할 프록시 객체가 150개라면 처음에 100개, 다음에 50개로 나눠 IN 쿼리를 날린다.
* BatchSize 사용하면 쿼리 수를 N+1이 아니라 `Row수/size`로 맞출 수 있다.
<br></br>
* Team을 가져올 때 Member는 Lazy 로딩 상태이다. (프록시 객체)
* Team 객체가 getMembers()를 호출할 때 result에 담긴 각 Team의 member를 한 번에 IN 쿼리로 size만큼 가져온다. 
```java
@Entity
public class Team {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String name;

    @BatchSize(size = 100)
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
}
```
```java
String query = "select t From Team t";
List<Team> result = em.createQuery(query, Team.class)
          .setFirstResult(0)
          .setMaxResults(2)
          .getResultList();

for (Team team : result) {
    System.out.println("team = " + team.getName() +" | members = " + team.getMembers().size());
}
```
<details>
<summary>실행 결과 비교</summary>
<div>

```sql
-- BatchSize 사용 전
Hibernate: 
    /* select
        t 
    From
        Team t */ select
            team0_.id as id1_3_,
            team0_.name as name2_3_ 
        from
            Team team0_ limit ?
---
Hibernate: 
    select
        members0_.team_id as team_id4_0_0_,
        members0_.id as id1_0_0_,
        members0_.id as id1_0_1_,
        members0_.age as age2_0_1_,
        members0_.name as name3_0_1_,
        members0_.team_id as team_id4_0_1_ 
    from
        Member members0_ 
    where
        members0_.team_id=?
team = 팀A | members = 2

Hibernate: 
    select
        members0_.team_id as team_id4_0_0_,
        members0_.id as id1_0_0_,
        members0_.id as id1_0_1_,
        members0_.age as age2_0_1_,
        members0_.name as name3_0_1_,
        members0_.team_id as team_id4_0_1_ 
    from
        Member members0_ 
    where
        members0_.team_id=?
team = 팀B | members = 1
-- ...
```
```sql
-- BatchSize 사용 후
Hibernate: 
    /* select
        t 
    From
        Team t */ select
            team0_.id as id1_3_,
            team0_.name as name2_3_ 
        from
            Team team0_ limit ?
---        
Hibernate: 
    /* load one-to-many jpql.Team.members */ select
        members0_.team_id as team_id4_0_1_,
        members0_.id as id1_0_1_,
        members0_.id as id1_0_0_,
        members0_.age as age2_0_0_,
        members0_.name as name3_0_0_,
        members0_.team_id as team_id4_0_0_ 
    from
        Member members0_ 
    where
        members0_.team_id in (
            ?, ?
        )

team = 팀A | members = 2
team = 팀B | members = 1
```
* WHERE 조건절을 하나의 IN절로 바꾸어 쿼리를 실행한다.


</div>
</details>

> `TIP` 글로벌 로딩 전략은 모두 지연 로딩, 최적화가 필요한 곳은 Fetch Join을 적용해라















