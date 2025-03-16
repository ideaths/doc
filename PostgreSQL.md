# PostgreSQL: Cài Đặt, Cấu Hình và Tối Ưu Hiệu Suất

## Mục lục
- [1. Tổng quan về PostgreSQL](#1-tổng-quan-về-postgresql)
  - [1.1 Kiến trúc PostgreSQL](#11-kiến-trúc-postgresql)
  - [1.2 Cơ chế hoạt động](#12-cơ-chế-hoạt-động)
- [2. Cài đặt PostgreSQL](#2-cài-đặt-postgresql)
  - [2.1 Cài đặt trên các hệ điều hành](#21-cài-đặt-trên-các-hệ-điều-hành)
  - [2.2 Cài đặt từ source code](#22-cài-đặt-từ-source-code)
- [3. Cấu hình cơ bản PostgreSQL](#3-cấu-hình-cơ-bản-postgresql)
  - [3.1 File postgresql.conf](#31-file-postgresqlconf)
  - [3.2 File pg_hba.conf](#32-file-pg_hbaconf)
  - [3.3 Các file cấu hình khác](#33-các-file-cấu-hình-khác)
- [4. Quản lý cơ sở dữ liệu và bảng](#4-quản-lý-cơ-sở-dữ-liệu-và-bảng)
  - [4.1 Tạo và quản lý databases](#41-tạo-và-quản-lý-databases)
  - [4.2 Thiết kế schema và tables](#42-thiết-kế-schema-và-tables)
  - [4.3 Data types và kích thước dữ liệu](#43-data-types-và-kích-thước-dữ-liệu)
- [5. Indexes và tối ưu truy vấn](#5-indexes-và-tối-ưu-truy-vấn)
  - [5.1 Các loại index](#51-các-loại-index)
  - [5.2 Tối ưu và phân tích truy vấn (EXPLAIN)](#52-tối-ưu-và-phân-tích-truy-vấn-explain)
  - [5.3 Quản lý index và bảo trì](#53-quản-lý-index-và-bảo-trì)
- [6. Tối ưu hiệu suất PostgreSQL](#6-tối-ưu-hiệu-suất-postgresql)
  - [6.1 Tối ưu bộ nhớ](#61-tối-ưu-bộ-nhớ)
  - [6.2 Tối ưu I/O và Disk](#62-tối-ưu-io-và-disk)
  - [6.3 Vacuum và Autovacuum](#63-vacuum-và-autovacuum)
  - [6.4 Cấu hình connection và performance](#64-cấu-hình-connection-và-performance)
- [7. Replication và High Availability](#7-replication-và-high-availability)
  - [7.1 Streaming Replication](#71-streaming-replication)
  - [7.2 Logical Replication](#72-logical-replication)
  - [7.3 Các công cụ HA và Load Balancing](#73-các-công-cụ-ha-và-load-balancing)
- [8. Backup và Restore](#8-backup-và-restore)
  - [8.1 Phương thức backup](#81-phương-thức-backup)
  - [8.2 Point-in-Time Recovery](#82-point-in-time-recovery)
  - [8.3 Chiến lược backup và restore](#83-chiến-lược-backup-và-restore)
- [9. Bảo mật PostgreSQL](#9-bảo-mật-postgresql)
  - [9.1 Xác thực và phân quyền](#91-xác-thực-và-phân-quyền)
  - [9.2 Mã hóa và SSL](#92-mã-hóa-và-ssl)
  - [9.3 Row-level security](#93-row-level-security)
- [10. Giám sát và Maintenance](#10-giám-sát-và-maintenance)
  - [10.1 Giám sát với Prometheus và Grafana](#101-giám-sát-với-prometheus-và-grafana)
  - [10.2 Bảo trì định kỳ](#102-bảo-trì-định-kỳ)
  - [10.3 Xử lý sự cố](#103-xử-lý-sự-cố)
- [11. Tính năng nâng cao](#11-tính-năng-nâng-cao)
  - [11.1 Partitioning](#111-partitioning)
  - [11.2 Foreign Data Wrappers](#112-foreign-data-wrappers)
  - [11.3 Extensions](#113-extensions)
  - [11.4 JSON và tính năng NoSQL](#114-json-và-tính-năng-nosql)

---

## 1. Tổng quan về PostgreSQL

### 1.1 Kiến trúc PostgreSQL

PostgreSQL là hệ quản trị cơ sở dữ liệu quan hệ và đối tượng (ORDBMS) mã nguồn mở, với hơn 30 năm phát triển. Nó được thiết kế để xử lý khối lượng công việc lớn từ hệ thống đơn lẻ đến ứng dụng doanh nghiệp với nhiều người dùng đồng thời.

**Mô hình kiến trúc:**

1. **Postmaster Process**: Process chính khởi động và quản lý các thành phần khác
2. **Backend Processes**: Xử lý các kết nối client riêng biệt
3. **Background Processes**: 
   - Writer: Ghi dữ liệu từ memory xuống disk
   - WAL Writer: Ghi Write-Ahead Logs
   - Autovacuum: Dọn dẹp "rác" và tối ưu không gian
   - Checkpointer: Đảm bảo dữ liệu đồng bộ
   - Stats Collector: Thu thập thông tin thống kê
   - Logger: Ghi log hệ thống

**Các thành phần lưu trữ:**
1. **Shared Buffers**: Bộ nhớ đệm dùng chung cho dữ liệu
2. **WAL Buffers**: Bộ nhớ đệm cho Write-Ahead Logs
3. **Data Files**: Lưu trữ dữ liệu thực tế trên disk
4. **Write-Ahead Logs (WAL)**: Ghi log trước khi thay đổi dữ liệu
5. **pg_xact**: Thông tin về transaction status

### 1.2 Cơ chế hoạt động

**Quá trình xử lý truy vấn:**
1. **Parser**: Phân tích cú pháp truy vấn
2. **Rewriter**: Xử lý rules, views
3. **Planner/Optimizer**: Lập kế hoạch thực thi tối ưu
4. **Executor**: Thực thi các bước theo plan

**MVCC (Multi-Version Concurrency Control):**
- Cho phép đọc và ghi đồng thời không bị blocking
- Mỗi transaction nhìn thấy snapshot của database tại thời điểm bắt đầu
- Không sử dụng read locks, tăng hiệu suất

**Transaction và ACID:**
- **Atomicity**: Transaction hoàn thành toàn bộ hoặc không hoàn thành
- **Consistency**: Database luôn ở trạng thái hợp lệ
- **Isolation**: Transaction độc lập với transaction khác
- **Durability**: Dữ liệu commit vẫn tồn tại sau khi hệ thống bị lỗi

## 2. Cài đặt PostgreSQL

### 2.1 Cài đặt trên các hệ điều hành

**Ubuntu/Debian:**
```bash
# Thêm repository
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Cập nhật package list
sudo apt-get update

# Cài đặt PostgreSQL (ví dụ PostgreSQL 14)
sudo apt-get install postgresql-14
```

**CentOS/RHEL:**
```bash
# Cài đặt repository
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# Cài đặt PostgreSQL
sudo yum install -y postgresql14-server

# Khởi tạo database
sudo /usr/pgsql-14/bin/postgresql-14-setup initdb

# Khởi động và enable dịch vụ
sudo systemctl enable postgresql-14
sudo systemctl start postgresql-14
```

**Docker:**
```bash
# Pull PostgreSQL image
docker pull postgres:14

# Khởi chạy container
docker run --name postgres-db -e POSTGRES_PASSWORD=mysecretpassword -p 5432:5432 -d postgres:14
```

### 2.2 Cài đặt từ source code

```bash
# Cài đặt các dependencies
sudo apt-get install -y build-essential libreadline-dev zlib1g-dev libssl-dev libxml2-dev libxslt1-dev

# Tải source code
wget https://ftp.postgresql.org/pub/source/v14.6/postgresql-14.6.tar.gz
tar xzf postgresql-14.6.tar.gz
cd postgresql-14.6

# Configure và compile
./configure --prefix=/usr/local/pgsql
make
sudo make install

# Tạo user và data directory
sudo adduser postgres
sudo mkdir /usr/local/pgsql/data
sudo chown postgres:postgres /usr/local/pgsql/data

# Khởi tạo database cluster
sudo -u postgres /usr/local/pgsql/bin/initdb -D /usr/local/pgsql/data

# Tạo service file
sudo nano /etc/systemd/system/postgresql.service

# Thêm nội dung sau
[Unit]
Description=PostgreSQL database server
After=network.target

[Service]
Type=forking
User=postgres
Group=postgres
Environment=PGDATA=/usr/local/pgsql/data
ExecStart=/usr/local/pgsql/bin/pg_ctl start -D ${PGDATA} -s -w -t 120
ExecStop=/usr/local/pgsql/bin/pg_ctl stop -D ${PGDATA} -s -m fast
ExecReload=/usr/local/pgsql/bin/pg_ctl reload -D ${PGDATA} -s
TimeoutSec=120

[Install]
WantedBy=multi-user.target

# Enable và start service
sudo systemctl daemon-reload
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

## 3. Cấu hình cơ bản PostgreSQL

### 3.1 File postgresql.conf

File `postgresql.conf` chứa các cấu hình chung cho PostgreSQL server. Một số tham số quan trọng:

**Kết nối:**
```
listen_addresses = '*'           # Địa chỉ lắng nghe (default: localhost)
port = 5432                      # Port mặc định
max_connections = 100            # Số lượng kết nối tối đa
```

**Bộ nhớ:**
```
shared_buffers = 128MB           # Kích thước bộ nhớ đệm dùng chung
work_mem = 4MB                   # Bộ nhớ cho các thao tác sort
maintenance_work_mem = 64MB      # Bộ nhớ cho các thao tác bảo trì
effective_cache_size = 4GB       # Ước tính bộ nhớ cache có sẵn
```

**Write-Ahead Log (WAL):**
```
wal_level = replica              # Mức độ chi tiết của WAL
max_wal_size = 1GB               # Kích thước tối đa của WAL trước khi checkpoint
min_wal_size = 80MB              # Kích thước tối thiểu của WAL
```

**Background Writer:**
```
bgwriter_delay = 200ms           # Thời gian chờ giữa các lần ghi
bgwriter_lru_maxpages = 100      # Số trang tối đa mỗi lần
bgwriter_lru_multiplier = 2.0    # Hệ số cho số trang cần ghi
```

**Autovacuum:**
```
autovacuum = on                  # Bật autovacuum
log_autovacuum_min_duration = 0  # Ghi log về hoạt động autovacuum
autovacuum_max_workers = 3       # Số lượng workers tối đa
autovacuum_naptime = 1min        # Thời gian nghỉ giữa các lần chạy
```

**Logging:**
```
log_destination = 'stderr'       # Nơi ghi log
logging_collector = on           # Bật collector để ghi log vào file
log_directory = 'log'            # Thư mục lưu log
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'  # Format tên file log
log_statement = 'none'           # Log các SQL statements (none, ddl, mod, all)
log_min_duration_statement = -1  # Log các query chạy lâu (ms), -1 để tắt
```

### 3.2 File pg_hba.conf

File `pg_hba.conf` quản lý quyền truy cập của client đến PostgreSQL. Mỗi dòng định nghĩa một rule:

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
host    all             all             192.168.1.0/24          md5
```

**Các loại METHOD phổ biến:**
- **trust**: Cho phép kết nối không cần xác thực
- **peer**: Sử dụng username của OS để xác thực (chỉ dùng cho local connections)
- **md5**: Yêu cầu mật khẩu được mã hóa MD5
- **scram-sha-256**: Phương thức xác thực an toàn hơn (từ PostgreSQL 10)
- **reject**: Từ chối kết nối

### 3.3 Các file cấu hình khác

**pg_ident.conf:**
- Ánh xạ identity giữa OS user và PostgreSQL user
- Format: `MAPNAME    SYSTEM-USERNAME    PG-USERNAME`

**recovery.conf (PostgreSQL < 12) / standby.signal (PostgreSQL ≥ 12):**
- Cấu hình cho standby server trong replication
- Trong PostgreSQL 12+, các cấu hình recovery được chuyển vào postgresql.conf và tạo file standby.signal để đánh dấu đây là standby server

## 4. Quản lý cơ sở dữ liệu và bảng

### 4.1 Tạo và quản lý databases

**Tạo database:**
```sql
CREATE DATABASE mydb
  WITH OWNER = myuser
       ENCODING = 'UTF8'
       LC_COLLATE = 'en_US.UTF-8'
       LC_CTYPE = 'en_US.UTF-8'
       TEMPLATE = template0
       CONNECTION LIMIT = -1;
```

**Các tùy chọn khác:**
- `TABLESPACE`: Chỉ định tablespace
- `ALLOW_CONNECTIONS`: Cho phép kết nối
- `IS_TEMPLATE`: Sử dụng làm template

**Quản lý database:**
```sql
-- Liệt kê databases
SELECT datname FROM pg_database;

-- Xem thông tin chi tiết
SELECT * FROM pg_database WHERE datname = 'mydb';

-- Đổi tên database
ALTER DATABASE mydb RENAME TO mynewdb;

-- Đổi owner
ALTER DATABASE mydb OWNER TO newowner;

-- Xóa database
DROP DATABASE mydb;
```

### 4.2 Thiết kế schema và tables

**Tạo schema:**
```sql
CREATE SCHEMA myschema AUTHORIZATION myuser;
```

**Tạo bảng:**
```sql
CREATE TABLE myschema.customers (
  customer_id SERIAL PRIMARY KEY,
  first_name VARCHAR(50) NOT NULL,
  last_name VARCHAR(50) NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  status SMALLINT DEFAULT 1,
  CONSTRAINT chk_status CHECK (status IN (0, 1, 2))
);
```

**Quản lý bảng:**
```sql
-- Thêm cột
ALTER TABLE myschema.customers ADD COLUMN phone VARCHAR(20);

-- Đổi kiểu dữ liệu
ALTER TABLE myschema.customers ALTER COLUMN phone TYPE VARCHAR(25);

-- Thêm constraint
ALTER TABLE myschema.customers ADD CONSTRAINT unique_phone UNIQUE (phone);

-- Tạo index
CREATE INDEX idx_customers_lastname ON myschema.customers(last_name);

-- Xóa cột
ALTER TABLE myschema.customers DROP COLUMN phone;

-- Xóa bảng
DROP TABLE myschema.customers;
```

**Quan hệ giữa các bảng:**
```sql
-- Tạo bảng với foreign key
CREATE TABLE myschema.orders (
  order_id SERIAL PRIMARY KEY,
  customer_id INTEGER NOT NULL,
  order_date TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  total_amount DECIMAL(10, 2) NOT NULL,
  CONSTRAINT fk_customer
    FOREIGN KEY (customer_id)
    REFERENCES myschema.customers(customer_id)
    ON DELETE RESTRICT
);
```

### 4.3 Data types và kích thước dữ liệu

**Kiểu số:**
- `SMALLINT`: 2 bytes, -32768 đến 32767
- `INTEGER`: 4 bytes, -2147483648 đến 2147483647
- `BIGINT`: 8 bytes, -9223372036854775808 đến 9223372036854775807
- `NUMERIC(p,s)`: Số chính xác với p chữ số và s chữ số thập phân
- `REAL`: 4 bytes, 6 chữ số thập phân
- `DOUBLE PRECISION`: 8 bytes, 15 chữ số thập phân

**Kiểu chuỗi:**
- `CHAR(n)`: Chuỗi độ dài cố định
- `VARCHAR(n)`: Chuỗi độ dài thay đổi
- `TEXT`: Chuỗi không giới hạn độ dài

**Kiểu thời gian:**
- `DATE`: Ngày (4 bytes)
- `TIME`: Thời gian (8 bytes)
- `TIMESTAMP`: Ngày và thời gian (8 bytes)
- `TIMESTAMP WITH TIME ZONE`: Ngày và thời gian có múi giờ (8 bytes)
- `INTERVAL`: Khoảng thời gian (16 bytes)

**Kiểu khác:**
- `BOOLEAN`: true/false
- `UUID`: Mã định danh duy nhất phổ quát
- `JSON/JSONB`: Dữ liệu JSON
- `ARRAY`: Mảng các phần tử
- `BYTEA`: Dữ liệu nhị phân
- `CIDR/INET`: Địa chỉ IP và mạng
- `POINT/LINE/POLYGON`: Dữ liệu hình học

**Chiến lược chọn data type:**
1. Sử dụng kiểu dữ liệu nhỏ nhất có thể
2. Cân nhắc sử dụng `TEXT` thay vì `VARCHAR` cho chuỗi lớn
3. `TIMESTAMP WITH TIME ZONE` tốt hơn `TIMESTAMP` trong hầu hết trường hợp
4. `JSONB` hiệu quả hơn `JSON` cho tìm kiếm và index
5. `UUID` phù hợp cho primary key phân tán

## 5. Indexes và tối ưu truy vấn

### 5.1 Các loại index

**B-tree Index (mặc định):**
```sql
CREATE INDEX idx_customers_email ON customers(email);
```
- Phù hợp cho các toán tử: =, <, <=, >, >=, BETWEEN, IN, LIKE (pattern bắt đầu với hằng số)
- Tốt nhất cho các truy vấn tìm kiếm chính xác và dải giá trị

**Hash Index:**
```sql
CREATE INDEX idx_customers_email_hash ON customers USING HASH (email);
```
- Chỉ phù hợp cho toán tử bằng (=)
- Nhanh hơn B-tree cho các truy vấn bằng đơn giản

**GiST (Generalized Search Tree):**
```sql
CREATE INDEX idx_locations_position ON locations USING GIST (position);
```
- Phù hợp cho dữ liệu không có thứ tự tự nhiên (dữ liệu hình học, text full-text search)
- Hỗ trợ các operator: &&, @>, <@, ~=, ...

**GIN (Generalized Inverted Index):**
```sql
CREATE INDEX idx_documents_content ON documents USING GIN (to_tsvector('english', content));
```
- Tối ưu cho truy vấn có nhiều khóa trùng lặp (array, jsonb, full-text search)
- Tốn nhiều không gian lưu trữ và thời gian cập nhật hơn

**BRIN (Block Range Index):**
```sql
CREATE INDEX idx_logs_timestamp ON logs USING BRIN (timestamp);
```
- Phù hợp cho bảng lớn với dữ liệu được sắp xếp tự nhiên
- Nhỏ hơn nhiều so với B-tree, hiệu quả cho bảng hàng TB

**Partial Index:**
```sql
CREATE INDEX idx_invoices_unpaid ON invoices (invoice_date) WHERE status = 'unpaid';
```
- Chỉ lập chỉ mục cho một tập hợp con của bảng
- Tiết kiệm không gian và cải thiện hiệu suất insert/update

**Multi-column Index:**
```sql
CREATE INDEX idx_customers_name ON customers(last_name, first_name);
```
- Hữu ích cho truy vấn theo nhiều cột
- Thứ tự cột quan trọng: cột hay được dùng trong WHERE đầu tiên

**Expression Index:**
```sql
CREATE INDEX idx_customers_lower_email ON customers(lower(email));
```
- Lập chỉ mục cho biểu thức thay vì giá trị cột
- Sử dụng khi truy vấn thường xuyên dùng hàm trên cột

### 5.2 Tối ưu và phân tích truy vấn (EXPLAIN)

**EXPLAIN:**
```sql
EXPLAIN SELECT * FROM customers WHERE email = 'user@example.com';
```

**EXPLAIN ANALYZE:**
```sql
EXPLAIN ANALYZE SELECT * 
FROM orders o 
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date > '2023-01-01';
```

**EXPLAIN VERBOSE:**
```sql
EXPLAIN (ANALYZE, VERBOSE, BUFFERS) 
SELECT * FROM orders WHERE total_amount > 1000;
```

**Các thuật ngữ trong EXPLAIN:**
- **Seq Scan**: Quét tuần tự toàn bộ bảng
- **Index Scan**: Sử dụng index để tìm kiếm
- **Index Only Scan**: Chỉ đọc từ index, không truy cập heap
- **Bitmap Index Scan**: Tạo bitmap từ nhiều index scans
- **Nested Loop**: Join từng cặp hàng
- **Hash Join**: Build hash table trước, rồi probe
- **Merge Join**: Sort rồi merge hai tập đã sắp xếp

**Cải thiện hiệu suất truy vấn:**
1. **Tránh SELECT ***: Chỉ chọn cột cần thiết
2. **Kết hợp nhiều truy vấn**: Sử dụng JOIN thay vì nhiều truy vấn riêng lẻ
3. **Sử dụng prepared statements**: Tái sử dụng execution plans
4. **Tránh sử dụng hàm trên WHERE**: Nếu có thể, viết WHERE col = value thay vì WHERE function(col) = value
5. **Sử dụng LIMIT**: Giới hạn kết quả trả về

**Ví dụ tối ưu truy vấn:**
```sql
-- Trước khi tối ưu
SELECT * 
FROM orders 
WHERE EXTRACT(YEAR FROM order_date) = 2023;

-- Sau khi tối ưu
SELECT order_id, customer_id, order_date, total_amount
FROM orders 
WHERE order_date BETWEEN '2023-01-01' AND '2023-12-31';
```

### 5.3 Quản lý index và bảo trì

**Kiểm tra kích thước index:**
```sql
SELECT
    indexrelname AS index_name,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    idx_scan AS index_scans
FROM pg_stat_user_indexes
JOIN pg_statio_user_indexes USING (indexrelid)
ORDER BY pg_relation_size(indexrelid) DESC;
```

**Tìm index không được sử dụng:**
```sql
SELECT
    idstat.schemaname AS schema_name,
    idstat.relname AS table_name,
    indexrelname AS index_name,
    pg_size_pretty(pg_relation_size(idstat.indexrelid)) AS index_size,
    idx_scan AS index_scans
FROM pg_stat_user_indexes idstat
JOIN pg_statio_user_indexes statio ON idstat.indexrelid = statio.indexrelid
WHERE idx_scan = 0
ORDER BY pg_relation_size(idstat.indexrelid) DESC;
```

**Rebuild index:**
```sql
REINDEX INDEX idx_customers_email;
REINDEX TABLE customers;
REINDEX SCHEMA public;
```

**Quản lý index trong bảng lớn:**
1. **Tạo index concurrently**: Không khóa bảng khi tạo
   ```sql
   CREATE INDEX CONCURRENTLY idx_logs_timestamp ON logs(timestamp);
   ```

2. **Xóa index concurrently**:
   ```sql
   DROP INDEX CONCURRENTLY idx_logs_timestamp;
   ```

3. **Tạo và thay thế index**:
   ```sql
   CREATE INDEX idx_customers_email_new ON customers(lower(email));
   DROP INDEX idx_customers_email;
   ALTER INDEX idx_customers_email_new RENAME TO idx_customers_email;
   ```

## 6. Tối ưu hiệu suất PostgreSQL

### 6.1 Tối ưu bộ nhớ

**shared_buffers**:
- Đây là bộ nhớ đệm được chia sẻ giữa các connections
- Thường đặt = 25% RAM (server dedicated cho PostgreSQL)
- Không nên quá lớn để tránh swapping
```
shared_buffers = 2GB   # 25% của 8GB RAM
```

**work_mem**:
- Bộ nhớ cho các thao tác sắp xếp và hash
- Mỗi connection và mỗi bước của query plan có thể sử dụng đến work_mem
- Công thức: RAM / (max_connections * số thao tác cần bộ nhớ trung bình)
```
work_mem = 20MB        # Với 100 connections
```

**maintenance_work_mem**:
- Bộ nhớ cho các thao tác bảo trì (VACUUM, CREATE INDEX, ALTER TABLE)
- Thường lớn hơn work_mem nhiều
```
maintenance_work_mem = 256MB
```

**effective_cache_size**:
- Ước tính bộ nhớ cache hệ điều hành sẵn có (không phải cấu hình)
- Thường đặt khoảng 50-75% RAM
```
effective_cache_size = 6GB  # 75% của 8GB RAM
```

**Các tham số bộ nhớ khác:**
```
temp_buffers = 16MB           # Bộ nhớ cho các temporary tables
max_prepared_transactions = 0 # Số lượng transactions có thể prepare (2-phase commit)
```

### 6.2 Tối ưu I/O và Disk

**WAL (Write-Ahead Log) Settings:**
```
# Kích thước tối đa trước khi checkpoint
max_wal_size = 1GB

# Kích thước tối thiểu để giữ WAL files
min_wal_size = 80MB

# Mức độ chi tiết của WAL
wal_level = replica  # minimal, replica, logical

# Đồng bộ WAL với disk
synchronous_commit = on  # on, off, remote_apply, remote_write, local
```

**Checkpoint Settings:**
```
# Khoảng thời gian giữa các checkpoints tự động
checkpoint_timeout = 5min

# Số lượng WAL segments giữa các checkpoint
checkpoint_completion_target = 0.9  # Kéo dài thời gian checkpoint
```

**Storage Parameters:**
```
# Tắt fsync trong trường hợp recovery từ crash không quan trọng
fsync = on  # KHÔNG ĐƯỢC TẮT trong production

# Phương thức ghi trang
full_page_writes = on  # Nên để on trừ khi storage có battery-backed cache
```

**Tối ưu tablespaces:**
- Tách biệt WAL và data directories trên các ổ đĩa khác nhau
- Đặt các bảng hay truy cập trên ổ SSD
- Đặt các bảng ít truy cập trên ổ HDD

```sql
-- Tạo tablespace trên ổ SSD
CREATE TABLESPACE fastspace LOCATION '/ssd/postgresql/data';

-- Chuyển bảng thường xuyên truy cập qua tablespace mới
ALTER TABLE active_users SET TABLESPACE fastspace;
```

### 6.3 Vacuum và Autovacuum

**VACUUM là gì?**
- Quá trình dọn dẹp "rác" (dead tuples) và lấy lại không gian
- Cần thiết vì MVCC (Multi-Version Concurrency Control)
- Ngăn chặn transaction ID wraparound và bloat

**VACUUM thủ công:**
```sql
-- Xóa dead tuples, cập nhật statistics
VACUUM ANALYZE mytable;

-- Xóa dead tuples, nén lại bảng (chậm, khóa bảng)
VACUUM FULL mytable;

-- Chỉ xóa dead tuples, không cập nhật statistics
VACUUM mytable;
```

**Cấu hình Autovacuum:**
```
# Bật/tắt autovacuum
autovacuum = on

# Số workers tối đa
autovacuum_max_workers = 3

# Thời gian giữa các lần chạy
autovacuum_naptime = 1min

# Số transactions trước khi autovacuum chạy
autovacuum_vacuum_threshold = 50
autovacuum_analyze_threshold = 50

# Phần trăm của bảng thay đổi trước khi autovacuum chạy
autovacuum_vacuum_scale_factor = 0.1  # 10% của bảng
autovacuum_analyze_scale_factor = 0.05  # 5% của bảng
```

**Cấu hình Autovacuum cho từng bảng:**
```sql
ALTER TABLE mytable SET (
  autovacuum_vacuum_threshold = 100,
  autovacuum_vacuum_scale_factor = 0.2,
  autovacuum_analyze_threshold = 50,
  autovacuum_analyze_scale_factor = 0.1
);
```

**Monitoring Autovacuum:**
```sql
-- Xem trạng thái autovacuum
SELECT datname, usename, pid, state, query, xact_start, query_start
FROM pg_stat_activity
WHERE query LIKE 'autovacuum:%';

-- Xem thông tin vacuum trên từng bảng
SELECT relname, n_live_tup, n_dead_tup, last_vacuum, last_autovacuum
FROM pg_stat_user_tables;
```

### 6.4 Cấu hình connection và performance

**Connection Settings:**
```
# Số lượng connections tối đa
max_connections = 100

# Thời gian chờ tối đa cho connection
tcp_keepalives_idle = 60
tcp_keepalives_interval = 10
tcp_keepalives_count = 6

# Connection pooling (nên dùng pgBouncer)
```

**Statement và Transaction Timeout:**
```
# Thời gian tối đa một statement được phép chạy
statement_timeout = 60s

# Thời gian tối đa một transaction được phép chạy
idle_in_transaction_session_timeout = 60s

# Thời gian tối đa chờ lock
lock_timeout = 10s
```

**Track và Log Performance:**
```
# Enable query statistics
track_activities = on
track_counts = on
track_io_timing = on

# Log slow queries
log_min_duration_statement = 1000  # Log queries chạy > 1000ms
```

**Sử dụng Connection Poolers:**
- **pgBouncer**: Nhẹ, đơn giản, hiệu quả
- **Odyssey**: Cao cấp, ưu tiên bảo mật
- **PgPool-II**: Nhiều tính năng (load balancing, query caching)

**Cài đặt pgBouncer:**
```bash
sudo apt-get install pgbouncer

# Cấu hình /etc/pgbouncer/pgbouncer.ini
[databases]
mydb = host=localhost port=5432 dbname=mydb

[pgbouncer]
listen_addr = *
listen_port = 6432
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 20
```

## 7. Replication và High Availability

### 7.1 Streaming Replication

**Cấu hình Primary Server (postgresql.conf):**
```
listen_addresses = '*'
wal_level = replica
max_wal_senders = 10
wal_keep_size = 1GB
```

**Cấu hình Primary Server (pg_hba.conf):**
```
host    replication     replicator      192.168.1.0/24        md5
```

**Tạo user replication:**
```sql
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'password';
```

**Cài đặt Standby Server:**
1. Tạo base backup:
```bash
pg_basebackup -h primary_host -U replicator -D /var/lib/postgresql/14/main -P -v -R
```

2. Cấu hình standby (postgresql.conf):
```
primary_conninfo = 'host=primary_host port=5432 user=replicator password=password'
```

3. Tạo standby.signal (PostgreSQL 12+):
```bash
touch /var/lib/postgresql/14/main/standby.signal
```

4. Khởi động standby server:
```bash
systemctl start postgresql
```

**Monitoring Replication:**
```sql
-- Trên Primary
SELECT client_addr, state, sent_lsn, write_lsn, flush_lsn, replay_lsn
FROM pg_stat_replication;

-- Trên Standby
SELECT 
    pg_last_wal_receive_lsn(),
    pg_last_wal_replay_lsn(),
    pg_last_xact_replay_timestamp();
```

**Replication Slots:**
```sql
-- Tạo replication slot trên Primary
SELECT pg_create_physical_replication_slot('standby1_slot');

-- Cấu hình Standby sử dụng slot
primary_conninfo = 'host=primary_host port=5432 user=replicator password=password application_name=standby1'
primary_slot_name = 'standby1_slot'
```

### 7.2 Logical Replication

**Cấu hình Publisher (postgresql.conf):**
```
wal_level = logical
max_replication_slots = 10
max_wal_senders = 10
```

**Tạo Publication trên Publisher:**
```sql
-- Cho toàn bộ bảng trong schema
CREATE PUBLICATION my_publication FOR ALL TABLES IN SCHEMA public;

-- Hoặc cho một số bảng cụ thể
CREATE PUBLICATION my_publication FOR TABLE customers, orders;
```

**Tạo Subscription trên Subscriber:**
```sql
CREATE SUBSCRIPTION my_subscription
CONNECTION 'host=publisher_host port=5432 dbname=mydb user=replicator password=password'
PUBLICATION my_publication;
```

**Monitoring Logical Replication:**
```sql
-- Trên Publisher
SELECT * FROM pg_publication;
SELECT * FROM pg_publication_tables;
SELECT slot_name, plugin, slot_type, active FROM pg_replication_slots;

-- Trên Subscriber
SELECT * FROM pg_subscription;
SELECT * FROM pg_stat_subscription;
```

### 7.3 Các công cụ HA và Load Balancing

**Patroni:**
- Framework HA cho PostgreSQL sử dụng DCS (Distributed Consensus Store)
- Tự động failover và leader election

**Cài đặt Patroni:**
```bash
pip install patroni[etcd]

# Tạo file cấu hình patroni.yml
cat > /etc/patroni.yml << EOF
scope: postgres-cluster
namespace: /db/
name: node1

restapi:
  listen: 0.0.0.0:8008
  connect_address: 192.168.1.10:8008

etcd:
  host: 192.168.1.20:2379

bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
      parameters:
        max_connections: 100
        shared_buffers: 1GB
  initdb:
    - encoding: UTF8
    - data-checksums

postgresql:
  listen: 0.0.0.0:5432
  connect_address: 192.168.1.10:5432
  data_dir: /var/lib/postgresql/14/main
  bin_dir: /usr/lib/postgresql/14/bin
  pgpass: /tmp/pgpass
  authentication:
    replication:
      username: replicator
      password: password
    superuser:
      username: postgres
      password: password
  parameters:
    wal_level: replica
EOF

# Chạy Patroni
patroni /etc/patroni.yml
```

**HAProxy:**
- Load balancer cho cluster PostgreSQL
- Chuyển hướng write requests đến primary, read requests đến standby

```
# Cấu hình HAProxy
listen postgres
  bind *:5432
  option httpchk
  http-check expect status 200
  default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
  server node1 192.168.1.10:5432 maxconn 100 check port 8008
  server node2 192.168.1.11:5432 maxconn 100 check port 8008
  server node3 192.168.1.12:5432 maxconn 100 check port 8008
```

**pgpool-II:**
- Middleware cung cấp connection pooling, load balancing, và high availability
- Có thể thực hiện online recovery và query caching

## 8. Backup và Restore

### 8.1 Phương thức backup

**pg_dump (logical backup):**
```bash
# Backup một database
pg_dump -h localhost -U postgres -d mydb -F c -f mydb.backup

# Backup toàn bộ cluster
pg_dumpall -h localhost -U postgres > full_backup.sql
```

**Các format pg_dump:**
- `-F p`: Plain SQL (mặc định)
- `-F c`: Custom format (nén, parallel restore)
- `-F d`: Directory format (parallel dump và restore)
- `-F t`: TAR format

**pg_basebackup (physical backup):**
```bash
# Full backup
pg_basebackup -h localhost -U replicator -D /backup/base -P -v

# Với WAL streaming
pg_basebackup -h localhost -U replicator -D /backup/base -P -v -X stream
```

**Sử dụng continuous archiving:**
1. Cấu hình WAL archiving (postgresql.conf):
```
wal_level = replica
archive_mode = on
archive_command = 'test ! -f /archive/%f && cp %p /archive/%f'
```

2. Thực hiện base backup:
```bash
pg_basebackup -h localhost -U postgres -D /backup/base -P -v
```

### 8.2 Point-in-Time Recovery

**Cấu hình archive_command:**
```
archive_command = 'test ! -f /archive/%f && cp %p /archive/%f'
```

**Thực hiện recovery:**
1. Restore base backup:
```bash
rsync -av /backup/base/ /var/lib/postgresql/14/main/
```

2. Tạo recovery.conf (PostgreSQL < 12) hoặc cấu hình postgresql.conf (PostgreSQL ≥ 12):
```
# PostgreSQL < 12 (recovery.conf)
restore_command = 'cp /archive/%f %p'
recovery_target_time = '2023-03-16 15:30:00'

# PostgreSQL ≥ 12 (postgresql.conf)
restore_command = 'cp /archive/%f %p'
recovery_target_time = '2023-03-16 15:30:00'
```

3. Tạo signal file (PostgreSQL ≥ 12):
```bash
touch /var/lib/postgresql/14/main/recovery.signal
```

4. Khởi động PostgreSQL:
```bash
systemctl start postgresql
```

5. Sau khi recovery hoàn tất, chuyển sang mode bình thường:
```sql
SELECT pg_wal_replay_resume();  -- PostgreSQL < 12
SELECT pg_promote();  -- PostgreSQL ≥ 12
```

### 8.3 Chiến lược backup và restore

**Chiến lược backup hỗn hợp:**

1. **Weekly full backup:**
```bash
pg_basebackup -h localhost -U replicator -D /backup/base_$(date +%Y%m%d) -P -v
```

2. **Daily incremental backup (WAL files):**
- Được thực hiện tự động qua archive_command

3. **Hourly logical backup của tables quan trọng:**
```bash
pg_dump -h localhost -U postgres -d mydb -t important_table -F c -f /backup/important_$(date +%Y%m%d%H).backup
```

**Automation với pgBackRest:**
```bash
# Cài đặt
apt-get install pgbackrest

# Cấu hình /etc/pgbackrest.conf
[global]
repo1-path=/var/lib/pgbackrest
repo1-retention-full=2

[mycluster]
pg1-path=/var/lib/postgresql/14/main

# Thực hiện full backup
pgbackrest --stanza=mycluster backup --type=full

# Thực hiện incremental backup
pgbackrest --stanza=mycluster backup --type=incr

# Restore
pgbackrest --stanza=mycluster restore --target-time="2023-03-16 15:30:00"
```

**Kiểm tra tính toàn vẹn của backup:**
```bash
# Kiểm tra logical backup
pg_restore -l mydb.backup

# Thử restore vào database tạm
createdb test_restore
pg_restore -d test_restore mydb.backup

# Kiểm tra physical backup
pgbackrest --stanza=mycluster check
```

## 9. Bảo mật PostgreSQL

### 9.1 Xác thực và phân quyền

**Phương thức xác thực (pg_hba.conf):**
```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             postgres                                peer
local   all             all                                     md5
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             ::1/128                 scram-sha-256
host    replication     replicator      192.168.1.0/24          scram-sha-256
```

**Tạo role và user:**
```sql
-- Tạo role
CREATE ROLE app_readonly;
GRANT CONNECT ON DATABASE mydb TO app_readonly;
GRANT USAGE ON SCHEMA public TO app_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO app_readonly;

-- Tạo user
CREATE USER john WITH PASSWORD 'secure_password';
GRANT app_readonly TO john;

-- Tạo user với quyền quản trị
CREATE USER admin WITH PASSWORD 'very_secure_password' SUPERUSER;
```

**Quản lý quyền:**
```sql
-- Cấp quyền cụ thể
GRANT SELECT, INSERT, UPDATE ON TABLE customers TO app_readwrite;
GRANT EXECUTE ON FUNCTION calculate_total(integer) TO app_readwrite;

-- Thu hồi quyền
REVOKE UPDATE ON TABLE customers FROM app_readwrite;

-- Xem quyền
\dp customers
SELECT * FROM information_schema.table_privileges WHERE table_name = 'customers';
```

**Quản lý passwords:**
```sql
-- Đổi password
ALTER USER john WITH PASSWORD 'new_secure_password';

-- Đặt thời hạn password
ALTER USER john VALID UNTIL '2023-12-31';

-- Thiết lập encryption mặc định
SET password_encryption = 'scram-sha-256';
```

### 9.2 Mã hóa và SSL

**Bật SSL (postgresql.conf):**
```
ssl = on
ssl_cert_file = 'server.crt'
ssl_key_file = 'server.key'
ssl_ca_file = 'root.crt'
```

**Tạo SSL certificates:**
```bash
# Tạo CA
openssl req -new -x509 -days 365 -nodes -out root.crt -keyout root.key -subj "/CN=postgres-ca"

# Tạo server key
openssl req -new -nodes -out server.csr -keyout server.key -subj "/CN=postgres-server"
openssl x509 -req -in server.csr -days 365 -CA root.crt -CAkey root.key -CAcreateserial -out server.crt

# Tạo client key
openssl req -new -nodes -out client.csr -keyout client.key -subj "/CN=postgres-client"
openssl x509 -req -in client.csr -days 365 -CA root.crt -CAkey root.key -CAcreateserial -out client.crt
```

**Yêu cầu SSL (pg_hba.conf):**
```
hostssl all             all             0.0.0.0/0               scram-sha-256 clientcert=verify-ca
```

**Data-at-rest encryption:**
- Sử dụng LUKS hoặc dm-crypt cho disk encryption
- Sử dụng pgcrypto extension cho column-level encryption

```sql
-- Cài đặt pgcrypto
CREATE EXTENSION pgcrypto;

-- Mã hóa dữ liệu
INSERT INTO secure_data (id, encrypted_data)
VALUES (1, pgp_sym_encrypt('sensitive data', 'password'));

-- Giải mã dữ liệu
SELECT id, pgp_sym_decrypt(encrypted_data, 'password') FROM secure_data;
```

### 9.3 Row-level security

**Bật Row-Level Security:**
```sql
-- Bật RLS trên bảng
ALTER TABLE customer_data ENABLE ROW LEVEL SECURITY;

-- Tạo policy
CREATE POLICY customer_data_policy ON customer_data
    USING (customer_id = current_setting('app.current_customer_id')::integer);

-- Policy cho từng loại thao tác
CREATE POLICY customer_data_view ON customer_data
    FOR SELECT
    USING (customer_id = current_setting('app.current_customer_id')::integer);

CREATE POLICY customer_data_update ON customer_data
    FOR UPDATE
    USING (customer_id = current_setting('app.current_customer_id')::integer);
```

**Sử dụng RLS với application users:**
```sql
-- Tạo table với ownership phù hợp
CREATE TABLE customer_data (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    data TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Bypass RLS cho owner
ALTER TABLE customer_data FORCE ROW LEVEL SECURITY;

-- Thiết lập current_customer_id tại runtime
SET app.current_customer_id = '42';
```

**Giám sát RLS:**
```sql
-- Xem các policies
SELECT * FROM pg_policies;

-- Kiểm tra quyền truy cập
EXPLAIN (ANALYZE, VERBOSE) SELECT * FROM customer_data;
```

## 10. Giám sát và Maintenance

### 10.1 Giám sát với Prometheus và Grafana

**Cài đặt postgres_exporter:**
```bash
# Tải postgres_exporter
wget https://github.com/prometheus-community/postgres_exporter/releases/download/v0.11.1/postgres_exporter-0.11.1.linux-amd64.tar.gz
tar xvf postgres_exporter-0.11.1.linux-amd64.tar.gz
cp postgres_exporter-0.11.1.linux-amd64/postgres_exporter /usr/local/bin/

# Tạo user và database
CREATE USER postgres_exporter WITH PASSWORD 'password';
GRANT pg_monitor TO postgres_exporter;

# Tạo .pgpass file
echo "*:5432:postgres:postgres_exporter:password" > ~/.pgpass
chmod 600 ~/.pgpass

# Tạo service
cat > /etc/systemd/system/postgres_exporter.service << EOF
[Unit]
Description=Prometheus PostgreSQL Exporter
After=network.target

[Service]
User=postgres
Environment=DATA_SOURCE_NAME="user=postgres_exporter host=localhost port=5432 dbname=postgres sslmode=disable"
ExecStart=/usr/local/bin/postgres_exporter
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# Start service
systemctl daemon-reload
systemctl enable postgres_exporter
systemctl start postgres_exporter
```

**Cấu hình Prometheus:**
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'postgres'
    static_configs:
      - targets: ['localhost:9187']
        labels:
          instance: 'postgres_main'
```

**Dashboard Grafana cho PostgreSQL:**
- Import dashboard ID 9628, 455, hoặc 12485
- Tạo alerts cho:
  - Connection utilization > 80%
  - Replication lag > 30s
  - Transaction ID wraparound < 10,000,000
  - Deadlocks
  - Slow queries

**Các metrics quan trọng:**

| Metric | Mô tả | Threshold cảnh báo |
|--------|-------|-------------------|
| **Connections** |||
| pg_stat_activity_count | Số lượng connections | >80% max_connections |
| pg_stat_activity_max_tx_duration | Transaction thời gian dài | >5 phút |
| **Replication** |||
| pg_stat_replication_lag_bytes | Độ trễ replication theo bytes | >50MB |
| pg_stat_replication_lag_time | Độ trễ replication theo thời gian | >30s |
| **Database Health** |||
| pg_stat_database_deadlocks | Số deadlocks | >0 |
| pg_stat_database_blks_hit_ratio | Buffer cache hit ratio | <0.98 |
| **Transaction IDs** |||
| pg_database_age_oldest_xid | Tuổi của oldest XID | >10,000,000 |
| **Disk Usage** |||
| pg_database_size_bytes | Kích thước database | >80% disk space |
| **Performance** |||
| pg_stat_statements_seconds_total | Thời gian thực thi query | Queries chậm >1s |
| pg_locks_count | Số locks | Tăng đột biến |

### 10.2 Bảo trì định kỳ

**VACUUM và ANALYZE thủ công:**
```sql
-- Bảo trì toàn bộ database
VACUUM ANALYZE;

-- Bảo trì bảng cụ thể
VACUUM ANALYZE mytable;

-- Xử lý bloat (nếu cần)
VACUUM FULL mytable;
```

**Maintenance cơ sở dữ liệu:**
```sql
-- Xây dựng lại indices
REINDEX TABLE mytable;

-- Cập nhật thông tin thống kê
ANALYZE mytable;

-- Reclaim không gian và freeze tuples cũ
VACUUM FREEZE mytable;
```

**Kiểm tra và xử lý transaction ID wraparound:**
```sql
-- Kiểm tra tuổi của transaction IDs
SELECT datname, age(datfrozenxid) FROM pg_database ORDER BY age(datfrozenxid) DESC;

-- Nếu quá cao, chạy
VACUUM FREEZE;
```

**Lập lịch bảo trì:** 
```bash
# Crontab để chạy vacuum và analyze vào mỗi đêm
0 2 * * * psql -c "VACUUM ANALYZE;"

# Chạy VACUUM FREEZE hàng tuần
0 1 * * 0 psql -c "VACUUM FREEZE;"

# Kiểm tra và cảnh báo transaction ID wraparound
0 */6 * * * psql -c "SELECT datname, age(datfrozenxid) FROM pg_database ORDER BY age(datfrozenxid) DESC LIMIT
```

**Quản lý logs:**
```bash
# Xoay vòng logs hàng tuần
0 0 * * 0 find /var/log/postgresql -name "*.log" -mtime +7 -exec gzip {} \;
```

### 10.3 Xử lý sự cố

**Xác định và kill queries đang chạy lâu:**
```sql
-- Tìm queries chạy lâu
SELECT pid, now() - pg_stat_activity.query_start AS duration, query
FROM pg_stat_activity
WHERE state = 'active' AND now() - pg_stat_activity.query_start > interval '5 minutes';

-- Kill query cụ thể
SELECT pg_cancel_backend(12345);  -- Cancel nhẹ nhàng
SELECT pg_terminate_backend(12345);  -- Forced termination
```

**Xác định deadlocks và blocking:**
```sql
-- Tìm blocking transactions
SELECT blocked_locks.pid AS blocked_pid,
         blocked_activity.usename AS blocked_user,
         blocking_locks.pid AS blocking_pid,
         blocking_activity.usename AS blocking_user,
         blocked_activity.query AS blocked_statement,
         blocking_activity.query AS blocking_statement
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.DATABASE IS NOT DISTINCT FROM blocked_locks.DATABASE
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
    AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
    AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
    AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
    AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
    AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
    AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.GRANTED;
```

**Xác định table bloat:**
```sql
SELECT schemaname, tablename, 
    pg_size_pretty(pg_total_relation_size(schemaname || '.' || tablename)) as total_size,
    pg_size_pretty(pg_relation_size(schemaname || '.' || tablename)) as table_size,
    pg_size_pretty(pg_total_relation_size(schemaname || '.' || tablename) - 
                  pg_relation_size(schemaname || '.' || tablename)) as index_size,
    round(100 * pg_relation_size(schemaname || '.' || tablename) / 
          pg_total_relation_size(schemaname || '.' || tablename)) as table_percentage
FROM pg_stat_user_tables
ORDER BY pg_total_relation_size(schemaname || '.' || tablename) DESC
LIMIT 10;
```

**Xử lý ổ đĩa đầy:**
```bash
# Tìm các file WAL lớn
find /var/lib/postgresql/14/main/pg_wal -type f -name "*" -ls | sort -k 7 -r | head -10

# Xóa bỏ các bảng tạm nếu cần
psql -c "SELECT pg_size_pretty(pg_relation_size(oid)), relname FROM pg_class WHERE relkind = 'r' AND relname LIKE 'pg_temp%' ORDER BY pg_relation_size(oid) DESC;"

# Có thể mở rộng không gian disk
lvextend -L +10G /dev/mapper/vg-postgres
resize2fs /dev/mapper/vg-postgres
```

**Recovery sau crash:**
```bash
# Kiểm tra logs để xác định nguyên nhân
tail -100 /var/log/postgresql/postgresql-14-main.log

# Kiểm tra consistency
systemctl stop postgresql
cd /var/lib/postgresql/14/main
sudo -u postgres /usr/lib/postgresql/14/bin/pg_checksums --check

# Nếu cần, thực hiện recovery
cp -a /backup/base/* /var/lib/postgresql/14/main/
systemctl start postgresql
```

## 11. Tính năng nâng cao

### 11.1 Partitioning

**Tạo Partitioned Table:**
```sql
-- Tạo parent table
CREATE TABLE measurements (
    city_id         int not null,
    logdate         date not null,
    temperature     int,
    humidity        int
) PARTITION BY RANGE (logdate);

-- Tạo partitions
CREATE TABLE measurements_y2022m01 PARTITION OF measurements
    FOR VALUES FROM ('2022-01-01') TO ('2022-02-01');

CREATE TABLE measurements_y2022m02 PARTITION OF measurements
    FOR VALUES FROM ('2022-02-01') TO ('2022-03-01');
    
CREATE TABLE measurements_y2022m03 PARTITION OF measurements
    FOR VALUES FROM ('2022-03-01') TO ('2022-04-01');
```

**Range Partitioning:**
```sql
CREATE TABLE sales (
    sale_id         SERIAL,
    sale_date       DATE NOT NULL,
    amount          DECIMAL(10,2)
) PARTITION BY RANGE (sale_date);

CREATE TABLE sales_q1_2023 PARTITION OF sales
    FOR VALUES FROM ('2023-01-01') TO ('2023-04-01');
    
CREATE TABLE sales_q2_2023 PARTITION OF sales
    FOR VALUES FROM ('2023-04-01') TO ('2023-07-01');
```

**List Partitioning:**
```sql
CREATE TABLE customers (
    customer_id     SERIAL,
    country_code    CHAR(2) NOT NULL,
    name            TEXT
) PARTITION BY LIST (country_code);

CREATE TABLE customers_asia PARTITION OF customers
    FOR VALUES IN ('CN', 'JP', 'KR', 'VN', 'TH');
    
CREATE TABLE customers_europe PARTITION OF customers
    FOR VALUES IN ('DE', 'FR', 'IT', 'ES', 'UK');
```

**Hash Partitioning:**
```sql
CREATE TABLE orders (
    order_id        SERIAL,
    customer_id     INTEGER NOT NULL,
    order_date      DATE,
    total_amount    DECIMAL(12,2)
) PARTITION BY HASH (customer_id);

CREATE TABLE orders_p0 PARTITION OF orders
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);
    
CREATE TABLE orders_p1 PARTITION OF orders
    FOR VALUES WITH (MODULUS 4, REMAINDER 1);
    
CREATE TABLE orders_p2 PARTITION OF orders
    FOR VALUES WITH (MODULUS 4, REMAINDER 2);
    
CREATE TABLE orders_p3 PARTITION OF orders
    FOR VALUES WITH (MODULUS 4, REMAINDER 3);
```

**Quản lý Partitioning:**
```sql
-- Thêm partition mới
CREATE TABLE measurements_y2022m04 PARTITION OF measurements
    FOR VALUES FROM ('2022-04-01') TO ('2022-05-01');
    
-- Gỡ bỏ partition
ALTER TABLE measurements DETACH PARTITION measurements_y2022m01;

-- Xóa dữ liệu cũ dễ dàng
DROP TABLE measurements_y2022m01;
```

### 11.2 Foreign Data Wrappers

**Cài đặt postgres_fdw:**
```sql
CREATE EXTENSION postgres_fdw;
```

**Kết nối đến PostgreSQL từ xa:**
```sql
-- Tạo foreign server
CREATE SERVER foreign_server
    FOREIGN DATA WRAPPER postgres_fdw
    OPTIONS (host 'remote_host', port '5432', dbname 'remote_db');
    
-- Tạo user mapping
CREATE USER MAPPING FOR local_user
    SERVER foreign_server
    OPTIONS (user 'remote_user', password 'password');
    
-- Tạo foreign table
CREATE FOREIGN TABLE remote_customers (
    id INTEGER NOT NULL,
    name TEXT,
    email TEXT
)
SERVER foreign_server
OPTIONS (schema_name 'public', table_name 'customers');
```

**Làm việc với foreign tables:**
```sql
-- Truy vấn bình thường
SELECT * FROM remote_customers;

-- Join với local tables
SELECT l.order_id, l.order_date, r.name
FROM local_orders l
JOIN remote_customers r ON l.customer_id = r.id;
```

**MySQL Foreign Data Wrapper:**
```sql
-- Cài đặt mysql_fdw
CREATE EXTENSION mysql_fdw;

-- Tạo foreign server
CREATE SERVER mysql_server
    FOREIGN DATA WRAPPER mysql_fdw
    OPTIONS (host 'mysql_host', port '3306');
    
-- Tạo user mapping
CREATE USER MAPPING FOR postgres
    SERVER mysql_server
    OPTIONS (username 'mysql_user', password 'password');
    
-- Tạo foreign table
CREATE FOREIGN TABLE mysql_users (
    id INTEGER,
    username TEXT,
    email TEXT
)
SERVER mysql_server
OPTIONS (database 'mysql_db', table_name 'users');
```

**File Foreign Data Wrapper:**
```sql
-- Cài đặt file_fdw
CREATE EXTENSION file_fdw;

-- Tạo foreign server
CREATE SERVER file_server
    FOREIGN DATA WRAPPER file_fdw;
    
-- Tạo foreign table
CREATE FOREIGN TABLE csv_data (
    id INTEGER,
    name TEXT,
    value NUMERIC
)
SERVER file_server
OPTIONS (filename '/path/to/file.csv', format 'csv', header 'true');
```

### 11.3 Extensions

**Extension Management:**
```sql
-- Liệt kê các extensions có sẵn
SELECT * FROM pg_available_extensions;

-- Liệt kê các extensions đã cài đặt
SELECT * FROM pg_extension;

-- Cài đặt extension
CREATE EXTENSION pg_stat_statements;

-- Gỡ bỏ extension
DROP EXTENSION pg_stat_statements;
```

**Một số extensions hữu ích:**

1. **pg_stat_statements**: Thống kê và giám sát queries
```sql
CREATE EXTENSION pg_stat_statements;

-- Xem thông tin các queries chạy lâu nhất
SELECT query, calls, total_time, mean_time
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;
```

2. **PostGIS**: Hỗ trợ dữ liệu địa lý
```sql
CREATE EXTENSION postgis;

-- Tạo bảng với dữ liệu địa lý
CREATE TABLE locations (
    id SERIAL PRIMARY KEY,
    name TEXT,
    location GEOMETRY(Point, 4326)
);

-- Thêm dữ liệu
INSERT INTO locations (name, location)
VALUES ('Eiffel Tower', ST_SetSRID(ST_MakePoint(2.2945, 48.8584), 4326));

-- Tìm kiếm dựa trên khoảng cách
SELECT name, ST_Distance(
    location::geography,
    ST_SetSRID(ST_MakePoint(2.3522, 48.8566), 4326)::geography
) AS distance_in_meters
FROM locations
ORDER BY distance_in_meters;
```

3. **pgcrypto**: Mã hóa dữ liệu
```sql
CREATE EXTENSION pgcrypto;

-- Mã hóa password
SELECT crypt('my_password', gen_salt('bf'));

-- Kiểm tra password
SELECT (crypt('my_password', stored_password) = stored_password) AS password_match
FROM users WHERE username = 'john';
```

4. **pg_partman**: Quản lý partition tự động
```sql
CREATE EXTENSION pg_partman;

-- Tạo partitioned table
CREATE TABLE time_series (
    id SERIAL,
    created_at TIMESTAMP NOT NULL,
    value NUMERIC
) PARTITION BY RANGE (created_at);

-- Tự động tạo partitions theo ngày
SELECT partman.create_parent(
    'public.time_series',
    'created_at',
    'daily',
    'native',
    p_start_date := CURRENT_DATE
);

-- Tạo maintenance cron job
UPDATE partman.part_config
SET retention = '30 days', retention_keep_table = false
WHERE parent_table = 'public.time_series';
```

5. **pg_repack**: Tổ chức lại bảng và indices mà không cần lock
```sql
CREATE EXTENSION pg_repack;

-- Reorganize table
SELECT pg_repack.repack_table('mytable');
```

### 11.4 JSON và tính năng NoSQL

**Lưu trữ và truy vấn JSON:**
```sql
-- Tạo bảng với JSON data
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    data JSONB
);

-- Thêm dữ liệu
INSERT INTO products (name, data)
VALUES (
    'Smartphone',
    '{"brand": "Samsung", "model": "Galaxy S21", "specs": {"cpu": "Exynos", "ram": 8, "storage": 128}}'
);

-- Truy vấn JSON
SELECT name, data->'brand' AS brand, data->'specs'->>'ram' AS ram
FROM products;

-- Tìm kiếm dựa trên JSON
SELECT * FROM products
WHERE data @> '{"brand": "Samsung"}';

-- Lọc dựa trên JSON values
SELECT * FROM products
WHERE (data->'specs'->>'ram')::int > 6;
```

**JSON Indexing:**
```sql
-- GIN index cho tìm kiếm '@>' operator
CREATE INDEX idx_products_data ON products USING GIN (data);

-- Index cho expressions cụ thể
CREATE INDEX idx_products_brand ON products ((data->>'brand'));
```

**JSON Aggregation:**
```sql
-- Array to JSON
SELECT json_agg(name) FROM products;

-- JSON Object
SELECT json_object_agg(name, data) FROM products;

-- Tổng hợp JSON từ hai bảng
SELECT c.name, 
       json_agg(json_build_object('id', p.id, 'name', p.name, 'price', p.price))
FROM categories c
JOIN products p ON c.id = p.category_id
GROUP BY c.name;
```

**JSON Generation Functions:**
```sql
-- Tạo JSON từ rows
SELECT row_to_json(p) FROM products p;

-- Tạo JSON từ các giá trị
SELECT json_build_object(
    'product_id', id,
    'product_name', name,
    'details', data
) FROM products;

-- Tạo JSON từ arrays
SELECT json_build_array(id, name, data) FROM products;
```

---

Tài liệu này cung cấp một hướng dẫn toàn diện về PostgreSQL từ cài đặt cơ bản đến các tính năng nâng cao. Tùy thuộc vào nhu cầu cụ thể, bạn có thể cần điều chỉnh các tham số và cấu hình phù hợp với workload và môi trường của mình.
