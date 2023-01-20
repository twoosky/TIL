# JPA란
## 1. JPA란
* JPA(Java Persistence API)는 자바 진영의 `ORM` 기술 표준이다.
* ORM(Object-relational mapping)
  * 객체는 객체대로 설계, 관계형 데이터베이스는 관계형 데이터베이스대로 설계
  *  ORM 프레임워크가 중간에서 매핑
* JPA 동작: JPA는 애플리케이션과 JDBC 사이에서 동작한다.  
  1. JAVA 애플리케이션에서 JPA에게 명령   
  2. JPA가 SQL을 만들어 JDBC API를 사용해 DB에 전송    
  3. DB로부터 반환된 결과를 JAVA 애플리케이션에서 사용
  <img src="https://user-images.githubusercontent.com/50009240/197345390-f7c3eeb3-6f49-4e88-a12c-55e40f19ec20.png" width="500" height="200">
* JPA 사용 이유
  * 생산성 증가, SQL 중심적인 개발에서 객체 중심으로 개발
  * 패러다임의 불일치 해결
## 2. JPA 문법
* EntityManagerFactory는 하나만 생성해 애플리케이션 전체에서 공유
* 트랜잭션 단위로 기능을 만들때 항상 EntityManagerFactory를 통해 EntityManager를 새로 만들어야 한다.
* EntityManager는 쓰레드간에 공유하지 않는다. (사용하고 버려야 한다.)
* **JPA의 모든 데이터 변경은 트랜잭션 안에서 실행**
```java
public static void main(String[] args) {
     EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

     EntityManager em = emf.createEntityManager();

     EntityTransaction tx = em.getTransaction();
     tx.begin();

     try {
        /*
           Entity 선언 및 변경 ...
        */
         tx.commit();
     } catch (Exception e) {
         tx.rollback();
     } finally {
         em.close();  // EntityManager는 내부적으로  DB Connection을 갖고있으므로 꼭 닫아줘야한다.
     }
     emf.close();
 }
 ```
* JPQL 특징
  * 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리
  * SQL을 추상화해서 특정 데이터베이스 SQL에 의존적이지 않다.
   
   
   
   
   
   
   
   
   
   
   
   
   
