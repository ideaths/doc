# Redis Sentinel: Hướng Dẫn Triển Khai & Vận Hành Toàn Diện

## Mục lục
- [1. Tổng quan về Redis Sentinel](#1-tổng-quan-về-redis-sentinel)
  - [1.1 Redis Sentinel là gì?](#11-redis-sentinel-là-gì)
  - [1.2 Kiến trúc và cách hoạt động](#12-kiến-trúc-và-cách-hoạt-động)
  - [1.3 So sánh với Redis Cluster](#13-so-sánh-với-redis-cluster)
- [2. Triển khai Redis Sentinel](#2-triển-khai-redis-sentinel)
  - [2.1 Yêu cầu và chuẩn bị](#21-yêu-cầu-và-chuẩn-bị)
  - [2.2 Cài đặt Redis](#22-cài-đặt-redis)
  - [2.3 Cấu hình Redis Master-Replica](#23-cấu-hình-redis-master-replica)
  - [2.4 Cấu hình Redis Sentinel](#24-cấu-hình-redis-sentinel)
  - [2.5 Khởi động hệ thống](#25-khởi-động-hệ-thống)
- [3. Cấu hình chi tiết Sentinel](#3-cấu-hình-chi-tiết-sentinel)
  - [3.1 File cấu hình sentinel.conf](#31-file-cấu-hình-sentinelconf)
  - [3.2 Các tham số cấu hình quan trọng](#32-các-tham-số-cấu-hình-quan-trọng)
  - [3.3 Tùy chỉnh cho môi trường Production](#33-tùy-chỉnh-cho-môi-trường-production)
- [4. Quản lý và vận hành](#4-quản-lý-và-vận-hành)
  - [4.1 Giám sát hệ thống Sentinel](#41-giám-sát-hệ-thống-sentinel)
  - [4.2 Xử lý sự cố](#42-xử-lý-sự-cố)
  - [4.3 Bảo trì và nâng cấp](#43-bảo-trì-và-nâng-cấp)
- [5. Kết nối tới Sentinel từ ứng dụng](#5-kết-nối-tới-sentinel-từ-ứng-dụng)
  - [5.1 Redis Clients hỗ trợ Sentinel](#51-redis-clients-hỗ-trợ-sentinel)
  - [5.2 Ví dụ kết nối từ các ngôn ngữ](#52-ví-dụ-kết-nối-từ-các-ngôn-ngữ)
  - [5.3 Best practices](#53-best-practices)
- [6. Tình huống triển khai thực tế](#6-tình-huống-triển-khai-thực-tế)
  - [6.1 Sentinel trong môi trường Docker](#61-sentinel-trong-môi-trường-docker)
  - [6.2 Sentinel với Kubernetes](#62-sentinel-với-kubernetes)
  - [6.3 Sentinel trong Cloud](#63-sentinel-trong-cloud)
- [7. Bảo mật Redis Sentinel](#7-bảo-mật-redis-sentinel)
  - [7.1 Xác thực và mã hóa](#71-xác-thực-và-mã-hóa)
  - [7.2 Phân quyền](#72-phân-quyền)
  - [7.3 Network Security](#73-network-security)
- [8. Nâng cao và tối ưu hóa](#8-nâng-cao-và-tối-ưu-hóa)
  - [8.1 Fine-tuning tham số](#81-fine-tuning-tham-số)
  - [8.2 Mở rộng hệ thống](#82-mở-rộng-hệ-thống)
  - [8.3 Kết hợp với Redis Cluster](#83-kết-hợp-với-redis-cluster)

## 1. Tổng quan về Redis Sentinel

### 1.1 Redis Sentinel là gì?

Redis Sentinel là một hệ thống phân tán được thiết kế để giải quyết các vấn đề về tính sẵn sàng cao (high availability) cho Redis. Nó cung cấp các tính năng sau:

- **Monitoring**: Tự động kiểm tra trạng thái hoạt động của master và replica nodes
- **Notification**: Thông báo cho admin hoặc các ứng dụng khác khi phát hiện sự cố
- **Automatic failover**: Tự động đề xuất và thực hiện việc chuyển đổi replica thành master khi master gặp sự cố
- **Configuration provider**: Đóng vai trò là nguồn cấu hình cho client, giúp client biết địa chỉ của master hiện tại

Sentinel không phải là một giải pháp phân vùng dữ liệu (sharding) như Redis Cluster, mà là một giải pháp đảm bảo tính sẵn sàng cao cho một cặp master-replica thông thường.

### 1.2 Kiến trúc và cách hoạt động

Kiến trúc Redis Sentinel bao gồm:

1. **Redis master**: Node Redis chính, xử lý cả đọc và ghi
2. **Redis replicas**: Các node sao chép dữ liệu từ master, thường chỉ xử lý đọc
3. **Sentinel nodes**: Các tiến trình giám sát hoạt động độc lập với Redis

![Redis Sentinel Architecture](https://i.imgur.com/rxnxC38.png)

**Cách hoạt động**:

1. **Giám sát**: Mỗi Sentinel liên tục ping các Redis instances để kiểm tra tình trạng hoạt động.
2. **Phát hiện sự cố**: Khi một Sentinel phát hiện master không phản hồi, nó sẽ đánh dấu master là "SDOWN" (Subjectively Down).
3. **Xác nhận sự cố**: Sentinel sẽ yêu cầu các Sentinel khác xác nhận tình trạng. Khi đủ số lượng Sentinel xác nhận, master được đánh dấu là "ODOWN" (Objectively Down).
4. **Bầu chọn Leader**: Các Sentinel sẽ bầu chọn một leader (sử dụng thuật toán Raft) để thực hiện failover.
5. **Failover**: Sentinel leader sẽ chọn replica phù hợp nhất để trở thành master mới.
6. **Reconfiguration**: Sentinel sẽ cấu hình lại replica mới thành master, đồng thời cấu hình các replica khác để sao chép từ master mới.
7. **Thông báo**: Sentinel sẽ thông báo cho các clients về thay đổi master.

### 1.3 So sánh với Redis Cluster

| Tính năng | Redis Sentinel | Redis Cluster |
|-----------|----------------|---------------|
| **Mục đích chính** | High availability | Data sharding + High availability |
| **Scale write operations** | Không hỗ trợ | Có (qua sharding) |
| **Tự động failover** | Có | Có |
| **Số lượng nodes tối thiểu** | 3 Sentinels + 1 Master + 1 Replica | 6 nodes (3 masters + 3 replicas) |
| **Phức tạp cài đặt** | Đơn giản | Phức tạp hơn |
| **Phức tạp quản lý** | Trung bình | Cao |
| **Phân vùng dữ liệu** | Không | Có |
| **Cấu hình client** | Cần hỗ trợ Sentinel | Cần hỗ trợ Cluster |

**Khi nào chọn Sentinel:**
- Khi bạn cần high availability nhưng không cần phân vùng dữ liệu
- Khi dữ liệu của bạn có thể lưu trữ trên một Redis instance
- Khi ứng dụng không yêu cầu khả năng mở rộng cao về hiệu suất ghi

**Khi nào chọn Cluster:**
- Khi dữ liệu quá lớn cho một node đơn lẻ
- Khi cần hiệu suất ghi cao qua nhiều node
- Khi cần cả khả năng mở rộng và tính sẵn sàng cao

## 2. Triển khai Redis Sentinel

### 2.1 Yêu cầu và chuẩn bị

**Yêu cầu hệ thống tối thiểu:**
- 3 máy chủ (hoặc VMs) riêng biệt để đảm bảo tính sẵn sàng cao
- Mỗi máy chủ nên có ít nhất:
  - 2 CPU cores
  - 4GB RAM
  - 10GB disk space
- Kết nối mạng ổn định giữa các nodes

**Topo khuyến nghị:**
- 1 Redis master + 2 Redis replicas
- 3 Sentinel nodes (mỗi node trên một máy chủ khác nhau)

**Chuẩn bị:**
- Đảm bảo các servers có thể kết nối với nhau thông qua các ports:
  - 6379: Port mặc định của Redis
  - 26379: Port mặc định của Sentinel
- Cấu hình firewall cho phép kết nối đến các ports trên
- Đồng bộ thời gian giữa các nodes (sử dụng NTP)

### 2.2 Cài đặt Redis

**Trên Ubuntu/Debian:**
```bash
# Cài đặt Redis
apt update
apt install -y redis-server

# Hoặc cài đặt từ source
wget http://download.redis.io/redis-stable.tar.gz
tar xvzf redis-stable.tar.gz
cd redis-stable
make
make install
```

**Trên CentOS/RHEL:**
```bash
# Cài đặt Redis
yum install -y epel-release
yum install -y redis

# Hoặc cài đặt từ source như trên
```

**Kiểm tra cài đặt:**
```bash
redis-server --version
redis-cli --version
```

### 2.3 Cấu hình Redis Master-Replica

Giả sử chúng ta có 3 máy chủ với IP:
- 192.168.1.101 (Master + Sentinel)
- 192.168.1.102 (Replica + Sentinel)
- 192.168.1.103 (Replica + Sentinel)

**Cấu hình Redis Master (192.168.1.101):**
```bash
# Chỉnh sửa /etc/redis/redis.conf hoặc /etc/redis.conf
bind 0.0.0.0
port 6379
daemonize yes
pidfile /var/run/redis/redis-server.pid
logfile /var/log/redis/redis-server.log

# Bảo mật
requirepass StrongPassword123
masterauth StrongPassword123

# Persistence
dir /var/lib/redis
appendonly yes
appendfilename "appendonly.aof"
```

**Cấu hình Redis Replica (192.168.1.102 và 192.168.1.103):**
```bash
# Chỉnh sửa /etc/redis/redis.conf hoặc /etc/redis.conf
bind 0.0.0.0
port 6379
daemonize yes
pidfile /var/run/redis/redis-server.pid
logfile /var/log/redis/redis-server.log

# Cấu hình replica
replicaof 192.168.1.101 6379
masterauth StrongPassword123

# Bảo mật
requirepass StrongPassword123

# Persistence
dir /var/lib/redis
appendonly yes
appendfilename "appendonly.aof"
```

**Lưu ý quan trọng:**
- Tham số `requirepass` đặt mật khẩu cho Redis server
- Tham số `masterauth` cho phép replica xác thực với master
- Tham số `replicaof` (hoặc `slaveof` trong các phiên bản cũ) chỉ định địa chỉ IP và port của master

### 2.4 Cấu hình Redis Sentinel

**Tạo file cấu hình sentinel.conf trên cả 3 máy chủ:**

```bash
# Tạo thư mục cho Sentinel
mkdir -p /etc/redis-sentinel
touch /etc/redis-sentinel/sentinel.conf
```

**Nội dung cấu hình cơ bản cho sentinel.conf:**
```
# Cấu hình cơ bản
port 26379
daemonize yes
pidfile "/var/run/redis/redis-sentinel.pid"
logfile "/var/log/redis/redis-sentinel.log"
dir "/var/lib/redis"

# Cấu hình monitor
sentinel monitor mymaster 192.168.1.101 6379 2
sentinel auth-pass mymaster StrongPassword123

# Cấu hình failover
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1

# Bảo vệ Sentinel
sentinel deny-scripts-reconfig yes

# Thông báo
# sentinel notification-script mymaster /var/redis/notify.sh
# sentinel client-reconfig-script mymaster /var/redis/reconfig.sh
```

**Giải thích các tham số cấu hình:**
- `sentinel monitor mymaster 192.168.1.101 6379 2`: Giám sát master với IP 192.168.1.101, port 6379, cần ít nhất 2 Sentinel đồng ý để thực hiện failover
- `sentinel auth-pass mymaster StrongPassword123`: Mật khẩu để kết nối với master
- `sentinel down-after-milliseconds mymaster 5000`: Thời gian không phản hồi (5 giây) trước khi coi master là down
- `sentinel failover-timeout mymaster 60000`: Thời gian tối đa (60 giây) cho một quy trình failover
- `sentinel parallel-syncs mymaster 1`: Số lượng replicas được cấu hình lại đồng thời trong quá trình failover

### 2.5 Khởi động hệ thống

**Khởi động Redis trên tất cả các nodes:**
```bash
# Khởi động Redis
systemctl start redis-server
# Hoặc
service redis-server start
# Hoặc (nếu không dùng systemd)
redis-server /etc/redis/redis.conf

# Kiểm tra trạng thái
systemctl status redis-server
```

**Tạo service cho Sentinel:**
```bash
# Tạo file systemd service
cat > /etc/systemd/system/redis-sentinel.service << EOF
[Unit]
Description=Redis Sentinel
After=network.target

[Service]
ExecStart=/usr/bin/redis-sentinel /etc/redis-sentinel/sentinel.conf
Restart=always
User=redis
Group=redis

[Install]
WantedBy=multi-user.target
EOF

# Reload systemd
systemctl daemon-reload
```

**Khởi động Sentinel trên tất cả các nodes:**
```bash
# Khởi động Sentinel
systemctl start redis-sentinel
# Hoặc
service redis-sentinel start
# Hoặc (nếu không dùng systemd)
redis-sentinel /etc/redis-sentinel/sentinel.conf

# Kiểm tra trạng thái
systemctl status redis-sentinel
```

**Kiểm tra hệ thống:**
```bash
# Kiểm tra master-replica
redis-cli -h 192.168.1.101 -a StrongPassword123 info replication

# Kiểm tra Sentinel
redis-cli -p 26379 sentinel master mymaster
redis-cli -p 26379 sentinel slaves mymaster
redis-cli -p 26379 sentinel sentinels mymaster
```

## 3. Cấu hình chi tiết Sentinel

### 3.1 File cấu hình sentinel.conf

File cấu hình sentinel.conf có hai phần chính:
1. **Cấu hình tĩnh**: Những tham số bạn đặt trực tiếp
2. **Cấu hình động**: Những tham số được Sentinel tự động thêm vào trong quá trình hoạt động

**Ví dụ cấu hình đầy đủ hơn:**
```
# Cấu hình mạng
port 26379
bind 0.0.0.0
protected-mode yes

# Cấu hình chung
daemonize yes
pidfile "/var/run/redis/redis-sentinel.pid"
logfile "/var/log/redis/redis-sentinel.log"
dir "/var/lib/redis"

# Cấu hình Sentinel
sentinel monitor mymaster 192.168.1.101 6379 2
sentinel auth-pass mymaster StrongPassword123
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1

# Tuỳ chọn thông báo và script
sentinel notification-script mymaster /opt/redis/notify.sh
sentinel client-reconfig-script mymaster /opt/redis/reconfig.sh

# Bảo mật
sentinel deny-scripts-reconfig yes

# Announce
sentinel announce-ip 192.168.1.101
sentinel announce-port 26379
```

**Lưu ý:** Sau khi Sentinel đã chạy, nó sẽ tự động thêm các tham số cấu hình động vào file sentinel.conf. Không nên chỉnh sửa thủ công file cấu hình sau khi Sentinel đã chạy, trừ khi bạn đã dừng hoàn toàn tất cả các instances của Sentinel.

### 3.2 Các tham số cấu hình quan trọng

**Monitoring và Detection:**
- `sentinel monitor <master-name> <ip> <port> <quorum>`: Xác định master cần giám sát và số lượng Sentinel tối thiểu cần thiết để xác nhận sự cố
- `sentinel down-after-milliseconds <master-name> <milliseconds>`: Thời gian tối đa master không phản hồi trước khi bị coi là down
- `sentinel auth-pass <master-name> <password>`: Mật khẩu để kết nối với master

**Failover configs:**
- `sentinel failover-timeout <master-name> <milliseconds>`: Thời gian tối đa cho một quy trình failover
- `sentinel parallel-syncs <master-name> <numreplicas>`: Số lượng replicas có thể được đồng bộ đồng thời sau failover
- `sentinel min-replicas-to-write <master-name> <count>`: Nếu số lượng replicas còn lại nhỏ hơn count, master sẽ từ chối ghi
- `sentinel min-replicas-max-lag <master-name> <seconds>`: Độ trễ tối đa cho phép giữa master và replica

**Script configs:**
- `sentinel notification-script <master-name> <script-path>`: Script sẽ chạy khi có sự kiện xảy ra
- `sentinel client-reconfig-script <master-name> <script-path>`: Script chạy khi có failover
- `sentinel deny-scripts-reconfig yes|no`: Ngăn chặn việc cấu hình lại các script qua API

**Network configs:**
- `sentinel announce-ip <ip>`: Địa chỉ IP được thông báo cho các Sentinel khác
- `sentinel announce-port <port>`: Port được thông báo cho các Sentinel khác

### 3.3 Tùy chỉnh cho môi trường Production

**Cấu hình quorum tối ưu:**
- Với 3 Sentinel: quorum = 2
- Với 5 Sentinel: quorum = 3
- Với 7 Sentinel: quorum = 4

```
sentinel monitor mymaster 192.168.1.101 6379 2
```

**Điều chỉnh thời gian phát hiện và failover:**
```
# Phát hiện sự cố nhanh hơn (3 giây)
sentinel down-after-milliseconds mymaster 3000

# Giảm thời gian failover
sentinel failover-timeout mymaster 30000
```

**Cấu hình thông báo:**
```
# Script thông báo (ví dụ gửi email hoặc Slack)
sentinel notification-script mymaster /opt/redis/notify.sh

# Nội dung file notify.sh
#!/bin/bash
# $1: Loại sự kiện
# $2: Loại instance (master/replica)
# $3: Tên instance (mymaster)
# $4: IP cũ
# $5: Port cũ
# $6: IP mới
# $7: Port mới

echo "Redis event: $1 $2 $3 $4:$5 -> $6:$7" | mail -s "Redis Sentinel Alert" admin@example.com
```

**Cấu hình client-reconfig-script:**
```
sentinel client-reconfig-script mymaster /opt/redis/reconfig.sh

# Nội dung file reconfig.sh
#!/bin/bash
# $1: IP master cũ
# $2: Port master cũ
# $3: IP master mới
# $4: Port master mới
# $5: Trạng thái master mới (failover hoặc switch-master)
# $6: Tên master (mymaster)

# Cập nhật cấu hình DNS hoặc Load Balancer
curl -X POST "http://admin:password@haproxy-admin/api/setmaster?ip=$3&port=$4"
```

**Cấu hình bảo mật nâng cao:**
```
# Bảo vệ Sentinel không cho phép scripts tự động reconfigure
sentinel deny-scripts-reconfig yes

# Đảm bảo kết nối an toàn giữa Sentinel và Redis
sentinel auth-pass mymaster StrongPassword123
```

## 4. Quản lý và vận hành

### 4.1 Giám sát hệ thống Sentinel

**Kiểm tra trạng thái của cụm Sentinel:**
```bash
# Kiểm tra thông tin master
redis-cli -p 26379 SENTINEL master mymaster

# Kiểm tra danh sách replicas
redis-cli -p 26379 SENTINEL replicas mymaster

# Kiểm tra danh sách Sentinel
redis-cli -p 26379 SENTINEL sentinels mymaster

# Kiểm tra trạng thái chung của Sentinel
redis-cli -p 26379 SENTINEL info-cache mymaster
```

**Kiểm tra replication trên master:**
```bash
redis-cli -h <master-ip> -a <password> INFO replication
```

**Kiểm tra logs:**
```bash
# Xem Sentinel logs
tail -f /var/log/redis/redis-sentinel.log

# Xem Redis logs
tail -f /var/log/redis/redis-server.log
```

**Giám sát chỉ số với Redis INFO:**
```bash
# Trên Master/Replica
redis-cli -h <node-ip> -a <password> INFO stats
redis-cli -h <node-ip> -a <password> INFO clients
redis-cli -h <node-ip> -a <password> INFO memory
```

**Thiết lập Prometheus/Grafana monitoring:**

1. Cài đặt Redis Exporter:
```bash
wget https://github.com/oliver006/redis_exporter/releases/download/v1.31.4/redis_exporter-v1.31.4.linux-amd64.tar.gz
tar xvf redis_exporter-v1.31.4.linux-amd64.tar.gz
cd redis_exporter-v1.31.4.linux-amd64
./redis_exporter -redis.addr=redis://:<password>@<host>:6379
```

2. Cấu hình Prometheus scrape Redis Exporter metrics:
```yaml
scrape_configs:
  - job_name: 'redis'
    static_configs:
      - targets: ['<redis-exporter-host>:9121']
```

3. Import Redis Dashboard cho Grafana (Dashboard ID: 763)

### 4.2 Xử lý sự cố

**Kiểm tra failover thủ công:**
```bash
redis-cli -p 26379 SENTINEL failover mymaster
```

**Xử lý split-brain (nhiều master):**
```bash
# Kiểm tra tất cả các Sentinel để xác định master thực sự
for i in 101 102 103; do echo "Checking 192.168.1.$i:"; redis-cli -h 192.168.1.$i -p 26379 SENTINEL master mymaster | grep ip; done

# Nếu phát hiện nhiều master, reset quorum trên tất cả các Sentinel
for i in 101 102 103; do redis-cli -h 192.168.1.$i -p 26379 SENTINEL reset mymaster; done
```

**Sửa chữa khi Sentinel mất đồng bộ:**
```bash
# Đặt lại thông tin master
redis-cli -p 26379 SENTINEL REMOVE mymaster
redis-cli -p 26379 SENTINEL MONITOR mymaster <ip> <port> <quorum>
redis-cli -p 26379 SENTINEL SET mymaster auth-pass StrongPassword123
redis-cli -p 26379 SENTINEL SET mymaster down-after-milliseconds 5000
```

**Khôi phục sau khi master cũ (đã fail) trở lại:**
```bash
# Kết nối đến master cũ và cấu hình lại thành replica
redis-cli -h <old-master-ip> -a <password>
> REPLICAOF <new-master-ip> 6379
```

**Xử lý khi Sentinel không thể thực hiện failover:**
1. Kiểm tra logs để xác định nguyên nhân
2. Kiểm tra quorum - có đủ số lượng Sentinel hoạt động không?
3. Kiểm tra kết nối mạng giữa các node
4. Kiểm tra bảo mật - mật khẩu và quyền truy cập có đúng không?

### 4.3 Bảo trì và nâng cấp

**Quy trình bảo trì node Sentinel:**
```bash
# 1. Kiểm tra trạng thái trước khi bảo trì
redis-cli -p 26379 SENTINEL master mymaster

# 2. Dừng Sentinel
systemctl stop redis-sentinel

# 3. Thực hiện bảo trì (cập nhật, sửa cấu hình, v.v.)

# 4. Khởi động lại Sentinel
systemctl start redis-sentinel

# 5. Kiểm tra trạng thái sau khi bảo trì
redis-cli -p 26379 SENTINEL master mymaster
```

**Quy trình bảo trì Redis replica:**
```bash
# 1. Kiểm tra replication trước khi bảo trì
redis-cli -h <master-ip> -a <password> INFO replication

# 2. Dừng Redis replica
systemctl stop redis-server

# 3. Thực hiện bảo trì

# 4. Khởi động lại Redis replica
systemctl start redis-server

# 5. Kiểm tra replication sau khi bảo trì
redis-cli -h <replica-ip> -a <password> INFO replication
```

**Quy trình bảo trì Redis master:**
```bash
# 1. Thực hiện manual failover để chuyển master sang node khác
redis-cli -p 26379 SENTINEL failover mymaster

# 2. Kiểm tra trạng thái mới
redis-cli -p 26379 SENTINEL master mymaster

# 3. Đảm bảo failover đã hoàn tất
sleep 10

# 4. Dừng Redis master cũ (hiện là replica)
systemctl stop redis-server

# 5. Thực hiện bảo trì

# 6. Khởi động lại Redis
systemctl start redis-server
```

**Nâng cấp Redis version:**
```bash
# Nâng cấp từng node một, bắt đầu từ replicas

# 1. Nâng cấp replica
# Dừng replica
systemctl stop redis-server

# Cài đặt phiên bản mới
apt update && apt install -y redis-server
# hoặc nâng cấp từ source

# Kiểm tra phiên bản mới
redis-server --version

# Khởi động lại replica
systemctl start redis-server

# 2. Failover và nâng cấp master cũ
redis-cli -p 26379 SENTINEL failover mymaster
# Đợi failover hoàn tất rồi nâng cấp master cũ (như bước 1)

# 3. Nâng cấp Sentinel
# Nâng cấp từng node một
systemctl stop redis-sentinel
# Cài đặt phiên bản mới
systemctl start redis-sentinel
```

## 5. Kết nối tới Sentinel từ ứng dụng

### 5.1 Redis Clients hỗ trợ Sentinel

Các Redis client phổ biến hỗ trợ Sentinel:

**Node.js**:
- `ioredis` - Hỗ trợ đầy đủ Sentinel
- `node-redis` (phiên bản mới) - Cơ bản tích hợp Sentinel

**Python**:
- `redis-py` - Hỗ trợ Sentinel

**Java**:
- `Jedis` - Hỗ trợ Sentinel
- `Lettuce` - Hỗ trợ Sentinel
- `Redisson` - Hỗ trợ Sentinel

**Ruby**:
- `redis-rb` - Hỗ trợ Sentinel

**PHP**:
- `Predis` - Hỗ trợ Sentinel
- `PhpRedis` - Hỗ trợ Sentinel

**Go**:
- `go-redis` - Hỗ trợ Sentinel
- `redigo` + add-on - Có thể tích hợp Sentinel

**C#/.NET**:
- `StackExchange.Redis` - Hỗ trợ Sentinel

### 5.2 Ví dụ kết nối từ các ngôn ngữ

**Node.js (ioredis):**
```javascript
const Redis = require('ioredis');

const redis = new Redis({
  sentinels: [
    { host: '192.168.1.101', port: 26379 },
    { host: '192.168.1.102', port: 26379 },
    { host: '192.168.1.103', port: 26379 }
  ],
  name: 'mymaster',
  password: 'StrongPassword123',
  db: 0
});

redis.set('key', 'value', (err) => {
  if (err) {
    console.error('Error:', err);
  } else {
    console.log('Key set successfully');
  }
});
```

**Python:**
```python
from redis.sentinel import Sentinel

sentinel = Sentinel([
    ('192.168.1.101', 26379),
    ('192.168.1.102', 26379),
    ('192.168.1.103', 26379)
], socket_timeout=0.5)

# Get master
master = sentinel.master_for('mymaster', password='StrongPassword123', db=0)
master.set('key', 'value')

# Get replica
replica = sentinel.slave_for('mymaster', password='StrongPassword123', db=0)
value = replica.get('key')
print(value)
```

**Java (Jedis):**
```java
import redis.clients.jedis.*;

Set<String> sentinels = new HashSet<>();
sentinels.add("192.168.1.101:26379");
sentinels.add("192.168.1.102:26379");
sentinels.add("192.168.1.103:26379");

JedisSentinelPool pool = new JedisSentinelPool("mymaster", sentinels, 
    new JedisPoolConfig(), 2000, "StrongPassword123", 0);

try (Jedis jedis = pool.getResource()) {
    jedis.set("key", "value");
    String value = jedis.get("key");
    System.out.println(value);
}
```

**PHP (Predis):**
```php
<?php
require 'vendor/autoload.php';

$sentinels = [
    'tcp://192.168.1.101:26379',
    'tcp://192.168.1.102:26379',
    'tcp://192.168.1.103:26379',
];

$options = [
    'replication' => 'sentinel',
    'service' => 'mymaster',
    'parameters' => [
        'password' => 'StrongPassword123',
        'database' => 0,
    ]
];

$client = new Predis\Client($sentinels, $options);
$client->set('key', 'value');
$value = $client->get('key');
echo $value;
```

**Go (go-redis):**
```go
package main

import (
    "fmt"
    "github.com/go-redis/redis/v8"
    "context"
)

func main() {
    ctx := context.Background()
    
    client := redis.NewFailoverClient(&redis.FailoverOptions{
        MasterName:    "mymaster",
        SentinelAddrs: []string{"192.168.1.101:26379", "192.168.1.102:26379", "192.168.1.103:26379"},
        Password:      "StrongPassword123",
        DB:            0,
    })
    
    err := client.Set(ctx, "key", "value", 0).Err()
    if err != nil {
        panic(err)
    }
    
    val, err := client.Get(ctx, "key").Result()
    if err != nil {
        panic(err)
    }
    fmt.Println("key", val)
}
```

### 5.3 Best practices

**Xử lý kết nối:**
1. **Kết nối đến nhiều Sentinel**: Luôn cung cấp địa chỉ của tất cả các Sentinel.
2. **Retry mechanism**: Cài đặt cơ chế retry khi kết nối bị mất.
3. **Connection pooling**: Sử dụng connection pool để quản lý kết nối hiệu quả.

**Mẫu thiết kế cho ứng dụng:**
```
+-----------------+
| Application     |
+-----------------+
        |
+-------------------------------+
| Redis Client with Sentinel    |
| Support                       |
+-------------------------------+
        |
        v
+------------------+      +------------------+       +------------------+
| Sentinel Node 1  | <--> | Sentinel Node 2  |  <--> | Sentinel Node 3  |
+------------------+      +------------------+       +------------------+
        |                       |                          |
        v                       v                          v
+------------------+      +------------------+       +------------------+
| Redis Master     | <--- | Redis Replica 1  |  <--- | Redis Replica 2  |
+------------------+      +------------------+       +------------------+
```

**Xử lý failover trong ứng dụng:**
```javascript
// Node.js example with ioredis
const Redis = require('ioredis');

const redis = new Redis({
  sentinels: [
    { host: '192.168.1.101', port: 26379 },
    { host: '192.168.1.102', port: 26379 },
    { host: '192.168.1.103', port: 26379 }
  ],
  name: 'mymaster',
  password: 'StrongPassword123',
  db: 0,
  reconnectOnError: function(err) {
    const targetError = 'READONLY';
    if (err.message.includes(targetError)) {
      // Khi gặp lỗi READONLY, thử kết nối lại
      return true;
    }
  }
});

redis.on('error', function(error) {
  console.error('Redis error:', error);
});

redis.on('+switch-master', function(channel, message) {
  console.log('Master switched:', message);
});
```

**Read-Write splitting:**
```python
# Python example
from redis.sentinel import Sentinel

sentinel = Sentinel([
    ('192.168.1.101', 26379),
    ('192.168.1.102', 26379),
    ('192.168.1.103', 26379)
], socket_timeout=0.5)

# For writes
master = sentinel.master_for('mymaster', password='StrongPassword123', db=0)

# For reads
replica = sentinel.slave_for('mymaster', password='StrongPassword123', db=0)

# Write operation
master.set('key', 'value')

# Read operation (from replica)
value = replica.get('key')
```

## 6. Tình huống triển khai thực tế

### 6.1 Sentinel trong môi trường Docker

**Docker Compose setup:**

```yaml
version: '3'

services:
  redis-master:
    image: redis:7
    container_name: redis-master
    volumes:
      - ./redis-master.conf:/usr/local/etc/redis/redis.conf
    command: redis-server /usr/local/etc/redis/redis.conf
    networks:
      - redis-net
    ports:
      - "6379:6379"

  redis-replica-1:
    image: redis:7
    container_name: redis-replica-1
    volumes:
      - ./redis-replica-1.conf:/usr/local/etc/redis/redis.conf
    command: redis-server /usr/local/etc/redis/redis.conf
    networks:
      - redis-net
    ports:
      - "6380:6379"
    depends_on:
      - redis-master

  redis-replica-2:
    image: redis:7
    container_name: redis-replica-2
    volumes:
      - ./redis-replica-2.conf:/usr/local/etc/redis/redis.conf
    command: redis-server /usr/local/etc/redis/redis.conf
    networks:
      - redis-net
    ports:
      - "6381:6379"
    depends_on:
      - redis-master

  sentinel-1:
    image: redis:7
    container_name: redis-sentinel-1
    volumes:
      - ./sentinel-1.conf:/usr/local/etc/redis/sentinel.conf
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    networks:
      - redis-net
    ports:
      - "26379:26379"
    depends_on:
      - redis-master
      - redis-replica-1
      - redis-replica-2

  sentinel-2:
    image: redis:7
    container_name: redis-sentinel-2
    volumes:
      - ./sentinel-2.conf:/usr/local/etc/redis/sentinel.conf
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    networks:
      - redis-net
    ports:
      - "26380:26379"
    depends_on:
      - redis-master
      - redis-replica-1
      - redis-replica-2

  sentinel-3:
    image: redis:7
    container_name: redis-sentinel-3
    volumes:
      - ./sentinel-3.conf:/usr/local/etc/redis/sentinel.conf
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    networks:
      - redis-net
    ports:
      - "26381:26379"
    depends_on:
      - redis-master
      - redis-replica-1
      - redis-replica-2

networks:
  redis-net:
    driver: bridge
```

**File cấu hình redis-master.conf:**
```
port 6379
bind 0.0.0.0
requirepass StrongPassword123
masterauth StrongPassword123
appendonly yes
```

**File cấu hình redis-replica-1.conf và redis-replica-2.conf:**
```
port 6379
bind 0.0.0.0
replicaof redis-master 6379
masterauth StrongPassword123
requirepass StrongPassword123
appendonly yes
```

**File cấu hình sentinel-1.conf (tương tự cho sentinel-2.conf và sentinel-3.conf):**
```
port 26379
bind 0.0.0.0
sentinel monitor mymaster redis-master 6379 2
sentinel auth-pass mymaster StrongPassword123
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1
sentinel announce-ip sentinel-1
sentinel announce-port 26379
```

**Lưu ý quan trọng cho Docker:**
1. Cần sử dụng `sentinel announce-ip` và `sentinel announce-port` để Sentinel có thể thông báo địa chỉ đúng
2. Service discovery trong Docker cần được xử lý cẩn thận để đảm bảo các container có thể tìm thấy nhau
3. Nên sử dụng volume để lưu dữ liệu Redis bên ngoài container
4. Xem xét sử dụng Docker Swarm hoặc Kubernetes cho môi trường production

### 6.2 Sentinel với Kubernetes

**StatefulSet setup cho Redis:**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  serviceName: redis
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7
        ports:
        - containerPort: 6379
        command:
        - bash
        - "-c"
        - |
          if [[ ${HOSTNAME} == "redis-0" ]]; then
            redis-server --bind 0.0.0.0 --port 6379 --requirepass ${REDIS_PASSWORD} --masterauth ${REDIS_PASSWORD} --appendonly yes
          else
            redis-server --bind 0.0.0.0 --port 6379 --replicaof redis-0.redis 6379 --requirepass ${REDIS_PASSWORD} --masterauth ${REDIS_PASSWORD} --appendonly yes
          fi
        env:
        - name: REDIS_PASSWORD
          valueFrom:
            secretKeyRef:
              name: redis-secret
              key: password
        volumeMounts:
        - name: redis-data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: redis-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

**Deployment cho Sentinel:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-sentinel
spec:
  replicas: 3
  selector:
    matchLabels:
      app: redis-sentinel
  template:
    metadata:
      labels:
        app: redis-sentinel
    spec:
      containers:
      - name: sentinel
        image: redis:7
        ports:
        - containerPort: 26379
        command:
        - bash
        - "-c"
        - |
          cat > /tmp/sentinel.conf <<EOF
          port 26379
          bind 0.0.0.0
          sentinel monitor mymaster redis-0.redis 6379 2
          sentinel auth-pass mymaster ${REDIS_PASSWORD}
          sentinel down-after-milliseconds mymaster 5000
          sentinel failover-timeout mymaster 60000
          sentinel parallel-syncs mymaster 1
          EOF
          redis-sentinel /tmp/sentinel.conf
        env:
        - name: REDIS_PASSWORD
          valueFrom:
            secretKeyRef:
              name: redis-secret
              key: password
```

**Services cho Redis và Sentinel:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  selector:
    app: redis
  clusterIP: None
  ports:
  - port: 6379
    targetPort: 6379
---
apiVersion: v1
kind: Service
metadata:
  name: redis-sentinel
spec:
  selector:
    app: redis-sentinel
  ports:
  - port: 26379
    targetPort: 26379
```

**Secret cho mật khẩu Redis:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: redis-secret
type: Opaque
data:
  password: U3Ryb25nUGFzc3dvcmQxMjM=  # base64 encoded "StrongPassword123"
```

**Lưu ý cho Kubernetes:**
1. Sử dụng StatefulSet cho Redis để có tên host ổn định
2. Cần cấu hình thích hợp `PersistentVolumes` cho lưu trữ dữ liệu
3. Sử dụng Kubernetes Secrets để quản lý mật khẩu
4. Cân nhắc sử dụng Helm chart có sẵn cho Redis Sentinel

### 6.3 Sentinel trong Cloud

**AWS ElastiCache Redis (không hỗ trợ Sentinel trực tiếp):**
- AWS ElastiCache cung cấp tính năng Multi-AZ với auto-failover tương tự như Sentinel
- Sử dụng ElastiCache Redis với Replicas và Auto Failover
- Các service endpoints do AWS quản lý sẽ tự động chuyển hướng đến node master

**Giải pháp thay thế trên AWS:**
- Tự triển khai Redis Sentinel trên EC2
- Sử dụng AWS Auto Scaling Groups để quản lý các Redis nodes
- Kết hợp với Route53 Health Checks để routing DNS

**Google Cloud Memorystore:**
- Dịch vụ Redis có sẵn với tính năng High Availability và Auto Failover

**Azure Cache for Redis:**
- Hỗ trợ cấu hình Redis với replica và auto-failover

**Tự triển khai trên Cloud VMs:**
1. Tạo VMs ở các availability zones khác nhau
2. Cài đặt Redis và Sentinel như hướng dẫn trước đó
3. Cấu hình VPC và Security Groups phù hợp
4. Sử dụng Load Balancer của cloud provider để cung cấp endpoint ổn định

**Cấu hình Firewall cho Redis Sentinel trên Cloud:**
```
Inbound rules:
- TCP 6379: Redis port (từ app servers)
- TCP 26379: Sentinel port (từ app servers và các Sentinel khác)
- TCP 16379: Redis cluster bus (giữa các Redis nodes)
```

## 7. Bảo mật Redis Sentinel

### 7.1 Xác thực và mã hóa

**Bảo mật Redis với mật khẩu:**
```
# Trong redis.conf
requirepass StrongPasswordHere
masterauth StrongPasswordHere

# Trong sentinel.conf
sentinel auth-pass mymaster StrongPasswordHere
```

**Bật TLS cho Redis (Redis 6.0+):**
```
# Trong redis.conf
tls-port 6380
tls-cert-file /path/to/redis.crt
tls-key-file /path/to/redis.key
tls-ca-cert-file /path/to/ca.crt
tls-auth-clients yes
tls-replication yes
```

**Cấu hình TLS cho Sentinel (Redis 6.0+):**
```
# Trong sentinel.conf
tls-port 26380
tls-cert-file /path/to/sentinel.crt
tls-key-file /path/to/sentinel.key
tls-ca-cert-file /path/to/ca.crt
tls-replication yes
```

**Tạo certificates:**
```bash
# Tạo CA key và certificate
openssl genrsa -out ca.key 4096
openssl req -x509 -new -nodes -sha256 -key ca.key -days 3650 -out ca.crt -subj "/C=US/ST=State/L=City/O=Organization/CN=Redis CA"

# Tạo Redis server key và CSR
openssl genrsa -out redis.key 2048
openssl req -new -key redis.key -out redis.csr -subj "/C=US/ST=State/L=City/O=Organization/CN=Redis Server"

# Ký CSR
openssl x509 -req -sha256 -in redis.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out redis.crt -days 365
```

### 7.2 Phân quyền

**Redis ACL (Redis 6.0+):**
```
# Trong redis.conf
user default off
user admin on >StrongPassword123 +@all ~*
user replication on >ReplicationPassword +@replication +psync +replconf +ping
user sentinel on >SentinelPassword +@sentinel +@connection +@admin
```

**Sử dụng ACL với Sentinel:**
```
# Trong sentinel.conf
sentinel auth-user mymaster sentinel
sentinel auth-pass mymaster SentinelPassword
```

**Bảo vệ commands:**
```
# Vô hiệu hóa các lệnh nguy hiểm
# Trong redis.conf
rename-command FLUSHALL ""
rename-command FLUSHDB ""
rename-command CONFIG ""
rename-command SHUTDOWN ""
```

### 7.3 Network Security

**Firewall Rules:**
```bash
# Chỉ mở port cần thiết
iptables -A INPUT -p tcp --dport 6379 -s 192.168.1.0/24 -j ACCEPT
iptables -A INPUT -p tcp --dport 26379 -s 192.168.1.0/24 -j ACCEPT
iptables -A INPUT -p tcp --dport 6379 -j DROP
iptables -A INPUT -p tcp --dport 26379 -j DROP
```

**Redis bind address:**
```
# Trong redis.conf
bind 127.0.0.1 192.168.1.101
protected-mode yes
```

**Sentinel bind address:**
```
# Trong sentinel.conf
bind 127.0.0.1 192.168.1.101
protected-mode yes
```

**Sử dụng VPC và Subnets:**
- Đặt Redis và Sentinel nodes trong mạng private
- Sử dụng VPN hoặc bastion host để truy cập quản trị
- Cấu hình Security Groups để chỉ cho phép traffic giữa các nodes liên quan

## 8. Nâng cao và tối ưu hóa

### 8.1 Fine-tuning tham số

**Tham số Sentinel nâng cao:**
```
# Trong sentinel.conf

# Đảm bảo đủ thời gian để phát hiện lỗi nhưng không quá nhạy
sentinel down-after-milliseconds mymaster 5000

# Tối ưu thời gian giải quyết sự cố
sentinel failover-timeout mymaster 60000

# Tránh quá tải network khi failover bằng cách giới hạn số lượng replicas được đồng bộ cùng lúc
sentinel parallel-syncs mymaster 1

# Thiết lập min-slaves
sentinel min-replicas-to-write mymaster 1
sentinel min-replicas-max-lag mymaster 10
```

**Redis tuning:**
```
# Trong redis.conf

# Tối ưu persistence
appendonly yes
appendfsync everysec
no-appendfsync-on-rewrite yes
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# Tối ưu bộ nhớ
maxmemory 4gb
maxmemory-policy volatile-lru
```

**Tối ưu OS:**
```bash
# Tăng giới hạn file descriptors
echo "redis soft nofile 65535" >> /etc/security/limits.conf
echo "redis hard nofile 65535" >> /etc/security/limits.conf

# Tắt transparent huge pages
echo never > /sys/kernel/mm/transparent_hugepage/enabled

# Tối ưu swap
sysctl vm.swappiness=10

# Tối ưu network
sysctl net.core.somaxconn=65535
sysctl net.ipv4.tcp_max_syn_backlog=65535
```

### 8.2 Mở rộng hệ thống

**Thêm Replica vào hệ thống:**
1. Cài đặt và cấu hình Redis trên node mới
2. Cấu hình replica để connect đến master
3. Kiểm tra trạng thái replication
4. Cập nhật Sentinel nếu cần

```bash
# Kiểm tra master hiện tại
redis-cli -p 26379 SENTINEL get-master-addr-by-name mymaster

# Cấu hình replica mới
vi /etc/redis/redis.conf
# Thêm:
replicaof <master-ip> 6379
masterauth <password>

# Khởi động Redis trên replica mới
systemctl start redis-server

# Kiểm tra trạng thái
redis-cli -h <master-ip> -a <password> INFO replication
```

**Thêm Sentinel node:**
1. Cài đặt Redis trên node mới
2. Tạo cấu hình sentinel.conf cơ bản
3. Khởi động Sentinel
4. Sentinel sẽ tự động phát hiện và đồng bộ cấu hình

```bash
# Cấu hình Sentinel mới
cat > /etc/redis-sentinel/sentinel.conf << EOF
port 26379
bind 0.0.0.0
sentinel monitor mymaster <master-ip> 6379 <quorum>
sentinel auth-pass mymaster <password>
EOF

# Khởi động Sentinel
systemctl start redis-sentinel

# Kiểm tra trạng thái
redis-cli -p 26379 SENTINEL master mymaster
```

**Mở rộng sang nhiều datacenter:**
- Đảm bảo số lượng Sentinel ở mỗi datacenter đủ nhiều
- Đặt quorum thích hợp để tránh split-brain
- Cân nhắc độ trễ mạng giữa các datacenter

### 8.3 Kết hợp với Redis Cluster

**Redis Sentinel và Redis Cluster:**
- Redis Sentinel: high availability cho 1 master
- Redis Cluster: sharding và high availability cho nhiều masters

**Kết hợp cả hai:**
1. Sử dụng Redis Cluster để phân chia dữ liệu
2. Mỗi master node trong cluster có ít nhất 1 replica
3. Cluster tự xử lý failover, không cần Sentinel

**Hoặc nâng cấp từ Sentinel lên Cluster:**
1. Backup dữ liệu từ hệ thống Sentinel
2. Cài đặt Redis Cluster mới
3. Khôi phục dữ liệu vào Cluster
4. Cập nhật ứng dụng để sử dụng Redis Cluster thay vì Sentinel

**Lựa chọn giữa Sentinel và Cluster:**
- Kích thước dữ liệu < 4-8GB: Sentinel có thể là đủ
- Kích thước dữ liệu lớn: Cluster là lựa chọn tốt hơn
- Hiệu suất ghi cao: Cluster giúp phân tán tải
- Độ phức tạp: Sentinel đơn giản hơn để quản lý

---

Tài liệu này cung cấp một hướng dẫn toàn diện về Redis Sentinel từ cài đặt đến vận hành và tối ưu hóa. Tùy thuộc vào nhu cầu cụ thể, bạn có thể cần điều chỉnh các tham số và cấu hình để phù hợp với môi trường của mình.
