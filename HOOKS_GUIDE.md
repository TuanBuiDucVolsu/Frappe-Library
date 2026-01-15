# Hướng dẫn Hooks trong Frappe/ERPNext

## Mục lục
1. [Tổng quan về Hooks](#tổng-quan-về-hooks)
2. [Cách tìm tất cả Hooks có sẵn](#cách-tìm-tất-cả-hooks-có-sẵn)
3. [Chi tiết các loại Hooks](#chi-tiết-các-loại-hooks)
4. [Ví dụ sử dụng](#ví-dụ-sử-dụng)
5. [Best Practices](#best-practices)

---

## Tổng quan về Hooks

### Hooks là gì?

**Hooks** là cơ chế cho phép các app trong Frappe/ERPNext:
- Đăng ký các function/callback để chạy tại các thời điểm cụ thể
- Tùy chỉnh hành vi của hệ thống mà không cần sửa code core
- Mở rộng chức năng của các app khác

### File hooks.py

Mỗi app trong Frappe có file `hooks.py` ở thư mục gốc của app:
```
apps/
  ├── frappe/
  │   └── hooks.py          # Hooks của Frappe Framework
  ├── erpnext/
  │   └── hooks.py          # Hooks của ERPNext
  └── your_app/
      └── hooks.py          # Hooks của app bạn
```

### Cách Frappe load Hooks

Frappe tự động:
1. Quét tất cả apps đã cài đặt
2. Load file `hooks.py` từ mỗi app
3. Merge tất cả hooks lại với nhau
4. Cache kết quả để tăng hiệu suất

```python
# Frappe tự động load hooks từ tất cả apps
hooks = frappe.get_hooks()  # Lấy tất cả hooks
hook_value = frappe.get_hooks("hook_name")  # Lấy hook cụ thể
```

---

## Cách tìm tất cả Hooks có sẵn

### 1. List tất cả Hooks bằng Python

**Trong Bench Console:**
```bash
bench --site your_site console
```

```python
import frappe

# Lấy tất cả hooks
all_hooks = frappe.get_hooks()

# In ra tất cả hook names
for hook_name in sorted(all_hooks.keys()):
    print(hook_name)

# Lưu vào file
with open("all_hooks.txt", "w") as f:
    for hook_name in sorted(all_hooks.keys()):
        f.write(f"{hook_name}\n")
```

**Tạo script riêng:**
```python
# list_hooks.py
import frappe
import json

def list_all_hooks():
    frappe.init(site="your_site")
    frappe.connect()
    
    hooks = frappe.get_hooks()
    
    print("=" * 80)
    print("ALL AVAILABLE HOOKS")
    print("=" * 80)
    
    for hook_name in sorted(hooks.keys()):
        hook_value = hooks[hook_name]
        print(f"\n{hook_name}:")
        if isinstance(hook_value, (list, tuple)):
            for item in hook_value:
                print(f"  - {item}")
        elif isinstance(hook_value, dict):
            for key, value in hook_value.items():
                print(f"  {key}: {value}")
        else:
            print(f"  {hook_value}")
    
    # Lưu JSON
    with open("all_hooks.json", "w") as f:
        json.dump(dict(hooks), f, indent=2, default=str)
    
    print(f"\n\nTotal hooks: {len(hooks)}")

if __name__ == "__main__":
    list_all_hooks()
```

Chạy:
```bash
bench --site your_site execute list_hooks.py
```

### 2. Xem file hooks.py của các Apps

```bash
# Xem hooks của Frappe (core hooks)
cat apps/frappe/frappe/hooks.py

# Xem hooks của ERPNext
cat apps/erpnext/erpnext/hooks.py

# Xem hooks của custom apps
cat apps/your_app/your_app/hooks.py
```

### 3. Tìm Hooks bằng grep

```bash
# Tìm tất cả nơi sử dụng hooks
grep -r "frappe.get_hooks" apps/frappe apps/erpnext

# Tìm hook cụ thể
grep -r "get_hooks.*hook_name" apps/

# Tìm trong Python files
grep -r "get_hooks" --include="*.py" apps/

# Tìm hooks được sử dụng (extract hook names)
grep -roh "frappe\.get_hooks(\"[^\"]*\")" apps/ | sort -u
```

### 4. Xem Documentation

```bash
cat apps/frappe/hooks.md
```

**Lưu ý:** File `hooks.md` không đầy đủ. Nhiều hooks chỉ được phát hiện qua:
- Xem code sử dụng hooks
- Xem hooks.py của các apps
- Thử nghiệm

---

## Chi tiết các loại Hooks

### 1. App Information Hooks

**Mục đích:** Định nghĩa thông tin cơ bản về app

```python
app_name = "your_app"                    # Tên app (slug)
app_title = "Your App"                   # Tên hiển thị
app_publisher = "Your Company"           # Nhà phát hành
app_description = "Description"          # Mô tả
app_email = "support@yourcompany.com"   # Email liên hệ
app_license = "MIT"                      # License
app_logo_url = "/assets/your_app/logo.svg"  # Logo URL
app_home = "/desk"                       # Route mặc định
develop_version = "15.x.x-develop"       # Version đang phát triển
```

**Ví dụ:**
```python
# frappe/hooks.py
app_name = "frappe"
app_title = "Frappe Framework"
app_publisher = "Frappe Technologies"
```

---

### 2. Installation Hooks

**Mục đích:** Chạy code khi cài đặt/gỡ cài đặt app

```python
before_install = "your_app.install.before_install"
after_install = "your_app.install.after_install"
before_uninstall = "your_app.install.before_uninstall"
after_uninstall = "your_app.install.after_uninstall"

# Chạy khi BẤT KỲ app nào được cài/gỡ
before_app_install = "your_app.install.before_any_app_install"
after_app_install = "your_app.install.after_any_app_install"
before_app_uninstall = "your_app.install.before_any_app_uninstall"
after_app_uninstall = "your_app.install.after_any_app_uninstall"

before_tests = "your_app.tests.before_tests"
```

**Ví dụ:**
```python
# erpnext/hooks.py
after_install = "erpnext.setup.install.after_install"
```

**Function signature:**
```python
def after_install():
    # Tạo master data, setup defaults, etc.
    pass
```

---

### 3. JavaScript/CSS Hooks

**Mục đích:** Load JS/CSS files vào các trang khác nhau

#### A. App-level (Desk UI)

```python
app_include_js = [
    "your_app.bundle.js",      # Bundle file
    "custom.js"                # Hoặc file riêng lẻ
]

app_include_css = [
    "your_app.bundle.css",
    "custom.css"
]

app_include_icons = [
    "/assets/your_app/icons/icons.svg"
]
```

#### B. Website-level

```python
web_include_js = ["website_script.js"]
web_include_css = ["website.css"]
web_include_icons = ["/assets/your_app/icons/icons.svg"]
```

#### C. DocType-specific

```python
doctype_js = {
    "Sales Order": "public/js/sales_order.js",
    "Purchase Order": "public/js/purchase_order.js"
}

doctype_list_js = {
    "Sales Order": ["public/js/sales_order_list.js"]
}

doctype_tree_js = {
    "Account": "public/js/account_tree.js"
}

doctype_calendar_js = {
    "Event": "public/js/event_calendar.js"
}
```

#### D. Page-specific

```python
page_js = {
    "setup-wizard": "public/js/setup_wizard.js",
    "print": "public/js/print.js"
}
```

#### E. Email CSS

```python
email_css = ["email.bundle.css"]
```

**Ví dụ:**
```python
# erpnext/hooks.py
app_include_js = "erpnext.bundle.js"
app_include_css = "erpnext.bundle.css"
doctype_js = {
    "Address": "public/js/address.js",
    "Communication": "public/js/communication.js"
}
```

---

### 4. Document Events (doc_events)

**Mục đích:** Hook vào các sự kiện của Document (DocType)

```python
doc_events = {
    # Hook cho TẤT CẢ doctypes
    "*": {
        "on_update": [
            "your_app.hooks.on_update_handler",
        ],
        "on_submit": [
            "your_app.hooks.on_submit_handler",
        ],
        "on_cancel": [
            "your_app.hooks.on_cancel_handler",
        ],
        "on_trash": [
            "your_app.hooks.on_trash_handler",
        ],
    },
    
    # Hook cho doctype cụ thể
    "Sales Order": {
        "before_insert": "your_app.hooks.so.before_insert",
        "after_insert": "your_app.hooks.so.after_insert",
        "before_validate": "your_app.hooks.so.before_validate",
        "validate": "your_app.hooks.so.validate",
        "before_save": "your_app.hooks.so.before_save",
        "on_update": "your_app.hooks.so.on_update",
        "before_submit": "your_app.hooks.so.before_submit",
        "on_submit": "your_app.hooks.so.on_submit",
        "on_cancel": "your_app.hooks.so.on_cancel",
        "on_update_after_submit": "your_app.hooks.so.on_update_after_submit",
        "on_change": "your_app.hooks.so.on_change",
        "after_rename": "your_app.hooks.so.after_rename",
        "on_trash": "your_app.hooks.so.on_trash",
        "after_delete": "your_app.hooks.so.after_delete",
    }
}
```

**Các events có sẵn:**
- `before_insert` - Trước khi insert document mới
- `after_insert` - Sau khi insert document mới
- `before_validate` - Trước khi validate
- `validate` - Khi validate (có thể throw exception)
- `before_save` - Trước khi save
- `on_update` - Khi document được update
- `before_submit` - Trước khi submit
- `on_submit` - Khi document được submit
- `on_cancel` - Khi document được cancel
- `on_update_after_submit` - Khi update document đã submit
- `on_change` - Khi field nào đó thay đổi
- `after_rename` - Sau khi rename document
- `on_trash` - Khi document bị xóa (trước khi xóa khỏi DB)
- `after_delete` - Sau khi document bị xóa khỏi DB

**Function signature:**
```python
def on_submit_handler(doc, method):
    # doc: Document object
    # method: Tên method đang được gọi
    pass
```

**Ví dụ:**
```python
# your_app/hooks.py
doc_events = {
    "Sales Order": {
        "on_submit": [
            "your_app.hooks.sales_order.send_notification",
            "your_app.hooks.sales_order.create_task",
        ],
        "before_save": "your_app.hooks.sales_order.validate_custom_field",
    }
}

# your_app/hooks/sales_order.py
def send_notification(doc, method):
    frappe.sendmail(
        recipients=["manager@company.com"],
        subject=f"Sales Order {doc.name} submitted",
        message=f"SO {doc.name} has been submitted"
    )

def validate_custom_field(doc, method):
    if doc.custom_field and not doc.custom_other_field:
        frappe.throw("Custom Other Field is required")
```

---

### 5. Scheduler Events

**Mục đích:** Định nghĩa background jobs chạy theo lịch

```python
scheduler_events = {
    # Cron jobs (cron expression)
    "cron": {
        "0/5 * * * *": [  # Mỗi 5 phút
            "your_app.tasks.every_5_minutes",
        ],
        "0/15 * * * *": [  # Mỗi 15 phút
            "your_app.tasks.every_15_minutes",
        ],
        "0 0 * * *": [  # Mỗi ngày lúc 00:00
            "your_app.tasks.daily_at_midnight",
        ],
    },
    
    # Chạy mỗi lần scheduler tick
    "all": [
        "your_app.tasks.on_every_tick",
    ],
    
    # Chạy hàng giờ
    "hourly": [
        "your_app.tasks.hourly_task",
    ],
    
    # Chạy hàng giờ (maintenance)
    "hourly_maintenance": [
        "your_app.tasks.hourly_cleanup",
    ],
    
    # Chạy hàng ngày
    "daily": [
        "your_app.tasks.daily_task",
    ],
    
    # Chạy hàng ngày (long running)
    "daily_long": [
        "your_app.tasks.daily_long_task",
    ],
    
    # Chạy hàng ngày (maintenance)
    "daily_maintenance": [
        "your_app.tasks.daily_cleanup",
    ],
    
    # Chạy hàng tuần (long running)
    "weekly_long": [
        "your_app.tasks.weekly_task",
    ],
    
    # Chạy hàng tháng
    "monthly": [
        "your_app.tasks.monthly_task",
    ],
}
```

**Function signature:**
```python
def hourly_task():
    # Chạy mỗi giờ
    frappe.logger().info("Hourly task running")
    pass
```

**Ví dụ:**
```python
# frappe/hooks.py
scheduler_events = {
    "cron": {
        "0/15 * * * *": [
            "frappe.email.doctype.email_account.email_account.pull",
        ],
    },
    "daily": [
        "frappe.desk.doctype.event.event.send_event_digest",
    ],
}
```

---

### 6. Permission Hooks

**Mục đích:** Tùy chỉnh permission logic

```python
# Thêm điều kiện WHERE vào query khi list/report
permission_query_conditions = {
    "Sales Order": "your_app.permissions.get_so_query_conditions",
    "Customer": "your_app.permissions.get_customer_query_conditions",
}

# Kiểm tra permission ở document level
has_permission = {
    "Sales Order": "your_app.permissions.has_so_permission",
    "Customer": "your_app.permissions.has_customer_permission",
}

# Permission cho website
has_website_permission = {
    "Address": "your_app.permissions.has_address_website_permission",
}
```

**Function signatures:**
```python
def get_so_query_conditions(user):
    """Trả về điều kiện WHERE để filter documents"""
    if "Sales Manager" in frappe.get_roles():
        return ""
    else:
        return f"`tabSales Order`.owner = '{user}'"

def has_so_permission(doc, user, permission_type="read"):
    """Kiểm tra user có permission trên document cụ thể không"""
    if permission_type == "read":
        # Custom logic
        return True
    return False
```

**Ví dụ:**
```python
# frappe/hooks.py
permission_query_conditions = {
    "Event": "frappe.desk.doctype.event.event.get_permission_query_conditions",
}

has_permission = {
    "Event": "frappe.desk.doctype.event.event.has_permission",
}
```

---

### 7. Request/Job Hooks

**Mục đích:** Hook vào HTTP request lifecycle và background jobs

```python
# HTTP Request hooks
before_request = [
    "your_app.middleware.before_request",
    "your_app.middleware.log_request",
]

after_request = [
    "your_app.middleware.after_request",
    "your_app.middleware.cleanup",
]

# Background Job hooks
before_job = [
    "your_app.jobs.before_job",
]

after_job = [
    "your_app.jobs.after_job",
    "your_app.jobs.cleanup",
]
```

**Function signatures:**
```python
def before_request():
    # Chạy trước mỗi HTTP request
    pass

def after_request(response):
    # Chạy sau mỗi HTTP request
    # response: Flask response object
    return response

def before_job():
    # Chạy trước background job
    pass

def after_job():
    # Chạy sau background job
    pass
```

**Ví dụ:**
```python
# frappe/hooks.py
before_request = [
    "frappe.recorder.record",
    "frappe.monitor.start",
    "frappe.rate_limiter.apply",
]
```

---

### 8. Session Hooks

**Mục đích:** Hook vào session lifecycle

```python
on_session_creation = [
    "your_app.session.on_session_created",
]

on_login = "your_app.session.on_user_login"
on_logout = "your_app.session.on_user_logout"
```

**Function signatures:**
```python
def on_session_created():
    # Khi session mới được tạo
    pass

def on_user_login():
    # Khi user login
    pass

def on_user_logout():
    # Khi user logout
    pass
```

**Ví dụ:**
```python
# frappe/hooks.py
on_session_creation = [
    "frappe.core.doctype.activity_log.feed.login_feed",
]

on_login = "frappe.desk.doctype.note.note._get_unseen_notes"
on_logout = "frappe.core.doctype.session_default_settings.session_default_settings.clear_session_defaults"
```

---

### 9. Boot Info Hooks

**Mục đích:** Thêm data vào `frappe.boot` khi user login

```python
extend_bootinfo = [
    "your_app.boot.add_custom_data",
    "your_app.boot.add_user_settings",
]
```

**Function signature:**
```python
def add_custom_data(bootinfo):
    """Thêm data vào bootinfo"""
    bootinfo.custom_data = {
        "setting1": "value1",
        "setting2": "value2",
    }
```

**Ví dụ:**
```python
# frappe/hooks.py
extend_bootinfo = [
    "frappe.utils.telemetry.add_bootinfo",
    "frappe.core.doctype.user_permission.user_permission.send_user_permissions",
]
```

---

### 10. Override Hooks

**Mục đích:** Override các class/method có sẵn

```python
# Override DocType controller class
extend_doctype_class = {
    "Address": "your_app.custom.address.CustomAddress",
}

# Override DocType controller class (thay thế hoàn toàn)
override_doctype_class = {
    "User": "your_app.custom.user.CustomUser",
}

# Override whitelisted methods
override_whitelisted_methods = {
    "frappe.utils.file_manager.download_file": "your_app.api.file.download_file",
}
```

**Ví dụ:**
```python
# erpnext/hooks.py
extend_doctype_class = {
    "Address": "erpnext.accounts.custom.address.ERPNextAddress",
}
```

---

### 11. Website Hooks

**Mục đích:** Tùy chỉnh website behavior

```python
# Routing rules
website_route_rules = [
    {"from_route": "/custom/<category>", "to_route": "Custom Page"},
]

# Redirects
website_redirects = [
    {"source": "/old", "target": "/new", "forward_query_parameters": True},
]

# Base template
base_template = "templates/base.html"

# Home page
home_page = "index"
website_user_home_page = "portal"
role_home_page = {
    "Customer": "portal",
    "Supplier": "portal",
}

# Clear cache
website_clear_cache = [
    "your_app.website.clear_cache",
]

# Portal menu
portal_menu_items = [
    "your_app.website.get_portal_menu",
]

# Sidebar
look_for_sidebar_json = [
    "your_app.website.get_sidebar",
]
```

---

### 12. PDF Hooks

**Mục đích:** Tùy chỉnh PDF generation

```python
pdf_header_html = "your_app.pdf.get_header_html"
pdf_body_html = "your_app.pdf.get_body_html"
pdf_footer_html = "your_app.pdf.get_footer_html"
pdf_generator = "your_app.pdf.generate_pdf"

get_print_format_template = "your_app.pdf.get_template"
```

**Function signatures:**
```python
def get_header_html(context):
    """Trả về HTML cho header"""
    return "<div>Header</div>"

def get_body_html(context):
    """Trả về HTML cho body"""
    return "<div>Body</div>"
```

---

### 13. Jinja Hooks

**Mục đích:** Thêm functions/filters vào Jinja templates

```python
jinja = {
    "methods": "your_app.jinja.get_globals",
    "filters": [
        "your_app.jinja.custom_filter",
    ],
}
```

**Function signatures:**
```python
def get_globals():
    """Trả về dict các global functions"""
    return {
        "custom_function": lambda x: x * 2,
    }

def custom_filter(value, arg):
    """Jinja filter"""
    return f"{value} - {arg}"
```

**Sử dụng trong template:**
```jinja
{{ value | custom_filter("test") }}
{{ custom_function(5) }}
```

---

### 14. Standard Queries

**Mục đích:** Định nghĩa query mặc định cho Link fields

```python
standard_queries = {
    "User": "your_app.queries.user_query",
    "Customer": "your_app.queries.customer_query",
}
```

**Function signature:**
```python
def user_query(doctype, txt, searchfield, start, page_len, filters):
    """Trả về list users"""
    return frappe.db.sql("""
        SELECT name, full_name
        FROM `tabUser`
        WHERE name LIKE %(txt)s
        LIMIT %(start)s, %(page_len)s
    """, {
        "txt": f"%{txt}%",
        "start": start,
        "page_len": page_len
    })
```

---

### 15. Other Important Hooks

```python
# Notification config
notification_config = "your_app.notifications.get_config"

# Desktop icons
get_desktop_icons = "your_app.desktop.get_icons"

# Setup wizard
setup_wizard_stages = "your_app.setup.get_stages"
setup_wizard_complete = "your_app.setup.complete"
setup_wizard_test = "your_app.setup.test"

# Boot session
boot_session = "your_app.boot.boot_session"

# Leaderboards
leaderboards = "your_app.leaderboard.get_leaderboards"

# Filters config
filters_config = "your_app.filters.get_config"

# Print settings
additional_print_settings = "your_app.print.get_settings"

# Sounds
sounds = [
    {"name": "custom", "src": "/assets/your_app/sounds/custom.mp3", "volume": 0.1},
]

# Migrate
before_migrate = ["your_app.migrate.before_migrate"]
after_migrate = ["your_app.migrate.after_migrate"]

# OTP methods
otp_methods = ["OTP App", "Email", "SMS", "Custom OTP"]

# User data fields (GDPR)
user_data_fields = [
    {"doctype": "Custom Doc", "strict": True},
]

# Global search
global_search_doctypes = {
    "Default": [
        {"doctype": "Custom Doc"},
    ]
}

# Ignore links on delete
ignore_links_on_delete = [
    "Custom Log",
]

# Log clearing
default_log_clearing_doctypes = {
    "Custom Log": 30,  # Giữ 30 ngày
}

# Persistent cache
persistent_cache_keys = [
    "custom-cache-*",
]

# Add to apps screen
add_to_apps_screen = [
    {
        "name": "your_app",
        "logo": "/assets/your_app/logo.svg",
        "title": "Your App",
        "route": "/desk",
    }
]
```

---

## Ví dụ sử dụng

### Ví dụ 1: Hook vào Sales Order on_submit

```python
# your_app/hooks.py
doc_events = {
    "Sales Order": {
        "on_submit": [
            "your_app.hooks.sales_order.send_email",
            "your_app.hooks.sales_order.create_task",
        ],
    }
}

# your_app/hooks/sales_order.py
import frappe

def send_email(doc, method):
    """Gửi email khi SO được submit"""
    frappe.sendmail(
        recipients=[doc.contact_email],
        subject=f"Sales Order {doc.name} Submitted",
        message=f"Your Sales Order {doc.name} has been submitted successfully."
    )

def create_task(doc, method):
    """Tạo task cho sales team"""
    task = frappe.get_doc({
        "doctype": "Task",
        "subject": f"Follow up on SO {doc.name}",
        "status": "Open",
        "priority": "High",
    })
    task.insert()
```

### Ví dụ 2: Scheduler Event

```python
# your_app/hooks.py
scheduler_events = {
    "daily": [
        "your_app.tasks.send_daily_report",
    ],
    "cron": {
        "0/30 * * * *": [  # Mỗi 30 phút
            "your_app.tasks.sync_data",
        ],
    },
}

# your_app/tasks.py
import frappe

def send_daily_report():
    """Gửi báo cáo hàng ngày"""
    # Logic gửi email báo cáo
    pass

def sync_data():
    """Đồng bộ dữ liệu mỗi 30 phút"""
    # Logic sync
    pass
```

### Ví dụ 3: Permission Hook

```python
# your_app/hooks.py
permission_query_conditions = {
    "Sales Order": "your_app.permissions.get_so_conditions",
}

# your_app/permissions.py
import frappe

def get_so_conditions(user):
    """Chỉ cho phép user xem SO của mình (trừ System Manager)"""
    if "System Manager" in frappe.get_roles(user):
        return ""
    else:
        return f"`tabSales Order`.owner = '{user}'"
```

### Ví dụ 4: Custom JavaScript

```python
# your_app/hooks.py
app_include_js = "your_app.bundle.js"
doctype_js = {
    "Sales Order": "public/js/sales_order.js",
}

# your_app/public/js/sales_order.js
frappe.ui.form.on("Sales Order", {
    refresh: function(frm) {
        // Custom logic
        if (frm.doc.status === "Draft") {
            frm.add_custom_button("Custom Action", function() {
                // Do something
            });
        }
    }
});
```

### Ví dụ 5: Override Class

```python
# your_app/hooks.py
extend_doctype_class = {
    "Sales Order": "your_app.custom.sales_order.CustomSalesOrder",
}

# your_app/custom/sales_order.py
import frappe
from erpnext.selling.doctype.sales_order.sales_order import SalesOrder

class CustomSalesOrder(SalesOrder):
    def validate(self):
        super().validate()  # Gọi validate của parent
        # Custom validation
        if self.custom_field and not self.custom_other_field:
            frappe.throw("Custom Other Field is required")
    
    def on_submit(self):
        super().on_submit()
        # Custom logic on submit
        pass
```

---

## Best Practices

### 1. Tổ chức code

```
your_app/
├── hooks.py                    # Định nghĩa hooks
├── hooks/
│   ├── __init__.py
│   ├── sales_order.py          # Hooks cho Sales Order
│   ├── purchase_order.py       # Hooks cho Purchase Order
│   └── permissions.py           # Permission hooks
├── tasks/
│   └── scheduler.py            # Scheduler tasks
└── custom/
    └── sales_order.py          # Override classes
```

### 2. Naming conventions

- **Hook functions:** `on_submit_handler`, `before_save_handler`
- **Hook files:** `sales_order.py`, `permissions.py`
- **Hook modules:** `your_app.hooks.sales_order`

### 3. Error handling

```python
def on_submit_handler(doc, method):
    try:
        # Your logic
        pass
    except Exception as e:
        frappe.log_error(f"Error in on_submit: {str(e)}")
        # Không throw exception để không block submit
```

### 4. Performance

- **Tránh** heavy operations trong `before_request`, `on_update`
- **Dùng** background jobs cho tasks nặng
- **Cache** data khi cần thiết

### 5. Testing

```python
# Test hook function
def test_on_submit_hook():
    so = frappe.get_doc({
        "doctype": "Sales Order",
        # ... fields
    }).insert()
    
    so.submit()
    
    # Verify hook was called
    assert frappe.db.exists("Task", {"subject": f"Follow up on SO {so.name}"})
```

### 6. Documentation

- **Comment** mục đích của mỗi hook
- **Document** function signatures
- **Giải thích** business logic

### 7. Debugging

```python
# Enable developer mode để reload hooks mỗi request
# frappe.conf.developer_mode = 1

# Clear cache sau khi sửa hooks
bench --site your_site clear-cache

# Check hooks được load
import frappe
hooks = frappe.get_hooks("your_hook_name")
print(hooks)
```

---

## Troubleshooting

### Hook không chạy?

1. **Clear cache:**
   ```bash
   bench --site your_site clear-cache
   ```

2. **Check syntax:**
   ```python
   # Đảm bảo không có syntax error trong hooks.py
   python -m py_compile your_app/hooks.py
   ```

3. **Check function path:**
   ```python
   # Đảm bảo function path đúng
   frappe.get_attr("your_app.hooks.sales_order.on_submit")  # Không lỗi
   ```

4. **Check hook được load:**
   ```python
   import frappe
   hooks = frappe.get_hooks("doc_events")
   print(hooks.get("Sales Order", {}))
   ```

### Hook chạy nhiều lần?

- Hooks được merge từ tất cả apps
- Nếu nhiều apps định nghĩa cùng hook → tất cả đều chạy
- Kiểm tra bằng cách list hooks:
  ```python
  hooks = frappe.get_hooks("doc_events")
  print(hooks["Sales Order"]["on_submit"])
  ```

---

## Tài liệu tham khảo

- **Frappe hooks.py:** `apps/frappe/frappe/hooks.py`
- **ERPNext hooks.py:** `apps/erpnext/erpnext/hooks.py`
- **Frappe hooks.md:** `apps/frappe/hooks.md`
- **Frappe Documentation:** https://frappeframework.com/docs

---

## Kết luận

Hooks là cơ chế mạnh mẽ để mở rộng Frappe/ERPNext mà không cần sửa code core. Hiểu rõ các loại hooks và cách sử dụng sẽ giúp bạn:

- Tùy chỉnh hành vi hệ thống
- Tích hợp với hệ thống bên ngoài
- Mở rộng chức năng của các app khác
- Tạo app mới với hooks riêng

**Lưu ý:** Luôn test hooks kỹ lưỡng và clear cache sau khi thay đổi!
