# Selling Settings - Hướng dẫn đầy đủ

File này giải thích **tất cả các trường (fields) và chức năng của chúng** trong **Selling Settings** của ERPNext.

---

## 📋 Tổng quan

**Selling Settings** là một Single DocType (chỉ có 1 record duy nhất) dùng để cấu hình các thiết lập mặc định và hành vi của module Selling trong ERPNext.

**Vị trí:** Selling > Setup > Selling Settings

---

## 🔷 1. Customer Defaults Section

### 1.1. Customer Naming By (`cust_master_name`)

**Field Type:** Select  
**Default:** "Customer Name"  
**Options:**
- `Customer Name` - Sử dụng tên customer làm ID
- `Naming Series` - Sử dụng naming series (ví dụ: CUST-00001)
- `Auto Name` - Tự động tạo tên

**Chức năng:**
- Xác định cách hệ thống đặt tên (naming) cho Customer
- Khi chọn "Naming Series", field `customer_name` sẽ được ẩn và sử dụng naming series
- Được lưu vào `frappe.defaults` để sử dụng toàn hệ thống

**Code:**
```python
# File: selling_settings.py
from erpnext.utilities.naming import set_by_naming_series

set_by_naming_series(
    "Customer",
    "customer_name",
    self.get("cust_master_name") == "Naming Series",
    hide_name_field=False,
)
```

**Ví dụ:**
- `Customer Name`: Customer có ID = "ABC Company"
- `Naming Series`: Customer có ID = "CUST-00001"
- `Auto Name`: Customer có ID tự động tạo

---

### 1.2. Default Customer Group (`customer_group`)

**Field Type:** Link (Customer Group)  
**Default:** None

**Chức năng:**
- Thiết lập Customer Group mặc định khi tạo Customer mới
- Được lưu vào `frappe.defaults` để sử dụng toàn hệ thống
- Có thể override khi tạo Customer cụ thể

**Ví dụ:**
- Default Customer Group = "Retail"
- Khi tạo Customer mới, field "Customer Group" sẽ tự động = "Retail"

---

### 1.3. Default Territory (`territory`)

**Field Type:** Link (Territory)  
**Default:** None

**Chức năng:**
- Thiết lập Territory mặc định khi tạo Customer mới
- Được lưu vào `frappe.defaults` để sử dụng toàn hệ thống
- Có thể override khi tạo Customer cụ thể

**Ví dụ:**
- Default Territory = "North"
- Khi tạo Customer mới, field "Territory" sẽ tự động = "North"

---

## 🔷 2. Item Price Settings Section

### 2.1. Default Price List (`selling_price_list`)

**Field Type:** Link (Price List)  
**Default:** None

**Chức năng:**
- Thiết lập Price List mặc định cho các Sales Transactions (Quotation, Sales Order, Sales Invoice, Delivery Note)
- Được lưu vào `frappe.defaults` để sử dụng toàn hệ thống
- Có thể override trong từng transaction

**Ví dụ:**
- Default Price List = "Standard Selling"
- Khi tạo Sales Order mới, field "Price List" sẽ tự động = "Standard Selling"

---

### 2.2. Maintain Same Rate Throughout Sales Cycle (`maintain_same_sales_rate`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, hệ thống sẽ **bắt buộc** rate (giá) phải giống nhau trong toàn bộ sales cycle
- Sales cycle: Quotation → Sales Order → Delivery Note → Sales Invoice
- Nếu rate khác nhau, hệ thống sẽ thực hiện action được cấu hình trong `maintain_same_rate_action`

**Logic:**
```python
# File: erpnext/utilities/transaction_base.py

def validate_rate_with_reference_doc(self, ref_details):
    action, role_allowed_to_override = frappe.get_cached_value(
        "Selling Settings", "None", 
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
- Quotation: Item A, Rate = 100
- Sales Order: Item A, Rate = 100 ✅ (giống nhau)
- Sales Invoice: Item A, Rate = 120 ❌ (khác nhau → báo lỗi hoặc cảnh báo)

---

### 2.3. Action if Same Rate is Not Maintained (`maintain_same_rate_action`)

**Field Type:** Select  
**Default:** "Stop"  
**Options:**
- `Stop` - Dừng và báo lỗi, không cho phép lưu
- `Warn` - Cảnh báo nhưng vẫn cho phép lưu

**Dependency:** Chỉ hiển thị khi `maintain_same_sales_rate` = 1

**Chức năng:**
- Xác định hành động khi rate không giống nhau trong sales cycle
- `Stop`: Ngăn chặn việc lưu transaction nếu rate khác
- `Warn`: Hiển thị cảnh báo nhưng vẫn cho phép lưu

**Ví dụ:**
- `maintain_same_sales_rate` = 1
- `maintain_same_rate_action` = "Stop"
- Khi rate khác → Hệ thống sẽ **throw error** và không cho lưu

---

### 2.4. Role Allowed to Override Stop Action (`role_to_override_stop_action`)

**Field Type:** Link (Role)  
**Default:** None

**Dependency:** Chỉ hiển thị khi `maintain_same_sales_rate` = 1 VÀ `maintain_same_rate_action` = "Stop"

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
- `role_to_override_stop_action` = "Sales Manager"
- User có role "Sales Manager" → Có thể lưu dù rate khác
- User không có role "Sales Manager" → Bị chặn

---

### 2.5. Use Prices from Default Price List as Fallback (`fallback_to_default_price_list`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, nếu không tìm thấy giá trong Price List hiện tại, hệ thống sẽ **tự động lấy giá từ Default Price List**
- Hữu ích khi có nhiều Price Lists nhưng muốn đảm bảo luôn có giá

**Validation:**
```python
# File: selling_settings.py

def validate_fallback_to_default_price_list(self):
    if (
        self.fallback_to_default_price_list
        and frappe.get_single_value("Stock Settings", "auto_insert_price_list_rate_if_missing")
    ):
        # Cảnh báo nếu cả 2 settings đều enabled
        frappe.msgprint(
            "This can lead to prices from the default price list being inserted into the transaction price list."
        )
```

**Ví dụ:**
- Default Price List = "Standard Selling"
- Transaction Price List = "Special Price List"
- Item A không có trong "Special Price List"
- Nếu `fallback_to_default_price_list` = 1 → Lấy giá từ "Standard Selling"
- Nếu `fallback_to_default_price_list` = 0 → Giá = 0 hoặc null

---

### 2.6. Allow User to Edit Price List Rate in Transactions (`editable_price_list_rate`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, user có thể **chỉnh sửa** rate từ Price List trong transactions
- Khi disabled, rate từ Price List là **read-only** (chỉ đọc)

**Code:**
```python
# File: selling_settings.py

def validate(self):
    frappe.db.set_default("editable_price_list_rate", self.get("editable_price_list_rate", ""))
```

**Ví dụ:**
- `editable_price_list_rate` = 0 → Rate field là read-only
- `editable_price_list_rate` = 1 → User có thể sửa rate

---

### 2.7. Validate Selling Price for Item Against Purchase Rate or Valuation Rate (`validate_selling_price`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, hệ thống sẽ **validate** selling price (giá bán) phải lớn hơn hoặc bằng purchase rate (giá mua) hoặc valuation rate (giá trị kho)
- Ngăn chặn việc bán với giá thấp hơn giá mua (trừ khi có lý do đặc biệt)

**Ví dụ:**
- Item A: Purchase Rate = 100, Valuation Rate = 100
- Selling Price = 80 ❌ (nếu `validate_selling_price` = 1 → báo lỗi)
- Selling Price = 120 ✅

---

### 2.8. Calculate Product Bundle Price based on Child Items' Rates (`editable_bundle_item_rates`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, giá của Product Bundle sẽ được **tính toán** dựa trên giá của các child items
- Khi disabled, giá của Product Bundle là **read-only** và phải set thủ công

**Code:**
```python
# File: selling_settings.py

def toggle_editable_rate_for_bundle_items(self):
    editable_bundle_item_rates = cint(self.editable_bundle_item_rates)
    
    make_property_setter(
        "Packed Item",
        "rate",
        "read_only",
        not (editable_bundle_item_rates),  # Nếu enabled → read_only = False
        "Check",
        validate_fields_for_doctype=False,
    )
```

**Ví dụ:**
- Product Bundle "Computer Set" gồm: Monitor (100), Keyboard (20), Mouse (10)
- `editable_bundle_item_rates` = 1 → Bundle rate = 130 (tự động tính)
- `editable_bundle_item_rates` = 0 → Bundle rate phải set thủ công

---

### 2.9. Allow Negative rates for Items (`allow_negative_rates_for_items`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép nhập **rate âm** (negative rate) cho items
- Hữu ích cho các trường hợp đặc biệt như credit notes, adjustments

**Ví dụ:**
- `allow_negative_rates_for_items` = 0 → Rate phải >= 0
- `allow_negative_rates_for_items` = 1 → Rate có thể < 0 (ví dụ: -50)

---

## 🔷 3. Transaction Settings Section

### 3.1. Is Sales Order Required for Sales Invoice & Delivery Note Creation? (`so_required`)

**Field Type:** Select  
**Default:** "No"  
**Options:**
- `No` - Không bắt buộc
- `Yes` - Bắt buộc

**Chức năng:**
- Khi "Yes", **bắt buộc** phải có Sales Order trước khi tạo Sales Invoice hoặc Delivery Note
- Khi "No", có thể tạo Sales Invoice hoặc Delivery Note trực tiếp (không cần Sales Order)

**Ví dụ:**
- `so_required` = "Yes" → Phải tạo Sales Order trước, sau đó mới tạo Sales Invoice
- `so_required` = "No" → Có thể tạo Sales Invoice trực tiếp

---

### 3.2. Is Delivery Note Required for Sales Invoice Creation? (`dn_required`)

**Field Type:** Select  
**Default:** "No"  
**Options:**
- `No` - Không bắt buộc
- `Yes` - Bắt buộc

**Chức năng:**
- Khi "Yes", **bắt buộc** phải có Delivery Note trước khi tạo Sales Invoice
- Khi "No", có thể tạo Sales Invoice trực tiếp (không cần Delivery Note)

**Ví dụ:**
- `dn_required` = "Yes" → Phải tạo Delivery Note trước, sau đó mới tạo Sales Invoice
- `dn_required` = "No" → Có thể tạo Sales Invoice trực tiếp

---

### 3.3. Sales Update Frequency in Company and Project (`sales_update_frequency`)

**Field Type:** Select  
**Default:** "Daily"  
**Options:**
- `Monthly` - Cập nhật hàng tháng
- `Each Transaction` - Cập nhật mỗi transaction
- `Daily` - Cập nhật hàng ngày

**Chức năng:**
- Xác định tần suất cập nhật Project và Company dựa trên Sales Transactions
- Ảnh hưởng đến hiệu suất hệ thống (cập nhật thường xuyên hơn = chậm hơn)

**Ví dụ:**
- `sales_update_frequency` = "Each Transaction" → Cập nhật mỗi khi có Sales Order/Sales Invoice
- `sales_update_frequency` = "Daily" → Cập nhật 1 lần/ngày (qua scheduled job)
- `sales_update_frequency` = "Monthly" → Cập nhật 1 lần/tháng

---

### 3.4. Blanket Order Allowance (%) (`blanket_order_allowance`)

**Field Type:** Float  
**Default:** 0

**Chức năng:**
- Xác định **phần trăm** được phép bán vượt quá số lượng trong Blanket Order
- Hữu ích khi cần linh hoạt trong việc thay đổi số lượng đặt hàng

**Ví dụ:**
- Blanket Order: Quantity = 1000
- `blanket_order_allowance` = 10%
- Số lượng tối đa có thể bán = 1000 + (1000 * 10%) = 1100

---

### 3.5. Allow Item to be Added Multiple Times in a Transaction (`allow_multiple_items`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép **thêm cùng một item nhiều lần** trong cùng một transaction
- Khi disabled, mỗi item chỉ có thể xuất hiện 1 lần (nếu thêm lại sẽ merge vào dòng hiện có)

**Ví dụ:**
- `allow_multiple_items` = 0 → Item A chỉ có thể thêm 1 lần
- `allow_multiple_items` = 1 → Item A có thể thêm nhiều lần (ví dụ: 2 dòng với warehouse khác nhau)

---

### 3.6. Allow Multiple Sales Orders Against a Customer's Purchase Order (`allow_against_multiple_purchase_orders`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép tạo **nhiều Sales Orders** từ cùng một Customer's Purchase Order
- Khi disabled, mỗi Customer's Purchase Order chỉ có thể tạo 1 Sales Order

**Ví dụ:**
- Customer's Purchase Order: PO-001
- `allow_against_multiple_purchase_orders` = 0 → Chỉ tạo được 1 Sales Order từ PO-001
- `allow_against_multiple_purchase_orders` = 1 → Có thể tạo nhiều Sales Orders từ PO-001

---

### 3.7. Allow Sales Order Creation For Expired Quotation (`allow_sales_order_creation_for_expired_quotation`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép tạo Sales Order từ **Quotation đã hết hạn** (expired)
- Khi disabled, chỉ cho phép tạo Sales Order từ Quotation còn hiệu lực

**Ví dụ:**
- Quotation: Valid Until = 2024-01-01 (đã hết hạn)
- `allow_sales_order_creation_for_expired_quotation` = 0 → Không thể tạo Sales Order
- `allow_sales_order_creation_for_expired_quotation` = 1 → Vẫn có thể tạo Sales Order

---

### 3.8. Don't Reserve Sales Order Qty on Sales Return (`dont_reserve_sales_order_qty_on_sales_return`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, khi tạo Sales Return (Credit Note), hệ thống sẽ **không reserve** lại quantity trong Sales Order
- Khi disabled, quantity trong Sales Order sẽ được reserve lại khi có Sales Return

**Ví dụ:**
- Sales Order: Item A, Qty = 100
- Delivery Note: Item A, Qty = 100 (đã deliver)
- Sales Return: Item A, Qty = 20 (return lại)
- `dont_reserve_sales_order_qty_on_sales_return` = 0 → Reserve lại 20 vào Sales Order
- `dont_reserve_sales_order_qty_on_sales_return` = 1 → Không reserve lại

---

### 3.9. Hide Customer's Tax ID from Sales Transactions (`hide_tax_id`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **ẩn** field "Tax ID" của Customer trong các Sales Transactions
- Áp dụng cho: Sales Order, Sales Invoice, Delivery Note

**Code:**
```python
# File: selling_settings.py

def toggle_hide_tax_id(self):
    _hide_tax_id = cint(self.hide_tax_id)
    
    for doctype in ("Sales Order", "Sales Invoice", "Delivery Note"):
        make_property_setter(
            doctype, "tax_id", "hidden", _hide_tax_id, "Check", ...
        )
        make_property_setter(
            doctype, "tax_id", "print_hide", _hide_tax_id, "Check", ...
        )
```

**Ví dụ:**
- `hide_tax_id` = 1 → Field "Tax ID" bị ẩn trong Sales Order, Sales Invoice, Delivery Note
- `hide_tax_id` = 0 → Field "Tax ID" hiển thị bình thường

---

### 3.10. Enable Discount Accounting for Selling (`enable_discount_accounting`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, hệ thống sẽ tạo **additional ledger entries** cho discounts trong một Discount Account riêng
- Cho phép theo dõi chi tiết các khoản giảm giá trong kế toán

**Code:**
```python
# File: selling_settings.py

def toggle_discount_accounting_fields(self):
    enable_discount_accounting = cint(self.enable_discount_accounting)
    
    # Hiển thị/ẩn discount_account field trong Sales Invoice Item
    make_property_setter(
        "Sales Invoice Item",
        "discount_account",
        "hidden",
        not (enable_discount_accounting),
        "Check",
        ...
    )
    
    # Hiển thị/ẩn additional_discount_account field trong Sales Invoice
    make_property_setter(
        "Sales Invoice",
        "additional_discount_account",
        "hidden",
        not (enable_discount_accounting),
        "Check",
        ...
    )
```

**Ví dụ:**
- `enable_discount_accounting` = 1 → Có field "Discount Account" trong Sales Invoice Item và "Additional Discount Account" trong Sales Invoice
- `enable_discount_accounting` = 0 → Các field này bị ẩn

---

### 3.11. Enable Cut-Off Date on Bulk Delivery Note Creation (`enable_cutoff_date_on_bulk_delivery_note_creation`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép set **cut-off date** khi tạo Delivery Note hàng loạt (bulk)
- Hữu ích để kiểm soát thời gian giao hàng trong batch processing

**Ví dụ:**
- `enable_cutoff_date_on_bulk_delivery_note_creation` = 1 → Có field "Cut-Off Date" khi tạo Delivery Note hàng loạt
- `enable_cutoff_date_on_bulk_delivery_note_creation` = 0 → Không có field này

---

### 3.12. Allow Quotation with Zero Quantity (`allow_zero_qty_in_quotation`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép submit **Quotation với quantity = 0**
- Hữu ích cho các trường hợp như Rate Contracts (hợp đồng giá cố định nhưng số lượng chưa xác định)

**Ví dụ:**
- `allow_zero_qty_in_quotation` = 1 → Có thể submit Quotation với Item có Qty = 0
- `allow_zero_qty_in_quotation` = 0 → Không thể submit Quotation nếu có Item có Qty = 0

---

### 3.13. Allow Sales Order with Zero Quantity (`allow_zero_qty_in_sales_order`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép submit **Sales Order với quantity = 0**
- Hữu ích cho các trường hợp như Rate Contracts (hợp đồng giá cố định nhưng số lượng chưa xác định)

**Ví dụ:**
- `allow_zero_qty_in_sales_order` = 1 → Có thể submit Sales Order với Item có Qty = 0
- `allow_zero_qty_in_sales_order` = 0 → Không thể submit Sales Order nếu có Item có Qty = 0

---

## 🔷 4. Experimental Section

### 4.1. Use Legacy (Client side) Reactivity (`use_legacy_js_reactivity`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, sử dụng **legacy client-side reactivity** (cách cũ)
- Khi disabled, sử dụng **server-side reactivity** (cách mới)
- Dùng để tương thích ngược với các customizations cũ

**Ví dụ:**
- `use_legacy_js_reactivity` = 1 → Sử dụng cách tính toán cũ (client-side)
- `use_legacy_js_reactivity` = 0 → Sử dụng cách tính toán mới (server-side)

---

## 🔷 5. Subcontracting Inward Settings Section

### 5.1. Allow Delivery of Overproduced Qty (`allow_delivery_of_overproduced_qty`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép deliver **toàn bộ quantity** của finished goods được sản xuất từ Subcontracting Inward Order (kể cả phần vượt quá)
- Khi disabled, chỉ cho phép deliver **đúng số lượng đã order**

**Ví dụ:**
- Subcontracting Inward Order: Ordered Qty = 100
- Produced Qty = 120 (vượt quá 20)
- `allow_delivery_of_overproduced_qty` = 0 → Chỉ deliver được 100
- `allow_delivery_of_overproduced_qty` = 1 → Có thể deliver 120

---

### 5.2. Deliver Scrap Items (`deliver_scrap_items`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **Scrap Items** được tạo từ Finished Good cũng sẽ được thêm vào Stock Entry khi deliver Finished Good đó
- Khi disabled, Scrap Items không được tự động thêm vào

**Ví dụ:**
- Finished Good: Computer (Qty = 10)
- Scrap Items: Broken Parts (Qty = 2)
- `deliver_scrap_items` = 1 → Khi deliver Computer, Broken Parts cũng được thêm vào Stock Entry
- `deliver_scrap_items` = 0 → Broken Parts không được tự động thêm

---

## 🔷 6. Các Methods trong selling_settings.py

### 6.1. `validate()`

**Chức năng:**
- Validate và lưu các settings vào `frappe.defaults`
- Set naming series cho Customer
- Validate `fallback_to_default_price_list`

**Code:**
```python
def validate(self):
    # Lưu vào defaults
    for key in [
        "cust_master_name",
        "customer_group",
        "territory",
        "maintain_same_sales_rate",
        "editable_price_list_rate",
        "selling_price_list",
    ]:
        frappe.db.set_default(key, self.get(key, ""))
    
    # Set naming series
    set_by_naming_series(
        "Customer",
        "customer_name",
        self.get("cust_master_name") == "Naming Series",
        hide_name_field=False,
    )
    
    # Validate fallback
    self.validate_fallback_to_default_price_list()
```

---

### 6.2. `on_update()`

**Chức năng:**
- Toggle các property setters khi settings được update
- Ẩn/hiện các fields dựa trên settings

**Code:**
```python
def on_update(self):
    self.toggle_hide_tax_id()
    self.toggle_editable_rate_for_bundle_items()
    self.toggle_discount_accounting_fields()
```

---

### 6.3. `toggle_hide_tax_id()`

**Chức năng:**
- Ẩn/hiện field "tax_id" trong Sales Order, Sales Invoice, Delivery Note

---

### 6.4. `toggle_editable_rate_for_bundle_items()`

**Chức năng:**
- Set read_only cho field "rate" trong Packed Item dựa trên `editable_bundle_item_rates`

---

### 6.5. `toggle_discount_accounting_fields()`

**Chức năng:**
- Ẩn/hiện và set mandatory cho các discount account fields trong Sales Invoice

---

## 📝 Tóm tắt

### Các nhóm settings chính:

1. **Customer Defaults** - Thiết lập mặc định cho Customer
2. **Item Price Settings** - Cấu hình giá và pricing
3. **Transaction Settings** - Cấu hình hành vi của transactions
4. **Experimental** - Các tính năng thử nghiệm
5. **Subcontracting Inward Settings** - Cấu hình cho Subcontracting

### Các settings quan trọng nhất:

1. **`maintain_same_sales_rate`** - Duy trì giá đồng nhất trong sales cycle
2. **`so_required`** - Bắt buộc Sales Order
3. **`dn_required`** - Bắt buộc Delivery Note
4. **`selling_price_list`** - Price List mặc định
5. **`customer_group`** - Customer Group mặc định

---

## 🔗 Tài liệu tham khảo

- **File source:**
  - `apps/erpnext/erpnext/selling/doctype/selling_settings/selling_settings.json`
  - `apps/erpnext/erpnext/selling/doctype/selling_settings/selling_settings.py`
  - `apps/erpnext/erpnext/utilities/transaction_base.py` (validate_rate_with_reference_doc)
