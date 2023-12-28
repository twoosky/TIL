# kafka-console-producer.sh

### 1. 토픽에 데이터(Record) 전송
* 카프카 클러스터 정보(--bootstrap-server), 토픽 이름을 통해 레코드 전송 가능
* 입력하는 값은 레코드의 message value값으로 설정되어 전송된다.
```bash
$ kafka-console-producer.sh \
  --bootstrap-server my-kafka:9092 \
  --topic hello.kafka

>hello
>kafka
```

### 2. 메시지 키를 갖는 레코드 전송
* message key를 갖는 레코드를 전송하기 위해서는 parse.key=true 옵션을 명시해야 한다.
* key.separator 옵션은 메시지 키와 메시지 값을 구분하는 구분자값이다. (기본값은 탭)
```bash
$ kafka-console-producer.sh \
  --bootstrap-server my-kafka:9092 \
  --topic hello.kafka \
  --property "parse.key=true" \
  --property "key.separator=:"

>key1:no1
>key2:no2
>key3:no3
```


**메시지 키와 메시지 값이 포함된 레코드의 파티션 할당 방식**

<img width="600" alt="image" src="https://github.com/twoosky/TIL/assets/50009240/adfbf48d-d951-4a80-ba02-d2a60b13639d">


* parse.key=true 옵션을 통해 메시지 키와 메시지 값이 포함된 레코드가 파티션에 전송된다.
* 메시지 키를 지정하지 않은 경우 메시지 키가 Null로 설정된다.
* 메시지 키가 null인 경우 프로듀서가 파티션으로 전송할 때 레코드 배치 단위(레코드 전송 묶음)로 라운드 로빈으로 파티션에 전송된다.
* 메시지 키가 존재하는 경우에는 키의 해시값을 통해 하나의 파티션에만 할당된다.
* 즉, 메시지 키가 동일한 경우 동일한 파티션으로 전송된다. 이를 통해 메시지 키가 같은 레코드에 대해서 순서를 보장할 수 있다.
  * 하나의 파티션에만 존재하므로, 하나의 컨슈머가 순서대로 소비함
  * 위 그림에서는 메시지 키가 K1인 레코드는 파티션 1에만 할당된 것을 볼 수 있음
