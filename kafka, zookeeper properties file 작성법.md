# zookeeper ( zookeeper.properties )
``` ruby
# Licensed to the Apache Software Foundation (ASF) under one or more  
# contributor license agreements.  See the NOTICE file distributed with  
# this work for additional information regarding copyright ownership.  
# The ASF licenses this file to You under the Apache License, Version 2.0  
# (the "License"); you may not use this file except in compliance with  
# the License.  You may obtain a copy of the License at  
#   
#    http://www.apache.org/licenses/LICENSE-2.0  
#   
# Unless required by applicable law or agreed to in writing, software  
# distributed under the License is distributed on an "AS IS" BASIS,  
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
# See the License for the specific language governing permissions and  
# limitations under the License.  
///////////// 라이센스 관련 내용 //////////////


# snapshot 데이터를 저장할 경로를 지정
dataDir=/tmp/zookeeper  
# 클라이언트가 connect할 port 번호 지정
clientPort=2181  
# 하나의 클라이언트에서 동시 접속하는 개수 제한, 기본값은 60이며, 0은 무제한
maxClientCnxns=0  
# port 충돌을 방지하려면 admin server 비활성화(false)
admin.enableServer=false  
# admin.serverPort=8080  

```   


# Kafka ( server.properties )
``` ruby
# config/server.properties
 
############################# Server Basics #############################
 
# Broker의 ID로 Cluster내 Broker를 구분하기 위해 사용(Unique 값)
broker.id=0
 
############################# Socket Server Settings #############################
 
# Broker가 사용하는 호스트와 포트를 지정, 형식은 PLAINTEXT://your.host.name:port 을 사용
listeners=PLAINTEXT://:9092
 
# Producer와 Consumer가 접근할 호스트와 포트를 지정, 기본값은 listeners를 사용
advertised.listeners=PLAINTEXT://localhost:9092
 
 
# 네트워크 요청을 처리하는 Thread의 개수, 기본값 3
num.network.threads=3
 
# I/O가 생길때 마다 생성되는 Thread의 개수, 기본값 8
num.io.threads=8
 
# socket 서버가 사용하는 송수신 버퍼 (SO_SNDBUF, SO_RCVBUF) 사이즈, 기본값 102400
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
 
# 서버가 받을 수 있는 최대 요청 사이즈이며, 서버 메모리가 고갈 되는 것 방지
# JAVA의 Heap 보다 작게 설정해야 함, 기본값 104857600
socket.request.max.bytes=104857600
 
############################# Log Basics #############################
 
# 로그 파일을 저장할 디렉터리의 쉼표로 구분할 수 있음
log.dirs=C:/dev/kafka_2.13-2.6.0/logs
 
# 토픽당 파티션의 수를 의미, 
# 입력한 수만큼 병렬처리 가능, 데이터 파일도 그만큼 늘어남
num.partitions=1
 
# 시작 시 log 복구 및 종료 시 flushing에 사용할 데이터 directory당 Thread 개수
# 이 값은 RAID 배열에 데이터 directory에 대해 증가하도록 권장 됨
num.recovery.threads.per.data.dir=1
 
############################# Internal Topic Settings #############################
# 내부 Topic인 "_consumer_offsets", "_transaction_state"에 대한 replication factor
# 개발환경 : 1, 운영할 경우 가용성 보장을 위해 1 이상 권장(3 정도)
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
 
############################# Log Retention Policy #############################
 
# 세그먼트 파일의 삭제 주기, 기본값 hours, 168시간(7일)
# 옵션 [ bytes, ms, minutes, hours ] 
log.retention.hours=168
 
# 토픽별로 수집한 데이터를 보관하는 파일
# 세그먼트 파일의 최대 크기, 기본값 1GB
# 세그먼트 파일의 용량이 차면 새로운 파일을 생성
log.segment.bytes=1073741824
 
# 세그먼트 파일의 삭제 여부를 체크하는 주기, 기본값 5분(보존 정책)
log.retention.check.interval.ms=300000
 
############################# Zookeeper #############################
 
# 주키퍼의 접속 정보
# 쉼표(,)로 많은 연결 서버 포트 설정 가능
# 모든 kafka znode의 Root directory
zookeeper.connect=localhost:2181
 
# 주키퍼 접속 시도 제한시간(time out)
zookeeper.connection.timeout.ms=18000
 
 
############################# Group Coordinator Settings #############################
 
# GroupCoordinator 설정 - 컨슈머 rebalance를 지연시키는 시간
# 개발환경 : 테스트 편리를 위해 0으로 정의
# 운영환경 : 3초의 기본값을 설정하는게 좋음
 group.initial.rebalance.delay.ms=0

```
