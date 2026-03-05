## Tài liệu đào tạo nghiệp vụ mua hàng ERPNext (cho Dev)

Tài liệu dành cho **nhân viên dev mới** nắm luồng nghiệp vụ **mua hàng (Buying / Procurement)** trong ERPNext, các DocType liên quan và cách chúng liên kết với nhau để phục vụ phân tích, debug và custom.

---

## 1. Tổng quan module Buying (Mua hàng)

Module **Buying** trong ERPNext dùng để quản lý toàn bộ quy trình mua hàng:

- **Supplier (Nhà cung cấp)**: Thông tin NCC, nhóm NCC, điều khoản thanh toán, tài khoản ngân hàng, currency, price list mặc định.
- **Purchase Transactions (Giao dịch mua)**: Material Request, Request for Quotation, Supplier Quotation, Purchase Order, Purchase Receipt, Purchase Invoice, Purchase Return, Debit Note.
- **Supplier Scorecard**: Đánh giá hiệu quả nhà cung cấp (giá, thời gian giao, chất lượng…).
- **Buying Reports**: Báo cáo phân tích mua hàng (Purchase Analytics, Purchase Order Analysis, Procurement Tracker, …).
- **Buying Settings**: Cấu hình quy trình (có bắt buộc PO/PR trước PI hay không, v.v.).

---

## 2. Luồng nghiệp vụ mua hàng (Procurement Cycle)

Luồng chuẩn từ nhu cầu đến thanh toán:

```text
Material Request → Request for Quotation (RFQ) → Supplier Quotation → Purchase Order (PO) → Purchase Receipt (PR) → Purchase Invoice (PI) → Payment Entry
```

| Bước | DocType | Mô tả ngắn |
|------|---------|-----------|
| 1 | **Material Request** | Phiếu yêu cầu vật tư: bộ phận sử dụng/kho đề xuất cần mua gì, số lượng, thời điểm. |
| 2 | **Request for Quotation (RFQ)** | Gửi yêu cầu báo giá cho nhiều nhà cung cấp. |
| 3 | **Supplier Quotation** | NCC trả lời báo giá; dùng để so sánh, chọn NCC. |
| 4 | **Purchase Order (PO)** | Đơn đặt hàng chính thức cho NCC, dựa trên Material Request / Supplier Quotation. |
| 5 | **Purchase Receipt (PR)** | Phiếu nhập hàng: ghi nhận số lượng thực tế nhận kho, cập nhật tồn kho. |
| 6 | **Purchase Invoice (PI)** | Hóa đơn mua hàng từ NCC, cập nhật chi phí và công nợ phải trả. Có thể tạo từ PO hoặc PR. |
| 7 | **Payment Entry** | Chứng từ thanh toán: trả tiền cho NCC theo hóa đơn. |

**Lưu ý cho dev:**

- Không phải công ty nào cũng dùng đủ 7 bước; có nơi **bỏ RFQ** hoặc **cho phép PI trực tiếp từ PO** (không qua PR) → phụ thuộc `Buying Settings` và cấu hình riêng từng Supplier.
- Trên PI, quan hệ đến PO/PR thể hiện qua các field link trong child items (tương đương `against_purchase_order`, `against_purchase_receipt`).

---

## 3. Các DocType chính và quan hệ

### 3.1 Material Request

- **Mục đích**: Ghi nhận **nhu cầu** vật tư (type: Purchase, Material Transfer, Manufacture, …). Đối với mua hàng là loại **Purchase**.
- **Trường chính**:
  - `material_request_type`, `schedule_date`
  - `items` (Item, qty, warehouse mong muốn)
- **Chuyển tiếp**:
  - Từ Material Request → tạo **Request for Quotation** hoặc **Purchase Order** trực tiếp (tùy quy trình).

### 3.2 Request for Quotation (RFQ)

- **Mục đích**: Gửi yêu cầu báo giá tới nhiều nhà cung cấp cho cùng 1 list item.
- **Trường chính**:
  - `suppliers` (danh sách Supplier), `items` (các Item cần báo giá)
- **Chuyển tiếp**:
  - Từ RFQ → mỗi NCC gửi lại **Supplier Quotation** (có thể qua portal).

### 3.3 Supplier Quotation

- **Mục đích**: Lưu báo giá của từng NCC.
- **Trường chính**:
  - `supplier`, `transaction_date`, `valid_till`, `items` (Item, qty, rate), `buying_price_list`, `taxes_and_charges`
- **Chuyển tiếp**:
  - Chọn 1 hoặc nhiều Supplier Quotation phù hợp → **Create → Purchase Order**.

### 3.4 Purchase Order (PO)

- **Mục đích**: Đơn đặt hàng chính thức gửi NCC; là cam kết mua hàng.
- **Trường quan trọng**:
  - `supplier`, `transaction_date`, `schedule_date`
  - `company`, `set_warehouse`, `items` (Purchase Order Item: item_code, qty, rate, uom,…)
  - `buying_price_list`, `taxes_and_charges`, `incoterm`, `payment_terms`
- **Quan hệ**:
  - PO có thể được tạo **từ**: Material Request / Supplier Quotation.
  - PO là nguồn để tạo **Purchase Receipt** và **Purchase Invoice**.
  - Trên PR/PI có tham chiếu ngược đến PO (link trong child items).
- **Logic trạng thái**:
  - % Received, % Billed dựa theo PR và PI đã tạo từ PO.

### 3.5 Purchase Receipt (PR)

- **Module**: chủ yếu thuộc **Stock** (giao với Buying).
- **Mục đích**: Ghi nhận hàng **thực tế nhập kho**, cập nhật tồn kho.
- **Trường quan trọng**:
  - `supplier`, `posting_date`, `posting_time`, `items` (accepted_qty, rejected_qty, warehouse, serial/batch nếu có)
- **Quan hệ**:
  - Tạo từ PO: **Create → Purchase Receipt** trên PO.
  - Mỗi dòng PR có thể link ngược tới dòng PO tương ứng; PR ảnh hưởng % Received trên PO.
- **Tác động**:
  - Submit PR → tăng tồn kho, cập nhật sổ kho (Stock Ledger).

### 3.6 Purchase Invoice (PI)

- **Module**: **Accounts** (giao với Buying & Stock).
- **Mục đích**: Hóa đơn mua từ NCC, ghi nhận chi phí và **Accounts Payable**.
- **Trường quan trọng**:
  - `supplier`, `posting_date`, `items` (item_code, qty, rate), tài khoản chi phí, thuế, v.v.
- **Quan hệ**:
  - Có thể tạo **từ**:
    - **Purchase Order** (chưa nhập kho nếu quy trình cho phép).
    - **Purchase Receipt** (phổ biến: nhận hàng trước, sau đó nhận hóa đơn).
  - PI cập nhật **% Billed** trên PO.
- **Sau khi Submit**:
  - Ghi nhận bút toán chi phí, công nợ phải trả, thuế đầu vào,…

### 3.7 Payment Entry

- **Mục đích**: Ghi nhận **thanh toán** cho NCC.
- **Quan hệ**:
  - Thường được tạo từ PI: **Create → Payment Entry**.
  - Chọn mode of payment, bank/cash account, allocate số tiền cho từng PI.

---

## 4. Master data quan trọng trong Buying

| DocType | Vai trò trong mua hàng |
|--------|-------------------------|
| **Supplier** | Thông tin NCC: nhóm, tax, bank, điều khoản thanh toán, currency, price list mặc định,… |
| **Supplier Group** | Phân loại NCC (local, oversea, theo ngành, …). |
| **Item** | Vật tư/hàng hóa/dịch vụ cần mua; có thể setup default supplier, default purchasing UOM, reorder level,… |
| **Item Price** | Giá mua/bán theo Price List (buying/selling). |
| **Price List** | Bảng giá (buying_price_list trên Supplier Quotation/PO/PI). |
| **Warehouse** | Kho nhập hàng (warehouse trên PR, default trên PO). |
| **Terms and Conditions** | Điều khoản in trên PO/PI. |
| **Purchase Taxes and Charges Template** | Mẫu thuế, phí cho giao dịch mua. |
| **Buying Settings** | Quy định có bắt buộc PO/PR trước PI hay không, default account v.v. |

---

## 5. Workspace Buying & truy cập nhanh

- **Workspace:** `Buying`
  - Shortcut thường có:
    - **Item**
    - **Material Request**
    - **Purchase Order**
    - **Purchase Invoice**
    - **Purchase Analytics**
    - **Purchase Order Analysis**
    - **Dashboard**
    - **Learn Procurement** (link tới khóa học procurement trên Frappe School).

Dev mới có thể vào workspace này để xem nhanh luồng object và các báo cáo chính.

---

## 6. Báo cáo quan trọng trong Buying (để hiểu luồng & debug)

| Báo cáo | Mục đích |
|--------|---------|
| **Purchase Analytics** | Phân tích số liệu mua theo Supplier, Item, nhóm,… |
| **Purchase Order Analysis** | Xem trạng thái từng PO và từng dòng: đã nhận/đã bill bao nhiêu, còn lại bao nhiêu. |
| **Purchase Order Trends** | Xu hướng PO theo thời gian, NCC, nhóm, dự án,… |
| **Procurement Tracker** | Theo dõi toàn bộ luồng: Material Request → RFQ → PO → PR → PI cho từng Item. |
| **Supplier-wise Sales Analytics** | Phân tích lịch sử mua theo NCC. |
| **Items to Order and Receive** | Gợi ý item cần đặt/mua, cần nhận. |
| **Purchase Receipt Trends / Purchase Invoice Trends** | Xu hướng nhập hàng / hóa đơn mua theo thời gian. |
| **Item-wise Purchase History** | Lịch sử mua cho từng Item: NCC, giá, số lượng, thời gian. |

---

## 7. Gợi ý cho Dev khi phân tích & custom quy trình mua hàng

- **Hiểu rõ link giữa các chứng từ**  
  - Từ **Material Request** / RFQ / Supplier Quotation → **Purchase Order** → **Purchase Receipt** → **Purchase Invoice** → **Payment Entry**.  
  - Trong code/SQL, các link thể hiện ở:
    - Child table item (ví dụ: `purchase_order_item`, `purchase_receipt_item`, `purchase_invoice_item`) với field tham chiếu đến document trước đó.
    - Các field tỷ lệ % Received / % Billed trên PO.

- **Cẩn trọng với tác động tồn kho & kế toán**  
  - **Purchase Receipt** ảnh hưởng **Stock Ledger** (tồn kho).  
  - **Purchase Invoice** ảnh hưởng **General Ledger** (công nợ, chi phí, thuế).  
  - Khi override hooks (`on_submit`, `on_cancel`) nên hiểu rõ bút toán và stock entry phát sinh.

- **Giá & thuế**  
  - Giá mua auto lấy từ **Item Price**, **Buying Price List**, **Pricing Rule**.  
  - Thuế lấy từ **Purchase Taxes and Charges Template**.  
  - Nếu custom pricing/discount, nên can thiệp ở `validate`/`before_save` đúng DocType (Supplier Quotation, PO, PI).

- **Linh hoạt quy trình**  
  - Nhiều tổ chức:
    - Bỏ qua RFQ/Supplier Quotation, tạo trực tiếp PO từ Material Request.
    - Cho phép **Purchase Invoice without PO / without PR** (cấu hình trong `Buying Settings` hoặc override ở từng Supplier).  
  → Khi dev, cần tôn trọng các flag cấu hình này.

- **Nâng cao: Purchase Return / Debit Note**  
  - Khi trả hàng cho NCC, ERPNext dùng **Debit Note** / **Purchase Return**, tạo từ **Purchase Invoice**.  
  - Điều này đảo ngược một phần bút toán & tồn kho đã ghi nhận trước đó.

---

## 8. Tài liệu tham khảo (cho dev tự đào sâu)

- **Tổng quan Buying**:  
  - `https://docs.erpnext.com/docs/user/manual/en/buying`
- **Procurement Cycle Overview**:  
  - Material Request → RFQ → Supplier Quotation → PO → PR → PI → Payment Entry  
  - `https://docs.erpnext.com/docs/user/manual/en/procurement-cycle-overview`
- **Material Request**: `https://docs.erpnext.com/docs/user/manual/en/material-request`
- **Request for Quotation**: `https://docs.erpnext.com/docs/user/manual/en/request-for-quotation`
- **Supplier Quotation**: `https://docs.erpnext.com/docs/user/manual/en/buying/supplier-quotation`
- **Purchase Order**: `https://docs.erpnext.com/docs/user/manual/en/purchase-order`
- **Purchase Receipt**: `https://docs.epamext.com/docs/user/manual/en/purchase-receipt`
- **Purchase Invoice**: `https://docs.erpnext.com/docs/user/manual/en/purchase-invoice`
- **Payment Entry**: `https://docs.erpnext.com/docs/user/manual/en/payment-entry`
- **Buying Reports**: `https://docs.erpnext.com/docs/user/manual/en/buying_reports`
- **Buying Settings**: `https://docs.erpnext.com/docs/user/manual/en/buying-settings`
- **Supplier / Supplier Essentials**:  
  - `https://docs.erpnext.com/docs/user/manual/en/supplier`  
  - `https://docs.erpnext.com/docs/user/manual/en/supplier-essentials`

---

## 9. Checklist kiến thức cho dev mới (Buying)

- [ ] Hiểu luồng chuẩn: **Material Request → RFQ → Supplier Quotation → Purchase Order → Purchase Receipt → Purchase Invoice → Payment Entry**.  
- [ ] Biết DocType nào thuộc **Buying**, DocType nào thuộc **Stock**, **Accounts** nhưng tham gia quy trình mua.  
- [ ] Biết tạo PO từ Material Request/Supplier Quotation, tạo PR từ PO, tạo PI từ PO/PR trên giao diện.  
- [ ] Nắm được cách tính **% Received** và **% Billed** trên PO (dựa trên PR, PI).  
- [ ] Nắm master data: Supplier, Supplier Group, Item, Item Price, Price List, Warehouse, Purchase Taxes & Charges Template, Buying Settings.  
- [ ] Biết mở và đọc các báo cáo: **Purchase Analytics**, **Purchase Order Analysis**, **Procurement Tracker**, **Item-wise Purchase History** để debug số liệu.  
- [ ] Biết các tình huống đặc biệt: **Purchase without PO/PR**, **Purchase Return / Debit Note**.  

---

*Tài liệu đào tạo nghiệp vụ mua hàng ERPNext – Phiên bản cho Dev. Cập nhật theo codebase và tài liệu chính thức ERPNext.*

