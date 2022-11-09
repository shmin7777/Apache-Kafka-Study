#### topic 만듬
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --topic test --partitions 3  


#### 메시지 보냄
bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test2  


#### 메시지 받음
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test2 --from-begin  

![image](https://user-images.githubusercontent.com/67637716/200835978-557ac6b3-6998-4cc9-b1ae-1a3fe45dee01.png)  


알아 낸 점 : partition을 1개만 했을 땐, 보낸 순서대로 받지만 partition이 여러개면 병렬로 데이터를 받기 때문에 순서가 보장되지 않는다.  


### cosumer group으로 받을 경우
![image](https://user-images.githubusercontent.com/67637716/200836660-9499124a-8b0f-40e2-ad54-c43f733ae6d2.png)  

bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test2 -group testgroup --from-beginning  

consumer를 group으로 지정해졌을 경우, 아까 test2 topic으로 받았던 offset을 기록해서 알고있기 때문에  
전에 받았던 data는 나오지 않고 그다음으로 받은 데이터를 처리한다.  






