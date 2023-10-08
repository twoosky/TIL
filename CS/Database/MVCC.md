# MVCC
* 일반적으로 레코드 레벨의 트랜잭션을 지원하는 DBMS가 제공하는 기능이다.
* MVCC의 가장 큰 목적은 잠금을 사용하지 않는 일관된 읽기를 제공하는 것이다. (InnoDB 언두 로그를 통해 제공)
* MVCC는 Multi Version Concurreny Control의 약자로, 하나의 레코드에 대해 여러 개의 버전이 동시에 관리된다.

## MVCC 예시
격리 수준이 READ_COMMITTED인 MySQL 서버에서 InnoDB 스토리지 엔진을 사용하는 테이블의 데이터 변경을 어떻게 처리하는지 확인해보자.
READ_COMMITED는 COMMIT된 데이터만 다른 트랜잭션에서 확인 가능하다.
```sql
START TRANSACTION;

CREATE TABLE member (
    m_id INT NOT NULL,
    m_name VARCHAR(20) NOT NULL,
    m_area VARCHAR(100) NOT NULL,
    PRIMARY KEY (m_id),
    INDEX ix_area (m_area)
);

INSERT INTO member (m_id, m_name, m_area) VALUES (1, '홍길동', '서울');

COMMIT;
```
INSERT 문이 실행되면 데이터베이스의 메모리(버퍼풀)와 디스크 각각에 데이터가 저장된다.

<img width="470" alt="image" src="https://github.com/twoosky/TIL/assets/50009240/05da0468-0152-420b-8e41-026e27f4274a">

그리고, 다음과 같이 UPDATE 문을 실행해 데이터를 갱신한다고 하자.
```sql
Transaction A> START TRANSACTION;
Transaction A> UPDATE member SET m_area="경기" WHERE id=1;
```

UPDATE 문이 실행되면 커밋 여부와 관계없이 일단 InnoDB 버퍼풀의 데이터는 갱신되고 언두 로그에는 변경 전의 데이터가 복사된다.  
디스크에도 수정한 값으로 갱신돼야 하는데, 이는 백그라운드 쓰레드에 의해서 처리가 되므로 시점에 따라 아직 갱신되지 않았을 수 있다. (MySQL은 ACID를 보장하므로 일반적으로 버퍼풀과 데이터 파일은 동일한 상태라고 가정해도 된다.)

<img width="480" alt="image" src="https://github.com/twoosky/TIL/assets/50009240/b13ae690-714c-4751-b55d-7170108c226b">

MVCC를 통해 동일한 레코드가 여러 버전으로 관리되고 있고, 이를 통해 커밋 이전의 데이터 읽기, 데이터 롤백 등이 가능한 것이다.
그렇다면, 트랜잭션이 COMMIT이나 ROLLBACK되지 않은 상태에서 다른 트랜잭션이 해당 레코드를 조회하면 어떻게 될까?
```sql
Transaction B> SELECT * FROM member WHERE id=1;
+------+-----------+--------+
| m_id | m_name    | m_area |
+------+-----------+--------+
|   12 | 홍길동      | 서울    |
+------+-----------+--------+
```

현재는 READ_COMMITTED 격리 수준이기 때문에 언두 영역에 있는 데이터를 읽어 m_area가 '서울'로 반환된다.

해당 결과는 격리 수준(Isolation level)에 따라 달라진다. 격리 수준이 READ_UNCOMMITED인 경우에는 InnoDB 버퍼 풀의 변경된 데이터를 읽어서 반환한다.
격리 수준이 READ_COMMITED, REPEATABLE_READ, SERIALIZABLE인 경우에는 아직 커밋되지 않았기 때문에 언두 영역의 변경 전 데이터를 반환한다.

만약 COMMIT을 하면 InnoDB는 더 이상의 변경 없이 지금의 상태를 영구적으로 만든다. 커밋이 된다고 언두 영역의 데이터가 항상 바로 삭제되지는 않는다. 언두 영역을 필요로 하는 트랜잭션이 더는 없을 때 비로소 삭제된다.
ROLLBACK을 하면 언두 영역에 백업된 데이터를 버퍼 풀로 다시 복구하고, 언두 영역의 내용은 삭제해버린다.
<br></br>
## 잠금 없는 일관된 읽기(Non-Locking Consistent Read)
* 동시성 제어를 위해 락을 사용하는 경우, 동시 요청 처리 속도가 매우 떨어진다.
* 그래서, InnoDB 스토리지 엔진은 MVCC 기술을 이용해 잠금을 걸지 않고 읽기 작업을 수행한다.
* 격리 수준 SERIALIZABLE을 제외한 모든 수준에서 INSERT와 연결되지 않은 순수한 읽기(SELECT) 작업은 다른 트랜잭션의 변경 작업과 관계없이 항상 잠금을 대기하지 않고 바로 실행된다.
* 특정 사용자가 레코드를 변경하고 아직 커밋을 수행하지 않았더라도, 다른 사용자의 SELECT 작업을 방해하지 않는다. (다른 사용자는 언두 로그의 데이터를 읽으므로)
* 이를 잠금 없는 일관된 읽기(Consistent Non-Locking Read)라고 표현하며, 이는 언두 로그를 통해 가능하다.

<img width="400" alt="image" src="https://github.com/twoosky/TIL/assets/50009240/9e05159f-8811-4f79-bc94-e8e096935d5a">

**주의점**
* 오랜 시간동안 활성 상태인 트랜잭션으로 인해 MySQL 서버가 느려지거나 문제가 발생할 수 있다.
* 일관된 읽기를 위해 해당 트랜잭션의 언두 로그를 삭제하지 못하고 유지해야되기 때문이다.
* 따라서, 트랜잭션의 범위를 최소화하여 꼭 필요한 곳에만 한정하는 것이 좋다.

## Reference
* RealMySQL 8.0
* https://mangkyu.tistory.com/288





