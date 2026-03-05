# Bài tập đào tạo Dev mới – 3 phân hệ Bán hàng, Mua hàng, Kho (ERPNext)

Bài tập thực hành trên môi trường ERPNext (dev/sandbox) để nhân viên dev mới nắm vững nghiệp vụ **Selling**, **Buying** và **Stock**. Làm tuần tự theo từng phần; mỗi bài có mục tiêu và gợi ý kiểm tra.

---

## Chuẩn bị chung

- Có sẵn: **Company**, **Item** (ít nhất 2–3 item có tồn kho), **Customer**, **Supplier**, **Warehouse** (ít nhất 2 kho).
- Nếu chưa có, tạo dữ liệu mẫu theo hướng dẫn trong tài liệu từng phân hệ.

---

# PHẦN 1: BÁN HÀNG (Selling)

## Bài 1.1 – Luồng bán hàng đầy đủ (Quotation → SO → DN → SI)

**Mục tiêu:** Thực hiện trọn luồng từ báo giá đến hóa đơn và kiểm tra liên kết giữa các chứng từ.

**Yêu cầu:**

1. Tạo **Quotation** cho 1 Customer với 2 Item, số lượng tùy chọn, có Selling Price List và (nếu dùng) Taxes and Charges.
2. Từ Quotation, dùng nút **Create → Sales Order** để tạo **Sales Order**. Kiểm tra: dữ liệu Item, qty, rate, thuế đã sang SO chưa.
3. Từ Sales Order, tạo **Delivery Note** (Create → Delivery Note). Nhập số lượng giao (có thể giao một phần). Submit Delivery Note.
4. Từ **Delivery Note** (hoặc từ SO), tạo **Sales Invoice**. Submit Sales Invoice.
5. Kiểm tra trên **Sales Order**: trường (hoặc chỉ số) liên quan % Delivered, % Billed đã cập nhật đúng chưa.

**Kiểm tra:**

- Mở từng chứng từ (Quotation, SO, DN, SI) và xác nhận field tham chiếu ngược: SO tham chiếu Quotation, DN/SI tham chiếu SO (hoặc DN).
- Trên **Stock Balance** / **Stock Ledger**: tồn kho của các Item đã **giảm** đúng số lượng đã giao trên Delivery Note chưa.

---

## Bài 1.2 – Bán hàng không qua giao hàng (SO → SI trực tiếp)

**Mục tiêu:** Hiểu trường hợp SO có **Skip Delivery Note** (bán dịch vụ / không xuất kho).

**Yêu cầu:**

1. Tạo **Sales Order** với 1 Item (có thể là Item loại Service hoặc Item thường). Bật tùy chọn **Skip Delivery Note** (nếu có trên form).
2. Không tạo Delivery Note. Từ SO, tạo trực tiếp **Sales Invoice**.
3. Submit Sales Invoice.
4. Giải thích: Trong trường hợp này, tồn kho có thay đổi không? Vì sao?

**Kiểm tra:**

- Sales Invoice đã ghi nhận doanh thu và công nợ; nếu Item là Stock Item và không có DN, tùy cấu hình có thể không giảm tồn (cần đọc tài liệu hoặc kiểm tra Stock Ledger).

---

## Bài 1.3 – Báo cáo và debug số liệu bán hàng

**Mục tiêu:** Sử dụng báo cáo Selling để kiểm tra dữ liệu và làm quen với cách debug.

**Yêu cầu:**

1. Mở báo cáo **Sales Analytics**. Lọc theo khoảng thời gian có SO/DN/SI vừa tạo. Nêu ý nghĩa các số liệu (hoặc chart) hiển thị.
2. Mở **Sales Order Analysis**. Tìm các Sales Order vừa tạo; kiểm tra cột (hoặc thông tin) liên quan **To Deliver**, **To Bill**, **Completed**.
3. Giả sử khách hàng báo “Đơn X chưa thấy hóa đơn”. Nêu các bước bạn sẽ kiểm tra trong ERPNext (từ SO → DN → SI, và báo cáo nào dùng để đối chiếu).

**Nâng cao:**

- Dùng **Sales Funnel** (nếu có) để xem trạng thái luồng bán hàng; giải thích cách đọc funnel.

---

# PHẦN 2: MUA HÀNG (Buying)

## Bài 2.1 – Luồng mua hàng đầy đủ (Material Request → PO → PR → PI)

**Mục tiêu:** Thực hiện trọn luồng từ yêu cầu mua đến nhập kho và hóa đơn mua.

**Yêu cầu:**

1. Tạo **Material Request** với Material Request Type = **Purchase**, 2 Item, số lượng và Warehouse yêu cầu. Submit.
2. Từ Material Request, tạo **Purchase Order** (Create → Purchase Order). Hoặc tạo PO trực tiếp và “Get Items from” Material Request. Chọn Supplier, kiểm tra giá. Submit PO.
3. Từ Purchase Order, tạo **Purchase Receipt**. Nhập số lượng nhận (có thể nhận một phần). Submit Purchase Receipt.
4. Mở **Stock Balance** hoặc **Stock Ledger**: xác nhận tồn kho **tăng** đúng số lượng đã nhận trên PR.
5. Từ **Purchase Receipt** (hoặc từ PO), tạo **Purchase Invoice**. Submit PI.
6. Kiểm tra trên **Purchase Order**: % Received, % Billed (hoặc tương đương) đã cập nhật đúng chưa.

**Kiểm tra:**

- Trên PO, PR, PI: các field tham chiếu ngược (against_sales_order không có; against_purchase_order, against_purchase_receipt) đúng với luồng vừa làm.
- Công nợ nhà cung cấp (Accounts Payable) tăng sau khi Submit PI.

---

## Bài 2.2 – Luồng có báo giá (RFQ (Request for Quotation) → Supplier Quotation → PO)

**Mục tiêu:** Làm quen với bước RFQ và Supplier Quotation trước khi đặt hàng.

**Yêu cầu:**

1. Tạo **Request for Quotation** với 1 hoặc 2 Item, thêm 2 Supplier vào danh sách. Submit RFQ.
2. Tạo 2 **Supplier Quotation** (mỗi NCC một báo giá) với giá khác nhau cho cùng Item. Có thể link với RFQ (nếu form hỗ trợ).
3. Chọn 1 Supplier Quotation “thắng thầu”, từ đó tạo **Purchase Order**. Submit PO.
4. (Tùy chọn) Tiếp tục tạo PR và PI như Bài 2.1 để đóng vòng.

**Kiểm tra:**

- PO có tham chiếu tới Supplier Quotation (hoặc RFQ) không? Nêu ý nghĩa của việc so sánh nhiều báo giá trong thực tế.

---

## Bài 2.3 – Báo cáo và debug số liệu mua hàng

**Mục tiêu:** Dùng báo cáo Buying để kiểm tra và debug.

**Yêu cầu:**

1. Mở **Purchase Order Analysis**. Lọc theo thời gian và (nếu có) Supplier. Tìm các PO vừa tạo; giải thích ý nghĩa **To Receive**, **To Bill**, **Completed** (hoặc tương đương).
2. Mở **Procurement Tracker** (nếu có). Tìm 1 Item đã có trong Material Request / PO. Mô tả thông tin mà báo cáo cung cấp (từ yêu cầu → đặt hàng → nhận → bill).
3. Giả sử user báo “Đơn mua Y đã nhận hàng nhưng chưa thấy hóa đơn”. Nêu các bước kiểm tra trong ERPNext (PO → PR → PI, và báo cáo nào dùng).

**Nâng cao:**

- Dùng **Purchase Analytics** (hoặc Purchase Order Trends), lọc theo Item/Supplier; giải thích cách đọc số liệu.

---

# PHẦN 3: KHO (Stock)

## Bài 3.1 – Cấu trúc kho và tồn kho

**Mục tiêu:** Hiểu Warehouse và cách xem tồn theo kho.

**Yêu cầu:**

1. Liệt kê cấu trúc **Warehouse** trong site (cây kho: Company → … → kho con). Nếu chưa có, tạo 2 Warehouse (ví dụ: Kho A, Kho B) thuộc cùng Company.
2. Mở báo cáo **Stock Balance**. Lọc theo 1 Warehouse và 1 Item đã có tồn. Ghi lại: Số lượng tồn, Giá trị (nếu có). Giải thích ý nghĩa các cột chính.
3. Mở **Stock Ledger** cho cùng Item và Warehouse. Giải thích: mỗi dòng trong báo cáo tương ứng với loại chứng từ nào (Purchase Receipt, Delivery Note, Stock Entry, …)?

**Kiểm tra:**

- Sau khi làm Bài 2.1 (PR) và Bài 1.1 (DN), mở lại Stock Ledger cho Item đó: có đủ dòng “in” từ PR và “out” từ DN không?

---

## Bài 3.2 – Điều chuyển kho (Stock Entry – Material Transfer)

**Mục tiêu:** Thực hiện chuyển hàng giữa hai kho và kiểm tra Stock Ledger.

**Yêu cầu:**

1. Tạo **Stock Entry** với Purpose = **Material Transfer**. Chọn kho nguồn (From Warehouse) và kho đích (To Warehouse). Thêm 1 Item và số lượng (nhỏ hơn hoặc bằng tồn tại kho nguồn). Submit.
2. Mở **Stock Balance** cho Item đó tại **từng** kho (kho nguồn và kho đích). Số lượng tồn đã thay đổi đúng chưa (giảm ở nguồn, tăng ở đích)?
3. Mở **Stock Ledger** cho Item đó (không lọc warehouse hoặc lọc cả 2 kho). Giải thích: vì sao có 2 dòng (hoặc 2 nhóm dòng) cho cùng 1 Stock Entry (một out, một in)?

**Kiểm tra:**

- Trên form Stock Entry, field `purpose` và các trường warehouse phải khớp với tác động tồn kho bạn quan sát.

---

## Bài 3.3 – Điều chỉnh tồn kho (Stock Entry – Material Issue / Receipt hoặc Stock Reconciliation)

**Mục tiêu:** Làm quen với việc điều chỉnh tồn khi kiểm kê hoặc sai lệch.

**Yêu cầu:**

1. Chọn 1 Item và 1 Warehouse có tồn. Ghi lại số lượng tồn hiện tại (từ Stock Balance).
2. Tạo **Stock Entry** với Purpose = **Material Receipt** (hoặc **Material Issue** tùy bài). Nhập 1 Item, 1 Warehouse, số lượng (ví dụ +10 hoặc -5). Submit.
3. Kiểm tra **Stock Balance** và **Stock Ledger**: tồn và các dòng ledger đã thay đổi đúng chưa?
4. (Nếu môi trường có **Stock Reconciliation**) Mở Stock Reconciliation, chọn Item + Warehouse, nhập “Quantity Count” khác với tồn hệ thống. Lưu/Submit và quan sát: hệ thống tạo bút toán điều chỉnh như thế nào?

**Kiểm tra:**

- Không dùng Stock Entry Material Receipt/Issue cho nghiệp vụ đã có chứng từ chuẩn (PR/DN). Chỉ dùng để nhập/xuất nội bộ hoặc điều chỉnh.

---

## Bài 3.4 – Pick List và Delivery Note (tích hợp Selling – Stock)

**Mục tiêu:** Nắm vai trò Pick List trong luồng xuất kho bán hàng.

**Yêu cầu:**

1. Có sẵn 1 **Sales Order** (từ Bài 1.1 hoặc tạo mới) với ít nhất 1 Item, có kho (Set Warehouse). Submit SO.
2. Từ Sales Order, tạo **Pick List** (Create → Pick List). Kiểm tra: các dòng Item, số lượng, kho lấy hàng đã điền từ SO chưa. Có thể chỉnh kho từng dòng nếu nhiều kho. Submit Pick List.
3. Từ **Pick List**, tạo **Delivery Note**. Submit Delivery Note.
4. Kiểm tra **Stock Ledger**: số lượng xuất có khớp với Delivery Note không? Giải thích: Pick List có tạo dòng Stock Ledger không, hay chỉ Delivery Note mới làm giảm tồn?

**Kiểm tra:**

- Trên Delivery Note có tham chiếu tới Pick List và SO; trên SO trạng thái delivered/billed cập nhật đúng.

---

# PHẦN 4: TÍCH HỢP BA PHÂN HỆ

## Bài 4.1 – Vòng tròn đầy đủ: Mua → Tồn kho → Bán

**Mục tiêu:** Thực hiện một vòng kín: mua hàng nhập kho, sau đó bán chính hàng đó và xuất kho.

**Yêu cầu:**

1. Chọn 1 **Item** và 1 **Warehouse**. Ghi lại tồn ban đầu (Stock Balance).
2. **Mua hàng:** Tạo PO → PR (nhập 1 số lượng, ví dụ 50). Submit PR. Kiểm tra tồn tăng.
3. **Bán hàng:** Tạo SO (cùng Item, số lượng ≤ tồn hiện có, cùng Warehouse) → DN (giao đủ hoặc một phần) → SI. Submit DN và SI.
4. Kiểm tra **Stock Balance** và **Stock Ledger** cho Item + Warehouse: tồn cuối = tồn đầu + số nhập (PR) − số xuất (DN). Giải thích từng loại chứng từ tạo dòng Stock Ledger như thế nào.

**Kiểm tra:**

- Công nợ NCC (sau PI) và công nợ khách (sau SI) có thể kiểm tra qua báo cáo kế toán hoặc Payment Entry.

---

## Bài 4.2 – Debug tình huống “Tồn kho sai”

**Mục tiêu:** Rèn luyện tư duy debug tồn kho khi user báo “số liệu không khớp”.

**Kịch bản:** User báo: “Item Z tại Kho A tồn trên hệ thống là 100 nhưng kiểm kê thực tế chỉ còn 80.”

**Yêu cầu:**

1. Nêu **các bước** bạn sẽ làm trong ERPNext để tìm nguyên nhân (gợi ý: Stock Ledger, danh sách chứng từ gần nhất, kiểm tra PR/DN/Stock Entry/Stock Reconciliation).
2. Giả sử nguyên nhân là “có 1 Delivery Note ghi nhận xuất 20 nhưng thực tế chưa xuất”. Bạn sẽ xử lý thế nào (hủy DN, điều chỉnh, hay tạo chứng từ bù)? Nêu rõ ưu/nhược điểm từng hướng (chỉ mô tả, không bắt buộc thao tác trên site).
3. Nếu nguyên nhân là “kiểm kê đúng là 80, hệ thống sai”: nên dùng chứng từ nào để đưa tồn hệ thống về 80? Các bước thao tác (tóm tắt).

**Kiểm tra:**

- Hiểu được Stock Ledger là nguồn chân lý; mọi thay đổi tồn đều phải có chứng từ tương ứng.

---

## Bài 4.3 – Tóm tắt luồng và liên kết chứng từ (tự luận / thảo luận)

**Mục tiêu:** Củng cố kiến thức về liên kết giữa các DocType.

**Yêu cầu:**

1. Vẽ (hoặc liệt kê) **sơ đồ luồng** từ Material Request đến Payment Entry (mua hàng) và từ Quotation đến Payment Entry (bán hàng). Ghi rõ trên mỗi mũi tên: DocType nguồn → DocType đích và điều kiện (ví dụ: “Từ SO tạo DN khi cần giao hàng”).
2. Liệt kê **ít nhất 3 DocType** ảnh hưởng trực tiếp đến **Stock Ledger Entry** (tạo hoặc điều chỉnh dòng tồn). Với mỗi DocType, nêu ảnh hưởng (tăng/giảm tồn, điều chuyển, điều chỉnh).
3. Nêu **sự khác nhau** giữa: (a) Delivery Note và (b) Stock Entry purpose Material Issue trong bối cảnh “xuất hàng ra khỏi kho”. Khi nào dùng cái nào?

**Kiểm tra:**

- Đáp án tham chiếu tài liệu: DAO_TAO_NGHIEP_VU_BAN_HANG_ERPNext.md, DAO_TAO_NGHIEP_VU_MUA_HANG_ERPNext.md, DAO_TAO_NGHIEP_VU_KHO_ERPNext.md.

---

# BẢNG THEO DÕI HOÀN THÀNH

| Bài | Phân hệ | Mô tả ngắn | Hoàn thành |
|-----|---------|------------|------------|
| 1.1 | Selling | Luồng Quotation → SO → DN → SI | ☐ |
| 1.2 | Selling | SO → SI (skip DN) | ☐ |
| 1.3 | Selling | Báo cáo & debug bán hàng | ☐ |
| 2.1 | Buying | MR → PO → PR → PI | ☐ |
| 2.2 | Buying | RFQ → Supplier Quotation → PO | ☐ |
| 2.3 | Buying | Báo cáo & debug mua hàng | ☐ |
| 3.1 | Stock | Warehouse, Stock Balance, Stock Ledger | ☐ |
| 3.2 | Stock | Stock Entry Material Transfer | ☐ |
| 3.3 | Stock | Điều chỉnh tồn (Stock Entry / Reconciliation) | ☐ |
| 3.4 | Stock | Pick List → Delivery Note | ☐ |
| 4.1 | Tích hợp | Mua → Kho → Bán (vòng tròn) | ☐ |
| 4.2 | Tích hợp | Debug “tồn kho sai” | ☐ |
| 4.3 | Tích hợp | Tóm tắt luồng & liên kết chứng từ | ☐ |

---

*Bài tập đào tạo Dev – 3 phân hệ Bán hàng, Mua hàng, Kho (ERPNext)*
