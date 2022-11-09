#### topic 만듬
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --topic test --partitions 3  


#### 메시지 보냄
bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test2  


#### 메시지 받음
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test2 --from-begin  

![image](https://user-images.githubusercontent.com/67637716/200835978-557ac6b3-6998-4cc9-b1ae-1a3fe45dee01.png)  


알아 낸 점 : partition을 1개만 했을 땐, 보낸 순서대로 받지만 partition이 여러개면 병렬로 데이터를 받기 때문에 순서가 보장되지 않는다.  



