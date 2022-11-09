```  java
public static void main(String[] args) {
		SpringApplication.run(KafkaTestApplication.class, args);

		Properties configs = new Properties();
		// 두 개 이상의 브로커 정보(ip, port)를 설정하도록 권장 HA를 위해
		configs.put("bootstrap.servers", "localhost:9092");
		// Byte array, String, Integer Serializer를 사용할 수 있다.
		// key는 메시지를 보내면 토픽의 파티션이 지정될 때 사용
		configs.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
		configs.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

		// 카프카 프로듀서의 인스턴스 만든다.
		KafkaProducer<String, String> producer = new KafkaProducer<>(configs);
		// 전송할 객체
		// 파라미터 개수의 따라 topic, key, value설정 가능
		// key를 특정한 hash값으로 변환시켜 파티션과 1:1 매칭을 하게 됨 -> 따라서 같은 파티션에만 value가 들어가게됨
		// 하지만 토픽에 파티션을 추가하는 순간 key-파티션의 일관성을 보장되지않음. 따라서 키를 생성하고나선 파티션을 추가하지 않는걸 권장
		// 여기선 topic과 value
		ProducerRecord<String, String> recode = new ProducerRecord<String, String>("click_log", "login");

		producer.send(recode);

		producer.close();
	}

```  
