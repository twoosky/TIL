# Topic, partition
* `토픽`은 카프카에서 데이터를 구분하기 위해 사용하는 단위
* 토픽은 1개 이상의 파티션을 소유하고 있다.
* `파티션`에는 프로듀서가 보낸 데이터들이 저장되는데, 이를 `레코드(record)`라고 한다.
* 파티션은 FIFO 구조와 같이 먼저 들어간 레코드는 컨슈머가 먼저 가져가게 된다.
* 카프카에서는 컨슈머가 레코드를 가져가더라도 파티션에서 삭제되지 않는다.
* 이러한 특징 때문에, 토픽의 레코드는 여러 컨슈머 그룹들이 여러번 가져갈 수 있다.

<img src="https://github.com/twoosky/TIL/assets/50009240/87c55752-2337-428e-b2e4-25f66899e004" width="550" height="180">

## 토픽 생성시 파티션이 배치되는 방법
* 파티션이 5개인 토픽을 생성한 경우 0번 브로커부터 시작하여 **round-robin 방식으로 리더 파티션들이 생성**된다.
* 프로듀서/컨슈머는 리더 파티션이 있는 브로커와 통신하여 데이터를 주고받으므로, 여러 브로커에 골고루 네트워크 통신을 하게 된다.
* 이를 통해, 특정 서버(브로커)에 통신이 집중되는 현상을 막고, 대용량 데이터에 대응할 수 있게 된다.
* 만약, 특정 브로커에 파티션이 몰리는 경우에는 `kafka-reassign-partitions.sh` 명령으로 파티션을 재분배할 수 있다.

<img src="https://github.com/twoosky/TIL/assets/50009240/0d3a4b49-82ce-403d-942e-4cffe356aada" width="530" height="280">

## 파티션 개수와 컨슈머 개수의 처리량
* 컨슈머와 파티션은 `1:n` 관계이다.
* 하나의 파티션은 컨슈머 그룹 내 하나의 컨슈머에 연결될 수 있지만, 하나의 컨슈머는 여러 개의 파티션과 연결될 수 있다.
* 많은 레코드를 병렬로 처리하기 위해선 파티션 개수와 컨슈머의 개수를 늘릴 수 있다.
* 프로듀서에서 초당 10개의 데이터를 파티션으로 보내지만, 컨슈머는 초당 1개의 데이터를 처리할 수 있는 경우 데이터 지연이 발생한다.
  --> 파티션 개수와 컨슈머 개수를 늘려 데이터 처리량 증가시켜야 함
* **카프카에서 파티션 개수를 줄이는 것은 지원하지 않는다**
<img src="https://github.com/twoosky/TIL/assets/50009240/582bc4c5-c8f4-489d-b3e0-e62ec32208cb" width="550" height="200">

## 클라이언트와 메타데이터
* 카프카 클라이언트(producer/consumer)는 통신하고자 하는 리더 파티션의 위치를 알기 위해 데이터를 주고(producer), 받기(consumer) 전에 메타데이터를 브로커로부터 전달받는다.
* 카프카 클라이언트는 반드시 리더 파티션과 통신해야한다. 잘못된 브로커로 데이터를 요청하면 LEADER_NOT_AVAILABLE 익셉션이 발생한다.
* 대부분의 경우 메타데이터 리프래시 이슈로 발생하므로, 메타데이터 리프래시 간격을 확인하고, 클라이언트가 정상적인 메타데이터를 갖고 있는지 확인해야 한다.

<img src="https://github.com/twoosky/TIL/assets/50009240/c6800e85-69a5-4f95-b325-e3ac85376c60" width="600" height="180">

**카프카 프로듀서 메타데이터 옵션**
* `metadata.max.age.ms`: 메타데이터를 강제로 리프래시하는 간격. 기본값은 5분
* `metadata.max.idle.ms`: 프로듀서가 유휴상태일 경우 메타데이터를 캐시에 유지하는 기간, 예를 들어 프로듀서가 특정 토픽으로 데이터를 보낸 이후 지정한 시간이 지나고 나면 강제로 메타데이터를 리프래시. 기본값은 5분