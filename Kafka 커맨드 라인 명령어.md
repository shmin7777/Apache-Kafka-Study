기본적으로 kafka home(/usr/opt/kafka/kafka_2.12-3.2.0) 에서 작업  
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
--replication-factor : 레플리카 개수 지정(브로커 개수 이하로 설정 가능)
--partitions : 파티션 개수 설정
--config : 각종 토픽 설정 가능(retention.ms, segment.byte 등)
--create : 토픽 생성
--delete : 토픽 제거
--describe : 토픽 상세 확인
--list : 카프카 클러스터의 토픽 리스트 확인
--version : 대상 카프카 클러스터 버젼 확인
```  

