# Essential Metrics Monitoring Guide

This guide provides a comprehensive list of key metrics to monitor for various systems and technologies to ensure optimal performance, reliability, and availability.

## Table of Contents
- [Database Systems](#database-systems)
  - [MySQL](#mysql)
  - [PostgreSQL](#postgresql)
  - [MongoDB](#mongodb)
- [Message Queue & Streaming](#message-queue--streaming)
  - [Kafka](#kafka)
- [Search & Analytics](#search--analytics)
  - [Elasticsearch](#elasticsearch)
- [Caching](#caching)
  - [Redis](#redis)
- [Infrastructure](#infrastructure)
  - [Operating System](#operating-system)
- [Application Performance](#application-performance)
  - [Application Metrics](#application-metrics)
- [Container Orchestration](#container-orchestration)
  - [Kubernetes](#kubernetes)
- [Web Servers & Proxies](#web-servers--proxies)
  - [Nginx](#nginx)
  - [Ingress Nginx](#ingress-nginx)
- [Service Mesh & Network](#service-mesh--network)
  - [Istio](#istio)
  - [Cilium](#cilium)

## Database Systems

### MySQL

| Metric | Description | Warning Threshold | Critical Threshold | Troubleshooting |
|--------|-------------|-------------------|---------------------|----------------|
| **Connection & Threads** |||||
| Connections | Số lượng kết nối được thử thiết lập | >80% max_connections | >90% max_connections | Kiểm tra connection pool của ứng dụng; Tăng max_connections nếu cần; Tìm và khắc phục các kết nối treo |
| Threads_connected | Số lượng kết nối đang mở | >80% max_connections | >90% max_connections | Kiểm tra kết nối không sử dụng (SHOW PROCESSLIST); Tăng timeout hoặc giảm connection từ ứng dụng |
| Threads_running | Số lượng thread đang chạy truy vấn | >30 | >50 | Xem truy vấn đang chạy (SHOW PROCESSLIST); Tối ưu truy vấn gây tắc nghẽn (EXPLAIN); Kiểm tra locks |
| Aborted_connects | Số lượng kết nối thất bại | >0 | >10/phút | Kiểm tra error log; Xác minh quyền truy cập; Kiểm tra cấu hình firewall; Kiểm tra giới hạn kết nối OS |
| **Query Performance** |||||
| Slow_queries | Số lượng truy vấn vượt quá long_query_time | >0 | >10/phút | Phân tích slow query log; Chạy EXPLAIN cho các truy vấn; Tạo index nếu cần; Tối ưu các truy vấn |
| Questions | Số lượng câu lệnh được thực thi | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Xem có tiến trình bất thường không; Kiểm tra các job định kỳ; Xem xét tải của ứng dụng |
| Com_* | Số lượng các câu lệnh cụ thể (select, insert, v.v.) | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Kiểm tra ứng dụng gây tăng đột biến; Theo dõi các câu lệnh cụ thể để phát hiện vấn đề |
| **Cache Efficiency** |||||
| Innodb_buffer_pool_hit_ratio | Tỷ lệ % đọc từ bộ đệm so với từ đĩa | <95% | <90% | Tăng kích thước innodb_buffer_pool_size; Tối ưu schema và truy vấn; Thực hiện dọn dẹp dữ liệu |
| Innodb_buffer_pool_utilization | Tỷ lệ % sử dụng buffer pool | >95% | >98% | Tăng kích thước innodb_buffer_pool_size; Giảm bớt bảng trong database |
| Table_open_cache_hits / misses | Hiệu quả cache cho table handlers | hits / (hits + misses) < 0.9 | hits / (hits + misses) < 0.8 | Tăng giá trị table_open_cache; Xem xét số bảng ứng dụng sử dụng |
| **Storage & I/O** |||||
| Innodb_data_reads/writes | Số lượng thao tác đọc/ghi đĩa | Tăng đột ngột >30% | Tăng đột ngột >50% | Kiểm tra tải I/O hệ thống; Tối ưu truy vấn; Tăng innodb_buffer_pool_size; Xem xét nâng cấp phần cứng |
| Innodb_log_waits | Số lần chờ do buffer log quá nhỏ | >0 | >10 | Tăng innodb_log_buffer_size; Kiểm tra workload ghi; Điều chỉnh innodb_flush_log_at_trx_commit |
| Created_tmp_disk_tables | Số bảng tạm được tạo trên đĩa (không phải trong bộ nhớ) | >10/giây | >20/giây | Tăng tmp_table_size và max_heap_table_size; Tối ưu các truy vấn GROUP BY/ORDER BY; Thêm index |
| **Locking & Contention** |||||
| Innodb_row_lock_waits | Số lần phải chờ khóa hàng | >0 | >10/phút | Tìm transaction gây khóa (SHOW ENGINE INNODB STATUS); Tối ưu truy vấn; Giảm thời gian transaction |
| Innodb_row_lock_time_avg | Thời gian trung bình để có được khóa hàng (ms) | >10ms | >50ms | Tối ưu truy vấn gây khóa; Chia nhỏ transaction lớn; Xem xét điều chỉnh isolation level |
| Table_locks_waited | Số lần chờ khóa bảng | >0 | >10/phút | Chuyển đổi các bảng MyISAM sang InnoDB; Xem xét các truy vấn gây khóa bảng; Tối ưu các thao tác DDL |
| **Replication** |||||
| Seconds_Behind_Master | Độ trễ sao chép theo giây | >30 giây | >300 giây | Kiểm tra tải I/O trên replica; Tìm truy vấn dài gây delay; Tăng hiệu suất máy replica; Kiểm tra bottleneck mạng |
| Slave_IO_Running | Trạng thái luồng IO | Khác "Yes" | Khác "Yes" trong >5 phút | Kiểm tra kết nối mạng; Xem error log; Đảm bảo user replication có quyền đủ; Kiểm tra file binary logs |
| Slave_SQL_Running | Trạng thái luồng SQL | Khác "Yes" | Khác "Yes" trong >5 phút | Kiểm tra error trên replica; Xem có truy vấn không tương thích không; Sửa lỗi data inconsistency |
| **Error Rates** |||||
| Ongoing Deadlocks | Giá trị từ SHOW ENGINE INNODB STATUS | >0 | >5/giờ | Xem chi tiết deadlock từ logs; Tối ưu truy vấn; Đảm bảo truy cập dữ liệu theo cùng thứ tự; Thêm index |
| Foreign key errors | Số lượng từ error log | >0 | >10 | Xem error log; Kiểm tra ràng buộc dữ liệu nhập vào từ ứng dụng; Đảm bảo dữ liệu nhất quán |
| **General Server Stats** |||||
| Uptime | Thời gian hoạt động của server tính bằng giây | Khởi động lại không mong muốn | Nhiều lần khởi động lại | Kiểm tra system logs; Xem OOM errors; Kiểm tra watchdog; Xem xét tăng ổn định hệ thống |
| Available disk space | Không gian đĩa trống trên các volume cơ sở dữ liệu | <20% | <10% | Dọn dẹp binary logs; Xóa bỏ dump cũ; Nén/archive dữ liệu lâu ngày; Mở rộng disk space |

### PostgreSQL

| Metric | Description | Warning Threshold | Critical Threshold | Troubleshooting |
|--------|-------------|-------------------|---------------------|----------------|
| **Connection Management** |||||
| pg_stat_activity_count | Số lượng kết nối đang hoạt động | >80% max_connections | >90% max_connections | Kiểm tra các kết nối đang mở (pg_stat_activity); Cấu hình connection pooling; Tăng max_connections |
| idle_in_transaction_count | Số phiên trong trạng thái idle transaction | >5 | >20 | Kiểm tra các transaction treo; Điều chỉnh idle_in_transaction_session_timeout; Tìm ứng dụng giữ transaction mở |
| longest_transaction_duration | Thời gian của transaction dài nhất | >10 phút | >1 giờ | Kiểm tra transaction dài; Tối ưu business logic; Chia nhỏ batch operation; Đặt transaction timeout |
| **Query Performance** |||||
| pg_stat_statements_calls | Top queries theo số lần gọi | N/A | N/A | Xác định và tối ưu các truy vấn thường xuyên; Xem xét caching; Đánh giá xem các truy vấn có cần thiết không |
| pg_stat_statements_total_time | Top queries theo thời gian thực thi | >1 giây | >10 giây | Phân tích EXPLAIN; Thêm/sửa chỉ mục; Viết lại truy vấn; Cập nhật thống kê (ANALYZE) |
| slow_queries_rate | Các truy vấn chạy lâu hơn ngưỡng | >5/phút | >20/phút | Bật log_min_duration_statement; Chạy EXPLAIN ANALYZE; Xem xét thay đổi schema; Đánh index |
| **Cache Efficiency** |||||
| pg_stat_database_blks_hit_ratio | Tỷ lệ hit buffer cache | <0.98 | <0.95 | Tăng shared_buffers; VACUUM bảng phình; REINDEX chỉ mục; Kiểm tra bảng lớn và truy vấn xấu |
| temp_files_size | Kích thước file tạm được tạo | >100MB/phút | >1GB/phút | Tăng work_mem; Tối ưu truy vấn với JOIN lớn; Tránh DISTINCT không cần thiết |
| temp_bytes_rate | Tốc độ tạo dữ liệu tạm | >10MB/giây | >100MB/giây | Tăng work_mem; Tối ưu sắp xếp và hash; Kiểm tra tác vụ batch lớn; Thêm index phù hợp |
| **Lock Contention** |||||
| pg_locks_count | Số lượng khóa theo loại | Tăng >50% | Tăng >100% | Kiểm tra locks hiện tại (pg_locks); Tìm transaction chặn dài; Tối ưu truy vấn hoặc sơ đồ dữ liệu |
| pg_locks_waiting | Số phiên đang chờ khóa | >0 | >5 | Xác định các query gây khóa chặn (pg_blocking_pids()); Hủy transaction kẹt; Tối ưu tải công việc |
| lock_wait_time_max | Thời gian tối đa một truy vấn đã chờ khóa | >30 giây | >5 phút | Kiểm tra deadlock; Tìm transaction lớn; Xem xét các kỹ thuật tránh xung đột |
| **I/O Activity** |||||
| pg_stat_database_tup_fetched/returned | Số hàng được lấy/trả về | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Kiểm tra workload mới; Đánh giá hiệu suất truy vấn; Tối ưu index; Cấu hình autovacuum |
| pg_stat_database_tup_inserted/updated/deleted | Các thao tác ghi | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Kiểm tra batch process bất thường; Tối ưu multiple updates; Điều chỉnh WAL và checkpoint |
| checkpoint_write_time | Thời gian dành cho việc ghi checkpoint | >50% checkpoint_timeout | >80% checkpoint_timeout | Tăng checkpoint_timeout; Điều chỉnh max_wal_size; Đảm bảo I/O đủ nhanh; Phân phối tải đĩa |
| **Replication** |||||
| pg_stat_replication_lag_bytes | Độ trễ sao chép tính bằng bytes | >50MB | >500MB | Kiểm tra tải networkl; Xác nhận hiệu suất server replica; Xem xét việc giảm workload |
| pg_stat_replication_lag_time | Độ trễ sao chép tính bằng giây | >30 giây | >300 giây | Kiểm tra transaction lớn chưa commit; Tăng hiệu suất replica; Kiểm tra I/O bottleneck |
| max_replay_lag | Thời gian trễ dài nhất trong phát lại WAL | >60 giây | >300 giây | Tăng tài nguyên cho replica; Kiểm tra I/O; Đánh giá các truy vấn chạy trên replica |
| **VACUUM & Database Maintenance** |||||
| pg_stat_user_tables_dead_tup | Số lượng tuples "chết" | >10000 | >100000 | Điều chỉnh autovacuum_scale_factor; Chạy VACUUM thủ công; Xem xét giảm transaction dài |
| pg_stat_user_tables_autovacuum_count | Số lần chạy autovacuum | >10/giờ/bảng | >30/giờ/bảng | Điều chỉnh autovacuum; Tối ưu cấu hình vacuum; Giảm số lượng update/delete |
| pg_stat_database_age_frozen_xid | Tuổi của XID cũ nhất | >1,000,000,000 | >1,500,000,000 | Chạy VACUUM FREEZE; Điều chỉnh autovacuum_freeze_max_age; Giám sát wraparound |
| **Table & Index Stats** |||||
| table_bloat_percentage | Phần trăm không gian bảng bị phình to | >30% | >50% | Chạy VACUUM FULL (khi ít traffic); Thiết lập fillfactor; Rebuilt bảng |
| index_bloat_percentage | Phần trăm không gian chỉ mục bị phình to | >30% | >50% | Chạy REINDEX; Đặt lịch index maintenance; Tối ưu workload ghi |
| index_scans_vs_seq_scans_ratio | Tỷ lệ quét chỉ mục so với quét tuần tự | <0.7 | <0.5 | Kiểm tra các chỉ mục thiếu; Phân tích truy vấn; Cập nhật thống kê; Tối ưu index |
| **Transaction IDs** |||||
| transaction_wraparound | Thời gian cho đến khi ID giao dịch bị cuộn tròn | <1 tháng | <1 tuần | Chạy VACUUM FREEZE; Xem xét vacuum aggressive; Check autovacuum; Lên kế hoạch đặt lịch vacuum |
| **Storage** |||||
| pg_database_size_bytes | Kích thước cơ sở dữ liệu | >80% dung lượng đĩa | >90% dung lượng đĩa | Xóa temporary files; Dọn WAL; Dọn logs; Thêm storage; Archive dữ liệu cũ |
| wal_files_count | Số lượng tệp WAL | >50 | >100 | Kiểm tra checkpoint; Tăng max_wal_size; Kiểm tra replica lag; Xóa WAL cũ nếu có thể |
| wal_bytes_rate | Tốc độ tạo WAL | Thay đổi đột ngột >50% | >100MB/giây | Giảm transaction rate; Giới hạn workload; Kiểm tra dữ liệu bulk; Tối ưu DDL và DML |

### MongoDB

| Metric | Description | Warning Threshold | Critical Threshold | Troubleshooting |
|--------|-------------|-------------------|---------------------|----------------|
| **Connection & Resources** |||||
| connections.current | Số lượng kết nối hiện tại | >80% connections.totalCreated | >90% connections.totalCreated | Tăng limit kết nối; Thiết lập connection pool; Kiểm tra ứng dụng không đóng kết nối |
| connections.available | Kết nối còn khả dụng | <100 | <20 | Tăng giới hạn kết nối (maxIncomingConnections); Giảm kết nối không cần thiết |
| globalLock.currentQueue | Số thao tác đang chờ khóa | >10 | >20 | Tìm các hoạt động blocking; Tái cấu trúc hoạt động ghi; Tối ưu truy vấn; Thêm index |
| **Operation Performance** |||||
| opcounters.query | Số thao tác truy vấn mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Tìm client gây tăng tải; Kiểm tra truy vấn không hiệu quả; Xem xét caching |
| opcounters.insert | Số thao tác chèn mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Kiểm tra bulk inserts; Tối ưu index; Xem xét tải của ứng dụng |
| opcounters.update | Số thao tác cập nhật mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Kiểm tra mass updates; Tìm update không có index; Xem xét pattern ghi |
| opcounters.delete | Số thao tác xóa mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Kiểm tra bulk deletes; Xem xét pattern xóa; Tối ưu index cho các truy vấn xóa |
| **Query Efficiency** |||||
| metrics.queryExecutor.scanned | Số tài liệu được quét | >1000/truy vấn | >10000/truy vấn | Thêm index phù hợp; Tối ưu truy vấn để giảm scan; Xem xét projection |
| metrics.queryExecutor.scannedObjects | Số đối tượng được quét | >1000/truy vấn | >10000/truy vấn | Thêm index để giảm object scan; Xem xét covered query; Tối ưu truy vấn |
| ratio_scanned_to_returned | Tỷ lệ tài liệu quét so với trả về | >100:1 | >1000:1 | Thêm/tối ưu index; Giới hạn truy vấn; Tái cấu trúc schema; Dùng aggregation pipeline |
| **Memory Usage** |||||
| mem.resident | Lượng bộ nhớ resident sử dụng | >80% bộ nhớ hệ thống | >90% bộ nhớ hệ thống | Giới hạn bộ nhớ WiredTiger; Xác định data size lớn; Sharding data; Tăng RAM |
| mem.virtual | Lượng bộ nhớ ảo sử dụng | >2x bộ nhớ resident | >4x bộ nhớ resident | Kiểm tra memory mapping; Giảm dataset kích thước lớn; Kiểm tra cấu hình OS |
| wiredTiger.cache.maximum | Giới hạn cache WiredTiger | N/A | N/A | Điều chỉnh wiredTigerCacheSizeGB; Mốc khởi đầu: 50-60% của RAM |
| wiredTiger.cache.used | Lượng cache WiredTiger đã sử dụng | >80% wiredTiger.cache.maximum | >90% wiredTiger.cache.maximum | Tăng kích thước cache; Xác định workload gây sử dụng cao; Tối ưu truy vấn |
| **Replication** |||||
| repl.lag | Độ trễ sao chép | >10 giây | >60 giây | Kiểm tra tải trên primary; Tăng tài nguyên cho secondary; Kiểm tra bottleneck network |
| repl.oplog.window | Cửa sổ oplog tính bằng giờ | <24 giờ | <12 giờ | Tăng kích thước oplog; Giảm lượng hoạt động ghi trên primary; Tăng số lượng secondary |
| **Storage** |||||
| metrics.storage.freelist.search.bucketExhausted | Chỉ số phân mảnh | >0 | >10/giờ | Chạy compact; Lập kế hoạch defragment; Tái cấu trúc dữ liệu; Tối ưu schema |
| db.stats.storageSize | Tổng kích thước lưu trữ | >80% dung lượng đĩa | >90% dung lượng đĩa | Xóa temporary files; Compact collections; Thêm storage; Archive dữ liệu cũ |
| db.stats.indexSize | Tổng kích thước chỉ mục | >5GB mỗi chỉ mục | >10GB mỗi chỉ mục | Đánh giá độ cần thiết của index; Tối ưu index; Giám sát index usage; Xóa index không sử dụng |
| **Backend Operations** |||||
| backgroundFlushing.average_ms | Thời gian flush trung bình | >100ms | >500ms | Kiểm tra storage I/O; Phân phối tải I/O; Xem xét I/O scheduling; Tăng throughput disk |
| metrics.cursor.open.total | Số lượng con trỏ đang mở | >10000 | >50000 | Kiểm tra ứng dụng không đóng cursor; Giảm số lượng truy vấn dài; Xác định cursor leak |
| metrics.cursor.timedOut | Số con trỏ hết thời gian | >0 | >10/giờ | Tăng thời gian timeout; Tối ưu truy vấn lâu; Giới hạn kết quả trả về |
| **WiredTiger Specific** |||||
| wiredTiger.concurrentTransactions.read.available | Số lượng vé đọc khả dụng | <10 | <5 | Giảm số truy vấn đồng thời; Tối ưu thời gian truy vấn; Tối ưu batch operations |
| wiredTiger.concurrentTransactions.write.available | Số lượng vé ghi khả dụng | <10 | <5 | Giảm số write đồng thời; Trải đều write operations; Phân phối workload |
| wiredTiger.cache.eviction-worker-running | Số luồng eviction worker đang chạy | >50% số lõi CPU | N/A | Tăng kích thước cache WiredTiger; Kiểm tra workload ghi; Cân nhắc sharding |

## Message Queue & Streaming

### Kafka

| Metric | Description | Warning Threshold | Critical Threshold | Troubleshooting |
|--------|-------------|-------------------|---------------------|----------------|
| **Broker Metrics** |||||
| UnderReplicatedPartitions | Số lượng partition có replica không đủ | >0 | >10 | Xác định broker có vấn đề; Kiểm tra tài nguyên broker; Kiểm tra kết nối mạng; Cân bằng lại partitions |
| OfflinePartitionsCount | Số lượng partition không có leader đang hoạt động | >0 | >1 | Xác định broker chết; Kiểm tra lỗi ZooKeeper; Phục hồi broker bị lỗi; Kiểm tra disk errors |
| ActiveControllerCount | Số lượng controller đang hoạt động (nên là 1) | !=1 | N/A | Kiểm tra ZooKeeper connection; Xem log controller; Restart controller nếu cần |
| RequestHandlerAvgIdlePercent | % thời gian rảnh trung bình của request handler | <30% | <10% | Thêm broker; Tăng num.io.threads; Phân phối lại partitions; Tối ưu application client |
| NetworkProcessorAvgIdlePercent | % thời gian rảnh trung bình của network processor | <30% | <10% | Tăng num.network.threads; Kiểm tra network bottleneck; Thêm broker; Tối ưu request size |
| **Topic & Partition Metrics** |||||
| MessagesInPerSec | Tốc độ tin nhắn đến | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Xác định client gây tăng; Tăng partition count; Thêm broker; Tối ưu batch size |
| BytesInPerSec | Số bytes nhận vào mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Kiểm tra bottleneck network; Xem xét compression; Kiểm tra workload bất thường |
| BytesOutPerSec | Số bytes gửi đi mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Kiểm tra số consumer tăng; Giám sát hiệu suất disk; Tối ưu fetch size |
| LogSize | Kích thước log của partition | >80% dung lượng đĩa | >90% dung lượng đĩa | Tăng dung lượng ổ đĩa; Giảm thời gian lưu giữ; Xem xét log compaction; Tối ưu message size |
| PartitionCount | Số lượng partition trên một broker | >4000 | >8000 | Tái cân bằng partitions; Thêm broker; Tối ưu lại thiết kế topic; Kiểm tra memory usage |
| LeaderCount | Số lượng leader partition trên một broker | Mất cân bằng >20% giữa các broker | Mất cân bằng >40% giữa các broker | Chạy leader rebalancing; Tái phân phối partitions; Kiểm tra health của từng broker |
| **Producer Metrics** |||||
| request-latency-avg | Độ trễ truy vấn trung bình | >20ms | >100ms | Kiểm tra CPU broker; Tối ưu I/O; Tăng batch size; Kiểm tra acks setting |
| request-rate | Số lượng yêu cầu mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Kiểm tra client mới; Tối ưu batch size; Xem xét linger.ms; Tăng parallelism |
| response-rate | Số lượng phản hồi mỗi giây | request-rate - response-rate > 0 | (request-rate - response-rate) / request-rate > 0.1 | Tăng tài nguyên broker; Kiểm tra timeout; Đánh giá sức khỏe broker |
| record-error-rate | Tỷ lệ lỗi của record | >0 | >0.01 | Kiểm tra lỗi serialization; Xem log lỗi; Xem dữ liệu message; Kiểm tra network issues |
| **Consumer Metrics** |||||
| records-lag-max | Độ trễ tối đa tính theo số lượng record | >1000 | >10000 | Tăng consumer instances; Tối ưu xử lý message; Tăng partition count; Scale consumer group |
| records-consumed-rate | Tốc độ tiêu thụ record | Giảm >30% | Giảm >50% | Tìm consumer chậm; Kiểm tra xử lý bottleneck; Tìm lỗi consumer; Tối ưu batch processing |
| bytes-consumed-rate | Số bytes tiêu thụ mỗi giây | Giảm >30% | Giảm >50% | Kiểm tra network throughput; Tối ưu message size; Tăng max.partition.fetch.bytes |
| fetch-rate | Số lượng yêu cầu fetch mỗi giây | Giảm >30% | Giảm >50% | Tối ưu consumer fetch config; Tăng max.poll.records; Kiểm tra processing time per record |
| **JVM Metrics** |||||
| heap-usage-bytes | Lượng heap JVM sử dụng | >80% max heap | >90% max heap | Tăng heap size; Chạy GC sớm hơn; Xác định memory leak; Tối ưu workload |
| gc-collection-time | Thời gian thu gom rác | >500ms | >1s | Tối ưu heap size; Điều chỉnh GC settings; Kiểm tra memory pressure; Giảm object allocation |
| **ZooKeeper Metrics** |||||
| ZooKeeperRequestLatencyMs | Độ trễ yêu cầu ZooKeeper | >10ms | >100ms | Kiểm tra ZooKeeper load; Tăng ZooKeeper resources; Tối ưu ensemble size; Kiểm tra disk I/O |
| ZooKeeperNumAliveConnections | Số lượng kết nối đang hoạt động tới ZooKeeper | <số lượng broker | <(số lượng broker - 2) | Kiểm tra network giữa broker và ZooKeeper; Xem log lỗi; Restart broker không kết nối |
| OutstandingRequests | Số lượng yêu cầu đang chờ xử lý | >10 | >20 | Tăng hiệu suất ZooKeeper; Kiểm tra tải trên ZooKeeper; Xem xét leader election |

## Search & Analytics

### Elasticsearch

| Metric | Description | Warning Threshold | Critical Threshold | Troubleshooting |
|--------|-------------|-------------------|---------------------|----------------|
| **Cluster Health** |||||
| cluster_health.status | Trạng thái tổng thể của cluster | Yellow | Red | Xác định unassigned shards; Kiểm tra node failure; Check disk space; Sửa cluster settings |
| cluster_health.initializing_shards | Số lượng shard đang khởi tạo | >0 trong >10 phút | >0 trong >30 phút | Xem node logs; Kiểm tra disk I/O; Xem CPU usage; Tăng resources cho recovery |
| cluster_health.relocating_shards | Số lượng shard đang di chuyển | >0 trong >10 phút | >0 trong >30 phút | Kiểm tra disk space; Xem cluster balance; Check network issues; Tăng hiệu suất network |
| cluster_health.unassigned_shards | Số lượng shard chưa được gán | >0 | >10 | Thêm node; Kiểm tra disk space; Check cluster.routing settings; Tăng node capacity |
| **Node Stats** |||||
| jvm.mem.heap_used_percent | Phần trăm heap JVM đã sử dụng | >80% | >90% | Tăng heap size; Tối ưu queries; Thêm nodes; Field data circuit breaker |
| jvm.gc.collectors.old.collection_time | Thời gian thu gom rác thế hệ cũ | >500ms | >2s | Tối ưu heap size; Theo dõi heap dumps; Điều chỉnh GC settings; Tìm memory leak |
| jvm.gc.collectors.old.collection_count | Số lần thu gom rác thế hệ cũ | >10/giờ | >30/giờ | Xem memory pressure; Tăng heap size; Tối ưu queries; Giảm field data cache |
| process.cpu.percent | Phần trăm CPU sử dụng | >80% | >90% | Kiểm tra queries phức tạp; Tăng node count; Tối ưu agregation; Thêm CPU capacity |
| **Indexing Performance** |||||
| indices.indexing.index_total | Tổng số tài liệu đã được đánh chỉ mục | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Xác định index flood; Tối ưu bulk indexing; Giám sát client behavior |
| indices.indexing.index_time_in_millis | Tổng thời gian dành cho việc đánh chỉ mục | >50ms trung bình mỗi tài liệu | >200ms trung bình mỗi tài liệu | Tối ưu mapping; Tăng refresh_interval; Điều chỉnh bulk size; Kiểm tra I/O |
| indices.indexing.index_current | Số lượng tài liệu đang được đánh chỉ mục | >50 | >200 | Kiểm tra indexing bottleneck; Theo dõi CPU usage; Theo dõi thread pool; Tăng concurrent indexing |
| indices.indexing.delete_total | Tổng số tài liệu đã bị xóa | Tăng đột ngột | N/A | Xác định bulk deletes; Xem xét dùng bulk APIs; Tối ưu delete-by-query |
| **Search Performance** |||||
| indices.search.query_total | Tổng số truy vấn | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Giám sát slow logs; Xác định client mới; Tối ưu query patterns; Caching |
| indices.search.query_time_in_millis | Tổng thời gian dùng cho truy vấn | >100ms trung bình mỗi truy vấn | >500ms trung bình mỗi truy vấn | Phân tích slow logs; Tối ưu query; Thêm filter cache; Điều chỉnh routing |
| indices.search.fetch_total | Tổng số thao tác fetch | N/A | N/A | Giảm size của documents; Dùng source filtering; Tối ưu fetch phase |
| indices.search.fetch_time_in_millis | Tổng thời gian dùng cho fetch | >50ms trung bình mỗi fetch | >200ms trung bình mỗi fetch | Giảm document size; Dùng _source filtering; Tối ưu stored fields |
| indices.search.scroll_total | Tổng số thao tác scroll | N/A | N/A | Tối ưu scroll size; Giảm open scrolls; Dùng PIT (point in time) thay thế |
| indices.search.scroll_time_in_millis | Thời gian dành cho thao tác scroll | >500ms trung bình mỗi scroll | >2s trung bình mỗi scroll | Điều chỉnh scroll size; Giảm document size; Giảm scroll timeout |
| **Memory Usage & Cache** |||||
| indices.fielddata.memory_size_in_bytes | Kích thước bộ đệm field data | >30% heap | >40% heap | Tăng circuit breaker; Tối ưu aggregations; Dùng doc values; Giới hạn fields |
| indices.fielddata.evictions | Số lần đẩy field data ra khỏi bộ đệm | >0 | >100 | Tăng field data cache size; Tối ưu aggregation; Dùng doc values; Giảm unique values |
| indices.query_cache.memory_size_in_bytes | Kích thước bộ đệm truy vấn | >10% heap | >20% heap | Tăng indices.queries.cache.size; Điều chỉnh query pattern; Xem xét hiệu suất cache |
| indices.query_cache.evictions | Số lần đẩy truy vấn ra khỏi bộ đệm | >1000/phút | >5000/phút | Tăng query cache size; Tối ưu các filter queries; Đánh giá cache hit ratio |
| indices.request_cache.memory_size_in_bytes | Kích thước bộ đệm yêu cầu | >5% heap | >10% heap | Tăng indices.requests.cache.size; Xem xét shard request cache |
| **Disk Usage** |||||
| fs.total.available_in_bytes | Dung lượng đĩa còn trống | <20% | <10% | Thêm disk space; Xóa indices cũ; Tiến hành forcemerge; Xóa tài liệu cũ |
| indices.store.size_in_bytes | Tổng kích thước của các chỉ mục | >80% dung lượng đĩa | >90% dung lượng đĩa | Xóa indices cũ; Snapshot và xóa data; Thêm storage; Giảm số replicas |
| **Circuit Breakers** |||||
| breakers.*.tripped | Số lần ngắt mạch | >0 | >5 | Tăng breaker limit; Tối ưu queries; Giảm độ phức tạp; Tìm queries gây nén bộ nhớ |
| **Thread Pool** |||||
| thread_pool.*.rejected | Số lượng tác vụ bị từ chối trong thread pool | >0 | >10 | Tăng thread pool queue size; Giảm concurrency; Tối ưu bulk requests; Giám sát CPU |
| thread_pool.*.queue | Kích thước hàng đợi của thread pool | >80% của tối đa | >90% của tối đa | Tăng queue size; Giảm traffic; Giảm kích thước batch; Thêm nodes |
| **Transport & HTTP** |||||
| transport.rx_size_in_bytes | Kích thước lưu lượng transport nhận được | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Kiểm tra internal traffic; Tối ưu số shards; Xem xét replica count; Kiểm tra network |
| transport.tx_size_in_bytes | Kích thước lưu lượng transport gửi đi | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Kiểm tra node communication; Tối ưu discovery; Kiểm tra replication; Tối ưu recovery |
| http.current_open | Số lượng kết nối HTTP đang mở | >80% tối đa | >90% tối đa | Tăng http.max_content_length; Kiểm tra client behavior; Tăng timeouts |

## Caching

### Redis

| Metric | Description | Warning Threshold | Critical Threshold | Troubleshooting |
|--------|-------------|-------------------|---------------------|----------------|
| **Server Health** |||||
| uptime_in_seconds | Thời gian hoạt động của server | Khởi động lại không mong muốn | Nhiều lần khởi động lại | Kiểm tra system logs; Xem lỗi OOM; Điều chỉnh cấu hình; Kiểm tra watchdog kills |
| connected_clients | Số lượng client đang kết nối | >80% maxclients | >90% maxclients | Tăng maxclients; Tìm các kết nối lâu; Kiểm tra connection leaks trong ứng dụng |
| blocked_clients | Số lượng client đang chờ trên thao tác blocking | >5 | >20 | Kiểm tra blocking operations; Xem xét lại thiết kế ứng dụng; Timeout cho BLPOP/BRPOP |
| rejected_connections | Số kết nối bị từ chối do vượt quá maxclients | >0 | >100 | Tăng maxclients; Tối ưu connection pooling; Kiểm tra connection leaks |
| **Memory Usage** |||||
| used_memory | Tổng lượng bộ nhớ Redis sử dụng | >80% maxmemory | >90% maxmemory | Tăng maxmemory; Xác định keys tiêu thụ nhiều bộ nhớ; Xóa dữ liệu không cần thiết |
| used_memory_rss | Tổng lượng bộ nhớ Redis sử dụng theo góc nhìn của OS | >150% used_memory | >200% used_memory | Chạy MEMORY PURGE; Kiểm tra memory fragmentation; Restart Redis nếu cần |
| mem_fragmentation_ratio | Tỷ lệ giữa RSS / bộ nhớ sử dụng | <0.8 hoặc >1.5 | <0.7 hoặc >2.0 | Chạy MEMORY PURGE; Thay đổi tải làm việc; Restart Redis nếu > 1.5; Thêm memory nếu < 1 |
| evicted_keys | Số lượng key bị xóa do giới hạn maxmemory | >0 | >1000/giờ | Tăng maxmemory; Điều chỉnh maxmemory-policy; Xem xét lại TTL keys; Tối ưu key size |
| expired_keys | Số lượng key đã hết hạn | Tăng đột ngột | N/A | Xem xét cơ chế expired_keys; Điều chỉnh TTL; Giám sát access patterns |
| **Command Execution** |||||
| instantaneous_ops_per_sec | Số lượng lệnh thực thi mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Tìm client gây tăng lưu lượng; Xem xét caching pattern; Tìm loops trong ứng dụng |
| keyspace_hits | Số lần tìm kiếm key thành công | N/A | N/A | Theo dõi cache hit ratio; Điều chỉnh TTL; Tiền tải cache |
| keyspace_misses | Số lần tìm kiếm key thất bại | keyspace_misses/keyspace_hits > 0.5 | keyspace_misses/keyspace_hits > 1 | Tăng key TTL; Giảm số lần xóa key; Xem xét warm-up cache; Kiểm tra eviction |
| **Persistence** |||||
| rdb_last_bgsave_status | Trạng thái của thao tác RDB save gần nhất | "error" | "error" cho nhiều lần liên tiếp | Kiểm tra disk space; Xem log lỗi; Sửa quyền file; Kiểm tra đường dẫn lưu |
| rdb_changes_since_last_save | Số thay đổi kể từ lần lưu trữ cuối cùng | >10,000 | >100,000 | Tăng tần suất RDB; Điều chỉnh save settings; Kiểm tra cơ chế backup |
| aof_last_write_status | Trạng thái của thao tác ghi AOF gần nhất | "error" | "error" cho nhiều lần liên tiếp | Kiểm tra disk space; Kiểm tra quyền; Xem log lỗi; Xác minh disk health |
| aof_last_rewrite_status | Trạng thái của thao tác viết lại AOF gần nhất | "error" | "error" cho nhiều lần liên tiếp | Kiểm tra disk space; Xem CPU usage; Xem memory usage trong quá trình rewrite |
| **Replication** |||||
| connected_slaves | Số lượng replica đang kết nối | <số lượng dự kiến | 0 (khi mong đợi có replica) | Kiểm tra kết nối replica; Xem network; Xem logs replica; Kiểm tra xác thực |
| master_link_status | Trạng thái kết nối với master | "down" | "down" trong >1 phút | Kiểm tra network giữa master-replica; Xem logs; Kiểm tra authen; Kiểm tra disk I/O |
| master_last_io_seconds_ago | Thời gian kể từ lần tương tác cuối cùng với master | >30 | >60 | Kiểm tra network; Xem replica logs; Kiểm tra master status; Xác minh port access |
| **Keyspace** |||||
| db{X}.keys | Số lượng key trong cơ sở dữ liệu X | Giảm đột ngột >10% | Giảm đột ngột >30% | Kiểm tra eviction; Xem workload pattern; Kiểm tra FLUSHDB/FLUSHALL; Theo dõi client |
| db{X}.expires | Số lượng key có thời hạn trong cơ sở dữ liệu X | N/A | N/A | Nếu quá thấp: Xem xét thêm TTL; Nếu quá cao: Điều chỉnh TTL dài hơn |
| **Latency** |||||
| latency_percentiles_usec | Phân vị độ trễ tính bằng microsecond | p99 > 1000 | p99 > 10000 | Kiểm tra commands chậm (SLOWLOG); Xem disk/network I/O; Tránh blocking ops; Giảm big keys |
| **Commands** |||||
| cmdstat_* | Thống kê về các lệnh cụ thể | N/A | N/A | Phân tích command patterns; Tìm non-optimized commands; Kiểm tra O(N) operations |
| **Sentinel Specific** |||||
| sentinel_masters | Số lượng master được giám sát bởi Sentinel | ≠ số lượng dự kiến | N/A | Kiểm tra Sentinel config; Xem Sentinel logs; Kiểm tra master status |
| sentinel_tilt | Cờ cho chế độ tilt của Sentinel | >0 | N/A | Kiểm tra system clock skew; Xem logs; Theo dõi system load; Kiểm tra network |
| sentinel_running_scripts | Số lượng script đang thực thi | >0 | >3 | Kiểm tra script chạy lâu; Xác định vấn đề notification; Giám sát script performance |

## Infrastructure

### Operating System

| Metric | Description | Warning Threshold | Critical Threshold | Troubleshooting |
|--------|-------------|-------------------|---------------------|----------------|
| **CPU Usage** |||||
| cpu_usage_user | Thời gian CPU dành cho không gian người dùng | >70% | >90% | Xác định process sử dụng CPU cao; Tối ưu application; Cân nhắc scale out; Kiểm tra CPU throttling |
| cpu_usage_system | Thời gian CPU dành cho kernel | >40% | >70% | Kiểm tra I/O; Xem xét kernel parameters; Kiểm tra syscalls quá mức; Xem xét driver issues |
| cpu_usage_iowait | Thời gian CPU chờ I/O | >20% | >40% | Xác định I/O bottleneck; Kiểm tra disk performance; Nâng cấp storage; Tối ưu I/O patterns |
| cpu_usage_steal | Thời gian CPU bị ảo hóa chiếm dụng | >5% | >20% | Di chuyển sang host khác; Kiểm tra neighbor VMs; Liên hệ cloud provider; Scale up instance |
| load_average | Tải trung bình của hệ thống | >số lõi CPU | >2*số lõi CPU | Xác định process gây tải; Tìm các hot spots; Scale hệ thống; Tối ưu workload |
| **Memory Usage** |||||
| memory_used_percent | Phần trăm bộ nhớ đã sử dụng | >85% | >95% | Tìm process sử dụng nhiều bộ nhớ; Kiểm tra memory leaks; Tăng RAM; Kiểm tra cache usage |
| memory_available_bytes | Bộ nhớ khả dụng | <10% tổng bộ nhớ | <5% tổng bộ nhớ | Giải phóng page cache; Giảm cached data; Kill/restart memory-intensive processes |
| swap_used_percent | Phần trăm swap đã sử dụng | >50% | >80% | Giảm memory pressure; Tối ưu applications; Tăng RAM; Điều chỉnh swappiness |
| swap_in_rate | Tốc độ swap vào từ đĩa | >10MB/s | >50MB/s | Tìm process swap-heavy; Tăng physical memory; Giảm memory usage; Kiểm tra memory leaks |
| swap_out_rate | Tốc độ swap ra đĩa | >10MB/s | >50MB/s | Tìm process gây swap; Tăng RAM; Tối ưu memory usage; Kiểm tra memory pressure |
| **Disk Usage** |||||
| disk_used_percent | Phần trăm dung lượng đĩa đã sử dụng | >80% | >90% | Dọn log files; Xóa temporary files; Nén data cũ; Thêm disk space; Di chuyển data |
| disk_inodes_used_percent | Phần trăm inode đã sử dụng | >80% | >90% | Tìm thư mục với nhiều file nhỏ; Dọn temporary files; Kiểm tra runaway processes |
| **Disk I/O** |||||
| disk_io_reads | Số thao tác đọc đĩa mỗi giây | Liên tục >1000 IOPS | Liên tục >5000 IOPS | Xác định process I/O-heavy; Tối ưu read patterns; Sử dụng caching; Nâng cấp disks |
| disk_io_writes | Số thao tác ghi đĩa mỗi giây | Liên tục >1000 IOPS | Liên tục >5000 IOPS | Kiểm tra process ghi nhiều; Batch write operations; Tối ưu fsync; Nâng cấp storage |
| disk_io_read_bytes | Thông lượng đọc đĩa | >100MB/s | >500MB/s | Xác định process đọc nhiều; Sử dụng buffer cache; Tối ưu application; Sử dụng SSD |
| disk_io_write_bytes | Thông lượng ghi đĩa | >100MB/s | >500MB/s | Tìm process ghi nhiều; Điều chỉnh buffer flush; Tối ưu logging; Nâng cấp disks |
| disk_io_await | Thời gian trung bình cho các yêu cầu I/O | >20ms | >100ms | Kiểm tra độ trễ disk; Xác định contention; Nâng cấp disk; Phân phối I/O đồng đều |
| disk_io_util | Phần trăm thời gian CPU dành cho các yêu cầu I/O | >80% | >95% | Xác định disk saturation; Nâng cấp disks; Sử dụng RAID; Phân phối workload I/O |
| **Network** |||||
| network_in_bytes | Lưu lượng mạng vào | >70% băng thông interface | >90% băng thông interface | Xác định traffic pattern; Tối ưu network stack; Tăng bandwidth; Giảm unnecessary traffic |
| network_out_bytes | Lưu lượng mạng ra | >70% băng thông interface | >90% băng thông interface | Xác định data transfer lớn; Kiểm tra outbound connections; Tối ưu packet size |
| network_in_errors | Lỗi gói tin vào | >0 | >100/phút | Kiểm tra network hardware; Xem xét driver issues; Kiểm tra MTU setting; Xem xét duplex mismatch |
| network_out_errors | Lỗi gói tin ra | >0 | >100/phút | Xem hardware issues; Kiểm tra network driver; Xem xét packet collisions; Kiểm tra CPU saturation |
| network_in_dropped | Gói tin vào bị drop | >0 | >100/phút | Tăng buffer size; Kiểm tra firewall; Kiểm tra NIC buffer; Theo dõi traffic bursts |
| network_out_dropped | Gói tin ra bị drop | >0 | >100/phút | Kiểm tra queue length; Kiểm tra flow control; Tối ưu TCP window; Tăng buffer size |
| **File System** |||||
| open_file_descriptors | Số lượng file descriptor đang mở | >80% tối đa | >90% tối đa | Tăng file descriptor limit; Kiểm tra process leaking FDs; Theo dõi application behavior |
| **Process** |||||
| process_count | Tổng số tiến trình | >500 | >1000 | Kiểm tra forking process; Xem xét runaway processes; Giới hạn short-lived processes |
| zombie_process_count | Số lượng tiến trình zombie | >5 | >20 | Kill parent process; Reboot nếu cần; Tìm parent process chưa wait() |
| **System** |||||
| uptime | Thời gian hoạt động của hệ thống | Khởi động lại không mong muốn | Nhiều lần khởi động lại | Kiểm tra kernel panic; Xem logs; Kiểm tra hardware failures; Theo dõi power issues |
| context_switches | Số lần chuyển đổi ngữ cảnh mỗi giây | Tăng liên tục >30% | Tăng liên tục >50% | Kiểm tra thread contention; Tối ưu threading model; Giảm task switching |
| interrupts | Số lượng ngắt mỗi giây | Tăng liên tục >30% | Tăng liên tục >50% | Kiểm tra driver issues; Xem xét hardware interrupts; Tối ưu IRQ balancing |

## Application Performance

### Application Metrics

| Metric | Description | Warning Threshold | Critical Threshold | Troubleshooting |
|--------|-------------|-------------------|---------------------|----------------|
| **Request Handling** |||||
| request_rate | Số lượng yêu cầu mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Xác định traffic pattern bất thường; Kiểm tra bot/ddos; Tìm client mới; Scale capacity |
| request_duration_seconds | Thời gian xử lý yêu cầu | p95 > SLO ứng dụng | p99 > SLO ứng dụng | Phân tích performance bottlenecks; Profiling code; Tối ưu database queries; Caching |
| request_errors_total | Tổng số lỗi yêu cầu | Tỷ lệ lỗi >1% | Tỷ lệ lỗi >5% | Kiểm tra error logs; Gỡ lỗi API; Xác minh input validation; Kiểm tra dependencies |
| request_timeout_total | Tổng số yêu cầu bị timeout | Tỷ lệ timeout >0.1% | Tỷ lệ timeout >1% | Tăng timeout limits; Xác định slow operations; Kiểm tra resource constraints; Circuit breakers |
| **Resource Usage** |||||
| app_cpu_usage | Mức sử dụng CPU của ứng dụng | >70% đã cấp phát | >90% đã cấp phát | Profiling để tìm CPU bottlenecks; Caching results; Tối ưu loops; Parallel processing |
| app_memory_usage | Mức sử dụng bộ nhớ của ứng dụng | >80% đã cấp phát | >90% đã cấp phát | Kiểm tra memory leaks; Heap dumps; Tối ưu object allocation; Garbage collection tuning |
| app_thread_count | Số lượng thread | >80% tối đa đã cấu hình | >90% tối đa đã cấu hình | Tối ưu thread pool; Kiểm tra thread leaks; Non-blocking I/O; Async processing |
| **Connection Pools** |||||
| db_connections_active | Số kết nối cơ sở dữ liệu đang hoạt động | >80% kích thước pool | >90% kích thước pool | Tăng connection pool size; Tìm long-running queries; Giảm connection hold time |
| db_connections_idle | Số kết nối cơ sở dữ liệu đang rảnh | <10% kích thước pool | <5% kích thước pool | Tăng pool size; Tối ưu connection usage; Giảm transaction time |
| db_connection_wait_time | Thời gian chờ kết nối cơ sở dữ liệu | >10ms | >100ms | Tăng pool size; Giảm thời gian giữ connection; Tối ưu DB queries |
| **Garbage Collection** |||||
| gc_collection_count | Số lượng garbage collection | >10/phút | >30/phút | Tối ưu object allocation; Tăng heap size; Điều chỉnh GC parameters; Memory profiling |
| gc_collection_time | Thời gian dành cho garbage collection | >10% thời gian CPU | >20% thời gian CPU | Tuning GC; Tối ưu object lifecycle; Xem xét memory leaks; Reduce object churn |
| **Business Metrics** |||||
| transaction_rate | Số giao dịch nghiệp vụ mỗi giây | Giảm đột ngột >20% | Giảm đột ngột >50% | Kiểm tra user behavior; Xác minh business flows; Xem xét bug lỗi; Kiểm tra chuyển đổi |
| transaction_error_rate | Số giao dịch nghiệp vụ thất bại | >1% | >5% | Xem log lỗi chi tiết; Xác minh input validation; Kiểm tra tích hợp; Sửa bugs |
| user_login_success_rate | Tỷ lệ đăng nhập thành công | <95% | <90% | Kiểm tra authentication issues; Xác minh user store; Theo dõi brute force; Kiểm tra UX |
| checkout_success_rate | Tỷ lệ thanh toán thành công | <95% | <90% | Đánh giá payment gateway; Kiểm tra validation flow; Xem xét UX issues; Theo dõi fraud detection |
| **Caching** |||||
| cache_hit_ratio | Tỷ lệ cache hit | <80% | <60% | Tăng TTL; Preloading cache; Tối ưu cache key; Warm up caches; Xem xét cache invalidation |
| cache_miss_rate | Số lần cache miss mỗi giây | Tăng đột ngột >30% | Tăng đột ngột >50% | Tăng cache size; Điều chỉnh cache policy; Tối ưu cache key distribution |
| **External Dependencies** |||||
| external_service_response_time | Thời gian phản hồi của dịch vụ bên ngoài | >500ms | >2s | Timeouts; Retry logic; Circuit breakers; Fallbacks; Liên hệ service provider |
| external_service_error_rate | Tỷ lệ lỗi khi gọi dịch vụ bên ngoài | >1% | >5% | Retry logic; Circuit breaking; Fallbacks; Kiểm tra hợp đồng API; Liên hệ provider |
| **Queue/Async Processing** |||||
| queue_size | Số lượng mục trong hàng đợi | >1000 | >10000 | Tăng consumer count; Tối ưu xử lý message; Giảm producer rate; Scale out |
| queue_latency | Thời gian tin nhắn nằm trong hàng đợi | >30s | >5phút | Thêm consumers; Tối ưu processing time; Sắp xếp ưu tiên messages; Tăng parallel processing |
| queue_consumer_lag | Độ trễ của consumer (tin nhắn) | >1000 | >10000 | Tăng consumer instances; Scale consumer group; Tối ưu consumer code; Batch processing |
| **Circuit Breakers** |||||
| circuit_breaker_open_count | Số lượng circuit breaker đang mở | >0 | >3 | Xác định root cause dependency failures; Troubleshoot external services; Implement fallbacks |
| circuit_breaker_half_open_count | Số lượng circuit breaker đang nửa mở | >3 | >5 | Theo dõi half-open circuit breakers; Troubleshoot dependency services |
| **Custom Business KPIs** |||||
| active_users | Số lượng người dùng đang hoạt động | Giảm đột ngột >20% | Giảm đột ngột >50% | Kiểm tra user experience; Xem xét lỗi ứng dụng; Theo dõi analytics; Check outages |
| conversion_rate | Tỷ lệ chuyển đổi người dùng | Giảm đột ngột >10% | Giảm đột ngột >30% | Đánh giá user journey; A/B testing; Kiểm tra usability; Xác định dropoff points |
| average_order_value | Giá trị trung bình của đơn hàng | Giảm đột ngột >10% | Giảm đột ngột >30% | Phân tích pricing; Xem xét product mix; Kiểm tra promotional impact; Market changes |

## Container Orchestration

### Kubernetes

| Metric | Description | Warning Threshold | Critical Threshold | Troubleshooting |
|--------|-------------|-------------------|---------------------|----------------|
| **Node Status** |||||
| node_status_condition | Điều kiện của node (Ready, DiskPressure, v.v.) | Not Ready | Not Ready trong >5 phút | Kiểm tra kubelet logs; Xem system logs; Check container runtime; Drain node nếu cần |
| node_cpu_usage_percentage | Phần trăm sử dụng CPU | >80% | >90% | Tìm pod tiêu thụ nhiều; Thêm resource limits; Scale out nodes; Tối ưu workloads |
| node_memory_usage_percentage | Phần trăm sử dụng bộ nhớ | >80% | >90% | Xác định pod sử dụng nhiều; Kiểm tra memory leaks; Thêm limits; Scale nodes |
| node_disk_usage_percentage | Phần trăm sử dụng đĩa | >80% | >90% | Tìm pod tiêu thụ nhiều disk; Dọn container logs; Xóa unused images; Thêm storage |
| node_pod_capacity_usage | Phần trăm capacity pod đã sử dụng | >80% | >90% | Tăng max pods per node; Thêm nodes; Tối ưu pod packing; Giảm pod count |
| **Pod Status** |||||
| pod_status_phase | Trạng thái hiện tại của pod | Không phải Running | Không phải Running trong >5 phút | Xem pod logs; Kiểm tra events; Check image; Xác minh health check |
| pod_status_ready | Trạng thái sẵn sàng của pod | Không Ready | Không Ready trong >5 phút | Kiểm tra readiness probe; Xem container logs; Check dependencies; Check resource |
| pod_container_status_waiting | Trạng thái chờ của container | True với CrashLoopBackOff | True với CrashLoopBackOff trong >5 phút | Xem container logs; Tìm application crash; Check environment vars; Kiểm tra image |
| pod_container_status_terminated | Trạng thái kết thúc của container | Liên tục bị kết thúc | Liên tục bị kết thúc với lỗi | Kiểm tra exit code; Xem logs; Check resource requirements; Xác minh health |
| pod_restarts_total | Tổng số lần khởi động lại pod | >5 trong 1 giờ | >10 trong 1 giờ | Xem logs trước khi restart; Check memory limits; Xác định application crashes |
| **Resource Usage** |||||
| pod_cpu_usage_percentage | Phần trăm sử dụng CPU của pod | >80% giới hạn | >90% giới hạn | Tối ưu code; Tăng CPU limits; Horizontal scaling; Check hotspots |
| pod_memory_usage_percentage | Phần trăm sử dụng bộ nhớ của pod | >80% giới hạn | >90% giới hạn | Memory profiling; Tăng limits; Kiểm tra leaks; Tối ưu object usage |
| pod_network_receive_bytes | Số bytes mạng được nhận bởi pod | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Tìm traffic pattern bất thường; Kiểm tra protocols; Xem xét throttling |
| pod_network_transmit_bytes | Số bytes mạng được gửi bởi pod | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Kiểm tra outbound traffic; Tìm DDoS; Monitoring APIs; Optimize payload |
| **API Server** |||||
| apiserver_request_total | Tổng số yêu cầu tới API server | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Tìm client gây nhiều requests; Rate limit clients; Decrease poll frequency |
| apiserver_request_duration_seconds | Thời gian xử lý yêu cầu API | p99 >1s | p99 >5s | Scale API servers; Tối ưu etcd; Check API server resources; Tối ưu admission controllers |
| apiserver_request_error_rate | Tỷ lệ lỗi yêu cầu API | >1% | >5% | Kiểm tra validation; Xem error logs; Tìm bugs; Xác định các request patterns |
| **Deployments** |||||
| deployment_status_replicas | Số lượng replica của deployment | <replicas mong muốn | <50% replicas mong muốn | Kiểm tra pod logs; Xem events; Check resource constraints; Validating webhooks |
| deployment_status_replicas_available | Số lượng replica khả dụng | <replicas mong muốn | <50% replicas mong muốn | Kiểm tra readiness probes; Check pod health; Kiểm tra dependencies |
| deployment_spec_replicas | Số lượng replica mong muốn | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Xác định ai thay đổi HPA; Check automation; Xem controller logs |
| **StatefulSets** |||||
| statefulset_status_replicas | Số lượng replica của StatefulSet | <replicas mong muốn | <50% replicas mong muốn | Kiểm tra pod events; Xem storage issues; Check headless service; Verify PVCs |
| statefulset_status_replicas_ready | Số lượng replica sẵn sàng | <replicas mong muốn | <50% replicas mong muốn | Kiểm tra readiness; Check init containers; Verify volume mounts |
| **DaemonSets** |||||
| daemonset_status_desired_nodes | Số node DaemonSet cần chạy | N/A | N/A | Kiểm tra node selector; Verify taints/tolerations; Kiểm tra scheduler |
| daemonset_status_current_nodes | Số node DaemonSet đang chạy | <nodes mong muốn | <90% nodes mong muốn | Kiểm tra pod status; Xem node affinity; Check taints; Verify resources |
| **Jobs/CronJobs** |||||
| job_status_succeeded | Trạng thái hoàn thành của Job | Thất bại | Liên tục thất bại | Xem logs của pod; Tăng backoff limit; Check command args; Verify logic |
| job_status_duration_seconds | Thời gian chạy của Job | >thời gian timeout | >2*thời gian timeout | Tối ưu job performance; Kiểm tra resources; Verify external dependencies |
| cronjob_status_last_schedule_time | Thời gian kể từ lần lịch trình CronJob cuối | >1.5*khoảng thời gian lịch trình | >2*khoảng thời gian lịch trình | Kiểm tra cron expression; Verify CronJob controller; Check history limit |
| **HPA (Horizontal Pod Autoscaler)** |||||
| hpa_status_current_replicas | Số lượng replica pod hiện tại | N/A | N/A | Giám sát liên tục; Điều chỉnh min/max replicas |
| hpa_status_desired_replicas | Số lượng replica pod mong muốn | N/A | N/A | Kiểm tra metric being used; Check scaling algorithm |
| hpa_spec_min_replicas | Số lượng replica tối thiểu | N/A | N/A | Điều chỉnh dựa trên min load; Verify baseline performance |
| hpa_spec_max_replicas | Số lượng replica tối đa | hiện tại = tối đa trong >30 phút | hiện tại = tối đa trong >2 giờ | Tăng max replicas; Tối ưu pod resource usage; Scale other components |
| **etcd** |||||
| etcd_server_has_leader | Liệu server etcd có leader hay không | False | False trong >1 phút | Kiểm tra etcd logs; Verify quorum; Check network stability; Restart etcd |
| etcd_server_leader_changes_total | Số lần thay đổi leader | >3 trong 1 giờ | >10 trong 1 giờ | Kiểm tra network stability; Check disk latency; Verify clock sync; Isolate nodes |
| etcd_mvcc_db_size_in_bytes | Kích thước cơ sở dữ liệu etcd | >80% quota | >90% quota | Tăng quota-backend-bytes; Compact compaction; Defrag; Clean leases |
| **Volume Status** |||||
| volume_operation_total_seconds | Thời gian thực hiện các thao tác với volume | >30s | >300s | Kiểm tra storage provider; Check CSI driver; Verify volume capabilities |
| persistentvolume_status_phase | Trạng thái của PersistentVolume | Không phải Bound | Không phải Bound trong >30 phút | Kiểm tra storage class; Verify provisioner; Check reclaim policy |
| persistentvolumeclaim_status_phase | Trạng thái của PersistentVolumeClaim | Không phải Bound | Không phải Bound trong >30 phút | Kiểm tra storage class; Check capacity; Verify access modes; Check events |

## Web Servers & Proxies

### Nginx

| Metric | Description | Warning Threshold | Critical Threshold | Troubleshooting |
|--------|-------------|-------------------|---------------------|----------------|
| **Connection Status** |||||
| nginx_connections_active | Số lượng kết nối đang hoạt động | >70% worker_connections | >90% worker_connections | Tăng worker_connections; Tăng worker_processes; Theo dõi keep-alive; Check client traffic |
| nginx_connections_reading | Số kết nối Nginx đang đọc yêu cầu | >30% active | >50% active | Tăng client_header_timeout; Kiểm tra slow clients; Check network issues |
| nginx_connections_writing | Số kết nối Nginx đang ghi phản hồi | >30% active | >50% active | Tăng send_timeout; Kiểm tra upstream trả về chậm; Check network throttling |
| nginx_connections_waiting | Số kết nối rảnh đang chờ yêu cầu | >70% active | >90% active | Giảm keepalive_timeout; Tăng worker_connections; Check keep-alive clients |
| **Request Processing** |||||
| nginx_http_requests_total | Tổng số yêu cầu HTTP | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Tìm traffic pattern bất thường; Check for DDoS; Monitor client behavior |
| nginx_http_requests_per_second | Số yêu cầu HTTP mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Tăng capacity; Enable caching; Rate limiting; Check for attacks |
| **Performance** |||||
| nginx_request_time | Thời gian xử lý yêu cầu | p95 >500ms | p99 >2s | Check upstream performance; Tối ưu location blocks; Enable caching; Verify configs |
| nginx_upstream_response_time | Thời gian phản hồi của upstream | p95 >500ms | p99 >2s | Kiểm tra backend perf; Tối ưu application; Sửa lỗi backend; Tăng timeout |
| **Error Rates** |||||
| nginx_http_4xx_rate | Tỷ lệ phản hồi 4xx | >5% | >10% | Kiểm tra client requests; Check URL patterns; Verify authentication; Review access logs |
| nginx_http_5xx_rate | Tỷ lệ phản hồi 5xx | >1% | >5% | Kiểm tra upstream failures; Review error logs; Check application; Verify connectivity |
| nginx_upstream_next | Số yêu cầu chuyển sang server tiếp theo | >0 | >100/phút | Kiểm tra fail server đầu; Verify health checks; Check backend availability |
| nginx_upstream_fails | Số yêu cầu thất bại tới upstream server | >0 | >100/phút | Kiểm tra upstream health; Verify connectivity; Check timeouts; Review error logs |
| **SSL** |||||
| nginx_ssl_handshake_failures | Số lần thất bại trong bắt tay SSL | >0 | >100/phút | Kiểm tra SSL config; Verify certificate; Check client SSL support; Review logs |
| nginx_ssl_session_reuses | Số lần tái sử dụng phiên SSL | <50% | <30% | Tăng ssl_session_cache; Check session tickets; Verify client behavior |
| **Cache Performance** |||||
| nginx_cache_hit_ratio | Tỷ lệ cache hit | <70% | <50% | Tối ưu cache keys; Adjust expires headers; Increase cache size; Check vary headers |
| nginx_cache_size | Kích thước cache | >80% max_size | >90% max_size | Tăng max_size; Điều chỉnh inactive timeout; Tối ưu storage |
| **Resource Usage** |||||
| nginx_worker_processes | Số lượng tiến trình worker | ≠ số core CPU | N/A | Điều chỉnh worker_processes; Set auto; Check CPU usage |
| nginx_worker_connections | Số kết nối tối đa mỗi worker | >80% | >90% | Tăng worker_connections; Check ulimit -n; Increase system limits |

### Ingress Nginx

| Metric | Description | Warning Threshold | Critical Threshold | Troubleshooting |
|--------|-------------|-------------------|---------------------|----------------|
| **Controller Status** |||||
| nginx_ingress_controller_config_last_reload_successful | Lần tải lại cấu hình cuối cùng thành công | False | False trong >5 phút | Kiểm tra controller logs; Verify config syntax; Check CRDs; Review changes |
| nginx_ingress_controller_success | Tỷ lệ thành công của ingress controller | <100% | <95% | Xem controller logs; Check resource limits; Verify configuration |
| **Request Handling** |||||
| nginx_ingress_controller_requests | Tổng số yêu cầu từ client | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Kiểm tra traffic patterns; Check client behavior; Verify rate limiting |
| nginx_ingress_controller_request_duration_seconds | Thời gian xử lý yêu cầu | p95 >1s | p99 >5s | Check backend response time; Verify timeout settings; Optimize backend |
| nginx_ingress_controller_request_size | Kích thước yêu cầu | Tăng đột ngột >30% | Tăng đột ngột >50% | Kiểm tra client payload; Check for attacks; Verify upstream limits |
| nginx_ingress_controller_response_size | Kích thước phản hồi | Tăng đột ngột >30% | Tăng đột ngột >50% | Kiểm tra backend responses; Check compression; Verify content types |
| **Error Rates** |||||
| nginx_ingress_controller_status_4xx | Tỷ lệ phản hồi 4xx | >5% | >10% | Kiểm tra client requests; Verify routes; Check authentication; Review logs |
| nginx_ingress_controller_status_5xx | Tỷ lệ phản hồi 5xx | >1% | >5% | Kiểm tra backend failures; Verify service health; Check connectivity |
| **SSL** |||||
| nginx_ingress_controller_ssl_expire_time_seconds | Thời gian còn lại trước khi chứng chỉ SSL hết hạn | <30 ngày | <7 ngày | Cập nhật certificates; Setup auto-renewal; Check cert-manager |
| nginx_ingress_controller_ssl_handshake_errors_count | Số lỗi bắt tay SSL | >0 | >100/phút | Kiểm tra client TLS version; Verify cipher suites; Check certificate chain |
| **Upstream Services** |||||
| nginx_ingress_controller_upstream_latency_seconds | Độ trễ của dịch vụ upstream | p95 >1s | p99 >5s | Kiểm tra backend performance; Trace requests; Optimize services; Check connectivity |
| nginx_ingress_controller_upstream_status_4xx | Tỷ lệ 4xx từ upstream | >5% | >10% | Kiểm tra backend validation; Check routes; Verify request format |
| nginx_ingress_controller_upstream_status_5xx | Tỷ lệ 5xx từ upstream | >1% | >5% | Kiểm tra backend errors; Check service health; Verify resources; Review logs |
| **Socket Status** |||||
| nginx_ingress_controller_socket_queue_usage_pct | Tỷ lệ sử dụng hàng đợi socket | >70% | >90% | Tăng queue size; Optimize network settings; Check client connections |
| nginx_ingress_controller_worker_connections_usage_pct | Tỷ lệ sử dụng kết nối worker | >70% | >90% | Tăng max-worker-connections; Scale controller; Optimize concurrency |
| **Resource Usage** |||||
| nginx_ingress_controller_cpu_usage | Sử dụng CPU | >80% giới hạn | >90% giới hạn | Tăng CPU limits; Scale controller replicas; Optimize configuration |
| nginx_ingress_controller_memory_usage | Sử dụng bộ nhớ | >80% giới hạn | >90% giới hạn | Tăng memory limits; Check leaks; Optimize cache settings; Scale replicas |

## Service Mesh & Network

### Istio

| Metric | Description | Warning Threshold | Critical Threshold | Troubleshooting |
|--------|-------------|-------------------|---------------------|----------------|
| **Request Handling** |||||
| istio_requests_total | Tổng số yêu cầu | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Kiểm tra traffic patterns; Verify client behavior; Check service scaling |
| istio_request_duration_milliseconds | Thời gian xử lý yêu cầu | p95 >500ms | p99 >2s | Phân tích service latency; Check envoy configs; Optimize application; Review network |
| istio_request_bytes | Kích thước yêu cầu | Tăng đột ngột >30% | Tăng đột ngột >50% | Kiểm tra request payload; Validate client behavior; Monitor abnormal patterns |
| istio_response_bytes | Kích thước phản hồi | Tăng đột ngột >30% | Tăng đột ngột >50% | Kiểm tra response size; Verify data retrieval; Check compression settings |
| **Error Rates** |||||
| istio_requests_total (4xx) | Tỷ lệ lỗi 4xx | >5% | >10% | Kiểm tra client requests; Review routing rules; Check authentication; Validate payload |
| istio_requests_total (5xx) | Tỷ lệ lỗi 5xx | >1% | >5% | Kiểm tra service failures; Check timeouts; Verify dependencies; Review logs |
| istio_request_duration_milliseconds (timeout) | Số lần yêu cầu bị timeout | >0.1% | >1% | Tăng timeout settings; Verify service performance; Check resource constraints |
| **Circuit Breaking** |||||
| istio_circuit_breakers_triggered | Số lần kích hoạt ngắt mạch | >0 | >10/phút | Kiểm tra service health; Verify resource limits; Check connection pool; Review traffic |
| istio_circuit_breakers_current_open | Số lượng ngắt mạch đang mở | >0 | >3 | Kiểm tra service performance; Fix failing services; Increase resources |
| **Connection Metrics** |||||
| istio_tcp_connections_opened_total | Số kết nối TCP được mở | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Kiểm tra connection patterns; Review service-to-service communication |
| istio_tcp_connections_closed_total | Số kết nối TCP được đóng | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Kiểm tra connection lifecycle; Verify service stability; Check networking |
| istio_tcp_received_bytes_total | Số bytes TCP nhận được | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Kiểm tra data transfer patterns; Verify payload size; Check protocols |
| istio_tcp_sent_bytes_total | Số bytes TCP gửi đi | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% | Kiểm tra outbound traffic; Verify response size; Monitor bandwidth usage |
| **mTLS** |||||
| istio_mtls_request_percent | Phần trăm yêu cầu mTLS | <90% (khi yêu cầu mTLS) | <50% (khi yêu cầu mTLS) | Kiểm tra PeerAuthentication; Verify certificates; Check policy; Review logs |
| **Control Plane** |||||
| pilot_conflict_inbound_listener | Xung đột listener inbound | >0 | >10 | Kiểm tra service port conflicts; Verify sidecars; Check mesh config |
| pilot_conflict_outbound_listener_http | Xung đột listener outbound HTTP | >0 | >10 | Kiểm tra service entries; Verify virtual services; Check port conflicts |
| pilot_conflict_outbound_listener_tcp | Xung đột listener outbound TCP | >0 | >10 | Kiểm tra TCP service entries; Verify destination rules; Check overlapping CIDR |
| pilot_services | Số lượng dịch vụ được theo dõi | Thay đổi đột ngột >10% | Thay đổi đột ngột >30% | Kiểm tra service registration; Verify service discovery; Monitor Kubernetes API |
| pilot_xds_push_context_errors | Lỗi ngữ cảnh push XDS | >0 | >10 | Kiểm tra Pilot logs; Verify configuration; Check resource versions; Monitor API conflicts |
| **Sidecars** |||||
| sidecar_injection_success_total | Số lần tiêm sidecar thành công | <100% | <90% | Kiểm tra webhook configuration; Verify namespace labels; Check pod specs; Review logs |
| sidecar_injection_failure_total | Số lần tiêm sidecar thất bại | >0 | >10 | Kiểm tra webhook errors; Verify permissions; Check resource limits; Review pod specs |
| **Proxy Status** |||||
| pilot_xds_cds_reject | Cấu hình CDS bị từ chối | >0 | >10 | Kiểm tra cluster configurations; Verify endpoint definitions; Review pilot logs |
| pilot_xds_eds_reject | Cấu hình EDS bị từ chối | >0 | >10 | Kiểm tra endpoint configurations; Verify service health; Check network policies |
| pilot_xds_rds_reject | Cấu hình RDS bị từ chối | >0 | >10 | Kiểm tra route configurations; Verify VirtualServices; Check path conflicts |
| pilot_xds_lds_reject | Cấu hình LDS bị từ chối | >0 | >10 | Kiểm tra listener configurations; Verify port conflicts; Check gateway settings |
| pilot_proxy_convergence_time | Thời gian hội tụ cấu hình proxy | >5s | >30s | Tối ưu istiod resources; Check proxy resources; Verify configuration complexity |
| **Gateway** |||||
| pilot_k8s_cfg_events | Sự kiện cấu hình Kubernetes | Đột biến | Đột biến liên tục | Kiểm tra configuration churn; Verify CI/CD processes; Monitor deployment frequency |
| pilot_virt_services | Số lượng dịch vụ ảo | Thay đổi đột ngột >10% | Thay đổi đột ngột >30% | Kiểm tra VirtualService creation; Verify routing rules; Monitor configuration changes
