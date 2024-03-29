다른 메시징 시스템에서는 컨슈머가 데이터를 가져가게 되면 데이터가 사라지게 되는데,   
카프카에서는 커뉴머가 데이터를 가져가도 데이터가 사라지지 않는다.  

이와 같은 특징은 카프카를 데이터 파이프라인으로 운영하는데 매우 핵심적인 역할을 한다.  
<hr>  


데이터는 토픽 내부의 파티션에 저장되는데 kafka consumer는 파티션에 저장된 데이터를 가져온다.  
데이터를 가져오는 것을 폴링(polling)이라고 한다.  

## 컨슈머의 역할
* Topic의 partition으로 부터 데이터 polling
    * 메시지를 가져와서 특정 데이터베이스에 저장하거나 또 다른 파이프라인에 저장할 수 있다. 
* Partition offset 위치 기록(commit)
    * 오프셋이란 파티션에 있는 데이터의 번호를 뜻함
* Consumer group을 통해 병렬 처리
    * 파티션 개수에 따라 컨슈머를 여러개를 만들면 병렬처리가 가능하기 떄문에 더욱 빠른 속도로 데이터 처리 가능



```  java
package com.example.kafka;

import java.time.Duration;
import java.util.Arrays;
import java.util.Properties;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class KafkaTestApplication {

	public static void main(String[] args) {
		SpringApplication.run(KafkaTestApplication.class, args);

		Properties configs = new Properties();
		// 두 개 이상의 브로커 정보(ip, port)를 설정하도록 권장 HA를 위해
		configs.put("bootstrap.servers", "localhost:9092");
		// 그룹 아이디 지정
		// 컨슈머 그룹 , 컨슈머들의 묶음
		configs.put("group.id", "click_log_group");
		// key와 value에 대한 직렬화 설정
		configs.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
		configs.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

//		KafkaProducer<String, String> producer = new KafkaProducer<>(configs);
		// 카프카 컨슈머 인스턴스 만든다.
		// 데이터를 읽고 처리할 수 있다.
		KafkaConsumer<String, String> consumer = new KafkaConsumer<>(configs);

		// 컨슈머 그룹을 정하고 어느 카프카 브로커에서 데이터를 가지고 올지 선언 했으니
		// 어느 토픽을 대상으로 데이터를 가져올지 선언해야함
		// consume all partitions from topic
		consumer.subscribe(Arrays.asList("click_log"));

		// 특정 토픽의 전체 파티션이 아니라 일부 파티션의 데이터만 가지고 오고 싶다면
		// key가 존재하는 데이터라면, 이 방식을 통해 데이터의 순서를 보장하는 데이터 처리를 할 수 있다.
//		TopicPartition partition0 = new TopicPartition("topicName", 0); // topicname, partitionNum
//		TopicPartition partition1 = new TopicPartition("topicName", 1); // topicname, partitionNum
//		consumer.assign(Arrays.asList(partition0, partition1));
		
		// 폴링 루프 구문
		// poll()메서드가 포함된 무한 루프
		// consumer api의 핵심은 브로커로부터 연속적으로
		// 컨슈머가 허락하는 한 많은 데이터를 읽는 것
		// 폴링 루프는 컨슈머 api의 핵심 로직
		// poll method를 통해 데이터를 가져오는데, 설정한 시간동안 데이터를 기다리게 된다.
		while (true) {
			// poll(Duration)은 기존 poll(long)과는 다소 다르게 동작한다.
			// broker에 데이터를 가져오도록 요청하고 나서 duration timeout이 날때 까지 데이터가 브로커로부터 가져오지 못하면
			// poll(long)은 long시간 만큼 기다렸을 때 가져올 데이터가 없으면 무기한으로 기다리는 이슈
			// poll(Duration)은 즉시 빈 collection을 반환
			
			// recodes는 date의 묶음 list
			// 데이터를 처리할 떄는 가장 작은 단위인 record로 나누어 처리해야한다.
			ConsumerRecords<String, String> records = consumer.poll(500); // deprecated
			ConsumerRecords<String, String> records2 = consumer.poll(Duration.ofMillis(500));
			for (ConsumerRecord<String, String> record : records2) {
				System.out.println("topic::" + record.topic());
				System.out.println("value::" + record.value()); // 이전에 producer가 전송한 데이터
			}
		}
	}

}

```  


# 컨슈머가 Data를 전달받는 과정

![image](https://user-images.githubusercontent.com/67637716/200756157-90277b5f-b8af-45ef-994c-701217930e20.png)  
key = null인 경우, 2개의 파티션에 라운드로빈으로 데이터를 넣는다.  
이렇게 파티션에 데이터는 파티션 내에서 고유한 번호를 가지게 되는데 이것을 off-set이라고 부른다.  
![image](https://user-images.githubusercontent.com/67637716/200756357-d87e2160-faa6-4110-bdf9-2e29db936322.png)  
offset은 토픽별로 그리고 파티션별로 별개로 지정된다.  

offset은 `컨슈머가 데이터를 어느 지점까지 읽었는지 확인`하는 용도.  
![image](https://user-images.githubusercontent.com/67637716/200756796-b910a83f-0251-4cbc-aaf1-27693e2f4102.png)  
컨슈머가 데이터를 읽기 시작하면, offset을 commit하게 되는데 가져간 내용에 대한 정보는 `__consumer_offset` topic에 저장한다.  


컨슈머가 사고로 실행이 중지되었을때, __consumer_offset은 어느 파티션에 어떤 offset을 읽고 있었는지, 중지되었던 시점을 알고 있으므로  
시작위치부터 다시 복구하여 데이터 처리를 할 수 있다.  
컨슈머에 이슈가 발생하더라도 데이터의 처리시점을 복구할 수 있는 고가용성의 특징을 가지고 있다.  

## 컨슈머의 생성 개수

click_log라는 topic 1개와 파티션 2개라고 가정.  
#### 같은 컨슈머 그룹
1. consumer가 1개일 때  
![image](https://user-images.githubusercontent.com/67637716/200757575-2f8e2a2a-7415-4a7b-b585-0e5c74ff10ec.png)  
컨슈머가 1개일 때 두개의 파티션에서 데이터를 가져간다.  

2. consumer가 2개일 때  
![image](https://user-images.githubusercontent.com/67637716/200757695-29309fe1-4766-47d7-8226-b3c7130cfc4b.png)  
각 컨슈머가 각각의 파티션을 할당하여 데이터를 가져와서 처리  

3. consumer가 3개 이상일 때

이미 파티션들이 각 컨슈머에 할당되었기 때문에 더이상 할당될 파티션이 없어서 동작되지 않음.  
![image](https://user-images.githubusercontent.com/67637716/200757996-c38bb693-2f8b-430a-a176-64660f2b8781.png)  
여러 파티션을 가진 토픽에 대해서 컨슈머를 병렬처리하고 싶다면, 컨슈머를 파티션의 개수보다 적은 개수로 실행해야한다.  

#### 다른 컨슈머 그룹
각기 다른 컨슈머 그룹에 속한 컨슈머들은 다른 컨슈머 그룹에 영향을 끼치지 않는다.  
![image](https://user-images.githubusercontent.com/67637716/200758377-5ba4da21-33e6-44f8-be97-fcfd3662afb7.png)  

__consumer_offset 토픽에는 컨슈머 그룹별로 토픽별로 offset을 나누어 저장하기 때문이다.  

하나의 토픽으로 데이터는 다양한 역할을 하는 컨슈머들이 각자 원하는 데이터로 처리할 수 있다.  


## auto commit
```  
enable.auto.commit : 자동 오프셋 커밋 여부 , default: true
auto.commit.interval.ms : 자동 오프셋 커밋일 때 interval 시간, default 5초

configs.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, true);
configs.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, 60000);
```  

* 일정 간격, poll() 메서드 호출 시 자동 commit. commit 관련 코드를 작성할 필요 없어 편리하다.  
* 속도가 가장 빠름
* 중복 또는 유실이 발생 할 수 있음
	* server 장애로 인해 중단시, offset commit이 되지 않아, 데이터가 중복 또는 유실 될 수 있음.
	* 일부 데이터가 중복/유실되도 상관 없는 곳(GPS 등)에서 사용
	
![image](https://user-images.githubusercontent.com/67637716/201509603-1a324483-15eb-4ef4-82af-6f60c022af4c.png)  

EX : 결제를 했는데 2번 결재됨 등  

#### 오토 커밋을 사용하지 않는다
enable.auto.commit=false  

밑의 두가지 방법을 사용하여 commit을 제어해야한다.  
1) commitSync() : 동기 커밋  
2) commitAsync() : 비동기 커밋  

#### commitSync()
* ConsumerRecord 처리 순서 보장
* 가장 느림(커밋이 완료될 때까지 block)
* poll() 메서드로 반환된 ConsumerRecord의 마지막 offset을 커밋
* Map<TopicPartition, OffsetAndMetadata>을 통해 오프셋 지정 커밋 가능
	* 1개가 처리될때마다 1번씩 offset commit가능  
![image](https://user-images.githubusercontent.com/67637716/201509903-6b6e3796-b25e-4582-ab93-f2c9264a613f.png)  

``` java
while (true) {
	ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
	records.forEach(record -> {
		System.out.println(record.value());
	});

	try {
		consumer.commitSync();
	}catch(CommitFailedException e) {
		System.err.println("commit failed");
	}
}

//// offset 지정 커밋
while (true) {
	ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
	Map<TopicPartition,	OffsetAndMetadata> currentOffset = new HashMap<>();
	records.forEach(record -> {
		currentOffset.put(new TopicPartition(record.topic(), record.partition()),
				new OffsetAndMetadata(record.offset() + 1, null));
		try {
		consumer.commitSync(currentOffset);
		}catch(CommitFailedException e) {
			System.err.println("commit failed");
		}
		System.out.println(record.value());
	});
}
```  



#### commitAsync()
* 동기 커밋보다 빠름
	* 커밋을 요청하는 시간동안 polling을 기다리지 않음
* 중복이 발생할 수 있음
	* 일시적인 통신 문제로 이전 offset보다 이후 offset이 먼저 커밋 될때
* consumerRecord 처리 순서 보장 x
	* 처리 순서가 중요한 서비스(주문, 재고관리 등)에서는 사용 제한


``` java
while (true) {
	ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
	records.forEach(record -> {
		  System.out.println(record.value());
	});

	consumer.commitAsync();
}
// callback을 통해 offset commit동 가능해 보임.  

```  

## 리밸런스
컨슈머 그룹의 파티션 소유권이 변경될 때 일어나는 현상  
* 리밸런스를 하는동안 일시적으로 메시지를 가져올 수 없음
* 리밸런스 발생시 데이터 유실/중복 발생 가능성 있음
	* 리밸런스 리스너 - commitSync() 또는 추가적인 방법(unique key)으로 데이터 유실/중복 방지
* consumer.close() 호출시 또는 consumer의 세션이 끊어졌을 때 발생한다.

![image](https://user-images.githubusercontent.com/67637716/201510597-57e017d8-b7ac-4d8b-8d56-e4327226e103.png)  

session timeout 시간을 보통 hearbeat 시간 * 3 으로 한다고함.  


#### Consumer rebalance listener  
![image](https://user-images.githubusercontent.com/67637716/201510659-4b365203-707b-4b2c-8315-e41ddefd270e.png)  

```  java
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(configs);

consumer.subscribe(Arrays.asList(TOPIC_NAME), new ConsumerRebalanceListener() {

	@Override
	public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
		// 파티션이 끊어졌을때
		// offset commit 
		consumer.commitSync(currentOffsets); 
	
		// consumer가 많을 때 리밸런스 시간 측정을 통한 컨슈머 모니터링 가능
	}

	@Override
	public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
		// 파티션을 새로 할당 받았을 때 
	}

});
```  

## consumer wakeup
consumer를 정상적으로 종료할 때 사용.  
polling할때 wakeup exception발생해서 정상적으로 종료할 수 있다.  

``` java
 while (true) {
  ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
  for (ConsumerRecord<String, String> record : records) {
      System.out.println(record.value());
  }
  try {
			consumer.commitSync(currentOffset);
		} catch (CommitFailedException e) {
			System.err.println("commit failed");
		}
}
```  

#### SIGKILL(강제 중단)으로 인한 중복 처리 발생 예시  
1) poll 호출
	- 마지막 커밋된 오프셋이 100
	- records 100개 반환 : 오프셋 101 ~ 200
	- ![image](https://user-images.githubusercontent.com/67637716/201511374-b89a45d6-6869-4e50-80f9-31e60ddcce5d.png)  
2) records loop 구문 수행
3) record.value()  150번 오프셋 print 중 SIGKILL 호출
	- 101번 ~ 150번 오프셋 처리완료, 151 ~ 200 미처리
	- ![image](https://user-images.githubusercontent.com/67637716/201511411-09d95556-7c6d-4422-94f0-c4bbef671c2c.png)  
4) offset 200 커밋 불가
	- 브로커에는 100번 오프셋이 마지막 커밋
	- 컨슈머 재시작시 다시 오프셋 101부터 처리 시작

#### wakeup()을 통한 graceful shutdown 필수!
* SIGTERM을 통한 shutdown signal로 kill하여 처리한 데이터 커밋 필요
* SIGKILL(9)는 프로세스 강제 종료로 커밋 불가 -> 중복/유실 발생

안정적으로 종료가능.  
리밸런스가 발생한 것을 브로커에게 명시적으로 전달할 수 있다.  


``` java

Runtime.getRuntime().addShutdownHook(new Thread() {
	@Override
	public void run() {
		consumer.wakeup();
	}
});

try {
	while (true) {
		ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
		for (ConsumerRecord<String, String> record : records) {
			System.out.println(record.value());
		}
		try {
			consumer.commitSync(currentOffset);
		} catch (CommitFailedException e) {
			System.err.println("commit failed");
		}
	}
} catch (WakeupException e) {
	System.err.println("poll() method trigger wakeupException");
} finally {
	consumer.commitSync();
	consumer.close();
}
```  











 





