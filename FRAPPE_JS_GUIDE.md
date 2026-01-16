# Frappe JS - Hướng dẫn cơ bản

Hướng dẫn này giải thích cách **Frappe JavaScript** hoạt động, từ khởi động đến các thành phần chính.

---

## 📦 1. Khởi động (Boot Process)

### 1.1. Entry Point

Khi trang web được load, Frappe JS bắt đầu từ file `desk.js`:

```javascript
$(document).ready(function () {
    frappe.start_app();
});
```

### 1.2. Application Class

`frappe.Application` là class chính quản lý toàn bộ ứng dụng:

```javascript
frappe.Application = class Application {
    constructor() {
        this.startup();
    }

    startup() {
        // 1. Khởi tạo realtime và model
        frappe.realtime.init();
        frappe.model.init();

        // 2. Load bootinfo (dữ liệu từ server)
        this.load_bootinfo();
        
        // 3. Load user permissions
        this.load_user_permissions();
        
        // 4. Tạo UI (navbar, sidebar)
        this.make_nav_bar();
        this.make_sidebar();
        
        // 5. Setup router và routing
        this.set_route();
        
        // 6. Trigger events
        $(document).trigger("startup");
        $(document).trigger("app_ready");
    }
}
```

### 1.3. Boot Info (`frappe.boot`)

`frappe.boot` chứa **tất cả dữ liệu** được load từ server khi khởi động:

```javascript
// Trong HTML template (app.html)
frappe.boot = {{ boot | json }};
```

**Nội dung chính trong `frappe.boot`:**
- `user` - Thông tin user hiện tại
- `sysdefaults` - System defaults
- `modules` - Danh sách modules
- `desk_settings` - Cài đặt desktop
- `__messages` - Translations
- `docs` - Metadata của DocTypes
- `page_info` - Thông tin pages
- `lang` - Ngôn ngữ hiện tại
- `csrf_token` - CSRF token
- Và nhiều thông tin khác...

**Ví dụ sử dụng:**
```javascript
// Lấy thông tin user
const userName = frappe.boot.user.name;

// Lấy system defaults
const company = frappe.boot.sysdefaults.company;

// Lấy translations
const messages = frappe.boot.__messages;
```

---

## 🔷 2. Namespace và Cấu trúc

### 2.1. `frappe.provide()`

Frappe sử dụng **namespace pattern** để tổ chức code:

```javascript
frappe.provide = function (namespace) {
    var nsl = namespace.split(".");
    var parent = window;
    for (var i = 0; i < nsl.length; i++) {
        var n = nsl[i];
        if (!parent[n]) {
            parent[n] = {};
        }
        parent = parent[n];
    }
    return parent;
};
```

**Ví dụ:**
```javascript
// Tạo namespace
frappe.provide("erpnext.buying");

// Sử dụng
erpnext.buying.BuyingController = class { ... };
```

**Các namespace chính:**
- `frappe.model` - Quản lý documents và data
- `frappe.ui.form` - Form handling
- `frappe.ui.list` - List view
- `frappe.router` - Routing
- `frappe.utils` - Utility functions
- `frappe.db` - Database operations (AJAX)
- `frappe.call` - API calls

---

## 🗺️ 3. Router và Routing

### 3.1. Router Object

`frappe.router` quản lý routing trong ứng dụng:

```javascript
frappe.router = {
    current_route: null,
    routes: {},
    factory_views: ["form", "list", "report", "tree", "print", "dashboard"],
    
    // Set route
    set_route: function(path) { ... },
    
    // Route đến một path
    route: function() { ... },
    
    // Kiểm tra xem có phải app route không
    is_app_route: function(pathname) { ... }
};
```

### 3.2. Cách sử dụng Router

```javascript
// Chuyển đến một route
frappe.set_route("Form", "Customer", "CUST-001");
frappe.set_route("List", "Customer");
frappe.set_route("Report", "Sales Register");

// Lấy route hiện tại
const currentRoute = frappe.router.current_route;

// Lắng nghe thay đổi route
$(document).on("route-change", function(e, route) {
    console.log("Route changed to:", route);
});
```

### 3.3. Route Format

- **Form**: `Form/Doctype/Docname` hoặc `Form/Doctype/New Doctype`
- **List**: `List/Doctype`
- **Report**: `Report/Report Name`
- **Page**: `Page/Page Name`
- **Custom**: Tùy chỉnh theo app

---

## 📝 4. Form Lifecycle

### 4.1. FrappeForm Class

`frappe.ui.form.Form` là class chính quản lý form:

```javascript
frappe.ui.form.Form = class FrappeForm {
    constructor(doctype, parent, in_form, doctype_layout_name) {
        this.doctype = doctype;
        this.docname = "";
        this.doc = {};  // Document data
        this.meta = {}; // DocType metadata
        this.cscript = new frappe.ui.form.Controller({ frm: this });
        this.fields_dict = {}; // Dictionary of fields
        this.fields = [];      // Array of fields
        // ...
    }
}
```

### 4.2. Form Lifecycle Methods

**1. `refresh()` - Load document và render form:**
```javascript
refresh(docname) {
    // Load document từ server
    // Render form
    // Trigger events
}
```

**2. `setup()` - Setup form (chỉ chạy 1 lần khi form được tạo):**
```javascript
// Trong client script
frappe.ui.form.on("Customer", {
    setup: function(frm) {
        // Chạy 1 lần khi form được tạo
        console.log("Form setup");
    }
});
```

**3. `onload()` - Chạy khi document được load:**
```javascript
frappe.ui.form.on("Customer", {
    onload: function(frm) {
        // Chạy mỗi khi document được load
        console.log("Document loaded:", frm.doc);
    }
});
```

**4. `refresh()` - Chạy khi form được refresh:**
```javascript
frappe.ui.form.on("Customer", {
    refresh: function(frm) {
        // Chạy mỗi khi form được refresh
        console.log("Form refreshed");
    }
});
```

**5. Field-specific handlers:**
```javascript
frappe.ui.form.on("Customer", {
    customer_name: function(frm) {
        // Chạy khi field "customer_name" thay đổi
        console.log("Customer name changed:", frm.doc.customer_name);
    },
    
    "customer_name, email_id": function(frm) {
        // Chạy khi một trong các field thay đổi
    }
});
```

### 4.3. Form Events Flow

```
1. Form được tạo
   └─> setup() (chỉ 1 lần)

2. Document được load
   └─> onload()
   └─> onload_post_render()

3. Form được refresh
   └─> refresh()
   └─> refresh_fields()

4. Field thay đổi
   └─> field-specific handler
   └─> validate() (nếu có)
```

### 4.4. Global Variable `cur_frm`

`cur_frm` là biến global trỏ đến form hiện tại đang active:

```javascript
// Trong client script file
cur_frm.cscript.custom_method = function() {
    // Custom method
};

// Hoặc trong form event handler
frappe.ui.form.on("Customer", {
    refresh: function(frm) {
        // frm và cur_frm đều trỏ đến cùng một object
        console.log(frm === cur_frm); // true
    }
});
```

---

## 🔧 4.5. Form Methods (Các phương thức của Form)

### 4.5.1. Custom Buttons

**`add_custom_button(label, fn, group)`** - Thêm custom button vào toolbar:

```javascript
frappe.ui.form.on("Sales Order", {
    refresh: function(frm) {
        // Thêm button vào toolbar chính
        frm.add_custom_button(__("Create Delivery Note"), function() {
            frappe.model.open_mapped_doc({
                method: "erpnext.selling.doctype.sales_order.sales_order.make_delivery_note",
                frm: frm
            });
        });
        
        // Thêm button vào group "Actions"
        frm.add_custom_button(__("Print"), function() {
            frm.print_doc();
        }, __("Actions"));
        
        // Thêm button vào group "Create"
        frm.add_custom_button(__("Quotation"), function() {
            frappe.set_route("Form", "Quotation", "New Quotation");
        }, __("Create"));
    }
});
```

**`remove_custom_button(label, group)`** - Xóa custom button:

```javascript
frm.remove_custom_button("Create Delivery Note");
frm.remove_custom_button("Print", "Actions");
```

**`clear_custom_buttons()`** - Xóa tất cả custom buttons:

```javascript
frm.clear_custom_buttons();
```

### 4.5.2. Field Queries (`get_query`)

**Cho fields trong parent form:**

```javascript
frappe.ui.form.on("Purchase Order", {
    setup: function(frm) {
        // Set query cho field trong parent form
        frm.set_query("supplier", function() {
            return {
                filters: {
                    "supplier_type": "Company"
                }
            };
        });
    }
});
```

**Cho fields trong child table (grid):**

```javascript
frappe.ui.form.on("Purchase Order", {
    setup: function(frm) {
        // Set query cho field trong child table
        // Syntax: frm.set_query(child_field, parent_table_field, query_function)
        frm.set_query("item_code", "items", function(doc, cdt, cdn) {
            // doc: main document (Purchase Order)
            // cdt: child doctype name ("Purchase Order Item")
            // cdn: child document name
            return {
                filters: {
                    "is_stock_item": 1
                }
            };
        });
        
        // Hoặc dùng cách trực tiếp
        frm.fields_dict["items"].grid.get_field("item_code").get_query = function(doc, cdt, cdn) {
            return {
                filters: {
                    "is_stock_item": 1,
                    "company": doc.company  // Lọc theo company của main doc
                }
            };
        };
    }
});
```

**Sử dụng custom server-side query:**

```javascript
frappe.ui.form.on("Purchase Voucher", {
    setup: function(frm) {
        // Sử dụng custom Python method để query
        frm.fields_dict["items"].grid.get_field("payable_account").get_query = function(doc, cdt, cdn) {
            return {
                query: "mbwnext_advanced_accounting.mbwnext_advanced_accounting.doctype.purchase_voucher.purchase_voucher.payable_ac",
                filters: {
                    company: doc.company
                }
            };
        };
    }
});
```

**Ví dụ từ ảnh:**

```javascript
// Cho field vat_account trong grid service_pv_items
frm.fields_dict["service_pv_items"].grid.get_field("vat_account").get_query = function(doc, cdt, cdn) {
    return {
        filters: {
            company: doc.company
        }
    };
};

// Cho field payable_account trong grid items với custom query
frm.fields_dict["items"].grid.get_field("payable_account").get_query = function(doc, cdt, cdn) {
    return {
        query: "mbwnext_advanced_accounting.mbwnext_advanced_accounting.doctype.purchase_voucher.purchase_voucher.payable_ac",
        filters: {
            company: doc.company
        }
    };
};
```

### 4.5.3. Auto-fetch (`add_fetch`)

**`add_fetch(link_field, source_field, target_field, target_doctype)`** - Tự động fetch giá trị từ linked document:

```javascript
frappe.ui.form.on("Sales Order", {
    setup: function(frm) {
        // Khi customer được chọn, tự động fetch customer_name và email_id
        frm.add_fetch("customer", "customer_name", "customer_name");
        frm.add_fetch("customer", "email_id", "contact_email");
        
        // Cho child table
        frm.add_fetch("item_code", "item_name", "item_name", "Sales Order Item");
        frm.add_fetch("item_code", "standard_rate", "rate", "Sales Order Item");
    }
});
```

### 4.5.4. Set và Get Values

**`set_value(field, value, if_missing, skip_dirty_trigger)`** - Set giá trị cho field:

```javascript
// Set một field
frm.set_value("customer_name", "New Customer Name");

// Set nhiều fields cùng lúc
frm.set_value({
    "customer_name": "New Customer Name",
    "email_id": "customer@example.com",
    "phone": "1234567890"
});

// Set chỉ khi field chưa có giá trị
frm.set_value("transaction_date", frappe.datetime.get_today(), true);

// Set cho child table field
frm.set_value("items", [
    { item_code: "ITEM-001", qty: 10 },
    { item_code: "ITEM-002", qty: 20 }
]);
```

**`get_value(fieldname)`** - Lấy giá trị field (từ `frm.doc`):

```javascript
// Lấy giá trị
const customerName = frm.doc.customer_name;
const grandTotal = frm.doc.grand_total;

// Hoặc dùng get_field
const field = frm.get_field("customer_name");
const value = field.get_value();
```

**`get_formatted(fieldname)`** - Lấy giá trị đã format:

```javascript
const formattedDate = frm.get_formatted("transaction_date");
const formattedCurrency = frm.get_formatted("grand_total");
```

### 4.5.5. Field Manipulation

**`get_field(fieldname)`** - Lấy field object:

```javascript
const customerField = frm.get_field("customer");
customerField.set_value("CUST-001");
```

**`refresh_field(fieldname)`** - Refresh field (re-render):

```javascript
frm.set_value("customer_name", "New Name");
frm.refresh_field("customer_name"); // Refresh để hiển thị giá trị mới
```

**`set_df_property(fieldname, property, value, docname, table_field, table_row_name)`** - Set property của DocField:

```javascript
// Set read_only
frm.set_df_property("customer_name", "read_only", 1);

// Set hidden
frm.set_df_property("email_id", "hidden", 1);

// Set required
frm.set_df_property("phone", "reqd", 1);

// Set options (cho Select field)
frm.set_df_property("status", "options", "Draft\nSubmitted\nCancelled");

// Set cho child table field
frm.set_df_property("item_code", "read_only", 1, null, "items");
```

**`get_docfield(fieldname1, fieldname2)`** - Lấy DocField object:

```javascript
// Lấy DocField của parent
const customerField = frm.get_docfield("customer");

// Lấy DocField của child table
const itemCodeField = frm.get_docfield("items", "item_code");
```

### 4.5.6. Field Toggle Methods

**`toggle_enable(fnames, enable)`** - Enable/disable fields:

```javascript
// Disable fields
frm.toggle_enable(["customer_name", "email_id"], false);

// Enable fields
frm.toggle_enable(["customer_name", "email_id"], true);
```

**`toggle_reqd(fnames, mandatory)`** - Set required/optional:

```javascript
// Set required
frm.toggle_reqd(["phone", "email_id"], true);

// Set optional
frm.toggle_reqd(["phone", "email_id"], false);
```

**`toggle_display(fnames, show)`** - Show/hide fields:

```javascript
// Hide fields
frm.toggle_display(["email_id", "phone"], false);

// Show fields
frm.toggle_display(["email_id", "phone"], true);
```

### 4.5.7. Child Table Methods

**`add_child(fieldname, values)`** - Thêm row mới vào child table:

```javascript
// Thêm row mới
const newRow = frm.add_child("items");
newRow.item_code = "ITEM-001";
newRow.qty = 10;
newRow.rate = 100;

// Hoặc set values ngay
const newRow = frm.add_child("items", {
    item_code: "ITEM-001",
    qty: 10,
    rate: 100
});

frm.refresh_field("items");
```

**`clear_table(fieldname)`** - Xóa tất cả rows trong child table:

```javascript
frm.clear_table("items");
frm.refresh_field("items");
```

### 4.5.8. Form State Methods

**`is_new()`** - Kiểm tra document có phải mới không:

```javascript
if (frm.is_new()) {
    // Document chưa được save
    frm.set_value("transaction_date", frappe.datetime.get_today());
}
```

**`is_dirty()`** - Kiểm tra form có thay đổi chưa:

```javascript
if (frm.is_dirty()) {
    frappe.confirm("You have unsaved changes. Are you sure you want to leave?");
}
```

**`dirty()`** - Đánh dấu form là dirty (có thay đổi):

```javascript
frm.dirty(); // Mark form as changed
```

**`save()`** - Save document:

```javascript
frm.save(); // Save document

// Save với callback
frm.save().then(() => {
    frappe.msgprint("Document saved successfully");
});
```

**`save_notify()`** - Save và hiển thị notification:

```javascript
frm.save_notify();
```

**`reload_doc()`** - Reload document từ server:

```javascript
frm.reload_doc(); // Reload document
```

### 4.5.9. API Call Methods

**`call(method, args, callback)`** - Gọi server method:

```javascript
// Cách 1: Truyền method name
frm.call({
    method: "erpnext.selling.doctype.sales_order.sales_order.get_item_details",
    args: {
        item_code: "ITEM-001",
        warehouse: "Main Warehouse"
    },
    callback: function(r) {
        if (r.message) {
            frm.set_value("items", r.message);
        }
    }
});

// Cách 2: Truyền method name trực tiếp
frm.call("get_item_details", {
    item_code: "ITEM-001"
}, function(r) {
    if (r.message) {
        frm.set_value("items", r.message);
    }
});
```

### 4.5.10. Navigation Methods

**`scroll_to_field(fieldname)`** - Scroll đến field:

```javascript
frm.scroll_to_field("customer_name");
```

**`scroll_to_element()`** - Scroll đến element được chỉ định trong route_options:

```javascript
frappe.route_options = {
    scroll_to: {
        fieldname: "customer_name"
    }
};
```

### 4.5.11. Permission Methods

**`has_perm(ptype)`** - Kiểm tra permission:

```javascript
if (frm.has_perm("write")) {
    // User có quyền write
}

if (frm.has_perm("delete")) {
    // User có quyền delete
}
```

**`set_read_only()`** - Set form thành read-only dựa trên permissions:

```javascript
frm.set_read_only();
```

### 4.5.12. Other Useful Methods

**`trigger(event, doctype, docname)`** - Trigger event manually:

```javascript
frm.trigger("customer"); // Trigger customer field change
```

**`field_map(fnames, fn)`** - Map function lên nhiều fields:

```javascript
frm.field_map(["customer_name", "email_id"], function(field) {
    field.read_only = 1;
});
```

**`set_currency_labels(fields_list, currency, parentfield)`** - Set currency label cho fields:

```javascript
frm.set_currency_labels(["grand_total", "total"], "USD");
```

**`disable_save(disable)`** - Disable/enable save button:

```javascript
frm.disable_save(true);  // Disable save
frm.disable_save(false); // Enable save
```

### 4.5.13. Document Actions Methods

**`save(save_action, callback, btn, on_error)`** - Save document:

```javascript
// Save document
frm.save();

// Save với callback
frm.save("Save", function(r) {
    if (!r.exc) {
        frappe.msgprint("Document saved successfully");
    }
});

// Save với error handler
frm.save("Save", function(r) {
    // Success
}, null, function() {
    // Error
});
```

**`save_or_update()`** - Save hoặc Update tùy theo docstatus:

```javascript
frm.save_or_update(); // Tự động chọn Save hoặc Update
```

**`savesubmit(btn, callback, on_error)`** - Save và Submit:

```javascript
frm.savesubmit(null, function() {
    frappe.msgprint("Document submitted successfully");
});
```

**`savecancel(btn, callback, on_error)`** - Save và Cancel:

```javascript
frm.savecancel(null, function() {
    frappe.msgprint("Document cancelled successfully");
});
```

**`discard(btn, callback, on_error)`** - Discard document (xóa bỏ thay đổi):

```javascript
frm.discard(null, function() {
    frappe.msgprint("Document discarded");
});
```

**`savetrash()`** - Delete document:

```javascript
frm.savetrash(); // Xóa document
```

**`amend_doc()`** - Amend document (tạo bản sửa đổi):

```javascript
frm.amend_doc(); // Tạo amended document
```

### 4.5.14. Document State Methods

**`reload_doc()`** - Reload document từ server:

```javascript
frm.reload_doc(); // Reload document
```

**`switch_doc(docname)`** - Chuyển sang document khác:

```javascript
frm.switch_doc("CUST-002"); // Chuyển sang document khác
```

**`copy_doc(onload, from_amend)`** - Copy document:

```javascript
// Copy document
frm.copy_doc(function(newdoc) {
    // Callback khi document được copy
    newdoc.customer_name = "New Customer";
});

// Copy từ amended document
frm.copy_doc(null, true);
```

**`rename_doc()`** - Rename document:

```javascript
frm.rename_doc(); // Mở dialog để rename
```

### 4.5.15. Navigation Methods

**`navigate_records(prev)`** - Navigate đến record trước/sau:

```javascript
frm.navigate_records(0); // Next record
frm.navigate_records(1); // Previous record
```

**`print_doc()`** - Print document:

```javascript
frm.print_doc(); // Mở print view
```

**`email_doc(message)`** - Email document:

```javascript
frm.email_doc("Please review this document");
```

**`share_doc()`** - Share document:

```javascript
frm.share_doc(); // Mở share dialog
```

### 4.5.16. Form State Check Methods

**`is_new()`** - Kiểm tra document có phải mới không:

```javascript
if (frm.is_new()) {
    // Document chưa được save
}
```

**`is_dirty()`** - Kiểm tra form có thay đổi chưa:

```javascript
if (frm.is_dirty()) {
    // Form có thay đổi chưa save
}
```

**`dirty()`** - Đánh dấu form là dirty:

```javascript
frm.dirty(); // Mark form as changed
```

**`is_form_builder()`** - Kiểm tra có phải form builder không:

```javascript
if (frm.is_form_builder()) {
    // Đang ở form builder
}
```

### 4.5.17. Permission Methods

**`has_perm(ptype)`** - Kiểm tra permission:

```javascript
if (frm.has_perm("write")) {
    // User có quyền write
}
```

**`get_perm(permlevel, access_type)`** - Lấy permission:

```javascript
const canWrite = frm.get_perm(0, "write");
```

**`has_read_permission()`** - Kiểm tra quyền đọc:

```javascript
if (frm.has_read_permission()) {
    // User có quyền đọc
}
```

**`fetch_permissions()`** - Fetch permissions từ server:

```javascript
frm.fetch_permissions();
```

### 4.5.18. Form Control Methods

**`enable_save()`** - Enable save button:

```javascript
frm.enable_save();
```

**`disable_save(set_dirty = false)`** - Disable save button:

```javascript
frm.disable_save(); // Disable save
frm.disable_save(true); // Disable save nhưng vẫn cho phép dirty
```

**`disable_form()`** - Disable toàn bộ form:

```javascript
frm.disable_form(); // Set read-only và disable save
```

**`set_read_only()`** - Set form thành read-only:

```javascript
frm.set_read_only();
```

### 4.5.19. Document Info Methods

**`get_doc()`** - Lấy document object:

```javascript
const doc = frm.get_doc(); // Lấy document từ locals
```

**`get_docinfo()`** - Lấy document info (comments, versions, etc.):

```javascript
const docinfo = frm.get_docinfo();
console.log(docinfo.comments);
console.log(docinfo.versions);
```

**`get_title()`** - Lấy title của document:

```javascript
const title = frm.get_title();
```

**`get_involved_users()`** - Lấy danh sách users liên quan:

```javascript
const users = frm.get_involved_users();
```

### 4.5.20. Tab Methods

**`set_active_tab(tab)`** - Set active tab:

```javascript
const tab = frm.layout.tabs[0];
frm.set_active_tab(tab);
```

**`get_active_tab()`** - Lấy active tab:

```javascript
const activeTab = frm.get_active_tab();
```

### 4.5.21. Child Table Advanced Methods

**`update_in_all_rows(table_fieldname, fieldname, value)`** - Update giá trị trong tất cả rows:

```javascript
// Update rate trong tất cả rows của items table
frm.update_in_all_rows("items", "rate", 100);
```

**`get_sum(table_fieldname, fieldname)`** - Tính tổng giá trị trong child table:

```javascript
const totalQty = frm.get_sum("items", "qty");
const totalAmount = frm.get_sum("items", "amount");
```

**`get_selected()`** - Lấy danh sách rows được chọn:

```javascript
const selected = frm.get_selected();
// Returns: { items: [[parentfield, name], ...] }
```

### 4.5.22. Field Advanced Methods

**`set_fields_as_options(fieldname, reference_doctype, filter_function, default_options, table_fieldname)`** - Set fields của doctype khác làm options:

```javascript
// Set fields của Customer làm options cho Select field
frm.set_fields_as_options(
    "field_to_populate",
    "Customer",
    (df) => df.fieldtype === "Data", // Filter function
    ["name", "customer_name"] // Default options
);
```

**`set_indicator_formatter(fieldname, get_color, get_text)`** - Set formatter cho indicator field:

```javascript
frm.set_indicator_formatter(
    "status",
    (doc) => doc.status === "Active" ? "green" : "red",
    (doc) => doc.status
);
```

**`field_map(fnames, fn)`** - Map function lên nhiều fields:

```javascript
// Set read_only cho nhiều fields
frm.field_map(["customer_name", "email_id"], function(field) {
    field.read_only = 1;
});
```

### 4.5.23. Make/Create Methods

**`can_create(doctype)`** - Kiểm tra có thể tạo document không:

```javascript
if (frm.can_create("Delivery Note")) {
    // Có thể tạo Delivery Note
}
```

**`make_new(doctype)`** - Tạo document mới từ form hiện tại:

```javascript
frm.make_new("Delivery Note"); // Tạo Delivery Note mới
```

**`set_link_field(doctype, new_doc)`** - Set link fields khi tạo document mới:

```javascript
// Internal method, thường được gọi tự động bởi make_new
```

### 4.5.24. UI Display Methods

**`set_intro(txt, color)`** - Set intro message:

```javascript
frm.set_intro("Please fill all required fields", "blue");
```

**`set_footnote(txt)`** - Set footnote:

```javascript
frm.set_footnote("Note: All fields are required");
```

**`show_success_action()`** - Hiển thị success action sau khi save:

```javascript
// Tự động được gọi sau khi save thành công
```

**`show_conflict_message()`** - Hiển thị message khi có conflict:

```javascript
// Tự động được gọi khi document bị modified
```

**`show_submit_message()`** - Hiển thị message để submit:

```javascript
// Tự động được gọi trong refresh
```

**`show_web_link()`** - Hiển thị web link nếu document có route:

```javascript
// Tự động được gọi trong refresh
```

**`add_web_link(path, label)`** - Thêm web link:

```javascript
frm.add_web_link("/my-page", "View on Website");
```

### 4.5.25. Grid/Table Methods

**`open_grid_row()`** - Mở grid row form:

```javascript
const gridForm = frm.open_grid_row();
```

### 4.5.26. Internal/Setup Methods

**`setup()`** - Setup form (internal):

```javascript
// Tự động được gọi khi form được tạo
```

**`setup_meta()`** - Setup metadata (internal):

```javascript
// Tự động được gọi trong constructor
```

**`setup_std_layout()`** - Setup standard layout (internal):

```javascript
// Tự động được gọi trong setup
```

**`watch_model_updates()`** - Watch model updates (internal):

```javascript
// Tự động được gọi trong setup
```

**`setup_notify_on_rename()`** - Setup rename notification (internal):

```javascript
// Tự động được gọi trong setup
```

**`setup_file_drop()`** - Setup file drop (internal):

```javascript
// Tự động được gọi trong setup
```

**`setup_doctype_actions()`** - Setup doctype actions (internal):

```javascript
// Tự động được gọi trong setup
```

**`setup_image_autocompletions_in_markdown()`** - Setup image autocompletions (internal):

```javascript
// Tự động được gọi trong onload_post_render
```

**`setup_docinfo_change_listener()`** - Setup docinfo change listener (internal):

```javascript
// Tự động được gọi trong switch_doc
```

**`trigger_onload(switched)`** - Trigger onload event (internal):

```javascript
// Tự động được gọi trong refresh
```

**`initialize_new_doc()`** - Initialize new document (internal):

```javascript
// Tự động được gọi trong trigger_onload
```

**`render_form(switched)`** - Render form (internal):

```javascript
// Tự động được gọi trong trigger_onload hoặc refresh
```

**`onload_post_render()`** - Post render after onload (internal):

```javascript
// Tự động được gọi sau khi render
```

**`refresh_header(switched)`** - Refresh header (internal):

```javascript
// Tự động được gọi trong render_form
```

**`refresh_fields()`** - Refresh all fields (internal):

```javascript
// Tự động được gọi trong render_form
```

**`cleanup_refresh()`** - Cleanup after refresh (internal):

```javascript
// Tự động được gọi trong refresh_fields
```

**`trigger_link_fields()`** - Trigger link fields (internal):

```javascript
// Tự động được gọi trong initialize_new_doc
```

**`check_reload()`** - Check if document needs reload (internal):

```javascript
// Tự động được gọi trong refresh
```

**`check_doctype_conflict(docname)`** - Check doctype conflict (internal):

```javascript
// Tự động được gọi trong refresh
```

**`rename_notify(dt, old, name)`** - Handle rename notification (internal):

```javascript
// Tự động được gọi khi document được rename
```

**`execute_action(action)`** - Execute doctype action (internal):

```javascript
// Tự động được gọi khi custom button được click
```

**`validate_form_action(action, resolve)`** - Validate form action (internal):

```javascript
// Tự động được gọi trước khi save/submit/cancel
```

**`handle_save_fail(btn, on_error)`** - Handle save failure (internal):

```javascript
// Tự động được gọi khi save fail
```

**`mark_mask_fields_readonly()`** - Mark masked fields as readonly (internal):

```javascript
// Tự động được gọi trong refresh
```

**`configure_breadcrumb_width()`** - Configure breadcrumb width (internal):

```javascript
// Tự động được gọi trong render_form
```

**`focus_on_first_input()`** - Focus on first input (internal):

```javascript
// Tự động được gọi trong render_form
```

**`run_after_load_hook()`** - Run after load hook (internal):

```javascript
// Tự động được gọi trong render_form
```

**`add_form_keyboard_shortcuts()`** - Add keyboard shortcuts (internal):

```javascript
// Tự động được gọi trong setup
```

**`show_submission_queue_banner()`** - Show submission queue banner (internal):

```javascript
// Tự động được gọi trong refresh_header
```

**`show_workflow_read_only_banner()`** - Show workflow read-only banner (internal):

```javascript
// Tự động được gọi trong refresh_header
```

**`_cancel_all(r, btn, callback, on_error)`** - Cancel all linked documents (internal):

```javascript
// Internal method cho cancel
```

**`_cancel(btn, callback, on_error, skip_confirm)`** - Cancel document (internal):

```javascript
// Internal method cho cancel
```

**`_discard(btn, on_error, skip_confirm)`** - Discard document (internal):

```javascript
// Internal method cho discard
```

**`validate_and_save(save_action, callback, btn, on_error, resolve, reject)`** - Validate and save (internal):

```javascript
// Internal method cho save
```

---

## 💾 5. Model và Data Management

### 5.1. `frappe.model`

`frappe.model` quản lý documents và data:

```javascript
frappe.model = {
    // Lấy document
    get_doc: function(doctype, name) { ... },
    
    // Set giá trị field
    set_value: function(doctype, docname, fieldname, value) { ... },
    
    // Kiểm tra permissions
    can_read: function(doctype) { ... },
    can_create: function(doctype) { ... },
    can_write: function(doctype) { ... },
    can_delete: function(doctype) { ... },
    
    // Events
    on: function(doctype, event, callback) { ... },
    trigger: function(doctype, event, fieldname, value, doc) { ... }
};
```

### 5.2. Lấy Document

```javascript
// Lấy document từ cache hoặc server
const doc = frappe.model.get_doc("Customer", "CUST-001");

// Hoặc từ form
const doc = frm.doc;
```

### 5.3. Set Value

```javascript
// Set giá trị field
frappe.model.set_value("Customer", "CUST-001", "customer_name", "New Name");

// Hoặc từ form
frm.set_value("customer_name", "New Name");
```

### 5.4. Permissions

```javascript
// Kiểm tra permissions
if (frappe.model.can_read("Customer")) {
    // User có quyền đọc
}

if (frappe.model.can_create("Customer")) {
    // User có quyền tạo
}
```

### 5.5. Model Events

```javascript
// Lắng nghe thay đổi document
frappe.model.on("Customer", "*", function(fieldname, value, doc) {
    console.log("Field changed:", fieldname, value);
});

// Lắng nghe thay đổi field cụ thể
frappe.model.on("Customer", "customer_name", function(fieldname, value, doc) {
    console.log("Customer name changed:", value);
});
```

---

## 🌐 6. API Calls

### 6.1. `frappe.call()`

Gọi API method từ server:

```javascript
frappe.call({
    method: "erpnext.selling.doctype.sales_order.sales_order.make_delivery_note",
    args: {
        source_name: "SO-001"
    },
    callback: function(r) {
        if (r.message) {
            // Xử lý response
            console.log("Success:", r.message);
        }
    }
});
```

### 6.2. `frappe.db.get_value()`

Lấy giá trị một field:

```javascript
frappe.db.get_value("Customer", "CUST-001", "customer_name", (r) => {
    if (r.message) {
        console.log("Customer name:", r.message.customer_name);
    }
});
```

### 6.3. `frappe.db.get_list()`

Lấy danh sách documents:

```javascript
frappe.db.get_list("Customer", {
    filters: { status: "Active" },
    fields: ["name", "customer_name", "email_id"],
    limit: 10
}, (r) => {
    if (r.message) {
        console.log("Customers:", r.message);
    }
});
```

### 6.4. `frappe.db.set_value()`

Set giá trị field:

```javascript
frappe.db.set_value("Customer", "CUST-001", "customer_name", "New Name", (r) => {
    if (r.message) {
        console.log("Updated successfully");
    }
});
```

---

## 🎨 7. UI Components

### 7.1. Dialog

```javascript
// Tạo dialog
let d = new frappe.ui.Dialog({
    title: "My Dialog",
    fields: [
        {
            label: "Name",
            fieldname: "name",
            fieldtype: "Data"
        },
        {
            label: "Email",
            fieldname: "email",
            fieldtype: "Data"
        }
    ],
    primary_action_label: "Save",
    primary_action: function(values) {
        console.log("Values:", values);
        d.hide();
    }
});

d.show();
```

### 7.2. Message

```javascript
// Hiển thị message
frappe.msgprint({
    title: "Success",
    indicator: "green",
    message: "Operation completed successfully"
});

// Hoặc đơn giản
frappe.msgprint("Hello World");
```

### 7.3. Confirm

```javascript
frappe.confirm("Are you sure?", function() {
    // User clicked Yes
    console.log("Confirmed");
}, function() {
    // User clicked No
    console.log("Cancelled");
});
```

### 7.4. Prompt

```javascript
frappe.prompt("Enter your name:", (value) => {
    console.log("Name:", value);
});
```

---

## 🔧 8. Utilities

### 8.1. `frappe.utils`

```javascript
// Format currency
frappe.utils.format_currency(1000, "USD"); // "$1,000.00"

// Format number
frappe.utils.format_number(1000); // "1,000"

// Format date
frappe.utils.format_date("2024-01-01"); // "01/01/2024"

// Debounce
const debouncedFn = frappe.utils.debounce(function() {
    console.log("Debounced");
}, 1000);

// Throttle
const throttledFn = frappe.utils.throttle(function() {
    console.log("Throttled");
}, 1000);
```

### 8.2. `frappe.translate`

```javascript
// Translate string
__("Hello World"); // Trả về bản dịch nếu có

// Format với arguments
__("Hello {0}", ["World"]); // "Hello World"
```

---

## 📋 9. Ví dụ Tổng hợp

### 9.1. Form Script đầy đủ

```javascript
// File: erpnext/selling/doctype/sales_order/sales_order.js

frappe.ui.form.on("Sales Order", {
    // Setup (chạy 1 lần)
    setup: function(frm) {
        // Setup custom buttons
        frm.add_custom_button(__("Create Delivery Note"), function() {
            frappe.model.open_mapped_doc({
                method: "erpnext.selling.doctype.sales_order.sales_order.make_delivery_note",
                frm: frm
            });
        });
    },
    
    // Onload (mỗi khi document được load)
    onload: function(frm) {
        // Set default values
        if (frm.is_new()) {
            frm.set_value("transaction_date", frappe.datetime.get_today());
        }
    },
    
    // Refresh (mỗi khi form được refresh)
    refresh: function(frm) {
        // Show/hide buttons based on status
        if (frm.doc.status === "Submitted") {
            frm.add_custom_button(__("Cancel"), function() {
                frm.cancel();
            });
        }
    },
    
    // Field change handlers
    customer: function(frm) {
        // Load customer details
        if (frm.doc.customer) {
            frappe.db.get_value("Customer", frm.doc.customer, 
                ["customer_name", "email_id"], (r) => {
                    if (r.message) {
                        frm.set_value("customer_name", r.message.customer_name);
                    }
                });
        }
    },
    
    "customer, transaction_date": function(frm) {
        // Validate khi customer hoặc transaction_date thay đổi
        if (frm.doc.customer && frm.doc.transaction_date) {
            // Do something
        }
    }
});
```

### 9.2. Custom Page

```javascript
// File: my_app/my_app/page/my_custom_page/my_custom_page.js

frappe.pages['my-custom-page'].on_page_load = function(wrapper) {
    let page = frappe.ui.make_app_page({
        parent: wrapper,
        title: "My Custom Page",
        single_column: true
    });
    
    // Add content
    $(frappe.render_template("my_custom_page", {})).appendTo(page.body);
    
    // Add button
    page.add_inner_button("Refresh", function() {
        load_data();
    });
    
    function load_data() {
        frappe.call({
            method: "my_app.api.get_data",
            callback: function(r) {
                if (r.message) {
                    // Render data
                    console.log("Data:", r.message);
                }
            }
        });
    }
    
    load_data();
};
```

### 9.3. List View Customization

```javascript
// File: erpnext/selling/doctype/customer/customer_list.js

frappe.listview_settings['Customer'] = {
    // Add custom button
    add_fields: ["status"],
    
    // Format row
    formatters: {
        status: function(value) {
            return value === "Active" 
                ? '<span class="badge badge-success">Active</span>'
                : '<span class="badge badge-danger">Inactive</span>';
        }
    },
    
    // Custom button
    get_indicator: function(doc) {
        return [__(doc.status), doc.status === "Active" ? "green" : "red", "status,=," + doc.status];
    },
    
    // Onload
    onload: function(listview) {
        // Add custom filter
        listview.page.add_inner_button("Active Customers", function() {
            listview.filter_area.add([[listview.doctype, "status", "=", "Active"]]);
        });
    }
};
```

---

## 🔄 10. Event System

### 10.1. Document Events

```javascript
// Lắng nghe events
$(document).on("form-refresh", function(e, frm) {
    console.log("Form refreshed:", frm.doctype);
});

$(document).on("route-change", function(e, route) {
    console.log("Route changed:", route);
});

$(document).on("app_ready", function() {
    console.log("App is ready");
});
```

### 10.2. Custom Events

```javascript
// Trigger custom event
$(document).trigger("my-custom-event", [data1, data2]);

// Lắng nghe custom event
$(document).on("my-custom-event", function(e, data1, data2) {
    console.log("Custom event:", data1, data2);
});
```

---

## 📚 11. Best Practices

### 11.1. Code Organization

- **Form scripts**: Đặt trong `[app]/[module]/doctype/[doctype]/[doctype].js`
- **List scripts**: Đặt trong `[app]/[module]/doctype/[doctype]/[doctype]_list.js`
- **Custom pages**: Đặt trong `[app]/[app]/page/[page_name]/[page_name].js`

### 11.2. Performance

- Sử dụng `debounce` cho các hàm được gọi nhiều lần
- Cache data khi có thể
- Tránh gọi API không cần thiết

### 11.3. Error Handling

```javascript
frappe.call({
    method: "my_app.api.get_data",
    callback: function(r) {
        if (r.exc) {
            frappe.msgprint({
                title: __("Error"),
                indicator: "red",
                message: r.exc
            });
            return;
        }
        // Process data
    }
});
```

---

## 🔗 12. Tài liệu tham khảo

- **File source**: `apps/frappe/frappe/public/js/frappe/`
- **Form**: `apps/frappe/frappe/public/js/frappe/form/`
- **Router**: `apps/frappe/frappe/public/js/frappe/router.js`
- **Model**: `apps/frappe/frappe/public/js/frappe/model/`
- **Boot**: `apps/frappe/frappe/boot.py`

---

## 📝 Tóm tắt

1. **Khởi động**: `frappe.Application` khởi tạo app, load `frappe.boot`
2. **Namespace**: Sử dụng `frappe.provide()` để tổ chức code
3. **Router**: `frappe.router` quản lý routing
4. **Form**: `frappe.ui.form.Form` quản lý form với lifecycle methods
5. **Model**: `frappe.model` quản lý documents và data
6. **API**: `frappe.call()`, `frappe.db.*` để gọi API
7. **UI**: `frappe.ui.Dialog`, `frappe.msgprint()`, etc.
8. **Events**: Sử dụng jQuery events để lắng nghe và trigger
