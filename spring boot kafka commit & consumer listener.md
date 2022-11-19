https://docs.spring.io/spring-kafka/reference/html/#kafka-listener-annotation  


# @KafkaListener
![image](https://user-images.githubusercontent.com/67637716/202839722-6b6c6381-77c3-41d5-89bb-ba955507c13f.png)  

기본적으로 @KafkaListener를 붙이면 producer가 해당 topic에 message를 보내면  
Consumer가 해당 topic을 subscribe하여 topic을 받아올 수 있다.  

이때, kafkaListenerContainerFactory(단일 thread) OR ConcurrentMessageListenerContainer(multi thread)를 BEAN으로 등록 해주어야 한다.(등록하지 않으면 error)  
``` java
    public ConsumerFactory<String, ChatMessageDTO> consumerFactory() {
        // com.teamride.messenger.server.dto.ChatMessageDTO;
        // package com.teamride.messenger.client.dto.ChatMessageDTO;
        // 경로가 다르기 때문에 역직렬화 시 같은 Entity로 인식을 못함
        // package 경로까지 보기 떄문인데 addTrustedPackages를 해줌으로서 해결
        JsonDeserializer<ChatMessageDTO> deserializer = new JsonDeserializer<>(ChatMessageDTO.class);
        deserializer.setRemoveTypeHeaders(false);
        deserializer.addTrustedPackages("*");
        deserializer.setUseTypeMapperForKey(true);
        
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, KafkaConstants.KAFKA_BROKER);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, KafkaConstants.GROUP_ID);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, deserializer);
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
        return new DefaultKafkaConsumerFactory<>(props, new StringDeserializer(), deserializer);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, ChatMessageDTO> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, ChatMessageDTO> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(3); // thread 수
        System.out.println("default ack mode(batch):"+factory.getContainerProperties().getAckMode());
        factory.getContainerProperties().setAckMode(AckMode.MANUAL_IMMEDIATE); // offset 수동 커밋을 위함
        
        return factory;
    }
```  

여러가지 설정을 하고 ConcurrentKafkaListenerContainerFactory를 Bean으로 만든다.  

이 때 auto commit과 관련된 설정을 할 수 있다.  
enable.auto.commit은 default false 이다.  

ConcurrentKafkaListenerContainerFactory에서 여러가지 설정을  추가해 줄수 있는데,  
ContainerProperties를 얻어와서 ackMode를 설정해 줄 수 있다.  

![image](https://user-images.githubusercontent.com/67637716/202840026-b90484cd-c950-4404-a1bc-8d64beb349a1.png)  

``` java
@KafkaListener(topics = KafkaConstants.CHAT_CLIENT, groupId = KafkaConstants.GROUP_ID)
	public void listen(ChatMessageDTO message, Acknowledgment ack) {
		log.info("Received Msg chat-client " + message);
		ack.acknowledge();
  }
```  

ack mode를 MANUAL_IMMEDIATE로 설정한 후, 위와 같이 사용할 수 있다.  

acknowledge()가 궁금하여 source를 따라가 보았다.  

![image](https://user-images.githubusercontent.com/67637716/202840094-8daa871a-0a1c-4131-90bd-454abe77fd35.png)  

결국 record를 하나하나 commit해주는 java code가 나오는 것을 알 수 있었다.  




