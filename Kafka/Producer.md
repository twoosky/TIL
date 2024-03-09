# Producer
* 카프카에서 데이터의 시작점은 프로듀서이다.
* 프로듀서 애플리케이션은 카프카에 필요한 데이터를 선언하고, 특정 토픽의 파티션에 전송한다.
* 프로듀서는 리더 파티션을 갖는 카프카 브로커와 직접 통신한다 (리더 파티션하고만 통신, 팔로워 파티션과는 통신 X)
* 프로듀서는 카프카 브로커로 데이터를 전송할 때 내부적으로 파티셔너, 배치 생성 단계를 거친다.

<img width="500" alt="image" src="https://github.com/twoosky/TIL/assets/50009240/c820ae83-4f55-4356-8786-d70fa6ed3b31">


> 카프카 클러스터에는 여러 개의 브로커 존재 -> 각 브로커에는 여러 개의 토픽 존재 -> 각 토픽에는 여러 개의 파티션 존재 (리더/팔로워)

## Producer 내부 구조
* `ProducerRecord`: 프로듀서에서 생성하는 레코드. 오프셋은 미포함 (오프셋은 파티션에 저장된 후 지정됨)
  * 토픽과 메시지 값만 존재하면 데이터 전송 가능
* `send()`: 레코드 전송 요청 메서드
* `Partitioner`: 어느 파티션으로 전송할지 지정하는 역할. 기본값은 DefaultPartitioner로 설정됨
  * 레코드의 메시지 키에 따라 파티션 지정 (동일한 메시지 키를 갖는 레코드는 동일한 파티션에 전송됨)
* `Accumulator`: 배치로 묶어 전송할 데이터를 모으는 버퍼
  * TCP 통신으로 데이터를 전송할 때 한 번에 최대한 많은 데이터를 전송하기 위해 배치로 묶어 전송 (데이터 처리량 향상)
  * 배치 크기만큼 버퍼에 데이터가 쌓이게 되면 Sender를 통해 카프카 브로커로 데이터 전송

<img width="570" alt="image" src="https://github.com/twoosky/TIL/assets/50009240/959d16e5-a768-4c87-970b-3853f6472047">

## Partitioner
* 프로듀서의 파티셔너 종류에는 UniformStickyPartitioner와 RoundRobinPartitioner가 있다.
* 카프카 클라이언트 라이브러리 2.5.0 버전에서 파티셔너를 지정하지 않은 경우 UniformStickyPartitioner로 기본 설정된다.

**메시지 키가 있을 경우 동작**
* 두 파티셔너 모두 메시지 키의 해시값과 파티션을 매칭하여 레코드 전송
* 동일한 메시지 키를 갖는 레코드는 동일한 파티션 번호에 전달된다.
* ***만약 파티션 개수가 변경될 경우 메시지 키와 파티션 번호 매칭은 깨지게 된다.***
* 따라서, 컨슈머 개수를 고려해 파티션 개수를 충분히 설정해야한다.

**메시지 키가 없는 경우 동작**
* RoundRobinPartitioner
  * ProducerRecord가 들어오는 대로 파티션을 순회하면서 레코드 전송
  * Accumulator에서 배치로 묶이는 데이터가 적기 때문에 전송 성능이 낮다.
* UniformStickyPartitioner
  * Accumulator에서 레코드들이 배치로 묶일 때까지 기다린 후 전송
  * 배치 단위로 파티션을 순회하면서 전송하기 때문에 모든 파티션에 분배되어 전송된다. ([참고](https://blog.voidmainvoid.net/360))
  * RoundRobinPartitioner 파티셔너에 비해 전송 성능이 높다.

**커스텀 Partitioner**
* 카프카 클라이언트 라이브러리에서는 사용자 지정 파티셔너를 생성하기 위한 Partitioner 인터페이스를 제공한다.
* Partitioner 인터페이스를 상속받은 사용자 정의 클래스에서 *메시지 키 또는 메시지 값에 따른 파티션 지정 로직을 정의할 수 있다.*
  * ex) message key가 '서울'인 경우 항상 0번 파티션으로 보내라
  * ex) message value에 '인천'이 포함된 경우 항상 1번 파티션으로 보내라

## Producer 옵션
**필수 옵션**
* `bootstrap.servers`: 프로듀서가 데이터를 전송할 카프카 클러스터에 속한 브로커의 **호스트 이름:포트** 를 1개 이상 작성한다.
  * 2개 이상 브로커 정보를 입력하면 일부 브로커에 이슈가 발생해도 데이터 전송 가능
* `key.serializer`: 레코드의 메시지 키를 직렬화하는 클래스를 지정한다.
* `value.serializer`: 레코드의 메시지 값을 직렬화하는 클래스를 지정한다.
  * 프로듀서는 데이터를 직렬화해 레코드를 전송한다. 이를 통해 모든 타입의 데이터를 카프카 브로커에 저장할 수 있다.
  * 카프카 브로커에서는 직렬화된 데이터를 세그먼트 로그로 남기고, 로그를 토대로 컨슈머가 브로커로부터 데이터를 가져가 처리한다.
  * 컨슈머에서는 역직렬화한다. 이를 위해선 어떻게 직렬화가 되어있는지 알아야 한다.
    * 토픽 이름에 직렬화 방법을 포함하는 것도 좋은 방법, 직렬화 방법을 String으로 통일하는 것도 방법

**선택 옵션**
* `acks`: 프로듀서가 전송한 데이터가 브로커들에 정상적으로 저장되었는지 전송 성공 여부를 확인하기 위한 옵션
  * 0, 1, -1(all) 중 하나로 설정 가능. 기본값은 1
* `linger.ms`: 배치를 전송하기 전까지 기다리는 최소 시간, 기본값은 0이다.
* `retries`: 브로커로부터 에러를 받고 난 뒤 재전송을 시도하는 횟수를 지정한다. 기본값은 2147483647
* `max.in.flight.requests.per.connection`: 한 번에 요청하는 최대 커넥션 수, 설정된 값만큼 sender에서 동시에 데이터 전송. 기본값은 5
* `partitioner.class`: 파티셔너 클래스 지정 옵션, 기본값은 defaultPartitioner
* `enable.idempotence`: 멱등성 프로듀서로 동작할지 여부 설정, 2.5.0버전의 기본값은 false, 3점대 버전의 기본값은 true
* `transactional.id`: 프로듀서가 레코드를 전송할 때 레코드를 트랜잭션 단위로 묶을지 여부 설정, 기본값은 null