## Hướng dẫn sử dụng QR Fulco

### 1. Chuẩn bị & khái niệm

- **QR Fulco**: Chuỗi `FULCO|Item|Serial/Batch|PR|Kho`.
- **Các màn hình chính**:
  - `Purchase Receipt` (Phiếu nhập kho).
  - `Purchase Receipt Check` (Kiểm đếm nhập kho).
  - `Delivery Note` (Phiếu xuất kho).
  - `Delivery Note Check` (Kiểm hàng xuất kho).
  - `QR Code` (Doctype quản lý QR).
  - Trang web: `product_info`, `warranty_lookup`.

### 2. Nhập kho mua hàng – Sinh QR & kiểm đếm

1. **Tạo Đơn mua hàng (PO)**  
   Làm như ERPNext chuẩn: chọn Nhà cung cấp, Item, SL, Kho nhập.

2. **Tạo Phiếu nhập (Purchase Receipt)**  
   - Tạo PR từ PO.  
   - Khai báo **Serial/Batch** cho từng dòng (theo chuẩn ERPNext).

3. **Sinh QR theo Serial/Batch sau khi Submit PR**  
   - Submit PR.  
   - Mở lại PR → menu **Fulco → “Sinh mã QR cho phiếu”**.  
   - Hệ thống:
     - Lấy tất cả Serial/Batch trong PR.
     - Sinh **1 QR cho mỗi Serial/Batch** (chứa Item, Serial/Batch, PR, Kho).
     - Hiển thị danh sách ảnh QR → **click vào ảnh để tải về** và in tem.

4. **Kiểm đếm nhập kho bằng QR (Purchase Receipt Check)**  
   - Mở DocType **`Purchase Receipt Check`** → New.  
   - Chọn **`Purchase Receipt`** tương ứng.  
   - Hệ thống load bảng: Item, Tên, SL cần nhập, Kho.
   - Quét QR:
     - Focus ô **“Quét mã (Barcode/QR)”** rồi dùng máy quét / điện thoại (camera) / nút **Thử quét (Test)**.
     - Mỗi QR quét:
       - Hệ thống nhận diện Item + Serial/Batch.
       - Cộng **`Số lượng đã quét`**, ghi lại serial đã quét.
       - **Cảnh báo** nếu: quét trùng serial, sai hàng so với PR, vượt số lượng yêu cầu.
   - Nhấn **Save** để lưu kết quả (trạng thái: Đủ hàng / Thiếu / Thừa).

5. **Submit Phiếu nhập**  
   - Mở lại **Purchase Receipt** tương ứng.  
   - Chỉ Submit được nếu:
     - Đã có **`Purchase Receipt Check`** cho PR đó.
     - Trạng thái Check = **“Đủ hàng”** (không có chênh lệch).
   - Submit xong: hàng vào kho, QR dùng tiếp cho xuất kho & bảo hành.

### 3. Xuất kho bán hàng – Kiểm đếm QR & xử lý chênh lệch

1. **Tạo phiếu xuất (Delivery Note)**  
   Theo chuẩn: từ Sales Order hoặc nhập tay.

2. **Kiểm hàng xuất kho bằng QR (Delivery Note Check)**  
   - Mở DocType **`Delivery Note Check`** → New.  
   - Chọn **`Delivery Note`** cần kiểm.  
   - Hệ thống load bảng: Item, SL yêu cầu.
   - Quét QR:
     - Focus ô **“Quét mã (Barcode/QR)”** → quét bằng máy quét / camera / Test.  
     - Hệ thống:
       - Nhận diện Item/Serial/Batch.
       - Cộng **`Số lượng thực xuất`**, tính chênh lệch.
       - Nếu mặt hàng **không có trong DN** → tạo dòng thừa, cảnh báo.
   - Save lại để cập nhật trạng thái (Đủ / Thiếu / Thừa / Thiếu & Thừa).

3. **Xử lý chênh lệch (nếu thiếu)**  
   - Nếu `has_mismatch = 1` và có dòng **thiếu**:  
     - Bấm nút **“Xử lý chênh lệch”** trên `Delivery Note Check`.  
     - Hệ thống:
       - Tạo **Delivery Note mới** với SL = `Số lượng thực xuất`.
       - Nếu DN gốc còn Draft → xóa DN gốc; nếu đã Submit thì chỉ tạo DN mới, bạn tự hủy DN gốc.

4. **Submit Phiếu xuất**  
   - Mở **Delivery Note** (mới, sau xử lý nếu có).  
   - Chỉ Submit được nếu:
     - Có `Delivery Note Check` tương ứng.
     - Trạng thái Check = **“Đủ hàng”**.

### 4. Tra cứu sản phẩm & bảo hành bằng QR

1. **Tra cứu thông tin sản phẩm (public)**  
   - QR trên tem có thể trỏ đến: `/product_info?code=...`  
   - Hiển thị: mã hàng, tên, nhóm, thương hiệu, mô tả, ảnh.

2. **Tra cứu bảo hành (nội bộ & khách hàng)**  
   - URL: `/warranty_lookup?code=...` (quét QR hoặc nhập Serial/Item/QR Fulco).  
   - Hiển thị:
     - Thông tin sản phẩm.
     - Serial.
     - Trạng thái bảo hành: **Chưa kích hoạt / Đang bảo hành / Hết hạn**.
     - Ngày hết hạn (nếu có), khách hàng sở hữu.
     - Lịch sử Warranty Claim (sửa chữa).

### 5. Quản lý tập trung mã QR (Doctype `QR Code`)

1. **Xem danh sách QR**  
   - Mở DocType **`QR Code`**.  
   - Mỗi bản ghi là **1 QR duy nhất**:
     - `QR Value`, `Ảnh QR`.
     - Liên kết: Item, Serial, Batch, PR/SE/DN/Sales Return, Kho hiện tại.
     - Trạng thái hàng hoá: Hàng trong kho / Hàng trong kho (sau SE) / Hàng đã bán / Hàng trả lại / Disabled.

2. **Tra cứu ngược từ QR**  
   - Lấy chuỗi quét được → tìm trong `QR Code`.  
   - Từ đó xem nhanh sản phẩm, serial, lô, chứng từ liên quan, kho và trạng thái hiện tại.

### 6. Lưu ý vận hành

- Máy quét QR hoạt động như bàn phím: chỉ cần focus vào ô “Quét mã” và bấm scan.
- Để tránh lỗi:
  - Luôn **Submit PR** rồi mới sinh QR để in tem.
  - Luôn tạo **Purchase Receipt Check** và **Delivery Note Check** trước khi Submit chứng từ.
- Logic Serial, Batch, bảo hành vẫn dựa trên **ERPNext chuẩn**; Fulco chỉ thêm lớp QR, kiểm đếm và tra cứu.
