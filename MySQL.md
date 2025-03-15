# Hướng Dẫn Chi Tiết Tuning MySQL

## Mục lục
- [1. Cấu hình quan trọng trong `my.cnf`](#1-cấu-hình-quan-trọng-trong-mycnf)
  - [1.1 Bộ nhớ và Cache](#11-bộ-nhớ-và-cache)
  - [1.2 Quản lý Kết Nối và Hiệu Năng](#12-quản-lý-kết-nối-và-hiệu-năng)
- [2. Tối ưu Hệ thống Lưu trữ (Storage Engine)](#2-tối-ưu-hệ-thống-lưu-trữ-storage-engine)
- [3. Cải thiện hiệu suất truy vấn (SQL Performance)](#3-cải-thiện-hiệu-suất-truy-vấn-sql-performance)
- [4. Tối ưu Bản ghi và Bảng](#4-tối-ưu-bản-ghi-và-bảng)
- [5. Cấu hình Replica (Replication)](#5-cấu-hình-replica-replication)
- [6. Giám sát và Bảo trì](#6-giám-sát-và-bảo-trì)
- [7. Backup & Restore](#7-backup--restore)
- [8. Quy trình tuning MySQL hiệu quả](#8-quy-trình-tuning-mysql-hiệu-quả)

---

## 1. Cấu hình quan trọng trong `my.cnf`

### 1.1 Bộ nhớ và Cache

#### InnoDB Buffer Pool Size
```ini
[mysqld]
innodb_buffer_pool_size = 4G
innodb_buffer_pool_instances = 8
```

**Hướng dẫn chi tiết:**
- **Vai trò**: Buffer pool là bộ nhớ đệm chính của InnoDB, lưu trữ dữ liệu và chỉ mục. Càng lớn càng ít phải đọc từ đĩa.
- **Cách tính**: Nên đặt khoảng 70-80% RAM cho MySQL nếu server chỉ chạy MySQL:
  - Server 16GB RAM: đặt 10-12GB
  - Server 32GB RAM: đặt 22-24GB
  - Server 64GB RAM: đặt 48-50GB
- **Kiểm tra hiệu quả**: Theo dõi tỷ lệ hit (buffer pool hit ratio):
  ```sql
  SHOW STATUS LIKE 'Innodb_buffer_pool_read_requests';
  SHOW STATUS LIKE 'Innodb_buffer_pool_reads';
  ```
  Tính: (read_requests - reads) / read_requests * 100
  Nếu < 95%, cần tăng size buffer pool.

- **innodb_buffer_pool_instances**: Chia buffer pool thành nhiều instances giúp giảm competition giữa các threads:
  - Nếu innodb_buffer_pool_size < 1GB: để instances = 1
  - Nếu >= 1GB: đặt số instances bằng số core CPU (tối đa 8 cho MySQL 5.7)
  - MySQL 8.0 tự động tính toán giá trị tối ưu

#### Table Open Cache
```ini
table_open_cache = 4000
```

**Hướng dẫn chi tiết:**
- **Vai trò**: Lưu trữ số lượng table handler (giúp truy cập nhanh hơn vào bảng)
- **Cách tính**: Số bảng * số kết nối cùng lúc
- **Kiểm tra**: Nếu `Opened_tables` tăng nhanh, cần tăng tham số này:
  ```sql
  SHOW GLOBAL STATUS LIKE 'Opened_tables';
  ```

#### Các tham số bổ sung
```ini
# Tối ưu InnoDB Log
innodb_log_file_size = 512M 
innodb_log_buffer_size = 16M

# Tăng hiệu suất đọc ghi
innodb_read_io_threads = 8
innodb_write_io_threads = 8

# Tối ưu bộ nhớ đệm sort & join
sort_buffer_size = 4M
join_buffer_size = 4M
read_buffer_size = 3M
read_rnd_buffer_size = 4M
```

**Lưu ý quan trọng**: 
1. Không đặt các buffer quá lớn (sort_buffer, join_buffer...) vì chúng được cấp phát cho mỗi kết nối.
2. Khi thay đổi, chỉ thay đổi 1-2 tham số mỗi lần và đo lường tác động.

### 1.2 Quản lý Kết Nối và Hiệu Năng

```ini
max_connections = 1000
max_user_connections = 200
wait_timeout = 300
interactive_timeout = 300
```

**Hướng dẫn chi tiết:**
- **max_connections**: 
  - Theo dõi số kết nối đang sử dụng: `SHOW STATUS LIKE 'Max_used_connections';`
  - Đặt 1.5 - 2 lần giá trị max_used_connections
  - Lưu ý: Mỗi kết nối tiêu tốn bộ nhớ (khoảng 2-3MB mỗi kết nối)

- **wait_timeout và interactive_timeout**:
  - Giá trị mặc định (28800s = 8h) thường quá lớn
  - Nên giảm xuống 300-600s (5-10 phút) cho hầu hết ứng dụng
  - Kiểm tra kết nối không hoạt động: `SHOW PROCESSLIST;`

- **thread_pool** (MySQL Enterprise hoặc Percona/MariaDB):
  ```ini
  thread_pool_size = 16 # Thường bằng số CPU cores
  ```
  Giúp quản lý hiệu quả các connections khi có nhiều kết nối đồng thời.

## 2. Tối ưu Hệ thống Lưu trữ (Storage Engine)

### InnoDB vs MyISAM

**Ưu điểm của InnoDB:**
- Hỗ trợ transactions
- Row-level locking (thay vì table-level)
- Phục hồi crash tốt hơn
- Hiệu suất cao hơn cho workloads ghi nhiều
- Foreign key constraints

**Cách chuyển đổi từ MyISAM sang InnoDB:**
```sql
-- Kiểm tra storage engine hiện tại
SELECT table_name, engine FROM information_schema.tables WHERE table_schema = 'database_name';

-- Đổi sang InnoDB
ALTER TABLE table_name ENGINE = InnoDB;
```

**Lưu ý khi chuyển đổi:**
1. Sao lưu dữ liệu trước khi thực hiện
2. Thực hiện từng bảng một, theo dõi hiệu suất
3. Các bảng lớn có thể mất nhiều thời gian chuyển đổi
4. Cần không gian đĩa gấp đôi tạm thời

### Cấu hình InnoDB nâng cao
```ini
# Tắt double-write buffer nếu dùng ổ SSD chất lượng cao
innodb_doublewrite = 0  # Cẩn thận khi dùng

# File-per-table (mỗi bảng 1 file .ibd riêng)
innodb_file_per_table = 1

# Tối ưu flush
innodb_flush_log_at_trx_commit = 2  # 1 an toàn nhất, 2 cân bằng
innodb_flush_method = O_DIRECT  # Bypass OS cache (tốt cho servers có nhiều RAM)
```

## 3. Cải thiện hiệu suất truy vấn (SQL Performance)

### Sử dụng EXPLAIN chi tiết

```sql
EXPLAIN SELECT * FROM orders WHERE customer_id = 123;
```

**Cách đọc kết quả EXPLAIN:**
- **select_type**: SIMPLE, SUBQUERY, JOIN...
- **type**: Từ tệ đến tốt: ALL (full scan) > index > range > ref > eq_ref > const/system
- **key**: Index được sử dụng
- **rows**: Số dòng ước tính phải duyệt
- **Extra**: "Using filesort", "Using temporary" là các cảnh báo về hiệu suất

**Ví dụ phân tích EXPLAIN:**
```
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table  | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | orders | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 1000 |    10.00 | Using where |
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------------+
```
👆 Cần tạo index vì đang dùng full table scan (type: ALL)

### Chiến lược đánh index hiệu quả

**Các trường hợp nên tạo index:**
1. Các trường hay dùng trong mệnh đề WHERE
2. Các trường dùng trong JOIN (khóa ngoại)
3. Các trường hay sắp xếp (ORDER BY)
4. Các trường hay grouping (GROUP BY)

**Hướng dẫn tạo index:**
```sql
-- Index đơn
CREATE INDEX idx_customer_id ON orders(customer_id);

-- Index kết hợp (composite)
CREATE INDEX idx_customer_created ON orders(customer_id, created_at);

-- Unique index
CREATE UNIQUE INDEX idx_order_number ON orders(order_number);
```

**Chiến lược nâng cao:**
- Đặt các trường thường lọc "=" đầu tiên trong composite index
- Optimize order: equality > range > sort > group by
- Chỉ mục bao phủ (Covering index): Thêm các trường thường SELECT vào INCLUDE để tránh truy cập bảng

**Đánh giá hiệu quả index:**
```sql
-- Kiểm tra index đang được sử dụng
SHOW INDEX FROM orders;

-- Phân tích hiệu quả index
SELECT
  table_schema,
  table_name,
  index_name,
  stat_value 
FROM performance_schema.table_io_waits_summary_by_index_usage
ORDER BY stat_value DESC;
```

### Phát hiện và tối ưu Slow Queries

**Cấu hình Slow Query Log:**
```ini
slow_query_log = 1
slow_query_log_file = /var/log/mysql/mysql-slow.log
long_query_time = 2  # Ghi lại queries chạy > 2 giây
log_queries_not_using_indexes = 1  # Ghi queries không dùng index
```

**Phân tích log với pt-query-digest (Percona):**
```bash
pt-query-digest /var/log/mysql/mysql-slow.log > analysis.txt
```

**Các kỹ thuật tối ưu queries chậm:**
1. Thêm index phù hợp
2. Viết lại queries (tránh SELECT *, chỉ lấy cột cần thiết)
3. Sử dụng LIMIT nếu không cần tất cả dữ liệu
4. Tạo Views, Stored Procedures cho queries phức tạp
5. Cân nhắc caching ở tầng ứng dụng

## 4. Tối ưu Bản ghi và Bảng

### Chọn kiểu dữ liệu tối ưu

**Các nguyên tắc chọn kiểu dữ liệu:**
1. Dùng kiểu dữ liệu nhỏ nhất có thể:
   - TINYINT (1 byte): 0-255 thay vì INT (4 bytes)
   - MEDIUMINT (3 bytes): 0-16.7M thay vì BIGINT (8 bytes)
   - VARCHAR(50) chính xác hơn VARCHAR(255) nếu biết giới hạn

2. Dùng kiểu số thay vì văn bản khi có thể:
   - CHAR(13) 📞 -> BIGINT tiết kiệm ~3 lần dung lượng

3. Chiến lược lưu trữ DATE/TIME:
   - DATE: 3 bytes
   - DATETIME: 8 bytes
   - TIMESTAMP: 4 bytes (nhưng giới hạn đến 2038)
   
**Ví dụ tạo bảng tối ưu:**
```sql
CREATE TABLE orders (
  id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,  -- UNSIGNED tăng dải giá trị
  customer_id INT UNSIGNED NOT NULL,
  total DECIMAL(10,2) NOT NULL,  -- Chính xác cho tiền tệ
  status TINYINT NOT NULL,  -- 0=pending, 1=processing, 2=completed...
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  INDEX idx_customer (customer_id),
  INDEX idx_created (created_at)
) ENGINE=InnoDB;
```

### Duy trì hiệu suất bảng lâu dài

**Phân tích và tối ưu bảng thường xuyên:**
```sql
-- Phân tích bảng để cập nhật thống kê
ANALYZE TABLE orders;

-- Defragment và rebuild indexes
OPTIMIZE TABLE orders;
```

**Phân vùng (Partitioning) cho bảng lớn:**
```sql
-- Phân vùng theo thời gian (mỗi tháng)
CREATE TABLE access_logs (
    id INT NOT NULL,
    created_at DATETIME NOT NULL,
    message VARCHAR(255),
    PRIMARY KEY (id, created_at)
)
PARTITION BY RANGE (YEAR(created_at) * 100 + MONTH(created_at)) (
    PARTITION p202201 VALUES LESS THAN (202202),
    PARTITION p202202 VALUES LESS THAN (202203),
    PARTITION p202203 VALUES LESS THAN (202204),
    -- ...
    PARTITION future VALUES LESS THAN MAXVALUE
);
```

**Chiến lược xử lý dữ liệu cũ:**
1. **Archiving**: Di chuyển dữ liệu cũ sang bảng archive
    ```sql
    INSERT INTO orders_archive SELECT * FROM orders WHERE created_at < NOW() - INTERVAL 1 YEAR;
    DELETE FROM orders WHERE created_at < NOW() - INTERVAL 1 YEAR;
    ```

2. **Sử dụng TTL với EVENT Scheduler:**
    ```sql
    CREATE EVENT cleanup_old_logs
    ON SCHEDULE EVERY 1 DAY
    DO
      DELETE FROM logs WHERE created_at < NOW() - INTERVAL 6 MONTH;
    ```

## 5. Cấu hình Replica (Replication)

### Cài đặt Master-Slave chi tiết

**Trên Master (my.cnf):**
```ini
[mysqld]
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
binlog_format = ROW
binlog_row_image = MINIMAL  # Chỉ ghi những cột thay đổi
expire_logs_days = 7  # Tự động xóa binary logs cũ
max_binlog_size = 100M

# Xác định database cần replicate
binlog_do_db = mydatabase  # hoặc
binlog_ignore_db = information_schema,performance_schema
```

**Trên Slave (my.cnf):**
```ini
[mysqld]
server-id = 2  # Phải khác server-id của master
relay_log = /var/log/mysql/mysql-relay-bin.log
read_only = 1  # Chỉ cho phép đọc
log_slave_updates = 1  # Hữu ích cho cấu hình master->slave->slave

# Tùy chọn để tăng hiệu suất replication
slave_compressed_protocol = 1  # Nén dữ liệu replicate qua mạng
```

**Tạo user trên Master:**
```sql
CREATE USER 'replica'@'%' IDENTIFIED BY 'StrongPassword123!';
GRANT REPLICATION SLAVE ON *.* TO 'replica'@'%';
FLUSH PRIVILEGES;
```

**Khởi tạo Slave (lấy thông tin từ Master):**
```sql
-- Trên Master - Lấy vị trí binary log
FLUSH TABLES WITH READ LOCK;
SHOW MASTER STATUS;
-- Ghi lại File và Position từ kết quả trên
-- Tạo backup với mysqldump hoặc Percona XtraBackup
-- Sau đó:
UNLOCK TABLES;

-- Trên Slave
CHANGE MASTER TO
  MASTER_HOST='master_ip',
  MASTER_USER='replica',
  MASTER_PASSWORD='StrongPassword123!',
  MASTER_LOG_FILE='mysql-bin.000001',  -- Từ SHOW MASTER STATUS
  MASTER_LOG_POS=120;  -- Từ SHOW MASTER STATUS
START SLAVE;
```

**Kiểm tra và xử lý lỗi:**
```sql
-- Kiểm tra trạng thái:
SHOW SLAVE STATUS\G

-- Kiểm tra 2 trường quan trọng:
-- 1. Slave_IO_Running: Yes
-- 2. Slave_SQL_Running: Yes

-- Nếu bị lỗi:
STOP SLAVE;
-- Sửa lỗi, thường là skip query gây lỗi:
SET GLOBAL sql_slave_skip_counter = 1;
START SLAVE;
```

### Chiến lược Replication nâng cao

**1. Semi-synchronous Replication:**
```ini
# Trên Master:
plugin-load = "rpl_semi_sync_master=semisync_master.so"
rpl_semi_sync_master_enabled = 1
rpl_semi_sync_master_timeout = 10000  # 10 giây

# Trên Slave:
plugin-load = "rpl_semi_sync_slave=semisync_slave.so"
rpl_semi_sync_slave_enabled = 1
```

**2. Cấu hình GTID (Global Transaction Identifier):**
```ini
# Trên cả Master và Slave:
gtid_mode = ON
enforce_gtid_consistency = ON
```

**3. Replication Filters để chọn dữ liệu cần replication:**
```ini
# Trên Slave - Chỉ replicate một số bảng:
replicate_wild_do_table = mydb.customer%
replicate_wild_do_table = mydb.order%
replicate_wild_ignore_table = mydb.log%
```

## 6. Giám sát và Bảo trì

### Công cụ giám sát

**1. Percona Monitoring and Management (PMM)**
- Cài đặt PMM Server: Dễ dàng với Docker
  ```bash
  docker run -d -p 80:80 -p 443:443 --name pmm-server percona/pmm-server:latest
  ```
- Cài đặt PMM Client trên MySQL server
  ```bash
  wget https://repo.percona.com/apt/percona-release_latest.generic_all.deb
  dpkg -i percona-release_latest.generic_all.deb
  apt-get update
  apt-get install pmm2-client
  pmm-admin config --server-insecure-tls --server-url=https://admin:admin@pmm-server-ip
  pmm-admin add mysql --username=pmm --password=password
  ```

**2. Hệ thống giám sát Prometheus + Grafana**

#### Cài đặt và cấu hình MySQL Exporter

1. **Tạo user MySQL cho exporter:**
```sql
CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'strong_password';
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';
GRANT SELECT ON performance_schema.* TO 'exporter'@'localhost';
FLUSH PRIVILEGES;
```

2. **Cài đặt mysqld_exporter:**
```bash
# Tải và cài đặt mysqld_exporter
wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.14.0/mysqld_exporter-0.14.0.linux-amd64.tar.gz
tar xvf mysqld_exporter-0.14.0.linux-amd64.tar.gz
cp mysqld_exporter-0.14.0.linux-amd64/mysqld_exporter /usr/local/bin/
chmod +x /usr/local/bin/mysqld_exporter

# Tạo file cấu hình
cat > ~/.my.cnf << EOF
[client]
user=exporter
password=strong_password
EOF

chmod 600 ~/.my.cnf
```

3. **Tạo service cho mysqld_exporter:**
```bash
cat > /etc/systemd/system/mysqld_exporter.service << EOF
[Unit]
Description=Prometheus MySQL Exporter
After=network.target mysql.service

[Service]
User=prometheus
ExecStart=/usr/local/bin/mysqld_exporter \
  --config.my-cnf=~/.my.cnf \
  --collect.global_status \
  --collect.info_schema.innodb_metrics \
  --collect.auto_increment.columns \
  --collect.info_schema.processlist \
  --collect.binlog_size \
  --collect.info_schema.tablestats \
  --collect.global_variables \
  --collect.info_schema.query_response_time \
  --collect.info_schema.userstats \
  --collect.info_schema.tables \
  --collect.perf_schema.tablelocks \
  --collect.perf_schema.file_events \
  --collect.perf_schema.eventswaits \
  --collect.perf_schema.indexiowaits \
  --collect.perf_schema.tableiowaits \
  --web.listen-address=0.0.0.0:9104

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable mysqld_exporter
systemctl start mysqld_exporter
```

#### Cài đặt Prometheus

```bash
# Tải và cài đặt Prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
tar xvf prometheus-2.45.0.linux-amd64.tar.gz
cp prometheus-2.45.0.linux-amd64/{prometheus,promtool} /usr/local/bin/

# Tạo thư mục cấu hình
mkdir -p /etc/prometheus /var/lib/prometheus

# Tạo file cấu hình
cat > /etc/prometheus/prometheus.yml << EOF
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'mysql'
    static_configs:
      - targets: ['localhost:9104']
        labels:
          instance: 'mysql_main'
EOF

# Tạo service cho Prometheus
cat > /etc/systemd/system/prometheus.service << EOF
[Unit]
Description=Prometheus Time Series Collection and Processing Server
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable prometheus
systemctl start prometheus
```

#### Cài đặt và cấu hình Grafana

```bash
# Thêm repository Grafana
cat > /etc/apt/sources.list.d/grafana.list << EOF
deb https://packages.grafana.com/oss/deb stable main
EOF
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

# Cài đặt Grafana
apt-get update
apt-get install -y grafana

# Khởi động Grafana
systemctl daemon-reload
systemctl enable grafana-server
systemctl start grafana-server
```

#### Tạo Dashboard MySQL trên Grafana

1. **Truy cập Grafana**: http://your-server:3000 (mặc định: admin/admin)
2. **Thêm Data Source**:
   - Configuration > Data Sources > Add data source
   - Chọn Prometheus
   - URL: http://localhost:9090
   - Lưu và Test

3. **Import Dashboard có sẵn**:
   - Dashboards > Import
   - Nhập ID: 7362 (MySQL Overview) hoặc 7371 (MySQL InnoDB Metrics)
   - Chọn Prometheus data source và Import

#### Các Metrics MySQL quan trọng cần giám sát

**1. Các metrics hiệu năng cơ bản:**
```
- mysql_global_status_questions (Tổng số queries)
- mysql_global_status_threads_connected (Số kết nối hiện tại)
- mysql_global_status_threads_running (Số thread đang chạy)
- mysql_global_status_slow_queries (Số lượng slow queries)
- mysql_global_status_queries (Queries per second)
```

**2. Metrics InnoDB quan trọng:**
```
- mysql_global_status_innodb_buffer_pool_read_requests (Tổng số read requests)
- mysql_global_status_innodb_buffer_pool_reads (Số lần phải đọc từ đĩa)
- mysql_global_status_innodb_row_lock_waits (Số lần phải chờ khóa hàng)
- mysql_global_status_innodb_row_lock_time (Thời gian chờ khóa hàng)
- mysql_global_status_innodb_data_writes (Số lượng write operations)
```

**3. Metrics Replication:**
```
- mysql_slave_status_seconds_behind_master (Replica lag)
- mysql_slave_status_slave_io_running (IO Thread status)
- mysql_slave_status_slave_sql_running (SQL Thread status)
```

#### Thiết lập Cảnh báo (Alerting)

1. **Cảnh báo kết nối cao:**
   - Metric: `mysql_global_status_threads_connected`
   - Condition: > 80% của max_connections

2. **Cảnh báo replica lag:**
   - Metric: `mysql_slave_status_seconds_behind_master`
   - Condition: > 300 seconds (5 phút)

3. **Cảnh báo tỷ lệ Buffer Pool Hit thấp:**
   - Expression: `100 * (1 - mysql_global_status_innodb_buffer_pool_reads / mysql_global_status_innodb_buffer_pool_read_requests)`
   - Condition: < 95%

4. **Cảnh báo đĩa đầy:**
   - Metric: `node_filesystem_avail_bytes{mountpoint="/var/lib/mysql"}`
   - Condition: < 10% dung lượng

#### Những Metrics chính cần giám sát và thresholds khuyến nghị

| Metric | Mô tả | Threshold cảnh báo | Threshold quan trọng |
|--------|-------|-------------------|---------------------|
| **Tổng quan hiệu năng** |||
| QPS (Queries per second) | Số lượng truy vấn mỗi giây | Tăng/giảm đột biến >30% | Tăng/giảm đột biến >50% |
| Connections | Số kết nối hiện tại | >80% max_connections | >90% max_connections |
| Threads_running | Số lượng threads đang chạy | >30 | >50 |
| **Cache & Bộ nhớ** |||
| Buffer pool hit ratio | % truy vấn được đáp ứng từ bộ đệm | <98% | <95% |
| Buffer pool utilization | Tỷ lệ sử dụng bộ đệm | >95% | >98% |
| **Hoạt động I/O** |||
| Innodb_data_writes | Ghi nhận hoạt động ghi | Tăng đột biến >30% | Tăng đột biến >50% |
| Innodb_data_reads | Ghi nhận hoạt động đọc | Tăng đột biến >30% | Tăng đột biến >50% |
| Disk Utilization | Mức sử dụng đĩa | >80% | >90% |
| **Replication** |||
| Seconds_Behind_Master | Độ trễ replica | >30 giây | >300 giây |
| Slave_IO_Running | Trạng thái thread IO | Khác "Yes" | N/A |
| Slave_SQL_Running | Trạng thái thread SQL | Khác "Yes" | N/A |
| **Locks & Deadlocks** |||
| Innodb_row_lock_waits | Số lần phải đợi khóa hàng | Tăng liên tục | >100/phút |
| Deadlocks | Số deadlocks | >0 | >5/giờ |

#### Biểu đồ quan trọng trên Grafana

1. **MySQL Overview**:
   - QPS & Connection Usage (2 trục)
   - Innodb Buffer Pool (Free vs Used)
   - MySQL Command Counters (Select, Insert, Update, Delete)
   
2. **MySQL Performance**:
   - Slow Queries Rate
   - Table Locks
   - Buffer Pool Hit Ratio
   - Top 5 Slow Queries (bảng)
   
3. **MySQL Replication**:
   - Replica Lag (biểu đồ)
   - Replication Status (panel)
   - Binary Log Size Growth

### Bảo trì định kỳ

**Checklist bảo trì hàng tuần:**
1. Kiểm tra error log: `tail -100 /var/log/mysql/error.log`
2. Kiểm tra slow query log: `pt-query-digest /var/log/mysql/slow.log`
3. Phân tích tables: `mysqlcheck -A --analyze`
4. Kiểm tra replica lag: `SHOW SLAVE STATUS\G`

**Checklist bảo trì hàng tháng:**
1. Kiểm tra và tối ưu indexes
   ```sql
   SELECT * FROM sys.schema_unused_indexes;
   SELECT * FROM sys.schema_redundant_indexes;
   ```
2. Đánh giá việc sử dụng tài nguyên
   ```sql
   SELECT * FROM sys.host_summary;
   SELECT * FROM sys.memory_global_by_current_bytes;
   ```
3. OPTIMIZE các bảng hay được cập nhật
4. Kiểm tra và xóa binary logs cũ nếu cần

## 7. Backup & Restore

### Chiến lược backup toàn diện

**1. Logical Backup với mysqldump:**
```bash
# Backup toàn bộ
mysqldump -u root -p --all-databases --events --routines --triggers \
  --single-transaction --master-data=2 > full_backup_$(date +%Y%m%d).sql

# Backup từng database (song song)
for DB in $(mysql -u root -p -e "SHOW DATABASES;" | grep -v "Database\|information_schema\|performance_schema"); do
  mysqldump -u root -p --single-transaction --master-data=2 $DB > "${DB}_$(date +%Y%m%d).sql" &
done
wait
```

**2. Physical Backup với Percona XtraBackup:**
```bash
# Full backup
xtrabackup --backup --target-dir=/backup/full --user=root --password=password

# Incremental backup (sau khi đã có full backup)
xtrabackup --backup --target-dir=/backup/inc1 \
  --incremental-basedir=/backup/full \
  --user=root --password=password
```

**3. Point-in-time Recovery:**
```bash
# 1. Restore full backup
xtrabackup --prepare --apply-log-only --target-dir=/backup/full

# 2. Áp dụng incremental backups
xtrabackup --prepare --apply-log-only --target-dir=/backup/full \
  --incremental-dir=/backup/inc1

# 3. Hoàn thành prepare
xtrabackup --prepare --target-dir=/backup/full

# 4. Restore
xtrabackup --copy-back --target-dir=/backup/full
chown -R mysql:mysql /var/lib/mysql
```

**Xác minh backup:**
```bash
# Test tạo Instance tạm MySQL từ backup
mkdir /tmp/mysql_verify
xtrabackup --copy-back --target-dir=/backup/full --datadir=/tmp/mysql_verify

# Khởi động với MySQL tạm và kiểm tra
mysqld --no-defaults --skip-networking --datadir=/tmp/mysql_verify
mysql -uroot -p -e "SHOW DATABASES;"
```

## 8. Quy trình tuning MySQL hiệu quả

### Quy trình tuning khoa học

**Bước 1: Xác định hiện trạng**
- Chạy MySQL Tuner: 
  ```bash
  wget https://raw.githubusercontent.com/major/MySQLTuner-perl/master/mysqltuner.pl
  perl mysqltuner.pl
  ```
- Lấy thông số hiện tại:
  ```sql
  SHOW GLOBAL VARIABLES;
  SHOW GLOBAL STATUS;
  ```

**Bước 2: Xác định vấn đề**
- Tỷ lệ cache hit thấp? → Tăng buffer pools
- Slow queries nhiều? → Tối ưu queries và indexes
- I/O cao? → Kiểm tra cấu hình storage và flush
- Connections nhiều? → Tối ưu connection pool ở ứng dụng

**Bước 3: Thử nghiệm có kiểm soát**
1. Chỉ thay đổi 1-2 tham số một lần
2. Ghi lại trạng thái trước khi thay đổi
3. Giám sát kỹ sau khi thay đổi (24-48h)
4. So sánh trước/sau dựa trên metrics cụ thể

**Bước 4: Đánh giá và lặp lại**
1. Nếu cải thiện: ghi nhận và tiếp tục với tham số khác
2. Nếu tệ hơn: quay lại cấu hình trước đó
3. Lặp lại quy trình cho đến khi đạt hiệu suất mong muốn

### Một số tuning parameters cho các workload phổ biến

**Cho OLTP (Online Transaction Processing - Web/App):**
```ini
innodb_buffer_pool_size = 70% RAM
innodb_log_file_size = 512M
innodb_flush_log_at_trx_commit = 1
innodb_file_per_table = 1
max_connections = 500-1000
```

**Cho OLAP (Online Analytical Processing - Data Warehouse):**
```ini
innodb_buffer_pool_size = 70% RAM
innodb_buffer_pool_instances = 1-2
join_buffer_size = 8M
sort_buffer_size = 8M
read_rnd_buffer_size = 8M
innodb_flush_log_at_trx_commit = 0 hoặc 2
```

**Cho Database có dung lượng lớn hơn RAM (Big Data):**
```ini
innodb_buffer_pool_size = 50% RAM
innodb_buffer_pool_instances = 8+
innodb_read_io_threads = 16
innodb_write_io_threads = 16
innodb_doublewrite = 0 (nếu dùng raid controller có battery backup)
table_open_cache = 8000+
```

**Cho Replica (Slave DB Server):**
```ini
innodb_flush_log_at_trx_commit = 2
sync_binlog = 0
read_only = 1
innodb_flush_method = O_DIRECT
```

### Các công cụ hỗ trợ tuning khác

1. **pt-variable-advisor (Percona)**:
   ```bash
   pt-variable-advisor localhost
   ```

2. **MySQLTuner**:
   ```bash
   wget https://raw.githubusercontent.com/major/MySQLTuner-perl/master/mysqltuner.pl
   perl mysqltuner.pl --user root --pass <password>
   ```

3. **tuning-primer**:
   ```bash
   wget https://launchpad.net/mysql-tuning-primer/trunk/1.6-r1/+download/tuning-primer.sh
   chmod +x tuning-primer.sh
   ./tuning-primer.sh
   ```

4. **Sample my.cnf tối ưu theo RAM**:
   - [https://github.com/major/MySQLTuner-perl/tree/master/sample_mysqltuner.cnf](https://github.com/major/MySQLTuner-perl/tree/master/sample_mysqltuner.cnf)

---

Tài liệu này cung cấp hướng dẫn cơ bản cho việc tuning MySQL. Mỗi cơ sở dữ liệu có đặc thù riêng nên cần điều chỉnh phù hợp với workload và môi trường cụ thể. Luôn back up dữ liệu trước khi thực hiện các thay đổi quan trọng.
