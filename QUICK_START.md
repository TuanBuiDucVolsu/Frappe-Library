# ⚡ VNPT eContract - Quick Start Guide

## 🎯 Cấu hình 5 phút

### 1️⃣ Vào Settings

```
Menu → Setup → VNPT eContract Settings
```

### 2️⃣ Điền thông tin tối thiểu

```yaml
☑ Enabled: Yes
Base URL: https://gateway-bus-econtract-v2-poc.vnpt.vn/
Account: your_username
Password: ••••••••
Document Type ID: 6
```

### 3️⃣ Save

Click **[Save]** → Hệ thống tự động login và lưu token

---

## 🚀 Gửi hợp đồng đầu tiên

### 1️⃣ Tạo Contract

```
Menu → Contract → New
- Điền thông tin cơ bản
- Click [Submit]
```

### 2️⃣ Gửi lên VNPT

```
[VNPT eContract] ▼ → Send to VNPT eContract
```

### 3️⃣ Kiểm tra

```
✓ VNPT Contract ID: 123456
✓ VNPT Status: Chờ ký
```

**DONE!** ✅

---

## 📋 Bảng giá trị các trường quan trọng

### Platform ID (nen_tang_id)
```
1 = Web
2 = Android
3 = iOS
4 = SDK Web ← Khuyến nghị
5 = SDK Android
6 = SDK iOS
```

### Contract Flow ID (loai_luong_hd_id)
```
1 = Hợp đồng cơ bản ← Khuyến nghị
2 = Hợp đồng nâng cao
```

### Permission To View (quyen_xem_id)
```
1 = Tất cả ← Khuyến nghị
2 = Nội bộ
```

### Document Group (nhom_tai_lieu_id)
```
1 = Tài liệu có ký ← Khuyến nghị
2 = Tài liệu không ký
```

### Phương thức xác thực (pt_xac_thuc_id)
```
1 = Không xác thực
2 = SMS OTP
3 = Email OTP
4 = Mã xác thực ← Khuyến nghị cho đối tác
7 = Email OTP + eKYC
8 = SMS OTP + eKYC
```

### Hình thức ký (hinh_thuc_ky)
```
1 = Ký ảnh
2 = SMS OTP
3 = Email OTP ← Phổ biến
4 = SmartCA ← Khuyến nghị
5 = VNPT ký số
6 = USB Token
7 = Email OTP + eKYC
8 = SMS OTP + eKYC

Ví dụ kết hợp: "3,4,1" = Cho phép Email OTP, SmartCA, hoặc Ký ảnh
```

---

## 🔧 Troubleshooting nhanh

### ❌ "Settings is disabled"
```
→ Tick ☑ Enabled → Save
```

### ❌ "Unauthorized"
```python
# Console: Force refresh token
settings = frappe.get_single("VNPT eContract Settings")
settings.db_set("access_token", None)
frappe.db.commit()
```

### ❌ "Contract must be submitted"
```
→ Vào Contract → Click [Submit]
```

### ❌ "Document Type ID not found"
```
→ Login VNPT Portal → Danh mục → Loại tài liệu
→ Ghi nhớ ID đúng → Update Settings
```

---

## 📞 Môi trường

### POC (Test):
```
API: https://gateway-bus-econtract-v2-poc.vnpt.vn/
Portal: https://econtract-v2-poc.vnpt.vn/
```

### Production:
```
API: https://gateway-bus-econtract.vnpt.vn/
Portal: https://econtract-v2.vnpt.vn/
```

---

## 🎓 Hướng dẫn chi tiết

Xem file `SETUP_GUIDE.md` để có hướng dẫn đầy đủ từng bước!

---

**Happy Signing! 🎉**


# 📝 VNPT eContract Settings - Template Cấu hình

## Copy template này và điền thông tin của bạn

---

## ✅ Section 1: CONNECTION (Bắt buộc)

```
┌─────────────────────────────────────────────────────────┐
│ Enabled: [☑]                                            │
│ Auto Download Signed: [☑] hoặc [☐]                      │
└─────────────────────────────────────────────────────────┘
```

### Thông tin kết nối:

```yaml
# ─────────── CONNECTION ───────────
Base URL: https://gateway-bus-econtract-v2-poc.vnpt.vn/
          # ⚠️ Phải có dấu "/" ở cuối!
          # POC: gateway-bus-econtract-v2-poc.vnpt.vn
          # PROD: gateway-bus-econtract.vnpt.vn

Account: _______________
         # Tài khoản VNPT cung cấp
         # Ví dụ: admincompany, admin_abc

Password: _______________
          # Mật khẩu đăng nhập VNPT Portal

Client ID: _______________
           # (Tùy chọn - nếu VNPT cung cấp)
           # Để trống nếu không có

Client Secret: _______________
               # (Tùy chọn - nếu VNPT cung cấp)
               # Để trống nếu không có

# ─────────── COLUMN BREAK ───────────

Default Print Format: _______________
                      # (Tùy chọn) Chọn format in Contract
                      # Để trống = dùng Standard

Subject Prefix: Hợp đồng
                # Tiền tố cho tên hợp đồng trên VNPT
                # Kết quả: "Hợp đồng CONTRACT-001"

Customer ID: _______________
             # (Tùy chọn - nếu VNPT cung cấp)
             # Để trống nếu không có

Platform ID: 4
             # 1=Web, 2=Android, 3=iOS
             # 4=SDK Web (khuyến nghị)
             # 5=SDK Android, 6=SDK iOS
```

---

## ✅ Section 2: CONTRACT DEFAULTS (Bắt buộc)

```yaml
# ─────────── CONTRACT DEFAULTS ───────────

Document Type ID: _______________
                  # ⚠️ BẮT BUỘC!
                  # Lấy từ VNPT Portal → Danh mục → Loại tài liệu
                  # Ví dụ:
                  #   6 = Hợp đồng mua bán
                  #  11 = Hợp đồng lao động
                  #  33 = Hợp đồng dịch vụ

Contract Flow ID: 1
                  # 1 = Hợp đồng cơ bản (khuyến nghị)
                  # 2 = Hợp đồng nâng cao

Permission To View: 1
                    # 1 = Tất cả mọi người (khuyến nghị)
                    # 2 = Chỉ nội bộ

# ─────────── COLUMN BREAK ───────────

Document Group: 1
                # 1 = Tài liệu có ký (khuyến nghị)
                # 2 = Tài liệu không ký

flagCeca: _______________
          # (Tùy chọn) 0 hoặc 1
          # Để trống = không dùng CECA
```

---

## ✅ Section 3: SIGNERS (Tùy chọn)

### ⚠️ Nếu KHÔNG thêm Signers:
- Hợp đồng sẽ ở trạng thái "Bản nháp" trên VNPT
- Bạn phải thêm người ký thủ công trên VNPT Portal

### ✅ Nếu THÊM Signers:
- Hợp đồng tự động gửi đến người ký
- Email thông báo tự động gửi

### Template Signer (copy từng row):

```yaml
# ─────── Signer 1 (Nội bộ) ───────
Thứ tự ký: 1
Tên người ký: _______________
Email: _______________@company.vn
SĐT: 09_______________
Phương thức xác thực: 4  # Mã xác thực
Hình thức ký: 3,4,1  # Email OTP, SmartCA, Ký ảnh
Vai trò ký: 2  # 1=Ký nháy, 2=Ký duyệt, 3=Ký dấu
Sequential Signing: ☑
Allow Delegate: ☐
Allow Add Signer: ☐
Message: (để trống)
Page: (để trống nếu không cần vị trí cố định)
X, Y, W, H: (để trống)

# ─────── Signer 2 (Đối tác) ───────
Thứ tự ký: 2
Tên người ký: _______________
Email: _______________@partner.com
SĐT: (để trống nếu dùng USB Token)
Phương thức xác thực: 4  # Mã xác thực
Hình thức ký: 6  # USB Token
               # hoặc: 6,4,5  # USB Token + SmartCA + VNPT ký số
Vai trò ký: 0  # Không xác định
Sequential Signing: ☑
Allow Delegate: ☐
Allow Add Signer: ☐
Message: (để trống)
Page: (để trống)
X, Y, W, H: (để trống)
```

---

## ✅ Section 4: WEBHOOK (Tùy chọn)

### Nếu cần nhận callback từ VNPT:

```yaml
# ─────────── WEBHOOK ───────────

Require Bearer Token: ☐  # Tắt cho đơn giản

Webhook Secret: (để trống)

X-APP-CB-KEY: _______________
              # Tự tạo key bất kỳ
              # Ví dụ: "my_secret_key_123"

X-APP-CB-SECRET: _______________
                 # Tự tạo secret bất kỳ
                 # Ví dụ: "my_secret_abc_xyz"
```

### URL Callback cung cấp cho VNPT:

```
https://your-erpnext-domain.com/api/method/mbwnext_econtract_service.api.api.vnpt_webhook_document_status
```

**Ví dụ:**
```
https://erp.mycompany.vn/api/method/mbwnext_econtract_service.api.api.vnpt_webhook_document_status
```

---

## 📝 Ví dụ Cấu hình Hoàn chỉnh (POC Test)

```yaml
# ========== BASIC INFO ==========
Enabled: ☑
Auto Download Signed: ☑

# ========== CONNECTION ==========
Base URL: https://gateway-bus-econtract-v2-poc.vnpt.vn/
Account: admincompany
Password: YourPassword@123
Client ID: (để trống)
Client Secret: (để trống)

Default Print Format: (để trống)
Subject Prefix: Hợp đồng
Customer ID: (để trống)
Platform ID: 4

# ========== CONTRACT DEFAULTS ==========
Document Type ID: 6
Contract Flow ID: 1
Permission To View: 1
Document Group: 1
flagCeca: (để trống)

# ========== SIGNERS (Ví dụ) ==========
[Signer 1]
  Thứ tự: 1
  Tên: Giám đốc A
  Email: giamdoc@mycompany.vn
  SĐT: 0912345678
  Phương thức: 4
  Hình thức ký: 3,4,1
  
[Signer 2]
  Thứ tự: 2
  Tên: Đối tác B
  Email: doitac@partner.com
  SĐT: (trống)
  Phương thức: 4
  Hình thức ký: 6

# ========== WEBHOOK ==========
X-APP-CB-KEY: my_test_key_2026
X-APP-CB-SECRET: my_test_secret_abc123xyz
```

---

## 🎯 Cấu hình Tối thiểu (Chỉ 4 fields bắt buộc)

```yaml
✅ Enabled: Yes
✅ Base URL: https://gateway-bus-econtract-v2-poc.vnpt.vn/
✅ Account: your_username
✅ Password: your_password
✅ Document Type ID: 6
```

**→ Đủ để gửi hợp đồng!** (Signers thêm sau trên VNPT Portal)

---

## 🧪 Test Checklist

- [ ] Settings đã Save thành công
- [ ] Field "Access Token" có giá trị (tự động)
- [ ] Field "Token Updated On" có giá trị (tự động)
- [ ] Tạo Contract → Submit
- [ ] Click "Send to VNPT eContract"
- [ ] Nhận message "Sent. Envelope: ..."
- [ ] Contract có "VNPT Contract ID"
- [ ] Login VNPT Portal → Thấy hợp đồng

---

## 🆘 Lỗi thường gặp

### "VNPT eContract Settings is disabled"
```
→ Tick ☑ Enabled
```

### "base_url is required"
```
→ Điền Base URL với dấu "/" cuối
```

### "loai_tl_id is required"
```
→ Điền Document Type ID (VD: 6)
```

### "Contract must be submitted"
```
→ Submit Contract trước khi gửi
```

### "Unauthorized"
```
→ Kiểm tra Account/Password
→ Hoặc xóa token để login lại
```

---

## 📚 Các file tài liệu

- **QUICK_START.md** (file này) - Bắt đầu nhanh
- **SETUP_GUIDE.md** - Hướng dẫn chi tiết
- **VNPT_ECONTRACT_V2_FEATURES.md** - Tài liệu đầy đủ

---

**🚀 Chúc bạn test thành công!**

