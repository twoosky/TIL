# Index

## Index란
`Index`는 테이블에 대한 동작 속도를 높여주는 자료구조이다.    
`Index`는 해당 table의 컬럼을 색인화(따로 파일로 저장) 하여 검색시 해당 table의 레코드를 full scan이 아닌 색인화 되어있는 index 파일을 검색함으로써 검색속도를 빠르게한다.


> DB Index는 흔히 **책의 목차**에 비유된다. 책에 목차가 없다면, 특정 내용을 찾기 위해 책의 첫 페이지부터 시작해서, 원하는 내용이 나올 때까지 모든 페이지를 일일이 찾아 읽을 것이다.
> 책의 분량이 많거나 찾고자 하는 내용이 책의 후반부에 위치한다면, 검색에 오랜 시간이 걸릴 것이다.  
> 이때 책에 목차가 존재한다면, 책의 분량이 많더라도 특정 내용을 빠르게 찾을 수 있다. 이처럼 목차는 검색에 소요되는 시간을 비약적으로 줄여준다.

<br>
<img src="https://miro.medium.com/max/1050/1*sz3PldJHpc_cCTbk1YR7LA.png" width="700" height="200">

위의 예에서 테이블의 레코드는 id를 기준으로 정렬되어 있다. 위 테이블에서 특정 name을 가지는 레코드를 검색하려면 테이블의 첫번째 레코드부터 해당 레코드까지 반복해야 한다.
레코드 개수가 n개라 했을 때, 이 경우 시간 복잡도는 **O(n)** 이 된다.  



이때 name이 알파벳 순으로 정렬되고, 각 레코드들이 실제 table의 레코드를 가리키는 `Index`를 갖는다 해보자.
이 index를 사용하여 특정 name을 검색하면, 레코드들이 name을 기준으로 정렬되어 있기 때문에 *이진 탐색*이 가능해지므로 시간 복잡도는 **O(log n)** 이 된다.

## Index SQL
**INDEX 생성**  
```sql
CREATE INDEX [인덱스 명] ON [테이블 명]([컬럼1, 컬럼2, ..]);

CREATE INDEX user_id_index ON user(user_id);
```

**UNIQUE INDEX 생성**  
* Unique index는 중복 값을 허용하지 않는 인덱스이다.  
* 대상이 되는 컬럼에 중복된 값이 포함되어 있으면 Unique index는 만들 수 없다.  
* 또한, Unique index를 생성한 후 Unique index의 대상인 컬럼에 이미 존재하는 값은 테이블에 추가할 수 없다.
```sql
CREATE UNIQUE INDEX [인덱스 명] ON [테이블 명]([컬럼1, 컬럼2, ..]);

CREATE UNIQUE INDEX user_name_index ON user(name);
```

**DROP INDEX**  
* Drop Index 절은 테이블의 index를 제거할 때 사용한다.  
* 위 구문은 MySQL의 경우 사용할 수 있는 구문이다.
```sql
ALTER TABLE [테이블 명] DROP INDEX [인덱스 명];
```

## Unique Index 예시
아래와 같은 테이블을 생성한 뒤 아래의 값을 삽입했다 하자.
```sql
CREATE TABLE user(
    user_id INT(11) NOT NULL auto_increment,
    name VARCHAR(50) NOT NULL,
    address VARCHAR(100) NOT NULL,
    PRIMARY KEY('user_id'),
)

insert into user values ('devkuma', 'Seoul');
insert into user values ('kimkc', 'Busan');
insert into user values ('araikuma', 'Seoul');
```
> 1. name 컬럼을 대상으로 Unique index 생성  
> 결과: Unique index 생성 성공
```sql
create unique index user_name_index on user(name);
```
> 2. Unique index의 대상 컬럼(name)에 이미 존재하는 값(devkuma)을 테이블에 삽입  
> 결과: Error 발생
```sql
insert into user values ('devkuma', 'Busan');
Error: UNIQUE constraint failed: user.name
```
> 3. 중복된 값(seoul)이 존재하는 address 컬럼을 대상으로 Unique index 생성  
> 결과: Error 발생
```sql
create unique index user_address_index on user(address);
Error: UNIQUE constraint failed: user.address
```

## 성능 테스트


## Index 자료 구조
DB Index에 적합한 자료 구조로는 크게 `Hash Table`, `B-Tree`, `B+Tree` 등이 있다.  

## 1. Hash Table

<img src="https://user-images.githubusercontent.com/56240505/133892261-f95fd474-7955-4f8f-81f8-457d71ac6324.png" width="500" height="350">

`Hash Table`은 key-Value로 이루어진 데이터를 저장하는데 특화된 자료 구조이다.  
해시 테이블 기반의 DB Index는 특정 컬럼의 값과 실제 데이터의 위치를 Key-Value로 사용한다.

> Hash Table index 원리  
> 
해시 테이블은 내부에 **Bucket(버켓)** 이라는 배열이 존재한다. 해시 함수를 통해 Key를 고유한 해시 값으로 변환시키는데, 이를 버켓 배열의 인덱스로 사용하며 해당 인덱스에 Value(실제 데이터의 위치)를 저장한다. 위 그림에서 *John Smith*에 대한 레코드를 찾는다고 하자.
1. 찾는 레코드의 값(John Smith)을 해시 함수에 넣는다.
2. 해시 함수의 결과로 버켓 인덱스(02)를 얻는다.
3. 해당 버켓(02) 인덱스에서 원하는 레코드(John Smith)의 실제 데이터 위치(주소)를 찾는다.

> Hash Table index 시간 복잡도  
   
해시함수를 통해 Key값으로 Value가 저장되어 있는 버켓 인덱스를 바로 산출할 수 있기 때문에, 해시 테이블의 평균적인 시간 복잡도는 **O(1)** 이다.  
하지만 해시 함수를 제대로 정의하지 않으면 해시 함수를 통해 산출한 해시 값이 중복되는 **해시 충돌** 이 발생할 수 있다. 너무 많은 해시 충돌이 발생하면 검색 성능이 하락해 시간 복잡도가 **O(N)** 에 수렴할 수 있다.

> Hash Table index 한계     

Index 자료 구조로 해시 테이블을 사용하는 경우는 매우 제한적이다.   
해시 함수는 Key가 조금이라도 다르면 완전히 다른 해시 값을 생성한다. 이러한 해시 테이블을 사용하는 Index의 경우 WHERE 조건의 등호(=) 연산에는 효율이 좋지만, 부등호 연산(>, <)은 부적합하다. 해시 테이블은 내부 데이터들이 정렬되어 있지 않아 범위(>, <) 조건 질의 시 탐색이 효율적이지 않다.

## 2. B-Tree
<img src="https://user-images.githubusercontent.com/56240505/133893917-705097ca-4f0b-4f6a-8540-435988282643.png" width="500" height="170">

<img src="https://user-images.githubusercontent.com/56240505/133895971-588e05fd-a013-4a39-9e89-86efd1b4d5c5.png" width="500" height="180">

`B-Tree`란 자식 노드가 2개 이상인 트리를 의미한다. 이진검색 트리처럼 각 Key의 왼쪽 자식은 항상 Key보다 작은 값을, 오른쪽 자식은 큰 값을 갖는다.  
B-Tree 기반의 DB Index는 특정 컬럼의 값(Key)에 해당하는 노드의 데이터 위치(Value)를 저장한다.  
B-Tree 자료 구조의 내부 동작 원리는 [자료구조-그림으로 알아보는 B-Tree](https://velog.io/@emplam27/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-%EA%B7%B8%EB%A6%BC%EC%9C%BC%EB%A1%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EB%8A%94-B-Tree) 참조

> B-Tree 시간 복잡도

B-Tree의 Key-Value 값들은 항상 Key를 기준으로 오름차순 정렬이다. 이로 인해 부등호 연산(>, <)에 대해 해시 테이블보다 효율적인 데이터 탐색이 가능하다. 또한 B-Tree는 균형 트리(Balanced Tree)로서, 최상위 루트 노드에서 리프 노드까지의 거리가 모두 동일하기 때문에 평균 시간 복잡도는 **O(logN)** 이다.

> B-Tree 한계

* B-Tree 기반의 DB Index는 Index가 적용된 테이블에 데이터 갱신(INSERT, UPDATE, DELETE)가 반복되다보면, 트리의 균형이 깨지면서 성능이 악화된다.  
* DB Index 컬럼은 부등호(>,<)를 이용한 순차 검색 연산이 자주 발생한다. B-Tree가 해시 테이블보다 부등호를 이용한 검색 연산 성능이 좋지만, 순차 검색의 경우 중위 순회를 하기 때문에 효율이 좋지 않다.
* 이러한 이유로 MySQL 엔진인 InnoDB는 B-Tree를 확장 및 개선한 B+Tree를 Index의 자료 구조로 사용한다.

## 3. B+Tree
<img src="https://velog.velcdn.com/images%2Fsjy5386%2Fpost%2Fadb3e389-f636-40a7-95f7-317b8a4e3267%2Fb--search.jpg" width="600" height="250">

`B+Tree`는 B-Tree를 확장 및 개선한 자료 구조로서, 말단의 리프 노드에만 데이터의 위치(Value)를 관리한다. 중간 브랜치 노드에 Value가 없어서 B-Tree보다 메모리를 덜 차지하는 만큼, 노드의 메모리에 더 많은 Key를 저장할 수 있다.  
또한 하나의 노드에 더 많은 Key를 저장할 수 있기에 트리의 높이가 더 낮아져 성능을 향상시킬 수 있다.

> B+Tree Index 구성

* 실제 데이터가 저장된 레코드에 대한 주소가 기록된 단말 노드(leaf node)
* 단말 노드를 찾아가기 위한 검색키와 하위 노드의 주소로 구성된 중간 노드(internal node)
* 경로의 출발점이 되는 루트 노드(root node)

> B+Tree Index 과정

위 그림에서 9번에 대한 레코드를 찾는다고 하자.
1. 찾는 레코드(9)와 루트 노드의 검색키(5)를 비교한다.
2. 찾는 레코드(9)가 검색키 값(5)보다 큰 값이므로 오른쪽의 포인터를 따라 하위 노드로 이동한다.
3. 중간 노드에는 두 개의 검색키(7,8)이 있는데 찾는 레코드(9)는 이보다 더 크므로 오른쪽 포인터를 따라간다.
4. 이렇게 단말 노드에 도달하는 원하는 레코드(9)의 Value(실제 데이터 위치)값을 찾을 수 있다.

> B+Tree 장점

말단의 리프 노드들끼리는 **LinkedList** 구조로 서로를 참조하고 있다. 따라서 부등호(>, <)를 이용한 순차 검색 연산을 하는 경우, 많은 노드를 방문해야 하는 B-Tree에 비해 B+Tree는 말단 리프 노드를 저장한 LinkedList를 한 번만 탐색하는 등 속도 이점이 있다.

## Index 고려사항
`Index`는 항상 최신 상태로 정렬되기 위해, 데이터 갱신(INSERT, UPDATE, DELETE) 작업에 대해 추가적인 연산이 발생한다.
1. INSERT: 새로운 데이터에 대한 인덱스가 추가된다.
2. DELETE: 삭제하는 데이터의 인덱스를 제거한다.
3. UPDATE: 기존의 인덱스를 제거하고, 갱신된 데이터에 대해 인덱스를 추가한다.


앞서 살펴본 Index 트리 자료 구조는 값이 추가 혹은 삭제될 때마다, 트리 균형을 위해 트리 구조의 재분배 및 합병 등 복잡한 연산이 발생한다.  
따라서 **데이터 갱신보다는 조회에 주로 사용되는 컬럼** 에 Index를 생성하는 것이 유리하다.

## Index 대상 컬럼 선정
일반적으로 **Cardinality가 높은 컬럼**을 우선적으로 인덱싱하는 것이 검색 성능에 유리하다.  
Cardinality란 특정 데이터 집합의 유니크(Unique)한 값의 개수를 의미한다.  
* `성별 컬럼`: 남/여 2가지 값만 존재하므로 *중복도가 높으며 카디널리티가 낮다.*
* `주민번호 컬럼`: 개인마다 고유한 값이 존재하므로 *중복도가 낮으며 카디널리티가 높다.*


Cardinality 높은 컬럼의 경우, Index를 통해 데이터를 더 많이 필터링할 수 있다.
