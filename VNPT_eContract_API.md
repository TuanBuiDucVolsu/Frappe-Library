# VNPT eContract API – Tóm tắt tích hợp (internal note)

> Tài liệu này được “gói lại” từ nội dung VNPT eContract Premium Docs mà bạn đã cung cấp (định dạng chung + các endpoint mẫu).  
> Mục tiêu: dùng làm **cheat-sheet** cho team khi tích hợp (ví dụ ERPNext/Frappe → VNPT eContract).

---

## 1) Quy ước chung

- **Tên field**: theo **camelCase** cho input/output (ví dụ: `userName`, `cusCode`, `dayOfBirth`…).
- **Response body**: các API trả về cấu trúc thống nhất:

```json
{
  "data": {},
  "success": true,
  "code": 0,
  "messages": []
}
```

### Ý nghĩa field trong response

- **`data`** *(object|array|string)*: dữ liệu trả về.
- **`success`** *(boolean)*:
  - `true`: request thành công
  - `false`: request thất bại
- **`code`** *(int)*:
  - `0`: mặc định (thường là thành công)
  - `100`: cần xác nhận OTP (flow 2 bước)
  - `401`: chưa xác thực (token sai/hết hạn)
  - Lưu ý: một số ví dụ có `code: 200` dù `success: true`.
- **`messages`** *(array[string])*: mô tả thông tin từ hệ thống / lỗi chi tiết.

> Lưu ý naming: phần mô tả có thể ghi `message` nhưng ví dụ dùng `messages`. Khi parse, nên hỗ trợ cả 2 nếu gặp môi trường khác nhau.

---

## 2) Xác thực (Token-based Authorization)

VNPT eContract dùng **Bearer access token** theo mô hình **Authorization Code** (định danh bằng `username`/`password` và chọn `companyId`).

### Header chuẩn

- `Content-Type: application/json; charset=utf-8`
- `Authorization: Bearer <access_token>`

### Thời hạn token

- **Token (Authorization code) hiệu lực 24 giờ** kể từ lúc response trả về (có thể cấu hình theo hệ thống tích hợp).
- Các API yêu cầu `Authorize: Bearer token` đều cần login trước.

### Mã HTTP thường gặp khi dùng token

- `200 OK`: thành công
- `401 Unauthorized`: token sai/hết hạn
- `502 Bad Gateway`: timeout

---

## 3) Webhook trạng thái chứng từ

### Mục đích

Đối tác dựng 1 REST API endpoint để VNPT eContract gọi sang khi trạng thái chứng từ thay đổi.

### Yêu cầu endpoint phía đối tác

- **URL**: do đối tác cung cấp (ví dụ `https://domain-partner.com/api/document/status`)
- **Method**: `POST`
- **Content-Type**: `application/json`
- **Authorization**: `Bearer token` *(tuỳ chọn – nếu đối tác muốn bảo mật)*

### Payload mẫu (đối tác nhận)

> Lưu ý: payload webhook dùng key kiểu PascalCase như `DocumentId`, `DocumentNo`…

```json
{
  "DocumentNo": "HD.001",
  "DocumentId": "fa07d61e-0fff-41a3-e618-08dad69d1b1b",
  "DocumentStatus": {
    "Value": -1,
    "Description": "Đã bị từ chối"
  },
  "Reason": "Mã chứng từ không đúng quy định",
  "DatetimeNow": "2024/11/06 14:56:50"
}
```

### Các field có thể có trong webhook

- **`DocumentId`** *(string)*: id chứng từ
- **`DocumentNo`** *(string)*: mã chứng từ
- **`DocumentStatus`** *(object)*: trạng thái (tham khảo định nghĩa trạng thái ở API danh sách chứng từ)
- **`Reason`** *(string)*: lý do hủy/từ chối
- **`BatchProcessId`** *(string)*: id lô xử lý (xử lý hàng loạt quy trình)
- **`BatchImportId`** *(int)*: id lô import (hoàn tất tạo lô chứng từ)
- **`DocumentIdInBatchImport`** *(Array[int])*: danh sách id chứng từ trong lô (hoàn tất tạo lô)
- **`DatetimeNow`** *(Datetime)*: thời điểm thực hiện

### Response đối tác trả về cho VNPT

```json
{
  "success": true,
  "messages": "Document status updated successfully."
}
```

---

## 4) Đăng nhập & xác thực

### 4.1) Đăng nhập (username/password) lấy token

- **URL**: `api/auth/password-login`
- **Method**: `POST`
- **Content-Type**: `application/json`
- **Body**:
  - `username` *(string, required)*
  - `password` *(string, required)*
  - `companyId` *(int, required)*

> Nếu chưa có `companyId`: gọi `password-login` **bỏ qua `companyId`** để lấy access token → gọi API lấy danh sách công ty → gọi lại `password-login` với `companyId` cần đăng nhập.

Request mẫu:

```json
{
  "username": "username.demo@email.com",
  "password": "123456&abcdef",
  "companyId": 668
}
```

Response mẫu:

```json
{
  "data": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9……",
  "success": true,
  "code": 0,
  "messages": ["Authenticate and choose company success"]
}
```

### 4.2) Lấy danh sách công ty

- **URL**: `/api/auth/get-companies`
- **Method**: `GET`
- **Authorize**: Bearer token
- **Parameter**: none

### 4.3) Chọn công ty

- **URL**: `api/auth/choose-company`
- **Method**: `POST`
- **Content-Type**: `application/json`
- **Body**: `comId` *(int, required)*

### 4.4) Đăng nhập bằng mã xử lý (process code)

- **URL**: `/auth/process-code-login`
- **Method**: `POST`
- **Content-Type**: `application/json`
- **Body**: `processCode` *(string, required)*

---

## 5) Quản lý mẫu chứng từ (Document Templates)

### 5.1) Danh sách mẫu chứng từ

- **URL**: `/api/document-templates`
- **Method**: `GET`
- **Authorize**: Bearer token
- **Query** *(optional)*: `search`, `page`, `pageSize`
- Response `data.items[]` gồm: `id`, `name`, `fileName`, `downloadUrl`, `pdfDownloadUrl`, `xlsxDownloadUrl`,…

### 5.2) Lấy chi tiết mẫu chứng từ theo id

- **URL**: `/api/document-templates/{id}`
- **Method**: `GET`
- **Authorize**: Bearer token

### 5.3) Tạo mẫu chứng từ

- **URL**: `/api/document-templates/create`
- **Method**: `POST`
- **Content-Type**: `multipart/form-data`
- **Authorize**: Bearer token
- **Form-data**:
  - `file` *(file, required)*: `.docx`
  - `name` *(string, required)*
  - `description` *(string, optional)*

### 5.4) Cập nhật mẫu chứng từ

- **URL**: `/api/document-templates/update`
- **Method**: `POST`
- **Content-Type**: `multipart/form-data`
- **Authorize**: Bearer token
- **Form-data**:
  - `id` *(int, required)*
  - `file` *(file, optional)*
  - `name` *(string, optional)*
  - `description` *(string, optional)*

---

## 6) Quản lý mẫu quy trình (Process Templates)

### Danh sách mẫu quy trình

- **URL**: `/api/process-templates`
- **Method**: `GET`
- **Authorize**: Bearer token
- **Query** *(optional)*: `search`, `page`, `pageSize`

---

## 7) Quản lý chứng từ (Documents)

### 7.1) Trạng thái chứng từ (`status`)

- `1`: Chứng từ mới
- `2`: Sẵn sàng
- `3`: Đang xử lý
- `4`: Đã hoàn tất
- `5`: Đang hiệu chỉnh
- `-3`: Đã hủy
- `-2`: Đã xóa
- `-1`: Đã bị từ chối

### 7.2) Trạng thái hợp đồng (`contractStatus`)

- `0`: Không xác định
- `1`: Còn hiệu lực
- `2`: Gần hết hiệu lực
- `-2`: Chưa có hiệu lực
- `-1`: Hết hiệu lực

### 7.3) Danh sách chứng từ

- **URL**: `/api/documents`
- **Method**: `GET`
- **Authorize**: Bearer token
- **Query** *(optional, rất nhiều)*:
  - Lọc theo `status`, `contractStatus`, `documentTemplateId`, `departmentId`, `documentTypeId`, `recipientIds[]`, `createdByUserId`, `batchImportId`
  - Lọc thời gian: `createdFrom`, `createdTo`, `completedFrom`, `completedTo`
  - Tìm kiếm: `search` (theo mã hoặc tên)
  - Phân trang: `page`, `pageSize`
  - Cờ trạng thái: `waitToApprove`, `waitToSignDigital`, `waitToSignDraw`,…

Response thường có `data.items[]` và mỗi item có `downloadUrl` (dùng để tải PDF).

### 7.4) Tải PDF chứng từ

- Dùng `downloadUrl` trong response (dạng `{HOST}/Api/Download?token=...`) để HTTP GET tải file.

### 7.5) Danh sách loại chứng từ

- **URL**: `/api/document-types`
- **Method**: `GET`
- **Authorize**: Bearer token
- **Query** *(optional)*: `search`, `page`, `pageSize`

### 7.6) Tạo mới chứng từ (upload PDF từ hệ thống ngoài)

- **URL**: `/api/documents/create`
- **Method**: `POST`
- **Content-Type**: `multipart/form-data`
- **Authorize**: Bearer token
- **Form-data**:
  - `no` *(string, required)*: mã chứng từ (không trùng)
  - `subject` *(string, required)*
  - `file` *(file, required)*: PDF
  - `typeId` *(int, required)*
  - `departmentId` *(int, required)*
  - `description` *(string, optional)*
  - `documentTemplateId` *(int, optional)*
  - `customerCode`, `customerInformation` *(optional)*
  - `relatedDocumentIds[]` *(optional)*
  - `attachments[]` *(optional)*

### 7.7) Cập nhật chứng từ (khi còn trạng thái “mới khởi tạo”)

- **URL**: `/api/documents/update`
- **Method**: `POST`
- **Content-Type**: `multipart/form-data`
- **Authorize**: Bearer token
- **Form-data**:
  - `id` *(string, required)*: id chứng từ
  - các field còn lại tương tự create (optional)

### 7.8) Cập nhật quy trình chứng từ (update-process)

- **URL**: `/api/documents/update-process`
- **Method**: `POST`
- **Content-Type**: `application/json`
- **Authorize**: Bearer token
- **Body**:
  - `id` *(string, required)*: id chứng từ
  - `processInOrder` *(boolean)*: tuần tự hay không
  - `processes` *(array[object])*: danh sách bước xử lý

Mỗi item trong `processes`:

```json
{
  "orderNo": 1,
  "processedByUserCode": "baoth",
  "accessPermissionCode": "DR",
  "position": "20,750,150,820",
  "pageSign": 1
}
```

#### `accessPermissionCode` (quyền xử lý)

- `V`: chỉ xem
- `D`: ký số
- `E`: ký điện tử
- `EKYC`: ký eKYC
- `DR`: ký nháy
- `A`: phê duyệt
- `F`: điền dữ liệu
- `C`: điều phối

> Lưu ý tuần tự: `F` luôn xếp trước `D/E/EKYC/DR` khi `processInOrder = true`.

### 7.9) Gửi quy trình chứng từ (send-process)

- **URL**: `/api/documents/send-process/{documentId}`
- **Method**: `POST`
- **Authorize**: Bearer token
- Trả về chứng từ ở trạng thái xử lý, thường kèm `waitingProcess` (bước đang chờ xử lý tiếp theo).

### 7.10) Xử lý chứng từ (ký/phê duyệt) – flow OTP 2 bước

> API này thường dùng khi hệ thống khác **thực hiện ký thay** (hoặc tích hợp sâu). Nếu người dùng ký trực tiếp trên web/app VNPT eContract thì hệ thống tích hợp thường **không gọi** API này.

- **URL**: `/api/documents/process`
- **Method**: `POST`
- **Content-Type**: `application/json`
- **Authorize**: Bearer token
- **Body chính**:
  - `processId` *(required)*: lấy từ `waitingProcess.id` (sau `send-process`) hoặc từ `process-code-login`
  - `reason` *(required)*
  - `reject` *(required boolean)*
  - `otp` *(required)*:
    - lần 1: `null` → hệ thống gửi OTP và trả `code: 100`
    - lần 2: gửi OTP nhận được → xử lý ký/từ chối
  - Các thông tin hiển thị chữ ký: `signatureDisplayMode`, `signatureImage` (base64), `signingPage`, `signingPosition`, `signatureText`, `fontSize`, `showReason`, `confirmTermsConditions`

### 7.11) Gửi thông báo chứng từ đã xử lý hoàn tất

- **URL**: `/api/documents/send-notify/{documentId}`
- **Method**: `POST`
- **Authorize**: Bearer token

### 7.12) Hủy chứng từ

- **URL**: `/api/documents/cancel`
- **Method**: `POST`
- **Authorize**: Bearer token
- **Body**: `id` *(required)*, `reason` *(string)*
- Lưu ý: chứng từ chỉ bị hủy bởi người tạo.

### 7.13) Xóa chứng từ

- **URL**: `/api/documents/delete/{id}`
- **Method**: `DELETE`
- **Authorize**: Bearer token
- Query/body có thể có `permanent=true` để xóa vĩnh viễn.

### 7.14) Tạo lô chứng từ xử lý hàng loạt (batch process)

- **URL**: `/api/documents/create-batch-process`
- **Method**: `POST`
- **Content-Type**: `application/json`
- **Authorize**: Bearer token
- **Body**:
  - `documentIds` *(array[string], required)*
  - `accessPermissionCode` *(string, required)*: `V/D/DR/A/...`

### 7.15) Xử lý chứng từ hàng loạt (batch-process)

- **URL**: `/api/documents/batch-process`
- **Method**: `POST`
- **Content-Type**: `application/json`
- **Authorize**: Bearer token
- Tương tự `/api/documents/process` và cũng có flow OTP 2 bước (lần 1 `otp:null` → `code:100`).

### 7.16) Lấy chi tiết chứng từ theo id

- **URL**: `/api/documents/{id}`
- **Method**: `GET`
- **Authorize**: Bearer token

---

## 8) Batch Imports (tạo chứng từ theo mẫu/đục lỗ)

### 8.1) Danh sách lô import

- **URL**: `/api/batch-imports`
- **Method**: `GET`
- **Authorize**: Bearer token
- **Query** *(optional)*: `search`, `page`, `pageSize`

### 8.2) Import lô bằng file Excel

- **URL**: `/api/batch-imports/create`
- **Method**: `POST`
- **Content-Type**: `multipart/form-data`
- **Authorize**: Bearer token
- **Form-data**:
  - `file` *(file, required)*: excel template
  - `name` *(string, required)*
  - `departmentId` *(int, required)*
  - `documentTypeId` *(int, required)*
  - `documentTemplateId` *(int, required)*

### 8.3) Import lô nâng cao (create-advanced)

- **URL**: `/api/batch-imports/create-advanced`
- **Method**: `POST`
- **Content-Type**: `application/json`
- **Authorize**: Bearer token
- **Body**:
  - `name`, `departmentId`, `documentTypeId`, `documentTemplateId` *(required)*
  - `parameters` *(List[string], required)*
  - `rows` *(List[List[string]], required)*

#### Quy tắc `parameters` bắt buộc (6 phần tử đầu, đúng thứ tự)

1. `{{D.FileName}}`
2. `{{D.No}}` (mã chứng từ, không trùng)
3. `{{D.Subject}}`
4. `{{D.ExpiryDate}}` (dd/MM/yyyy)
5. `{{D.Description}}`
6. `{{D.IsOrder}}` (Y/N – xử lý tuần tự)

Sau đó là các cặp quy trình:

- `{{P.Code}}` (mã người nhận)
- `{{P.AccessPermission}}` (V/D/DR/A…)

Cuối cùng là các biến đục lỗ `{{ten_bien}}` đúng như trong file `.docx` mẫu.

> Không được đổi thứ tự: **Thông tin cơ bản → Quy trình ký → Biến đục lỗ**.

### 8.4) Gửi quy trình cho lô import

- **URL**: `/api/batch-imports/send/{id}`
- **Method**: `POST`
- **Authorize**: Bearer token
- Lưu ý `code: 202`: lô đang xử lý, cần gọi lại sau vài giây (tùy số lượng chứng từ).

---

## 9) Users / Roles / Departments / Groups (tham khảo nhanh)

### Users

- `GET /api/users` (lọc `search`, `status`, `signMethod`, `departmentId`, `page`, `pageSize`,…)
- `GET /api/users/{id}`
- `POST /api/users/create` (JSON)
- `POST /api/users/update` (JSON)
- `POST /api/users/create-or-update` (JSON, body là array)

### SmartCA certificates (gắn CTS cho user)

- `POST /api/users/smart-ca/add`
- `POST /api/users/smart-ca/update`
- `POST /api/users/smart-ca/delete`

### Roles

- `GET /api/roles`

### Departments

- `GET /api/departments`

### User groups

- `GET /api/user-groups`

---

## 10) Luồng tích hợp “tạo chứng từ cơ bản” (từ hệ thống ngoài → VNPT eContract)

Luồng chuẩn (đã lược bỏ các tương tác ký số/HSM):

1. Hệ thống ngoài xuất file `.docx`/`.pdf` (thực tế API create yêu cầu PDF).
2. Login lấy token.
3. Lấy danh sách `documentTypeId` và `departmentId`.
4. `POST /api/documents/create` → lấy `documentId`.
5. `POST /api/documents/update-process` → thiết lập người xử lý + quyền + tọa độ ký.
6. `POST /api/documents/send-process/{documentId}` → gửi quy trình.
7. Người dùng xử lý trên VNPT eContract (ký/duyệt…).
8. Hệ thống ngoài đồng bộ trạng thái:
   - ưu tiên webhook, hoặc
   - polling `GET /api/documents/{id}` / `GET /api/documents`.
9. Khi hoàn tất: tải PDF bằng `downloadUrl` và lưu về hệ thống ngoài.

---

## 11) Gợi ý mapping nhanh cho ERPNext `Contract`

- `Contract` → `document.no`: dùng `Contract.name` hoặc field “Contract Number” (phải unique).
- `Contract` → `subject`: tiêu đề hiển thị (VD: `Hợp đồng <no>`).
- Print Format → PDF → upload vào `file`.
- Lưu `documentId` trả về vào custom field của `Contract` để đối chiếu webhook/polling.

