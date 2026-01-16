# Buying Settings - Hướng dẫn đầy đủ

File này giải thích **tất cả các trường (fields) và chức năng của chúng** trong **Buying Settings** của ERPNext.

---

## 📋 Tổng quan

**Buying Settings** là một Single DocType (chỉ có 1 record duy nhất) dùng để cấu hình các thiết lập mặc định và hành vi của module Buying trong ERPNext.

**Vị trí:** Buying > Setup > Buying Settings

---

## 🔷 1. Naming Series and Price Defaults Section

### 1.1. Supplier Naming By (`supp_master_name`)

**Field Type:** Select  
**Default:** "Supplier Name"  
**Options:**
- `Supplier Name` - Sử dụng tên supplier làm ID
- `Naming Series` - Sử dụng naming series (ví dụ: SUP-00001)
- `Auto Name` - Tự động tạo tên

**Chức năng:**
- Xác định cách hệ thống đặt tên (naming) cho Supplier
- Khi chọn "Naming Series", field `supplier_name` sẽ được ẩn và sử dụng naming series
- Được lưu vào `frappe.defaults` để sử dụng toàn hệ thống

**Code:**
```python
# File: buying_settings.py
from erpnext.utilities.naming import set_by_naming_series

set_by_naming_series(
    "Supplier",
    "supplier_name",
    self.get("supp_master_name") == "Naming Series",
    hide_name_field=False,
)
```

**Ví dụ:**
- `Supplier Name`: Supplier có ID = "ABC Supplier"
- `Naming Series`: Supplier có ID = "SUP-00001"
- `Auto Name`: Supplier có ID tự động tạo

---

### 1.2. Default Supplier Group (`supplier_group`)

**Field Type:** Link (Supplier Group)  
**Default:** None

**Chức năng:**
- Thiết lập Supplier Group mặc định khi tạo Supplier mới
- Được lưu vào `frappe.defaults` để sử dụng toàn hệ thống
- Có thể override khi tạo Supplier cụ thể

**Ví dụ:**
- Default Supplier Group = "Raw Material Suppliers"
- Khi tạo Supplier mới, field "Supplier Group" sẽ tự động = "Raw Material Suppliers"

---

### 1.3. Default Buying Price List (`buying_price_list`)

**Field Type:** Link (Price List)  
**Default:** None

**Chức năng:**
- Thiết lập Price List mặc định cho các Purchase Transactions (Request for Quotation, Supplier Quotation, Purchase Order, Purchase Invoice, Purchase Receipt)
- Được lưu vào `frappe.defaults` để sử dụng toàn hệ thống
- Có thể override trong từng transaction

**Ví dụ:**
- Default Buying Price List = "Standard Buying"
- Khi tạo Purchase Order mới, field "Price List" sẽ tự động = "Standard Buying"

---

### 1.4. Maintain Same Rate Throughout the Purchase Cycle (`maintain_same_rate`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, hệ thống sẽ **bắt buộc** rate (giá) phải giống nhau trong toàn bộ purchase cycle
- Purchase cycle: Request for Quotation → Supplier Quotation → Purchase Order → Purchase Receipt → Purchase Invoice
- Nếu rate khác nhau, hệ thống sẽ thực hiện action được cấu hình trong `maintain_same_rate_action`

**Logic:**
```python
# File: erpnext/utilities/transaction_base.py

def validate_rate_with_reference_doc(self, ref_details):
    buying_doctypes = ["Purchase Order", "Purchase Invoice", "Purchase Receipt"]
    
    if self.doctype in buying_doctypes:
        action, role_allowed_to_override = frappe.get_cached_value(
            "Buying Settings", "None", 
            ["maintain_same_rate_action", "role_to_override_stop_action"]
        )
        
        # So sánh rate với reference document
        if abs(flt(d.rate - ref_rate, d.precision("rate"))) >= 0.01:
            if action == "Stop":
                # Dừng và báo lỗi
                frappe.throw(...)
            else:
                # Cảnh báo nhưng vẫn cho phép
                frappe.msgprint(..., indicator="orange")
```

**Ví dụ:**
- Purchase Order: Item A, Rate = 100
- Purchase Receipt: Item A, Rate = 100 ✅ (giống nhau)
- Purchase Invoice: Item A, Rate = 120 ❌ (khác nhau → báo lỗi hoặc cảnh báo)

**Lưu ý:** Khi `maintain_same_rate` = 1, `set_landed_cost_based_on_purchase_invoice_rate` sẽ tự động = 0

---

### 1.5. Action If Same Rate is Not Maintained (`maintain_same_rate_action`)

**Field Type:** Select  
**Default:** "Stop"  
**Options:**
- `Stop` - Dừng và báo lỗi, không cho phép lưu
- `Warn` - Cảnh báo nhưng vẫn cho phép lưu

**Dependency:** Chỉ hiển thị khi `maintain_same_rate` = 1

**Chức năng:**
- Xác định hành động khi rate không giống nhau trong purchase cycle
- `Stop`: Ngăn chặn việc lưu transaction nếu rate khác
- `Warn`: Hiển thị cảnh báo nhưng vẫn cho phép lưu

**Ví dụ:**
- `maintain_same_rate` = 1
- `maintain_same_rate_action` = "Stop"
- Khi rate khác → Hệ thống sẽ **throw error** và không cho lưu

---

### 1.6. Role Allowed to Override Stop Action (`role_to_override_stop_action`)

**Field Type:** Link (Role)  
**Default:** None

**Dependency:** Chỉ hiển thị khi `maintain_same_rate` = 1 VÀ `maintain_same_rate_action` = "Stop"

**Chức năng:**
- Cho phép role cụ thể **override** action "Stop"
- User có role này vẫn có thể lưu transaction dù rate khác nhau
- Các user khác vẫn bị chặn

**Code:**
```python
# File: erpnext/utilities/transaction_base.py

if action == "Stop":
    if role_allowed_to_override not in frappe.get_roles():
        # User không có role override → báo lỗi
        stop_actions.append(...)
    # User có role override → cho phép
```

**Ví dụ:**
- `role_to_override_stop_action` = "Purchase Manager"
- User có role "Purchase Manager" → Có thể lưu dù rate khác
- User không có role "Purchase Manager" → Bị chặn

---

## 🔷 2. Transaction Settings Section

### 2.1. Is Purchase Order Required for Purchase Invoice & Receipt Creation? (`po_required`)

**Field Type:** Select  
**Default:** "No"  
**Options:**
- `No` - Không bắt buộc
- `Yes` - Bắt buộc

**Chức năng:**
- Khi "Yes", **bắt buộc** phải có Purchase Order trước khi tạo Purchase Invoice hoặc Purchase Receipt
- Khi "No", có thể tạo Purchase Invoice hoặc Purchase Receipt trực tiếp (không cần Purchase Order)

**Ví dụ:**
- `po_required` = "Yes" → Phải tạo Purchase Order trước, sau đó mới tạo Purchase Invoice
- `po_required` = "No" → Có thể tạo Purchase Invoice trực tiếp

---

### 2.2. Is Purchase Receipt Required for Purchase Invoice Creation? (`pr_required`)

**Field Type:** Select  
**Default:** "No"  
**Options:**
- `No` - Không bắt buộc
- `Yes` - Bắt buộc

**Chức năng:**
- Khi "Yes", **bắt buộc** phải có Purchase Receipt trước khi tạo Purchase Invoice
- Khi "No", có thể tạo Purchase Invoice trực tiếp (không cần Purchase Receipt)

**Ví dụ:**
- `pr_required` = "Yes" → Phải tạo Purchase Receipt trước, sau đó mới tạo Purchase Invoice
- `pr_required` = "No" → Có thể tạo Purchase Invoice trực tiếp

---

### 2.3. Blanket Order Allowance (%) (`blanket_order_allowance`)

**Field Type:** Float  
**Default:** 0

**Chức năng:**
- Xác định **phần trăm** được phép mua vượt quá số lượng trong Blanket Order
- Hữu ích khi cần linh hoạt trong việc thay đổi số lượng đặt hàng

**Ví dụ:**
- Blanket Order: Quantity = 1000
- `blanket_order_allowance` = 10%
- Số lượng tối đa có thể mua = 1000 + (1000 * 10%) = 1100

---

### 2.4. Update frequency of Project (`project_update_frequency`)

**Field Type:** Select  
**Default:** "Each Transaction"  
**Options:**
- `Each Transaction` - Cập nhật mỗi transaction
- `Manual` - Cập nhật thủ công

**Chức năng:**
- Xác định tần suất cập nhật Project dựa trên Total Purchase Cost
- Ảnh hưởng đến hiệu suất hệ thống (cập nhật thường xuyên hơn = chậm hơn)

**Ví dụ:**
- `project_update_frequency` = "Each Transaction" → Cập nhật mỗi khi có Purchase Order/Purchase Invoice
- `project_update_frequency` = "Manual" → Phải cập nhật thủ công

---

### 2.5. Allow Item To Be Added Multiple Times in a Transaction (`allow_multiple_items`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép **thêm cùng một item nhiều lần** trong cùng một transaction
- Khi disabled, mỗi item chỉ có thể xuất hiện 1 lần (nếu thêm lại sẽ merge vào dòng hiện có)

**Ví dụ:**
- `allow_multiple_items` = 0 → Item A chỉ có thể thêm 1 lần
- `allow_multiple_items` = 1 → Item A có thể thêm nhiều lần (ví dụ: 2 dòng với warehouse khác nhau)

---

### 2.6. Bill for Rejected Quantity in Purchase Invoice (`bill_for_rejected_quantity_in_purchase_invoice`)

**Field Type:** Check  
**Default:** 1 (True)

**Chức năng:**
- Khi enabled, **Rejected Quantity** sẽ được **bao gồm** khi tạo Purchase Invoice từ Purchase Receipt
- Khi disabled, Rejected Quantity sẽ không được tính vào Purchase Invoice

**Code:**
```python
# File: buying_settings.py

def validate(self):
    if not self.bill_for_rejected_quantity_in_purchase_invoice:
        self.set_valuation_rate_for_rejected_materials = 0
```

**Ví dụ:**
- Purchase Receipt: Item A, Qty = 100, Rejected Qty = 10
- `bill_for_rejected_quantity_in_purchase_invoice` = 1 → Purchase Invoice sẽ có Qty = 100 (bao gồm rejected)
- `bill_for_rejected_quantity_in_purchase_invoice` = 0 → Purchase Invoice sẽ có Qty = 90 (không bao gồm rejected)

---

### 2.7. Set Valuation Rate for Rejected Materials (`set_valuation_rate_for_rejected_materials`)

**Field Type:** Check  
**Default:** 0 (False)

**Dependency:** Chỉ hiển thị khi `bill_for_rejected_quantity_in_purchase_invoice` = 1

**Chức năng:**
- Khi enabled, hệ thống sẽ tạo **accounting entry** cho materials rejected trong Purchase Receipt
- Cho phép theo dõi chi phí của rejected materials trong kế toán

**Ví dụ:**
- `bill_for_rejected_quantity_in_purchase_invoice` = 1
- `set_valuation_rate_for_rejected_materials` = 1 → Tạo GL Entry cho rejected materials
- `set_valuation_rate_for_rejected_materials` = 0 → Không tạo GL Entry

---

### 2.8. Disable Last Purchase Rate (`disable_last_purchase_rate`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, hệ thống sẽ **không tự động điền** Last Purchase Rate khi thêm item vào Purchase Order/Purchase Invoice
- Khi disabled, hệ thống sẽ tự động điền Last Purchase Rate từ lần mua trước

**Ví dụ:**
- Item A: Last Purchase Rate = 100
- `disable_last_purchase_rate` = 0 → Khi thêm Item A, rate tự động = 100
- `disable_last_purchase_rate` = 1 → Khi thêm Item A, rate = 0 hoặc null (phải nhập thủ công)

---

### 2.9. Show Pay Button in Purchase Order Portal (`show_pay_button`)

**Field Type:** Check  
**Default:** 1 (True)

**Dependency:** Chỉ hiển thị khi Payments app được cài đặt

**Chức năng:**
- Khi enabled, hiển thị nút **"Pay"** trong Purchase Order Portal (supplier portal)
- Cho phép supplier thanh toán trực tiếp từ portal

**Ví dụ:**
- `show_pay_button` = 1 → Supplier có thể thấy và click nút "Pay" trong portal
- `show_pay_button` = 0 → Nút "Pay" bị ẩn

---

### 2.10. Set Landed Cost Based on Purchase Invoice Rate (`set_landed_cost_based_on_purchase_invoice_rate`)

**Field Type:** Check  
**Default:** 0 (False)

**Dependency:** Chỉ hiển thị khi `maintain_same_rate` = 0

**Chức năng:**
- Khi enabled, hệ thống sẽ **điều chỉnh** incoming rate (được set trong Purchase Receipt) dựa trên Purchase Invoice rate
- Hữu ích khi rate trong Purchase Invoice khác với Purchase Receipt

**Lưu ý:** Không thể enable cùng lúc với `maintain_same_rate`

**Ví dụ:**
- Purchase Receipt: Item A, Rate = 100
- Purchase Invoice: Item A, Rate = 120
- `set_landed_cost_based_on_purchase_invoice_rate` = 1 → Incoming rate sẽ được cập nhật thành 120
- `set_landed_cost_based_on_purchase_invoice_rate` = 0 → Incoming rate vẫn là 100

---

### 2.11. Use Transaction Date Exchange Rate (`use_transaction_date_exchange_rate`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, sử dụng **Exchange Rate** trên transaction date của Purchase Invoice thay vì kế thừa từ Purchase Order
- Chỉ áp dụng cho Purchase Invoice
- Hữu ích khi exchange rate thay đổi giữa Purchase Order và Purchase Invoice

**Ví dụ:**
- Purchase Order: Date = 2024-01-01, Exchange Rate = 1 USD = 24,000 VND
- Purchase Invoice: Date = 2024-01-15, Exchange Rate = 1 USD = 24,500 VND
- `use_transaction_date_exchange_rate` = 1 → Sử dụng rate 24,500 VND (từ invoice date)
- `use_transaction_date_exchange_rate` = 0 → Sử dụng rate 24,000 VND (từ purchase order)

---

### 2.12. Allow Purchase Order with Zero Quantity (`allow_zero_qty_in_purchase_order`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép submit **Purchase Order với quantity = 0**
- Hữu ích cho các trường hợp như Rate Contracts (hợp đồng giá cố định nhưng số lượng chưa xác định)

**Ví dụ:**
- `allow_zero_qty_in_purchase_order` = 1 → Có thể submit Purchase Order với Item có Qty = 0
- `allow_zero_qty_in_purchase_order` = 0 → Không thể submit Purchase Order nếu có Item có Qty = 0

---

### 2.13. Allow Request for Quotation with Zero Quantity (`allow_zero_qty_in_request_for_quotation`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép submit **Request for Quotation với quantity = 0**
- Hữu ích cho các trường hợp như Rate Contracts

**Ví dụ:**
- `allow_zero_qty_in_request_for_quotation` = 1 → Có thể submit RFQ với Item có Qty = 0
- `allow_zero_qty_in_request_for_quotation` = 0 → Không thể submit RFQ nếu có Item có Qty = 0

---

### 2.14. Allow Supplier Quotation with Zero Quantity (`allow_zero_qty_in_supplier_quotation`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép submit **Supplier Quotation với quantity = 0**
- Hữu ích cho các trường hợp như Rate Contracts

**Ví dụ:**
- `allow_zero_qty_in_supplier_quotation` = 1 → Có thể submit Supplier Quotation với Item có Qty = 0
- `allow_zero_qty_in_supplier_quotation` = 0 → Không thể submit Supplier Quotation nếu có Item có Qty = 0

---

## 🔷 3. Subcontracting Settings Section

### 3.1. Backflush Raw Materials of Subcontract Based On (`backflush_raw_materials_of_subcontract_based_on`)

**Field Type:** Select  
**Default:** "BOM"  
**Options:**
- `BOM` - Dựa trên BOM (Bill of Materials)
- `Material Transferred for Subcontract` - Dựa trên số lượng material đã transfer

**Chức năng:**
- Xác định cách hệ thống **backflush** (trừ kho) raw materials trong subcontracting
- `BOM`: Trừ kho dựa trên BOM của finished good
- `Material Transferred for Subcontract`: Trừ kho dựa trên số lượng material đã transfer thực tế

**Ví dụ:**
- Finished Good: Computer (BOM: 1 Monitor, 1 Keyboard, 1 Mouse)
- `backflush_raw_materials_of_subcontract_based_on` = "BOM" → Trừ kho: 1 Monitor, 1 Keyboard, 1 Mouse (theo BOM)
- `backflush_raw_materials_of_subcontract_based_on` = "Material Transferred for Subcontract" → Trừ kho theo số lượng đã transfer thực tế

---

### 3.2. Over Transfer Allowance (%) (`over_transfer_allowance`)

**Field Type:** Float  
**Default:** 0

**Dependency:** Chỉ hiển thị khi `backflush_raw_materials_of_subcontract_based_on` = "BOM"

**Chức năng:**
- Xác định **phần trăm** được phép transfer vượt quá số lượng đã order
- Hữu ích khi cần linh hoạt trong việc transfer materials

**Ví dụ:**
- Purchase Order: Quantity = 100 units
- `over_transfer_allowance` = 10%
- Số lượng tối đa có thể transfer = 100 + (100 * 10%) = 110 units

---

### 3.3. Validate Consumed Qty (as per BOM) (`validate_consumed_qty`)

**Field Type:** Check  
**Default:** 0 (False)

**Dependency:** Chỉ hiển thị khi `backflush_raw_materials_of_subcontract_based_on` = "Material Transferred for Subcontract"

**Chức năng:**
- Khi enabled, hệ thống sẽ **validate** raw materials consumed quantity dựa trên FG BOM required quantity
- Đảm bảo số lượng consumed không vượt quá số lượng cần thiết theo BOM

**Ví dụ:**
- Finished Good: Computer (BOM: 1 Monitor, 1 Keyboard, 1 Mouse)
- Material Transferred: 2 Monitors, 1 Keyboard, 1 Mouse
- `validate_consumed_qty` = 1 → Validate: Consumed qty phải = BOM required qty (1 Monitor)
- `validate_consumed_qty` = 0 → Không validate

---

### 3.4. Auto Create Subcontracting Order (`auto_create_subcontracting_order`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **Subcontracting Order (Draft)** sẽ được **tự động tạo** khi submit Purchase Order
- Giảm thiểu công việc thủ công

**Ví dụ:**
- Purchase Order: Item = Finished Good (subcontracted)
- `auto_create_subcontracting_order` = 1 → Tự động tạo Subcontracting Order (Draft) khi submit PO
- `auto_create_subcontracting_order` = 0 → Phải tạo Subcontracting Order thủ công

---

### 3.5. Auto Create Purchase Receipt (`auto_create_purchase_receipt`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **Purchase Receipt (Draft)** sẽ được **tự động tạo** khi submit Subcontracting Receipt
- Giảm thiểu công việc thủ công

**Ví dụ:**
- Subcontracting Receipt: Finished Good received
- `auto_create_purchase_receipt` = 1 → Tự động tạo Purchase Receipt (Draft) khi submit Subcontracting Receipt
- `auto_create_purchase_receipt` = 0 → Phải tạo Purchase Receipt thủ công

---

## 🔷 4. Request for Quotation Section

### 4.1. Fixed Outgoing Email Account (`fixed_email`)

**Field Type:** Link (Email Account)  
**Default:** None

**Chức năng:**
- Khi set, hệ thống sẽ **không sử dụng** user's Email hoặc standard outgoing Email account để gửi Request for Quotations
- Thay vào đó, sử dụng Email Account được chỉ định
- Hữu ích khi muốn dùng một email account cố định cho tất cả RFQ

**Ví dụ:**
- `fixed_email` = "purchasing@company.com"
- Khi gửi RFQ, hệ thống sẽ dùng "purchasing@company.com" thay vì user's email

---

## 🔷 5. Các Methods trong buying_settings.py

### 5.1. `validate()`

**Chức năng:**
- Validate và lưu các settings vào `frappe.defaults`
- Set naming series cho Supplier
- Validate `bill_for_rejected_quantity_in_purchase_invoice`

**Code:**
```python
def validate(self):
    # Lưu vào defaults
    for key in ["supplier_group", "supp_master_name", "maintain_same_rate", "buying_price_list"]:
        frappe.db.set_default(key, self.get(key, ""))
    
    # Set naming series
    set_by_naming_series(
        "Supplier",
        "supplier_name",
        self.get("supp_master_name") == "Naming Series",
        hide_name_field=False,
    )
    
    # Validate rejected quantity
    if not self.bill_for_rejected_quantity_in_purchase_invoice:
        self.set_valuation_rate_for_rejected_materials = 0
```

---

### 5.2. `before_save()`

**Chức năng:**
- Kiểm tra và validate `maintain_same_rate` trước khi save

---

### 5.3. `check_maintain_same_rate()`

**Chức năng:**
- Nếu `maintain_same_rate` = 1, tự động set `set_landed_cost_based_on_purchase_invoice_rate` = 0
- Đảm bảo không conflict giữa 2 settings

**Code:**
```python
def check_maintain_same_rate(self):
    if self.maintain_same_rate:
        self.set_landed_cost_based_on_purchase_invoice_rate = 0
```

---

## 📝 Tóm tắt

### Các nhóm settings chính:

1. **Naming Series and Price Defaults** - Thiết lập mặc định cho Supplier và Price List
2. **Transaction Settings** - Cấu hình hành vi của transactions
3. **Subcontracting Settings** - Cấu hình cho Subcontracting
4. **Request for Quotation** - Cấu hình cho RFQ

### Các settings quan trọng nhất:

1. **`maintain_same_rate`** - Duy trì giá đồng nhất trong purchase cycle
2. **`po_required`** - Bắt buộc Purchase Order
3. **`pr_required`** - Bắt buộc Purchase Receipt
4. **`buying_price_list`** - Price List mặc định
5. **`supplier_group`** - Supplier Group mặc định
6. **`bill_for_rejected_quantity_in_purchase_invoice`** - Có tính rejected quantity vào invoice không

### So sánh với Selling Settings:

| Feature | Buying Settings | Selling Settings |
|---------|----------------|-----------------|
| **Naming** | Supplier Naming | Customer Naming |
| **Default Group** | Supplier Group | Customer Group |
| **Default Price List** | Buying Price List | Selling Price List |
| **Maintain Same Rate** | Purchase Cycle | Sales Cycle |
| **Required Documents** | PO, PR | SO, DN |
| **Special Features** | Subcontracting, Rejected Qty | Discount Accounting, Tax ID |

---

## 🔗 Tài liệu tham khảo

- **File source:**
  - `apps/erpnext/erpnext/buying/doctype/buying_settings/buying_settings.json`
  - `apps/erpnext/erpnext/buying/doctype/buying_settings/buying_settings.py`
  - `apps/erpnext/erpnext/utilities/transaction_base.py` (validate_rate_with_reference_doc)
