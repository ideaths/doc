# Elasticsearch: Cài Đặt, Cấu Hình và Tối Ưu Hiệu Suất

## Mục lục
- [1. Tổng quan về Elasticsearch](#1-tổng-quan-về-elasticsearch)
  - [1.1 Kiến trúc Elasticsearch](#11-kiến-trúc-elasticsearch)
  - [1.2 Thuật ngữ cơ bản](#12-thuật-ngữ-cơ-bản)
- [2. Cài đặt và cấu hình cơ bản](#2-cài-đặt-và-cấu-hình-cơ-bản)
  - [2.1 Cài đặt Elasticsearch](#21-cài-đặt-elasticsearch)
  - [2.2 File cấu hình elasticsearch.yml](#22-file-cấu-hình-elasticsearchyml)
  - [2.3 File cấu hình JVM](#23-file-cấu-hình-jvm)
- [3. Cấu hình Cluster](#3-cấu-hình-cluster)
  - [3.1 Thiết lập Multi-node Cluster](#31-thiết-lập-multi-node-cluster)
  - [3.2 Discovery và Quorum](#32-discovery-và-quorum)
  - [3.3 Shard Allocation](#33-shard-allocation)
- [4. Quản lý và tối ưu Index](#4-quản-lý-và-tối-ưu-index)
  - [4.1 Thiết kế Index hiệu quả](#41-thiết-kế-index-hiệu-quả)
  - [4.2 Mappings và Settings](#42-mappings-và-settings)
  - [4.3 Index Lifecycle Management](#43-index-lifecycle-management)
- [5. Tối ưu hiệu suất truy vấn](#5-tối-ưu-hiệu-suất-truy-vấn)
  - [5.1 Phân tích và tối ưu truy vấn](#51-phân-tích-và-tối-ưu-truy-vấn)
  - [5.2 Caching và bộ nhớ](#52-caching-và-bộ-nhớ)
  - [5.3 Search Templates và Aggregations](#53-search-templates-và-aggregations)
- [6. Cấu hình phần cứng và khả năng mở rộng](#6-cấu-hình-phần-cứng-và-khả-năng-mở-rộng)
  - [6.1 Yêu cầu phần cứng](#61-yêu-cầu-phần-cứng)
  - [6.2 Chiến lược mở rộng](#62-chiến-lược-mở-rộng)
- [7. Giám sát với Prometheus và Grafana](#7-giám-sát-với-prometheus-và-grafana)
  - [7.1 Cài đặt Elasticsearch Exporter](#71-cài-đặt-elasticsearch-exporter)
  - [7.2 Cấu hình Prometheus](#72-cấu-hình-prometheus)
  - [7.3 Dashboard Grafana](#73-dashboard-grafana)
- [8. Tối ưu hiệu suất](#8-tối-ưu-hiệu-suất)
  - [8.1 Tối ưu cấu hình JVM](#81-tối-ưu-cấu-hình-jvm)
  - [8.2 Bulk Indexing](#82-bulk-indexing)
  - [8.3 Refresh và Flush](#83-refresh-và-flush)
- [9. Backup và Restore](#9-backup-và-restore)
  - [9.1 Snapshot Repository](#91-snapshot-repository)
  - [9.2 Tạo và khôi phục Snapshot](#92-tạo-và-khôi-phục-snapshot)
- [10. Bảo mật](#10-bảo-mật)
  - [10.1 Bảo mật cơ bản](#101-bảo-mật-cơ-bản)
  - [10.2 Mã hóa và SSL/TLS](#102-mã-hóa-và-ssltls)
  - [10.3 Role-based Access Control](#103-role-based-access-control)

---

## 1. Tổng quan về Elasticsearch

### 1.1 Kiến trúc Elasticsearch

Elasticsearch là một công cụ tìm kiếm và phân tích phân tán dựa trên Apache Lucene. Nó được thiết kế để lưu trữ, tìm kiếm và phân tích lượng lớn dữ liệu gần thời gian thực.

**Kiến trúc phân tầng:**

1. **Cluster**: Tập hợp các node có cùng cluster.name làm việc cùng nhau
2. **Node**: Một instance của Elasticsearch, là một tiến trình Java riêng biệt
3. **Index**: Tập hợp các document với cấu trúc tương tự
4. **Shard**: Phân đoạn của index, cho phép phân tán dữ liệu
5. **Replica**: Bản sao của shard chính, cung cấp khả năng chịu lỗi và tăng hiệu suất đọc

**Vai trò Node:**
- **Master-eligible**: Có thể được bầu làm master node để quản lý cluster
- **Data**: Lưu trữ dữ liệu và thực hiện CRUD và search operations
- **Ingest**: Xử lý các pipeline trước khi indexing
- **Coordinating**: Nhận và phối hợp các request từ client
- **Machine Learning**: Chạy machine learning jobs (X-Pack)

### 1.2 Thuật ngữ cơ bản

- **Document**: Đơn vị dữ liệu cơ bản, được lưu dưới dạng JSON
- **Field**: Trường dữ liệu trong document
- **Type**: Loại dữ liệu của field (text, keyword, numeric, date...)
- **Mapping**: Schema định nghĩa cấu trúc của document
- **Analyzer**: Bộ xử lý text để phân tích và đánh chỉ mục
- **Query DSL**: Ngôn ngữ JSON-based để tìm kiếm và lọc dữ liệu
- **Aggregation**: Framework để thực hiện phân tích dữ liệu

## 2. Cài đặt và cấu hình cơ bản

### 2.1 Cài đặt Elasticsearch

**Trên Ubuntu/Debian:**

```bash
# Cài đặt các dependencies
apt-get update
apt-get install -y openjdk-11-jdk apt-transport-https

# Thêm Elasticsearch repository
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list

# Cài đặt Elasticsearch
apt-get update
apt-get install -y elasticsearch
```

**Trên CentOS/RHEL:**

```bash
# Cài đặt Java
yum install -y java-11-openjdk

# Thêm Elasticsearch repository
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
cat > /etc/yum.repos.d/elasticsearch.repo << EOF
[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF

# Cài đặt Elasticsearch
yum install -y elasticsearch
```

**Với Docker:**

```bash
docker run -d \
  --name elasticsearch \
  -p 9200:9200 -p 9300:9300 \
  -e "discovery.type=single-node" \
  -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
  docker.elastic.co/elasticsearch/elasticsearch:7.17.9
```

### 2.2 File cấu hình elasticsearch.yml

File cấu hình chính nằm ở `/etc/elasticsearch/elasticsearch.yml`. Dưới đây là những cấu hình cơ bản:

```yaml
# Cấu hình Cluster
cluster.name: my-elasticsearch

# Cấu hình Node
node.name: node-1
node.roles: [master, data, ingest]

# Cấu hình Network
network.host: 0.0.0.0
http.port: 9200
transport.port: 9300

# Cấu hình Discovery
discovery.seed_hosts: ["127.0.0.1"]
cluster.initial_master_nodes: ["node-1"]

# Cấu hình đường dẫn
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch

# Cấu hình Memory
bootstrap.memory_lock: true  # Ngăn swap để tối ưu hiệu suất
```

### 2.3 File cấu hình JVM

File cấu hình JVM nằm ở `/etc/elasticsearch/jvm.options`:

```
# Cấu hình Heap Size
-Xms4g
-Xmx4g

# GC settings
-XX:+UseG1GC
-XX:G1ReservePercent=25
-XX:InitiatingHeapOccupancyPercent=30

# JVM temporary directory
-Djava.io.tmpdir=${ES_TMPDIR}
```

**Lưu ý quan trọng cho heap size:**
- Không vượt quá 50% RAM vật lý
- Không vượt quá Compressed Ordinary Object Pointers (compressed oops) threshold (~32GB)
- Không cấu hình quá nhỏ (tối thiểu 2GB cho môi trường production)

## 3. Cấu hình Cluster

### 3.1 Thiết lập Multi-node Cluster

**Cấu hình cho Node 1 (Master):**
```yaml
cluster.name: es-cluster
node.name: node-1
node.roles: [master, data]
network.host: 192.168.1.1
http.port: 9200
transport.port: 9300
discovery.seed_hosts: ["192.168.1.1:9300", "192.168.1.2:9300", "192.168.1.3:9300"]
cluster.initial_master_nodes: ["node-1"]
```

**Cấu hình cho Node 2 (Data):**
```yaml
cluster.name: es-cluster
node.name: node-2
node.roles: [data]
network.host: 192.168.1.2
http.port: 9200
transport.port: 9300
discovery.seed_hosts: ["192.168.1.1:9300", "192.168.1.2:9300", "192.168.1.3:9300"]
```

**Cấu hình cho Node 3 (Data):**
```yaml
cluster.name: es-cluster
node.name: node-3
node.roles: [data]
network.host: 192.168.1.3
http.port: 9200
transport.port: 9300
discovery.seed_hosts: ["192.168.1.1:9300", "192.168.1.2:9300", "192.168.1.3:9300"]
```

### 3.2 Discovery và Quorum

Elasticsearch sử dụng Zen Discovery để tìm và giao tiếp giữa các node. Trong Elasticsearch 7.x, cơ chế discovery được cải tiến với quorum-based decision making:

```yaml
# Danh sách các node có thể được kết nối
discovery.seed_hosts: ["192.168.1.1", "192.168.1.2", "192.168.1.3"]

# Các master node ban đầu (chỉ sử dụng khi khởi tạo cluster mới)
cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]

# Thời gian chờ khi discovery
discovery.request_timeout: 120s

# Cấu hình fault detection
cluster.fault_detection.leader_check.timeout: 10s
cluster.fault_detection.follower_check.timeout: 10s
```

**Best practice về số lượng master nodes:**
- Nên có số lẻ master-eligible nodes (3, 5, 7...)
- Số lượng tối thiểu: 3 (để tránh split-brain)
- Công thức tính quorum: (n/2) + 1 (với n là số master-eligible nodes)

### 3.3 Shard Allocation

Cân bằng tài nguyên cluster thông qua shard allocation:

```yaml
# Tắt re-balancing khi có node rời khỏi cluster
cluster.routing.allocation.enable: all

# Số primary shard thực hiện re-allocation song song
cluster.routing.allocation.cluster_concurrent_rebalance: 2

# Số shards di chuyển cùng lúc trên mỗi node
cluster.routing.allocation.node_concurrent_recoveries: 2

# Ngưỡng đĩa để allocation 
cluster.routing.allocation.disk.threshold_enabled: true
cluster.routing.allocation.disk.watermark.low: 85%
cluster.routing.allocation.disk.watermark.high: 90%
cluster.routing.allocation.disk.watermark.flood_stage: 95%
```

**Lưu ý về số lượng Shards và Replicas:**
- Số shards tối ưu phụ thuộc vào kích thước dữ liệu và số node
- Nguyên tắc: 1 shard nên từ 20GB-40GB
- Với 5 nodes: chọn khoảng 5-10 primary shards để đảm bảo phân phối đều
- Replicas: Thường là 1, có thể tăng lên trong trường hợp cần đọc nhiều

## 4. Quản lý và tối ưu Index

### 4.1 Thiết kế Index hiệu quả

**Time-based Index Strategy:**
- Tạo index theo khoảng thời gian (ngày, tuần, tháng)
- Ví dụ: `logs-2023.03.16`, `logs-2023.03.17`
- Lợi ích: Dễ dàng quản lý lifecycle và xóa dữ liệu cũ

**Index Template:**
```json
PUT _template/logs_template
{
  "index_patterns": ["logs-*"],
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "refresh_interval": "5s"
  },
  "mappings": {
    "properties": {
      "@timestamp": { "type": "date" },
      "message": { "type": "text" },
      "level": { "type": "keyword" },
      "service": { "type": "keyword" }
    }
  }
}
```

**Index Alias:**
```json
POST /_aliases
{
  "actions": [
    { "add": { "index": "logs-2023.03.16", "alias": "logs-current" } }
  ]
}
```

### 4.2 Mappings và Settings

**Dynamic vs Explicit Mappings:**

Dynamic mapping (tự động):
```json
PUT my_index/_doc/1
{
  "name": "John Doe",
  "age": 30,
  "created": "2023-03-16T10:00:00Z"
}
```

Explicit mapping (định nghĩa trước):
```json
PUT my_index
{
  "mappings": {
    "properties": {
      "name": { 
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "age": { "type": "integer" },
      "created": { "type": "date" }
    }
  }
}
```

**Các settings quan trọng:**
```json
PUT my_index
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "refresh_interval": "1s",
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "asciifolding"]
        }
      }
    }
  }
}
```

### 4.3 Index Lifecycle Management

ILM (Index Lifecycle Management) giúp quản lý tự động vòng đời của index:

```json
PUT _ilm/policy/logs_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_age": "7d",
            "max_size": "50gb"
          },
          "set_priority": {
            "priority": 100
          }
        }
      },
      "warm": {
        "min_age": "30d",
        "actions": {
          "shrink": {
            "number_of_shards": 1
          },
          "forcemerge": {
            "max_num_segments": 1
          },
          "set_priority": {
            "priority": 50
          }
        }
      },
      "cold": {
        "min_age": "60d",
        "actions": {
          "set_priority": {
            "priority": 0
          },
          "freeze": {}
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

**Áp dụng ILM với index template:**
```json
PUT _template/logs_template
{
  "index_patterns": ["logs-*"],
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "index.lifecycle.name": "logs_policy",
    "index.lifecycle.rollover_alias": "logs"
  }
}
```

## 5. Tối ưu hiệu suất truy vấn

### 5.1 Phân tích và tối ưu truy vấn

**Profile API:**
```json
GET my_index/_search
{
  "profile": true,
  "query": {
    "match": {
      "message": "error"
    }
  }
}
```

**Slow Log:**
```yaml
# elasticsearch.yml
index.search.slowlog.threshold.query.warn: 10s
index.search.slowlog.threshold.query.info: 5s
index.search.slowlog.threshold.query.debug: 2s
index.search.slowlog.threshold.query.trace: 500ms
index.search.slowlog.threshold.fetch.warn: 1s
index.search.slowlog.threshold.fetch.info: 800ms
index.search.slowlog.threshold.fetch.debug: 500ms
index.search.slowlog.threshold.fetch.trace: 200ms
```

**Chiến lược tối ưu truy vấn:**
1. Sử dụng filter thay vì query khi có thể (filter được cache)
2. Sử dụng page và size hợp lý
3. Chỉ lấy trường cần thiết với `_source` filtering
4. Tránh sử dụng `wildcard` và `regexp` queries

**Ví dụ truy vấn tối ưu:**
```json
GET my_index/_search
{
  "_source": ["title", "created"],
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "elasticsearch" } }
      ],
      "filter": [
        { "term": { "status": "published" } },
        { "range": { "created": { "gte": "now-1M" } } }
      ]
    }
  },
  "size": 20,
  "from": 0
}
```

### 5.2 Caching và bộ nhớ

Elasticsearch sử dụng nhiều loại cache khác nhau:

**Node Query Cache:**
```yaml
indices.queries.cache.size: 10%
```

**Shard Request Cache:**
```yaml
indices.requests.cache.size: 1%
```

**Field Data Cache:**
```yaml
indices.fielddata.cache.size: 10% 
```

**Circuit Breaker:**
```yaml
indices.breaker.total.limit: 70%
indices.breaker.request.limit: 60%
indices.breaker.fielddata.limit: 40%
```

### 5.3 Search Templates và Aggregations

**Search Templates:**
```json
POST _scripts/my_template
{
  "script": {
    "lang": "mustache",
    "source": {
      "query": {
        "match": {
          "{{field}}": "{{value}}"
        }
      }
    }
  }
}

GET _search/template
{
  "id": "my_template",
  "params": {
    "field": "message",
    "value": "error"
  }
}
```

**Tối ưu Aggregations:**
1. Sử dụng keyword thay vì text cho fields cần aggregation
2. Giới hạn số lượng buckets với `size` hoặc `shard_size`
3. Sử dụng pre-aggregations hoặc rollups cho dữ liệu lớn

```json
GET logs-*/_search
{
  "size": 0,
  "aggs": {
    "by_service": {
      "terms": {
        "field": "service.keyword",
        "size": 10,
        "shard_size": 100
      },
      "aggs": {
        "error_count": {
          "filter": {
            "term": {
              "level.keyword": "ERROR"
            }
          }
        }
      }
    }
  }
}
```

## 6. Cấu hình phần cứng và khả năng mở rộng

### 6.1 Yêu cầu phần cứng

**CPU:**
- Từ 4-8 CPU cores cho node cỡ trung bình
- 8-16 cores cho node xử lý workload nặng
- Elasticsearch ưu tiên CPU có hiệu năng single-thread tốt

**Memory:**
- 50% cho JVM heap (tối đa ~30GB để tận dụng compressed pointers)
- 50% cho Lucene, filesystem cache và OS

**Disk:**
- SSD được ưu tiên hơn HDD
- RAID 0 để tối ưu performance (cần backup strategy tốt)
- Nguyên tắc: 1.5x-2x dung lượng dữ liệu gốc

**Network:**
- 1 Gbps tối thiểu cho môi trường production
- 10 Gbps được khuyến nghị cho cluster lớn

### 6.2 Chiến lược mở rộng

**Scale Vertically (tăng tài nguyên node):**
- Thích hợp cho cluster nhỏ và trung bình
- Thêm RAM, CPU, SSD nhanh hơn
- Giới hạn: JVM heap size không nên quá 30-32GB

**Scale Horizontally (thêm node):**
- Thích hợp cho workload lớn
- Cân nhắc tỷ lệ master:data nodes
- Hot-warm-cold architecture

**Hot-Warm-Cold Architecture:**
```yaml
# Hot node (SSD, nhiều RAM)
node.attr.temperature: hot

# Warm node (HDD, ít RAM hơn)
node.attr.temperature: warm

# Index setting
PUT logs-*
{
  "settings": {
    "index.routing.allocation.require.temperature": "hot"
  }
}
```

**Shard Allocation Filtering:**
```yaml
# Allocation cho node cụ thể
PUT logs-current/_settings
{
  "index.routing.allocation.include.zone": "zone1",
  "index.routing.allocation.exclude.zone": "zone3"
}
```

## 7. Giám sát với Prometheus và Grafana

### 7.1 Cài đặt Elasticsearch Exporter

```bash
# Tải Elasticsearch Exporter
wget https://github.com/prometheus-community/elasticsearch_exporter/releases/download/v1.5.0/elasticsearch_exporter-1.5.0.linux-amd64.tar.gz
tar xvf elasticsearch_exporter-1.5.0.linux-amd64.tar.gz
cp elasticsearch_exporter-1.5.0.linux-amd64/elasticsearch_exporter /usr/local/bin/

# Tạo systemd service
cat > /etc/systemd/system/elasticsearch_exporter.service << EOF
[Unit]
Description=Elasticsearch Exporter
After=network.target

[Service]
User=prometheus
ExecStart=/usr/local/bin/elasticsearch_exporter \
  --es.uri=http://localhost:9200 \
  --es.all \
  --es.indices \
  --es.shards \
  --web.listen-address=:9114 \
  --web.telemetry-path=/metrics

[Install]
WantedBy=multi-user.target
EOF

# Khởi động service
systemctl daemon-reload
systemctl enable elasticsearch_exporter
systemctl start elasticsearch_exporter
```

### 7.2 Cấu hình Prometheus

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'elasticsearch'
    static_configs:
      - targets: ['localhost:9114']
        labels:
          instance: 'es-cluster'
```

### 7.3 Dashboard Grafana

Import Elasticsearch Dashboard có sẵn (ID: 266) hoặc tạo dashboard tùy chỉnh với các panel sau:

**Elasticsearch Cluster Health:**
```
elasticsearch_cluster_health_status{color="green"}
elasticsearch_cluster_health_status{color="yellow"}
elasticsearch_cluster_health_status{color="red"}
```

**JVM Heap Usage:**
```
elasticsearch_jvm_memory_used_bytes{area="heap"} / elasticsearch_jvm_memory_max_bytes{area="heap"} * 100
```

**Index Performance:**
```
rate(elasticsearch_indices_search_query_time_seconds[1m])
rate(elasticsearch_indices_indexing_index_time_seconds[1m])
```

**Node Metrics:**
```
elasticsearch_filesystem_data_available_bytes / elasticsearch_filesystem_data_size_bytes * 100
```

**Các Metrics quan trọng cần theo dõi:**

| Metric | Mô tả | Threshold cảnh báo |
|--------|-------|-------------------|
| Cluster Health | Trạng thái cluster | Khác "green" |
| JVM Heap | % sử dụng heap | >80% |
| GC Time | Thời gian GC | >1s cho Old GC |
| CPU Usage | % sử dụng CPU | >80% |
| Disk Usage | % sử dụng đĩa | >80% |
| Search Latency | Thời gian truy vấn | >500ms |
| Indexing Latency | Thời gian index | >100ms |
| Thread Pool Rejection | Số lượng rejected | >0 |

## 8. Tối ưu hiệu suất

### 8.1 Tối ưu cấu hình JVM

**Heap Size:**
```
# jvm.options
-Xms16g
-Xmx16g
```

**Nguyên tắc heap size:**
- Không vượt quá 50% RAM vật lý
- Không vượt quá 30-32GB để tận dụng compressed pointers
- Xms và Xmx nên bằng nhau để tránh resizing

**Garbage Collection:**
```
# jvm.options
# G1GC (mặc định cho Java 9+)
-XX:+UseG1GC
-XX:G1ReservePercent=25
-XX:InitiatingHeapOccupancyPercent=30

# GC logging
-Xlog:gc*,gc+age=trace,safepoint:file=/var/log/elasticsearch/gc.log:utctime,pid,tags:filecount=32,filesize=64m
```

**Theo dõi GC:**
```bash
# Sử dụng jstat để xem GC statistics
jstat -gc $(pgrep -f Elasticsearch) 5000
```

### 8.2 Bulk Indexing

**Cấu hình tối ưu cho Bulk Indexing:**
```yaml
# Tạm thời vô hiệu hóa refresh
PUT my_index/_settings
{
  "index": {
    "refresh_interval": "-1",
    "number_of_replicas": 0
  }
}

# Bulk indexing
POST _bulk
{ "index": { "_index": "my_index", "_id": "1" } }
{ "field1": "value1" }
{ "index": { "_index": "my_index", "_id": "2" } }
{ "field1": "value2" }
# ...

# Khôi phục lại cấu hình
PUT my_index/_settings
{
  "index": {
    "refresh_interval": "1s",
    "number_of_replicas": 1
  }
}
```

**Kích thước batch tối ưu:**
- 5MB-15MB là kích thước phù hợp cho bulk request
- Monitor thread pool rejection để điều chỉnh
- Số lượng documents phụ thuộc vào kích thước document

### 8.3 Refresh và Flush

**Refresh Interval:**
```yaml
# Cấu hình mặc định: 1s
PUT my_index/_settings
{
  "index": {
    "refresh_interval": "5s"  # Tăng lên cho hiệu năng indexing cao hơn
  }
}
```

**Translog Settings:**
```yaml
PUT my_index/_settings
{
  "index": {
    "translog.sync_interval": "5s",
    "translog.durability": "async"  # Cẩn thận: có thể mất dữ liệu khi crash
  }
}
```

**Forcemerge:**
```
POST my_index/_forcemerge?max_num_segments=1
```
Thực hiện sau khi bulk indexing hoặc trên các index không còn cập nhật.

## 9. Backup và Restore

### 9.1 Snapshot Repository

**Cấu hình repository:**
```yaml
# elasticsearch.yml
path.repo: ["/path/to/backup"]
```

**Đăng ký repository:**
```json
PUT _snapshot/my_backup
{
  "type": "fs",
  "settings": {
    "location": "/path/to/backup",
    "compress": true
  }
}
```

**Cloud storage (S3):**
```json
PUT _snapshot/s3_backup
{
  "type": "s3",
  "settings": {
    "bucket": "my-es-backups",
    "region": "ap-southeast-1",
    "compress": true
  }
}
```

### 9.2 Tạo và khôi phục Snapshot

**Tạo snapshot:**
```json
PUT _snapshot/my_backup/snapshot_1
{
  "indices": "index1,index2",
  "ignore_unavailable": true,
  "include_global_state": false
}
```

**Xem thông tin snapshot:**
```
GET _snapshot/my_backup/snapshot_1
```

**Khôi phục snapshot:**
```json
POST _snapshot/my_backup/snapshot_1/_restore
{
  "indices": "index1,index2",
  "rename_pattern": "index(.+)",
  "rename_replacement": "restored_index$1",
  "include_global_state": false
}
```

**Snapshot Lifecycle Management (SLM):**
```json
PUT _slm/policy/daily_snapshots
{
  "schedule": "0 30 1 * * ?",  # Cron syntax: 1:30 AM mỗi ngày
  "name": "<daily-snap-{now/d}>",
  "repository": "my_backup",
  "config": {
    "indices": ["*"],
    "ignore_unavailable": true,
    "include_global_state": false
  },
  "retention": {
    "expire_after": "30d",
    "min_count": 5,
    "max_count": 50
  }
}
```

## 10. Bảo mật

### 10.1 Bảo mật cơ bản

**Network restrictions:**
```yaml
# elasticsearch.yml
network.host: 192.168.1.10
discovery.seed_hosts: ["192.168.1.10", "192.168.1.11"]
http.port: 9200
transport.port: 9300
```

**Firewall configuration (UFW):**
```bash
ufw allow from 192.168.1.0/24 to any port 9200
ufw allow from 192.168.1.0/24 to any port 9300
```

**X-Pack Security (Basic):**
```yaml
# elasticsearch.yml
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
```

### 10.2 Mã hóa và SSL/TLS

**Tạo certificate:**
```bash
# Tạo CA
bin/elasticsearch-certutil ca

# Tạo certificate cho node
bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
```

**HTTP SSL:**
```yaml
# elasticsearch.yml
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.keystore.path: http.p12
xpack.security.http.ssl.truststore.path: http.p12
xpack.security.http.ssl.client_authentication: optional
```

**Kiểm tra SSL configuration:**
```bash
openssl s_client -connect localhost:9200
```

### 10.3 Role-based Access Control

**Tạo user:**
```bash
bin/elasticsearch-users useradd john -p secure_password -r superuser
```

**Tạo role:**
```json
POST _security/role/logs_read
{
  "cluster": ["monitor"],
  "indices": [
    {
      "names": ["logs-*"],
      "privileges": ["read", "view_index_metadata"]
    }
  ]
}
```

**Gán role cho user:**
```json
POST _security/user/loguser
{
  "password" : "logpassword",
  "roles" : [ "logs_read" ],
  "full_name" : "Log User",
  "email" : "loguser@example.com"
}
```

**API Key:**
```json
POST /_security/api_key
{
  "name": "application-key",
  "expiration": "1d", 
  "role_descriptors": {
    "role": {
      "cluster": ["monitor"],
      "indices": [
        {
          "names": ["logs-*"],
          "privileges": ["read"]
        }
      ]
    }
  }
}
```

---

Hướng dẫn này cung cấp cách cài đặt, cấu hình và tối ưu Elasticsearch cho môi trường production. Hãy điều chỉnh các cấu hình dựa trên nhu cầu cụ thể và đặc điểm workload của bạn.
