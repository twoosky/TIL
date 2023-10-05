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
* 현존하는 레코드(2, 3)에 걸리는 락이 레코드 락, 범위 내 실존하지 않는 인덱스 레코드에 걸리는 락이 갭 락
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

## 넥스트 락 (Next Lock)



























