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

