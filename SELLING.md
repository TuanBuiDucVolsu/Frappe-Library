# Tài liệu đào tạo nghiệp vụ bán hàng ERPNext (cho Dev)

Tài liệu dành cho **nhân viên dev mới** nắm luồng nghiệp vụ bán hàng trong ERPNext, các DocType liên quan và cách chúng liên kết với nhau để phục vụ phát triển/custom.

---

## 1. Tổng quan module Selling (Bán hàng)

Module **Selling** trong ERPNext gồm:

- **Báo cáo & phân tích:** Doanh số, doanh thu, xu hướng khách hàng.
- **Quản lý khách hàng:** Hồ sơ khách, lịch sử giao dịch, hạn mức tín dụng.
- **Hóa đơn:** Lập và quản lý hóa đơn bán, thanh toán, công nợ.
- **Đơn hàng:** Tạo và theo dõi đơn hàng từ khi có yêu cầu đến khi giao hàng.
- **Báo giá:** Tạo báo giá gửi khách, chuyển thành đơn hàng.

---

## 2. Luồng nghiệp vụ bán hàng (Sales Cycle)

Luồng chuẩn từ báo giá đến thu tiền:

```
Lead/Prospect → Quotation (Báo giá) → Sales Order (Đơn hàng) → Delivery Note (Phiếu giao hàng) → Sales Invoice (Hóa đơn) → Payment (Thanh toán)
```

| Bước | DocType | Mô tả ngắn |
|------|---------|------------|
| 1 | **Quotation** | Báo giá gửi khách: mặt hàng, số lượng, đơn giá, thuế, thời hạn hiệu lực. |
| 2 | **Sales Order (SO)** | Đơn hàng xác nhận: khách đặt, ngày giao, kho, điều kiện thanh toán. Có thể tạo từ Quotation. |
| 3 | **Delivery Note (DN)** | Phiếu giao hàng / phiếu xuất kho: ghi nhận hàng đã giao. Tạo từ SO. |
| 4 | **Sales Invoice (SI)** | Hóa đơn bán: lập bill thu tiền. Tạo từ SO hoặc DN. |
| 5 | **Payment Entry** | Ghi nhận thanh toán, đối soát công nợ. |

**Lưu ý cho Dev:**

- **Sales Order** có thể bỏ qua bước giao hàng (skip_delivery_note) → lập **Sales Invoice** trực tiếp từ SO (bán dịch vụ / không xuất kho).
- **Stock:** Khi Submit SO (nếu reserve_stock), tồn kho được “reserve”. Delivery Note làm giảm tồn thực tế. Sales Invoice cập nhật doanh thu và công nợ.

---

## 3. Các DocType chính và quan hệ

### 3.1 Quotation (Báo giá)

- **Module:** `erpnext.selling`
- **Mục đích:** Gửi báo giá cho Lead/Customer (quotation_to: Lead hoặc Customer).
- **Trường quan trọng:**  
  `quotation_to`, `party_name`, `transaction_date`, `valid_till`, `items` (child: Quotation Item), `selling_price_list`, `taxes_and_charges`, `terms_and_conditions`.
- **Chuyển tiếp:** Nút **Create** → **Sales Order** (nếu khách chấp nhận).

### 3.2 Sales Order (Đơn hàng bán)

- **Module:** `erpnext.selling` (doctype), giao với **Stock** (Delivery Note, Pick List) và **Accounts** (Sales Invoice).
- **Trường quan trọng:**  
  `customer`, `transaction_date`, `delivery_date`, `company`, `set_warehouse`, `reserve_stock`, `items` (Sales Order Item), `selling_price_list`, `taxes_and_charges`, `incoterm`, `payment_terms`.
- **Liên kết:**  
  - `against_sales_order` (trên Delivery Note, Sales Invoice) → tham chiếu ngược SO.
- **Luồng tạo chứng từ:**
  - SO → **Pick List** (soạn hàng, quét barcode).
  - SO → **Delivery Note** (giao hàng).
  - SO hoặc DN → **Sales Invoice** (lập hóa đơn).

### 3.3 Delivery Note (Phiếu giao hàng / Phiếu xuất kho)

- **Module:** `erpnext.stock`
- **Mục đích:** Ghi nhận hàng đã xuất/giao cho khách.
- **Liên kết:** `against_sales_order`, `customer`, từng dòng có thể có `serial_no`, `batch_no`.
- **Tính năng:** Có **Scan Barcode** để quét item/serial/batch khi xuất kho.

### 3.4 Sales Invoice (Hóa đơn bán)

- **Module:** `erpnext.accounts`
- **Mục đích:** Hóa đơn để thu tiền; cập nhật doanh thu và công nợ (Receivable).
- **Liên kết:** Lấy dòng từ **Sales Order** hoặc **Delivery Note** (against_sales_order, against_delivery_note).
- **Sau khi Submit:** Cập nhật sổ sách kế toán, công nợ khách hàng.

### 3.5 Các master data liên quan

| DocType | Vai trò |
|---------|--------|
| **Customer** | Đối tượng bán hàng (bắt buộc trên SO, DN, SI). |
| **Item** | Hàng hóa/dịch vụ; dùng trong Quotation, SO, DN, SI. |
| **Item Price** | Giá bán theo Price List (và có thể theo Customer). |
| **Price List** | Bảng giá (selling_price_list trên SO/Quotation). |
| **Warehouse** | Kho xuất (set_warehouse trên SO, DN). |
| **Sales Person / Sales Partner** | Nhân viên/đối tác bán; dùng báo cáo và hoa hồng. |
| **Territory** | Vùng bán hàng (phân cấp địa lý). |
| **Terms and Conditions** | Điều khoản in trên báo giá/đơn hàng. |
| **Sales Taxes and Charges Template** | Mẫu thuế (VAT, etc.) cho bán hàng. |

---

## 4. Workspace & truy cập nhanh

- **Workspace:** **Selling**  
  Đường dẫn cấu hình: `erpnext/selling/workspace/selling/selling.json`.
- Shortcut thường dùng: **Item**, **Sales Order**, **Sales Analytics**, **Point of Sale**, **Dashboard**.

---

## 5. Báo cáo quan trọng (cho hiểu luồng & debug)

| Báo cáo | Mục đích |
|---------|----------|
| **Sales Analytics** | Phân tích doanh số, trạng thái đơn. |
| **Sales Order Analysis** | Chi tiết đơn hàng, tồn, giao, bill. |
| **Sales Order Trends** | Xu hướng đơn hàng theo thời gian. |
| **Quotation Trends** | Xu hướng báo giá. |
| **Delivery Note Trends** | Xu hướng giao hàng. |
| **Sales Invoice Trends** | Xu hướng hóa đơn. |
| **Item-wise Sales History** | Lịch sử bán theo từng Item. |
| **Customer Acquisition and Loyalty** | Phân tích khách hàng. |

---

## 6. Luồng kho: SO → Pick List → Delivery Note (tóm tắt)

1. **Sales Order** (Submit) → có thể **reserve stock**.
2. **Pick List** (tạo từ SO): bộ phận kho soạn hàng, dùng **Scan Barcode** để xác nhận từng item/serial/batch đã lấy.
3. **Delivery Note**: tạo từ SO (hoặc từ Pick List). Ghi nhận hàng thực tế giao; có **Scan Barcode**.
4. **Sales Invoice**: tạo từ SO hoặc DN, đối ứng từng dòng (pending qty) để không ghi bill vượt số đã giao/đã đặt.

---

## 7. Gợi ý khi custom / dev

1. **Link giữa các chứng từ:**  
   Luôn nắm `against_sales_order`, `against_delivery_note`; khi tạo DN/SI từ SO, ERPNext tự điền pending items.
2. **Số dư tồn kho:**  
   SO (reserve) → Pick List → DN (actual stock decrease). Không nên thay đổi logic submit/cancel của DN nếu không rõ tác động kho.
3. **Giá và thuế:**  
   Lấy từ **Price List**, **Pricing Rule**, **Item Price**; thuế từ **Sales Taxes and Charges Template**. Custom giá nên can thiệp đúng hook (ví dụ `before_save`, `validate`).
4. **API:**  
   Có REST API để tạo DN từ SO, SI từ SO/DN; khi tích hợp hệ thống ngoài cần map đúng field và naming series.
5. **Barcode / Serial / Batch:**  
   Delivery Note và Pick List có sẵn Scan Barcode; custom thêm màn hình “kiểm hàng” (yêu cầu vs thực xuất) cần đọc dữ liệu từ DN và có thể gọi `erpnext.stock.utils.scan_barcode`.

---

## 8. Tài liệu tham khảo

- [ERPNext Selling Module](https://docs.erpnext.com/docs/user/manual/en/selling)
- [Sales Cycle (video)](https://docs.erpnext.com/docs/v13/user/videos/learn/sales-cycle)
- [Quotation](https://docs.erpnext.com/docs/user/manual/en/quotation)
- [Sales Invoice](https://docs.erpnext.com/docs/user/manual/en/sales-invoice)
- [Sales Integration](https://docs.erpnext.com/docs/user/manual/en/sales-integration) (tích hợp API)

---

## 9. Checklist nắm nghiệp vụ (cho Dev mới)

- [ ] Hiểu thứ tự: Quotation → Sales Order → Delivery Note → Sales Invoice → Payment.
- [ ] Biết DocType nào thuộc module Selling, Stock, Accounts.
- [ ] Biết cách tạo SO từ Quotation, DN từ SO, SI từ SO/DN (trên giao diện).
- [ ] Biết ý nghĩa `against_sales_order`, `against_delivery_note` và pending qty.
- [ ] Biết Pick List dùng để soạn hàng, DN để ghi nhận xuất/giao.
- [ ] Biết Item, Customer, Price List, Warehouse là master bắt buộc liên quan.
- [ ] Biết mở các báo cáo Sales Analytics, Sales Order Analysis để kiểm tra luồng số liệu.

---

*Tài liệu đào tạo nghiệp vụ bán hàng ERPNext – Phiên bản cho Dev. Cập nhật theo codebase và tài liệu chính thức ERPNext.*
