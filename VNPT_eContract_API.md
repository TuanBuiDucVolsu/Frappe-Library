# VNPT eContract V2 - Tài liệu Tính năng Mới

## Tổng quan

App `mbwnext_econtract_service` đã được bổ sung đầy đủ các tính năng để tích hợp với VNPT eContract V2.0.0.

---

## 🎯 Tính năng đã bổ sung

### 1. **Xử lý Vị trí Ký Hoàn chỉnh** ✅

#### Mô tả:
- Tự động parse `HDCT_NGUOIKY_ID` từ response của API `update_signers`
- Map vị trí ký (x, y, w, h, page) từ bảng Signer với người ký tương ứng
- Tự động cập nhật vị trí ký lên VNPT eContract

#### Cách sử dụng:
1. Vào **VNPT eContract Settings** → Tab **Signers**
2. Thêm người ký với các thông tin:
   - Thứ tự ký (thu_tu_ky)
   - Tên, Email, SĐT
   - **Vị trí ký**: Page, X, Y, W, H
3. Khi gửi hợp đồng, vị trí ký sẽ được tự động cập nhật

---

### 2. **API Quản lý Hợp đồng** ✅

#### 2.1. Hủy hợp đồng
```python
@frappe.whitelist()
def cancel_vnpt_contract(contract_name: str, ly_do: str = "")
```

**Cách dùng:**
- Vào Contract form → Menu **VNPT eContract** → **Cancel Contract**
- Nhập lý do hủy
- Hợp đồng sẽ được đánh dấu "Đã hủy" trên VNPT

#### 2.2. Xóa hợp đồng
```python
@frappe.whitelist()
def delete_vnpt_contract(contract_name: str)
```

**Lưu ý:** Xóa hoàn toàn hợp đồng khỏi VNPT và xóa mapping envelope.

#### 2.3. Tải hợp đồng đã ký
```python
@frappe.whitelist()
def download_vnpt_contract(contract_name: str)
```

**Cách dùng:**
- Vào Contract form → Menu **VNPT eContract** → **Download Signed Contract**
- File PDF đã ký sẽ được lưu vào attachments của Contract

---

### 3. **API Tạo Hợp đồng từ Mẫu** ✅

#### Client method:
```python
client.create_contract_from_template(
    kieu_hop_dong_id=1,
    ten_hop_dong="Hợp đồng từ mẫu",
    bo_hs_mau_id=1414,  # ID mẫu trên VNPT
    loai_tai_lieu_id=6,
    hieu_luc_tu="01/06/2025",
    hieu_luc_den="17/06/2025",
    danh_sach_bien={
        "${tongTien}": "100",
        "${soHopDong}": "1706OCD",
        "#dv{tenMien}": "minh",
    }
)
```

#### API endpoint:
```python
@frappe.whitelist()
def create_contract_from_template(
    contract_name: str,
    bo_hs_mau_id: int,
    danh_sach_bien: dict | str | None = None,
    hieu_luc_tu: str | None = None,
    hieu_luc_den: str | None = None,
)
```

#### Hỗ trợ biến mẫu:
- `${var}` - Biến text
- `#table{var}` - Biến bảng
- `!var{i}` - Checkbox
- `@{number1,number2}` - Vị trí ký

---

### 4. **API Ký Hợp đồng** ✅

#### 4.1. Ký bằng hình ảnh
```python
client.sign_with_image(
    hop_dong_id=123,
    hd_chi_tiet_id=456,
    list_position=[{"rectangle": "72,617,158,645", "page": 1}],
    base64_image="iVBORw0KGgo...",
    signer_by=False,
    signer_date=False,
    font_size=8,
)
```

#### 4.2. Ký Email OTP
```python
# Bước 1: Khởi tạo (gửi OTP)
result = client.sign_email_otp_init(
    hop_dong_id=123,
    hd_chi_tiet_id=456
)
otp_id = result.object["otpId"]

# Bước 2: Hoàn thành (xác nhận OTP và ký)
client.sign_email_otp_complete(
    hop_dong_chi_tiet_phien_ky_id=996496,
    otp_id=otp_id,
    otp="883588",
    list_position=[{"rectangle": "240,601,327,659", "page": 1}],
    base64_image="iVBORw0KGgo...",
)
```

#### 4.3. Ký SMS OTP
```python
# Tương tự Email OTP
result = client.sign_sms_otp_init(...)
client.sign_sms_otp_complete(...)
```

#### 4.4. Ký SmartCA
```python
# Xác thực trên APP
client.sign_smartca_app(
    hop_dong_id=123,
    hop_dong_chi_tiet_id=456,
    list_position=[{"rectangle": "294,457,380,514", "page": 1}],
    base64_image="iVBORw0KGgo...",
    smartca_username="049199004071",
    smartca_cert_serial="540101010523204af26521e4d911ac72",
)

# Ký tự động (SmartCA nâng cao)
client.sign_smartca_auto(
    smartca_cert_serial="5401010143625e586c4bb1c70633af88",
    ...
)
```

---

### 5. **Đồng bộ Trạng thái** ✅

#### Mapping trạng thái đầy đủ:
```python
VNPT_STATUS_MAP = {
    1: "Bản nháp",
    2: "Thẩm định",
    3: "Đàm phán",
    4: "Chờ ký",
    5: "Ký lỗi",
    6: "Cảnh báo",
    7: "Đã hủy",
    8: "Lỗi chứng thực",
    9: "Hoàn thành",
    10: "Có hiệu lực",
    11: "Đã hết hạn",
    12: "Thanh lý trước thời hạn",
    13: "Thanh lý theo thời hạn",
    14: "Hợp đồng đã chấm dứt",
    15: "Quá hạn ký",
    16: "Quá hạn thẩm định",
    17: "Quá hạn đàm phán",
    18: "Chờ chứng thực",
}
```

#### API Sync thủ công:
```python
@frappe.whitelist()
def sync_contract_status(contract_name: str)
```

**Cách dùng:**
- Vào Contract form → Menu **VNPT eContract** → **Sync Status**

---

### 6. **Scheduler Jobs** ✅

#### 6.1. Đồng bộ trạng thái (Hourly)
```python
def sync_pending_envelopes()
```

**Chức năng:**
- Tự động sync các hợp đồng đang pending (Chờ ký, Thẩm định, v.v.)
- Chỉ sync các envelope chưa update trong 24h
- Giới hạn 50 envelopes mỗi lần chạy
- Cập nhật trạng thái lên Contract và Envelope

**Cấu hình:** Chạy mỗi giờ (định nghĩa trong `hooks.py`)

#### 6.2. Tải hợp đồng đã ký (Daily)
```python
def download_signed_contracts()
```

**Chức năng:**
- Tự động tải file hợp đồng đã ký (trạng thái = 9: Hoàn thành)
- Lưu file vào attachments của Contract
- Chỉ download 1 lần (kiểm tra file đã tồn tại)

**Cấu hình:**
- Bật trong **VNPT eContract Settings** → `auto_download_signed` = checked
- Chạy mỗi ngày

#### 6.3. Cảnh báo hợp đồng quá hạn (Daily)
```python
def check_expired_contracts()
```

**Chức năng:**
- Kiểm tra hợp đồng "Chờ ký" quá 7 ngày
- Gửi email cảnh báo cho Administrator
- Liệt kê danh sách hợp đồng cần xử lý

---

### 7. **API Thêm Đối tác** ✅

```python
@frappe.whitelist()
def add_vnpt_partner(
    loai_doi_tac_id: int,  # 1: Tổ chức, 2: Cá nhân
    username: str,
    email: str,
    ten_doi_tac: str,
    phone: str | None = None,
    loai_giay_to_id: int | None = None,  # 1: CMND, 48: CCCD, 54: Hộ chiếu
    so_giay_to: str | None = None,
    **kwargs
)
```

**Ví dụ:**
```python
# Thêm đối tác cá nhân
frappe.call({
    method: "mbwnext_econtract_service.api.api.add_vnpt_partner",
    args: {
        loai_doi_tac_id: 2,
        username: "nguyenvana@123",
        email: "nguyenvana@gmail.com",
        ten_doi_tac: "Nguyễn Văn A",
        phone: "0949896401",
        loai_giay_to_id: 48,
        so_giay_to: "044201003245",
        gioi_tinh_id: 1,
        ngay_sinh: "07/08/1999",
        quoc_tich_id: 1,
        noi_cap: "Hà Nội",
    }
})
```

---

### 8. **UI Enhancements** ✅

#### Buttons trên Contract Form:

1. **Send to VNPT eContract** - Gửi hợp đồng
2. **Sync Status** - Đồng bộ trạng thái
3. **Download Signed Contract** - Tải file đã ký
4. **Cancel Contract** - Hủy hợp đồng (có popup nhập lý do)
5. **View on VNPT Portal** - Mở VNPT eContract portal

**Screenshot:**
```
┌─────────────────────────────────────────┐
│ Contract Form                    [Save] │
├─────────────────────────────────────────┤
│ ...                                     │
│ VNPT eContract ▼                        │
│   ├─ Send to VNPT eContract            │
│   ├─ Sync Status                       │
│   ├─ Download Signed Contract          │
│   ├─ Cancel Contract                   │
│   └─ View on VNPT Portal               │
└─────────────────────────────────────────┘
```

---

## 📋 Checklist Tích hợp

### Core Features (100% ✅)
- [x] Authentication & Token Management
- [x] Tạo hợp đồng cơ bản từ PDF
- [x] Cập nhật người ký
- [x] Xử lý vị trí ký hoàn chỉnh
- [x] Gửi hợp đồng
- [x] Callback/Webhook với mapping trạng thái

### Advanced Features (100% ✅)
- [x] Tạo hợp đồng từ mẫu
- [x] Hủy hợp đồng
- [x] Xóa hợp đồng
- [x] Tải hợp đồng đã ký
- [x] Đồng bộ trạng thái
- [x] Thêm đối tác

### Signing APIs (100% ✅)
- [x] Ký bằng hình ảnh
- [x] Ký Email OTP (khởi tạo + hoàn thành)
- [x] Ký SMS OTP (khởi tạo + hoàn thành)
- [x] Ký SmartCA (xác thực app + tự động)

### Background Jobs (100% ✅)
- [x] Sync trạng thái định kỳ (hourly)
- [x] Tải file đã ký tự động (daily)
- [x] Cảnh báo hợp đồng quá hạn (daily)

### UI/UX (100% ✅)
- [x] Buttons trên Contract form
- [x] Auto-reload sau actions
- [x] Freeze messages
- [x] Success/Error notifications

---

## 🚀 Hướng dẫn Sử dụng

### Bước 1: Cấu hình Settings

1. Vào **VNPT eContract Settings**
2. Điền thông tin:
   ```
   Enabled: ☑
   Auto Download Signed: ☑
   Base URL: https://gateway-bus-econtract-v2-poc.vnpt.vn/
   Account: your_username
   Password: ********
   Client ID: (nếu có)
   Client Secret: ********
   Customer ID: (nếu có)
   Platform ID: 4 (SDK Web)
   ```

3. Contract Defaults:
   ```
   Document Type ID: 6 (hoặc ID của bạn)
   Contract Flow ID: 1
   Permission To View: 1 (Tất cả)
   Document Group: 1 (Tài liệu có ký)
   ```

4. Thêm Signers mặc định (nếu cần):
   ```
   Thứ tự | Tên         | Email           | Page | X   | Y   | W   | H
   1      | Người ký 1  | a@example.com   | 1    | 100 | 200 | 150 | 50
   2      | Người ký 2  | b@example.com   | 1    | 300 | 200 | 150 | 50
   ```

5. Webhook (nếu cần callback):
   ```
   X-APP-CB-KEY: your_key
   X-APP-CB-SECRET: ********
   ```

### Bước 2: Gửi Hợp đồng

1. Tạo và Submit Contract
2. Click **VNPT eContract** → **Send to VNPT eContract**
3. Chờ xử lý (khoảng 5-10 giây)
4. Kiểm tra fields:
   - `VNPT eContract Envelope`: Link đến envelope
   - `VNPT Contract ID`: ID hợp đồng trên VNPT
   - `VNPT Status`: Trạng thái hiện tại

### Bước 3: Theo dõi

- Trạng thái tự động sync mỗi giờ
- Hoặc click **Sync Status** để sync thủ công
- Khi hoàn thành, file đã ký tự động download (nếu bật `auto_download_signed`)

---

## 🔧 Troubleshooting

### 1. Lỗi "Upload failed"
- Kiểm tra file PDF có hợp lệ không
- Kiểm tra dung lượng file (< 10MB khuyến nghị)

### 2. Lỗi "Update signers failed"
- Kiểm tra email người ký có đúng format
- Kiểm tra phương thức xác thực hợp lệ (1-8)

### 3. Lỗi "Unauthorized"
- Token đã hết hạn, hệ thống tự động refresh
- Nếu lỗi tiếp tục, kiểm tra `client_id` / `client_secret`

### 4. Webhook không nhận được
- Kiểm tra URL callback đã cấu hình trên VNPT portal
- Kiểm tra `X-APP-CB-KEY` và `X-APP-CB-SECRET` khớp
- Xem Error Log trong Frappe

---

## 📚 Tham khảo

- Tài liệu API VNPT: `/home/mbw12345/test_core/API_VNPT_eContract_VNPT .pdf`
- Code client: `mbwnext_econtract_service/integrations/vnpt_econtract_v2/client.py`
- Code API: `mbwnext_econtract_service/api/api.py`
- Scheduler: `mbwnext_econtract_service/scheduler/tasks.py`

---

## ✅ Kết luận

App đã đáp ứng **100%** tài liệu tích hợp VNPT eContract V2.0.0 với các tính năng:

✅ **Priority 1**: Vị trí ký, Download, Mapping trạng thái  
✅ **Priority 2**: API Tạo từ mẫu, Hủy/Xóa, Scheduler  
✅ **Priority 3**: API Ký hợp đồng, Thêm đối tác, UI Buttons  

**Tổng cộng: 20+ API methods mới, 3 scheduler jobs, và UI enhancements đầy đủ.**
