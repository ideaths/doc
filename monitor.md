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

| Metric | Description | Warning Threshold | Critical Threshold |
|--------|-------------|-------------------|---------------------|
| **Connection & Threads** |||
| Connections | Số lượng kết nối được thử thiết lập | >80% max_connections | >90% max_connections |
| Threads_connected | Số lượng kết nối đang mở | >80% max_connections | >90% max_connections |
| Threads_running | Số lượng thread đang chạy truy vấn | >30 | >50 |
| Aborted_connects | Số lượng kết nối thất bại | >0 | >10/phút |
| **Query Performance** |||
| Slow_queries | Số lượng truy vấn vượt quá long_query_time | >0 | >10/phút |
| Questions | Số lượng câu lệnh được thực thi | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| Com_* | Số lượng các câu lệnh cụ thể (select, insert, v.v.) | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| **Cache Efficiency** |||
| Innodb_buffer_pool_hit_ratio | Tỷ lệ % đọc từ bộ đệm so với từ đĩa | <95% | <90% |
| Innodb_buffer_pool_utilization | Tỷ lệ % sử dụng buffer pool | >95% | >98% |
| Table_open_cache_hits / misses | Hiệu quả cache cho table handlers | hits / (hits + misses) < 0.9 | hits / (hits + misses) < 0.8 |
| **Storage & I/O** |||
| Innodb_data_reads/writes | Số lượng thao tác đọc/ghi đĩa | Tăng đột ngột >30% | Tăng đột ngột >50% |
| Innodb_log_waits | Số lần chờ do buffer log quá nhỏ | >0 | >10 |
| Created_tmp_disk_tables | Số bảng tạm được tạo trên đĩa (không phải trong bộ nhớ) | >10/giây | >20/giây |
| **Locking & Contention** |||
| Innodb_row_lock_waits | Số lần phải chờ khóa hàng | >0 | >10/phút |
| Innodb_row_lock_time_avg | Thời gian trung bình để có được khóa hàng (ms) | >10ms | >50ms |
| Table_locks_waited | Số lần chờ khóa bảng | >0 | >10/phút |
| **Replication** |||
| Seconds_Behind_Master | Độ trễ sao chép theo giây | >30 giây | >300 giây |
| Slave_IO_Running | Trạng thái luồng IO | Khác "Yes" | Khác "Yes" trong >5 phút |
| Slave_SQL_Running | Trạng thái luồng SQL | Khác "Yes" | Khác "Yes" trong >5 phút |
| **Error Rates** |||
| Ongoing Deadlocks | Giá trị từ SHOW ENGINE INNODB STATUS | >0 | >5/giờ |
| Foreign key errors | Số lượng từ error log | >0 | >10 |
| **General Server Stats** |||
| Uptime | Thời gian hoạt động của server tính bằng giây | Khởi động lại không mong muốn | Nhiều lần khởi động lại |
| Available disk space | Không gian đĩa trống trên các volume cơ sở dữ liệu | <20% | <10% |

### PostgreSQL

| Metric | Description | Warning Threshold | Critical Threshold |
|--------|-------------|-------------------|---------------------|
| **Connection Management** |||
| pg_stat_activity_count | Số lượng kết nối đang hoạt động | >80% max_connections | >90% max_connections |
| idle_in_transaction_count | Số phiên trong trạng thái idle transaction | >5 | >20 |
| longest_transaction_duration | Thời gian của transaction dài nhất | >10 phút | >1 giờ |
| **Query Performance** |||
| pg_stat_statements_calls | Top queries theo số lần gọi | N/A | N/A |
| pg_stat_statements_total_time | Top queries theo thời gian thực thi | >1 giây | >10 giây |
| slow_queries_rate | Các truy vấn chạy lâu hơn ngưỡng | >5/phút | >20/phút |
| **Cache Efficiency** |||
| pg_stat_database_blks_hit_ratio | Tỷ lệ hit buffer cache | <0.98 | <0.95 |
| temp_files_size | Kích thước file tạm được tạo | >100MB/phút | >1GB/phút |
| temp_bytes_rate | Tốc độ tạo dữ liệu tạm | >10MB/giây | >100MB/giây |
| **Lock Contention** |||
| pg_locks_count | Số lượng khóa theo loại | Tăng >50% | Tăng >100% |
| pg_locks_waiting | Số phiên đang chờ khóa | >0 | >5 |
| lock_wait_time_max | Thời gian tối đa một truy vấn đã chờ khóa | >30 giây | >5 phút |
| **I/O Activity** |||
| pg_stat_database_tup_fetched/returned | Số hàng được lấy/trả về | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| pg_stat_database_tup_inserted/updated/deleted | Các thao tác ghi | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| checkpoint_write_time | Thời gian dành cho việc ghi checkpoint | >50% checkpoint_timeout | >80% checkpoint_timeout |
| **Replication** |||
| pg_stat_replication_lag_bytes | Độ trễ sao chép tính bằng bytes | >50MB | >500MB |
| pg_stat_replication_lag_time | Độ trễ sao chép tính bằng giây | >30 giây | >300 giây |
| max_replay_lag | Thời gian trễ dài nhất trong phát lại WAL | >60 giây | >300 giây |
| **VACUUM & Database Maintenance** |||
| pg_stat_user_tables_dead_tup | Số lượng tuples "chết" | >10000 | >100000 |
| pg_stat_user_tables_autovacuum_count | Số lần chạy autovacuum | >10/giờ/bảng | >30/giờ/bảng |
| pg_stat_database_age_frozen_xid | Tuổi của XID cũ nhất | >1,000,000,000 | >1,500,000,000 |
| **Table & Index Stats** |||
| table_bloat_percentage | Phần trăm không gian bảng bị phình to | >30% | >50% |
| index_bloat_percentage | Phần trăm không gian chỉ mục bị phình to | >30% | >50% |
| index_scans_vs_seq_scans_ratio | Tỷ lệ quét chỉ mục so với quét tuần tự | <0.7 | <0.5 |
| **Transaction IDs** |||
| transaction_wraparound | Thời gian cho đến khi ID giao dịch bị cuộn tròn | <1 tháng | <1 tuần |
| **Storage** |||
| pg_database_size_bytes | Kích thước cơ sở dữ liệu | >80% dung lượng đĩa | >90% dung lượng đĩa |
| wal_files_count | Số lượng tệp WAL | >50 | >100 |
| wal_bytes_rate | Tốc độ tạo WAL | Thay đổi đột ngột >50% | >100MB/giây |

### MongoDB

| Metric | Description | Warning Threshold | Critical Threshold |
|--------|-------------|-------------------|---------------------|
| **Connection & Resources** |||
| connections.current | Số lượng kết nối hiện tại | >80% connections.totalCreated | >90% connections.totalCreated |
| connections.available | Kết nối còn khả dụng | <100 | <20 |
| globalLock.currentQueue | Số thao tác đang chờ khóa | >10 | >20 |
| **Operation Performance** |||
| opcounters.query | Số thao tác truy vấn mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| opcounters.insert | Số thao tác chèn mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| opcounters.update | Số thao tác cập nhật mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| opcounters.delete | Số thao tác xóa mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| **Query Efficiency** |||
| metrics.queryExecutor.scanned | Số tài liệu được quét | >1000/truy vấn | >10000/truy vấn |
| metrics.queryExecutor.scannedObjects | Số đối tượng được quét | >1000/truy vấn | >10000/truy vấn |
| ratio_scanned_to_returned | Tỷ lệ tài liệu quét so với trả về | >100:1 | >1000:1 |
| **Memory Usage** |||
| mem.resident | Lượng bộ nhớ resident sử dụng | >80% bộ nhớ hệ thống | >90% bộ nhớ hệ thống |
| mem.virtual | Lượng bộ nhớ ảo sử dụng | >2x bộ nhớ resident | >4x bộ nhớ resident |
| wiredTiger.cache.maximum | Giới hạn cache WiredTiger | N/A | N/A |
| wiredTiger.cache.used | Lượng cache WiredTiger đã sử dụng | >80% wiredTiger.cache.maximum | >90% wiredTiger.cache.maximum |
| **Replication** |||
| repl.lag | Độ trễ sao chép | >10 giây | >60 giây |
| repl.oplog.window | Cửa sổ oplog tính bằng giờ | <24 giờ | <12 giờ |
| **Storage** |||
| metrics.storage.freelist.search.bucketExhausted | Chỉ số phân mảnh | >0 | >10/giờ |
| db.stats.storageSize | Tổng kích thước lưu trữ | >80% dung lượng đĩa | >90% dung lượng đĩa |
| db.stats.indexSize | Tổng kích thước chỉ mục | >5GB mỗi chỉ mục | >10GB mỗi chỉ mục |
| **Backend Operations** |||
| backgroundFlushing.average_ms | Thời gian flush trung bình | >100ms | >500ms |
| metrics.cursor.open.total | Số lượng con trỏ đang mở | >10000 | >50000 |
| metrics.cursor.timedOut | Số con trỏ hết thời gian | >0 | >10/giờ |
| **WiredTiger Specific** |||
| wiredTiger.concurrentTransactions.read.available | Số lượng vé đọc khả dụng | <10 | <5 |
| wiredTiger.concurrentTransactions.write.available | Số lượng vé ghi khả dụng | <10 | <5 |
| wiredTiger.cache.eviction-worker-running | Số luồng eviction worker đang chạy | >50% số lõi CPU | N/A |

## Message Queue & Streaming

### Kafka

| Metric | Description | Warning Threshold | Critical Threshold |
|--------|-------------|-------------------|---------------------|
| **Broker Metrics** |||
| UnderReplicatedPartitions | Số lượng partition có replica không đủ | >0 | >10 |
| OfflinePartitionsCount | Số lượng partition không có leader đang hoạt động | >0 | >1 |
| ActiveControllerCount | Số lượng controller đang hoạt động (nên là 1) | !=1 | N/A |
| RequestHandlerAvgIdlePercent | % thời gian rảnh trung bình của request handler | <30% | <10% |
| NetworkProcessorAvgIdlePercent | % thời gian rảnh trung bình của network processor | <30% | <10% |
| **Topic & Partition Metrics** |||
| MessagesInPerSec | Tốc độ tin nhắn đến | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| BytesInPerSec | Số bytes nhận vào mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| BytesOutPerSec | Số bytes gửi đi mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| LogSize | Kích thước log của partition | >80% dung lượng đĩa | >90% dung lượng đĩa |
| PartitionCount | Số lượng partition trên một broker | >4000 | >8000 |
| LeaderCount | Số lượng leader partition trên một broker | Mất cân bằng >20% giữa các broker | Mất cân bằng >40% giữa các broker |
| **Producer Metrics** |||
| request-latency-avg | Độ trễ truy vấn trung bình | >20ms | >100ms |
| request-rate | Số lượng yêu cầu mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| response-rate | Số lượng phản hồi mỗi giây | request-rate - response-rate > 0 | (request-rate - response-rate) / request-rate > 0.1 |
| record-error-rate | Tỷ lệ lỗi của record | >0 | >0.01 |
| **Consumer Metrics** |||
| records-lag-max | Độ trễ tối đa tính theo số lượng record | >1000 | >10000 |
| records-consumed-rate | Tốc độ tiêu thụ record | Giảm >30% | Giảm >50% |
| bytes-consumed-rate | Số bytes tiêu thụ mỗi giây | Giảm >30% | Giảm >50% |
| fetch-rate | Số lượng yêu cầu fetch mỗi giây | Giảm >30% | Giảm >50% |
| **JVM Metrics** |||
| heap-usage-bytes | Lượng heap JVM sử dụng | >80% max heap | >90% max heap |
| gc-collection-time | Thời gian thu gom rác | >500ms | >1s |
| **ZooKeeper Metrics** |||
| ZooKeeperRequestLatencyMs | Độ trễ yêu cầu ZooKeeper | >10ms | >100ms |
| ZooKeeperNumAliveConnections | Số lượng kết nối đang hoạt động tới ZooKeeper | <số lượng broker | <(số lượng broker - 2) |
| OutstandingRequests | Số lượng yêu cầu đang chờ xử lý | >10 | >20 |

## Search & Analytics

### Elasticsearch

| Metric | Description | Warning Threshold | Critical Threshold |
|--------|-------------|-------------------|---------------------|
| **Cluster Health** |||
| cluster_health.status | Trạng thái tổng thể của cluster | Yellow | Red |
| cluster_health.initializing_shards | Số lượng shard đang khởi tạo | >0 trong >10 phút | >0 trong >30 phút |
| cluster_health.relocating_shards | Số lượng shard đang di chuyển | >0 trong >10 phút | >0 trong >30 phút |
| cluster_health.unassigned_shards | Số lượng shard chưa được gán | >0 | >10 |
| **Node Stats** |||
| jvm.mem.heap_used_percent | Phần trăm heap JVM đã sử dụng | >80% | >90% |
| jvm.gc.collectors.old.collection_time | Thời gian thu gom rác thế hệ cũ | >500ms | >2s |
| jvm.gc.collectors.old.collection_count | Số lần thu gom rác thế hệ cũ | >10/giờ | >30/giờ |
| process.cpu.percent | Phần trăm CPU sử dụng | >80% | >90% |
| **Indexing Performance** |||
| indices.indexing.index_total | Tổng số tài liệu đã được đánh chỉ mục | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| indices.indexing.index_time_in_millis | Tổng thời gian dành cho việc đánh chỉ mục | >50ms trung bình mỗi tài liệu | >200ms trung bình mỗi tài liệu |
| indices.indexing.index_current | Số lượng tài liệu đang được đánh chỉ mục | >50 | >200 |
| indices.indexing.delete_total | Tổng số tài liệu đã bị xóa | Tăng đột ngột | N/A |
| **Search Performance** |||
| indices.search.query_total | Tổng số truy vấn | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| indices.search.query_time_in_millis | Tổng thời gian dùng cho truy vấn | >100ms trung bình mỗi truy vấn | >500ms trung bình mỗi truy vấn |
| indices.search.fetch_total | Tổng số thao tác fetch | N/A | N/A |
| indices.search.fetch_time_in_millis | Tổng thời gian dùng cho fetch | >50ms trung bình mỗi fetch | >200ms trung bình mỗi fetch |
| indices.search.scroll_total | Tổng số thao tác scroll | N/A | N/A |
| indices.search.scroll_time_in_millis | Thời gian dành cho thao tác scroll | >500ms trung bình mỗi scroll | >2s trung bình mỗi scroll |
| **Memory Usage & Cache** |||
| indices.fielddata.memory_size_in_bytes | Kích thước bộ đệm field data | >30% heap | >40% heap |
| indices.fielddata.evictions | Số lần đẩy field data ra khỏi bộ đệm | >0 | >100 |
| indices.query_cache.memory_size_in_bytes | Kích thước bộ đệm truy vấn | >10% heap | >20% heap |
| indices.query_cache.evictions | Số lần đẩy truy vấn ra khỏi bộ đệm | >1000/phút | >5000/phút |
| indices.request_cache.memory_size_in_bytes | Kích thước bộ đệm yêu cầu | >5% heap | >10% heap |
| **Disk Usage** |||
| fs.total.available_in_bytes | Dung lượng đĩa còn trống | <20% | <10% |
| indices.store.size_in_bytes | Tổng kích thước của các chỉ mục | >80% dung lượng đĩa | >90% dung lượng đĩa |
| **Circuit Breakers** |||
| breakers.*.tripped | Số lần ngắt mạch | >0 | >5 |
| **Thread Pool** |||
| thread_pool.*.rejected | Số lượng tác vụ bị từ chối trong thread pool | >0 | >10 |
| thread_pool.*.queue | Kích thước hàng đợi của thread pool | >80% của tối đa | >90% của tối đa |
| **Transport & HTTP** |||
| transport.rx_size_in_bytes | Kích thước lưu lượng transport nhận được | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| transport.tx_size_in_bytes | Kích thước lưu lượng transport gửi đi | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| http.current_open | Số lượng kết nối HTTP đang mở | >80% tối đa | >90% tối đa |

## Caching

### Redis

| Metric | Description | Warning Threshold | Critical Threshold |
|--------|-------------|-------------------|---------------------|
| **Server Health** |||
| uptime_in_seconds | Thời gian hoạt động của server | Khởi động lại không mong muốn | Nhiều lần khởi động lại |
| connected_clients | Số lượng client đang kết nối | >80% maxclients | >90% maxclients |
| blocked_clients | Số lượng client đang chờ trên thao tác blocking | >5 | >20 |
| rejected_connections | Số kết nối bị từ chối do vượt quá maxclients | >0 | >100 |
| **Memory Usage** |||
| used_memory | Tổng lượng bộ nhớ Redis sử dụng | >80% maxmemory | >90% maxmemory |
| used_memory_rss | Tổng lượng bộ nhớ Redis sử dụng theo góc nhìn của OS | >150% used_memory | >200% used_memory |
| mem_fragmentation_ratio | Tỷ lệ giữa RSS / bộ nhớ sử dụng | <0.8 hoặc >1.5 | <0.7 hoặc >2.0 |
| evicted_keys | Số lượng key bị xóa do giới hạn maxmemory | >0 | >1000/giờ |
| expired_keys | Số lượng key đã hết hạn | Tăng đột ngột | N/A |
| **Command Execution** |||
| instantaneous_ops_per_sec | Số lượng lệnh thực thi mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| keyspace_hits | Số lần tìm kiếm key thành công | N/A | N/A |
| keyspace_misses | Số lần tìm kiếm key thất bại | keyspace_misses/keyspace_hits > 0.5 | keyspace_misses/keyspace_hits > 1 |
| **Persistence** |||
| rdb_last_bgsave_status | Trạng thái của thao tác RDB save gần nhất | "error" | "error" cho nhiều lần liên tiếp |
| rdb_changes_since_last_save | Số thay đổi kể từ lần lưu trữ cuối cùng | >10,000 | >100,000 |
| aof_last_write_status | Trạng thái của thao tác ghi AOF gần nhất | "error" | "error" cho nhiều lần liên tiếp |
| aof_last_rewrite_status | Trạng thái của thao tác viết lại AOF gần nhất | "error" | "error" cho nhiều lần liên tiếp |
| **Replication** |||
| connected_slaves | Số lượng replica đang kết nối | <số lượng dự kiến | 0 (khi mong đợi có replica) |
| master_link_status | Trạng thái kết nối với master | "down" | "down" trong >1 phút |
| master_last_io_seconds_ago | Thời gian kể từ lần tương tác cuối cùng với master | >30 | >60 |
| **Keyspace** |||
| db{X}.keys | Số lượng key trong cơ sở dữ liệu X | Giảm đột ngột >10% | Giảm đột ngột >30% |
| db{X}.expires | Số lượng key có thời hạn trong cơ sở dữ liệu X | N/A | N/A |
| **Latency** |||
| latency_percentiles_usec | Phân vị độ trễ tính bằng microsecond | p99 > 1000 | p99 > 10000 |
| **Commands** |||
| cmdstat_* | Thống kê về các lệnh cụ thể | N/A | N/A |
| **Sentinel Specific** |||
| sentinel_masters | Số lượng master được giám sát bởi Sentinel | ≠ số lượng dự kiến | N/A |
| sentinel_tilt | Cờ cho chế độ tilt của Sentinel | >0 | N/A |
| sentinel_running_scripts | Số lượng script đang thực thi | >0 | >3 |

## Infrastructure

### Operating System

| Metric | Description | Warning Threshold | Critical Threshold |
|--------|-------------|-------------------|---------------------|
| **CPU Usage** |||
| cpu_usage_user | Thời gian CPU dành cho không gian người dùng | >70% | >90% |
| cpu_usage_system | Thời gian CPU dành cho kernel | >40% | >70% |
| cpu_usage_iowait | Thời gian CPU chờ I/O | >20% | >40% |
| cpu_usage_steal | Thời gian CPU bị ảo hóa chiếm dụng | >5% | >20% |
| load_average | Tải trung bình của hệ thống | >số lõi CPU | >2*số lõi CPU |
| **Memory Usage** |||
| memory_used_percent | Phần trăm bộ nhớ đã sử dụng | >85% | >95% |
| memory_available_bytes | Bộ nhớ khả dụng | <10% tổng bộ nhớ | <5% tổng bộ nhớ |
| swap_used_percent | Phần trăm swap đã sử dụng | >50% | >80% |
| swap_in_rate | Tốc độ swap vào từ đĩa | >10MB/s | >50MB/s |
| swap_out_rate | Tốc độ swap ra đĩa | >10MB/s | >50MB/s |
| **Disk Usage** |||
| disk_used_percent | Phần trăm dung lượng đĩa đã sử dụng | >80% | >90% |
| disk_inodes_used_percent | Phần trăm inode đã sử dụng | >80% | >90% |
| **Disk I/O** |||
| disk_io_reads | Số thao tác đọc đĩa mỗi giây | Liên tục >1000 IOPS | Liên tục >5000 IOPS |
| disk_io_writes | Số thao tác ghi đĩa mỗi giây | Liên tục >1000 IOPS | Liên tục >5000 IOPS |
| disk_io_read_bytes | Thông lượng đọc đĩa | >100MB/s | >500MB/s |
| disk_io_write_bytes | Thông lượng ghi đĩa | >100MB/s | >500MB/s |
| disk_io_await | Thời gian trung bình cho các yêu cầu I/O | >20ms | >100ms |
| disk_io_util | Phần trăm thời gian CPU dành cho các yêu cầu I/O | >80% | >95% |
| **Network** |||
| network_in_bytes | Lưu lượng mạng vào | >70% băng thông interface | >90% băng thông interface |
| network_out_bytes | Lưu lượng mạng ra | >70% băng thông interface | >90% băng thông interface |
| network_in_errors | Lỗi gói tin vào | >0 | >100/phút |
| network_out_errors | Lỗi gói tin ra | >0 | >100/phút |
| network_in_dropped | Gói tin vào bị drop | >0 | >100/phút |
| network_out_dropped | Gói tin ra bị drop | >0 | >100/phút |
| **File System** |||
| open_file_descriptors | Số lượng file descriptor đang mở | >80% tối đa | >90% tối đa |
| **Process** |||
| process_count | Tổng số tiến trình | >500 | >1000 |
| zombie_process_count | Số lượng tiến trình zombie | >5 | >20 |
| **System** |||
| uptime | Thời gian hoạt động của hệ thống | Khởi động lại không mong muốn | Nhiều lần khởi động lại |
| context_switches | Số lần chuyển đổi ngữ cảnh mỗi giây | Tăng liên tục >30% | Tăng liên tục >50% |
| interrupts | Số lượng ngắt mỗi giây | Tăng liên tục >30% | Tăng liên tục >50% |

## Application Performance

### Application Metrics

| Metric | Description | Warning Threshold | Critical Threshold |
|--------|-------------|-------------------|---------------------|
| **Request Handling** |||
| request_rate | Số lượng yêu cầu mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| request_duration_seconds | Thời gian xử lý yêu cầu | p95 > SLO ứng dụng | p99 > SLO ứng dụng |
| request_errors_total | Tổng số lỗi yêu cầu | Tỷ lệ lỗi >1% | Tỷ lệ lỗi >5% |
| request_timeout_total | Tổng số yêu cầu bị timeout | Tỷ lệ timeout >0.1% | Tỷ lệ timeout >1% |
| **Resource Usage** |||
| app_cpu_usage | Mức sử dụng CPU của ứng dụng | >70% đã cấp phát | >90% đã cấp phát |
| app_memory_usage | Mức sử dụng bộ nhớ của ứng dụng | >80% đã cấp phát | >90% đã cấp phát |
| app_thread_count | Số lượng thread | >80% tối đa đã cấu hình | >90% tối đa đã cấu hình |
| **Connection Pools** |||
| db_connections_active | Số kết nối cơ sở dữ liệu đang hoạt động | >80% kích thước pool | >90% kích thước pool |
| db_connections_idle | Số kết nối cơ sở dữ liệu đang rảnh | <10% kích thước pool | <5% kích thước pool |
| db_connection_wait_time | Thời gian chờ kết nối cơ sở dữ liệu | >10ms | >100ms |
| **Garbage Collection** |||
| gc_collection_count | Số lượng garbage collection | >10/phút | >30/phút |
| gc_collection_time | Thời gian dành cho garbage collection | >10% thời gian CPU | >20% thời gian CPU |
| **Business Metrics** |||
| transaction_rate | Số giao dịch nghiệp vụ mỗi giây | Giảm đột ngột >20% | Giảm đột ngột >50% |
| transaction_error_rate | Số giao dịch nghiệp vụ thất bại | >1% | >5% |
| user_login_success_rate | Tỷ lệ đăng nhập thành công | <95% | <90% |
| checkout_success_rate | Tỷ lệ thanh toán thành công | <95% | <90% |
| **Caching** |||
| cache_hit_ratio | Tỷ lệ cache hit | <80% | <60% |
| cache_miss_rate | Số lần cache miss mỗi giây | Tăng đột ngột >30% | Tăng đột ngột >50% |
| **External Dependencies** |||
| external_service_response_time | Thời gian phản hồi của dịch vụ bên ngoài | >500ms | >2s |
| external_service_error_rate | Tỷ lệ lỗi khi gọi dịch vụ bên ngoài | >1% | >5% |
| **Queue/Async Processing** |||
| queue_size | Số lượng mục trong hàng đợi | >1000 | >10000 |
| queue_latency | Thời gian tin nhắn nằm trong hàng đợi | >30s | >5phút |
| queue_consumer_lag | Độ trễ của consumer (tin nhắn) | >1000 | >10000 |
| **Circuit Breakers** |||
| circuit_breaker_open_count | Số lượng circuit breaker đang mở | >0 | >3 |
| circuit_breaker_half_open_count | Số lượng circuit breaker đang nửa mở | >3 | >5 |
| **Custom Business KPIs** |||
| active_users | Số lượng người dùng đang hoạt động | Giảm đột ngột >20% | Giảm đột ngột >50% |
| conversion_rate | Tỷ lệ chuyển đổi người dùng | Giảm đột ngột >10% | Giảm đột ngột >30% |
| average_order_value | Giá trị trung bình của đơn hàng | Giảm đột ngột >10% | Giảm đột ngột >30% |

## Container Orchestration

### Kubernetes

| Metric | Description | Warning Threshold | Critical Threshold |
|--------|-------------|-------------------|---------------------|
| **Node Status** |||
| node_status_condition | Điều kiện của node (Ready, DiskPressure, v.v.) | Not Ready | Not Ready trong >5 phút |
| node_cpu_usage_percentage | Phần trăm sử dụng CPU | >80% | >90% |
| node_memory_usage_percentage | Phần trăm sử dụng bộ nhớ | >80% | >90% |
| node_disk_usage_percentage | Phần trăm sử dụng đĩa | >80% | >90% |
| node_pod_capacity_usage | Phần trăm capacity pod đã sử dụng | >80% | >90% |
| **Pod Status** |||
| pod_status_phase | Trạng thái hiện tại của pod | Không phải Running | Không phải Running trong >5 phút |
| pod_status_ready | Trạng thái sẵn sàng của pod | Không Ready | Không Ready trong >5 phút |
| pod_container_status_waiting | Trạng thái chờ của container | True với CrashLoopBackOff | True với CrashLoopBackOff trong >5 phút |
| pod_container_status_terminated | Trạng thái kết thúc của container | Liên tục bị kết thúc | Liên tục bị kết thúc với lỗi |
| pod_restarts_total | Tổng số lần khởi động lại pod | >5 trong 1 giờ | >10 trong 1 giờ |
| **Resource Usage** |||
| pod_cpu_usage_percentage | Phần trăm sử dụng CPU của pod | >80% giới hạn | >90% giới hạn |
| pod_memory_usage_percentage | Phần trăm sử dụng bộ nhớ của pod | >80% giới hạn | >90% giới hạn |
| pod_network_receive_bytes | Số bytes mạng được nhận bởi pod | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| pod_network_transmit_bytes | Số bytes mạng được gửi bởi pod | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| **API Server** |||
| apiserver_request_total | Tổng số yêu cầu tới API server | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| apiserver_request_duration_seconds | Thời gian xử lý yêu cầu API | p99 >1s | p99 >5s |
| apiserver_request_error_rate | Tỷ lệ lỗi yêu cầu API | >1% | >5% |
| **Deployments** |||
| deployment_status_replicas | Số lượng replica của deployment | <replicas mong muốn | <50% replicas mong muốn |
| deployment_status_replicas_available | Số lượng replica khả dụng | <replicas mong muốn | <50% replicas mong muốn |
| deployment_spec_replicas | Số lượng replica mong muốn | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| **StatefulSets** |||
| statefulset_status_replicas | Số lượng replica của StatefulSet | <replicas mong muốn | <50% replicas mong muốn |
| statefulset_status_replicas_ready | Số lượng replica sẵn sàng | <replicas mong muốn | <50% replicas mong muốn |
| **DaemonSets** |||
| daemonset_status_desired_nodes | Số node DaemonSet cần chạy | N/A | N/A |
| daemonset_status_current_nodes | Số node DaemonSet đang chạy | <nodes mong muốn | <90% nodes mong muốn |
| **Jobs/CronJobs** |||
| job_status_succeeded | Trạng thái hoàn thành của Job | Thất bại | Liên tục thất bại |
| job_status_duration_seconds | Thời gian chạy của Job | >thời gian timeout | >2*thời gian timeout |
| cronjob_status_last_schedule_time | Thời gian kể từ lần lịch trình CronJob cuối | >1.5*khoảng thời gian lịch trình | >2*khoảng thời gian lịch trình |
| **HPA (Horizontal Pod Autoscaler)** |||
| hpa_status_current_replicas | Số lượng replica pod hiện tại | N/A | N/A |
| hpa_status_desired_replicas | Số lượng replica pod mong muốn | N/A | N/A |
| hpa_spec_min_replicas | Số lượng replica tối thiểu | N/A | N/A |
| hpa_spec_max_replicas | Số lượng replica tối đa | hiện tại = tối đa trong >30 phút | hiện tại = tối đa trong >2 giờ |
| **etcd** |||
| etcd_server_has_leader | Liệu server etcd có leader hay không | False | False trong >1 phút |
| etcd_server_leader_changes_total | Số lần thay đổi leader | >3 trong 1 giờ | >10 trong 1 giờ |
| etcd_mvcc_db_size_in_bytes | Kích thước cơ sở dữ liệu etcd | >80% quota | >90% quota |
| **Volume Status** |||
| volume_operation_total_seconds | Thời gian thực hiện các thao tác với volume | >30s | >300s |
| persistentvolume_status_phase | Trạng thái của PersistentVolume | Không phải Bound | Không phải Bound trong >30 phút |
| persistentvolumeclaim_status_phase | Trạng thái của PersistentVolumeClaim | Không phải Bound | Không phải Bound trong >30 phút |

## Web Servers & Proxies

### Nginx

| Metric | Description | Warning Threshold | Critical Threshold |
|--------|-------------|-------------------|---------------------|
| **Connection Status** |||
| nginx_connections_active | Số lượng kết nối đang hoạt động | >70% worker_connections | >90% worker_connections |
| nginx_connections_reading | Số kết nối Nginx đang đọc yêu cầu | >30% active | >50% active |
| nginx_connections_writing | Số kết nối Nginx đang ghi phản hồi | >30% active | >50% active |
| nginx_connections_waiting | Số kết nối rảnh đang chờ yêu cầu | >70% active | >90% active |
| **Request Processing** |||
| nginx_http_requests_total | Tổng số yêu cầu HTTP | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| nginx_http_requests_per_second | Số yêu cầu HTTP mỗi giây | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| **Performance** |||
| nginx_request_time | Thời gian xử lý yêu cầu | p95 >500ms | p99 >2s |
| nginx_upstream_response_time | Thời gian phản hồi của upstream | p95 >500ms | p99 >2s |
| **Error Rates** |||
| nginx_http_4xx_rate | Tỷ lệ phản hồi 4xx | >5% | >10% |
| nginx_http_5xx_rate | Tỷ lệ phản hồi 5xx | >1% | >5% |
| nginx_upstream_next | Số yêu cầu chuyển sang server tiếp theo | >0 | >100/phút |
| nginx_upstream_fails | Số yêu cầu thất bại tới upstream server | >0 | >100/phút |
| **SSL** |||
| nginx_ssl_handshake_failures | Số lần thất bại trong bắt tay SSL | >0 | >100/phút |
| nginx_ssl_session_reuses | Số lần tái sử dụng phiên SSL | <50% | <30% |
| **Cache Performance** |||
| nginx_cache_hit_ratio | Tỷ lệ cache hit | <70% | <50% |
| nginx_cache_size | Kích thước cache | >80% max_size | >90% max_size |
| **Resource Usage** |||
| nginx_worker_processes | Số lượng tiến trình worker | ≠ số core CPU | N/A |
| nginx_worker_connections | Số kết nối tối đa mỗi worker | >80% | >90% |

### Ingress Nginx

| Metric | Description | Warning Threshold | Critical Threshold |
|--------|-------------|-------------------|---------------------|
| **Controller Status** |||
| nginx_ingress_controller_config_last_reload_successful | Lần tải lại cấu hình cuối cùng thành công | False | False trong >5 phút |
| nginx_ingress_controller_success | Tỷ lệ thành công của ingress controller | <100% | <95% |
| **Request Handling** |||
| nginx_ingress_controller_requests | Tổng số yêu cầu từ client | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| nginx_ingress_controller_request_duration_seconds | Thời gian xử lý yêu cầu | p95 >1s | p99 >5s |
| nginx_ingress_controller_request_size | Kích thước yêu cầu | Tăng đột ngột >30% | Tăng đột ngột >50% |
| nginx_ingress_controller_response_size | Kích thước phản hồi | Tăng đột ngột >30% | Tăng đột ngột >50% |
| **Error Rates** |||
| nginx_ingress_controller_status_4xx | Tỷ lệ phản hồi 4xx | >5% | >10% |
| nginx_ingress_controller_status_5xx | Tỷ lệ phản hồi 5xx | >1% | >5% |
| **SSL** |||
| nginx_ingress_controller_ssl_expire_time_seconds | Thời gian còn lại trước khi chứng chỉ SSL hết hạn | <30 ngày | <7 ngày |
| nginx_ingress_controller_ssl_handshake_errors_count | Số lỗi bắt tay SSL | >0 | >100/phút |
| **Upstream Services** |||
| nginx_ingress_controller_upstream_latency_seconds | Độ trễ của dịch vụ upstream | p95 >1s | p99 >5s |
| nginx_ingress_controller_upstream_status_4xx | Tỷ lệ 4xx từ upstream | >5% | >10% |
| nginx_ingress_controller_upstream_status_5xx | Tỷ lệ 5xx từ upstream | >1% | >5% |
| **Socket Status** |||
| nginx_ingress_controller_socket_queue_usage_pct | Tỷ lệ sử dụng hàng đợi socket | >70% | >90% |
| nginx_ingress_controller_worker_connections_usage_pct | Tỷ lệ sử dụng kết nối worker | >70% | >90% |
| **Resource Usage** |||
| nginx_ingress_controller_cpu_usage | Sử dụng CPU | >80% giới hạn | >90% giới hạn |
| nginx_ingress_controller_memory_usage | Sử dụng bộ nhớ | >80% giới hạn | >90% giới hạn |

## Service Mesh & Network

### Istio

| Metric | Description | Warning Threshold | Critical Threshold |
|--------|-------------|-------------------|---------------------|
| **Request Handling** |||
| istio_requests_total | Tổng số yêu cầu | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| istio_request_duration_milliseconds | Thời gian xử lý yêu cầu | p95 >500ms | p99 >2s |
| istio_request_bytes | Kích thước yêu cầu | Tăng đột ngột >30% | Tăng đột ngột >50% |
| istio_response_bytes | Kích thước phản hồi | Tăng đột ngột >30% | Tăng đột ngột >50% |
| **Error Rates** |||
| istio_requests_total (4xx) | Tỷ lệ lỗi 4xx | >5% | >10% |
| istio_requests_total (5xx) | Tỷ lệ lỗi 5xx | >1% | >5% |
| istio_request_duration_milliseconds (timeout) | Số lần yêu cầu bị timeout | >0.1% | >1% |
| **Circuit Breaking** |||
| istio_circuit_breakers_triggered | Số lần kích hoạt ngắt mạch | >0 | >10/phút |
| istio_circuit_breakers_current_open | Số lượng ngắt mạch đang mở | >0 | >3 |
| **Connection Metrics** |||
| istio_tcp_connections_opened_total | Số kết nối TCP được mở | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| istio_tcp_connections_closed_total | Số kết nối TCP được đóng | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| istio_tcp_received_bytes_total | Số bytes TCP nhận được | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| istio_tcp_sent_bytes_total | Số bytes TCP gửi đi | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| **mTLS** |||
| istio_mtls_request_percent | Phần trăm yêu cầu mTLS | <90% (khi yêu cầu mTLS) | <50% (khi yêu cầu mTLS) |
| **Control Plane** |||
| pilot_conflict_inbound_listener | Xung đột listener inbound | >0 | >10 |
| pilot_conflict_outbound_listener_http | Xung đột listener outbound HTTP | >0 | >10 |
| pilot_conflict_outbound_listener_tcp | Xung đột listener outbound TCP | >0 | >10 |
| pilot_services | Số lượng dịch vụ được theo dõi | Thay đổi đột ngột >10% | Thay đổi đột ngột >30% |
| pilot_xds_push_context_errors | Lỗi ngữ cảnh push XDS | >0 | >10 |
| **Sidecars** |||
| sidecar_injection_success_total | Số lần tiêm sidecar thành công | <100% | <90% |
| sidecar_injection_failure_total | Số lần tiêm sidecar thất bại | >0 | >10 |
| **Proxy Status** |||
| pilot_xds_cds_reject | Cấu hình CDS bị từ chối | >0 | >10 |
| pilot_xds_eds_reject | Cấu hình EDS bị từ chối | >0 | >10 |
| pilot_xds_rds_reject | Cấu hình RDS bị từ chối | >0 | >10 |
| pilot_xds_lds_reject | Cấu hình LDS bị từ chối | >0 | >10 |
| pilot_proxy_convergence_time | Thời gian hội tụ cấu hình proxy | >5s | >30s |
| **Gateway** |||
| pilot_k8s_cfg_events | Sự kiện cấu hình Kubernetes | Đột biến | Đột biến liên tục |
| pilot_virt_services | Số lượng dịch vụ ảo | Thay đổi đột ngột >10% | Thay đổi đột ngột >30% |

### Cilium

| Metric | Description | Warning Threshold | Critical Threshold |
|--------|-------------|-------------------|---------------------|
| **Agent Status** |||
| cilium_agent_health | Trạng thái sức khỏe của agent | Không healthy | Không healthy trong >5 phút |
| cilium_agent_uptime_seconds | Thời gian hoạt động của agent | Khởi động lại không mong muốn | Nhiều lần khởi động lại |
| **API Processing** |||
| cilium_api_process_time_seconds | Thời gian xử lý API | p95 >1s | p99 >5s |
| cilium_api_rate_limit_duration_seconds | Thời gian chờ giới hạn tốc độ API | >0s | >1s |
| **Endpoints & Identities** |||
| cilium_endpoint_count | Số lượng endpoint được quản lý | Thay đổi đột ngột >10% | Thay đổi đột ngột >30% |
| cilium_endpoint_regeneration_time_stats_seconds | Thời gian tái tạo endpoint | p95 >1s | p99 >5s |
| cilium_endpoint_state | Số endpoint chưa sẵn sàng | >0 | >5% tổng số |
| cilium_identity_count | Số lượng identity được cấp phát | >80% giới hạn | >90% giới hạn |
| **Connectivity** |||
| cilium_unreachable_nodes | Số lượng node không thể kết nối | >0 | >1 |
| cilium_unreachable_health_endpoints | Số lượng health endpoint không thể kết nối | >0 | >5% tổng số |
| **Policy** |||
| cilium_policy_count | Số lượng policy đã được import | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| cilium_policy_regeneration_time_stats_seconds | Thời gian tái tạo policy | p95 >1s | p99 >5s |
| cilium_policy_import_errors | Số lỗi khi import policy | >0 | >5 |
| cilium_policy_endpoint_enforcement_status | Trạng thái thực thi policy | Không được kích hoạt khi mong đợi | Không được kích hoạt trong >5 phút |
| **IP Address Management** |||
| cilium_ipam_ips | Số lượng IP đã cấp phát | >80% phạm vi | >90% phạm vi |
| cilium_ipam_allocation_ops | Số thao tác cấp phát IP | Tăng đột ngột | Tăng liên tục |
| **Proxies & Services** |||
| cilium_proxy_redirects | Số lượng chuyển hướng proxy | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| cilium_proxy_upstream_reply_seconds | Thời gian để nhận phản hồi từ upstream | p95 >1s | p99 >5s |
| **BPF Maps** |||
| cilium_bpf_map_ops_total | Số thao tác trên BPF map | Tăng đột ngột | Tăng liên tục |
| cilium_bpf_maps_virtual_memory_max_bytes | Lượng bộ nhớ ảo sử dụng cho BPF maps | >80% giới hạn | >90% giới hạn |
| **Datapath** |||
| cilium_datapath_errors_total | Tổng số lỗi datapath | >0 | >100/phút |
| cilium_drop_count_total | Số lượng gói tin bị hủy | >0 | >1000/phút |
| cilium_forward_count_total | Số lượng gói tin được chuyển tiếp | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| **Hubble** |||
| hubble_flows_processed_total | Số luồng mạng được xử lý | Thay đổi đột ngột >30% | Thay đổi đột ngột >50% |
| hubble_flows_lost_total | Số luồng mạng bị mất | >0 | >1000/phút |
| hubble_tcp_flags | Các cờ TCP trong luồng | Tăng đột ngột cờ RST/FIN | Tỷ lệ cao cờ RST |
| **Operators** |||
| cilium_operator_process_cpu_seconds_total | Mức sử dụng CPU của operator | >80% | >90% |
| cilium_operator_process_memory_resident_bytes | Mức sử dụng bộ nhớ của operator | >80% giới hạn | >90% giới hạn |
