# 1️⃣ 설치와 kafka server 실행하기
아래 사이트에서 Binary downloads에 있는 파일을 다운받는다.  
https://kafka.apache.org/downloads  


다운로드 받은 파일은 적절한 위치에 압축을 풀어준다.  
```
$ tar -xzf kafka_2.13-2.6.0.tgz
```  
압축이 해제되면 동명의 폴더가 생길 것이다.  
kafka는 이 폴더 내 bin 하위 스크립트 파일을 읽음으로써 실행된다.  

```
$ cd kafka_2.13-2.6.0
```  
kafka는 zookeeper 위에서 돌아가므로 zookeeper를 먼저 실행한다.  

```
$ bin/zookeeper-server-start.sh config/zookeeper.properties
```  

다음은 kafka를 실행한다.  

```
$ bin/kafka-server-start.sh config/server.properties
```  

# 2️⃣ Topic 생성하기
localhost:9092 카프카 서버에 quickstart-events란 토픽을 생성한다.  

```
$ bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
```  

####현재 만들어져 있는 토픽 확인하기  
```
$ bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```  

#### 특정 토픽의 설정 확인하기
```
$ bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092
```  

# 3️⃣ Producer, Consumer 실행하기
콘솔에서 Producer와 Consumer를 실행하여 실시간으로 토픽에 event를 추가하고 받을 수 있다.  
터미널을 분할로 띄워서 진행해본다.  

#### Producer
```
bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092
```  

#### Consumer
```
bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
```  

## 테스트
(eclipse + Gradle + Spring boot 환경에서 진행.)  

#### kafka 종속 추가
```
implementation 'org.springframework.kafka:spring-kafka'
```  
#### application.yml  
```
server:
  port: 8082

spring:
  application:
    name: third-point
  kafka:
    consumer:
      group-id: my-test
      enable-auto-commit: true
      auto-offset-reset: latest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      max-poll-records: 1000
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    template:
      default-topic: quickstart-events
    bootstrap-servers: localhost:9092
  zipkin:
    sender:
      type: kafka
 
```  

#### kafkaConfig
``` java
@Configuration
@ConfigurationProperties("spring.kafka")
@Data
public class KafkaConfig {

    private String bootstrapServers;
    private Producer producer;
    private Consumer consumer;
    private Template template;

    @Data
    public static class Producer {
        private String keySerializer;
        private String valueSerializer;
    }

    @Data
    public static class Template {
        private String defaultTopic;
    }

    @Data
    public static class Consumer {
        private String groupId;
        private String enableAutoCommit;
        private String autoOffsetReset;
        private String keyDeserializer;
        private String valueDeserializer;
        private String maxPollRecords;
    }



}
```  


#### CustomConsumer
``` java
@Service
@RequiredArgsConstructor
@Slf4j
public class CustomConsumer {

    private final KafkaConfig config;
    private KafkaConsumer<String, String> consumer = null;

    @PostConstruct
    public void build(){
        Properties properties = new Properties();
        properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, config.getBootstrapServers());
        properties.setProperty(ConsumerConfig.GROUP_ID_CONFIG, config.getConsumer().getGroupId());
        properties.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, config.getConsumer().getKeyDeserializer());
        properties.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, config.getConsumer().getValueDeserializer());
        properties.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, config.getConsumer().getAutoOffsetReset());
        properties.setProperty(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, config.getConsumer().getMaxPollRecords());
        properties.setProperty(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, config.getConsumer().getEnableAutoCommit());
        consumer = new KafkaConsumer<>(properties);
    }

    @KafkaListener(topics = "${spring.kafka.template.default-topic}")
    public void consume(@Header(KafkaHeaders.RECEIVED_TOPIC) String topic, @Payload String payload){
        log.info("CONSUME TOPIC : "+topic);
        log.info("CONSUME PAYLOAD : "+payload);
    }

}

```  

#### CustomProducer
``` java
@Service
@RequiredArgsConstructor
@Slf4j
public class CustomProducer {

    private final KafkaConfig config;
    private KafkaProducer<String, String> producer = null;

    @PostConstruct
    public void build(){
        Properties properties = new Properties();
        properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, config.getBootstrapServers());
        properties.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, config.getProducer().getKeySerializer());
        properties.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, config.getProducer().getValueSerializer());
        producer = new KafkaProducer<>(properties);
    }

    public void send(String message) {
        ProducerRecord<String, String> record = new ProducerRecord<>(config.getTemplate().getDefaultTopic(), message);

        producer.send(record, new Callback() {
            @Override
            public void onCompletion(RecordMetadata metadata, Exception exception) {
                log.info("publish message: {}", message);
                if (exception!=null){
                    log.info(exception.getMessage());
                }
            }
        });
    }

}
```  


#### CustomProducerTest
CustomProducer를 사용해 메세지를 전송하는 테스트를 작성한다.  

``` java
@SpringBootTest
class CustomProducerTest {

    @Autowired
    private CustomProducer sut;

    @Test
    void test1(){
        sut.send("this message sent from spring boot application!");
    }

}
```  
kafka consumer를 실행한 뒤 테스트를 수행해보자.  


테스트 코드의 Producer가 전송한 메세지를 Consumer가 받았음을 알 수 있다.  

#### CustomConsumerTest
콘솔의 producer에서 메세지를 보내보자



IDE 내 콘솔에 다음과 같은 로그가 찍힐 것이다.  
``` 
2020-08-23 15:27:02.340  INFO [third-point,3c74a95e62f7f6bd,9b79595f0e9204ec,true] 16164 --- [ntainer#0-0-C-1] com.example.kafka.CustomConsumer         : CONSUME TOPIC : quickstart-events
2020-08-23 15:27:02.340  INFO [third-point,3c74a95e62f7f6bd,9b79595f0e9204ec,true] 16164 --- [ntainer#0-0-C-1] com.example.kafka.CustomConsumer         : CONSUME PAYLOAD : to spring boot
```  

#### ThirdController
Spring Boot Application의 CustomProducer와 CustomConsumer를 동시에 사용하도록 컨트롤러를 작성해보자.  

``` java
@RestController
@RequestMapping("/third")
@RequiredArgsConstructor
@Slf4j
public class ThirdController {

    @Autowired
    private CustomProducer producer;

    @PostMapping("/publish")
    public void producer(@RequestBody Map<String, String> message){
        log.info(">>> start event publish");
        producer.send(message.get("msg"));
    }

}
``` 
터미널에서 요청을 보내보자  

```  
$ curl -XPOST 'http://localhost:8082/third/publish' -H 'Content-Type:application/json' -d '{"msg":"hello world!"}'
```  
로그는 아래와 같다.  
```  
2020-08-23 15:21:36.098  INFO [third-point,d49790ceafd1d102,d49790ceafd1d102,true] 16164 --- [nio-8082-exec-7] com.example.controller.ThirdController   : >>> start event publish
2020-08-23 15:21:36.100  INFO [third-point,,,] 16164 --- [ad | producer-1] com.example.kafka.CustomProducer         : publish message: hello world!
2020-08-23 15:21:36.101  INFO [third-point,02f37a5c64e4874c,abda7d1fd8b18eea,false] 16164 --- [ntainer#0-0-C-1] com.example.kafka.CustomConsumer         : CONSUME TOPIC : quickstart-events
2020-08-23 15:21:36.101  INFO [third-point,02f37a5c64e4874c,abda7d1fd8b18eea,false] 16164 --- [ntainer#0-0-C-1] com.example.kafka.CustomConsumer         : CONSUME PAYLOAD : hello world!
```  
CustomProducer를 통해 hello world!라는 메세지를 발행하고, 이 이벤트를 CustomConsumer측에서 받아오는 것이다.
