# Dependency Properties - Hướng dẫn Chi tiết

File này giải thích **cách sử dụng Dependency Properties** trong Frappe, bao gồm cấu trúc lệnh, cú pháp, và ví dụ cụ thể.

---

## 📋 Tổng quan

Frappe có **4 loại Dependency Properties**:
1. **`depends_on`** - Điều khiển hiển thị field (show/hide)
2. **`mandatory_depends_on`** - Điều khiển field có bắt buộc hay không (required/optional)
3. **`read_only_depends_on`** - Điều khiển field có read-only hay không
4. **`collapsible_depends_on`** - Điều khiển section có được thu gọn mặc định hay không

---

## 🔷 1. Cấu trúc và Cú pháp

### 1.1. Các cách viết Expression

Frappe hỗ trợ **5 cách** để viết dependency expression:

#### **Cách 1: Fieldname đơn giản**
Chỉ cần tên field - field sẽ hiển thị nếu field đó có giá trị (truthy).

```javascript
// Ví dụ
depends_on: "status"
// Field sẽ hiển thị nếu doc.status có giá trị (không null, không undefined, không rỗng)
```

#### **Cách 2: Boolean trực tiếp**
Giá trị boolean - luôn true hoặc false.

```javascript
// Ví dụ
depends_on: true   // Luôn hiển thị
depends_on: false  // Luôn ẩn
```

#### **Cách 3: Function**
JavaScript function nhận `doc` làm tham số.

```javascript
// Ví dụ
depends_on: function(doc) {
    return doc.status === "Active";
}
```

#### **Cách 4: `eval:` prefix**
JavaScript expression được evaluate với context `{doc, parent}`.

```javascript
// Ví dụ
depends_on: "eval:doc.status === 'Active'"
```

#### **Cách 5: `fn:` prefix**
Gọi một function trong client script.

```javascript
// Ví dụ
depends_on: "fn:check_status"
// Sẽ gọi function check_status() trong client script
```

---

## 🔷 2. Context và Biến có sẵn

### 2.1. Trong `eval:` expression

Khi dùng `eval:`, bạn có quyền truy cập:

- **`doc`** - Document hiện tại (parent document nếu là child table field)
- **`parent`** - Parent document (nếu đang ở child table)

**Ví dụ:**
```javascript
depends_on: "eval:doc.status === 'Active'"
// doc = document hiện tại

depends_on: "eval:parent.company === 'Test Company'"
// parent = parent document (khi ở trong child table)
```

### 2.2. Trong Function

Function nhận `doc` làm tham số:

```javascript
depends_on: function(doc) {
    // doc = document hiện tại
    return doc.status === "Active";
}
```

### 2.3. Trong `fn:` prefix

Function được gọi từ client script, có thể truy cập `frm`:

```javascript
// Trong client script
frappe.ui.form.on("Sales Order", {
    check_status: function(frm) {
        return frm.doc.status === "Active";
    }
});

// Trong DocField
depends_on: "fn:check_status"
```

---

## 🔷 3. `depends_on` - Hiển thị Field

### 3.1. Cú pháp

**Mục đích:** Điều khiển khi nào field được **hiển thị** (show) hoặc **ẩn** (hide).

**Kết quả:**
- `true` hoặc truthy → Field được hiển thị
- `false` hoặc falsy → Field bị ẩn

### 3.2. Ví dụ

#### **Ví dụ 1: Phụ thuộc vào một field**

```json
{
    "fieldname": "cheque_number",
    "label": "Cheque Number",
    "fieldtype": "Data",
    "depends_on": "eval:doc.payment_type === 'Cheque'"
}
```

**Giải thích:** Field `cheque_number` chỉ hiển thị khi `payment_type` là `"Cheque"`.

#### **Ví dụ 2: Phụ thuộc vào nhiều điều kiện**

```json
{
    "fieldname": "delivery_date",
    "label": "Delivery Date",
    "fieldtype": "Date",
    "depends_on": "eval:doc.status === 'Submitted' && doc.delivery_required === 1"
}
```

**Giải thích:** Field chỉ hiển thị khi cả 2 điều kiện đều đúng.

#### **Ví dụ 3: Phụ thuộc vào docstatus**

```json
{
    "fieldname": "cancellation_reason",
    "label": "Cancellation Reason",
    "fieldtype": "Small Text",
    "depends_on": "eval:doc.docstatus === 2"
}
```

**Giải thích:** Field chỉ hiển thị khi document đã bị cancel (docstatus = 2).

#### **Ví dụ 4: Phụ thuộc vào fieldname đơn giản**

```json
{
    "fieldname": "customer_details",
    "label": "Customer Details",
    "fieldtype": "Section Break",
    "depends_on": "customer"
}
```

**Giải thích:** Section chỉ hiển thị khi field `customer` có giá trị.

#### **Ví dụ 5: Dùng function**

```json
{
    "fieldname": "special_discount",
    "label": "Special Discount",
    "fieldtype": "Percent",
    "depends_on": "fn:check_special_discount"
}
```

**Trong client script:**
```javascript
frappe.ui.form.on("Sales Order", {
    check_special_discount: function(frm) {
        return frm.doc.customer_type === "VIP" && frm.doc.grand_total > 10000;
    }
});
```

#### **Ví dụ 6: Phụ thuộc vào child table**

```json
{
    "fieldname": "item_details",
    "label": "Item Details",
    "fieldtype": "Section Break",
    "depends_on": "eval:doc.items && doc.items.length > 0"
}
```

**Giải thích:** Section chỉ hiển thị khi có ít nhất 1 item trong child table.

#### **Ví dụ 7: Dùng toán tử logic**

```json
{
    "fieldname": "bank_details",
    "label": "Bank Details",
    "fieldtype": "Section Break",
    "depends_on": "eval:doc.payment_type === 'Bank Transfer' || doc.payment_type === 'Cheque'"
}
```

**Giải thích:** Section hiển thị khi payment_type là một trong hai giá trị.

#### **Ví dụ 8: Kiểm tra giá trị trong array**

```json
{
    "fieldname": "priority",
    "label": "Priority",
    "fieldtype": "Select",
    "options": "Low\nMedium\nHigh",
    "depends_on": "eval:['High', 'Medium'].includes(doc.status)"
}
```

#### **Ví dụ 9: So sánh số**

```json
{
    "fieldname": "discount_amount",
    "label": "Discount Amount",
    "fieldtype": "Currency",
    "depends_on": "eval:doc.grand_total > 1000"
}
```

#### **Ví dụ 10: Kiểm tra null/undefined**

```json
{
    "fieldname": "notes",
    "label": "Notes",
    "fieldtype": "Small Text",
    "depends_on": "eval:doc.customer != null && doc.customer != ''"
}
```

---

## 🔷 4. `mandatory_depends_on` - Bắt buộc Field

### 4.1. Cú pháp

**Mục đích:** Điều khiển khi nào field trở thành **bắt buộc** (required/mandatory).

**Kết quả:**
- `true` hoặc truthy → Field trở thành required
- `false` hoặc falsy → Field không required

### 4.2. Ví dụ

#### **Ví dụ 1: Required khi payment type là Cheque**

```json
{
    "fieldname": "cheque_number",
    "label": "Cheque Number",
    "fieldtype": "Data",
    "mandatory_depends_on": "eval:doc.payment_type === 'Cheque'"
}
```

**Giải thích:** Field `cheque_number` chỉ bắt buộc khi `payment_type` là `"Cheque"`.

#### **Ví dụ 2: Required khi status là Active**

```json
{
    "fieldname": "activation_date",
    "label": "Activation Date",
    "fieldtype": "Date",
    "mandatory_depends_on": "eval:doc.status === 'Active'"
}
```

#### **Ví dụ 3: Required khi có giá trị trong field khác**

```json
{
    "fieldname": "delivery_address",
    "label": "Delivery Address",
    "fieldtype": "Small Text",
    "mandatory_depends_on": "eval:doc.delivery_required === 1"
}
```

#### **Ví dụ 4: Kết hợp với depends_on**

```json
{
    "fieldname": "bank_name",
    "label": "Bank Name",
    "fieldtype": "Data",
    "depends_on": "eval:doc.payment_type === 'Bank Transfer'",
    "mandatory_depends_on": "eval:doc.payment_type === 'Bank Transfer'"
}
```

**Giải thích:** Field vừa hiển thị vừa bắt buộc khi payment_type là "Bank Transfer".

---

## 🔷 5. `read_only_depends_on` - Read-only Field

### 5.1. Cú pháp

**Mục đích:** Điều khiển khi nào field trở thành **read-only** (chỉ đọc, không chỉnh sửa).

**Kết quả:**
- `true` hoặc truthy → Field trở thành read-only
- `false` hoặc falsy → Field có thể chỉnh sửa

### 5.2. Ví dụ

#### **Ví dụ 1: Read-only khi document đã submit**

```json
{
    "fieldname": "customer",
    "label": "Customer",
    "fieldtype": "Link",
    "options": "Customer",
    "read_only_depends_on": "eval:doc.docstatus === 1"
}
```

**Giải thích:** Field `customer` chỉ read-only khi document đã được submit.

#### **Ví dụ 2: Read-only khi status là Submitted**

```json
{
    "fieldname": "transaction_date",
    "label": "Transaction Date",
    "fieldtype": "Date",
    "read_only_depends_on": "eval:doc.status === 'Submitted'"
}
```

#### **Ví dụ 3: Read-only khi không phải new document**

```json
{
    "fieldname": "creation_date",
    "label": "Creation Date",
    "fieldtype": "Datetime",
    "read_only_depends_on": "eval:!doc.__islocal"
}
```

**Giải thích:** Field chỉ read-only khi document đã được save (không phải new).

#### **Ví dụ 4: Read-only dựa trên permission**

```json
{
    "fieldname": "salary",
    "label": "Salary",
    "fieldtype": "Currency",
    "read_only_depends_on": "fn:check_salary_permission"
}
```

**Trong client script:**
```javascript
frappe.ui.form.on("Employee", {
    check_salary_permission: function(frm) {
        // Chỉ HR Manager mới có thể edit salary
        return !frappe.user.has_role("HR Manager");
    }
});
```

---

## 🔷 6. `collapsible_depends_on` - Thu gọn Section

### 6.1. Cú pháp

**Mục đích:** Điều khiển khi nào section được **thu gọn mặc định** (collapsed by default).

**Chỉ áp dụng cho:** `fieldtype == "Section Break"` và `collapsible == 1`

**Kết quả:**
- `true` hoặc truthy → Section được thu gọn mặc định
- `false` hoặc falsy → Section mở rộng mặc định

### 6.2. Ví dụ

#### **Ví dụ 1: Thu gọn section khi status là Draft**

```json
{
    "fieldname": "additional_details",
    "label": "Additional Details",
    "fieldtype": "Section Break",
    "collapsible": 1,
    "collapsible_depends_on": "eval:doc.status === 'Draft'"
}
```

**Giải thích:** Section sẽ được thu gọn mặc định khi status là "Draft", mở rộng khi status khác.

#### **Ví dụ 2: Thu gọn khi không có items**

```json
{
    "fieldname": "item_section",
    "label": "Items",
    "fieldtype": "Section Break",
    "collapsible": 1,
    "collapsible_depends_on": "eval:!doc.items || doc.items.length === 0"
}
```

---

## 🔷 7. Các Toán tử và Hàm có thể dùng

### 7.1. Toán tử So sánh

```javascript
// So sánh bằng
doc.status === "Active"
doc.status == "Active"  // Không nên dùng (loose equality)

// So sánh không bằng
doc.status !== "Draft"
doc.status != "Draft"   // Không nên dùng

// So sánh lớn hơn/nhỏ hơn
doc.grand_total > 1000
doc.grand_total >= 1000
doc.grand_total < 5000
doc.grand_total <= 5000
```

### 7.2. Toán tử Logic

```javascript
// AND
doc.status === "Active" && doc.docstatus === 0

// OR
doc.payment_type === "Cheque" || doc.payment_type === "Bank Transfer"

// NOT
!doc.__islocal
doc.status !== "Cancelled"
```

### 7.3. Kiểm tra Giá trị

```javascript
// Kiểm tra có giá trị
doc.customer != null
doc.customer != undefined
doc.customer != ""

// Kiểm tra rỗng
doc.notes == null
doc.notes == ""
!doc.notes

// Kiểm tra array có phần tử
doc.items && doc.items.length > 0
doc.items.length > 0  // Có thể lỗi nếu items là null
```

### 7.4. Array Methods

```javascript
// Kiểm tra giá trị trong array
["Active", "Submitted"].includes(doc.status)
["Cheque", "Bank Transfer"].indexOf(doc.payment_type) !== -1

// Kiểm tra length
doc.items.length > 0
doc.items.length === 0
```

### 7.5. String Methods

```javascript
// Kiểm tra string
doc.customer_name.startsWith("VIP")
doc.customer_name.endsWith("Ltd")
doc.customer_name.includes("Test")
doc.customer_name.length > 10
```

### 7.6. Number Operations

```javascript
// So sánh số
doc.grand_total > 1000
doc.qty >= 10
doc.discount < 50

// Tính toán
doc.grand_total - doc.discount > 0
doc.qty * doc.rate > 1000
```

---

## 🔷 8. Ví dụ Thực tế Tổng hợp

### 8.1. Payment Voucher - Phụ thuộc Payment Type

```json
[
    {
        "fieldname": "payment_type",
        "label": "Payment Type",
        "fieldtype": "Select",
        "options": "Cash\nCheque\nBank Transfer\nCredit Card",
        "reqd": 1
    },
    {
        "fieldname": "cheque_section",
        "label": "Cheque Details",
        "fieldtype": "Section Break",
        "depends_on": "eval:doc.payment_type === 'Cheque'"
    },
    {
        "fieldname": "cheque_number",
        "label": "Cheque Number",
        "fieldtype": "Data",
        "depends_on": "eval:doc.payment_type === 'Cheque'",
        "mandatory_depends_on": "eval:doc.payment_type === 'Cheque'"
    },
    {
        "fieldname": "cheque_date",
        "label": "Cheque Date",
        "fieldtype": "Date",
        "depends_on": "eval:doc.payment_type === 'Cheque'",
        "mandatory_depends_on": "eval:doc.payment_type === 'Cheque'"
    },
    {
        "fieldname": "bank_section",
        "label": "Bank Details",
        "fieldtype": "Section Break",
        "depends_on": "eval:doc.payment_type === 'Bank Transfer'"
    },
    {
        "fieldname": "bank_name",
        "label": "Bank Name",
        "fieldtype": "Link",
        "options": "Bank",
        "depends_on": "eval:doc.payment_type === 'Bank Transfer'",
        "mandatory_depends_on": "eval:doc.payment_type === 'Bank Transfer'"
    },
    {
        "fieldname": "account_number",
        "label": "Account Number",
        "fieldtype": "Data",
        "depends_on": "eval:doc.payment_type === 'Bank Transfer'",
        "mandatory_depends_on": "eval:doc.payment_type === 'Bank Transfer'"
    }
]
```

### 8.2. Sales Order - Phụ thuộc Status và Docstatus

```json
[
    {
        "fieldname": "status",
        "label": "Status",
        "fieldtype": "Select",
        "options": "Draft\nSubmitted\nCancelled",
        "read_only_depends_on": "eval:doc.docstatus === 1"
    },
    {
        "fieldname": "cancellation_section",
        "label": "Cancellation Details",
        "fieldtype": "Section Break",
        "depends_on": "eval:doc.status === 'Cancelled' || doc.docstatus === 2"
    },
    {
        "fieldname": "cancellation_reason",
        "label": "Cancellation Reason",
        "fieldtype": "Small Text",
        "depends_on": "eval:doc.status === 'Cancelled' || doc.docstatus === 2",
        "mandatory_depends_on": "eval:doc.status === 'Cancelled' || doc.docstatus === 2"
    },
    {
        "fieldname": "delivery_section",
        "label": "Delivery Information",
        "fieldtype": "Section Break",
        "depends_on": "eval:doc.status === 'Submitted' && doc.delivery_required === 1"
    },
    {
        "fieldname": "delivery_date",
        "label": "Delivery Date",
        "fieldtype": "Date",
        "depends_on": "eval:doc.status === 'Submitted' && doc.delivery_required === 1",
        "mandatory_depends_on": "eval:doc.status === 'Submitted' && doc.delivery_required === 1"
    }
]
```

### 8.3. Employee - Phụ thuộc Employee Type

```json
[
    {
        "fieldname": "employee_type",
        "label": "Employee Type",
        "fieldtype": "Select",
        "options": "Full Time\nPart Time\nContract\nIntern"
    },
    {
        "fieldname": "contract_section",
        "label": "Contract Details",
        "fieldtype": "Section Break",
        "depends_on": "eval:doc.employee_type === 'Contract'"
    },
    {
        "fieldname": "contract_start_date",
        "label": "Contract Start Date",
        "fieldtype": "Date",
        "depends_on": "eval:doc.employee_type === 'Contract'",
        "mandatory_depends_on": "eval:doc.employee_type === 'Contract'"
    },
    {
        "fieldname": "contract_end_date",
        "label": "Contract End Date",
        "fieldtype": "Date",
        "depends_on": "eval:doc.employee_type === 'Contract'",
        "mandatory_depends_on": "eval:doc.employee_type === 'Contract'"
    },
    {
        "fieldname": "salary",
        "label": "Salary",
        "fieldtype": "Currency",
        "read_only_depends_on": "eval:doc.docstatus === 1 || doc.employee_type === 'Intern'"
    }
]
```

### 8.4. Invoice - Phụ thuộc nhiều điều kiện

```json
[
    {
        "fieldname": "discount_type",
        "label": "Discount Type",
        "fieldtype": "Select",
        "options": "None\nPercentage\nAmount"
    },
    {
        "fieldname": "discount_percentage",
        "label": "Discount Percentage",
        "fieldtype": "Percent",
        "depends_on": "eval:doc.discount_type === 'Percentage'",
        "mandatory_depends_on": "eval:doc.discount_type === 'Percentage'"
    },
    {
        "fieldname": "discount_amount",
        "label": "Discount Amount",
        "fieldtype": "Currency",
        "depends_on": "eval:doc.discount_type === 'Amount'",
        "mandatory_depends_on": "eval:doc.discount_type === 'Amount'"
    },
    {
        "fieldname": "tax_section",
        "label": "Tax Information",
        "fieldtype": "Section Break",
        "depends_on": "eval:doc.grand_total > 0 && doc.docstatus === 0"
    },
    {
        "fieldname": "tax_rate",
        "label": "Tax Rate",
        "fieldtype": "Percent",
        "depends_on": "eval:doc.grand_total > 0 && doc.docstatus === 0",
        "read_only_depends_on": "eval:doc.docstatus === 1"
    }
]
```

---

## 🔷 9. Child Table Fields

### 9.1. Truy cập Parent Document

Trong child table fields, bạn có thể truy cập parent document qua `parent`:

```json
{
    "fieldname": "warehouse",
    "label": "Warehouse",
    "fieldtype": "Link",
    "options": "Warehouse",
    "depends_on": "eval:parent.company === 'Test Company'"
}
```

**Giải thích:** Field trong child table chỉ hiển thị khi `company` của parent document là "Test Company".

### 9.2. Truy cập Child Document

Trong child table fields, `doc` là child document:

```json
{
    "fieldname": "item_code",
    "label": "Item Code",
    "fieldtype": "Link",
    "options": "Item"
},
{
    "fieldname": "batch_no",
    "label": "Batch Number",
    "fieldtype": "Data",
    "depends_on": "eval:doc.item_code && doc.item_code.includes('BATCH')"
}
```

**Giải thích:** Field `batch_no` chỉ hiển thị khi `item_code` có chứa "BATCH".

### 9.3. Ví dụ Child Table

```json
// Parent field
{
    "fieldname": "items",
    "label": "Items",
    "fieldtype": "Table",
    "options": "Sales Order Item"
}

// Child table fields
[
    {
        "fieldname": "item_code",
        "label": "Item Code",
        "fieldtype": "Link",
        "options": "Item"
    },
    {
        "fieldname": "warehouse",
        "label": "Warehouse",
        "fieldtype": "Link",
        "options": "Warehouse",
        "depends_on": "eval:parent.delivery_required === 1"
    },
    {
        "fieldname": "serial_no",
        "label": "Serial Number",
        "fieldtype": "Data",
        "depends_on": "eval:doc.item_code && doc.has_serial_no === 1",
        "mandatory_depends_on": "eval:doc.item_code && doc.has_serial_no === 1"
    }
]
```

---

## 🔷 10. Sử dụng Function (`fn:`)

### 10.1. Cách sử dụng

**Bước 1:** Tạo function trong client script:

```javascript
// File: erpnext/selling/doctype/sales_order/sales_order.js
frappe.ui.form.on("Sales Order", {
    check_delivery_required: function(frm) {
        // Logic phức tạp
        if (frm.doc.customer_type === "Retail" && frm.doc.grand_total > 1000) {
            return true;
        }
        if (frm.doc.delivery_method === "Express") {
            return true;
        }
        return false;
    }
});
```

**Bước 2:** Sử dụng trong DocField:

```json
{
    "fieldname": "delivery_date",
    "label": "Delivery Date",
    "fieldtype": "Date",
    "depends_on": "fn:check_delivery_required"
}
```

### 10.2. Function với Return Value

Function phải return `true` hoặc `false`:

```javascript
frappe.ui.form.on("Purchase Order", {
    check_approval_required: function(frm) {
        // Return true nếu cần approval
        return frm.doc.grand_total > 10000 && frm.doc.status === "Draft";
    }
});
```

```json
{
    "fieldname": "approver",
    "label": "Approver",
    "fieldtype": "Link",
    "options": "User",
    "depends_on": "fn:check_approval_required",
    "mandatory_depends_on": "fn:check_approval_required"
}
```

---

## 🔷 11. Lưu ý Quan trọng

### 11.1. So sánh String

**Luôn dùng `===` thay vì `==`:**

```javascript
// ✅ Đúng
depends_on: "eval:doc.status === 'Active'"

// ❌ Sai (có thể gây lỗi)
depends_on: "eval:doc.status == 'Active'"
```

### 11.2. Kiểm tra Null/Undefined

**Luôn kiểm tra null/undefined trước khi truy cập:**

```javascript
// ✅ Đúng
depends_on: "eval:doc.customer && doc.customer != ''"

// ❌ Có thể lỗi nếu customer là null
depends_on: "eval:doc.customer.length > 0"
```

### 11.3. Array Length

**Kiểm tra array tồn tại trước khi check length:**

```javascript
// ✅ Đúng
depends_on: "eval:doc.items && doc.items.length > 0"

// ❌ Có thể lỗi nếu items là null
depends_on: "eval:doc.items.length > 0"
```

### 11.4. Docstatus

**Docstatus là số, không phải string:**

```javascript
// ✅ Đúng
depends_on: "eval:doc.docstatus === 0"  // Draft
depends_on: "eval:doc.docstatus === 1"  // Submitted
depends_on: "eval:doc.docstatus === 2"  // Cancelled

// ❌ Sai
depends_on: "eval:doc.docstatus === '0'"
```

### 11.5. Check Field

**Check field trả về 0 hoặc 1:**

```javascript
// ✅ Đúng
depends_on: "eval:doc.delivery_required === 1"
depends_on: "eval:doc.delivery_required === 0"

// Hoặc
depends_on: "eval:!!doc.delivery_required"  // truthy check
```

### 11.6. Nested Fields

**Truy cập nested fields:**

```javascript
// ✅ Đúng
depends_on: "eval:doc.customer && doc.customer.name"
depends_on: "eval:doc.items && doc.items[0] && doc.items[0].item_code"
```

### 11.7. Performance

**Tránh expression phức tạp:**

```javascript
// ✅ Tốt - Đơn giản
depends_on: "eval:doc.status === 'Active'"

// ⚠️ Tránh - Quá phức tạp
depends_on: "eval:doc.items && doc.items.filter(i => i.qty > 0).length > 0 && doc.customer && doc.customer.includes('VIP')"
```

---

## 🔷 12. Debugging

### 12.1. Kiểm tra Expression

Bạn có thể test expression trong browser console:

```javascript
// Trong browser console
const doc = cur_frm.doc;
const expression = "doc.status === 'Active'";
const result = frappe.utils.eval(expression, { doc, parent: cur_frm.doc });
console.log(result); // true hoặc false
```

### 12.2. Common Errors

**Error: Invalid "depends_on" expression**

- Kiểm tra cú pháp JavaScript
- Kiểm tra fieldname có đúng không
- Kiểm tra dấu ngoặc kép/nháy đơn

**Field không hiển thị khi nên hiển thị:**

- Kiểm tra expression có return `true` không
- Kiểm tra giá trị của field phụ thuộc
- Kiểm tra docstatus, __islocal, etc.

**Field hiển thị khi không nên hiển thị:**

- Kiểm tra logic expression
- Kiểm tra toán tử logic (&&, ||)
- Kiểm tra so sánh (===, !==)

---

## 🔷 13. Best Practices

### 13.1. Ưu tiên `eval:` cho logic đơn giản

```javascript
// ✅ Tốt - Logic đơn giản
depends_on: "eval:doc.status === 'Active'"

// ✅ Tốt - Logic phức tạp
depends_on: "fn:check_complex_condition"
```

### 13.2. Sử dụng `fn:` cho logic phức tạp

```javascript
// Logic phức tạp nên đặt trong function
frappe.ui.form.on("Sales Order", {
    check_complex_condition: function(frm) {
        // Nhiều điều kiện phức tạp
        if (condition1 && condition2) {
            // Logic phức tạp
            return true;
        }
        return false;
    }
});
```

### 13.3. Kết hợp nhiều dependency properties

```json
{
    "fieldname": "special_discount",
    "label": "Special Discount",
    "fieldtype": "Percent",
    "depends_on": "eval:doc.customer_type === 'VIP'",
    "mandatory_depends_on": "eval:doc.customer_type === 'VIP' && doc.grand_total > 10000",
    "read_only_depends_on": "eval:doc.docstatus === 1"
}
```

### 13.4. Document các dependency phức tạp

```json
{
    "fieldname": "approval_section",
    "label": "Approval Details",
    "fieldtype": "Section Break",
    "description": "Only visible when amount > 10000 and status is Draft",
    "depends_on": "eval:doc.grand_total > 10000 && doc.status === 'Draft'"
}
```

---

## 🔷 14. Tóm tắt

### 14.1. Các Dependency Properties

| Property | Mục đích | Kết quả |
|----------|----------|---------|
| `depends_on` | Điều khiển hiển thị | `true` = hiển thị, `false` = ẩn |
| `mandatory_depends_on` | Điều khiển required | `true` = required, `false` = optional |
| `read_only_depends_on` | Điều khiển read-only | `true` = read-only, `false` = editable |
| `collapsible_depends_on` | Điều khiển collapsed | `true` = collapsed, `false` = expanded |

### 14.2. Các Cách viết Expression

1. **Fieldname đơn giản:** `"status"` - Hiển thị nếu field có giá trị
2. **Boolean:** `true` hoặc `false` - Luôn true/false
3. **Function:** `function(doc) { return ... }` - Function nhận doc
4. **eval:** `"eval:doc.status === 'Active'"` - JavaScript expression
5. **fn:** `"fn:check_status"` - Gọi function trong client script

### 14.3. Context Variables

- **`doc`** - Document hiện tại
- **`parent`** - Parent document (trong child table)

### 14.4. Common Patterns

```javascript
// So sánh bằng
doc.status === "Active"

// So sánh không bằng
doc.status !== "Draft"

// So sánh số
doc.grand_total > 1000

// Kiểm tra có giá trị
doc.customer != null && doc.customer != ""

// Kiểm tra array
doc.items && doc.items.length > 0

// Kiểm tra trong array
["Active", "Submitted"].includes(doc.status)

// Kết hợp điều kiện
doc.status === "Active" && doc.docstatus === 0

// Docstatus
doc.docstatus === 0  // Draft
doc.docstatus === 1  // Submitted
doc.docstatus === 2  // Cancelled

// New document
doc.__islocal  // true nếu chưa save
!doc.__islocal  // true nếu đã save
```

---

## 🔗 Tài liệu tham khảo

- **File source:** `apps/frappe/frappe/public/js/frappe/form/layout.js` (function `evaluate_depends_on_value`)
- **Frappe Documentation:** [Field Dependencies](https://frappeframework.com/docs/user/en/desk/customize/customize-form#field-dependencies)
