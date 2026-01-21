# 🚀 Hướng dẫn Cấu hình & Test VNPT eContract

## Mục tiêu
Hướng dẫn chi tiết từng bước để cấu hình và test gửi hợp đồng đầu tiên sang VNPT eContract.

---

## ✅ BƯỚC 1: Chuẩn bị thông tin từ VNPT

Trước khi cấu hình, bạn cần có các thông tin sau từ VNPT:

### 📋 Thông tin bắt buộc:

| Thông tin | Mô tả | Ví dụ |
|-----------|-------|-------|
| **Base URL** | Địa chỉ API gateway | `https://gateway-bus-econtract-v2-poc.vnpt.vn/` |
| **Tài khoản** | Username đăng nhập | `admincompany` |
| **Mật khẩu** | Password | `YourPassword@123` |
| **Document Type ID** | ID loại tài liệu | `6` (hoặc lấy từ VNPT) |

### 📋 Thông tin tùy chọn (nếu có):

| Thông tin | Mô tả | 
|-----------|-------|
| **Client ID** | Mã ứng dụng tích hợp |
| **Client Secret** | Mã bí mật ứng dụng |
| **Customer ID** | ID khách hàng/đối tác |

### 🔍 Cách lấy thông tin:

1. **Môi trường POC (Test):**
   - URL: `https://gateway-bus-econtract-v2-poc.vnpt.vn/`
   - Portal: `https://econtract-v2-poc.vnpt.vn/`

2. **Môi trường Production:**
   - URL: `https://gateway-bus-econtract.vnpt.vn/`
   - Portal: `https://econtract-v2.vnpt.vn/`

3. **Lấy Document Type ID:**
   - Login vào VNPT Portal
   - Vào **Danh mục** → **Loại tài liệu**
   - Ghi nhớ ID của loại bạn muốn dùng (VD: "Hợp đồng lao động" = 11)

---

## ✅ BƯỚC 2: Cấu hình VNPT eContract Settings

### 2.1. Truy cập Settings

```
Menu → Setup → VNPT eContract Settings
```

### 2.2. Tab "Connection" - Thông tin kết nối

```
┌─────────────────────────────────────────────────────────────┐
│ ☑ Enabled                                                   │
│ ☑ Auto Download Signed Contracts                           │
│                                                             │
│ ────────── Connection ─────────────────────────────────    │
│                                                             │
│ Base URL: https://gateway-bus-econtract-v2-poc.vnpt.vn/   │
│ Account: admincompany                                       │
│ Password: ••••••••                                         │
│ Client ID: (để trống nếu không có)                        │
│ Client Secret: (để trống nếu không có)                    │
│                                                             │
│ ────────── Column Break ────────────────────────────────   │
│                                                             │
│ Default Print Format: (chọn format in hợp đồng)           │
│ Subject Prefix: Hợp đồng                                   │
│ Customer ID: (để trống nếu không có)                       │
│ Platform ID: 4                                             │
└─────────────────────────────────────────────────────────────┘
```

**Chi tiết từng field:**

| Field | Giá trị | Bắt buộc | Ghi chú |
|-------|---------|----------|---------|
| **Enabled** | ☑ Checked | ✅ Có | Bật tích hợp |
| **Auto Download Signed** | ☑ Checked | ❌ Không | Tự động tải file đã ký |
| **Base URL** | `https://gateway-bus-econtract-v2-poc.vnpt.vn/` | ✅ Có | Phải có dấu `/` cuối |
| **Account** | `admincompany` | ✅ Có | Tài khoản VNPT cung cấp |
| **Password** | `YourPassword@123` | ✅ Có | Mật khẩu đăng nhập |
| **Client ID** | Để trống hoặc điền | ❌ Không | Nếu VNPT cung cấp |
| **Client Secret** | Để trống hoặc điền | ❌ Không | Nếu VNPT cung cấp |
| **Default Print Format** | Standard | ❌ Không | Format in Contract |
| **Subject Prefix** | `Hợp đồng` | ❌ Không | Tiền tố tên hợp đồng |
| **Customer ID** | Để trống hoặc điền | ❌ Không | Nếu VNPT cung cấp |
| **Platform ID** | `4` | ✅ Có | 4 = SDK Web (default) |

### 2.3. Tab "Contract Defaults" - Cấu hình mặc định

```
┌─────────────────────────────────────────────────────────────┐
│ ────────── Contract Defaults ────────────────────────────   │
│                                                             │
│ Document Type ID: 6                    ⚠️ BẮT BUỘC         │
│ Contract Flow ID: 1                                         │
│ Permission To View: 1                                       │
│                                                             │
│ ────────── Column Break ────────────────────────────────   │
│                                                             │
│ Document Group: 1                                          │
│ flagCeca: (để trống)                                       │
└─────────────────────────────────────────────────────────────┘
```

**Chi tiết từng field:**

| Field | Giá trị | Mô tả |
|-------|---------|-------|
| **Document Type ID** | `6` hoặc `11` | ⚠️ **BẮT BUỘC** - Lấy từ VNPT portal |
| **Contract Flow ID** | `1` | 1 = Hợp đồng cơ bản |
| **Permission To View** | `1` | 1 = Tất cả, 2 = Nội bộ |
| **Document Group** | `1` | 1 = Tài liệu có ký |
| **flagCeca** | Để trống hoặc `0` | 0/1 nếu có chứng thực CECA |

### 2.4. Tab "Signers" - Danh sách người ký (TÙY CHỌN)

Nếu bạn muốn tự động thêm người ký mặc định:

```
┌─────────────────────────────────────────────────────────────────────────┐
│ ────────── Signers ─────────────────────────────────────────────────   │
│                                                                         │
│ [Add Row]                                                              │
│                                                                         │
│ ┌─────┬─────────────┬──────────────────┬──────────┬────────┬─────────┐│
│ │ Thứ │ Tên         │ Email            │ SĐT      │ Phương │ Hình    ││
│ │ tự  │ người ký    │                  │          │ thức   │ thức ký ││
│ │     │             │                  │          │ xác    │         ││
│ │     │             │                  │          │ thực   │         ││
│ ├─────┼─────────────┼──────────────────┼──────────┼────────┼─────────┤│
│ │  1  │ Nguyễn VănA │ nvana@company.vn │ 09xxxxx  │   4    │ 3,4,1   ││
│ │  2  │ Đối tác XYZ │ doitac@xyz.com   │          │   4    │ 6       ││
│ └─────┴─────────────┴──────────────────┴──────────┴────────┴─────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```

**Ý nghĩa các trường:**

| Field | Giá trị ví dụ | Mô tả |
|-------|---------------|-------|
| **Thứ tự ký** | 1, 2, 3... | Thứ tự ký hợp đồng |
| **Tên người ký** | Nguyễn Văn A | Tên hiển thị |
| **Email** | nvana@company.vn | Email nhận thông báo |
| **SĐT** | 0912345678 | Số điện thoại (tùy chọn) |
| **Phương thức xác thực** | 4 | 1=None, 2=SMS, 3=Email, 4=Mã xác thực |
| **Hình thức ký** | 3,4,1 | 1=Ảnh, 2=SMS OTP, 3=Email OTP, 4=SmartCA, 5=VNPT ký số, 6=USB Token |

**Lưu ý:** 
- Nếu **không** thêm Signers ở đây, bạn sẽ phải thêm thủ công sau khi tạo hợp đồng trên VNPT Portal
- Thêm Signers ở đây để **tự động gửi** email đến người ký

### 2.5. Tab "Webhook" - Callback (TÙY CHỌN)

Để nhận thông báo khi trạng thái hợp đồng thay đổi:

```
┌─────────────────────────────────────────────────────────────┐
│ ────────── Webhook (optional) ──────────────────────────    │
│                                                             │
│ ☐ Require Bearer Token                                     │
│ Webhook Secret: (để trống)                                 │
│                                                             │
│ X-APP-CB-KEY: your_secret_key                              │
│ X-APP-CB-SECRET: ••••••••                                  │
└─────────────────────────────────────────────────────────────┘
```

**Lưu ý:** 
- Webhook URL sẽ là: `https://your-domain.com/api/method/mbwnext_econtract_service.api.api.vnpt_webhook_document_status`
- Cần cung cấp URL này cho VNPT để họ cấu hình callback

### 2.6. Lưu Settings

Click **Save** → Hệ thống sẽ:
- ✅ Validate thông tin
- ✅ Test connection (tự động login lần đầu)
- ✅ Lưu access token

---

## ✅ BƯỚC 3: Tạo Contract để Test

### 3.1. Tạo Contract mới

```
Menu → Contract → New Contract
```

**Điền thông tin tối thiểu:**

```
┌─────────────────────────────────────────────────────────────┐
│ Contract                                            [Save]  │
├─────────────────────────────────────────────────────────────┤
│ Party Name: Công ty ABC                                     │
│ Start Date: 21/01/2026                                      │
│ End Date: 21/01/2027                                        │
│ Contract Terms: Điều khoản hợp đồng test...                │
│                                                             │
│ ... (điền các field khác nếu cần)                          │
└─────────────────────────────────────────────────────────────┘
```

### 3.2. Submit Contract

Click **Submit** để chuyển trạng thái sang "Submitted"

⚠️ **Quan trọng:** Chỉ Contract đã Submit mới có thể gửi lên VNPT!

---

## ✅ BƯỚC 4: Gửi Contract lên VNPT eContract

### 4.1. Gửi qua UI

Sau khi Submit, click menu:

```
[VNPT eContract] ▼
  └─ Send to VNPT eContract
```

**Hệ thống sẽ:**
1. ✅ Render Contract thành PDF
2. ✅ Upload PDF lên VNPT
3. ✅ Tạo hợp đồng trên VNPT
4. ✅ Cập nhật người ký (nếu có cấu hình Signers)
5. ✅ Cập nhật vị trí ký (nếu có cấu hình)
6. ✅ Gửi hợp đồng → Người ký nhận email

### 4.2. Kiểm tra kết quả

Sau khi gửi thành công, bạn sẽ thấy:

```
┌─────────────────────────────────────────────────────────────┐
│ ✓ Sent. Envelope: VNPT-ENV-001                             │
│   Document ID: 123456                                       │
└─────────────────────────────────────────────────────────────┘
```

Contract sẽ có thêm thông tin:

```
┌─────────────────────────────────────────────────────────────┐
│ ────────── VNPT eContract ──────────────────────────────    │
│                                                             │
│ VNPT eContract Envelope: VNPT-ENV-001                      │
│ VNPT Contract ID: 123456                                    │
│ VNPT Status: Chờ ký                                         │
│ VNPT Last Sync: 21/01/2026 10:30:45                        │
└─────────────────────────────────────────────────────────────┘
```

### 4.3. Gửi qua Code (tùy chọn)

```python
# Trong Python Console
frappe.call(
    method="mbwnext_econtract_service.api.api.send_contract_to_vnpt",
    args={"contract_name": "CONTRACT-001"}
)
```

---

## ✅ BƯỚC 5: Kiểm tra trên VNPT Portal

### 5.1. Login vào VNPT Portal

```
URL: https://econtract-v2-poc.vnpt.vn/
Username: admincompany (tài khoản đã cấu hình)
Password: YourPassword@123
```

### 5.2. Kiểm tra hợp đồng

```
Menu → Hợp đồng → Hợp đồng đã tạo
```

Bạn sẽ thấy hợp đồng vừa gửi lên với:
- ✅ Tên hợp đồng: "Hợp đồng CONTRACT-001"
- ✅ Trạng thái: "Chờ ký" (nếu có Signers) hoặc "Bản nháp" (nếu chưa có)
- ✅ File PDF đính kèm

### 5.3. Thêm người ký (nếu chưa cấu hình Signers)

```
1. Click vào hợp đồng
2. Click [Thêm người ký]
3. Điền thông tin:
   - Email: doitac@company.com
   - Tên: Nguyễn Văn B
   - Phương thức: SmartCA / USB Token / Email OTP
4. Click [Gửi ký]
```

---

## ✅ BƯỚC 6: Đồng bộ trạng thái về ERPNext

### 6.1. Sync thủ công

Vào Contract → Menu:

```
[VNPT eContract] ▼
  └─ Sync Status
```

### 6.2. Sync tự động

Hệ thống tự động sync mỗi giờ qua Scheduler:
- Job: `sync_pending_envelopes`
- Chạy: Hourly
- Sync các hợp đồng đang "Chờ ký", "Thẩm định", v.v.

### 6.3. Xem log Scheduler

```bash
# SSH vào server
tail -f ~/frappe-bench/logs/schedule.log | grep "VNPT"
```

---

## ✅ BƯỚC 7: Test Workflow đầy đủ

### Kịch bản test hoàn chỉnh:

```
1. Cấu hình Settings với Signers
   └─ Signer 1: Nội bộ (Email OTP)
   └─ Signer 2: Đối tác (USB Token)

2. Tạo Contract → Submit

3. Gửi lên VNPT
   └─ Kết quả: Status = "Chờ ký"

4. Signer 1 nhận email → Ký qua Email OTP
   └─ Login VNPT Portal
   └─ Nhập OTP từ email
   └─ Ký thành công

5. Signer 2 nhận email → Ký qua USB Token
   └─ Login VNPT Portal
   └─ Cắm USB Token
   └─ Ký thành công

6. VNPT callback về ERPNext
   └─ Status tự động update = "Hoàn thành"

7. File đã ký tự động download (nếu bật auto_download_signed)
   └─ File lưu trong Contract attachments
```

---

## 🔍 Troubleshooting

### ❌ Lỗi 1: "VNPT eContract Settings is disabled"

**Nguyên nhân:** Chưa tick "Enabled"  
**Giải pháp:** Vào Settings → Tick ☑ Enabled → Save

---

### ❌ Lỗi 2: "Unauthorized (token invalid/expired)"

**Nguyên nhân:** 
- Username/Password sai
- Token hết hạn

**Giải pháp:**
```python
# Force refresh token
settings = frappe.get_single("VNPT eContract Settings")
settings.db_set("access_token", None)
settings.db_set("token_updated_on", None)
frappe.db.commit()

# Login lại sẽ tự động lấy token mới
```

---

### ❌ Lỗi 3: "Contract must be submitted"

**Nguyên nhân:** Contract chưa Submit  
**Giải pháp:** Vào Contract → Click [Submit]

---

### ❌ Lỗi 4: "Upload failed"

**Nguyên nhân:** 
- File PDF quá lớn (>10MB)
- Network timeout

**Giải pháp:**
- Tối ưu Print Format để giảm dung lượng PDF
- Tăng timeout trong code (mặc định 90s)

---

### ❌ Lỗi 5: "Document Type ID not found"

**Nguyên nhân:** `loai_tl_id` không tồn tại trên VNPT  
**Giải pháp:** 
- Login VNPT Portal
- Vào Danh mục → Loại tài liệu
- Ghi nhớ ID đúng và cập nhật lại Settings

---

### ❌ Lỗi 6: Không nhận callback từ VNPT

**Nguyên nhân:**
- Webhook URL chưa cấu hình trên VNPT
- X-APP-CB-KEY/SECRET sai

**Giải pháp:**
1. Kiểm tra callback URL:
   ```
   https://your-domain.com/api/method/mbwnext_econtract_service.api.api.vnpt_webhook_document_status
   ```
2. Cung cấp URL + X-APP-CB-KEY/SECRET cho VNPT
3. Test callback thủ công:
   ```bash
   curl -X POST https://your-domain.com/api/method/mbwnext_econtract_service.api.api.vnpt_webhook_document_status \
     -H "Content-Type: application/json" \
     -H "X-APP-CB-KEY: your_key" \
     -H "X-APP-CB-SECRET: your_secret" \
     -d '{"hopDongId": 123, "trangThai": 9}'
   ```

---

## 📊 Checklist hoàn chỉnh

### Trước khi test:

- [ ] Có thông tin đăng nhập VNPT (username, password)
- [ ] Biết Document Type ID của loại hợp đồng
- [ ] Đã cài app mbwnext_econtract_service
- [ ] Đã chạy `bench migrate`

### Cấu hình Settings:

- [ ] ☑ Enabled
- [ ] Base URL đúng (POC hoặc Production)
- [ ] Account và Password đúng
- [ ] Document Type ID đúng
- [ ] (Tùy chọn) Thêm Signers
- [ ] (Tùy chọn) Cấu hình Webhook
- [ ] Click Save thành công

### Test gửi hợp đồng:

- [ ] Tạo Contract
- [ ] Submit Contract
- [ ] Click "Send to VNPT eContract"
- [ ] Nhận thông báo thành công
- [ ] Contract có VNPT Contract ID
- [ ] Status = "Chờ ký" hoặc "Bản nháp"

### Kiểm tra trên VNPT Portal:

- [ ] Login thành công
- [ ] Thấy hợp đồng vừa tạo
- [ ] File PDF hiển thị đúng
- [ ] (Nếu có Signers) Email đã gửi đến người ký

### Test đầy đủ:

- [ ] Người ký nhận email
- [ ] Người ký login VNPT Portal
- [ ] Người ký ký thành công
- [ ] Callback về ERPNext (Status update)
- [ ] File đã ký download tự động

---

## 🎯 Cấu hình Tối thiểu để Test

Nếu chỉ muốn test nhanh, chỉ cần:

### Settings tối thiểu:

```yaml
✅ Enabled: Yes
✅ Base URL: https://gateway-bus-econtract-v2-poc.vnpt.vn/
✅ Account: admincompany
✅ Password: YourPassword@123
✅ Document Type ID: 6
✅ Contract Flow ID: 1
✅ Platform ID: 4
```

### Các field khác để mặc định:

```yaml
Auto Download: No (tắt)
Client ID: (để trống)
Client Secret: (để trống)
Customer ID: (để trống)
Signers: (để trống - thêm thủ công sau)
Webhook: (để trống)
```

**→ Với cấu hình này, bạn đã có thể gửi hợp đồng lên VNPT!**

---

## 📚 Tài liệu tham khảo

- API Documentation: `/home/mbw12345/test_core/API_VNPT_eContract_VNPT .pdf`
- Features Guide: `VNPT_ECONTRACT_V2_FEATURES.md`
- Code: `mbwnext_econtract_service/api/api.py`

---

## 🆘 Cần hỗ trợ?

1. **Xem Error Log:**
   ```
   Menu → Error Log → Filter: "VNPT"
   ```

2. **Xem Scheduler Log:**
   ```bash
   tail -f ~/frappe-bench/logs/schedule.log
   ```

3. **Test API trực tiếp:**
   ```python
   # Console
   settings = frappe.get_single("VNPT eContract Settings")
   from mbwnext_econtract_service.integrations.vnpt_econtract_v2.client import VNPTV2Client
   client = VNPTV2Client.from_settings(settings)
   
   # Test login
   token = client.login_and_store_token()
   print(f"Token: {token[:50]}...")
   ```

---

**✅ HOÀN THÀNH! Giờ bạn có thể bắt đầu test gửi hợp đồng sang VNPT eContract!** 🎉
