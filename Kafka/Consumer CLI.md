# kafka-console-consumer.sh

### 1. 토픽의 레코드 조회
* 카프카 클러스터 정보(--bootstrap-server), 토픽 이름(--topic)을 통해 레코드를 조회할 수 있다.
* 추가로, --from-beginning 옵션을 주면 토픽에 저장된 가장 첫번째 데이터부터 출력한다.
```bash
$ bin/kafka-console-consumer.sh \
  --bootstrap-server my-kafka:9092 \
  --topic hello.kafka \
  --from-beginning

hello
kafka
no1
no2
```
### 2. 레코드 메시지 키와 메시지 값 조회
* --property 옵션을 통해 메시지 키와 메시지 값을 함께 조회할 수 있다.
```bash
$ bin/kafka-console-consumer.sh \
  --bootstrap-server my-kafka:9092 \
  --topic hello.kafka \
  --property print.key=true \
  --property key.separator="-" \
  --from-beginning

null-hello
null-kafka
key1-no1
key2-no2
```

### 3. 최대 컨슘 메시지 개수 설정
* --max-messages 옵션을 통해 최대 consume 메시지 개수를 설정할 수 있다.
```bash
$ bin/kafka-console-consumer.sh \
  --bootstrap-server my-kafka:9092 \
  --topic hello.kafka \
  --from-beginning \
  --max-messages 1

hello
```

### 4. 특정 파티션만 컨슘하기
* --partition 옵션을 사용하면 특정 파티션만 consume할 수 있다.
* 동일한 메시지 키가 설정된 레코드들이 특정 파티션으로 전송되는지 확인하는 경우 사용
```bash
$ bin/kafka-console-consumer.sh \
  --bootstrap-server my-kafka:9092 \
  --topic hello.kafka \
  --partition 2 \
  --from-beginning
```

### 5. 레코드 오프셋 커밋하기
* --group 옵션을 통해 컨슈머 그룹에 커밋할 수 있다.
* 즉, 해당 컨슈머 그룹이 어느 레코드까지 읽었는지에 대한 데이터가 카프카 브로커에 저장된다.
```bash
$ bin/kafka-console-consumer.sh \
  --bootstrap-server my-kafka:9092 \
  --topic hello.kafka \
  --group hello-group \
  --from-beginning

hello
kafka
no1
no2
```
* 오프셋 커밋 후 토픽 리스트를 조회하면 __consumer_offsets 토픽이 생성된 것을 확인할 수 있다.
* 카프카 브로커는 각 컨슈머 그룹의 레코드 오프셋을 해당 토픽에 저장한다.

# kafka-consumer-groups.sh

### 1. consumer group 목록 조회
* consumer group은 따로 생성하는 명령을 날리지 않고, 컨슈머를 동작할 때 컨슈머 그룹이름을 지정하면 생성된다.
```bash
$ kafka-consumer-groups.sh \
  --bootstrap-server my-kafka:9092 \
  --list

hello-group
```

### 2. consumer group 상태 조회
* --describe 옵션을 통해 해당 컨슈머 그룹이 어떤 토픽을 대상으로 레코드를 가져갔는지 상태 확인 가능
* 파티션 번호, 현재까지 가져간 레코드 오프셋, 파티션 마지막 레코드 오프셋, 컨슈머 랙, 컨슈머 ID, 호스트 조회 가능
* 컨슈머 LAG은 파티션의 마지막 레코드 오프셋과 현재까지 가져간 레코드 오프셋의 차이다. (지연 발생 정도 파악 가능)
* CURRENT-OFFSET은 현재까지 가져간 레코드 오프셋, LOG-END-OFFSET은 파티션 마지막 레코드 오프셋
```bash
$ kafka-consumer-groups.sh \
  --bootstrap-server my-kafka:9092 \
  --group hello-group \
  --describe

Consumer group 'hello-group' has no active members.

GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
hello-group     hello.kafka     0          12              12              0               -               -               -
```

### 3. consumer group 오프셋 리셋
* --reset-offsets {오프셋 리셋 종류} --execute 옵션을 통해 컨슈머 그룹의 오프셋을 초기화할 수 있다.
* 오프셋 리셋 종류
  * --to-earliest: 가장 처음 오프셋(작은 번호)으로 리셋
  * --to-latest: 가장 마지막 오프셋(큰 번호)으로 리셋
  * --to-current: 현 시점 기준 오프셋으로 리셋
  * --to-datetime {YYYY-MM-DDTHH:mmSS.sss}: 특정 일시로 오프셋 리셋(레코드 타임스탬프 기준)
  * --to-offset {long}: 특정 오프셋으로 리셋
  * --shift-by {+/- long}: 현재 컨슈머 오프셋에서 앞뒤로 옮겨서 리셋 
```bash
$ kafka-consumer-groups.sh \
  --bootstrap-server my-kafka:9092 \
  --group hello-group \
  --topic hello.kafka2 \
  --reset-offsets --to-earliest --execute
```
