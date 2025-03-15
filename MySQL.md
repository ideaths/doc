Bạn có muốn file README.md được định dạng với tiêu đề, mã lệnh được highlight và có mục lục không? Ngoài ra, bạn có muốn thêm hướng dẫn cài đặt hoặc ví dụ cụ thể về từng phần tối ưu không?

Research is not currently supported in temporary chats.

Dưới đây là nội dung file README.md cho hướng dẫn cấu hình và tối ưu MySQL:

# Cấu Hình và Tối Ưu MySQL

## Mục lục
- [1. Cấu hình quan trọng trong `my.cnf`](#1-cấu-hình-quan-trọng-trong-mycfn)
  - [1.1 Bộ nhớ và Cache](#11-bộ-nhớ-và-cache)
  - [1.2 Quản lý Kết Nối và Hiệu Năng](#12-quản-lý-kết-nối-và-hiệu-năng)
- [2. Tối ưu Hệ thống Lưu trữ (Storage Engine)](#2-tối-ưu-hệ-thống-lưu-trữ-storage-engine)
- [3. Cải thiện hiệu suất truy vấn (SQL Performance)](#3-cải-thiện-hiệu-suất-truy-vấn-sql-performance)
- [4. Tối ưu Bản ghi và Bảng](#4-tối-ưu-bản-ghi-và-bảng)
- [5. Cấu hình Replica (Replication)](#5-cấu-hình-replica-replication)
- [6. Giám sát và Bảo trì](#6-giám-sát-và-bảo-trì)
- [7. Backup & Restore](#7-backup--restore)

---

## 1. Cấu hình quan trọng trong `my.cnf`
File `my.cnf` quyết định hiệu suất MySQL. Một số tham số quan trọng:

### 1.1 Bộ nhớ và Cache
#### InnoDB Buffer Pool Size
```ini
[mysqld]
innodb_buffer_pool_size = 4G
innodb_buffer_pool_instances = 8

➡ Nếu RAM là 16GB, bạn có thể đặt innodb_buffer_pool_size = 12G.

Query Cache

query_cache_type = 1
query_cache_size = 256M
query_cache_limit = 2M

➡ Lưu ý: Từ MySQL 8.0 trở đi, Query Cache đã bị loại bỏ.

Table Open Cache

table_open_cache = 4000

➡ Nếu có nhiều bảng, tăng giá trị này.

Thread Cache

thread_cache_size = 64



⸻

1.2 Quản lý Kết Nối và Hiệu Năng

Tối ưu số lượng kết nối đồng thời

max_connections = 1000
max_user_connections = 200

Kiểm soát Timeout

wait_timeout = 300
interactive_timeout = 300



⸻

2. Tối ưu Hệ thống Lưu trữ (Storage Engine)
	•	Nên sử dụng InnoDB thay vì MyISAM:

default_storage_engine = InnoDB

	•	Chuyển đổi từ MyISAM sang InnoDB:

ALTER TABLE table_name ENGINE = InnoDB;



⸻

3. Cải thiện hiệu suất truy vấn (SQL Performance)

Kiểm tra và tối ưu truy vấn bằng EXPLAIN

EXPLAIN SELECT * FROM orders WHERE customer_id = 123;

➡ Nếu thấy FULL TABLE SCAN (ALL), cần tạo index phù hợp.

Tối ưu chỉ mục (Indexing)

CREATE INDEX idx_customer_id ON orders(customer_id);

Bật Slow Query Log để theo dõi truy vấn chậm

slow_query_log = 1
slow_query_log_file = /var/log/mysql_slow.log
long_query_time = 2



⸻

4. Tối ưu Bản ghi và Bảng

Giảm kích thước dữ liệu
	•	Dùng TINYINT thay vì INT nếu giá trị nhỏ.
	•	Dùng VARCHAR(255) thay vì TEXT.

Dọn dẹp bảng

DELETE FROM logs WHERE created_at < NOW() - INTERVAL 6 MONTH;
OPTIMIZE TABLE my_table;



⸻

5. Cấu hình Replica (Replication)

Cấu hình trên Master (my.cnf):

server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
binlog_format = ROW

Cấu hình trên Slave (my.cnf):

server-id = 2
relay-log = /var/log/mysql/mysql-relay-bin.log

Kết nối Slave với Master:

CHANGE MASTER TO MASTER_HOST='master_ip', MASTER_USER='replica', MASTER_PASSWORD='password', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=4;
START SLAVE;

➡ Kiểm tra trạng thái bằng:

SHOW SLAVE STATUS \G;



⸻

6. Giám sát và Bảo trì

Công cụ giám sát hiệu suất
	•	Percona Monitoring and Management (PMM)
	•	Grafana + Prometheus
	•	MySQL Enterprise Monitor

⸻

7. Backup & Restore

Backup với mysqldump

mysqldump -u root -p --all-databases > backup.sql

Backup không gián đoạn bằng Percona XtraBackup

xtrabackup --backup --target-dir=/backup



⸻

Tóm tắt

✅ Tối ưu my.cnf để tăng hiệu suất.
✅ Indexing giúp tăng tốc độ truy vấn.
✅ Slow Query Log để theo dõi truy vấn chậm.
✅ Replication giúp mở rộng hệ thống.
✅ Backup & Monitoring đảm bảo an toàn dữ liệu.

🚀 Tối ưu MySQL giúp tăng hiệu suất và đảm bảo hệ thống vận hành ổn định!

Bạn có muốn mình bổ sung phần nào hoặc tạo file `README.md` để bạn tải về không? 🚀
