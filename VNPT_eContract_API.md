# VNPT eContract V2.0 (Gateway Bus) – Tóm tắt tích hợp

> Tóm tắt theo tài liệu “VNPT eContract – Tài liệu hướng dẫn tích hợp – V2.0.0 (VNPT IT3)”.

## Môi trường

- **POC**
  - **Domain API**: `https://gateway-bus-econtract-v2-poc.vnpt.vn/`
  - **Domain Web**: `https://econtract-v2-poc.vnpt.vn/`
- **PROD**
  - **Domain API**: `https://gateway-bus-econtract.vnpt.vn/`
  - **Domain Web**: `https://econtract-v2.vnpt.vn/`

## 1) Auth / Login

### 1.1 Login (tài khoản/mật khẩu)

- **POST** `/users-profile-service/auth/login`
- **Header**: `Content-Type: application/json`
- **Body**
  - `client_id` *(M)*: mã định danh ứng dụng tích hợp (có thể rỗng tuỳ môi trường)
  - `client_secret` *(M)*: bí mật ứng dụng (có thể rỗng tuỳ môi trường)
  - `khachhang_id` *(O)*
  - `taiKhoan` *(M)*
  - `matKhau` *(M)*
  - `nenTangId` *(M)*: `1 Web, 2 Android, 3 iOS, 4 SDK Web, 5 SDK Android, 6 SDK iOS`
- **Response**: `object.accessToken`, `object.expiresAt`

### 1.2 Login bằng mã truy cập

- **POST** `/users-profile-service/auth/login-ktk`

### 1.3 Login SSO

- **POST** `/users-profile-service/auth/sso`

## 2) Flow tạo hợp đồng cơ bản (ERP/ERPNext → VNPT eContract)

### Bước A — Upload file PDF

- **POST** `/econtract-saas-service/api/hopdong/upload-multi-files`
- **Header**
  - `Content-Type: multipart/form-data`
  - `Authorization: Bearer <JWT>`
- **Form (ví dụ 1 file)**
  - `rawData[0].orderNumber` *(int)*: số thứ tự
  - `rawData[0].hopDongType` *(int)*: `1` hợp đồng chính, `0` phụ lục/tài liệu đính kèm
  - `rawData[0].file` *(file)*: PDF
  - *(O)* `rawData[0].flagCeca`, `rawData[0].quyenXemId`, `rawData[0].nhomTaiLieuId`, `rawData[0].chiTietFileMauId`
- **Response**: `object[0].url`, `object[0].fileSize`, `object[0].fileName`

### Bước B — Tạo hợp đồng (cơ bản/nâng cao)

- **POST** `/econtract-saas-service/api/hopdong`
- **Header**: `Authorization: Bearer <JWT>`
- **Body (cơ bản)**
  - `p_ten_hd` *(M)*: tên hợp đồng
  - `p_loai_tl_id` *(M)*: ID loại tài liệu
  - `p_loai_luong_hd_id` *(M)*: `"1"` cơ bản, `"2"` nâng cao
  - `p_ghichu` *(O)*
  - `p_ct_hd` *(M)*: danh sách file
    - `p_file_ten`, `p_file_url`, `p_dungluong_file` *(M)*
    - `p_hd_chinh` *(M)*: `1/0`
    - `p_quyenxem_id` *(M)*: `1` tất cả, `2` nội bộ
    - `p_nhom_tailieu_id` *(M)*: `1` có ký, `2` không ký
- **Response**: `HOPDONG_ID` + danh sách `HD_CHITIET_ID`

### Bước C — Cập nhật người ký

- **POST** `/econtract-saas-service/api/hopdong/capnhat_nguoiky`
- **Header**: `Authorization: Bearer <JWT>`
- **Body**
  - `p_hd_chitiet_id` *(M)*: id file hợp đồng chi tiết
  - `p_all_file` *(M)*: `1/0`
  - `p_ct_nk` *(M)*: danh sách người ký (tên/email/sdt, phương thức xác thực, hình thức ký, thứ tự ký…)

### Bước D — Cập nhật vị trí ký (khi cần cấu hình tay)

- **POST** `/econtract-saas-service/api/hopdong/capnhat_chuky_vitri`

### Bước E — Gửi hợp đồng

- **POST** `/econtract-saas-service/api/hopdong/gui-hop-dong`
- **Body**: `{ "hopDongId": <id> }`

## 3) Tra cứu / tải hợp đồng

- **Chi tiết hợp đồng**
  - **GET** `/econtract-saas-service/api/hopdong/chitiet/{idHopDong}`
- **Tải bộ hợp đồng (zip)**
  - **GET** `/econtract-saas-service/api/hopdong/tai-bo-hopdong?hopDongId=<id>`
- **Tải 1 file trong bộ hợp đồng**
  - **GET** `/econtract-saas-service/api/hopdong/{hopDongId}/tai-file/{hopDongChiTietId}`

## 4) Callback trạng thái (VNPT → hệ thống tích hợp)

- **Method**: `POST`
- **Headers**
  - `Content-Type: application/json`
  - `X-APP-CB-KEY`
  - `X-APP-CB-SECRET`
- **Body tối thiểu**

```json
{
  "hopDongId": 123,
  "trangThai": 4,
  "danhSachNguoiKy": []
}
```

# Giải thích base tích hợp ERPNext `Contract` ↔ VNPT eContract V2

Tài liệu này mô tả các phần mình đã triển khai trong app `mbwnext_econtract_service` để tạo “base integration” giữa **ERPNext (DocType `Contract`)** và **VNPT eContract V2 (Gateway Bus)** theo tài liệu VNPT IT3 V2.0.0.

---

## 1) Mục tiêu & phạm vi

- **Mục tiêu**: từ `Contract` (ERPNext) có thể **đẩy PDF lên VNPT eContract**, **tạo hợp đồng**, **(tuỳ chọn) cấu hình người ký**, **gửi hợp đồng đi ký**, và **nhận callback trạng thái**.
- **Phạm vi base**: tập trung vào luồng “tạo hợp đồng cơ bản” và callback trạng thái.  
  Các chức năng ký sâu (SmartCA/OTP ký tự động, render từ template, auto-detect vị trí ký, download zip…) có thể bổ sung tiếp.

---

## 2) Vì sao phải bỏ “tài liệu cũ”

Bạn cung cấp 2 “hệ API” khác nhau:

- **API cũ**: dạng `/api/documents/...` (response `data/success/code/messages`, token 24h, webhook PascalCase `DocumentId`...).  
- **API V2.0 (Gateway Bus)**: dạng `users-profile-service/...` và `econtract-saas-service/...` (response `message/statusCode/status/object`, JWT thường ~1 giờ, callback header `X-APP-CB-KEY/SECRET`).

Base integration hiện tại đã được **chuyển sang V2** và mình đã **xoá hẳn** các phần theo tài liệu cũ để tránh nhầm lẫn.

---

## 3) Những thành phần đã tạo trong ERPNext/Frappe

### 3.1 DocType `VNPT eContract Settings` (Single)

**Mục đích**: nơi cấu hình kết nối và default tham số khi tạo hợp đồng từ ERPNext.

- **File**:
  - `mbwnext_econtract_service/doctype/vnpt_econtract_settings/vnpt_econtract_settings.json`
  - `mbwnext_econtract_service/doctype/vnpt_econtract_settings/vnpt_econtract_settings.py`

**Nhóm field chính**:

- **Connection / Auth**
  - `base_url`: ví dụ `https://gateway-bus-econtract-v2-poc.vnpt.vn/`
  - `tai_khoan`, `mat_khau`
  - `client_id`, `client_secret` (tuỳ môi trường/đối tác cấp)
  - `khachhang_id` (tuỳ môi trường)
  - `nen_tang_id` (mặc định `4` = SDK Web)
- **Contract Defaults**
  - `loai_tl_id`: `p_loai_tl_id` (bắt buộc)
  - `loai_luong_hd_id`: `p_loai_luong_hd_id` (mặc định `1` cơ bản)
  - `quyen_xem_id`, `nhom_tai_lieu_id`, `flag_ceca`: dùng khi upload/tạo chi tiết file
  - `default_print_format`: Print Format để render PDF từ `Contract`
  - `subject_prefix`: tiền tố tên hợp đồng hiển thị trên VNPT
- **Token (auto)**
  - `access_token`, `token_updated_on`: lưu token cache để tái sử dụng; client sẽ refresh trước khi hết hạn
- **Signers**
  - `signers` (table) để khai báo danh sách người ký mặc định
- **Callback**
  - `callback_key` (`X-APP-CB-KEY`)
  - `callback_secret` (`X-APP-CB-SECRET`)

### 3.2 DocType `VNPT eContract Envelope`

**Mục đích**: mapping nội bộ để ERPNext lưu quan hệ giữa `Contract` và hợp đồng VNPT.

- **File**:
  - `mbwnext_econtract_service/doctype/vnpt_econtract_envelope/vnpt_econtract_envelope.json`
  - `mbwnext_econtract_service/doctype/vnpt_econtract_envelope/vnpt_econtract_envelope.py`

**Field chính**:

- `contract`: Link đến ERPNext `Contract`
- `hop_dong_id`: ID hợp đồng bên VNPT (V2)
- `vnpt_status_value`, `vnpt_status_description`: trạng thái cập nhật từ callback/polling
- timestamps + `vnpt_reason`, `last_error`

> Lưu ý: các field “documentId/documentNo/downloadUrl…” của hệ cũ đã được loại bỏ khỏi envelope để tránh nhầm.

### 3.3 DocType table `VNPT eContract Signer`

**Mục đích**: cấu hình danh sách người ký mặc định trong settings.

- **File**:
  - `mbwnext_econtract_service/doctype/vnpt_econtract_signer/vnpt_econtract_signer.json`
  - `mbwnext_econtract_service/doctype/vnpt_econtract_signer/vnpt_econtract_signer.py`

**Field chính**:

- `thu_tu_ky`, `ten_tai_khoan`, `email`, `so_dt`
- `pt_xac_thuc_id` (2/3/4/7/8…)
- `hinh_thuc_ky` (`p_ht_ky`)
- cờ tuần tự/chuyển ký/thêm ký
- (đã chuẩn bị field) `page/x/y/w/h` nếu muốn nhập tay vị trí ký

---

## 4) Các custom field thêm vào ERPNext `Contract`

Để hiển thị mapping/trạng thái ngay trên `Contract`, mình thêm các custom field (tạo ở `after_install`).

- **File**: `mbwnext_econtract_service/install.py`

Các field chính:

- `vnpt_econtract_envelope` (Link -> `VNPT eContract Envelope`)
- `vnpt_econtract_hop_dong_id` (Int) — ID hợp đồng V2
- `vnpt_econtract_status`, `vnpt_econtract_last_sync`

> Nếu bạn muốn “clean UI” triệt để, có thể xoá các field không dùng nữa (mình chưa xoá tự động để tránh mất dữ liệu).

---

## 5) Luồng nghiệp vụ khi bấm “Send to VNPT eContract”

### 5.1 Nút trên form Contract (UI)

- **File**: `mbwnext_econtract_service/public/js/contract_vnpt_econtract.js`
- **Hook**: `mbwnext_econtract_service/hooks.py` khai báo `doctype_js` cho `Contract`

Điều kiện:

- chỉ hiển thị khi `Contract.docstatus == 1` (đã submit).

Khi bấm nút → gọi server method:

- `mbwnext_econtract_service.api.send_contract_to_vnpt(contract_name)`

### 5.2 Server method tạo & gửi hợp đồng (V2)

**File**: `mbwnext_econtract_service/api.py`

Luồng xử lý hiện tại:

1. **Render PDF từ Contract**
   - ERPNext dùng `Print Format` (nếu cấu hình `default_print_format`) để render PDF.
2. **Login lấy JWT**
   - client tự refresh token nếu hết hạn.
3. **Upload PDF**
   - gọi `POST /econtract-saas-service/api/hopdong/upload-multi-files`
   - lấy `url`, `fileSize`, `fileName` trả về.
4. **Tạo hợp đồng cơ bản**
   - gọi `POST /econtract-saas-service/api/hopdong` với `p_ct_hd` trỏ đến file đã upload.
   - lấy `HOPDONG_ID`.
5. **Tra cứu chi tiết hợp đồng**
   - gọi `GET /econtract-saas-service/api/hopdong/chitiet/{hopDongId}`
   - mục đích: lấy `HD_CHITIET_ID` của file “hợp đồng chính”.
6. **(Tuỳ chọn) Cập nhật người ký**
   - nếu trong Settings có `signers` thì gọi `POST /econtract-saas-service/api/hopdong/capnhat_nguoiky`.
7. **Gửi hợp đồng**
   - gọi `POST /econtract-saas-service/api/hopdong/gui-hop-dong`.
8. **Lưu mapping**
   - tạo/cập nhật `VNPT eContract Envelope` và ghi `hop_dong_id` vào `Contract`.

Phần “cập nhật vị trí ký” (`capnhat_chuky_vitri`) hiện để **mở rộng tiếp** vì cần map `HDCT_NGUOIKY_ID` cho từng signer (phải parse từ response update signer).

---

## 6) VNPT eContract V2 client (giao tiếp API)

**File**: `mbwnext_econtract_service/integrations/vnpt_econtract_v2/client.py`

Các điểm chính:

- **Response wrapper**: parse response V2 theo format `message/statusCode/status/object`
  - success thường là: `message="ECT-00000000"` và `statusCode=200`
- **Token cache**: lưu `access_token` + `token_updated_on` trong Settings
  - refresh sớm trước khi hết hạn (base hiện đặt ~50 phút vì ví dụ token ~1 giờ)
- **Các hàm chính đã implement**:
  - `login_and_store_token()`
  - `upload_single_pdf()`
  - `create_basic_contract()`
  - `contract_detail()`
  - `update_signers()`
  - `update_sign_positions()` (sẵn hàm, chưa dùng trong flow base)
  - `send_contract()`

---

## 7) Callback trạng thái (VNPT → ERPNext)

**Endpoint** (Frappe whitelisted):  
`POST /api/method/mbwnext_econtract_service.api.vnpt_webhook_document_status`

**Xác thực callback**:

- VNPT gửi header:
  - `X-APP-CB-KEY`
  - `X-APP-CB-SECRET`
- ERPNext so sánh với `callback_key` / `callback_secret` trong Settings.

**Payload** (tối thiểu theo doc):

- `hopDongId`
- `trangThai`
- (optional) `danhSachNguoiKy`

Khi nhận callback:

- tìm `VNPT eContract Envelope` theo `hop_dong_id`
- cập nhật trạng thái vào Envelope và đồng bộ field trạng thái vào `Contract`.

---

## 8) Polling job (fallback)

**File**: `mbwnext_econtract_service/tasks.py`  
**Hook**: `mbwnext_econtract_service/hooks.py` (hourly)

Mục đích:

- nếu callback chưa cấu hình/không ổn định, job sẽ chạy định kỳ để gọi `contract_detail(hop_dong_id)` và cập nhật `last_synced_at`.

---

## 9) Những file chính đã thay đổi/tạo mới

- `mbwnext_econtract_service/hooks.py`
- `mbwnext_econtract_service/api.py`
- `mbwnext_econtract_service/tasks.py`
- `mbwnext_econtract_service/install.py`
- `mbwnext_econtract_service/public/js/contract_vnpt_econtract.js`
- `mbwnext_econtract_service/integrations/vnpt_econtract_v2/client.py`
- `mbwnext_econtract_service/doctype/vnpt_econtract_settings/*`
- `mbwnext_econtract_service/doctype/vnpt_econtract_envelope/*`
- `mbwnext_econtract_service/doctype/vnpt_econtract_signer/*`
- `README.md`
- `VNPT_eContract_V2_API.md`

Các phần **đã xoá** (tài liệu cũ):

- `mbwnext_econtract_service/integrations/vnpt_econtract/*`
- `mbwnext_econtract_service/doctype/vnpt_econtract_process_step/*`

---

## 10) Cách vận hành nhanh (checklist)

1. Cài app + migrate + build:
   - `bench --site <site> install-app mbwnext_econtract_service`
   - `bench --site <site> migrate`
   - `bench build`
2. Cấu hình `VNPT eContract Settings`
   - base_url (gateway-bus-econtract-v2-...)
   - tài khoản/mật khẩu
   - `p_loai_tl_id`
   - callback key/secret (nếu dùng callback)
   - signers (nếu cần cấu hình người ký mặc định)
3. Submit `Contract` → bấm **Send to VNPT eContract**

