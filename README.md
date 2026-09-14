# Hệ thống SIEM Đa Đám Mây: Tập trung Logs từ OpenStack & AWS tích hợp ELK Stack

Giải pháp Quản lý Sự kiện và An ninh Thông tin (SIEM) tập trung được thiết kế cho môi trường điện toán đám mây lai (Hybrid Cloud), tích hợp **ELK Stack (v7.17)** để thu thập, chuẩn hóa và phát hiện các mối đe dọa bảo mật trên hạ tầng **OpenStack (Private Cloud)** và **AWS (Public Cloud)**.

---

## 🎥 Video Demo Thực nghiệm 3 Kịch bản Tấn công
Bạn có thể xem chi tiết quá trình giả lập tấn công và hệ thống SIEM kích hoạt cảnh báo thời gian thực qua các liên kết dưới đây:
* [Xem Demo Kịch bản 1: Tấn công SSH Brute-force (Ubuntu OS)](https://youtu.be/4cFDx3EoJbo)
* [Xem Demo Kịch bản 2: Dò tìm tài khoản AWS Console Login Failure](https://youtu.be/h6biN0oe4wk)
* [Xem Demo Kịch bản 3: Xóa tài nguyên S3 Bucket (Phá hoại dữ liệu)](https://youtu.be/2UlhfUk2h8Q)

---

## 🏗️ 1. Kiến trúc Hệ thống
Kiến trúc SIEM được phân thành 4 lớp chuyên biệt nhằm đảm bảo tính co giãn, hiệu năng cao và xử lý sự cố thời gian thực:
1. **Lớp Thu thập (Ingestion Layer):** Các tác nhân Filebeat triển khai trên máy chủ Host OpenStack và AWS API Module chủ động kéo log từ S3 Bucket.
2. **Lớp Xử lý & Lưu trữ (Processing & Storage Layer):** Logstash lọc và phân rã log thô thành **Lược đồ dữ liệu chung Elastic Common Schema (ECS)** dựa trên 5 ngữ cảnh an ninh (*Khi nào, Ai, Cái gì, Ở đâu, Kết quả*), sau đó lưu trữ tại Elasticsearch.
3. **Lớp Phát hiện (Detection Engine):** Kibana Detection Engine chạy các quy tắc truy vấn KQL (Kibana Query Language) được biên dịch từ các bộ luật Sigma chuẩn hóa.
4. **Lớp Trực quan hóa & Cảnh báo (Visualization & Alerting):** Kibana Security Dashboards cung cấp cảnh báo thời gian thực và dòng thời gian sự kiện trực quan cho kỹ sư SOC.

---

## 🛠️ 2. Công nghệ Sử dụng (Tech Stack)
* **SIEM Cốt lõi:** ELK Stack (Elasticsearch, Logstash, Kibana v7.17)
* **Tác nhân Thu thập Log:** Filebeat (viết bằng Go, tối ưu tài nguyên), AWS API Module
* **Private Cloud:** OpenStack (Kolla-Ansible Docker Containers: Keystone, Nova, Neutron)
* **Public Cloud:** AWS (CloudTrail, S3, IAM)
* **Hệ điều hành Mục tiêu:** Ubuntu Server (`/var/log/auth.log`)
* **Ngôn ngữ Phát hiện:** KQL / Sigma Rules

---

## 🚨 3. Các Kịch bản Tấn công & Quy tắc Phát hiện Thực nghiệm
Hệ thống đã được kiểm thử thành công đối với 3 kịch bản tấn công thực tế ánh xạ theo ma trận **MITRE ATT&CK**:

| Mã KB | Kịch bản Tấn công | Hệ thống Mục tiêu | Cơ chế Phát hiện (KQL / Ngưỡng) | Mức độ Cảnh báo |
| :--- | :--- | :--- | :--- | :--- |
| **KB-01** | Tấn công dò mật khẩu SSH Brute-force | Máy ảo Ubuntu (`victim-2`) | Threshold Rule: $>20$ lần đăng nhập thất bại trong 1 phút (`event.dataset: "system.auth"`) | **Critical (Nguy cấp)** |
| **KB-02** | Dò tìm tài khoản AWS Console Login Failure | AWS Control Plane | Threshold Rule: $>10$ lần lỗi truy cập trong 5 phút (`aws.cloudtrail` & `ConsoleLogin`) | **High (Cao)** |
| **KB-03** | Xóa tài nguyên S3 Bucket (Phá hoại dữ liệu) | AWS S3 Storage | Single Event Rule: Phát hiện tức thời lệnh gọi API (`DeleteBucket`, `DeleteObjects`) | **Critical (Nguy cấp)** |

---

## 📁 4. Cấu trúc Thư mục Dự án
```text
├── configs/
│   ├── logstash/         # File cấu hình filter và biểu thức grok của Logstash
│   └── filebeat/         # Cấu hình Filebeat cho OpenStack host và AWS S3
├── rules/
│   └── kql-sigma/        # Luật phát hiện Kibana và các quy tắc Sigma đã dịch
├── dashboards/           # Các bảng điều khiển (Dashboard) bảo mật xuất từ Kibana
└── docs/                 # Tài liệu chi tiết và sơ đồ kiến trúc
