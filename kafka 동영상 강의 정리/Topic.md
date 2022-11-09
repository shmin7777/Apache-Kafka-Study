# Topic
카프카에는 다양한 데이터가 들어갈 수 있는데 데이터가 들어갈 수 있는 공간을 Topic이라고 부른다.  

![image](https://user-images.githubusercontent.com/67637716/200747518-41996ac8-2aa0-4a92-b58b-17d69bb59f59.png)  

kafka에서는 토픽을 여러개 생성할 수 있다.  
데이터베이스의 테이블이나 파일시스템의 폴더와 유사하다.  

Producer는 카프카에 데이터를 넣게 되고 Comsumer는 데이터를 가져가게 된다.  
Topic은 이름을 가질 수 있다.  목적에 따라 명확하게 명시하는 것이 유지보수 시 편리하다.  


# Topic 내부  
하나의 토픽은 여러개의 partition으로 구성할 수 있다.  
첫번째 파티션 번호는 0번부터 시작.  
하나의 파티션은 큐와 같이 내부에 데이터가 파티션 끝에서부터 차곡차곡 쌓이게 된다.  
![image](https://user-images.githubusercontent.com/67637716/200747805-dc8d5b13-16fe-4f25-9568-acf23f3aa199.png)  

click log 토픽에 카프카 컨슈머가 붙게 되면  데이터를 가장 오래된 순서대로 가져가게 된다.  

더이상 데이터가 들어오지않으면 컨슈머가 또 다른데이터가 들어올때까지 기다린다.  

![image](https://user-images.githubusercontent.com/67637716/200747995-72b41ed1-7671-4517-8895-589cb3c37935.png)  

<b>컨슈머가 토픽 내부의 파티션에서 데이터를 가져가더라도 데이터는 삭제되지 않는다. </b>  

파티션에 그대로 남아서 새로운 컨슈머가 붙었을 때 다시 0번부터 가져가서 사용할 수 있다.  
 
![image](https://user-images.githubusercontent.com/67637716/200748065-9387fcbc-9fe0-4d29-ab2c-6dddf54fc2fd.png)  
```  
- 컨슈머 그룹이 달라야하고,
- auto.offset.reset = earliest 
```  
위와 같이 세팅되어 있어야한다.  
이 처럼 동일 데이터에 대해 두번 처리할 수 있는데, `카프카를 사용하는 아주 중요한 이유`이다.  


# Partition이 2개 이상인 경우
파티션을 늘리는 이유는 컨슈머의 개수를 늘려서 데이터 처리를 분산할 수 있다.  

프로듀서가 데이터를 보낼 때 키를 저장할 수 있다.  
![image](https://user-images.githubusercontent.com/67637716/200748593-9b8b0972-dbc7-4a86-a074-a6fea6afa402.png)  

파티션을 늘리는 것은 아주 조심해야한다.  
파티션을 늘리는 것은 가능하지만 줄일 수는 없다. 

![image](https://user-images.githubusercontent.com/67637716/200749986-5a43072a-d68b-4898-8d70-9983cae91b1e.png)  
키를 지정하는 경우, key를 특정한 hash값으로 변환시켜 파티션과 1:1매칭이 되게한다.  
따라서 위의 코드를 반복해서 실행시키면 각 파티션에 동일 key의 value만 쌓이게 되는 것. 

![image](https://user-images.githubusercontent.com/67637716/200750372-30eaa298-7e0c-4417-b05f-559da381d3c4.png)  
파티션을 1개 더 추가하게 되면, key와 파티션의 매칭이 깨지기 때문에  
토픽에 파티션을 추가하는 순간 키<->파티션의 일관성은 보장되지 않는다.  
key를 사용할 경우 이점을 유의해 파티션 개수를 생성하고 추후에 생성하지 않는 것을 권장.  





#### Partition의 recode는 언제 삭제되는가?
- log.retention.ms : 최대 record 보존 시간
- log.retention.byte : 최대 recored 보존 크기(byte) 
일정한 기간, 용량동안 데이터를 저장하고 적절하게 데이터가 삭제될 수 있다.  





