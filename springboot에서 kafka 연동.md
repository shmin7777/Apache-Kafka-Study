Springboot에서 Kafka의 특정 Topic에 메시지를 생산(Produce)하고 해당 Topic을 Listen.  
Kafka 서버에 해당 메시지가 전달되고, Springboot에서 이를 소비(Consume)할 준비가 되면 메시지를 pull 하는 아주아주 간단한 예제.  
<hr>

## 개발 환경 셋팅
1) 프로젝트 구조  
![image](https://user-images.githubusercontent.com/67637716/169437049-bab31418-240c-4956-8bef-261d88bd39dc.png)  

2) Kafka 설치  

3) build.gradle  
kafka 연동을 위해 spring-kafka 의존성을 추가.  
``` java
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'

    /* kafka */
    implementation 'org.springframework.kafka:spring-kafka'

    testImplementation('org.springframework.boot:spring-boot-starter-test') {
        exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    }
}
```   



## 구현하기
1) application.yml  
consumer와 producer에 대한 설정을 한다.  
``` java
spring:
  kafka:
    consumer:
      bootstrap-servers: localhost:9092
      group-id: foo
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
    producer:
      bootstrap-servers: localhost:9092
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
```  

#### spring.kafka.consumer
* bootstrap-servers
Kafka 클러스터에 대한 초기 연결에 사용할 호스트:포트쌍의 쉼표로 구분된 목록.  
글로벌 설정이 있어도, consumer.bootstrap-servers가 존재하면 consuemer 전용으로 오버라이딩.  

* group-id
Consumer는 Consumer Group이 존재하기 때문에, 유일하게 식별 가능한 Consumer Group을 작성.  

* auto-offset-reset
Kafka 서버에 초기 offset이 없거나, 서버에 현재 offset이 더 이상 없는 경우 수행할 작업을 작성.  

* Consumer Group
Consumer Group의 Consumer는 메시지를 소비할 때 Topic내에 Partition에서 다음에 소비할 offset이 어디인지 공유를 하고 있다.  
그런데 오류 등으로 인해. 이러한 offset 정보가 없어졌을 때 어떻게 offeset을 reset 할 것인지를 명시.  
  * latest : 가장 최근에 생산된 메시지로 offeset reset
  * earliest : 가장 오래된 메시지로 offeset reset
  * none : offset 정보가 없으면 Exception 발생
직접 Kafka Server에 접근하여 offset을 reset할 수 있지만, Spring에서 제공해주는 방식은 위와 같음.  

* key-deserializer / value-deserializer
Kafka에서 데이터를 받아올 때, key / value를 역직렬화.  
여기서 key와 value는 뒤에서 살펴볼 KafkaTemplate의 key, value를 의미.  
이 글에서는 메시지가 문자열 데이터이므로 StringDeserializer를 사용.  
JSON 데이터를 넘겨줄 것이라면 JsonDeserializer도 가능하다.  

#### spring.kafka.producer
* bootstrap-servers
consumer.bootstrap-servers와 동일한 내용이며, producer 전용으로 오버라이딩 하려면 작성.  

* key-serializer / value-serializer
Kafka에 데이터를 보낼 때, key / value를 직렬화.  
consumer에서 살펴본 key-deserializer, value-deserializer와 동일한 내용.  
더 많은 설정 내용은 여기서 다룰수 없기 때문에 공식 문서에서 kafka를 검색하셔서 참고!

 
```
💡 참고  

여기서는 Producer/Consumer 설정을 application.yml에 작성했지만, bean을 통해 설정하는 방법도 있다.  

Producer, Consumer의 설정을 여러 개로 관리하고 싶다면 bean으로 구현하는 것도 좋은 방법.  
```



2) KafkaController.java  
post 방식으로 message 데이터를 받아서, Producer 서비스로 전달.  
``` java 
import com.victolee.kafkaexam.Service.KafkaProducer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping(value = "/kafka")
public class KafkaController {
    private final KafkaProducer producer;

    @Autowired
    KafkaController(KafkaProducer producer) {
        this.producer = producer;
    }

    @PostMapping
    public String sendMessage(@RequestParam("message") String message) {
        this.producer.sendMessage(message);

        return "success";
    }
}
```  

3) KafkaProducer.java
KafkaTemplate에 Topic명과 Message를 전달.  
KafkaTemplate.send() 메서드가 실행되면, Kafka 서버로 메시지가 전송.  


``` java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

@Service
public class KafkaProducer {
    private static final String TOPIC = "exam";
    private final KafkaTemplate<String, String> kafkaTemplate;

    @Autowired
    public KafkaProducer(KafkaTemplate kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void sendMessage(String message) {
        System.out.println(String.format("Produce message : %s", message));
        this.kafkaTemplate.send(TOPIC, message);
    }
}
```  



4) KafkaConsumer.java  
Kafka로부터 메시지를 받으려면 @KafkaListener 어노테이션을 달아주면 됨.  

``` java
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

import java.io.IOException;

@Service
public class KafkaConsumer {

    @KafkaListener(topics = "exam", groupId = "foo")
    public void consume(String message) throws IOException {
        System.out.println(String.format("Consumed message : %s", message));
    }
}
```  

 
03. 테스트
1) Springboot & Kafka연동 확인  
테스트를 해보기전에, Kafka가 잘 실행되고 있는지 확인을 해보는게 좋다.  

Springboot 애플리케이션 실행시 아래의 이미지처럼 커넥션 실패 로그가 계속 출력이 되면,  
Kafka 서버가 실행이 안됐다던지 등의 이유로 연동이 안되고 있는 상태이므로 우선적으로 해결이 필요.  
![image](https://user-images.githubusercontent.com/67637716/169437999-098b3a83-2a87-4484-b804-7f0df9563e60.png)  

애플리케이션이 실행되면 Kafka 서버와 커넥션이 이루어지는데, Cosunmer의 @KafkaListener에서 설정한 exam 토픽을 자동으로 생성하는 것을 확인할 수 있다.  

참고로 이는 broker의 설정과 관련이 있다.  
auto.create.topcis.enable이 설정되어 있으면 topci이 없을경우 topic을 자동으로 생성.  


2) 메시지 pub/sub  
Springboot와 Kafka 연동이 확인되었으므로 이제 메시지 pub/sub 테스트를 해보자.  

먼저 Kafka 컨테이너에서 exam 토픽에 메시지가 전송되었는지 확인한다.  
```
# kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic exam
```  
아직은 메시지 발송된 것이 없으니 아무것도 출력되고 있지 않는다.  


API를 call.  
응답은 success가 출력되었으니, 정상 실행된 것을 확인할수 있다.  



다음으로 Kafka 컨테이너의 topic 메시지를 확인.  
```
 # kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic exam
```  
 

메시지를 publish하면 로그를 남기도록 Producer에서 작성했으므로 Springboot에서도 로그를 확인한다.  
produce와 consume 메시지가 잘 출력되고 있다.  
![image](https://user-images.githubusercontent.com/67637716/169438211-228381aa-fd42-416c-bdf2-063484ea1dc7.png)  
