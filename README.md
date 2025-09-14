# Real-time Log Ingestion & Processing with Multiple Pipelines

**Litaaya**

Đây là dự án sử dụng Docker để triển khai bộ ELK Stack (Elasticsearch, Logstash, Kibana) để thu thập, xử lý và phân tích log. Dự án này được thiết kế để xử lý nhiều loại dữ liệu log khác nhau qua các pipeline độc lập, bao gồm dữ liệu CICIDS 2018 và các dataset từ CERT Insider Threat.

---

## Table of Contents

- [Overview](#overview)
- [Architecture & Workflow](#architecture--workflow)
- [Project Structure](#project-structure)
- [Configuration Details](#configuration-details)
  - [Docker Compose](#docker-compose)
  - [Logstash Pipelines](#logstash-pipelines)
- [How to Run](#how-to-run)

---

## Overview

Dự án ELK#1 thực hiện các chức năng chính:

- **Ingestion**: Dữ liệu log được nhập từ các file CSV (ví dụ: CICIDS 2018, CERT Email, CERT Psychometric) nằm trong thư mục `/datasets` của Logstash.
- **Processing**: Logstash sử dụng mô hình đa pipeline để tách riêng các quy trình xử lý từng loại dữ liệu. Mỗi pipeline được định nghĩa trong các file cấu hình riêng biệt (ví dụ: `logstash_CICIDS.conf`, `logstash_CERT_email.conf`, `logstash_CERT_psychometric.conf`).
- **Storage & Visualization**: Dữ liệu đã xử lý được gửi sang Elasticsearch và được hiển thị, truy vấn qua Kibana.

---

## Architecture

### Chi tiết:
- **Logstash Pipelines**:  
  - Pipeline CICIDS: Đọc và xử lý file CSV từ `datasets/CICIDS_2018/`
  - Pipeline CERT_Email: Xử lý file CSV email từ `datasets/CERT_insider_threat/`
  - Pipeline CERT_Psychometric: Xử lý file CSV psychometric từ `datasets/CERT_insider_threat/`
  
  Mỗi pipeline thực hiện:
  - **Input**: Đọc file qua plugin `file` (với `sincedb_path => "NUL"` để luôn bắt đầu từ đầu file).
  - **Filter**: Sử dụng plugin `csv` để parse, `mutate` chuyển đổi kiểu dữ liệu, plugin `date` để xử lý trường thời gian, và các điều kiện để gán tag (vd: `"attack"`, `"benign"`).
  - **Output**: Gửi dữ liệu đã xử lý sang Elasticsearch với index riêng (ví dụ: `cicids2018-logs`, `cert-email-logs`, `cert-psychometric`).

- **Docker Compose**:  
  Quản lý các container cho **Elasticsearch**, **Logstash** và **Kibana**; các container được kết nối qua network chung (đặt tên là "elk"). Các volumes được gắn để truyền file cấu hình và dữ liệu giữa máy host và container.

---

## Configuration Details

### Docker Compose

File `docker-compose.yml` định nghĩa các dịch vụ:

- **Elasticsearch**:
  - Sử dụng image `elasticsearch:8.6.2`
  - Chạy ở chế độ single node (`discovery.type=single-node`)
  - Mở cổng 9200 (REST API) và 9300 (giao tiếp nội bộ, nếu cần)
  - Giới hạn bộ nhớ 1g

- **Kibana**:
  - Sử dụng image `kibana:8.6.2`
  - Mở cổng 5601 để truy cập giao diện web
  - Kết nối tới Elasticsearch qua biến môi trường `ELASTICSEARCH_HOSTS`
  - Có điều chỉnh NODE_OPTIONS để giới hạn bộ nhớ

- **Logstash**:
  - Sử dụng image `logstash:8.6.2`
  - Mở cổng 5044 để nhận dữ liệu đầu vào (từ Filebeat hoặc input file), và 9600 cho API monitoring
  - Áp dụng các volumes để gắn file cấu hình pipeline (`pipelines.yml`), thư mục `pipeline` (chứa các file *.conf) và `datasets` (dữ liệu nguồn)
  - Cấu hình bộ nhớ thông qua `LS_JAVA_OPTS`

### Logstash Pipelines

- File pipelines.yml định nghĩa các pipeline riêng
- Mỗi file cấu hình pipeline (ví dụ: logstash_CICIDS.conf) gồm:
  - Input: Đọc file CSV từ thư mục datasets
  - Filter: Sử dụng plugin csv, mutate, date và conditional để phân loại dữ liệu (chẳng hạn gán tag "attack" hoặc "benign")
  - Output: Gửi dữ liệu đã xử lý vào Elasticsearch với index cụ thể
Output: Gửi dữ liệu đã xử lý vào Elasticsearch với index cụ thể.

---

## How to run

- Clone dự án:
``` bash
git clone https://github.com/Litaaya/elk1.git
```
- Khởi động docker:
```
docker compose up -d
```
- Mở trình duyệt và truy cập: http://localhost:5601


