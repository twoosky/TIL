# kafka-topics.sh
### 1. kafka topic 생성 (기본 옵션)
* 카프카 클러스터 정보와 토픽 이름은 토픽을 생성하기 위한 필수 값이다.
* 카프카 클러스터 정보와 토픽 이름만 명시한 경우 파티션 개수, 복제 개수 등의 옵션은 브로커에 설정된 기본값으로 생성된다.
* 파티션 개수, 복제 개수, 토픽 데이터 유지 기간 옵션 등을 지정하여 토픽을 생성할 수도 있다.
```sh
$ kafka-topics.sh --create --bootstrap-server my-kafka:9092 --topic hello.kafka
```
```sh
$ kafka-topics.sh --create \
  --bootstrap-server my-kafka:9092 \
  --partitions 10 \
  --replication-factor 1 \
  --topic hello.kafka2 \
  --config retention.ms=172800000
```
### 2. kafka topic 정보 조회
* --describe 옵션을 통해 파티션 개수, 복제 개수 등의 정보를 조회할 수 있다.

```bash
$ kafka-topics.sh --bootstrap-server my-kafka:9092 --topic hello.kafka. --describe

Topic: hello.kafka.	TopicId: 2v9_sjJGRoyg2HEObWXO3A	PartitionCount: 1	ReplicationFactor: 1	Configs:
	Topic: hello.kafka.	Partition: 0	Leader: 1001	Replicas: 1001	Isr: 1001
```

### 3. kafka topic 목록 조회
* --list 옵션을 통해 생성된 토픽들의 이름을 조회할 수 있다.
```bash
$ kafka-topics.sh --bootstrap-server my-kafka:9092 --list

hello.kafka
hello.kafka2
```

### 4. kafka topic 파티션 개수 늘리기
* --alter 옵션을 통해 파티션의 개수를 늘릴 수 있다.
```bash
$ kafka-topics.sh --bootstrap-server my-kafka:9092 --topic hello.kafka2 --alter --partitions 11
```
* 파티션 개수를 늘릴 수는 있지만, 줄일 수는 없다. 파티션 개수를 줄이면 아래와 같이 InvalidPartitionsException이 발생한다.
* 분산 시스템에서 이미 분산된 데이터를 줄이는 방법은 매우 복잡하다. 따라서, 카프카는 파티션을 줄이는 로직을 제공하지 않는다.
```bash
$ kafka-topics.sh --bootstrap-server my-kafka:9092 --topic hello.kafka2 --alter --partitions 4

Error while executing topic command : Topic currently has 10 partitions, which is higher than the requested 4.
[2023-12-28 09:59:02,048] ERROR org.apache.kafka.common.errors.InvalidPartitionsException: Topic currently has 10 partitions, which is higher than the requested 4.
```
> 파티션 개수를 줄여야 하는 경우 토픽을 새로 만드는 것이 좋다.
