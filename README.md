# Frappe / ERPNext – Practical Knowledge Base

📚 Repository này dùng để **tổng hợp kiến thức, ghi chú và kinh nghiệm thực tế** khi làm việc với **Frappe Framework** và **ERPNext**, tập trung vào **customization, triển khai và vận hành hệ thống trong môi trường production**.

---

## 🎯 Mục tiêu
- Lưu trữ kiến thức Frappe / ERPNext một cách **có hệ thống**
- Ghi chú các **case thực tế**, lỗi thường gặp và cách xử lý
- Chia sẻ **best practices** khi customize và triển khai ERPNext
- Làm tài liệu tham khảo nhanh cho quá trình phát triển & bảo trì

---

## 🧠 Phạm vi nội dung

### 1. Frappe Framework
- Kiến trúc tổng thể
- App, DocType, Module
- Hooks & Events
- Permissions & Roles
- Background Jobs
- REST API & Integration
- Jinja, Web Form, Portal

### 2. ERPNext Core
- Buying & Selling
- Stock & Warehouse
- Accounting
- Manufacturing
- Projects
- HRMS
- CRM

### 3. Customization
- Custom App
- Custom Field vs Customize Form
- Client Script / Server Script
- Override & Monkey Patch
- Custom Print Format
- Workflow
- Notification & Email

### 4. Stock & Purchasing (Thực chiến)
- Purchase Order / Purchase Receipt
- Material Request
- Blanket Order
- KCS / QC Flow
- Multi-Warehouse Logic
- Stock Ledger & GL Entry

### 5. Accounting
- Chart of Accounts
- GL Entry Flow
- Multi Currency
- Taxes & Discounts
- Linking Stock & Accounting

### 6. File & S3
- File Storage Architecture
- S3 Integration
- Public vs Private File
- Preview DOCX / XLSX / PDF
- Drive App (Custom File System)

### 7. Deployment & Ops
- Bench CLI
- Production Setup
- Supervisor / Systemd
- Nginx
- Backup & Restore
- Performance Optimization

---

## 🗂️ Cấu trúc thư mục

```
.
├── frappe/
│   ├── architecture.md
│   ├── hooks.md
│   ├── permissions.md
│   └── background_jobs.md
│
├── erpnext/
│   ├── buying.md
│   ├── selling.md
│   ├── stock.md
│   ├── accounting.md
│   └── manufacturing.md
│
├── customization/
│   ├── custom_app.md
│   ├── client_script.md
│   ├── server_script.md
│   ├── workflow.md
│   └── print_format.md
│
├── stock_real_cases/
│   ├── blanket_order_flow.md
│   ├── kcs_quantity.md
│   └── multi_warehouse.md
│
├── file_s3/
│   ├── s3_config.md
│   ├── public_private.md
│   └── office_preview.md
│
├── deployment/
│   ├── bench.md
│   ├── production.md
│   └── backup_restore.md
│
└── README.md
```

---

## 🧪 Ví dụ nội dung mỗi file
- Mô tả vấn đề
- Phân tích logic
- Ưu / nhược điểm
- Code sample (nếu có)
- Lưu ý production

---

## 🛠️ Công nghệ sử dụng
- Frappe Framework
- ERPNext
- Python
- JavaScript
- Vue.js
- MariaDB / MySQL
- Redis
- S3-compatible Storage

---

## ⚠️ Lưu ý
- Tài liệu dựa trên **kinh nghiệm thực tế**, không thay thế tài liệu chính thức
- Một số giải pháp có thể **phụ thuộc version**
- Luôn kiểm tra lại trước khi áp dụng production

---

## 📎 Tài liệu tham khảo
- Frappe Documentation
- ERPNext Documentation
- Frappe GitHub
- ERPNext GitHub

---

## 👤 Tác giả
**Tolia Bui**  
Frappe / ERPNext Developer

---

## ⭐ Định hướng tương lai
- Bổ sung sơ đồ (diagram)
- Chuẩn hóa flow Stock & Accounting
- Tổng hợp lỗi thường gặp theo version
- Case study triển khai thực tế
