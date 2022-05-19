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

