# Apache Kafka: Triển Khai, Cấu Hình & Quản Trị Toàn Diện

## Mục lục
- [1. Tổng quan về Apache Kafka](#1-tổng-quan-về-apache-kafka)
  - [1.1 Kafka là gì?](#11-kafka-là-gì)
  - [1.2 Kiến trúc Kafka](#12-kiến-trúc-kafka)
  - [1.3 Các thành phần chính](#13-các-thành-phần-chính)
  - [1.4 Use cases phổ biến](#14-use-cases-phổ-biến)
- [2. Cài đặt và cấu hình cơ bản](#2-cài-đặt-và-cấu-hình-cơ-bản)
  - [2.1 Yêu cầu hệ thống](#21-yêu-cầu-hệ-thống)
  - [2.2 Cài đặt Kafka](#22-cài-đặt-kafka)
  - [2.3 Cài đặt ZooKeeper](#23-cài-đặt-zookeeper)
  - [2.4 Cấu hình Kafka Server](#24-cấu-hình-kafka-server)
  - [2.5 Khởi động và dừng Kafka](#25-khởi-động-và-dừng-kafka)
- [3. Triển khai Kafka Cluster](#3-triển-khai-kafka-cluster)
  - [3.1 Cấu hình Multi-broker](#31-cấu-hình-multi-broker)
  - [3.2 Replication Factor](#32-replication-factor)
  - [3.3 Partition Strategy](#33-partition-strategy)
  - [3.4 High Availability Setup](#34-high-availability-setup)
- [4. Quản lý Topics và Partitions](#4-quản-lý-topics-và-partitions)
  - [4.1 Tạo và xóa Topics](#41-tạo-và-xóa-topics)
  - [4.2 Quản lý Partitions](#42-quản-lý-partitions)
  - [4.3 Rebalance Partitions](#43-rebalance-partitions)
  - [4.4 Topic Configs](#44-topic-configs)
- [5. Producers và Consumers](#5-producers-và-consumers)
  - [5.1 Producer Configurations](#51-producer-configurations)
  - [5.2 Consumer Configurations](#52-consumer-configurations)
  - [5.3 Consumer Groups](#53-consumer-groups)
  - [5.4 Message Delivery Semantics](#54-message-delivery-semantics)
- [6. Kafka Connect](#6-kafka-connect)
  - [6.1 Cài đặt Kafka Connect](#61-cài-đặt-kafka-connect)
  - [6.2 Source Connectors](#62-source-connectors)
  - [6.3 Sink Connectors](#63-sink-connectors)
  - [6.4 Transformations](#64-transformations)
- [7. Kafka Streams](#7-kafka-streams)
  - [7.1 Kiến trúc Streams](#71-kiến-trúc-streams)
  - [7.2 Streams DSL](#72-streams-dsl)
  - [7.3 State Stores](#73-state-stores)
  - [7.4 Interactive Queries](#74-interactive-queries)
- [8. Monitoring và Operations](#8-monitoring-và-operations)
  - [8.1 Kafka Metrics](#81-kafka-metrics)
  - [8.2 JMX Monitoring](#82-jmx-monitoring)
  - [8.3 Prometheus và Grafana](#83-prometheus-và-grafana)
  - [8.4 Log Files](#84-log-files)
- [9. Bảo mật Kafka](#9-bảo-mật-kafka)
  - [9.1 Authentication](#91-authentication)
  - [9.2 Authorization](#92-authorization)
  - [9.3 Encryption](#93-encryption)
  - [9.4 Audit](#94-audit)
- [10. Performance Tuning](#10-performance-tuning)
  - [10.1 Broker Tuning](#101-broker-tuning)
  - [10.2 Producer Tuning](#102-producer-tuning)
  - [10.3 Consumer Tuning](#103-consumer-tuning)
  - [10.4 JVM Tuning](#104-jvm-tuning)
  - [10.5 OS Tuning](#105-os-tuning)
- [11. Troubleshooting](#11-troubleshooting)
  - [11.1 Common Issues](#111-common-issues)
  - [11.2 Partition Reassignment](#112-partition-reassignment)
  - [11.3 Recovery Scenarios](#113-recovery-scenarios)
  - [11.4 Debugging Tools](#114-debugging-tools)
- [12. Kafka trong Cloud và Containers](#12-kafka-trong-cloud-và-containers)
  - [12.1 Kafka trên Kubernetes](#121-kafka-trên-kubernetes)
  - [12.2 Kafka trên AWS](#122-kafka-trên-aws)
  - [12.3 Kafka trên GCP](#123-kafka-trên-gcp)
  - [12.4 Managed Kafka Services](#124-managed-kafka-services)

## 1. Tổng quan về Apache Kafka

### 1.1 Kafka là gì?

Apache Kafka là một nền tảng streaming phân tán, được thiết kế để xử lý luồng dữ liệu theo thời gian thực. Kafka được phát triển ban đầu tại LinkedIn và sau đó trở thành một dự án mã nguồn mở của Apache Foundation. Các đặc điểm chính của Kafka bao gồm:

- **Distributed**: Kafka chạy như một cluster trên một hay nhiều server
- **Scalable**: Dễ dàng mở rộng mà không cần downtime
- **Fault-Tolerant**: Chống chịu lỗi khi các node bị lỗi
- **High-Throughput**: Xử lý hàng triệu message mỗi giây
- **Low-Latency**: Độ trễ rất thấp, thường chỉ vài millisecond
- **Durable**: Lưu trữ dữ liệu an toàn với replication

Kafka đặc biệt phù hợp cho các use cases liên quan đến xử lý dữ liệu streaming, log aggregation, metrics collection, event sourcing, và các ứng dụng real-time.

### 1.2 Kiến trúc Kafka

Kiến trúc Kafka bao gồm các thành phần chính sau:

![Kafka Architecture](https://i.imgur.com/YQsGnqJ.png)

**Logical View**:
- **Topics**: Luồng dữ liệu được phân loại theo chủ đề
- **Partitions**: Mỗi topic được chia thành nhiều partitions
- **Offsets**: Mỗi message trong partition có một offset duy nhất
- **Consumer Groups**: Groups của các consumer để phân phối việc xử lý

**Physical View**:
- **Brokers**: Các server Kafka đảm nhiệm lưu trữ và xử lý requests
- **ZooKeeper**: Quản lý metadata và coordination (trong các phiên bản cũ)
- **Producers**: Ứng dụng gửi messages vào Kafka
- **Consumers**: Ứng dụng đọc messages từ Kafka

### 1.3 Các thành phần chính

#### Broker
- Server chạy Kafka process
- Quản lý partitions và xử lý requests từ clients
- Mỗi broker được định danh bởi một ID duy nhất
- Một Kafka cluster bao gồm nhiều brokers

#### Topic
- Channel logic để lưu trữ và phân phối messages
- Có thể có nhiều producers và consumers cho một topic
- Messages được lưu trữ trong các partitions của topic

#### Partition
- Mỗi topic được chia thành nhiều partitions
- Partitions cho phép parallelism trong quá trình xử lý
- Mỗi partition là một log được lưu trên disk
- Messages trong partition được đánh số theo offset (0, 1, 2,...)

#### Producer
- Ứng dụng gửi data vào Kafka
- Có thể lựa chọn gửi message tới specific partition
- Hỗ trợ nhiều modes: fire-and-forget, synchronous, asynchronous

#### Consumer
- Ứng dụng đọc data từ Kafka
- Mỗi consumer theo dõi offset của mình để biết đã đọc đến đâu
- Consumers có thể thuộc các consumer groups

#### Consumer Group
- Tập hợp các consumers làm việc cùng nhau
- Mỗi partition chỉ được consumed bởi một consumer trong group
- Cho phép horizontal scaling của việc consume

#### ZooKeeper (hoặc KRaft từ Kafka 2.8+)
- Quản lý metadata về Kafka cluster
- Theo dõi broker status, partitions, topics,...
- Từ Kafka 2.8+, KRaft mode cho phép Kafka hoạt động mà không cần ZooKeeper

### 1.4 Use cases phổ biến

#### Messaging System
- Thay thế các message queue truyền thống
- Hỗ trợ publish-subscribe và queue-based messaging patterns
- Ưu điểm: throughput cao, replication, fault-tolerance

#### Log Aggregation
- Tập trung logs từ nhiều applications và servers
- Chuẩn hóa và lưu trữ logs
- Phân phối logs tới các destinations khác nhau

#### Stream Processing
- Xử lý dữ liệu streaming theo thời gian thực
- Kết hợp với Kafka Streams, Apache Flink, Spark Streaming,...
- Event-driven microservices

#### Metrics và Monitoring
- Thu thập metrics từ nhiều services
- Lưu trữ và phân phối metrics đến các hệ thống monitoring
- Time-series analytics

#### Event Sourcing
- Lưu trữ tất cả các state changes dưới dạng event sequence
- Rebuild state từ event log
- CQRS (Command Query Responsibility Segregation)

#### Data Integration
- ETL (Extract, Transform, Load) processes
- Change Data Capture (CDC) từ databases
- Kết nối các hệ thống dữ liệu khác nhau thông qua Kafka Connect

## 2. Cài đặt và cấu hình cơ bản

### 2.1 Yêu cầu hệ thống

**Hardware Requirements**:
- **CPU**: Ít nhất 4 cores cho môi trường production
- **Memory**: Tối thiểu 8GB RAM, khuyến nghị 16GB+ cho production
- **Disk**: SSD cho hiệu suất tốt nhất, ít nhất 500GB cho production
- **Network**: 1Gbps+ cho production clusters

**Software Requirements**:
- **Java**: JDK 8+ (Kafka 3.x yêu cầu Java 11+)
- **Operating System**: Linux là nền tảng được hỗ trợ tốt nhất
- **ZooKeeper**: Bắt buộc cho Kafka < 3.0, tùy chọn cho Kafka 3.0+ (KRaft mode)

**Sizing Guidelines**:
- Số lượng brokers phụ thuộc vào throughput và reliability requirements
- 3 brokers là minimum để đảm bảo high availability
- 5-7 brokers cho các workloads lớn hơn

### 2.2 Cài đặt Kafka

#### Tải và giải nén Kafka
```bash
# Tải Kafka
wget https://downloads.apache.org/kafka/3.4.0/kafka_2.13-3.4.0.tgz

# Giải nén
tar -xzf kafka_2.13-3.4.0.tgz
cd kafka_2.13-3.4.0
```

#### Cài đặt từ package manager
**Debian/Ubuntu**:
```bash
# Cài đặt Java
apt-get update
apt-get install -y default-jre

# Kafka thường được cài đặt từ binary distribution, không phải từ package manager
```

**RHEL/CentOS**:
```bash
# Cài đặt Java
yum install -y java-11-openjdk

# Kafka thường được cài đặt từ binary distribution, không phải từ package manager
```

#### Tạo systemd service
```bash
# Tạo file service cho ZooKeeper
cat > /etc/systemd/system/zookeeper.service << EOF
[Unit]
Description=Apache ZooKeeper server
Documentation=http://zookeeper.apache.org
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=simple
ExecStart=/path/to/kafka/bin/zookeeper-server-start.sh /path/to/kafka/config/zookeeper.properties
ExecStop=/path/to/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
EOF

# Tạo file service cho Kafka
cat > /etc/systemd/system/kafka.service << EOF
[Unit]
Description=Apache Kafka server
Documentation=http://kafka.apache.org
Requires=zookeeper.service
After=zookeeper.service

[Service]
Type=simple
ExecStart=/path/to/kafka/bin/kafka-server-start.sh /path/to/kafka/config/server.properties
ExecStop=/path/to/kafka/bin/kafka-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
EOF

# Reload systemd
systemctl daemon-reload

# Enable services to start on boot
systemctl enable zookeeper
systemctl enable kafka
```

### 2.3 Cài đặt ZooKeeper

Kafka < 3.0 yêu cầu ZooKeeper để hoạt động. Từ Kafka 3.0, bạn có thể sử dụng KRaft mode để loại bỏ phụ thuộc vào ZooKeeper, nhưng ZooKeeper vẫn là lựa chọn phổ biến cho production.

#### ZooKeeper Standalone Mode
Cấu hình ZooKeeper trong file `config/zookeeper.properties`:
```properties
# ZooKeeper data directory
dataDir=/var/lib/zookeeper
# Client port
clientPort=2181
# Maximum number of client connections
maxClientCnxns=60
# Disable admin server
admin.enableServer=false
```

#### ZooKeeper Ensemble (Cluster) Mode
Để đảm bảo high availability, cấu hình ZooKeeper ensemble với ít nhất 3 nodes:

Cho node 1 (server.properties):
```properties
dataDir=/var/lib/zookeeper
clientPort=2181
maxClientCnxns=60
initLimit=5
syncLimit=2
server.1=zookeeper1:2888:3888
server.2=zookeeper2:2888:3888
server.3=zookeeper3:2888:3888
```

Tạo file myid trong dataDir:
```bash
# Trên node 1
echo "1" > /var/lib/zookeeper/myid

# Trên node 2
echo "2" > /var/lib/zookeeper/myid

# Trên node 3
echo "3" > /var/lib/zookeeper/myid
```

### 2.4 Cấu hình Kafka Server

Cấu hình Kafka trong file `config/server.properties`:

#### Cấu hình cơ bản
```properties
# Broker ID - phải là unique trong cluster
broker.id=0

# Listener configurations
listeners=PLAINTEXT://hostname:9092
advertised.listeners=PLAINTEXT://public-hostname:9092

# ZooKeeper connection
zookeeper.connect=zookeeper1:2181,zookeeper2:2181,zookeeper3:2181

# Log directory
log.dirs=/var/lib/kafka/logs

# Default number of partitions
num.partitions=3

# Default replication factor
default.replication.factor=3

# Min in-sync replicas
min.insync.replicas=2

# Enable auto topic creation
auto.create.topics.enable=false

# Enable topic deletion
delete.topic.enable=true
```

#### KRaft Mode (Kafka 3.0+)
Để sử dụng Kafka không cần ZooKeeper (KRaft mode):

```properties
# Enable KRaft
process.roles=broker,controller

# Controller quorum voters
controller.quorum.voters=1@broker1:9093,2@broker2:9093,3@broker3:9093

# Node ID
node.id=1

# Controller listener
controller.listener.names=CONTROLLER
listeners=PLAINTEXT://hostname:9092,CONTROLLER://hostname:9093
listener.security.protocol.map=PLAINTEXT:PLAINTEXT,CONTROLLER:PLAINTEXT
inter.broker.listener.name=PLAINTEXT

# Log directory
log.dirs=/var/lib/kafka/logs
```

### 2.5 Khởi động và dừng Kafka

#### Using shell scripts
```bash
# Start ZooKeeper
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties

# Start Kafka
bin/kafka-server-start.sh -daemon config/server.properties

# Stop Kafka
bin/kafka-server-stop.sh

# Stop ZooKeeper
bin/zookeeper-server-stop.sh
```

#### Using systemd
```bash
# Start ZooKeeper
systemctl start zookeeper

# Start Kafka
systemctl start kafka

# Stop Kafka
systemctl stop kafka

# Stop ZooKeeper
systemctl stop zookeeper

# Check status
systemctl status kafka
systemctl status zookeeper
```

#### Validating Installation
```bash
# Create test topic
bin/kafka-topics.sh --create --topic test --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1

# List topics
bin/kafka-topics.sh --list --bootstrap-server localhost:9092

# Produce test messages
bin/kafka-console-producer.sh --topic test --bootstrap-server localhost:9092
> Hello Kafka
> This is a test message
> ^C

# Consume test messages
bin/kafka-console-consumer.sh --topic test --bootstrap-server localhost:9092 --from-beginning
```

## 3. Triển khai Kafka Cluster

### 3.1 Cấu hình Multi-broker

Để cài đặt Kafka cluster với nhiều brokers, bạn cần cấu hình mỗi broker với ID duy nhất và networking phù hợp.

#### Cấu hình cho Broker 1 (server-1.properties)
```properties
broker.id=1
listeners=PLAINTEXT://broker1:9092
advertised.listeners=PLAINTEXT://broker1:9092
log.dirs=/var/lib/kafka-1/logs
zookeeper.connect=zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
```

#### Cấu hình cho Broker 2 (server-2.properties)
```properties
broker.id=2
listeners=PLAINTEXT://broker2:9092
advertised.listeners=PLAINTEXT://broker2:9092
log.dirs=/var/lib/kafka-2/logs
zookeeper.connect=zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
```

#### Cấu hình cho Broker 3 (server-3.properties)
```properties
broker.id=3
listeners=PLAINTEXT://broker3:9092
advertised.listeners=PLAINTEXT://broker3:9092
log.dirs=/var/lib/kafka-3/logs
zookeeper.connect=zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
```

#### Các tham số cluster quan trọng
```properties
# Auto leader rebalance
auto.leader.rebalance.enable=true

# Controller socket timeout
controller.socket.timeout.ms=30000

# Controller message queue size
controller.message.queue.size=10000

# Replica fetch max bytes
replica.fetch.max.bytes=1048576

# Replica lag time max
replica.lag.time.max.ms=30000

# Unclean leader election
unclean.leader.election.enable=false
```

### 3.2 Replication Factor

Replication factor xác định số lượng bản sao của mỗi partition được lưu trữ trong cluster. Replication giúp đảm bảo fault tolerance.

#### Cấu hình Replication
```properties
# Default replication factor cho topics mới
default.replication.factor=3

# Số lượng replicas tối thiểu cần sync để ghi thành công
min.insync.replicas=2
```

#### Tạo topic với replication factor
```bash
bin/kafka-topics.sh --create --topic my-replicated-topic \
  --bootstrap-server broker1:9092 \
  --partitions 3 \
  --replication-factor 3
```

#### Kiểm tra trạng thái replication
```bash
bin/kafka-topics.sh --describe --topic my-replicated-topic \
  --bootstrap-server broker1:9092
```

Output sẽ hiển thị thông tin về partitions, replicas, và ISR (In-Sync Replicas):
```
Topic: my-replicated-topic   Partition: 0    Leader: 1   Replicas: 1,2,3 Isr: 1,2,3
Topic: my-replicated-topic   Partition: 1    Leader: 2   Replicas: 2,3,1 Isr: 2,3,1
Topic: my-replicated-topic   Partition: 2    Leader: 3   Replicas: 3,1,2 Isr: 3,1,2
```

### 3.3 Partition Strategy

Partitions cho phép Kafka phân tán dữ liệu và song song hóa xử lý. Các chiến lược partition hiệu quả rất quan trọng cho hiệu suất.

#### Quyết định số lượng partitions
Số lượng partitions phụ thuộc vào:
- Throughput mong muốn
- Số lượng consumers trong một consumer group
- Broker resources (CPU, memory, disk I/O)

**Công thức thông dụng**:
```
Số partitions = max(Throughput ÷ Consumer throughput, Consumer threads)
```

#### Phân phối Partitions
Kafka phân phối partitions đều trên các brokers, nhưng cũng cần cân nhắc:
- Rack awareness (broker.rack property)
- Disk space utilization
- Network traffic
- Leader balancing

#### Partition Assignments
Producers quyết định gửi message đến partition nào dựa trên:
- Chỉ định partition cụ thể
- Sử dụng key (messages cùng key sẽ vào cùng partition)
- Round-robin nếu không có partition cụ thể hoặc key

```java
// Producer với custom partitioner
Properties props = new Properties();
props.put("partitioner.class", "com.example.MyCustomPartitioner");
Producer<String, String> producer = new KafkaProducer<>(props);
```

### 3.4 High Availability Setup

Để đảm bảo high availability cho Kafka cluster, cần triển khai nhiều brokers và cấu hình phù hợp.

#### Rack Awareness
Rack awareness giúp Kafka phân tán replicas qua các physical racks để tăng fault tolerance:

```properties
# Chỉ định rack cho broker
broker.rack=rack1
```

Sau đó triển khai các brokers khác nhau trên các racks khác nhau:
- Broker 1: rack1
- Broker 2: rack2
- Broker 3: rack3

#### Multiple Data Centers
Để đảm bảo DR (Disaster Recovery), bạn có thể:
- Sử dụng MirrorMaker 2 để replication cross-DC
- Triển khai Kafka Connect với Replicator
- Sử dụng Confluent Multi-Region Clusters

#### MirrorMaker 2 Setup
```bash
# Tạo file connect-mirror-maker.properties
cat > connect-mirror-maker.properties << EOF
# Source cluster
source.cluster.alias=primary
source.cluster.bootstrap.servers=primary-kafka1:9092,primary-kafka2:9092

# Target cluster
target.cluster.alias=backup
target.cluster.bootstrap.servers=backup-kafka1:9092,backup-kafka2:9092

# Enable topic creation in target cluster
topics.blacklist=_.*
auto.create.topics.enable=true
EOF

# Chạy MirrorMaker 2
bin/connect-mirror-maker.sh connect-mirror-maker.properties
```

#### Multi-broker Monitoring
Theo dõi tình trạng của brokers:
```bash
# Kiểm tra các brokers trong cluster
bin/zookeeper-shell.sh zookeeper1:2181 ls /brokers/ids

# Kiểm tra controller
bin/zookeeper-shell.sh zookeeper1:2181 get /controller
```

## 4. Quản lý Topics và Partitions

### 4.1 Tạo và xóa Topics

Topics là đơn vị logical cho việc lưu trữ và tổ chức messages trong Kafka.

#### Tạo Topic mới
```bash
bin/kafka-topics.sh --create \
  --bootstrap-server broker1:9092 \
  --topic my-topic \
  --partitions 8 \
  --replication-factor 3 \
  --config cleanup.policy=compact \
  --config retention.ms=604800000
```

#### Xóa Topic
```bash
# Đảm bảo delete.topic.enable=true trong server.properties
bin/kafka-topics.sh --delete \
  --bootstrap-server broker1:9092 \
  --topic my-topic
```

#### Liệt kê Topics
```bash
# Liệt kê tất cả topics
bin/kafka-topics.sh --list \
  --bootstrap-server broker1:9092
```

#### Xem chi tiết Topic
```bash
bin/kafka-topics.sh --describe \
  --bootstrap-server broker1:9092 \
  --topic my-topic
```

### 4.2 Quản lý Partitions

Partitions là cách Kafka phân chia dữ liệu của topics để đạt được scalability và parallel processing.

#### Thêm Partitions
```bash
bin/kafka-topics.sh --alter \
  --bootstrap-server broker1:9092 \
  --topic my-topic \
  --partitions 16
```

**Lưu ý**: Bạn chỉ có thể tăng số lượng partitions, không thể giảm.

#### Partition Assignment
Xem partition assignment hiện tại:
```bash
bin/kafka-topics.sh --describe \
  --bootstrap-server broker1:9092 \
  --topic my-topic
```

#### Manually Reassign Partitions
Tạo reassignment file:
```json
{
  "version":1,
  "partitions": [
    {"topic":"my-topic","partition":0,"replicas":[1,2,3]},
    {"topic":"my-topic","partition":1,"replicas":[2,3,1]},
    {"topic":"my-topic","partition":2,"replicas":[3,1,2]}
  ]
}
```

Thực hiện reassignment:
```bash
bin/kafka-reassign-partitions.sh \
  --bootstrap-server broker1:9092 \
  --reassignment-json-file reassignment.json \
  --execute
```

### 4.3 Rebalance Partitions

Rebalancing partitions giúp cân bằng tải giữa các brokers và tối ưu hiệu suất cluster.

#### Tự động tạo Reassignment Plan
```bash
# Generate reassignment plan
bin/kafka-reassign-partitions.sh \
  --bootstrap-server broker1:9092 \
  --topics-to-move-json-file topics.json \
  --broker-list "1,2,3,4" \
  --generate
```

Ví dụ với topics.json:
```json
{
  "topics": [
    {"topic": "my-topic"}
  ],
  "version": 1
}
```

#### Kiểm tra Progress của Reassignment
```bash
bin/kafka-reassign-partitions.sh \
  --bootstrap-server broker1:9092 \
  --reassignment-json-file reassignment.json \
  --verify
```

#### Leader Election
Preferred leader election:
```bash
bin/kafka-leader-election.sh \
  --bootstrap-server broker1:9092 \
  --election-type PREFERRED \
  --all-topic-partitions
```

### 4.4 Topic Configs

Kafka cho phép cấu hình chi tiết cho từng topic để tối ưu hóa performance và storage.

#### Set Topic Configs
```bash
bin/kafka-configs.sh --alter \
  --bootstrap-server broker1:9092 \
  --entity-type topics \
  --entity-name my-topic \
  --add-config retention.ms=86400000,max.message.bytes=1000000
```

#### List Topic Configs
```bash
bin/kafka-configs.sh --describe \
  --bootstrap-server broker1:9092 \
  --entity-type topics \
  --entity-name my-topic
```

#### Delete Topic Configs
```bash
bin/kafka-configs.sh --alter \
  --bootstrap-server broker1:9092 \
  --entity-type topics \
  --entity-name my-topic \
  --delete-config retention.ms
```

#### Các tham số cấu hình quan trọng

| Config | Mô tả | Giá trị thông dụng |
|--------|-------|-------------------|
| `cleanup.policy` | Chiến lược dọn dẹp dữ liệu | `delete` hoặc `compact` |
| `retention.ms` | Thời gian lưu trữ messages | `604800000` (7 days) |
| `retention.bytes` | Kích thước tối đa của partition | `1073741824` (1GB) |
| `segment.bytes` | Kích thước của các log segments | `1073741824` (1GB) |
| `segment.ms` | Thời gian trước khi roll log segment | `604800000` (7 days) |
| `min.insync.replicas` | Số lượng tối thiểu ISR cho ghi | `2` |
| `max.message.bytes` | Kích thước tối đa của message | `1000000` (1MB) |

## 5. Producers và Consumers

### 5.1 Producer Configurations

Producers gửi dữ liệu vào Kafka topics. Dưới đây là các cấu hình quan trọng cho producers.

#### Các tham số cấu hình cơ bản
```properties
# Cấu hình kết nối
bootstrap.servers=broker1:9092,broker2:9092,broker3:9092
key.serializer=org.apache.kafka.common.serialization.StringSerializer
value.serializer=org.apache.kafka.common.serialization.StringSerializer

# Performance tuning
batch.size=16384
linger.ms=5
buffer.memory=33554432
compression.type=snappy

# Reliability tuning
acks=all
retries=3
max.in.flight.requests.per.connection=5
enable.idempotence=true
```

#### Acks và Durability
- `acks=0`: Producer không đợi acknowledgment (highest throughput, no durability)
- `acks=1`: Producer đợi leader acknowledgment (balanced)
- `acks=all`: Producer đợi tất cả ISR acknowledgments (lowest throughput, highest durability)

#### Compression
Kafka hỗ trợ các kiểu nén:
- `none`: Không nén
- `gzip`: Nén tốt nhất nhưng CPU intensive
- `snappy`: Cân bằng giữa compression ratio và CPU usage
- `lz4`: Compression/decompression nhanh
- `zstd`: Compression tốt, hiệu suất tốt (từ Kafka 2.1.0)

#### Batching và Performance
```properties
# Tăng batch size cho throughput cao hơn
batch.size=131072

# Chờ để gom messages thành batch
linger.ms=10

# Buffer memory
buffer.memory=67108864
```

#### Ví dụ Java Producer
```java
Properties props = new Properties();
props.put("bootstrap.servers", "broker1:9092,broker2:9092,broker3:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("acks", "all");
props.put("retries", 3);
props.put("batch.size", 16384);
props.put("linger.ms", 5);
props.put("enable.idempotence", true);
props.put("compression.type", "snappy");

Producer<String, String> producer = new KafkaProducer<>(props);

// Sync send
producer.send(new ProducerRecord<>("my-topic", "key", "value")).get();

// Async send with callback
producer.send(new ProducerRecord<>("my-topic", "key", "value"), 
    (metadata, exception) -> {
        if (exception != null) {
            exception.printStackTrace();
        } else {
            System.out.println("Sent message to " + metadata.topic() + 
                " partition " + metadata.partition() + 
                " offset " + metadata.offset());
        }
    });

producer.close();
```

### 5.2 Consumer Configurations

Consumers đọc dữ liệu từ Kafka topics. Dưới đây là các cấu hình quan trọng cho consumers.

#### Các tham số cấu hình cơ bản
```properties
# Cấu hình kết nối
bootstrap.servers=broker1:9092,broker2:9092,broker3:9092
key.deserializer=org.apache.kafka.common.serialization.StringDeserializer
value.deserializer=org.apache.kafka.common.serialization.StringDeserializer
group.id=my-consumer-group

# Offset management
auto.offset.reset=earliest
enable.auto.commit=false

# Performance tuning
fetch.min.bytes=1
fetch.max.bytes=52428800
fetch.max.wait.ms=500
max.partition.fetch.bytes=1048576
max.poll.records=500
```

#### Offset Management
- `auto.offset.reset`: Xác định hành vi khi không có offset cho consumer group
  - `earliest`: Đọc từ đầu topic (equivalent với --from-beginning trong console consumer)
  - `latest`: Đọc từ end của topic (default)
  - `none`: Throw exception nếu không tìm thấy offset

- `enable.auto.commit`: Tự động commit offsets
  - `true`: Commit tự động theo auto.commit.interval.ms
  - `false`: Commit thủ công để kiểm soát tốt hơn

#### Consumption Patterns
**Đọc một lần (at-most-once)**:
```properties
enable.auto.commit=true
```

**Đọc ít nhất một lần (at-least-once)**:
```properties
enable.auto.commit=false
# Commit thủ công sau khi xử lý
```

**Đọc chính xác một lần (exactly-once)**:
```properties
enable.auto.commit=false
isolation.level=read_committed
# Sử dụng transactional APIs và consumer.commitSync()
```

#### Ví dụ Java Consumer
```java
Properties props = new Properties();
props.put("bootstrap.servers", "broker1:9092,broker2:9092,broker3:9092");
props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("group.id", "my-consumer-group");
props.put("auto.offset.reset", "earliest");
props.put("enable.auto.commit", "false");
props.put("max.poll.records", 500);

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("my-topic"));

try {
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
        for (ConsumerRecord<String, String> record : records) {
            System.out.printf("offset = %d, key = %s, value = %s%n", 
                record.offset(), record.key(), record.value());
            // Process record
        }
        // Commit offsets manually
        consumer.commitSync();
    }
} finally {
    consumer.close();
}
```

### 5.3 Consumer Groups

Consumer groups cho phép scale việc xử lý messages bằng cách phân chia partitions giữa các consumers.

#### Consumer Group Administration
```bash
# Liệt kê consumer groups
bin/kafka-consumer-groups.sh --bootstrap-server broker1:9092 --list

# Mô tả consumer group
bin/kafka-consumer-groups.sh --bootstrap-server broker1:9092 \
  --describe --group my-consumer-group

# Reset offsets
bin/kafka-consumer-groups.sh --bootstrap-server broker1:9092 \
  --group my-consumer-group \
  --topic my-topic \
  --reset-offsets --to-earliest --execute
```

#### Rebalance Listeners
```java
consumer.subscribe(Collections.singletonList("my-topic"), new ConsumerRebalanceListener() {
    @Override
    public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
        // Commit offsets for partitions being revoked
        consumer.commitSync();
    }

    @Override
    public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
        // Initialize when partitions are assigned
    }
});
```

#### Static Group Membership
```properties
# Cấu hình static group membership
group.instance.id=consumer-1

# Tăng session timeout để tránh rebalances không cần thiết
session.timeout.ms=60000
```

### 5.4 Message Delivery Semantics

Kafka cung cấp nhiều options cho việc gửi và nhận messages với các mức độ reliability khác nhau.

#### At-Most-Once Delivery
Producer:
```properties
acks=0
```

Consumer:
```properties
enable.auto.commit=true
auto.commit.interval.ms=1000
```

#### At-Least-Once Delivery
Producer:
```properties
acks=all
retries=3
max.in.flight.requests.per.connection=1
```

Consumer:
```properties
enable.auto.commit=false
# Commit thủ công sau khi xử lý
```

#### Exactly-Once Delivery
Producer:
```properties
enable.idempotence=true
acks=all
retries=Integer.MAX_VALUE
max.in.flight.requests.per.connection=5
```

Transactional Producer:
```java
// Khởi tạo transaction producer
props.put("transactional.id", "my-transactional-id");
Producer<String, String> producer = new KafkaProducer<>(props);
producer.initTransactions();

try {
    producer.beginTransaction();
    // Gửi messages
    producer.send(new ProducerRecord<>("my-topic", "key", "value"));
    // Commit transaction
    producer.commitTransaction();
} catch (Exception e) {
    producer.abortTransaction();
    throw e;
} finally {
    producer.close();
}
```

Consumer cho Exactly-Once:
```properties
isolation.level=read_committed
enable.auto.commit=false
```

## 6. Kafka Connect

### 6.1 Cài đặt Kafka Connect

Kafka Connect là một framework cho việc stream dữ liệu giữa Kafka và các hệ thống khác.

#### Standalone Mode
```bash
# Tạo file connect-standalone.properties
cat > connect-standalone.properties << EOF
bootstrap.servers=broker1:9092,broker2:9092,broker3:9092
key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=true
value.converter.schemas.enable=true
offset.storage.file.filename=/tmp/connect.offsets
offset.flush.interval.ms=10000
plugin.path=/usr/share/java,/path/to/connectors
EOF

# Khởi động Connect worker
bin/connect-standalone.sh connect-standalone.properties connector1.properties [connector2.properties ...]
```

#### Distributed Mode
```bash
# Tạo file connect-distributed.properties
cat > connect-distributed.properties << EOF
bootstrap.servers=broker1:9092,broker2:9092,broker3:9092
group.id=connect-cluster
key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=true
value.converter.schemas.enable=true
offset.storage.topic=connect-offsets
offset.storage.replication.factor=3
config.storage.topic=connect-configs
config.storage.replication.factor=3
status.storage.topic=connect-status
status.storage.replication.factor=3
plugin.path=/usr/share/java,/path/to/connectors
EOF

# Khởi động Connect cluster
bin/connect-distributed.sh connect-distributed.properties
```

#### REST API
Kafka Connect có một REST API để quản lý connectors:
```bash
# Liệt kê tất cả connectors
curl -X GET http://connect:8083/connectors

# Tạo connector mới
curl -X POST -H "Content-Type: application/json" \
  --data @connector-config.json \
  http://connect:8083/connectors

# Kiểm tra status của connector
curl -X GET http://connect:8083/connectors/my-connector/status
```

### 6.2 Source Connectors

Source connectors đưa dữ liệu từ hệ thống khác vào Kafka.

#### File Source Connector
```json
{
  "name": "file-source",
  "config": {
    "connector.class": "org.apache.kafka.connect.file.FileStreamSourceConnector",
    "tasks.max": "1",
    "file": "/path/to/input.txt",
    "topic": "file-input-topic"
  }
}
```

#### JDBC Source Connector
```json
{
  "name": "jdbc-source",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
    "tasks.max": "1",
    "connection.url": "jdbc:mysql://mysql:3306/mydb",
    "connection.user": "user",
    "connection.password": "password",
    "table.whitelist": "users",
    "mode": "incrementing",
    "incrementing.column.name": "id",
    "topic.prefix": "mysql-"
  }
}
```

#### MongoDB Source Connector
```json
{
  "name": "mongodb-source",
  "config": {
    "connector.class": "com.mongodb.kafka.connect.MongoSourceConnector",
    "tasks.max": "1",
    "connection.uri": "mongodb://mongo:27017",
    "database": "mydb",
    "collection": "users",
    "topic.prefix": "mongo-",
    "poll.max.batch.size": 1000,
    "poll.await.time.ms": 5000
  }
}
```

### 6.3 Sink Connectors

Sink connectors đưa dữ liệu từ Kafka sang hệ thống khác.

#### File Sink Connector
```json
{
  "name": "file-sink",
  "config": {
    "connector.class": "org.apache.kafka.connect.file.FileStreamSinkConnector",
    "tasks.max": "1",
    "file": "/path/to/output.txt",
    "topics": "file-output-topic"
  }
}
```

#### JDBC Sink Connector
```json
{
  "name": "jdbc-sink",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
    "tasks.max": "1",
    "connection.url": "jdbc:postgresql://postgres:5432/mydb",
    "connection.user": "user",
    "connection.password": "password",
    "topics": "pg-users",
    "auto.create": "true",
    "auto.evolve": "true",
    "insert.mode": "upsert",
    "pk.mode": "record_key",
    "pk.fields": "id"
  }
}
```

#### Elasticsearch Sink Connector
```json
{
  "name": "elasticsearch-sink",
  "config": {
    "connector.class": "io.confluent.connect.elasticsearch.ElasticsearchSinkConnector",
    "tasks.max": "1",
    "topics": "es-documents",
    "connection.url": "http://elasticsearch:9200",
    "type.name": "kafka-connect",
    "key.ignore": "true",
    "schema.ignore": "true",
    "behavior.on.null.values": "delete"
  }
}
```

### 6.4 Transformations

Single Message Transformations (SMTs) cho phép biến đổi messages trong Kafka Connect pipeline.

#### Rename fields
```json
{
  "name": "my-connector",
  "config": {
    "connector.class": "org.apache.kafka.connect.file.FileStreamSourceConnector",
    "tasks.max": "1",
    "file": "/path/to/input.txt",
    "topic": "my-topic",
    "transforms": "RenameField",
    "transforms.RenameField.type": "org.apache.kafka.connect.transforms.ReplaceField$Value",
    "transforms.RenameField.renames": "oldName:newName"
  }
}
```

#### Extract field
```json
{
  "transforms": "ExtractField",
  "transforms.ExtractField.type": "org.apache.kafka.connect.transforms.ExtractField$Value",
  "transforms.ExtractField.field": "data"
}
```

#### Cast types
```json
{
  "transforms": "Cast",
  "transforms.Cast.type": "org.apache.kafka.connect.transforms.Cast$Value",
  "transforms.Cast.spec": "age:int32,active:boolean"
}
```

#### Chaining transformations
```json
{
  "transforms": "RenameField,Cast",
  "transforms.RenameField.type": "org.apache.kafka.connect.transforms.ReplaceField$Value",
  "transforms.RenameField.renames": "oldName:newName",
  "transforms.Cast.type": "org.apache.kafka.connect.transforms.Cast$Value",
  "transforms.Cast.spec": "age:int32,active:boolean"
}
```

## 7. Kafka Streams

### 7.1 Kiến trúc Streams

Kafka Streams là một client library cho việc xử lý và phân tích dữ liệu từ Kafka.

#### Cấu hình cơ bản
```java
Properties props = new Properties();
props.put(StreamsConfig.APPLICATION_ID_CONFIG, "my-streams-app");
props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "broker1:9092,broker2:9092,broker3:9092");
props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
props.put(StreamsConfig.NUM_STREAM_THREADS_CONFIG, 4);
props.put(StreamsConfig.COMMIT_INTERVAL_MS_CONFIG, 10000);

StreamsBuilder builder = new StreamsBuilder();
// Build topology
KafkaStreams streams = new KafkaStreams(builder.build(), props);
streams.start();
```

#### Streams Concepts
- **Processor Topology**: Graph các stream processors
- **Source Processor**: Đọc từ Kafka topics
- **Sink Processor**: Ghi vào Kafka topics
- **Stream Processor**: Biến đổi dữ liệu

#### Fault Tolerance
- Kafka Streams lưu trữ state trong Kafka topics
- Tự động recovery khi applications restart
- Rebalancing tasks khi instances thêm/bớt

### 7.2 Streams DSL

Kafka Streams DSL là high-level API cho stream processing.

#### Ví dụ Word Count
```java
StreamsBuilder builder = new StreamsBuilder();

KStream<String, String> textLines = builder.stream("text-input-topic");
KTable<String, Long> wordCounts = textLines
    .flatMapValues(value -> Arrays.asList(value.toLowerCase().split("\\W+")))
    .groupBy((key, value) -> value)
    .count();

wordCounts.toStream().to("word-count-output", Produced.with(Serdes.String(), Serdes.Long()));
```

#### Filtering và Mapping
```java
KStream<String, String> source = builder.stream("input-topic");

// Filter
KStream<String, String> filtered = source.filter((key, value) -> value.contains("important"));

// Map keys
KStream<Integer, String> keyMapped = source.map((key, value) -> 
    KeyValue.pair(Integer.parseInt(key), value));

// Map values
KStream<String, Integer> valueMapped = source.mapValues(value -> value.length());

// FlatMap
KStream<String, String> flatMapped = source.flatMap((key, value) ->
    Arrays.asList(
        KeyValue.pair(key, value.toLowerCase()),
        KeyValue.pair(key, value.toUpperCase())
    ));
```

#### Joins
```java
// KStream-KStream join
KStream<String, String> left = builder.stream("left-topic");
KStream<String, String> right = builder.stream("right-topic");

KStream<String, String> joined = left.join(
    right,
    (leftValue, rightValue) -> leftValue + "-" + rightValue,
    JoinWindows.of(Duration.ofMinutes(5))
);

// KStream-KTable join
KStream<String, String> stream = builder.stream("stream-topic");
KTable<String, String> table = builder.table("table-topic");

KStream<String, String> joinedStreamTable = stream.join(
    table,
    (streamValue, tableValue) -> streamValue + "-" + tableValue
);
```

#### Windowing
```java
KStream<String, Long> clickstream = builder.stream("clicks", 
    Consumed.with(Serdes.String(), Serdes.Long()));

// Tumbling window (fixed size, non-overlapping)
clickstream
    .groupByKey()
    .windowedBy(TimeWindows.of(Duration.ofMinutes(5)))
    .count()
    .toStream()
    .to("clicks-per-5min");

// Hopping window (fixed size, overlapping)
clickstream
    .groupByKey()
    .windowedBy(TimeWindows.of(Duration.ofMinutes(5)).advanceBy(Duration.ofMinutes(1)))
    .count()
    .toStream()
    .to("clicks-per-5min-advancing-1min");

// Session window (dynamic size based on activity)
clickstream
    .groupByKey()
    .windowedBy(SessionWindows.with(Duration.ofMinutes(5)))
    .count()
    .toStream()
    .to("clicks-per-session");
```

### 7.3 State Stores

State Stores cho phép Kafka Streams lưu trữ và truy vấn dữ liệu cục bộ.

#### In-memory State Store
```java
StoreBuilder<KeyValueStore<String, Long>> countStore =
    Stores.keyValueStoreBuilder(
        Stores.inMemoryKeyValueStore("counts"),
        Serdes.String(),
        Serdes.Long()
    );

builder.addStateStore(countStore);

KStream<String, String> source = builder.stream("input-topic");
source.process(() -> new MyProcessor(), "counts");

class MyProcessor implements Processor<String, String, Void, Void> {
    private KeyValueStore<String, Long> kvStore;
    private ProcessorContext<Void, Void> context;

    @Override
    public void init(ProcessorContext<Void, Void> context) {
        this.context = context;
        this.kvStore = context.getStateStore("counts");
    }

    @Override
    public void process(Record<String, String> record) {
        String key = record.key();
        Long oldValue = kvStore.get(key);
        Long newValue = (oldValue == null) ? 1L : oldValue + 1L;
        kvStore.put(key, newValue);
    }

    @Override
    public void close() {}
}
```

#### Persistent State Store
```java
StoreBuilder<KeyValueStore<String, Long>> countStore =
    Stores.keyValueStoreBuilder(
        Stores.persistentKeyValueStore("persistent-counts"),
        Serdes.String(),
        Serdes.Long()
    )
    .withCachingEnabled();

builder.addStateStore(countStore);
```

#### State Store với DSL
```java
KTable<String, Long> counts = builder.stream("input-topic", 
    Consumed.with(Serdes.String(), Serdes.String()))
    .groupByKey()
    .count(Materialized.<String, Long, KeyValueStore<Bytes, byte[]>>as("counts-store")
        .withKeySerde(Serdes.String())
        .withValueSerde(Serdes.Long()));
```

### 7.4 Interactive Queries

Interactive Queries cho phép truy vấn state stores trực tiếp từ ứng dụng.

#### Local State Query
```java
ReadOnlyKeyValueStore<String, Long> keyValueStore =
    streams.store(StoreQueryParameters.fromNameAndType(
        "counts-store", QueryableStoreTypes.keyValueStore()));

// Get value by key
Long count = keyValueStore.get("key");

// Iterate over all keys
KeyValueIterator<String, Long> range = keyValueStore.all();
while (range.hasNext()) {
    KeyValue<String, Long> next = range.next();
    System.out.println("Key: " + next.key + ", Value: " + next.value);
}
range.close();

// Range query
KeyValueIterator<String, Long> range = keyValueStore.range("key1", "key2");
```

#### Remote State Query
Cấu hình application.server để enable remote queries:
```java
props.put(StreamsConfig.APPLICATION_SERVER_CONFIG, "host:port");
```

Implement REST endpoint để expose state:
```java
@RestController
public class StateStoreController {
    private final KafkaStreams streams;
    
    public StateStoreController(KafkaStreams streams) {
        this.streams = streams;
    }
    
    @GetMapping("/state/{key}")
    public ResponseEntity<Long> getCount(@PathVariable String key) {
        ReadOnlyKeyValueStore<String, Long> keyValueStore =
            streams.store(StoreQueryParameters.fromNameAndType(
                "counts-store", QueryableStoreTypes.keyValueStore()));
        
        Long count = keyValueStore.get(key);
        if (count != null) {
            return ResponseEntity.ok(count);
        } else {
            return ResponseEntity.notFound().build();
        }
    }
}
```

## 8. Monitoring và Operations

### 8.1 Kafka Metrics

Kafka cung cấp nhiều metrics qua JMX để giám sát hiệu suất và sức khỏe của cluster.

#### Broker Metrics
- **UnderReplicatedPartitions**: Số partitions không đủ replicas
- **OfflinePartitionsCount**: Số partitions offline
- **ActiveControllerCount**: Xác định controller (1 cho controller, 0 cho brokers khác)
- **RequestHandlerAvgIdlePercent**: CPU utilization của request handler threads
- **BytesInPerSec/BytesOutPerSec**: Network throughput

#### Topic & Partition Metrics
- **MessagesInPerSec**: Số messages được produce
- **BytesInPerSec/BytesOutPerSec**: Throughput cho một topic
- **ReplicationBytesInPerSec/ReplicationBytesOutPerSec**: Replication throughput
- **PartitionCount**: Số lượng partitions
- **LeaderCount**: Số lượng leadership partitions

#### Producer Metrics
- **batch-size-avg**: Kích thước trung bình của batches
- **record-send-rate**: Records/second được gửi
- **request-latency-avg**: Latency trung bình cho requests
- **record-error-rate**: Tỷ lệ lỗi khi gửi records

#### Consumer Metrics
- **records-consumed-rate**: Records/second được consume
- **fetch-rate**: Fetches/second
- **records-lag-max**: Độ trễ tối đa (khoảng cách giữa consumer và end of log)
- **bytes-consumed-rate**: Bytes/second được consume

### 8.2 JMX Monitoring

JMX (Java Management Extensions) là cách chính để expose metrics từ Kafka.

#### Cấu hình JMX
```bash
# Trên Kafka brokers
export JMX_PORT=9999
bin/kafka-server-start.sh config/server.properties
```

#### Kết nối với JMX
```bash
# Sử dụng JConsole
jconsole localhost:9999

# Sử dụng JMX client
jmxterm -l localhost:9999
```

#### Metrics Collection với tools
Cấu hình Kafka broker cho JMX:
```bash
export KAFKA_JMX_OPTS="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=9999 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false"
```

### 8.3 Prometheus và Grafana

Prometheus và Grafana là bộ đôi phổ biến để giám sát Kafka.

#### Cài đặt JMX Exporter
```bash
# Tải JMX Exporter
wget https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.16.1/jmx_prometheus_javaagent-0.16.1.jar

# Tạo file cấu hình kafka-2_0_0.yml
cat > kafka-2_0_0.yml << EOF
lowercaseOutputName: true
rules:
  - pattern: kafka.(\w+)<type=(.+), name=(.+)><>Value
    name: kafka_$1_$2_$3
    type: GAUGE
  - pattern: kafka.(\w+)<type=(.+), name=(.+)>(\w*):<>Value
    name: kafka_$1_$2_$3_$4
    type: GAUGE
EOF

# Cấu hình Kafka broker sử dụng JMX Exporter
export KAFKA_OPTS="-javaagent:/path/to/jmx_prometheus_javaagent-0.16.1.jar=8080:/path/to/kafka-2_0_0.yml"
```

#### Cấu hình Prometheus
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'kafka'
    static_configs:
      - targets: ['kafka1:8080', 'kafka2:8080', 'kafka3:8080']
```

#### Grafana Dashboard
Import Grafana dashboard cho Kafka (IDs phổ biến: 721, 7589, 10466)

### 8.4 Log Files

Kafka ghi logs tới nhiều locations khác nhau để debug và auditing.

#### Server Logs
```bash
# Default location
tail -f /path/to/kafka/logs/server.log

# Cấu hình log level và format
# Trong config/log4j.properties
log4j.rootLogger=INFO, stdout, kafkaAppender
```

#### Topic Partitions Logs
```bash
# Xem nội dung của partition log (offset đầu tiên, timestamp, ...)
bin/kafka-dump-log.sh --files /var/lib/kafka/data/my-topic-0/00000000000000000000.log --print-data-log
```

#### Kafka Log Cleanup
```bash
# Cấu hình log cleanup trong server.properties
log.retention.hours=168  # 7 days
log.retention.bytes=1073741824  # 1GB
log.segment.bytes=1073741824  # 1GB
log.cleaner.enable=true
```

## 9. Bảo mật Kafka

### 9.1 Authentication

Kafka hỗ trợ nhiều phương thức xác thực.

#### SASL/PLAIN
```properties
# server.properties
listeners=SASL_PLAINTEXT://broker1:9092
advertised.listeners=SASL_PLAINTEXT://broker1:9092
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.mechanism.inter.broker.protocol=PLAIN
sasl.enabled.mechanisms=PLAIN

# Tạo file JAAS config (config/kafka_server_jaas.conf)
KafkaServer {
  org.apache.kafka.common.security.plain.PlainLoginModule required
  username="admin"
  password="admin-secret"
  user_admin="admin-secret"
  user_alice="alice-secret"
  user_bob="bob-secret";
};
```

Cấu hình client:
```properties
# client.properties
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
```

Client JAAS config:
```
KafkaClient {
  org.apache.kafka.common.security.plain.PlainLoginModule required
  username="alice"
  password="alice-secret";
};
```

#### SASL/SCRAM
```properties
# server.properties
listeners=SASL_PLAINTEXT://broker1:9092
advertised.listeners=SASL_PLAINTEXT://broker1:9092
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.mechanism.inter.broker.protocol=SCRAM-SHA-256
sasl.enabled.mechanisms=SCRAM-SHA-256
```

Tạo SCRAM credentials:
```bash
bin/kafka-configs.sh --bootstrap-server broker1:9092 \
  --alter --add-config 'SCRAM-SHA-256=[password=alice-secret],SCRAM-SHA-512=[password=alice-secret]' \
  --entity-type users --entity-name alice
```

#### SSL/TLS
```properties
# server.properties
listeners=SSL://broker1:9093
advertised.listeners=SSL://broker1:9093
security.inter.broker.protocol=SSL

# SSL configurations
ssl.keystore.location=/path/to/kafka.server.keystore.jks
ssl.keystore.password=keystore-password
ssl.key.password=key-password
ssl.truststore.location=/path/to/kafka.server.truststore.jks
ssl.truststore.password=truststore-password
ssl.client.auth=required
```

Client configuration:
```properties
# client.properties
security.protocol=SSL
ssl.truststore.location=/path/to/client.truststore.jks
ssl.truststore.password=truststore-password
ssl.keystore.location=/path/to/client.keystore.jks
ssl.keystore.password=keystore-password
ssl.key.password=key-password
```

### 9.2 Authorization

Kafka sử dụng Access Control Lists (ACLs) để quản lý quyền truy cập.

#### Cấu hình Authorizer
```properties
# server.properties
authorizer.class.name=kafka.security.authorizer.AclAuthorizer
super.users=User:admin
```

#### Quản lý ACLs
```bash
# Thêm ACL để cho phép Alice produce vào "test-topic"
bin/kafka-acls.sh --bootstrap-server broker1:9092 \
  --add --allow-principal User:alice \
  --operation Write --topic test-topic

# Thêm ACL để cho phép Consumer Group "group1" consume từ "test-topic"
bin/kafka-acls.sh --bootstrap-server broker1:9092 \
  --add --allow-principal User:bob \
  --operation Read --topic test-topic \
  --group group1

# Liệt kê ACLs hiện tại
bin/kafka-acls.sh --bootstrap-server broker1:9092 --list
```

#### ACL cho Connectors và Streams
```bash
# ACL cho Kafka Connect
bin/kafka-acls.sh --bootstrap-server broker1:9092 \
  --add --allow-principal User:connect \
  --operation Read --topic connect-configs \
  --operation Write --topic connect-offsets \
  --operation Read --topic connect-status \
  --resource-pattern-type prefixed --topic connect- \
  --group connect-cluster

# ACL cho Kafka Streams
bin/kafka-acls.sh --bootstrap-server broker1:9092 \
  --add --allow-principal User:streams \
  --operation All --topic streams-app- \
  --resource-pattern-type prefixed \
  --group streams-app
```

### 9.3 Encryption

Kafka hỗ trợ hai loại encryption: encryption-in-transit và encryption-at-rest.

#### Encryption-in-Transit (SSL/TLS)
Được cấu hình thông qua SSL listener như trong mục Authentication.

Tạo certificates:
```bash
# Tạo CA
openssl req -new -x509 -keyout ca-key -out ca-cert -days 365

# Tạo server keystore
keytool -keystore kafka.server.keystore.jks -alias localhost -validity 365 -genkey
keytool -keystore kafka.server.keystore.jks -alias localhost -certreq -file cert-file
openssl x509 -req -CA ca-cert -CAkey ca-key -in cert-file -out cert-signed -days 365 -CAcreateserial
keytool -keystore kafka.server.keystore.jks -alias CARoot -import -file ca-cert
keytool -keystore kafka.server.keystore.jks -alias localhost -import -file cert-signed

# Tạo client truststore
keytool -keystore kafka.client.truststore.jks -alias CARoot -import -file ca-cert
```

#### Encryption-at-Rest
Kafka không có built-in encryption-at-rest. Bạn có thể sử dụng:

1. **Disk-level encryption**: LUKS, dm-crypt, eCryptfs
2. **HDFS encryption**: Nếu sử dụng HDFS để lưu trữ Kafka data

### 9.4 Audit

Kafka không có built-in auditing, nhưng có thể triển khai bằng nhiều cách.

#### Broker-side Audit
Sử dụng request.logger.name trong Kafka:
```properties
# server.properties
request.logger.name=REQUEST_LOGGER
log4j.logger.REQUEST_LOGGER=TRACE, requestAppender
```

#### Client-side Audit
Sử dụng Kafka Interceptors:
```java
// Producer interceptor
Properties props = new Properties();
List<String> interceptors = new ArrayList<>();
interceptors.add("com.example.AuditProducerInterceptor");
props.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG, interceptors);

// Consumer interceptor
Properties props = new Properties();
List<String> interceptors = new ArrayList<>();
interceptors.add("com.example.AuditConsumerInterceptor");
props.put(ConsumerConfig.INTERCEPTOR_CLASSES_CONFIG, interceptors);
```

#### Centralized Audit
Sử dụng Kafka Streams để collect và analyze audit data:
```java
StreamsBuilder builder = new StreamsBuilder();
KStream<String, String> auditStream = builder.stream("audit-events-topic");

// Process and store audit events
auditStream.foreach((key, value) -> {
    // Log audit event
    logger.info("Audit event: " + value);
});
```

## 10. Performance Tuning

### 10.1 Broker Tuning

#### Disk I/O
```properties
# Sử dụng multiple log directories trên different drives
log.dirs=/mnt/disk1/kafka-logs,/mnt/disk2/kafka-logs

# Tối ưu flush settings
log.flush.interval.messages=10000
log.flush.interval.ms=1000

# Sử dụng flush thay vì fsync
log.flush.scheduler.interval.ms=3000
```

#### Network
```properties
# Tăng socket buffer sizes
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600

# Tối ưu network threads
num.network.threads=8
num.io.threads=16
queued.max.requests=1000
```

#### Batching và Compression
```properties
# Compression type (producer cũng cần cấu hình tương ứng)
compression.type=producer
```

#### Log Compaction
```properties
# Cấu hình Log Cleaner
log.cleaner.enable=true
log.cleaner.threads=2
log.cleaner.dedupe.buffer.size=134217728
```

### 10.2 Producer Tuning

#### Batch Settings
```properties
# Tăng batch size
batch.size=131072  # 128KB

# Tăng linger time để tạo batches lớn hơn
linger.ms=10

# Tăng buffer memory
buffer.memory=67108864  # 64MB
```

#### Compression
```properties
# Sử dụng compression
compression.type=snappy  # hoặc gzip, lz4, zstd
```

#### Acks và Retries
```properties
# Cân bằng giữa throughput và reliability
acks=1  # 0 cho highest throughput, 'all' cho highest reliability

# Cấu hình retries
retries=3
retry.backoff.ms=100
```

#### Throughput Optimization
```properties
# Cho phép nhiều request in-flight
max.in.flight.requests.per.connection=5

# Sử dụng idempotent producer
enable.idempotence=true

# Tăng maximum request size
max.request.size=5242880  # 5MB
```

### 10.3 Consumer Tuning

#### Fetch Size
```properties
# Tăng fetch size
fetch.min.bytes=1024
fetch.max.bytes=52428800  # 50MB
max.partition.fetch.bytes=1048576  # 1MB
```

#### Commit Strategy
```properties
# Optimizing commits
enable.auto.commit=false  # Manual commits for better control
auto.commit.interval.ms=5000  # If auto commit is enabled
```

#### Polling
```properties
# Tối ưu polling
max.poll.records=500  # Số records tối đa mỗi poll
max.poll.interval.ms=300000  # Max time between polls
```

#### Concurrency
```properties
# Scale consumers
# Sử dụng multiple consumer instances trong consumer group
# Đảm bảo số lượng partitions >= consumer instances
```

### 10.4 JVM Tuning

#### Heap Size
```bash
export KAFKA_HEAP_OPTS="-Xms6g -Xmx6g"
```

#### Garbage Collection
```bash
export KAFKA_JVM_PERFORMANCE_OPTS="-server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -XX:G1HeapRegionSize=16M -XX:MinMetaspaceFreeRatio=50 -XX:MaxMetaspaceFreeRatio=80"
```

#### JVM Monitoring
```bash
export JMX_PORT=9999
```

### 10.5 OS Tuning

#### File Descriptors
```bash
# /etc/security/limits.conf
kafka soft nofile 65536
kafka hard nofile 65536
```

#### Virtual Memory
```bash
# /etc/sysctl.conf
vm.swappiness=1
vm.dirty_background_ratio=5
vm.dirty_ratio=80
```

#### Network
```bash
# /etc/sysctl.conf
net.core.wmem_max=2097152
net.core.rmem_max=2097152
net.ipv4.tcp_wmem=4096 65536 2097152
net.ipv4.tcp_rmem=4096 65536 2097152
```

#### Disk
```bash
# Cấu hình mount options (noatime)
mount -o remount,noatime /path/to/kafka/data

# Sử dụng noop/deadline IO scheduler cho SSDs
echo "deadline" > /sys/block/sda/queue/scheduler
```

## 11. Troubleshooting

### 11.1 Common Issues

#### Under-replicated Partitions
```bash
# Kiểm tra under-replicated partitions
bin/kafka-topics.sh --bootstrap-server broker1:9092 --describe | grep -i under-replicated

# Kiểm tra đồng bộ với ISRs
bin/kafka-topics.sh --bootstrap-server broker1:9092 --describe --topic my-topic

# Kiểm tra logs
grep "isr shrink" /path/to/kafka/logs/server.log
```

#### Consumer Lag
```bash
# Kiểm tra consumer lag
bin/kafka-consumer-groups.sh --bootstrap-server broker1:9092 --describe --group my-consumer-group

# Reset offsets nếu cần
bin/kafka-consumer-groups.sh --bootstrap-server broker1:9092 --group my-consumer-group --reset-offsets --to-latest --execute --topic my-topic
```

#### Broker Failures
```bash
# Kiểm tra broker status
bin/zookeeper-shell.sh zookeeper1:2181 ls /brokers/ids

# Kiểm tra controller
bin/zookeeper-shell.sh zookeeper1:2181 get /controller

# Kiểm tra logs
grep "shut down completed" /path/to/kafka/logs/server.log
```

#### Network Connectivity
```bash
# Test network connectivity
nc -zv broker1 9092

# Kiểm tra config
grep "listeners" /path/to/kafka/config/server.properties
grep "advertised.listeners" /path/to/kafka/config/server.properties
```

### 11.2 Partition Reassignment

Partition reassignment cho phép bạn di chuyển partition giữa các brokers.

#### Reassignment JSON
```json
{
  "version": 1,
  "partitions": [
    {
      "topic": "my-topic",
      "partition": 0,
      "replicas": [1, 2, 3]
    },
    {
      "topic": "my-topic",
      "partition": 1,
      "replicas": [2, 3, 1]
    }
  ]
}
```

#### Partition Reassignment Tools
```bash
# Generate reassignment plan
bin/kafka-reassign-partitions.sh --bootstrap-server broker1:9092 \
  --generate \
  --topics-to-move-json-file topics.json \
  --broker-list "1,2,3,4"

# Execute reassignment
bin/kafka-reassign-partitions.sh --bootstrap-server broker1:9092 \
  --execute \
  --reassignment-json-file reassignment.json

# Verify reassignment
bin/kafka-reassign-partitions.sh --bootstrap-server broker1:9092 \
  --verify \
  --reassignment-json-file reassignment.json
```

#### Throttle Reassignment
```bash
# Giới hạn bandwidth cho reassignment
bin/kafka-reassign-partitions.sh --bootstrap-server broker1:9092 \
  --execute \
  --reassignment-json-file reassignment.json \
  --throttle 10000000  # 10MB/s

# Reset throttle sau khi hoàn thành
bin/kafka-configs.sh --bootstrap-server broker1:9092 \
  --entity-type brokers \
  --entity-name 1 \
  --alter \
  --delete-config leader.replication.throttled.rate,follower.replication.throttled.rate
```

### 11.3 Recovery Scenarios

#### Corrupt Log Recovery
```bash
# Kiểm tra corruption
bin/kafka-log-dirs.sh --bootstrap-server broker1:9092 \
  --describe --log-dirs /var/lib/kafka/logs

# Nếu phát hiện corruption, xác định partition affected
bin/kafka-topics.sh --bootstrap-server broker1:9092 --describe

# Delete corrupt replica và reassign partition
rm -rf /var/lib/kafka/logs/my-topic-0

# Nếu cần, reassign partition
# Xem mục Partition Reassignment
```

#### Broker Failure Recovery
```bash
# Nếu broker failure, kiểm tra logs
grep "shut down completed" /path/to/kafka/logs/server.log

# Khởi động lại broker
systemctl start kafka

# Nếu broker không thể recovery, replace broker ID
# Cấu hình server.properties với cùng broker.id
# Khởi động broker mới
```

#### ZooKeeper Failure Recovery
```bash
# Kiểm tra ZooKeeper status
echo stat | nc zookeeper1 2181

# Khởi động lại ZooKeeper nếu cần
systemctl restart zookeeper

# Nếu ZooKeeper ensemble có vấn đề
# Kiểm tra ZooKeeper logs
cat /var/log/zookeeper/zookeeper.log
```

### 11.4 Debugging Tools

#### Log Inspection
```bash
# Server logs
tail -f /path/to/kafka/logs/server.log

# Controller logs
grep -i controller /path/to/kafka/logs/server.log

# ZooKeeper logs
tail -f /var/log/zookeeper/zookeeper.log
```

#### Metrics Inspection
```bash
# JMX Metrics
jconsole localhost:9999

# Dump all metrics
bin/kafka-run-class.sh kafka.tools.JmxTool \
  --jmx-url service:jmx:rmi:///jndi/rmi://localhost:9999/jmxrmi \
  --object-name kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec \
  --reporting-interval 1000
```

#### Network Debugging
```bash
# Check open ports
netstat -tuln | grep 9092

# Monitor network traffic
tcpdump -i eth0 port 9092 -w kafka.pcap

# Analyze with Wireshark
wireshark kafka.pcap
```

#### Data Inspection
```bash
# Check partition contents
bin/kafka-dump-log.sh --files /var/lib/kafka/logs/my-topic-0/00000000000000000000.log --print-data-log

# Find offsets by timestamp
bin/kafka-run-class.sh kafka.tools.GetOffsetShell \
  --bootstrap-server broker1:9092 \
  --topic my-topic \
  --time 1597084800000  # Unix timestamp in ms
```

## 12. Kafka trong Cloud và Containers

### 12.1 Kafka trên Kubernetes

#### Sử dụng Strimzi Operator
```yaml
# Cài đặt Strimzi Operator
kubectl apply -f https://strimzi.io/install/latest

# Tạo Kafka cluster
cat <<EOF | kubectl apply -f -
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: my-cluster
spec:
  kafka:
    version: 3.3.1
    replicas: 3
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - name: tls
        port: 9093
        type: internal
        tls: true
    config:
      offsets.topic.replication.factor: 3
      transaction.state.log.replication.factor: 3
      transaction.state.log.min.isr: 2
      default.replication.factor: 3
      min.insync.replicas: 2
      inter.broker.protocol.version: "3.3"
    storage:
      type: jbod
      volumes:
      - id: 0
        type: persistent-claim
        size: 100Gi
        deleteClaim: false
  zookeeper:
    replicas: 3
    storage:
      type: persistent-claim
      size: 100Gi
      deleteClaim: false
  entityOperator:
    topicOperator: {}
    userOperator: {}
EOF
```

#### Tạo Topic trong Kubernetes
```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: my-topic
  labels:
    strimzi.io/cluster: my-cluster
spec:
  partitions: 12
  replicas: 3
  config:
    retention.ms: 604800000
    segment.bytes: 1073741824
```

#### Expose Kafka ra ngoài cluster
```yaml
# Tạo Kafka với external access
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: my-cluster
spec:
  kafka:
    # ...previous config...
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - name: external
        port: 9094
        type: nodeport
        tls: false
```

### 12.2 Kafka trên AWS

#### AWS MSK (Managed Streaming for Kafka)
```bash
# Sử dụng AWS CLI để tạo MSK cluster
aws kafka create-cluster \
  --cluster-name msk-cluster \
  --broker-node-group-info file://broker-nodes.json \
  --kafka-version "2.8.1" \
  --number-of-broker-nodes 3 \
  --enhanced-monitoring "PER_BROKER" \
  --encryption-info file://encryption-info.json

# Lấy bootstrap brokers
aws kafka get-bootstrap-brokers --cluster-arn <cluster-arn>

# Tạo topic
bin/kafka-topics.sh --create \
  --bootstrap-server <bootstrap-brokers> \
  --topic my-topic \
  --partitions 12 \
  --replication-factor 3
```

#### Các options lưu trữ trên AWS
1. **EBS volumes** (MSK mặc định)
   - Throughput: Sử dụng EBS Provisioned IOPS
   - Multi-AZ: MSK tự động phân phối brokers qua các AZs

2. **Tự quản lý EC2**
   - Sử dụng i3 instances với local NVMe storage cho hiệu suất cao
   - Cấu hình multi-AZ với rack awareness

#### IAM và Security trên AWS
```bash
# Tạo IAM role cho MSK
aws iam create-role \
  --role-name MSKRole \
  --assume-role-policy-document file://trust-policy.json

# Attach MSK policy
aws iam attach-role-policy \
  --role-name MSKRole \
  --policy-arn "arn:aws:iam::aws:policy/AmazonMSKFullAccess"

# Cấu hình MSK với IAM authentication
aws kafka update-security \
  --cluster-arn <cluster-arn> \
  --current-cluster-kafka-version "2.8.1" \
  --client-authentication file://client-auth.json
```

### 12.3 Kafka trên GCP

#### Confluent Cloud trên GCP
```bash
# Sử dụng Confluent CLI
confluent login
confluent environment create "production"
confluent kafka cluster create "kafka-cluster" \
  --cloud gcp \
  --region us-central1 \
  --type dedicated \
  --availability single-zone \
  --cku 1

# Tạo API key
confluent api-key create --resource "kafka-cluster" --description "My API Key"

# Tạo topic
confluent kafka topic create my-topic \
  --partitions 12 \
  --cluster "kafka-cluster" \
  --config "retention.ms=604800000"
```

#### Self-managed trên GCP
1. **Deployment Architecture**
   - Sử dụng Google Compute Engine với SSD disks
   - Phân phối brokers qua multiple zones
   - Sử dụng VPC Network và firewall rules

2. **Storage Options**
   - Local SSD (ephemeral): Hiệu suất cao, cần data replication
   - Persistent Disk: Durable storage, nhiều tùy chọn (Standard, Balanced, SSD, Extreme)

### 12.4 Managed Kafka Services

#### Confluent Cloud
```bash
# Sử dụng Confluent CLI
confluent login
confluent environment create "dev"
confluent kafka cluster create "kafka-cluster-dev" \
  --cloud aws \
  --region us-east-1 \
  --type basic

# Tạo Connector
confluent connect create "s3-sink" \
  --config connector-config.json
```

#### Amazon MSK
```bash
# Tạo MSK Serverless cluster
aws kafka create-cluster-v2 \
  --cluster-name serverless-cluster \
  --provisioned serverless \
  --client-authentication unauthenticated

# Tạo MSK Connect connector
aws kafkaconnect create-connector \
  --connector-name "s3-sink" \
  --connector-configuration file://connector-config.json \
  --capacity auto-scaling \
  --kafka-cluster cluster-arn \
  --service-execution-role-arn role-arn \
  --plugins plugin-arn
```

#### Aiven for Kafka
```bash
# Sử dụng Aiven CLI
avn service create kafka-service \
  --service-type kafka \
  --plan business-4 \
  --cloud aws-us-east-1

# Tạo topic
avn service topic-create kafka-service my-topic \
  --partitions 12 \
  --replication 3

# Tạo Kafka Connect service
avn service create kafka-connect \
  --service-type kafka_connect \
  --plan business-4 \
  --cloud aws-us-east-1 \
  --project my-project
```

---

Tài liệu này cung cấp một hướng dẫn toàn diện về Apache Kafka từ cài đặt, cấu hình đến triển khai và bảo trì. Tùy thuộc vào nhu cầu cụ thể, bạn có thể điều chỉnh các tham số và cấu hình để phù hợp với workload và môi trường của mình.
