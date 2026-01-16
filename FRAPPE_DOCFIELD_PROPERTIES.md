# Tất cả các thuộc tính của Field trong DocType

File này liệt kê **tất cả các thuộc tính (properties)** có thể cấu hình cho một field trong DocType của Frappe v16.

---

## 📋 Tổng quan

Mỗi field trong DocType là một bản ghi của DocType `DocField`, có rất nhiều thuộc tính để cấu hình hành vi và hiển thị của field.

**Lưu ý:** Một số thuộc tính chỉ áp dụng cho một số fieldtype cụ thể (được chỉ định trong `depends_on`).

---

## 🔷 1. Thuộc tính Cơ bản (Basic Properties)

### 1.1. `fieldname` (Data)
**Tên field** - Tên duy nhất của field trong DocType.

- **Bắt buộc:** Có
- **Ví dụ:** `"customer_name"`, `"email_id"`, `"grand_total"`
- **Lưu ý:** Phải tuân theo quy tắc đặt tên (không có khoảng trắng, không ký tự đặc biệt)

### 1.2. `label` (Data)
**Nhãn hiển thị** - Text hiển thị cho field.

- **Bắt buộc:** Có
- **Ví dụ:** `"Customer Name"`, `"Email Address"`, `"Grand Total"`
- **Lưu ý:** Có thể dịch được nếu field là `translatable`

### 1.3. `fieldtype` (Select)
**Loại field** - Kiểu dữ liệu của field.

- **Bắt buộc:** Có
- **Các giá trị có thể:**
  - `Autocomplete` - Autocomplete field
  - `Attach` - File attachment
  - `Attach Image` - Image attachment
  - `Barcode` - Barcode scanner
  - `Button` - Button
  - `Check` - Checkbox
  - `Code` - Code editor
  - `Color` - Color picker
  - `Column Break` - Column break (layout)
  - `Currency` - Currency
  - `Data` - Text input
  - `Date` - Date picker
  - `Datetime` - Date and time picker
  - `Duration` - Duration
  - `Dynamic Link` - Dynamic link
  - `Float` - Decimal number
  - `Fold` - Collapsible section
  - `Geolocation` - Geolocation
  - `Heading` - Heading text
  - `HTML` - HTML content
  - `HTML Editor` - HTML editor
  - `Icon` - Icon picker
  - `Image` - Image display
  - `Int` - Integer
  - `JSON` - JSON data
  - `Link` - Link to another DocType
  - `Long Text` - Long text area
  - `Markdown Editor` - Markdown editor
  - `Password` - Password field
  - `Percent` - Percentage
  - `Phone` - Phone number
  - `Read Only` - Read-only text
  - `Rating` - Rating stars
  - `Section Break` - Section break (layout)
  - `Select` - Dropdown select
  - `Signature` - Signature pad
  - `Small Text` - Small text area
  - `Tab Break` - Tab break (layout)
  - `Table` - Child table
  - `Table MultiSelect` - Table multiselect
  - `Text` - Text area
  - `Text Editor` - Rich text editor
  - `Time` - Time picker

### 1.4. `description` (Small Text)
**Mô tả** - Mô tả ngắn gọn về field, hiển thị dưới label.

- **Bắt buộc:** Không
- **Ví dụ:** `"Enter the customer's full name"`

### 1.5. `documentation_url` (Data)
**URL tài liệu** - Link đến tài liệu về field này.

- **Bắt buộc:** Không
- **Fieldtype:** URL
- **Depends on:** Không áp dụng cho Tab Break, Section Break, Column Break, Button, HTML

---

## 🔷 2. Thuộc tính Giá trị (Value Properties)

### 2.1. `default` (Small Text)
**Giá trị mặc định** - Giá trị mặc định khi tạo document mới.

- **Bắt buộc:** Không
- **Ví dụ:** `"Draft"`, `frappe.utils.today()`, `frappe.session.user`
- **Lưu ý:** Có thể là Python expression hoặc JavaScript expression

### 2.2. `options` (Small Text)
**Tùy chọn** - Tùy thuộc vào fieldtype:
- **Select:** Danh sách các options (mỗi option một dòng)
- **Link:** Tên DocType được link đến
- **Dynamic Link:** Tên field chứa DocType
- **Table:** Tên DocType của child table
- **Table MultiSelect:** Tên DocType của child table
- **và các fieldtype khác...**

- **Bắt buộc:** Tùy fieldtype
- **Ví dụ:** 
  - Select: `"Draft\nSubmitted\nCancelled"`
  - Link: `"Customer"`
  - Table: `"Sales Order Item"`

### 2.3. `sort_options` (Check)
**Sắp xếp options** - Tự động sắp xếp options trong Select field theo thứ tự alphabet.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype == "Select"`

### 2.4. `length` (Int)
**Độ dài** - Độ dài tối đa cho field (cho Data, Link, Dynamic Link, Password, Select, Read Only, Attach, Attach Image, Int, Float, Currency, Percent).

- **Bắt buộc:** Không
- **Ví dụ:** `255`, `100`
- **Depends on:** Một số fieldtype cụ thể

### 2.5. `precision` (Select)
**Độ chính xác** - Số chữ số thập phân (cho Float, Currency, Percent).

- **Bắt buộc:** Không
- **Options:** `""`, `"0"`, `"1"`, `"2"`, `"3"`, `"4"`, `"5"`, `"6"`, `"7"`, `"8"`, `"9"`
- **Default:** `""` (sử dụng precision mặc định)
- **Depends on:** `fieldtype in ["Float", "Currency", "Percent"]`

### 2.6. `non_negative` (Check)
**Không âm** - Chỉ cho phép giá trị >= 0 (cho Int, Float, Currency, Percent).

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype in ["Int", "Float", "Currency", "Percent"]`

### 2.7. `not_nullable` (Check)
**Không được NULL** - Field không được phép có giá trị NULL trong database.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** Không áp dụng cho Check, Currency, Float, Int, Percent, Rating, Select, Table, Table MultiSelect

---

## 🔷 3. Thuộc tính Fetch (Fetch Properties)

### 3.1. `fetch_from` (Small Text)
**Fetch từ** - Tự động lấy giá trị từ field khác khi link field thay đổi.

- **Bắt buộc:** Không
- **Format:** `"link_field.source_field"`
- **Ví dụ:** `"customer.customer_name"`, `"item_code.item_name"`
- **Lưu ý:** Chỉ hoạt động với Link và Dynamic Link fields

### 3.2. `fetch_if_empty` (Check)
**Fetch nếu rỗng** - Chỉ fetch giá trị nếu field hiện tại đang rỗng.

- **Bắt buộc:** Không
- **Default:** `0` (false) - Luôn fetch khi link thay đổi
- **Description:** Nếu unchecked, giá trị sẽ luôn được re-fetch khi save

### 3.3. `link_filters` (JSON)
**Bộ lọc Link** - Bộ lọc tĩnh cho Link field (không phụ thuộc vào document).

- **Bắt buộc:** Không
- **Format:** JSON object
- **Ví dụ:** `{"status": "Active", "disabled": 0}`
- **Lưu ý:** Khác với `get_query` (dynamic filters)

---

## 🔷 4. Thuộc tính Hiển thị (Display Properties)

### 4.1. `hidden` (Check)
**Ẩn** - Ẩn field khỏi form (nhưng vẫn có trong database).

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Lưu ý:** Field vẫn có thể được truy cập qua code, chỉ ẩn trong UI

### 4.2. `read_only` (Check)
**Chỉ đọc** - Field không thể chỉnh sửa, chỉ có thể đọc.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Lưu ý:** Có thể set động qua `read_only_depends_on`

### 4.3. `read_only_depends_on` (Code)
**Read Only phụ thuộc** - JavaScript expression để set read_only động.

- **Bắt buộc:** Không
- **Format:** JavaScript expression
- **Ví dụ:** `"doc.status === 'Submitted'"`, `"doc.docstatus === 1"`
- **Options:** `"JS"`

### 4.4. `bold` (Check)
**In đậm** - Hiển thị label in đậm.

- **Bắt buộc:** Không
- **Default:** `0` (false)

### 4.5. `width` (Data)
**Chiều rộng** - Chiều rộng của field trong form.

- **Bắt buộc:** Không
- **Format:** CSS width (px, %, em, etc.)
- **Ví dụ:** `"300px"`, `"50%"`, `"10em"`

### 4.6. `max_height` (Data)
**Chiều cao tối đa** - Chiều cao tối đa của field (cho text areas).

- **Bắt buộc:** Không
- **Format:** CSS height
- **Ví dụ:** `"200px"`, `"10rem"`

### 4.7. `columns` (Int)
**Số cột** - Số cột trong List View hoặc Grid (tổng cột phải < 11).

- **Bắt buộc:** Không
- **Ví dụ:** `1`, `2`, `3`
- **Description:** Number of columns for a field in a List View or a Grid

### 4.8. `placeholder` (Data)
**Placeholder** - Text placeholder hiển thị trong input khi field rỗng.

- **Bắt buộc:** Không
- **Ví dụ:** `"Enter email address"`, `"Select customer"`

### 4.9. `hide_border` (Check)
**Ẩn border** - Ẩn border của section (cho Section Break).

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype == "Section Break"`

### 4.10. `button_color` (Select)
**Màu button** - Màu của button (cho Button field).

- **Bắt buộc:** Không
- **Options:** `""`, `"Default"`, `"Primary"`, `"Info"`, `"Success"`, `"Warning"`, `"Danger"`
- **Depends on:** `fieldtype == "Button"`

---

## 🔷 5. Thuộc tính Phụ thuộc (Dependency Properties)

### 5.1. `depends_on` (Code)
**Hiển thị phụ thuộc** - JavaScript expression để quyết định khi nào field được hiển thị.

- **Bắt buộc:** Không
- **Format:** JavaScript expression
- **Ví dụ:** `"doc.status === 'Draft'"`, `"doc.docstatus === 0"`
- **Options:** `"JS"`
- **Lưu ý:** Field sẽ bị ẩn nếu expression trả về false

### 5.2. `mandatory_depends_on` (Code)
**Bắt buộc phụ thuộc** - JavaScript expression để set field thành mandatory động.

- **Bắt buộc:** Không
- **Format:** JavaScript expression
- **Ví dụ:** `"doc.status === 'Active'"`, `"doc.payment_type === 'Cheque'"`
- **Options:** `"JS"`
- **Lưu ý:** Field sẽ trở thành required nếu expression trả về true

### 5.3. `collapsible` (Check)
**Có thể thu gọn** - Section có thể thu gọn/mở rộng (cho Section Break).

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype == "Section Break"`

### 5.4. `collapsible_depends_on` (Code)
**Thu gọn phụ thuộc** - JavaScript expression để quyết định section có được thu gọn mặc định không.

- **Bắt buộc:** Không
- **Format:** JavaScript expression
- **Options:** `"JS"`
- **Depends on:** `fieldtype == "Section Break" && collapsible == 1`

---

## 🔷 6. Thuộc tính Ràng buộc (Constraint Properties)

### 6.1. `reqd` (Check)
**Bắt buộc** - Field bắt buộc phải có giá trị.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** Không áp dụng cho Section Break, Column Break, Button, HTML
- **Lưu ý:** Có thể set động qua `mandatory_depends_on`

### 6.2. `unique` (Check)
**Duy nhất** - Giá trị của field phải duy nhất trong toàn bộ DocType.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Lưu ý:** Tạo unique index trong database

### 6.3. `set_only_once` (Check)
**Chỉ set một lần** - Field chỉ có thể được set một lần, sau đó không thể thay đổi.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Lưu ý:** Thường dùng cho các field như `creation_date`, `owner`

### 6.4. `no_copy` (Check)
**Không copy** - Field không được copy khi duplicate document.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Lưu ý:** Thường dùng cho các field như `name`, `creation`, `owner`

---

## 🔷 7. Thuộc tính Quyền (Permission Properties)

### 7.1. `permlevel` (Int)
**Mức quyền** - Mức quyền của field (0 = public, 1+ = restricted).

- **Bắt buộc:** Không
- **Default:** `0`
- **Depends on:** Không áp dụng cho Section Break, Column Break, Tab Break
- **Lưu ý:** Cần có DocPerm tương ứng với permlevel này

### 7.2. `ignore_user_permissions` (Check)
**Bỏ qua User Permissions** - Bỏ qua User Permissions khi query (cho Link, Dynamic Link, Table MultiSelect).

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype in ["Link", "Dynamic Link", "Table MultiSelect"]`
- **Lưu ý:** Cho phép user thấy tất cả records, không bị giới hạn bởi User Permissions

### 7.3. `allow_on_submit` (Check)
**Cho phép trên Submit** - Cho phép chỉnh sửa field khi document đã được submit.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `parent.is_submittable == 1`
- **Lưu ý:** Chỉ áp dụng cho submittable doctypes

---

## 🔷 8. Thuộc tính List View (List View Properties)

### 8.1. `in_list_view` (Check)
**Trong List View** - Hiển thị field trong List View.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `!is_virtual`
- **Lưu ý:** Field sẽ xuất hiện như một cột trong List View

### 8.2. `in_standard_filter` (Check)
**Trong Standard Filter** - Hiển thị field trong standard filter của List View.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Lưu ý:** Field sẽ xuất hiện trong filter sidebar

### 8.3. `in_filter` (Check)
**Trong Filter** - Field có thể được dùng trong filter.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Lưu ý:** Cho phép filter bằng field này

### 8.4. `in_global_search` (Check)
**Trong Global Search** - Field được index cho Global Search.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype in ["Data", "Select", "Table", "Text", "Text Editor", "Link", "Small Text", "Long Text", "Read Only", "Heading", "Dynamic Link"]`
- **Lưu ý:** Field sẽ được tìm kiếm khi user dùng Global Search

### 8.5. `in_preview` (Check)
**Trong Preview** - Hiển thị field trong preview popup.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype not in ["Table", "Table MultiSelect"]`

### 8.6. `sticky` (Check)
**Dính** - Field sẽ "dính" ở đầu List View khi scroll.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype not in ["Table", "Table MultiSelect"]`

---

## 🔷 9. Thuộc tính Print (Print Properties)

### 9.1. `print_hide` (Check)
**Ẩn khi in** - Ẩn field trong print format.

- **Bắt buộc:** Không
- **Default:** `0` (false)

### 9.2. `print_hide_if_no_value` (Check)
**Ẩn khi in nếu không có giá trị** - Ẩn field trong print format nếu field không có giá trị.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype in ["Int", "Float", "Currency", "Percent"]`

### 9.3. `print_width` (Data)
**Chiều rộng khi in** - Chiều rộng của field trong print format.

- **Bắt buộc:** Không
- **Format:** CSS width
- **Ví dụ:** `"100px"`, `"50%"`

### 9.4. `report_hide` (Check)
**Ẩn trong Report** - Ẩn field trong report.

- **Bắt buộc:** Không
- **Default:** `0` (false)

---

## 🔷 10. Thuộc tính Đặc biệt (Special Properties)

### 10.1. `is_virtual` (Check)
**Virtual** - Field không lưu trong database, chỉ tính toán động.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype != "Link"`
- **Lưu ý:** Virtual field không có trong database, giá trị được tính toán từ code

### 10.2. `search_index` (Check)
**Index** - Tạo index cho field trong database để tăng tốc độ tìm kiếm.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `!is_virtual`
- **Lưu ý:** Tạo database index, tăng tốc độ query nhưng làm chậm insert/update

### 10.3. `translatable` (Check)
**Có thể dịch** - Field có thể được dịch sang ngôn ngữ khác.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype in ["Data", "Select", "Text", "Small Text", "Text Editor"]`
- **Lưu ý:** Giá trị sẽ được lưu trong Translation DocType

### 10.4. `allow_in_quick_entry` (Check)
**Cho phép trong Quick Entry** - Field xuất hiện trong Quick Entry dialog.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype not in ["Tab Break", "Table"]`

### 10.5. `allow_bulk_edit` (Check)
**Cho phép Bulk Edit** - Cho phép edit nhiều rows cùng lúc trong child table.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype == "Table"`

### 10.6. `make_attachment_public` (Check)
**Làm attachment public** - Attachment mặc định là public (cho Attach, Attach Image).

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype in ["Attach", "Attach Image"]`

### 10.7. `ignore_xss_filter` (Check)
**Bỏ qua XSS Filter** - Không encode HTML tags như `<script>` hoặc ký tự như `<`, `>`.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Description:** Don't encode HTML tags like `<script>` or just characters like `<` or `>`, as they could be intentionally used in this field

### 10.8. `remember_last_selected_value` (Check)
**Nhớ giá trị cuối** - Nhớ giá trị được chọn lần cuối và set làm default cho lần sau (cho Link field).

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype == "Link"`
- **Lưu ý:** Giá trị được lưu trong `frappe.boot.user.last_selected_values`

### 10.9. `mask` (Check)
**Mask** - Mask giá trị field dựa trên user permissions (ẩn giá trị nhạy cảm).

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype in ["Select", "Read Only", "Phone", "Percent", "Password", "Link", "Int", "Float", "Dynamic Link", "Duration", "Datetime", "Currency", "Data", "Date"]`
- **Lưu ý:** Giá trị sẽ bị mask (hiển thị `****`) nếu user không có quyền xem

### 10.10. `show_on_timeline` (Check)
**Hiển thị trên Timeline** - Hiển thị field trên timeline (khi field bị hidden).

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `hidden == 1`
- **Lưu ý:** Chỉ áp dụng khi field bị hidden

### 10.11. `show_dashboard` (Check)
**Hiển thị Dashboard** - Hiển thị dashboard trong tab này (cho Tab Break).

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype == "Tab Break"`

---

## 🔷 11. Thuộc tính Duration (Duration Properties)

### 11.1. `hide_days` (Check)
**Ẩn ngày** - Ẩn phần ngày trong Duration field.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype == "Duration"`

### 11.2. `hide_seconds` (Check)
**Ẩn giây** - Ẩn phần giây trong Duration field.

- **Bắt buộc:** Không
- **Default:** `0` (false)
- **Depends on:** `fieldtype == "Duration"`

---

## 🔷 12. Thuộc tính Nội bộ (Internal Properties)

### 12.1. `parent` (Data)
**Parent DocType** - DocType chứa field này (tự động set).

- **Bắt buộc:** Có
- **Lưu ý:** Tự động set bởi Frappe, không cần set thủ công

### 12.2. `parentfield` (Data)
**Parent Field** - Field cha (cho child table fields, tự động set).

- **Bắt buộc:** Có (cho child table)
- **Lưu ý:** Tự động set bởi Frappe

### 12.3. `parenttype` (Data)
**Parent Type** - Luôn là "DocType" (tự động set).

- **Bắt buộc:** Có
- **Lưu ý:** Tự động set bởi Frappe

### 12.4. `oldfieldname` (Data)
**Tên field cũ** - Tên field trước khi rename (dùng để migration).

- **Bắt buộc:** Không
- **Hidden:** `1` (ẩn trong UI)
- **Lưu ý:** Tự động set khi rename field

### 12.5. `oldfieldtype` (Data)
**Loại field cũ** - Loại field trước khi thay đổi (dùng để migration).

- **Bắt buộc:** Không
- **Hidden:** `1` (ẩn trong UI)
- **Lưu ý:** Tự động set khi thay đổi fieldtype

---

## 🔷 13. Ví dụ Sử dụng

### 13.1. Field cơ bản

```json
{
    "fieldname": "customer_name",
    "label": "Customer Name",
    "fieldtype": "Data",
    "reqd": 1,
    "in_list_view": 1,
    "in_standard_filter": 1
}
```

### 13.2. Link field với fetch

```json
{
    "fieldname": "customer",
    "label": "Customer",
    "fieldtype": "Link",
    "options": "Customer",
    "reqd": 1,
    "in_list_view": 1,
    "fetch_from": "customer.customer_name"
}
```

### 13.3. Field với depends_on

```json
{
    "fieldname": "cheque_number",
    "label": "Cheque Number",
    "fieldtype": "Data",
    "depends_on": "doc.payment_type === 'Cheque'",
    "mandatory_depends_on": "doc.payment_type === 'Cheque'"
}
```

### 13.4. Select field

```json
{
    "fieldname": "status",
    "label": "Status",
    "fieldtype": "Select",
    "options": "Draft\nSubmitted\nCancelled",
    "default": "Draft",
    "reqd": 1,
    "in_list_view": 1
}
```

### 13.5. Child table

```json
{
    "fieldname": "items",
    "label": "Items",
    "fieldtype": "Table",
    "options": "Sales Order Item",
    "reqd": 1
}
```

### 13.6. Field với permissions

```json
{
    "fieldname": "salary",
    "label": "Salary",
    "fieldtype": "Currency",
    "permlevel": 1,
    "mask": 1
}
```

### 13.7. Virtual field

```json
{
    "fieldname": "total_amount",
    "label": "Total Amount",
    "fieldtype": "Currency",
    "is_virtual": 1,
    "read_only": 1
}
```

---

## 🔷 14. Truy cập trong Code

### 14.1. Python

```python
# Lấy DocField
docfield = frappe.get_meta("Customer").get_field("customer_name")

# Truy cập properties
print(docfield.fieldname)  # "customer_name"
print(docfield.label)      # "Customer Name"
print(docfield.fieldtype)  # "Data"
print(docfield.reqd)       # 1 hoặc 0
print(docfield.options)    # None hoặc giá trị options
```

### 14.2. JavaScript

```javascript
// Lấy DocField
const docfield = frappe.get_meta("Customer").fields.find(f => f.fieldname === "customer_name");

// Hoặc
const docfield = frappe.meta.docfield_map["Customer"]["customer_name"];

// Truy cập properties
console.log(docfield.fieldname);  // "customer_name"
console.log(docfield.label);      // "Customer Name"
console.log(docfield.fieldtype);  // "Data"
console.log(docfield.reqd);       // 1 hoặc 0
console.log(docfield.options);    // null hoặc giá trị options

// Trong form
const field = frm.get_docfield("customer_name");
console.log(field.fieldname);
console.log(field.reqd);
```

---

## 📝 Tóm tắt theo Nhóm

### Nhóm Cơ bản:
- `fieldname`, `label`, `fieldtype`, `description`, `documentation_url`

### Nhóm Giá trị:
- `default`, `options`, `sort_options`, `length`, `precision`, `non_negative`, `not_nullable`

### Nhóm Fetch:
- `fetch_from`, `fetch_if_empty`, `link_filters`

### Nhóm Hiển thị:
- `hidden`, `read_only`, `read_only_depends_on`, `bold`, `width`, `max_height`, `columns`, `placeholder`, `hide_border`, `button_color`

### Nhóm Phụ thuộc:
- `depends_on`, `mandatory_depends_on`, `collapsible`, `collapsible_depends_on`

### Nhóm Ràng buộc:
- `reqd`, `unique`, `set_only_once`, `no_copy`

### Nhóm Quyền:
- `permlevel`, `ignore_user_permissions`, `allow_on_submit`

### Nhóm List View:
- `in_list_view`, `in_standard_filter`, `in_filter`, `in_global_search`, `in_preview`, `sticky`

### Nhóm Print:
- `print_hide`, `print_hide_if_no_value`, `print_width`, `report_hide`

### Nhóm Đặc biệt:
- `is_virtual`, `search_index`, `translatable`, `allow_in_quick_entry`, `allow_bulk_edit`, `make_attachment_public`, `ignore_xss_filter`, `remember_last_selected_value`, `mask`, `show_on_timeline`, `show_dashboard`

### Nhóm Duration:
- `hide_days`, `hide_seconds`

### Nhóm Nội bộ:
- `parent`, `parentfield`, `parenttype`, `oldfieldname`, `oldfieldtype`

---

## 🔗 Tài liệu tham khảo

- **File source:** `apps/frappe/frappe/core/doctype/docfield/docfield.json`
- **Python class:** `apps/frappe/frappe/core/doctype/docfield/docfield.py`
- **Frappe Documentation:** [Field Properties](https://frappeframework.com/docs/user/en/desk/customize/customize-form)
