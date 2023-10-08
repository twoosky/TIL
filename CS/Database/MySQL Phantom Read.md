# MySQL Phantom Read

## REPEATABLE READ
* MySQL의 기본 격리 수준인 REPEATABLE READ은 MVCC를 통해 언두 영역에 백업된 이전 데이터를 이용해 동일 트랜잭션 내에서는 동일한 결과를 보여줄 수 있게 보장한다.
* 각각의 트랜잭션은 순차 증가하는 고유한 트랜잭션 번호가 존재하며, 언두 영역에는 어느 트랜잭션에 의해 저장/수정/삭제되었는지 트랜잭션 번호를 함께 저장한다. 
* REPEATABLE READ는 한 트랜잭션 내에서 동일한 결과를 보장하지만, 새로운 레코드가 추가되는 경우에 부정합이 생길 수 있다.

**예시**  

0. employees에 emp_no=500000, first_name=Lara인 레코드가 존재한다 가정.
1. 사용자 B가 emp_no=500000인 레코드 조회, 트랜잭션은 아직 종료되지 않은 상태이다.
2. 사용자 A가 emp_no=500000인 레코드 갱신, MVCC를 통해 InnoDB 버퍼풀 데이터는 변경되지만, 변경 전 데이터가 언두 로그에 저장된다.
4. 사용자 A COMMIT
5. 사용자 B가 emp_no=500000인 레코드 다시 조회, 언두 로그의 변경 전 데이터를 반환 

<img width="500" alt="image" src="https://github.com/twoosky/TIL/assets/50009240/3b83ff44-4719-44c7-be5b-2f9d4e8770f5">

## REPEATABLE READ Phantom Read
* REPEATABLE READ는 새로운 레코드의 추가까지는 막지 않는다.
* 따라서, 다른 트랜잭션에서 수행한 삽입 작업에 의해 레코드가 보였다 안 보였다 하는 현상을 유령 읽기(PHANTOM READ)라고 한다.
* MVCC에 의해 잠금 없는 SELECT 조회에서는 Phantom Read 현상이 발생하지 않는다. (자신보다 나중에 실행된 트랜잭션의 레코드는 무시하고, 일관된 읽기 가능)
* 하지만, SELECT FOR UPDATE를 통해 쓰기 잠금을 건 경우 Phantom Read가 발생할 수 있다.
> SELECT FOR UPDATE 쿼리는 SELECT 하는 레코드에 쓰기 잠금을 걸어야하는데, 언두 레코드에는 잠금을 걸수없다.
> 그래서 SELECT FOR UPDATE로 조회되는 레코드는 언두 영역의 변경 전 데이터를 가져오는 것이 아니라 실제 테이블의 레코드의 값을 가져오게 된다.

**예시**

0. employees에 emp_no>=500000인 레코드는 1개 존재
1. 사용자 B가 emp_no>=500000인 레코드 조회 (쓰기 잠금), 결과 1개 반환
2. 사용자 A가 emp_no=500001인 레코드 삽입
3. 사용자 A COMMIT
4. 사용자 B emp_no>=500000인 레코드 조회 (쓰기 잠금), 결과 2개 반환 (Phantom Read)

<img width="500" alt="image" src="https://github.com/twoosky/TIL/assets/50009240/a050240b-2c29-4fd2-aef8-e3d2136bbfaf">

## MySQL REPEATABLE READ Phantom Read
* MySQL은 다른 DBMS와 다르게 갭 락이 존재하기 때문에 일반적으로 REAPEATABLE READ에서 Phantom Read가 발생하지 않는다.

**위와 같은 상황에서 MySQL은 Phantom Read가 발생하지 않는다.**
* 사용자 B가 SELECT FOR UPDATE로 조회한 경우 MySQL은 emp_no가 500000인 레코드에는 레코드 락, id가 500000보다 큰 범위에는 갭 락으로 넥스트 키 락을 건다.
* 그 후, 사용자 A가 id가 500001인 레코드 INSERT를 시도한다면, 쓰기 잠금에 의해 사용자 B가 커밋/롤백될 때까지 기다린다.
* 사용자 A가 대기를 지나치게 오래하면 타임아웃이 발생하게 된다.

<img width="500" alt="image" src="https://github.com/twoosky/TIL/assets/50009240/7ac485be-d849-4aeb-bd00-f61c52360a3d">


**하지만, 아래와 같은 경우 MySQL에서 Phantom Read가 발생할 수 있다.**
* 사용자 B는 트랜잭션을 시작하고, 잠금없는 SELECT 문으로 데이터를 조회했다.
* 사용자 A는 INSERT 문을 사용해 데이터 추가 후 COMMIT을 했다. (쓰기 잠금 없으므로 커밋 가능)
* 그 후, 사용자 B가 SELECT FOR UPDATE으로 조회했다면, 언두 로그가 아닌 테이블로부터 레코드를 조회하므로 Phantom Read가 발생한다.

<img width="520" alt="스크린샷 2023-10-09 오전 5 10 45" src="https://github.com/twoosky/TIL/assets/50009240/9fde12fd-ea61-4ace-902e-0dbc5c117d33">

## MySQL에서의 Phantom Read 정리
|Before|After|Phantom Read|
|---|---|---|
|SELECT FOR UPDATE|SELECT| 갭락 때문에 팬텀리드 X|
|SELECT FOR UPDATE|SELECT FOR UPDATE| 갭락 때문에 팬텀리드 X|
|SELECT|SELECT| MVCC 때문에 팬텀리드 X|
|SELECT|SELECT FOR UPDATE| 팬텀 리드 O|
