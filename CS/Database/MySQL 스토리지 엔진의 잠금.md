# MySQL 스토리지 엔진의 잠금
* MySQL(InnoDB)에서 사용되는 락은 크게 스토리지 엔진 레벨과 MySQL 엔진 레벨로 나눌 수 있다.
* `스토리지 엔진 레벨`의 잠금은 레코드 기반의 잠금 기능 제공
* `MySQL 엔진 레벨`의 잠금은 테이블이나 데이터베이스 기반의 잠금 기능 제공
* 이번에는 MySQL 스토리지 엔진 레벨의 잠금에 대해 알아보자.
  * 레코드 락(Record Lock)
  * 갭 락(Gap Lock)
  * 넥스트 키 락(Next Key Lock)
  * 자동 증가 락(Auto Increment Lock)

## 레코드 락 (Recode Lock)
* 일반적으로 레코드 락은 테이블 레코드 자체를 잠그는 락을 의미한다.
* 다른 DBMS와 다르게 MySQL(InnoDB)에서의 레코드 락은 레코드가 아닌, 인덱스의 레코드를 잠근다.
* 인덱스가 하나도 없는 테이블이더라도 내부적으로 자동 생성된 클러스터 인덱스를 이용해 잠금을 설정한다.

**예시**
* employees 테이블에서 이름(first_name)이 Georgi인 사원은 전체 253명이 있다.
* 이름(first_name)이 Georgi이고, 성(last_name)이 Klassen인 사원은 1명만 존재한다.
* employees 테이블에는 이름(first_name) 컬럼만으로 구성된 인덱스 KEY ix_firstname (first_name)가 존재한다.
```sql
-- 인덱스 생성
CREATE INDEX ix_firstname ON employees (first_name);
```
```sql
-- employees 테이블에서 first_name이 'Georgi'인 구성원은 235명이다.
SELECT COUNT(*) FROM employees WHERE first_name='Georgi';

-- 그 중에서 last_name이 Klassen인 사원은 1명만 있다.
SELECT COUNT(*) FROM employees WHERE first_name='Georgi' AND last_name='Klassen';
```
* 이름(first_name)에만 인덱스가 걸려있는 상태에서 이름이 Georgi이고, 성이 Klassen인 사원의 입사 일자를 변경하는 UPDATE 쿼리를 실행한다고 하자.
```sql
UPDATE employees SET hire_date=NOW() WHERE first_name='Georgi' AND last_name='Klassen';
```
* UPDATE 문에 의해 영향받는 레코드는 1건이다. 하지만 1건을 업데이트하기 위해 235건의 인덱스 레코드에 잠금이 걸린다.
* 왜냐하면, MySQL은 테이블 레코드가 아닌 인덱스 레코드에 잠금을 걸기 때문이다. MySQL은 **인덱스를 통해 검색되는 모든 레코드에 잠금**을 걸게 된다.
* employees 테이블의 인덱스는 이름(first_name)으로만 구성되어 있기 때문에, first_name이 Georgi인 인덱스 레코드 235건에 모두 잠금을 걸게 된다.
* 만약 인덱스가 성(last_name)으로 구성되어 있다면 1건의 인덱스 레코드에만 잠금이 걸릴것이다. (따라서, 카디널리티가 높은 컬럼으로 인덱스를 구성하는 것이 유리)

**테이블에 적당한 인덱스가 없다면?**
* 적당한 인덱스가 없다는 것은, 검색의 대상이 되는 컬럼이 인덱스로 구성되어 있지 않는 경우이다.
* 이러한 경우에는 테이블을 풀 스캔하면서 UPDATE 작업을 하기 때문에, 테이블의 모든 레코드에 락을 걸게 된다. (동시성이 낮아짐)
* 따라서, MySQL에서는 인덱스 설계가 매우 중요하다.

## 갭 락 (Gap Lock)
* MySQL(InnoDB)은 다른 DBMS와 다르게 갭 락을 제공한다.
* 갭 락(Gap Lock)은 레코드 자체가 아니라, 레코드와 바로 인접한 레코드 사이의 간격만을 잠그는 것을 의미한다.
* 갭 락의 역할은 레코드와 레코드 사이의 간격에 새로운 레코드가 생성(INSERT)되는 것을 제어하는 것이다.
* 다른 트랜잭션에서 임의의 데이터가 추가되지 않도록 잠그려면 아래와 같은 쿼리를 실행해야 한다.
* 여기서 SELECT … FOR UPDATE 구문은 베타적 잠금(비관적 잠금, 쓰기 잠금)을 거는 것이다.

**예시**
* 갭 락은 인덱스 범위 조건 중에서 실제 레코드를 제외하고, 데이터가 추가될 수 있는 범위에 걸리게 된다.
* 인덱스는 정렬된 순서로 존재하므로, 현존하는 인덱스 레코드의 앞 뒤에 갭 락이 걸린다.
* 즉, 아래와 같이 갭 락을 건 경우 이름(first_name)이 Georgi인 데이터는 삽입할 수 없게된다.
```sql
-- 이름(first_name)이 Georgi인 인덱스 레코드 앞 뒤 레코드에 쓰기 잠금
SELECT * FROM employees WHERE first_name LIKE='Georgi' FOR UPDATE;
```

**예시2**
* number 테이블에는 num 컬럼만으로 구성된 인덱스 KEY ix_num (num)가 존재한다.
* number 테이블에는 num이 2,3인 데이터만 존재한다.
```sql
mysql> SELECT * FROM number;
+----+------+
| id | num  |
+----+------+
|  1 |    2 |
|  2 |    3 |
+----+------+
```
* 아래와 같이 Transaction A에서 num이 1 이상 5 이하인 컬럼에 쓰기 잠금을 걸어보자.
* 갭 락에 의해 현존하는 레코드(2, 3)을 제외한 num값이 1, 4, 5가 될 수 있는 인덱스 레코드에 잠금이 걸릴 것이다.
* 실존하는 레코드(2, 3)에 걸리는 락이 레코드 락, 범위 내 실존하지 않는 인덱스 레코드에 걸리는 락이 갭 락
* 실제로 잠금이 걸리는지 확인해보기 위해, Transaction B에서 num이 4인 데이터를 삽입해보자.
```sql
-- 쓰기 잠금
Transaction A> set autocommit=false;
Transaction A> SELECT * FROM number WHERE 1 <= num and num <=5 FOR UPDATE;
```
* 갭 락에 의해 num이 4인 데이터를 삽입할 수 없어 timeout 에러가 발생한다.
* 데이터를 삽입하려면 갭 락(쓰기 잠금)을 건 트랜잭션이 COMMIT/ROLLBACK 할 때까지 기다려야 한다.
```sql
-- 데이터 삽입
Transaction B> INSERT number VALUES(3, 4);
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
```

갭 락은 아직 존재하지는 않지만 지정된 범위에 해당하는 인덱스 테이블 공간을 대상으로 거는 잠금이다. 따라서 데이터의 유일성이 보장되는 프라이머리 키(PK) 또는 유니크 인덱스에 의한 작업에서는 갭 락이 사용되지 않는다. 그리고 이러한 갭 락은 뒤에서 살펴볼 Pantom Read(유령 읽기)를 방지하는데 도움이 된다.

## 넥스트 키 락 (Next Key Lock)
* 레코드 락과 갭 락을 합쳐 놓은 형태의 잠금을 넥스트 키 락(Next key lock)이라고 한다.
* 즉, 실존하는 인덱스 레코드와 레코드 사이 범위에 대해 모두 락을 거는 것을 의미한다.
* 위 예시의 경우 넥스트 락에 의해 num이 1,2,3,4,5인 인덱스 레코드에 모두 락이 걸리게 된다.

**주의점**
* InnoDB의 갭 락이나 넥스트 키 락은 바이너리 로그에 기록되는 쿼리가 레플리카 서버에서 실행될 때 소스 서버에서 만들어 낸 결과 와 동일한 결과를 만들어내도록 보장하는 것이 주목적이다.
* 하지만 넥스트 키 락과 갭 락으로 인해 데드락이 발생하거나 다른 트랜잭션을 기다리게 만드는 일이 자주 발생한다.
* 가능하다면 바이너리 로그 포맷을 ROW 형태로 바꿔서 넥스트 키 락이나 갭 락을 줄이는 것이 좋다.

## 자동 증가 락 (Auto Increment Lock) 
* MySQL에서는 자동 증가하는 숫자값을 추출하기 위해 AUTO_INCREMENT라는 컬럼 속성을 제공한다.
* AUTO_INCREMENT 컬럼이 사용된 테이블에 동시에 여러 레코드가 INSERT되는 경우, 저장되는 각 레코드는 중복되지 않고 저장된 순서대로 증가하는 숫자값을 가져야 한다.
* 따라서, InnoDB 스토리지 엔진에서는 이를 위해 AUTO_INCREMENT 락 이라는 테이블 수준의 잠금을 사용한다.
* 해당 락은 INSERT와 REPLACE와 같이 새로운 레코드를 저장하는 쿼리에서만 사용된다.
* 또한, 자동 증가 락은 잠금을 최소화하기 위해 한 번 증가하면 절대 자동으로 줄어들지 않는다. 트랜잭션이 롤백되더라도 증가값은 복구되지 않는다.
* 만약 해당 값을 초기화하려면 아래의 쿼리를 사용해야 한다.
```sql
ALTER TABLE tablename AUTO_INCREMENT = 1
```

### Reference
* RealMySQL 8.0
* https://mangkyu.tistory.com/298
* https://developer-nyong.tistory.com/55






