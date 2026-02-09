# Quy trình nghiệp vụ – App Fulco

Tài liệu mô tả luồng quy trình từ nhập kho đến xuất kho, bảo hành và quản lý mã QR trên nền ERPNext.

---

## 1. Vòng đời QR (nguyên tắc)

- **QR = ID duy nhất, không đổi suốt vòng đời**  
  Mỗi bản ghi DocType **QR Code** có một `qr_value` (chuỗi FULCO\|item\|serial\|batch\|pr\|warehouse). Giá trị này không thay đổi dù hàng chuyển kho hay xuất bán.

- **QR in ra KHÔNG phụ thuộc kho**  
  Tem QR in một lần (thường khi nhập kho). Khi hàng chuyển kho, không in lại tem; hệ thống chỉ cập nhật trường **Kho**, **Trạng thái** trên bản ghi QR Code.

- **Kho, trạng thái, lịch sử → truy ngược từ DocType QR Code**  
  Mọi thông tin hiện tại (kho, trạng thái) và liên kết chứng từ (PR, SE, DN, Sales Return) được lưu và truy vấn từ DocType **QR Code**.

---

## 2. Chuẩn bị (một lần / khi cần)

| Bước | Nội dung |
|------|----------|
| Quản lý tài khoản | Tạo/sửa/khóa user, phân quyền (dùng chuẩn ERPNext). |
| Quản lý mặt hàng | Thông tin Item, Barcode, Serial/Batch (ERPNext). Mã QR sản phẩm được sinh ở bước nhập kho. |
| Cấu hình Reorder (cho cảnh báo tồn) | Trong **Item** → bảng **Reorder Levels**: nhập **Warehouse** và **Re-order Level** (ngưỡng dưới) cho từng kho. |

---

## 3. Luồng nhập kho hàng mua (Purchase Receipt)

```
PO → PR (tạo & nhập Serial/Batch) → Submit PR → Sinh QR → In tem → Kiểm đếm QR (Purchase Receipt Check) → Submit PR (chỉ khi Đủ hàng)
```

### Bước 3.1. Tạo Đơn mua hàng (Purchase Order)

- Bộ phận mua tạo **Purchase Order** trên ERPNext.
- Xác định: Nhà cung cấp, danh sách Item, số lượng, kho nhập.
- **Chưa sinh QR** ở bước này.

### Bước 3.2. Tạo Phiếu nhập kho (Purchase Receipt)

- Nhân viên kho tạo **Purchase Receipt** từ PO.
- Hệ thống kế thừa: Item, số lượng, kho nhập.
- Với hàng **theo Serial**: khai báo danh sách Serial (hoặc để hệ thống sinh).
- Với hàng **theo Batch**: nhập Batch hoặc để ERPNext xử lý.
- **Lưu** PR (chưa Submit).

### Bước 3.3. Submit Purchase Receipt

- Submit PR để khoá số liệu nhập (Serial/Batch Bundle được tạo).
- Sau khi Submit mới thực hiện **Sinh mã QR cho phiếu**.

### Bước 3.4. Sinh mã QR cho sản phẩm

- Mở lại **Purchase Receipt** đã Submit.
- Chọn **Fulco → Sinh mã QR cho phiếu**.
- Hệ thống:
  - Với từng dòng có **Serial/Batch**: sinh **1 QR cho mỗi Serial/Batch** (Item, Serial/Batch, PR, Kho).
  - Với dòng **không** Serial/Batch: sinh **1 QR** theo Item + PR + Kho.
  - Tạo/cập nhật bản ghi **QR Code** và hiển thị ảnh QR; có thể **tải từng ảnh** hoặc mở bản ghi QR Code từ link.
- **Lưu ý:** Mỗi QR tương ứng **một bản ghi** DocType **QR Code** (1 Serial = 1 bản ghi; 1 Batch = 1 bản ghi; 1 dòng không serial/batch = 1 bản ghi).

### Bước 3.5. In tem QR và dán lên hàng

- In ảnh QR đã tải (hoặc dùng **In hàng loạt** từ List **QR Code**).
- Dán tem lên sản phẩm / bao bì / kiện tương ứng.
- Mỗi tem = **một định danh duy nhất** cho đơn vị hàng trong kho.

### Bước 3.6. Quét QR để kiểm đếm nhập kho

- Tạo bản ghi **Purchase Receipt Check** (Kiểm đếm nhập kho).
- Chọn **Purchase Receipt** tương ứng (trạng thái Draft) → hệ thống load bảng: Item, Tên, Số lượng yêu cầu, Kho.
- **Quét QR** (máy quét / camera / ô "Quét mã (Barcode/QR)"):
  - Mỗi lần quét: hệ thống nhận Item/Serial/Batch, **cộng Số lượng đã quét**, ghi serial đã quét.
  - **Cảnh báo:** quét trùng Serial, quét sai hàng so với PR, vượt số lượng yêu cầu.
- Hiển thị **real-time:** số lượng đã quét, số còn thiếu.
- **Save** để lưu trạng thái (Đủ hàng / Thiếu / Thừa).

### Bước 3.7. Xác nhận nhập kho (Submit PR)

- Mở lại **Purchase Receipt**.
- **Chỉ Submit được** khi:
  - Đã có **Purchase Receipt Check** cho PR đó.
  - Trạng thái Check = **Đủ hàng** (số quét = số nhập).
- Sau Submit: tồn kho được cập nhật; QR tiếp tục dùng cho xuất kho, bảo hành, truy xuất nguồn gốc.

---

## 4. Luồng xuất kho hàng bán (Delivery Note)

```
SO → DN (tạo phiếu xuất) → Kiểm đếm QR (Delivery Note Check) → Xử lý chênh lệch (nếu thiếu) → Submit DN
```

### Bước 4.1. Tạo / nhập chứng từ bán hàng

- Lập **Sales Order** → tạo **Delivery Note** (Phiếu xuất kho).
- Xác định: Item, số lượng, kho xuất.

### Bước 4.2. Chuẩn bị xuất kho

- Kho tiếp nhận yêu cầu theo **Delivery Note**.
- Hệ thống hiển thị số lượng cần xuất cho từng Item.

### Bước 4.3. Quét QR để kiểm đếm xuất kho

- Tạo bản ghi **Delivery Note Check** (Kiểm hàng xuất kho).
- Chọn **Delivery Note** cần kiểm → load bảng: Item, Số lượng yêu cầu.
- **Quét QR** trên từng sản phẩm/kiện:
  - Hệ thống nhận Item/Serial/Batch, cộng **Số lượng thực xuất**, đối soát với số yêu cầu.
  - **Cảnh báo:** quét sai mã, quét trùng QR, vượt số lượng.
- Hiển thị real-time: đã quét – còn thiếu.
- **Save** → trạng thái: Đủ hàng / Thiếu / Thừa / Thiếu và thừa.

### Bước 4.4. Xử lý chênh lệch (nếu thiếu)

- Nếu có **thiếu hàng** (số thực xuất < số yêu cầu):
  - Trên **Delivery Note Check** bấm **Xử lý chênh lệch**.
  - Hệ thống: tạo **Delivery Note mới** theo số lượng thực tế (gắn SO và DN gốc); nếu DN gốc đang Draft thì xóa DN gốc, nếu đã Submit thì chỉ tạo DN mới (hủy DN gốc thủ công nếu cần).

### Bước 4.5. Submit Phiếu xuất kho

- Mở **Delivery Note** (mới sau xử lý chênh lệch, hoặc DN gốc nếu đủ hàng).
- **Chỉ Submit được** khi:
  - Có **Delivery Note Check** tương ứng.
  - Trạng thái Check = **Đủ hàng**.
- Sau Submit:
  - **Fulco** tự cập nhật DocType **QR Code**: gán **Delivery Note**, **status = "Hàng đã bán"** cho các Serial/Batch thuộc DN đó.
  - Tồn kho cập nhật; QR dùng tiếp cho bảo hành, truy xuất nguồn gốc.

---

## 5. Chuyển kho (Stock Entry) và Hàng trả lại (Sales Return)

### 5.1. Stock Entry (chuyển kho)

- Khi Submit **Stock Entry** (Material Transfer hoặc loại có Serial/Batch):
  - Fulco cập nhật **QR Code** tương ứng: gán **Stock Entry**, **Warehouse** (kho đích), **status = "Hàng trong kho (sau SE)"**.

### 5.2. Sales Invoice (Hóa đơn trả lại – is_return)

- Khi Submit **Sales Invoice** với **is_return = 1**:
  - Fulco cập nhật **QR Code** tương ứng: gán **Sales Return** (link SI), **status = "Hàng trả lại"**.

---

## 6. Cảnh báo tồn kho đạt ngưỡng dưới

- **Scheduler** chạy **hàng ngày** (`fulco.tasks.daily`).
- Đọc **Item Reorder** (Warehouse + Re-order Level).
- So sánh với **Bin** (actual_qty): nếu **actual_qty < Re-order Level** thì tạo **Notification** cho user có role **Stock Manager** (danh sách Item + Kho + tồn + ngưỡng).
- **Cấu hình:** Trong **Item** → tab **Reorder Levels** nhập Warehouse và Re-order Level cho từng kho.

---

## 7. Bảo hành

### 7.1. Kích hoạt bảo hành (khi bàn giao)

- Khi quét QR tại thời điểm bàn giao cho khách:
  - Gọi API **`fulco.api.activate_warranty`** với tham số **code** (giá trị quét: FULCO\|... hoặc Serial No).
  - Hệ thống:
    - Chỉ xử lý khi code xác định được **Serial No**.
    - Kiểm tra Serial đã có **customer** (đã xuất kho).
    - Nếu chưa có **warranty_expiry_date**: tính từ **Item → Warranty Period** (vd. "12 Months") và ghi vào Serial No.
  - Cảnh báo nếu Serial đã được kích hoạt bảo hành trước đó.

### 7.2. Tra cứu bảo hành

- **Trang web (guest):** `/warranty_lookup?code=...` (quét QR hoặc nhập mã).
- **API:** `fulco.api.get_warranty_lookup(code)` (allow_guest).
- Hiển thị: thông tin sản phẩm, Serial, **trạng thái bảo hành** (Chưa kích hoạt / Đang bảo hành / Hết hạn), ngày hết hạn, khách sở hữu, **lịch sử sửa chữa** (Warranty Claim).

---

## 8. Tra cứu thông tin sản phẩm và truy xuất nguồn gốc

| Chức năng | Cách dùng |
|-----------|-----------|
| **Thông tin sản phẩm** | Trang `/product_info?code=...` hoặc API `fulco.api.get_product_info?code=...` (guest). QR tem có thể trỏ URL này. |
| **Truy xuất nguồn gốc** | API `fulco.api.get_traceability(search_value)` với barcode/Serial/Batch → trả về Customer, DN, SO, thời gian xuất, Item/Serial/Batch. |

---

## 9. In hàng loạt QR (từ DocType QR Code)

- Vào **Fulco → QR Code** (List).
- **Chọn** các bản ghi cần in (checkbox).
- Bấm **Fulco → In hàng loạt**.
- Hệ thống mở cửa sổ mới với ảnh QR + nhãn (Item/Serial/Batch) → dùng **Print** của trình duyệt để in.

---

## 10. Tóm tắt DocType QR Code – khi nào tạo / cập nhật

| Sự kiện | Hành động với QR Code |
|---------|------------------------|
| **Sinh QR từ Purchase Receipt** (Fulco → Sinh mã QR cho phiếu) | **Tạo mới** hoặc **cập nhật** bản ghi (theo `qr_value`): Item, Serial/Batch, PR, Warehouse, Ảnh QR, status = "Hàng trong kho". |
| **Submit Delivery Note** | **Cập nhật** QR Code khớp Serial/Batch: gán **Delivery Note**, status = **"Hàng đã bán"**. |
| **Submit Stock Entry** | **Cập nhật** QR Code khớp Serial/Batch: gán **Stock Entry**, **Warehouse**, status = **"Hàng trong kho (sau SE)"**. |
| **Submit Sales Invoice (is_return)** | **Cập nhật** QR Code khớp Serial/Batch: gán **Sales Return**, status = **"Hàng trả lại"**. |

---

## 11. Sơ đồ luồng tổng quát

```
[PO] → [PR - Draft] → [Submit PR] → [Sinh QR] → [In tem] → [Purchase Receipt Check - Quét QR]
         ↓                                                                        ↓
         └──────────────────── [Submit PR chỉ khi Check = Đủ hàng] ←──────────────┘
                                              ↓
                              [QR Code: status = Hàng trong kho]
                                              ↓
[SO] → [DN] → [Delivery Note Check - Quét QR] → [Xử lý chênh lệch nếu thiếu] → [Submit DN]
         ↓                                                                        ↓
         └──────────────────── [Submit DN chỉ khi Check = Đủ hàng] ←─────────────┘
                                              ↓
                              [QR Code: delivery_note, status = Hàng đã bán]
                                              ↓
                    [Kích hoạt bảo hành (activate_warranty)] / [Tra cứu warranty_lookup]
                    [Chuyển kho: Stock Entry → QR Code: stock_entry, status = Hàng trong kho (sau SE)]
                    [Trả hàng: Sales Invoice is_return → QR Code: sales_return, status = Hàng trả lại]
```

---

*Tài liệu cập nhật theo BA Fulco và code hiện tại (app Fulco trên ERPNext).*
