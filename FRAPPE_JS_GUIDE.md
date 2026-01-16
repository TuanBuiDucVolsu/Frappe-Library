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
