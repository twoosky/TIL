# @MappedSuperclass
* 공통 매핑 정보가 필요할 때 사용 (공통 column이 필요한 경우)
* `@MappedSuperclass`가 붙은 부모 클래스는 테이블로 생성하지 않고, 자식 클래스에게 매핑 정보만 제공
* @Entity는 실제 테이블과 매핑되지만, @MappedSuperclass는 실제 테이블과 매핑되지 않는다.
* 상속관계 매핑과 다른 개념이다. (부모 타입으로 조회, 검색 불가)
<br></br>
```java
@MappedSuperclass
public abstract class BaseEntity {

    private String createdBy;
    private LocalDateTime createdDate;
    private String lastModifiedBy;
    private LocalDateTime lastModifiedDate;
}
```
```java
@Entity
public class Member extends BaseEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
}
```
* 상속받은 @MappedSuperclass 객체의 매핑 정보를 포함해 자식객체 테이블이 생성된다.
```
Hibernate: 
    create table member (
       id bigint not null auto_increment,
        createdBy varchar(255),
        createdDate datetime(6),
        lastModifiedBy varchar(255),
        lastModifiedDate datetime(6),
        name varchar(255),
        primary key (id)
    ) engine=InnoDB
```

> `참고`: @Entity 클래스는 엔티티나 @MappedSuperclass로 지정한 클래스만 상속 가능
