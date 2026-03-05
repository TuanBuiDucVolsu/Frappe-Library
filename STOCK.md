## Tài liệu đào tạo nghiệp vụ kho (Stock) ERPNext (cho Dev)

Tài liệu dành cho **nhân viên dev mới** nắm luồng nghiệp vụ **kho / tồn kho (Stock)** trong ERPNext, các DocType liên quan và cách chúng liên kết với Selling, Buying để phục vụ phân tích, debug và custom.

---

## 1. Tổng quan module Stock (Kho)

Module **Stock** trong ERPNext quản lý:

- **Kho (Warehouse)**: Cấu trúc kho theo cây (nhà máy, chi nhánh, kho tổng, kho chi tiết, location...).  
- **Hàng tồn kho**: Số lượng, giá trị tồn theo kho, theo lô (Batch), theo Serial No.  
- **Giao dịch kho**: Nhập, xuất, điều chuyển nội bộ, sản xuất, kiểm kê, điều chỉnh.  
- **Báo cáo tồn kho**: Stock Ledger, Stock Balance, Stock Projected, Ageing, v.v.  
- **Cài đặt kho**: Stock Settings, phương pháp tính giá (FIFO, Moving Average, Weighted Average), auto Material Request, reorder level...

Module Stock giao tiếp chặt với:

- **Selling**: qua **Delivery Note**, **Pick List**.  
- **Buying**: qua **Purchase Receipt**.  
- **Manufacturing**: qua **Stock Entry** (Material Issue, Manufacture, Repack, Subcontract, v.v.).

---

## 2. Các luồng nghiệp vụ kho chính

### 2.1 Luồng nhập hàng mua (từ NCC)

Đây là phần giao với Buying:

```text
Purchase Order → Purchase Receipt → (Stock Ledger cập nhật tồn kho)
```

- **Purchase Receipt (PR)**: Khi Submit sẽ **tăng tồn kho** tại kho nhận.  
- Tất cả dòng PR được ghi vào **Stock Ledger Entry**.

### 2.2 Luồng xuất hàng bán (cho khách)

Giao với Selling:

```text
Sales Order → Pick List (soạn hàng) → Delivery Note → (Stock Ledger giảm tồn kho)
```

- **Delivery Note (DN)**: Khi Submit sẽ **giảm tồn kho** tại kho xuất.  
- DN cũng ghi vào **Stock Ledger Entry**.

### 2.3 Điều chuyển kho nội bộ (Material Transfer)

Khi chuyển hàng giữa các kho trong hệ thống:

```text
Stock Entry (Purpose: Material Transfer)
```

- Kho nguồn: xuất (-qty).  
- Kho đích: nhập (+qty).  
- Cùng 1 Stock Entry nhưng tạo 2 dòng Stock Ledger (out & in).

### 2.4 Điều chỉnh tồn kho (Stock Adjustment / Stock Reconciliation)

- Dùng khi kiểm kê thực tế lệch so với hệ thống:
  - **Stock Reconciliation**: nhập số lượng thực tế và (nếu cần) giá trị → hệ thống tạo các bút toán điều chỉnh tồn.  
  - **Stock Entry** (Purpose: Material Issue / Material Receipt) cũng có thể dùng điều chỉnh đơn giản.

### 2.5 Quy trình Pick & Pack (soạn hàng giao)

```text
Sales Order → Pick List → Delivery Note
```

- **Pick List**:  
  - Chỉ định từ kho nào sẽ pick từng Item; có thể dùng **Scan Barcode**.  
  - Không ảnh hưởng tồn kho; chỉ là hướng dẫn soạn hàng.  
- **Delivery Note**:  
  - Ghi nhận xuất kho thực tế theo SO / Pick List.

---

## 3. Các DocType kho quan trọng và quan hệ

### 3.1 Warehouse

- Cấu trúc cây, có thể có nhiều level (Company → Region → Warehouse...).  
- Dùng ở tất cả giao dịch kho: PR, DN, Stock Entry, Stock Reconciliation, v.v.

### 3.2 Stock Entry

DocType kho “đa năng”, với `purpose` khác nhau:

- **Material Transfer**: Điều chuyển nội bộ giữa kho.  
- **Material Issue**: Xuất kho (thường cho mục đích nội bộ, R&D, scrap...).  
- **Material Receipt**: Nhập kho không qua mua hàng (tài sản tự tạo, trả lại từ outsource...).  
- **Manufacture / Repack / Subcontract**: Dùng trong sản xuất – xuất nguyên vật liệu, nhập thành phẩm.  

Mỗi Stock Entry tạo các dòng **Stock Ledger Entry** tương ứng.

### 3.3 Stock Ledger Entry (SLE)

- Là **bảng giao dịch kho cấp thấp nhất**: mỗi dòng thể hiện 1 lần thay đổi tồn kho của 1 Item tại 1 Warehouse vào 1 thời điểm.  
- Tạo từ: Purchase Receipt, Delivery Note, Stock Entry, Stock Reconciliation, v.v.  
- Các báo cáo Stock Balance, Stock Projected sử dụng dữ liệu từ SLE.

### 3.4 Stock Balance / Stock Ledger (Report)

- **Stock Ledger**: Báo cáo dạng nhật ký toàn bộ chuyển động tồn kho (theo Item, Warehouse, ngày…).  
- **Stock Balance**: Số lượng + giá trị tồn tại thời điểm (hoặc tới ngày) → dùng để check nhanh số tồn và giá trị tồn.

### 3.5 Batch & Serial No

- **Batch**:
  - Sử dụng cho hàng có lô (ngày sản xuất, hết hạn).  
  - Mỗi lô là một Batch, Item có thể bật “Has Batch No”.  
  - Số lượng quản lý theo từng Batch.
- **Serial No**:
  - Dùng cho hàng serial theo từng đơn vị (device, máy móc…).  
  - Item bật “Has Serial No”, mỗi chiếc là 1 Serial No riêng.  
  - SLE lưu theo Serial No; báo cáo truy vết (Serial No and Batch Traceability).

### 3.6 Item & liên quan

Dù Item thuộc Selling/Buying, nhưng là **trung tâm** của kho:

- **Item**: khai báo UOM, trọng lượng, Item Group, thuộc tính batch/serial, default warehouse, reorder level…  
- **Item Group**, **Brand**, **Item Alternative**, **Item Manufacturer**: phục vụ phân loại, thay thế.  
- **UOM**: Đơn vị đo lường, conversion factor giữa Purchase UOM / Stock UOM / Sales UOM.

---

## 4. Workspace Stock & truy cập nhanh

Workspace `Stock` (theo `stock/workspace/stock/stock.json`) cung cấp:

- **Quick Access**:
  - `Item`, `Material Request`, `Stock Entry`, `Purchase Receipt`, `Delivery Note`,  
  - `Stock Ledger`, `Stock Balance`, `Dashboard`,  
  - `Learn Inventory Management` (link tới khóa học quản lý tồn kho).
- **Cards chính**:
  - `Items Catalogue` (Item, nhóm, giá).  
  - `Stock Transactions` (MR, Stock Entry, DN, PR, Pick List, Delivery Trip…).  
  - `Stock Reports` (các báo cáo tồn).  
  - `Settings` (Stock Settings, Warehouse, UOM, Item Variant Settings…).  
  - `Serial No and Batch`, `Tools`, `Key Reports`, `Other Reports`.

Dev mới có thể bắt đầu từ workspace này để thấy bức tranh tổng thể module Stock.

---

## 5. Báo cáo tồn kho quan trọng (để hiểu & debug)

Tùy version, nhưng thường có:

| Báo cáo | Mục đích |
|--------|---------|
| **Stock Ledger** | Nhật ký chuyển động tồn kho chi tiết theo thời gian. |
| **Stock Balance** | Số lượng & giá trị tồn hiện tại/tới ngày, theo Item + Warehouse. |
| **Stock Projected** | Tồn dự kiến (tính thêm SO, PO, Material Request...). |
| **Stock Ageing** | Phân tích tuổi tồn kho (hàng lâu ngày chưa xoay vòng). |
| **Serial No and Batch Traceability** | Truy vết hàng theo Serial/Batch qua PR, DN, Stock Entry, Manufacturing. |
| **Stock Summary / Stock Analysis** | Tổng hợp tồn và giá trị theo nhóm hàng, kho,… |

Khi debug chênh lệch tồn kho, thứ tự xem thường là:

1. Stock Ledger (giao dịch chi tiết).  
2. Stock Balance (tồn cuối kỳ).  
3. Các chứng từ gốc: PR, DN, Stock Entry, Stock Reconciliation.

---

## 6. Giao giữa Stock với Selling & Buying

- **Từ Buying → Stock**:
  - **Purchase Receipt**: PR Submit → SLE (+qty).  
  - Hủy PR: đảo ngược SLE tương ứng.

- **Từ Selling → Stock**:
  - **Delivery Note**: DN Submit → SLE (-qty).  
  - **Pick List**: không tạo SLE, chỉ hỗ trợ soạn hàng.

- **Điều chỉnh & kiểm kê**:
  - **Stock Reconciliation** và **Stock Entry (Material Issue/Receipt)** cũng ghi SLE.  
  - Cần cẩn trọng khi sửa/xóa các chứng từ này vì ảnh hưởng trực tiếp tới tồn kho và giá trị.

---

## 7. Gợi ý cho Dev khi làm việc với module kho

- **Luôn kiểm tra Stock Ledger**  
  Khi user báo “tồn sai”, hãy kiểm tra:
  - Các PR, DN, Stock Entry, Stock Reconciliation gần nhất.  
  - Xem SLE để hiểu vì sao số lượng đang như vậy.

- **Hiểu phương pháp tính giá**  
  - ERPNext hỗ trợ nhiều phương pháp (FIFO, Moving Average...).  
  - Khi dev liên quan tới valuation, cần đọc `Stock Settings` và các hàm core xử lý valuation.

- **Tôn trọng immutable ledger (v13+)**  
  - Từ các bản mới, Stock Ledger gần như bất biến; việc hủy chứng từ quá khứ có ràng buộc.  
  - Khi custom logic `on_cancel` hãy chú ý rule về backdated entries.

- **Batch / Serial No**  
  - Nếu Item có Batch/Serial, mọi giao dịch kho phải cung cấp batch_no/serial_no phù hợp.  
  - Các chỗ quét barcode (Scan Barcode) thường gọi API chung (`scan_barcode`) → phù hợp cho custom UI liên quan kho.

- **Tách rõ vai trò Stock Entry vs  PR/DN**  
  - **PR/DN**: luôn gắn với mua/bán (Supplier/Customer, PO/SO).  
  - **Stock Entry**: luồng nội bộ (transfer, issue, receipt, production…).  
  - Tránh lạm dụng Stock Entry cho nghiệp vụ đã có PR/DN chuẩn.

---

## 8. Tài liệu tham khảo (cho dev tự đào sâu)

- **Stock Entry**: `https://docs.erpnext.com/docs/user/manual/en/stock-entry`  
- **Stock Reconciliation**: `https://docs.erpnext.com/docs/user/manual/en/stock/stock-reconciliation`  
- **Delivery Note**: `https://docs.erpnext.com/docs/user/manual/en/stock/delivery-note`  
- **Material Transfer from Delivery Note / Purchase Receipt**:  
  - `https://docs.erpnext.com/docs/user/manual/en/stock/articles/material-transfer-from-delivery-note`  
- **Purchase Receipt**: `https://docs.erpnext.com/docs/user/manual/en/purchase-receipt`  
- **Material Request**: `https://docs.erpnext.com/docs/user/manual/en/material-request`  
- **Stock Reports** (Stock Ledger, Stock Balance, Stock Ageing...): tra cứu trong mục Stock Reports trên docs ERPNext.

---

## 9. Checklist kiến thức cho dev mới (Stock)

- [ ] Hiểu các luồng chính: Nhập (PR) → tồn tăng, Xuất (DN) → tồn giảm, Transfer → chuyển kho, Stock Entry & Stock Reconciliation → điều chỉnh/kiểm kê.  
- [ ] Biết đọc **Stock Ledger** và **Stock Balance** để giải thích tồn kho.  
- [ ] Nắm cấu trúc **Warehouse**, **Item**, **UOM**, Batch, Serial No.  
- [ ] Hiểu quan hệ giữa PR/DN/Stock Entry/Stock Reconciliation với Stock Ledger Entry.  
- [ ] Biết xem và sử dụng các báo cáo kho quan trọng trong workspace Stock.  
- [ ] Biết phân biệt khi nào dùng PR/DN, khi nào dùng Stock Entry.  
- [ ] Nhận thức được tác động của việc sửa/hủy chứng từ kho đến cả tồn kho và kế toán (qua Buying/Selling/Manufacturing).

---

*Tài liệu đào tạo nghiệp vụ kho (Stock) ERPNext – Phiên bản cho Dev. Cập nhật theo codebase và tài liệu chính thức ERPNext.*

