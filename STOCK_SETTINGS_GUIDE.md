# Stock Settings - Hướng dẫn đầy đủ

File này giải thích **tất cả các trường (fields) và chức năng của chúng** trong **Stock Settings** của ERPNext.

---

## 📋 Tổng quan

**Stock Settings** là một Single DocType (chỉ có 1 record duy nhất) dùng để cấu hình các thiết lập mặc định và hành vi của module Stock trong ERPNext.

**Vị trí:** Stock > Setup > Stock Settings

---

## 🔷 1. Defaults Tab

### 1.1. Item Defaults Section

#### 1.1.1. Item Naming By (`item_naming_by`)

**Field Type:** Select  
**Default:** "Item Code"  
**Options:**
- `Item Code` - Sử dụng Item Code làm ID (nhập thủ công)
- `Naming Series` - Sử dụng naming series (ví dụ: ITEM-00001)

**Chức năng:**
- Xác định cách hệ thống đặt tên (naming) cho Item
- Khi chọn "Naming Series", field `item_code` sẽ được ẩn và sử dụng naming series
- Được lưu vào `frappe.defaults` để sử dụng toàn hệ thống

**Code:**
```python
# File: stock_settings.py
from erpnext.utilities.naming import set_by_naming_series

set_by_naming_series(
    "Item",
    "item_code",
    self.get("item_naming_by") == "Naming Series",
    hide_name_field=True,
    make_mandatory=0,
)
```

**Ví dụ:**
- `Item Code`: Item có ID = "LAPTOP-001" (nhập thủ công)
- `Naming Series`: Item có ID = "ITEM-00001" (tự động)

---

#### 1.1.2. Default Valuation Method (`valuation_method`)

**Field Type:** Select  
**Default:** None  
**Options:**
- `FIFO` - First In First Out (Nhập trước xuất trước)
- `Moving Average` - Bình quân gia quyền
- `LIFO` - Last In First Out (Nhập sau xuất trước)

**Chức năng:**
- Xác định phương pháp tính giá trị tồn kho (valuation) mặc định cho Item
- Mỗi Item có thể có valuation method riêng, nếu không có thì dùng default này
- **Không thể thay đổi** nếu đã có Stock Ledger Entries cho items không có valuation method riêng

**Code:**
```python
# File: stock_settings.py

def cant_change_valuation_method(self):
    previous_valuation_method = doc_before_save.get("valuation_method")
    
    if previous_valuation_method and previous_valuation_method != self.valuation_method:
        # Kiểm tra có Stock Ledger Entries không
        sle = frappe.db.sql("""select name from `tabStock Ledger Entry` sle
            where exists(select name from tabItem
                where name=sle.item_code and (valuation_method is null or valuation_method='')) limit 1""")
        
        if sle:
            frappe.throw("Can't change the valuation method...")
```

**Ví dụ:**
- `FIFO`: Item nhập trước sẽ xuất trước
- `Moving Average`: Tính giá trị trung bình
- `LIFO`: Item nhập sau sẽ xuất trước

---

#### 1.1.3. Default Item Group (`item_group`)

**Field Type:** Link (Item Group)  
**Default:** None

**Chức năng:**
- Thiết lập Item Group mặc định khi tạo Item mới
- Được lưu vào `frappe.defaults` để sử dụng toàn hệ thống
- Có thể override khi tạo Item cụ thể

**Ví dụ:**
- Default Item Group = "Electronics"
- Khi tạo Item mới, field "Item Group" sẽ tự động = "Electronics"

---

#### 1.1.4. Default Warehouse (`default_warehouse`)

**Field Type:** Link (Warehouse)  
**Default:** None

**Chức năng:**
- Thiết lập Warehouse mặc định cho các stock transactions
- Được lưu vào `frappe.defaults` để sử dụng toàn hệ thống
- **Không thể** là Group Warehouse

**Code:**
```python
# File: stock_settings.py

def validate_warehouses(self):
    warehouse_fields = ["default_warehouse", "sample_retention_warehouse"]
    for field in warehouse_fields:
        if frappe.db.get_value("Warehouse", self.get(field), "is_group"):
            frappe.throw("Group Warehouses cannot be used in transactions...")
```

**Ví dụ:**
- Default Warehouse = "Main Warehouse"
- Khi tạo Purchase Receipt, field "Warehouse" sẽ tự động = "Main Warehouse"

---

#### 1.1.5. Sample Retention Warehouse (`sample_retention_warehouse`)

**Field Type:** Link (Warehouse)  
**Default:** None

**Chức năng:**
- Warehouse dùng để lưu trữ samples (mẫu) từ Quality Inspection
- **Không thể** là Group Warehouse
- Hữu ích cho việc quản lý samples riêng biệt

**Ví dụ:**
- Sample Retention Warehouse = "Sample Warehouse"
- Khi có Quality Inspection, samples sẽ được chuyển vào "Sample Warehouse"

---

#### 1.1.6. Default Stock UOM (`stock_uom`)

**Field Type:** Link (UOM)  
**Default:** None

**Chức năng:**
- Thiết lập Unit of Measure (UOM) mặc định cho Item
- Được lưu vào `frappe.defaults` để sử dụng toàn hệ thống
- Có thể override khi tạo Item cụ thể

**Ví dụ:**
- Default Stock UOM = "Nos" (Numbers)
- Khi tạo Item mới, field "Stock UOM" sẽ tự động = "Nos"

---

### 1.2. Price List Defaults Section

#### 1.2.1. Auto Insert Item Price If Missing (`auto_insert_price_list_rate_if_missing`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, hệ thống sẽ **tự động chèn** giá từ Price List nếu giá bị thiếu trong transaction
- Hữu ích để đảm bảo luôn có giá trong transactions

**Validation:**
```python
# File: stock_settings.py

def validate_auto_insert_price_list_rate_if_missing(self):
    if (
        self.auto_insert_price_list_rate_if_missing
        and frappe.get_single_value("Selling Settings", "fallback_to_default_price_list")
    ):
        # Cảnh báo nếu cả 2 settings đều enabled
        frappe.msgprint("This can lead to prices from the default price list being inserted...")
```

**Ví dụ:**
- Transaction: Item A, Rate = null
- `auto_insert_price_list_rate_if_missing` = 1 → Tự động lấy giá từ Price List
- `auto_insert_price_list_rate_if_missing` = 0 → Giá vẫn là null

---

#### 1.2.2. Update Price List Based On (`update_price_list_based_on`)

**Field Type:** Select  
**Default:** "Rate"  
**Options:**
- `Rate` - Dựa trên Rate trong transaction
- `Price List Rate` - Dựa trên Price List Rate trong transaction

**Dependency:** Chỉ hiển thị khi `auto_insert_price_list_rate_if_missing` = 1

**Chức năng:**
- Xác định giá trị nào được dùng để update Price List
- `Rate`: Update Price List dựa trên Rate field
- `Price List Rate`: Update Price List dựa trên Price List Rate field

**Ví dụ:**
- Transaction: Item A, Rate = 100, Price List Rate = 120
- `update_price_list_based_on` = "Rate" → Update Price List với giá 100
- `update_price_list_based_on` = "Price List Rate" → Update Price List với giá 120

---

#### 1.2.3. Update Existing Price List Rate (`update_existing_price_list_rate`)

**Field Type:** Check  
**Default:** 0 (False)

**Dependency:** Chỉ hiển thị khi `auto_insert_price_list_rate_if_missing` = 1

**Chức năng:**
- Khi enabled, hệ thống sẽ **update** giá hiện có trong Price List
- Khi disabled, chỉ insert giá mới nếu chưa có

**Ví dụ:**
- Price List: Item A = 100
- Transaction: Item A, Rate = 120
- `update_existing_price_list_rate` = 1 → Update Price List: Item A = 120
- `update_existing_price_list_rate` = 0 → Giữ nguyên: Item A = 100

---

### 1.3. Stock UOM Quantity Section

#### 1.3.1. Allow to Edit Stock UOM Qty for Sales Documents (`allow_to_edit_stock_uom_qty_for_sales`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép **chỉnh sửa** Stock UOM Quantity trong Sales Documents
- Áp dụng cho: Sales Order, Sales Invoice, Delivery Note, Quotation
- Khi enabled, precision của `conversion_factor` sẽ được set = 9

**Code:**
```python
# File: stock_settings.py

def change_precision_for_for_sales(self):
    if self.allow_to_edit_stock_uom_qty_for_sales:
        doctypes = ["Sales Order Item", "Sales Invoice Item", "Delivery Note Item", "Quotation Item"]
        self.make_property_setter_for_precision(doctypes)
```

**Ví dụ:**
- `allow_to_edit_stock_uom_qty_for_sales` = 0 → Stock UOM Qty là read-only
- `allow_to_edit_stock_uom_qty_for_sales` = 1 → User có thể sửa Stock UOM Qty

---

#### 1.3.2. Allow to Edit Stock UOM Qty for Purchase Documents (`allow_to_edit_stock_uom_qty_for_purchase`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép **chỉnh sửa** Stock UOM Quantity trong Purchase Documents
- Áp dụng cho: Purchase Order, Purchase Receipt, Purchase Invoice, Request for Quotation, Supplier Quotation, Material Request
- Khi enabled, precision của `conversion_factor` sẽ được set = 9

**Ví dụ:**
- `allow_to_edit_stock_uom_qty_for_purchase` = 0 → Stock UOM Qty là read-only
- `allow_to_edit_stock_uom_qty_for_purchase` = 1 → User có thể sửa Stock UOM Qty

---

#### 1.3.3. Allow UOM with Conversion Rate Defined in Item (`allow_uom_with_conversion_rate_defined_in_item`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, hệ thống chỉ cho phép chọn UOMs trong sales và purchase transactions **nếu conversion rate đã được set trong Item master**
- Đảm bảo tính nhất quán trong việc chuyển đổi UOM

**Ví dụ:**
- Item A: UOM = "Nos", Conversion Rate với "Box" = 10
- `allow_uom_with_conversion_rate_defined_in_item` = 1 → Chỉ cho phép chọn "Box" nếu conversion rate đã set
- `allow_uom_with_conversion_rate_defined_in_item` = 0 → Cho phép chọn bất kỳ UOM nào

---

## 🔷 2. Stock Validations Tab

### 2.1. Stock Transactions Settings Section

#### 2.1.1. Over Delivery/Receipt Allowance (%) (`over_delivery_receipt_allowance`)

**Field Type:** Float  
**Default:** 0

**Chức năng:**
- Xác định **phần trăm** được phép deliver hoặc receive vượt quá số lượng đã order
- Hữu ích khi cần linh hoạt trong việc giao/nhận hàng

**Ví dụ:**
- Purchase Order: Quantity = 100 units
- `over_delivery_receipt_allowance` = 10%
- Số lượng tối đa có thể receive = 100 + (100 * 10%) = 110 units

---

#### 2.1.2. Over Transfer Allowance (`mr_qty_allowance`)

**Field Type:** Float  
**Default:** 0

**Chức năng:**
- Xác định **phần trăm** được phép transfer vượt quá số lượng đã order trong Material Request
- Hữu ích khi cần linh hoạt trong việc transfer materials

**Ví dụ:**
- Material Request: Quantity = 100 units
- `mr_qty_allowance` = 10%
- Số lượng tối đa có thể transfer = 100 + (100 * 10%) = 110 units

---

#### 2.1.3. Over Picking Allowance (`over_picking_allowance`)

**Field Type:** Percent  
**Default:** 0

**Chức năng:**
- Xác định **phần trăm** được phép pick nhiều items hơn số lượng đã order trong Pick List
- Hữu ích khi cần linh hoạt trong việc picking

**Ví dụ:**
- Sales Order: Quantity = 100 units
- `over_picking_allowance` = 10%
- Số lượng tối đa có thể pick = 100 + (100 * 10%) = 110 units

---

#### 2.1.4. Role Allowed to Over Deliver/Receive (`role_allowed_to_over_deliver_receive`)

**Field Type:** Link (Role)  
**Default:** None

**Chức năng:**
- Cho phép role cụ thể **over deliver/receive** vượt quá allowance percentage
- User có role này có thể deliver/receive nhiều hơn mức cho phép
- Các user khác vẫn bị giới hạn bởi allowance

**Ví dụ:**
- `over_delivery_receipt_allowance` = 10%
- `role_allowed_to_over_deliver_receive` = "Stock Manager"
- User có role "Stock Manager" → Có thể deliver/receive vượt quá 10%
- User không có role "Stock Manager" → Bị giới hạn ở 10%

---

#### 2.1.5. Allow Negative Stock (`allow_negative_stock`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép **tồn kho âm** (negative stock)
- Khi disabled, hệ thống sẽ chặn các transactions dẫn đến negative stock
- **Không thể** enable cùng lúc với `enable_stock_reservation`

**Code:**
```python
# File: stock_settings.py

def validate_stock_reservation(self):
    if self.allow_negative_stock and self.enable_stock_reservation:
        frappe.throw("As Stock Reservation is enabled, you can not enable Allow Negative Stock.")
```

**Ví dụ:**
- Warehouse: Stock = 10 units
- Delivery Note: Quantity = 15 units
- `allow_negative_stock` = 0 → Báo lỗi (không đủ stock)
- `allow_negative_stock` = 1 → Cho phép (stock = -5 units)

---

#### 2.1.6. Show Barcode Field in Stock Transactions (`show_barcode_field`)

**Field Type:** Check  
**Default:** 1 (True)

**Chức năng:**
- Khi enabled, hiển thị **barcode field** trong stock transactions
- Áp dụng cho các fields: `barcode`, `barcodes`, `scan_barcode`

**Code:**
```python
# File: stock_settings.py

# show/hide barcode field
for name in ["barcode", "barcodes", "scan_barcode"]:
    frappe.make_property_setter(
        {"fieldname": name, "property": "hidden", "value": 0 if self.show_barcode_field else 1},
        validate_fields_for_doctype=False,
    )
```

**Ví dụ:**
- `show_barcode_field` = 1 → Hiển thị barcode field trong transactions
- `show_barcode_field` = 0 → Ẩn barcode field

---

#### 2.1.7. Convert Item Description to Clean HTML in Transactions (`clean_description_html`)

**Field Type:** Check  
**Default:** 1 (True)

**Chức năng:**
- Khi enabled, hệ thống sẽ **convert** Item Description thành clean HTML trong transactions
- Loại bỏ các HTML tags không cần thiết
- Khi enable lần đầu, sẽ clean tất cả descriptions hiện có

**Code:**
```python
# File: stock_settings.py

def validate_clean_description_html(self):
    if int(self.clean_description_html or 0) and not int(self.db_get("clean_description_html") or 0):
        # Clean all descriptions
        frappe.enqueue(
            "erpnext.stock.doctype.stock_settings.stock_settings.clean_all_descriptions",
            now=frappe.in_test,
            enqueue_after_commit=True,
        )
```

**Ví dụ:**
- Item Description: `<p>Laptop</p><br>`
- `clean_description_html` = 1 → "Laptop" (clean HTML)
- `clean_description_html` = 0 → Giữ nguyên HTML

---

#### 2.1.8. Allow Internal Transfers at Arm's Length Price (`allow_internal_transfer_at_arms_length_price`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, item rate **không điều chỉnh** theo valuation rate trong internal transfers, nhưng accounting vẫn sử dụng valuation rate
- Hữu ích cho các trường hợp transfer giữa các companies với giá thị trường

**Ví dụ:**
- Item Valuation Rate = 100
- Internal Transfer Rate = 120
- `allow_internal_transfer_at_arms_length_price` = 1 → Rate = 120 (không điều chỉnh), Accounting dùng 100
- `allow_internal_transfer_at_arms_length_price` = 0 → Rate = 100 (điều chỉnh theo valuation)

---

#### 2.1.9. Validate Material Transfer Warehouses (`validate_material_transfer_warehouses`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, hệ thống sẽ **validate** source và target warehouse trong Material Transfer Stock Entry phải khác nhau
- Nếu có inventory dimensions, có thể cho phép cùng warehouse nhưng ít nhất 1 dimension phải khác

**Ví dụ:**
- Material Transfer: Source Warehouse = "Main", Target Warehouse = "Main"
- `validate_material_transfer_warehouses` = 1 → Báo lỗi (phải khác nhau)
- `validate_material_transfer_warehouses` = 0 → Cho phép

---

## 🔷 3. Serial & Batch Item Tab

### 3.1. Serial & Batch Item Settings Section

#### 3.1.1. Allow existing Serial No to be Manufactured/Received again (`allow_existing_serial_no`)

**Field Type:** Check  
**Default:** 1 (True)

**Chức năng:**
- Khi enabled, cho phép **manufacture/receive** Serial No đã tồn tại
- Khi disabled, hệ thống sẽ chặn việc tạo Serial No trùng lặp

**Ví dụ:**
- Serial No "SN001" đã tồn tại
- `allow_existing_serial_no` = 1 → Cho phép manufacture/receive lại
- `allow_existing_serial_no` = 0 → Báo lỗi (Serial No đã tồn tại)

---

#### 3.1.2. Do Not Use Batch-wise Valuation (`do_not_use_batchwise_valuation`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, hệ thống sẽ sử dụng **Moving Average** valuation method để tính valuation rate cho batched items, không xem xét individual batch-wise incoming rate
- Khi disabled, sử dụng batch-wise valuation (mỗi batch có rate riêng)

**Ví dụ:**
- Batch 1: Rate = 100
- Batch 2: Rate = 120
- `do_not_use_batchwise_valuation` = 1 → Valuation Rate = (100 + 120) / 2 = 110 (Moving Average)
- `do_not_use_batchwise_valuation` = 0 → Valuation Rate theo từng batch riêng

---

#### 3.1.3. Auto Create Serial and Batch Bundle For Outward (`auto_create_serial_and_batch_bundle_for_outward`)

**Field Type:** Check  
**Default:** 1 (True)

**Chức năng:**
- Khi enabled, hệ thống sẽ **tự động tạo** Serial and Batch Bundle khi submit outward transactions (Delivery Note, Stock Entry, etc.)
- Khi disabled, phải tạo bundle thủ công

**Ví dụ:**
- Delivery Note: Item A (Serial No: SN001, SN002)
- `auto_create_serial_and_batch_bundle_for_outward` = 1 → Tự động tạo bundle với SN001, SN002
- `auto_create_serial_and_batch_bundle_for_outward` = 0 → Phải tạo bundle thủ công

---

#### 3.1.4. Pick Serial / Batch Based On (`pick_serial_and_batch_based_on`)

**Field Type:** Select  
**Default:** "FIFO"  
**Options:**
- `FIFO` - First In First Out
- `LIFO` - Last In First Out
- `Expiry` - Dựa trên ngày hết hạn

**Dependency:** Chỉ hiển thị khi `auto_create_serial_and_batch_bundle_for_outward` = 1

**Chức năng:**
- Xác định cách hệ thống **pick** Serial No / Batch khi tạo bundle tự động
- `FIFO`: Pick Serial/Batch nhập trước
- `LIFO`: Pick Serial/Batch nhập sau
- `Expiry`: Pick Serial/Batch sắp hết hạn trước

**Ví dụ:**
- Stock: Batch 1 (Expiry: 2024-01-01), Batch 2 (Expiry: 2024-02-01)
- `pick_serial_and_batch_based_on` = "Expiry" → Pick Batch 1 trước (sắp hết hạn)
- `pick_serial_and_batch_based_on` = "FIFO" → Pick Batch nhập trước

---

#### 3.1.5. Disable Serial No And Batch Selector (`disable_serial_no_and_batch_selector`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **vô hiệu hóa** Serial No và Batch selector trong transactions
- User không thể chọn Serial No / Batch thủ công

**Ví dụ:**
- `disable_serial_no_and_batch_selector` = 1 → Không thể chọn Serial No / Batch
- `disable_serial_no_and_batch_selector` = 0 → Có thể chọn Serial No / Batch

---

#### 3.1.6. Use Serial / Batch Fields (`use_serial_batch_fields`)

**Field Type:** Check  
**Default:** 1 (True)

**Chức năng:**
- Khi enabled, hệ thống sẽ **tự động tạo** Serial and Batch Bundle dựa trên Serial No / Batch fields khi submit transaction
- Khi disabled, phải tạo bundle thủ công

**Ví dụ:**
- Transaction: Serial No field = "SN001, SN002"
- `use_serial_batch_fields` = 1 → Tự động tạo bundle với SN001, SN002
- `use_serial_batch_fields` = 0 → Phải tạo bundle thủ công

---

#### 3.1.7. Do Not Update Serial / Batch on Creation of Auto Bundle (`do_not_update_serial_batch_on_creation_of_auto_bundle`)

**Field Type:** Check  
**Default:** 1 (True)

**Chức năng:**
- Khi enabled, **không update** serial / batch values trong stock transactions khi tạo auto Serial / Batch Bundle
- Khi disabled, sẽ update serial / batch values

**Ví dụ:**
- Transaction: Serial No field = "SN001"
- Auto Bundle tạo: SN001, SN002
- `do_not_update_serial_batch_on_creation_of_auto_bundle` = 1 → Giữ nguyên Serial No field = "SN001"
- `do_not_update_serial_batch_on_creation_of_auto_bundle` = 0 → Update Serial No field = "SN001, SN002"

---

### 3.2. Serial and Batch Bundle Section

#### 3.2.1. Set Serial and Batch Bundle Naming Based on Naming Series (`set_serial_and_batch_bundle_naming_based_on_naming_series`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, Serial and Batch Bundle sẽ sử dụng **naming series** thay vì auto name
- Khi disabled, sử dụng auto name

**Ví dụ:**
- `set_serial_and_batch_bundle_naming_based_on_naming_series` = 1 → Bundle ID = "BUNDLE-00001"
- `set_serial_and_batch_bundle_naming_based_on_naming_series` = 0 → Bundle ID = auto-generated

---

#### 3.2.2. Have Default Naming Series for Batch ID? (`use_naming_series`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, Batch ID sẽ sử dụng **naming series** với prefix được chỉ định
- Khi disabled, Batch ID là auto name

**Ví dụ:**
- `use_naming_series` = 1 → Batch ID = "BATCH-00001"
- `use_naming_series` = 0 → Batch ID = auto-generated

---

#### 3.2.3. Naming Series Prefix (`naming_series_prefix`)

**Field Type:** Data  
**Default:** "BATCH-"

**Dependency:** Chỉ hiển thị khi `use_naming_series` = 1

**Chức năng:**
- Xác định **prefix** cho Batch ID naming series
- Chỉ áp dụng khi `use_naming_series` = 1

**Ví dụ:**
- `naming_series_prefix` = "BATCH-" → Batch ID = "BATCH-00001"
- `naming_series_prefix` = "LOT-" → Batch ID = "LOT-00001"

---

## 🔷 4. Stock Reservation Tab

### 4.1. Stock Reservation Section

#### 4.1.1. Enable Stock Reservation (`enable_stock_reservation`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép **reserve** (giữ lại) một số lượng inventory cụ thể cho một order
- **Không thể** enable cùng lúc với `allow_negative_stock`
- **Không thể** disable nếu có open Stock Reservation Entries

**Code:**
```python
# File: stock_settings.py

def validate_stock_reservation(self):
    # Không thể enable cùng lúc với allow_negative_stock
    if self.allow_negative_stock and self.enable_stock_reservation:
        frappe.throw("As Allow Negative Stock is enabled, you can not enable Stock Reservation.")
    
    # Không thể disable nếu có reserved stock
    if not self.enable_stock_reservation:
        has_reserved_stock = frappe.db.exists(
            "Stock Reservation Entry", {"docstatus": 1, "status": ["!=", "Delivered"]}
        )
        if has_reserved_stock:
            frappe.throw("As there are reserved stock, you cannot disable Stock Reservation.")
```

**Ví dụ:**
- Sales Order: Quantity = 100 units
- Available Stock = 100 units
- `enable_stock_reservation` = 1 → Reserve 100 units cho Sales Order
- `enable_stock_reservation` = 0 → Không reserve

---

#### 4.1.2. Auto Reserve Stock (`auto_reserve_stock`)

**Field Type:** Check  
**Default:** 0 (False)

**Dependency:** Chỉ hiển thị khi `enable_stock_reservation` = 1

**Chức năng:**
- Khi enabled, hệ thống sẽ **tự động reserve** stock khi submit Sales Order, Work Order, hoặc Production Plan
- Khi disabled, phải reserve thủ công

**Ví dụ:**
- Sales Order: Quantity = 100 units
- `auto_reserve_stock` = 1 → Tự động reserve 100 units khi submit
- `auto_reserve_stock` = 0 → Phải reserve thủ công

---

#### 4.1.3. Allow Partial Reservation (`allow_partial_reservation`)

**Field Type:** Check  
**Default:** 1 (True)

**Dependency:** Chỉ hiển thị khi `enable_stock_reservation` = 1

**Chức năng:**
- Khi enabled, cho phép **reserve một phần** stock nếu không đủ
- Khi disabled, chỉ reserve khi đủ stock

**Ví dụ:**
- Sales Order: Quantity = 100 units
- Available Stock = 90 units
- `allow_partial_reservation` = 1 → Reserve 90 units (partial)
- `allow_partial_reservation` = 0 → Không reserve (phải đủ 100 units)

---

#### 4.1.4. Auto Reserve Stock for Sales Order on Purchase (`auto_reserve_stock_for_sales_order_on_purchase`)

**Field Type:** Check  
**Default:** 0 (False)

**Dependency:** Chỉ hiển thị khi `enable_stock_reservation` = 1

**Chức năng:**
- Khi enabled, stock sẽ được **tự động reserve** khi submit Purchase Receipt được tạo từ Material Request cho Sales Order
- Hữu ích cho make-to-order scenarios

**Ví dụ:**
- Sales Order: Quantity = 100 units
- Material Request: Quantity = 100 units (cho Sales Order)
- Purchase Receipt: Quantity = 100 units (từ Material Request)
- `auto_reserve_stock_for_sales_order_on_purchase` = 1 → Tự động reserve 100 units cho Sales Order
- `auto_reserve_stock_for_sales_order_on_purchase` = 0 → Không tự động reserve

---

### 4.2. Serial and Batch Reservation Section

#### 4.2.1. Auto Reserve Serial and Batch Nos (`auto_reserve_serial_and_batch`)

**Field Type:** Check  
**Default:** 1 (True)

**Dependency:** Chỉ hiển thị khi `enable_stock_reservation` = 1

**Chức năng:**
- Khi enabled, Serial and Batch Nos sẽ được **tự động reserve** dựa trên "Pick Serial / Batch Based On"
- Khi disabled, không tự động reserve Serial/Batch

**Ví dụ:**
- Sales Order: Item A (Serial No required)
- `auto_reserve_serial_and_batch` = 1 → Tự động reserve Serial No theo FIFO/LIFO/Expiry
- `auto_reserve_serial_and_batch` = 0 → Không tự động reserve Serial No

---

## 🔷 5. Quality Tab

### 5.1. Quality Inspection Settings Section

#### 5.1.1. Action If Quality Inspection Is Not Submitted (`action_if_quality_inspection_is_not_submitted`)

**Field Type:** Select  
**Default:** "Stop"  
**Options:**
- `Stop` - Dừng và báo lỗi, không cho phép lưu
- `Warn` - Cảnh báo nhưng vẫn cho phép lưu

**Chức năng:**
- Xác định hành động khi Quality Inspection chưa được submit
- `Stop`: Ngăn chặn việc lưu transaction nếu Quality Inspection chưa submit
- `Warn`: Hiển thị cảnh báo nhưng vẫn cho phép lưu

**Ví dụ:**
- Purchase Receipt: Quality Inspection = "QI-001" (chưa submit)
- `action_if_quality_inspection_is_not_submitted` = "Stop" → Báo lỗi, không cho lưu
- `action_if_quality_inspection_is_not_submitted` = "Warn" → Cảnh báo, vẫn cho lưu

---

#### 5.1.2. Action If Quality Inspection Is Rejected (`action_if_quality_inspection_is_rejected`)

**Field Type:** Select  
**Default:** "Stop"  
**Options:**
- `Stop` - Dừng và báo lỗi, không cho phép lưu
- `Warn` - Cảnh báo nhưng vẫn cho phép lưu

**Chức năng:**
- Xác định hành động khi Quality Inspection bị rejected
- `Stop`: Ngăn chặn việc lưu transaction nếu Quality Inspection rejected
- `Warn`: Hiển thị cảnh báo nhưng vẫn cho phép lưu

**Ví dụ:**
- Purchase Receipt: Quality Inspection = "QI-001" (rejected)
- `action_if_quality_inspection_is_rejected` = "Stop" → Báo lỗi, không cho lưu
- `action_if_quality_inspection_is_rejected` = "Warn" → Cảnh báo, vẫn cho lưu

---

#### 5.1.3. Allow to Make Quality Inspection after Purchase / Delivery (`allow_to_make_quality_inspection_after_purchase_or_delivery`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép tạo Quality Inspection **sau khi** submit Purchase Receipt hoặc Delivery Note
- Khi disabled, Quality Inspection phải được tạo trước

**Ví dụ:**
- Purchase Receipt: Status = "Submitted"
- `allow_to_make_quality_inspection_after_purchase_or_delivery` = 1 → Có thể tạo Quality Inspection sau
- `allow_to_make_quality_inspection_after_purchase_or_delivery` = 0 → Phải tạo Quality Inspection trước

---

## 🔷 6. Stock Planning Tab

### 6.1. Auto Material Request Section

#### 6.1.1. Raise Material Request When Stock Reaches Re-order Level (`auto_indent`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, hệ thống sẽ **tự động tạo** Material Request khi stock đạt đến re-order level
- Chạy qua scheduled job (scheduler)

**Ví dụ:**
- Item A: Re-order Level = 50 units
- Current Stock = 45 units
- `auto_indent` = 1 → Tự động tạo Material Request
- `auto_indent` = 0 → Không tự động tạo

---

#### 6.1.2. Notify by Email on Creation of Automatic Material Request (`reorder_email_notify`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, hệ thống sẽ **gửi email** khi tạo Automatic Material Request
- Hữu ích để thông báo cho Purchase Manager

**Ví dụ:**
- Automatic Material Request được tạo
- `reorder_email_notify` = 1 → Gửi email thông báo
- `reorder_email_notify` = 0 → Không gửi email

---

### 6.2. Inter Warehouse Transfer Settings Section

#### 6.2.1. Allow Material Transfer from Delivery Note to Sales Invoice (`allow_from_dn`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép **material transfer** từ Delivery Note đến Sales Invoice
- Hiển thị field `target_warehouse` trong Sales Invoice Item và Delivery Note Item

**Code:**
```python
# File: stock_settings.py

def toggle_warehouse_field_for_inter_warehouse_transfer(self):
    make_property_setter(
        "Sales Invoice Item",
        "target_warehouse",
        "hidden",
        1 - cint(self.allow_from_dn),
        "Check",
        ...
    )
    make_property_setter(
        "Delivery Note Item",
        "target_warehouse",
        "hidden",
        1 - cint(self.allow_from_dn),
        "Check",
        ...
    )
```

**Ví dụ:**
- `allow_from_dn` = 1 → Có field "Target Warehouse" trong Sales Invoice và Delivery Note
- `allow_from_dn` = 0 → Field "Target Warehouse" bị ẩn

---

#### 6.2.2. Allow Material Transfer from Purchase Receipt to Purchase Invoice (`allow_from_pr`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép **material transfer** từ Purchase Receipt đến Purchase Invoice
- Hiển thị field `from_warehouse` trong Purchase Invoice Item và Purchase Receipt Item

**Ví dụ:**
- `allow_from_pr` = 1 → Có field "From Warehouse" trong Purchase Invoice và Purchase Receipt
- `allow_from_pr` = 0 → Field "From Warehouse" bị ẩn

---

## 🔷 7. Stock Closing Tab

### 7.1. Control Historical Stock Transactions Section

#### 7.1.1. Stock Frozen Up To (`stock_frozen_upto`)

**Field Type:** Date  
**Default:** None

**Chức năng:**
- Xác định **ngày** mà trước đó không thể tạo hoặc sửa đổi stock transactions
- Hữu ích để bảo vệ dữ liệu lịch sử sau khi đã đóng sổ

**Code:**
```python
# File: stock_settings.py

def validate_pending_reposts(self):
    if self.stock_frozen_upto:
        check_pending_reposting(self.stock_frozen_upto)
```

**Ví dụ:**
- `stock_frozen_upto` = "2024-01-01"
- Không thể tạo/sửa stock transactions trước ngày 2024-01-01

---

#### 7.1.2. Freeze Stocks Older Than (Days) (`stock_frozen_upto_days`)

**Field Type:** Int  
**Default:** None

**Chức năng:**
- Xác định **số ngày** mà stock transactions cũ hơn không thể được sửa đổi
- Tính từ ngày hiện tại trở về trước

**Ví dụ:**
- `stock_frozen_upto_days` = 30
- Hôm nay = 2024-01-31
- Không thể sửa stock transactions trước ngày 2024-01-01 (30 ngày trước)

---

#### 7.1.3. Role Allowed to Edit Frozen Stock (`stock_auth_role`)

**Field Type:** Link (Role)  
**Default:** None

**Dependency:** Chỉ hiển thị khi `stock_frozen_upto` hoặc `stock_frozen_upto_days` được set

**Chức năng:**
- Cho phép role cụ thể **tạo/sửa** stock transactions dù đã bị frozen
- User có role này vẫn có thể làm việc với frozen transactions
- Các user khác vẫn bị chặn

**Ví dụ:**
- `stock_frozen_upto` = "2024-01-01"
- `stock_auth_role` = "Stock Manager"
- User có role "Stock Manager" → Có thể tạo/sửa transactions trước 2024-01-01
- User không có role "Stock Manager" → Bị chặn

---

#### 7.1.4. Role Allowed to Create/Edit Back-dated Transactions (`role_allowed_to_create_edit_back_dated_transactions`)

**Field Type:** Link (Role)  
**Default:** None

**Chức năng:**
- Cho phép role cụ thể **tạo/sửa** back-dated transactions (transactions có ngày trước latest stock transaction)
- Nếu để trống, tất cả users đều có thể tạo/sửa back-dated transactions
- Nếu set, chỉ users có role này mới có thể

**Ví dụ:**
- Latest Stock Transaction: 2024-01-15
- New Transaction: 2024-01-10 (back-dated)
- `role_allowed_to_create_edit_back_dated_transactions` = "Stock Manager"
- User có role "Stock Manager" → Có thể tạo transaction 2024-01-10
- User không có role "Stock Manager" → Bị chặn

---

## 🔷 8. Các Methods trong stock_settings.py

### 8.1. `validate()`

**Chức năng:**
- Validate và lưu các settings vào `frappe.defaults`
- Set naming series cho Item
- Validate warehouses, valuation method, stock reservation, etc.

---

### 8.2. `on_update()`

**Chức năng:**
- Toggle các property setters khi settings được update
- Ẩn/hiện các fields dựa trên settings

---

### 8.3. `validate_warehouses()`

**Chức năng:**
- Validate warehouses không phải là Group Warehouse

---

### 8.4. `cant_change_valuation_method()`

**Chức năng:**
- Ngăn chặn thay đổi valuation method nếu đã có Stock Ledger Entries

---

### 8.5. `validate_stock_reservation()`

**Chức năng:**
- Validate không thể enable cùng lúc `allow_negative_stock` và `enable_stock_reservation`
- Validate không thể disable `enable_stock_reservation` nếu có open Stock Reservation Entries

---

## 📝 Tóm tắt

### Các nhóm settings chính:

1. **Defaults** - Thiết lập mặc định cho Item, Warehouse, UOM, Valuation Method
2. **Stock Validations** - Cấu hình các validation rules
3. **Serial & Batch Item** - Cấu hình cho Serial No và Batch
4. **Stock Reservation** - Cấu hình cho Stock Reservation
5. **Quality** - Cấu hình cho Quality Inspection
6. **Stock Planning** - Cấu hình cho Material Request tự động
7. **Stock Closing** - Cấu hình cho việc đóng sổ và frozen transactions

### Các settings quan trọng nhất:

1. **`valuation_method`** - Phương pháp tính giá trị tồn kho
2. **`allow_negative_stock`** - Cho phép tồn kho âm
3. **`enable_stock_reservation`** - Bật Stock Reservation
4. **`over_delivery_receipt_allowance`** - Phần trăm được phép vượt quá
5. **`stock_frozen_upto`** - Đóng băng transactions cũ

---

## 🔗 Tài liệu tham khảo

- **File source:**
  - `apps/erpnext/erpnext/stock/doctype/stock_settings/stock_settings.json`
  - `apps/erpnext/erpnext/stock/doctype/stock_settings/stock_settings.py`
- **Documentation:**
  - Valuation Method: https://docs.erpnext.com/docs/v14/user/manual/en/stock/articles/calculation-of-valuation-rate-in-fifo-and-moving-average
