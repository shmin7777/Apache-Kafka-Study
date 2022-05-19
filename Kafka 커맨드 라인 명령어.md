기본적으로 kafka home(/usr/opt/kafka/kafka_2.12-3.2.0) 에서 작업  

# Kafka shell scripts
kafka-topics.sh  
👉 토픽 생성, 조회, 수정 등 역할  

kafka-console-consumer.sh  
👉 토픽의 레코드 즉시 조회  

kafka-console-producer.sh  
👉 토픽의 레코드를 전달(String)  

kafka-consumer-groups.sh  
👉 컨슈머그룹 조회, 컨슈머 오프셋 확인, 수정  

위 4개를 포함하여 33개의 Kafka shell script가 제공됨  

<hr>

## zookeeper / Kafka  config 파일 확인
``` 
# zookeeper
$ vi config/zookeeper.properties

# kafka
$ vi config/server.properties
```  

## zookeeper / Kafka  실행 확인
``` 
# zookeeper
$ vi bin/zookeeper-server-start.sh

# kafka
$ vi bin/kafka-server-start.sh
```  
- zookeeper/Kafka  서버를 실행시키는 쉘 파일  

- 실행 시 아래와 같이 실행 파일을 같이 넘겨 줘야한다  

```
$ bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
$ bin/kafka-server-start.sh -daemon config/server.properties  
```  


- 데몬으로 실행 시키기 위해서는 -daemon 파라미터 값을 같이 줘야한다 (백엔드 실행)  

- 카프카 실행 전 먼저 쥬키퍼가 실행 되어야 한다.  


## topic
####  기본 토픽 생성
``` 
bin/kafka-topics.sh \
--create \
--bootstrap-server my-kafka:9092 \
--topic [토픽 이름]
```  

#### 옵션을 추가한 토픽 생성
```
bin/kafka-topics.sh \
--create \
--bootstrap-server my-kafka:9092 \
--topic [토픽 이름] \
--partitions [생성할 파티션 수] \
--replication-factor [브로커 복제 계수] \
--config retention.ms=[토픽의 데이터를 유지할 시간 (단위: ms)]
```  

* kafka-topics.sh - 카프카 토픽을 생성, 삭제, 조회, 변경
```
--bootstrap-server : 토픽관련 명령어를 수행할 대상 카프카 클러스터
--replication-factor : 토픽의 파티션의 복제본을 몇 개를 생성할 지에 대한 설정(브로커 개수 이하로 설정 가능)
--partitions : 파티션 개수 설정
--config : 각종 토픽 설정 가능(retention.ms, segment.byte 등)
--create : 토픽 생성
--delete : 토픽 제거
--describe : 토픽 상세 확인
--list : 카프카 클러스터의 토픽 리스트 확인
--version : 대상 카프카 클러스터 버젼 확인
```  

#### 토픽 리스트 조회
```
bin/kafka-topics.sh \
--bootstrap-server my-kafka:9092 \
--list
```  

#### 특정 토픽 상세 조회
토픽별 토픽명, 존재하는 파티션 번호, 리더 브로커 번호, 복제 계수, ISR 등을 알 수 있다.  
```
bin/kafka-topics.sh \
--bootstrap-server my-kafka:9092 \
--topic [조회할 토픽 이름] \
--describe
```  


## Message 보내기
#### 메세지 전송
```
$  bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```  
* bin/kafka-console-producer.sh : 프로듀서 콘솔 실행파일

* --bootstrap-server : 호스트/포트 지정

* --topic : 메세지를 보낼 토픽 지정

- 위 명령어를 입력하면 입력 콘솔로 변한다.  

- 이때 기본적으로 한줄 단위로 분리된 메세지로 보내지게 된다.  


#### 메세지 읽기
```
$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```  
* bin/kafka-console-consumer.sh : 컨슈머 콘솔 실행파일

* --bootstrap-server : 호스트/포트 지정

* --topic : 메세지를 읽을 토픽 지정

* --from-beginning : 메세지를 어디서 부터 읽을 지 지정
