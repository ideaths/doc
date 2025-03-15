I. Cấu hình và Tối ưu MySQL

1. Cấu hình quan trọng trong my.cnf

Tệp cấu hình my.cnf (hoặc my.ini trên Windows) quyết định hiệu suất của MySQL. Một số tham số quan trọng cần điều chỉnh:

1.1 Bộ nhớ và Cache

1.1.1 InnoDB Buffer Pool Size
	•	Dành cho InnoDB Storage Engine.
	•	Chiếm khoảng 70-80% tổng RAM trên máy chủ MySQL.

[mysqld]
innodb_buffer_pool_size = 4G
innodb_buffer_pool_instances = 8

➡ Nếu RAM là 16GB, bạn có thể đặt innodb_buffer_pool_size = 12G.

1.1.2 Query Cache
	•	Tăng tốc độ truy vấn lặp lại nhưng không phù hợp với hệ thống có tần suất ghi cao.

query_cache_type = 1
query_cache_size = 256M
query_cache_limit = 2M

➡ Lưu ý: Từ MySQL 8.0 trở đi, Query Cache đã bị loại bỏ. Thay vào đó, hãy sử dụng ProxySQL hoặc memcached.

1.1.3 Table Open Cache
	•	Kiểm soát số lượng bảng có thể mở đồng thời.

table_open_cache = 4000

➡ Nếu có nhiều bảng, tăng giá trị này.

1.1.4 Thread Cache
	•	Giúp giảm thời gian tạo Thread mới cho mỗi kết nối.

thread_cache_size = 64



⸻

2. Quản lý Kết Nối và Hiệu Năng

2.1 Tối ưu số lượng kết nối đồng thời

max_connections = 1000
max_user_connections = 200

➡ Giúp MySQL xử lý nhiều kết nối mà không bị quá tải.

2.2 Kiểm soát Timeout

wait_timeout = 300
interactive_timeout = 300

➡ Giảm thời gian giữ kết nối mở mà không sử dụng.

⸻

3. Tối ưu Hệ thống Lưu trữ (Storage Engine)
	•	Nên sử dụng InnoDB thay vì MyISAM vì InnoDB hỗ trợ ACID, row-level locking, và crash recovery.

default_storage_engine = InnoDB

	•	Nếu đang sử dụng MyISAM, hãy chuyển sang InnoDB bằng lệnh:

ALTER TABLE table_name ENGINE = InnoDB;



⸻

4. Cải thiện hiệu suất truy vấn (SQL Performance)

4.1 Kiểm tra và tối ưu truy vấn bằng EXPLAIN
	•	Trước khi chạy truy vấn chậm, hãy kiểm tra kế hoạch thực thi của nó:

EXPLAIN SELECT * FROM orders WHERE customer_id = 123;

➡ Nếu thấy FULL TABLE SCAN (ALL), cần tạo index phù hợp.

4.2 Tối ưu chỉ mục (Indexing)
	•	Chỉ mục (INDEX) giúp tăng tốc độ truy vấn nhưng nếu lạm dụng có thể làm giảm hiệu suất ghi.
	•	Tạo index cho cột được sử dụng trong WHERE, JOIN, ORDER BY:

CREATE INDEX idx_customer_id ON orders(customer_id);

4.3 Bật Slow Query Log để theo dõi truy vấn chậm

slow_query_log = 1
slow_query_log_file = /var/log/mysql_slow.log
long_query_time = 2

➡ Ghi lại các truy vấn chạy lâu hơn 2 giây để tối ưu.

⸻

5. Tối ưu Bản ghi và Bảng

5.1 Giảm kích thước dữ liệu
	•	Sử dụng kiểu dữ liệu phù hợp:
	•	INT(11) vs TINYINT(3): Nếu chỉ cần lưu giá trị nhỏ, hãy dùng TINYINT để tiết kiệm bộ nhớ.
	•	VARCHAR vs TEXT: Dùng VARCHAR(255) thay vì TEXT nếu độ dài không quá lớn.

5.2 Dọn dẹp bảng
	•	Xóa bản ghi không cần thiết:

DELETE FROM logs WHERE created_at < NOW() - INTERVAL 6 MONTH;

	•	Optimize table để giảm fragmentation:

OPTIMIZE TABLE my_table;



⸻

6. Cấu hình Replica (Replication) để cân bằng tải

Nếu hệ thống cần mở rộng, có thể sử dụng Master-Slave Replication:
	1.	Cấu hình trên Master (my.cnf):

server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
binlog_format = ROW

	2.	Cấu hình trên Slave (my.cnf):

server-id = 2
relay-log = /var/log/mysql/mysql-relay-bin.log

	3.	Kết nối Slave với Master:

CHANGE MASTER TO MASTER_HOST='master_ip', MASTER_USER='replica', MASTER_PASSWORD='password', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=4;
START SLAVE;

➡ Kiểm tra trạng thái bằng:

SHOW SLAVE STATUS \G;



⸻

7. Giám sát và Bảo trì

7.1 Công cụ giám sát hiệu suất
	•	Percona Monitoring and Management (PMM)
	•	Grafana + Prometheus
	•	MySQL Enterprise Monitor

7.2 Backup & Restore
	•	Dùng mysqldump:

mysqldump -u root -p --all-databases > backup.sql

	•	Dùng Percona XtraBackup để backup nhanh mà không khóa bảng:

xtrabackup --backup --target-dir=/backup



⸻
