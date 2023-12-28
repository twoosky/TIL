# kafka-console-consumer.sh

### 1. 토픽의 레코드 조회하기
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
### 2. 레코드 메시지 키와 메시지 값 조회하기
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

### 3. 최대 컨슘 메시지 개수 설정하기
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
