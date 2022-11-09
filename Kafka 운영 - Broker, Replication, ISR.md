broker, replication , ISR는 운영에있어서 아주 중요한 역할을 함.  

Replication(복제)는 카프카 아키텍처의 핵심  
클러스테서 서버가 장애가 생길때 카프카의 가용성을 보장하는 가장 좋은 방법.  

<hr>  


# Kafka broker
카프카가 설치되어 있는 서버 단위  
보통 3개 이상의 broker로 구성하여 사용하는 것 권장  
파티션이 하나이고 replication이 1인 topic이 존재하고 브로커가 3대라면
브로커 3대중 1대에 해당 토픽정보가 저장됨

replication은 partition의 복제를 뜻함.  
replicatino이 2라면 파티션은 원본 1개와 복제본 한개로 총 2개가 존재함  

브로커 개수에 따라서 replication 개수가 제한된다.  
예를 들어 브로커 개수가 3이면 replication개수는 4가 될 수 없음.  

![image](https://user-images.githubusercontent.com/67637716/200744503-33cdd909-7a50-412d-becd-90b930a61119.png)   

원본 1개의 파티션은 Leader partition이라 부르고 나머지 복제 파티션은 Follower partition이라고 부른다.  
Leader, Follower partition을 합쳐서 ISR, (In Sync Replica)라고 볼 수 있다.  

# replication 을 사용하는 이유??
![image](https://user-images.githubusercontent.com/67637716/200744966-9be18797-2720-4a0b-b128-59d610e18d57.png)  

partition 의 고갸용성을 위해 사용됨  
브로커가 3개인 카프카에서  
replication 이 1이고 partition이 1인 topic이 존재한다고 할때.  
갑자기 브로커가 어떠한 이유로 사용 불가 하게 된다면,  더이상 해당 파티션을 복구할 수 없다.  

replication 이 2라면 브로커가 1개 가 죽더라도 Follower partition이 존재하므로 복제본으로 복구가 가능함.  
나머지 1개가 남은 Follower partition이 Leader partition역할을 승계하게 되는 것.  

# Leader partition과 Follower partition의 역할?
프로듀서가 토픽의 파티션에 전달한다.  
프로듀서가 토픽의 파티션에 데이터를 전달할 때 전달받는 주체가 바로 Leader partition.  

프로듀서에는 ack랑는 상세옵션이 있다. 그것을 통해 고갸용성을 유지할 수 있는데 이 옵션은 partition의 replication과 관련이 있다.  

ack는 0,1,all 옵션 3개중 1개를 골라 사용 할 수 있다.  

![image](https://user-images.githubusercontent.com/67637716/200745565-752d78fb-7422-4e14-a804-e025bcfc49eb.png)  
ack = 0일 경우 leader partition에 데이터를 전송하고 응답값을 받지 않는다.  
leader partition에 데이터를 정ㅇ상적으로 전송됐는지, 나머지 partition에 정상적으로 복제되었는지 알수 없고 보장 x  
속도는 빠르지만 데이터 유실가능성 있다.  
 
![image](https://user-images.githubusercontent.com/67637716/200745904-8cd66c9c-90f8-4eb1-a024-aaeae8781950.png)  
ack = 1일 경우  Leaderpartition에 데이터를 전송하고 응답값을 받는다.  
다만 나머지 파티션에 복제되었는지 알수 없다.  
만약 리더 파티션이 데이터를 받은 즉시 브로커가 장애가 난다면 나머지 파티션에 데이터가 미쳐 전송되지 못한 상태이므로  
이전에 ack 0 옵션과 같이 데이터 유실 가능성 있다.  

![image](https://user-images.githubusercontent.com/67637716/200745962-e33f72f1-5bd9-423b-85d9-5bd6a335b195.png)  
ack = all일 경우 1옵션에 추가로 follower partitiondㅔ 복제가 잘 되었는지 응답값을 받는다.
Leader partition에 데이터를 보낸 후 Follower partition에 데이터가 잘 저장되었는지 확인하는 절차를 거친다.  
데이터 유실은 없지만, 0, 1에 비해 확인하는 부분이 많기 때문에 속도가 현저히 느리다는 단점이 있다.




