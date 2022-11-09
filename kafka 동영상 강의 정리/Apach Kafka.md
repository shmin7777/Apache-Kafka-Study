# Kafka가 나오게 된 배경

#### Before Kafka  
![image](https://user-images.githubusercontent.com/67637716/200785355-e5214ced-a6aa-4537-acd4-bbc599cd727d.png)  

* end to end 연결 방식의 아키텍쳐
* 데이터 연동의 복잡성 증가(하드웨어, 운영체제, 장애 등)
* 각기 다른 데이터 파이프라인 연결 구조
* 확장에 엄청난 노력 필요

=> 모든 시스템으로 데이터를 전송 실시간 처리도 가능한 것
=> 데이터가 갑자기 많아지더라도 확장이 용이한 시스템이 필요함

#### After Kafka
![image](https://user-images.githubusercontent.com/67637716/200785772-e55668dc-9711-4f4a-bbfb-6505bf65a012.png)  

* 프로듀서/컨슈머 분리
* 메시지 데이터를 여러 컨슈머에게 허용
* 높은 처리량을 위한 메시지 최적화
* 스케일 아웃 가능(무중단으로 가능)
* 관련 생태계 제공

# Kafka broker
* 실행된 카프카 애플리케이션 서버 중 1대
* 3대 이상의 브로커로 클러스터 구성
* 주키퍼와 연동
    * 주키퍼의 역할: 메타데이터(브로커id, 컨트롤러id 등) 저장
    * 추후에는 주키퍼를 걷어낸다고 함
* n개 브로커 중 1대는 컨트롤러(Controller)기능 수행
    * 컨트롤러 : 각 브로커에게 담당 파티션 할당 수행.
    * 브로커 정상 모니터링 관리
    * 누가 컨트롤러 인지는 주키퍼에 저장
![image](https://user-images.githubusercontent.com/67637716/200787132-2d655e60-024a-44e7-a61c-5931cfb827af.png)  
``` java
// topic, key, message를 정하여 send
new ProducerRecord<String, String>("topic", "key", "message");

// topic의 data를 record로 받아온다.
ConsumerRecords<String, String> records = consumer.poll(1000);
for(ConsumerRecord<String, String> record : records){
  ...
}
```  
* 객체를 프로듀서에서 컨슈머로 전달하기 위해 Kafka 내부에 byte형태로 저장할 수 있도록 직렬화/역직렬화하여 사용
* 기본 제공 직렬화 class : StringSerializer, ShortSerializer 등
* 커스텀 직렬화 class를 통해 Custom Object 직렬화/역직렬화 가능


# Topic
![image](https://user-images.githubusercontent.com/67637716/200788559-ff88401c-2343-440d-b559-5740b34ba6ce.png)  
* 메시지 분류 단위
* n개의 파티션 할당 가능
* 파티션은 1개이상 반드시 존재해야함
* 각 파티션마다 고유한 오프셋(offset)을 가짐
* 메시지 처리순서는 파티션 별로 유지 관리된다.

![image](https://user-images.githubusercontent.com/67637716/200789018-e32d47e0-59e6-4c75-b6b5-b736191b5626.png)  
* 프로듀서는 레코드를 생성하여 브로커로 전송
* 전송된 레코드는 파티션에 신규 오프셋과 함께 기록
* 컨슈머는 브로커로 부터 레코드를 요청하여 가져감(polling)

### Kafka log and segment
* 실제로 메시지가 저장되는 파일시스템 단위 : 세그먼트
* 세그먼트는 시간 또는 크기 기준으로 닫힘
* 세그먼트가 닫힌 이후 일정 시간(또는 용량)에 따라 삭제 또는 압축(compact)

=> 카프카에 들어가는 데이터는 영원히 사용할수 있지만 일정기간 또는 용량에 대한 옵션을 주기 때문에 언젠가는 사라진다.  










     
   




