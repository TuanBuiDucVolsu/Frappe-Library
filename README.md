Tổng quan
File frappe/hooks.py định nghĩa các cấu hình và “điểm móc” (hooks) cho chính app frappe.
Các app khác (ERPNext, custom app) cũng có hooks.py với cấu trúc tương tự, nhưng frappe/hooks.py là “chuẩn gốc” để tham khảo.
Dưới đây là giải thích theo từng nhóm chức năng:
1. Thông tin ứng dụng

app_name = "frappe"
app_title = "Frappe Framework"
app_publisher = "Frappe Technologies"
app_description = "Full stack web framework with Python, Javascript, MariaDB, Redis, Node"
app_license = "MIT"
app_logo_url = "/assets/frappe/images/frappe-framework-logo.svg"
develop_version = "15.x.x-develop"
app_home = "/app/build"
app_email = "developers@frappe.io"

- app_name / app_title / app_description / app_publisher / app_email / app_license: metadata để hiển thị trong UI (App Installer, About, v.v.).
- app_logo_url: logo dùng cho app frappe.
- develop_version: nhánh phiên bản đang phát triển.
- app_home: route mặc định khi mở app này từ màn hình Apps.

2. Hooks cài đặt / gỡ cài đặt
before_install = "frappe.utils.install.before_install"
after_install = "frappe.utils.install.after_install"

after_app_install = "frappe.utils.install.auto_generate_icons_and_sidebar"
after_app_uninstall = "frappe.utils.install.delete_desktop_icon_and_sidebar"

- before_install / after_install: được gọi khi cài frappe lần đầu.
- after_app_install / after_app_uninstall: chạy khi bất kỳ app nào được cài/gỡ (Frappe dùng để auto tạo/xóa icons, sidebar,…).

3. Tích hợp JS/CSS cho Desk (backend UI)
page_js = {"setup-wizard": "public/js/frappe/setup_wizard.js"}

page_js = {"setup-wizard": "public/js/frappe/setup_wizard.js"}

app_include_js = [

    "libs.bundle.js",
    "desk.bundle.js",
    "list.bundle.js",
    "form.bundle.js",
    "controls.bundle.js",
    "report.bundle.js",
    "telemetry.bundle.js",
    "billing.bundle.js",
]

app_include_css = [

    "desk.bundle.css",
    "report.bundle.css",
]
app_include_icons = [

    "/assets/frappe/icons/lucide/icons.svg",
    "/assets/frappe/icons/timeless/icons.svg",
    "/assets/frappe/icons/espresso/icons.svg",
    "/assets/frappe/icons/desktop_icons/alphabets.svg",
]

- page_js: inject JS riêng cho 1 page cụ thể (setup-wizard).
- app_include_js / app_include_css: các bundle chính cho giao diện desk (Form, List, Report,...).
- app_include_icons: sprite SVG chứa các icon dùng trong giao diện.

4. JS/CSS cho Website
web_include_js = ["website_script.js"]
web_include_css = []
web_include_icons = [

    "/assets/frappe/icons/lucide/icons.svg",
    "/assets/frappe/icons/timeless/icons.svg",
    "/assets/frappe/icons/espresso/icons.svg",
]
email_css = ["email.bundle.css"]

- web_include_js / web_include_css: tài nguyên dùng trên website (frontend).
- web_include_icons: icon dùng trên website.
- email_css: CSS được chèn vào email HTML.

5. JS theo DocType / Page
doctype_js = {
    "Web Page": "public/js/frappe/utils/web_template.js",
    "Website Settings": "public/js/frappe/utils/web_template.js",
}

Khi mở form Web Page hoặc Website Settings → file JS tương ứng được inject.

6. Routing cho website
website_route_rules = [

    {"from_route": "/kb/<category>", "to_route": "Help Article"},
    {"from_route": "/profile", "to_route": "me"},
    {"from_route": "/desk/<path:app_path>", "to_route": "desk"},
]

website_redirects = [

    {"source": r"/app/(.*)", "target": r"/desk/\1", "forward_query_parameters": True},
    {"source": "/apps", "target": "/desk"},
    {"source": "/app", "target": "/desk"},
]

base_template = "templates/base.html"

- website_route_rules: map URL này sang DocType/route khác (kiểu URL routing).
- website_redirects: redirect từ URL cũ sang URL mới.
- base_template: Jinja template gốc cho mọi trang web.

7. Ghi file / Notification / Calendar
write_file_keys = ["file_url", "file_name"]

notification_config = "frappe.core.notifications.get_notification_config"

email_append_to = ["Event", "ToDo", "Communication"]
calendars = ["Event"]

- write_file_keys: các field name mà khi set sẽ trigger ghi file xuống disk.
- notification_config: function trả về cấu hình notification (được ERPNext & apps khác extend).
- email_append_to: các DocType có thể nhận email và tự tạo record.
- calendars: DocType nào xuất hiện trong calendar view.

8. Hooks login / session
on_session_creation = [

    "frappe.core.doctype.activity_log.feed.login_feed",
    "frappe.core.doctype.user.user.notify_admin_access_to_system_manager",
]

on_login = "frappe.desk.doctype.note.note._get_unseen_notes"
on_logout = "frappe.core.doctype.session_default_settings.session_default_settings.clear_session_defaults"

- on_session_creation: chạy khi session mới được tạo (login).
- on_login: chạy ngay sau khi login (vd show notes).
- on_logout: clear session defaults.

9. PDF hooks
pdf_header_html = "frappe.utils.pdf.pdf_header_html"
pdf_body_html = "frappe.utils.pdf.pdf_body_html"
pdf_footer_html = "frappe.utils.pdf.pdf_footer_html"
pdf_generator = "frappe.utils.pdf.get_chrome_pdf"

- Xác định hàm dùng để render HTML header/body/footer & hàm dùng để generate PDF (Chrome).

10. Permission hooks
permission_query_conditions = {
    "Event": "frappe.desk.doctype.event.event.get_permission_query_conditions",
    # ...
}

has_permission = {
    "Event": "frappe.desk.doctype.event.event.has_permission",
    # ...
}

has_website_permission = {"Address": "frappe.contacts.doctype.address.address.has_website_permission"}

- permission_query_conditions[Doctype]: thêm WHERE conditions khi query list/report cho Doctype đó.
- has_permission[Doctype]: function Python kiểm tra permission của user trên 1 record cụ thể.
- has_website_permission: tương tự, nhưng cho permission trên website.

11. Jinja hooks
jinja = {
    "methods": "frappe.utils.jinja_globals",
    "filters": [
    
        "frappe.utils.data.global_date_format",
        "frappe.utils.markdown",
        "frappe.website.utils.abs_url",
    ],
}

- methods: hàm trả về global functions có thể dùng trong template Jinja.
- filters: danh sách filter Jinja (vd {{ date | global_date_format }}).

12. Standard queries
standard_queries = {"User": "frappe.core.doctype.user.user.user_query"}

- Xác định query custom mặc định cho Link đến DocType User.

13. doc_events – hooks theo sự kiện của DocType
doc_events = {
    "*": {
        "on_update": [
    
            "frappe.desk.notifications.clear_doctype_notifications",
            "frappe.workflow.doctype.workflow_action.workflow_action.process_workflow_actions",
            # ...
        ],
        "after_rename": "frappe.desk.notifications.clear_doctype_notifications",
        "on_cancel": [
            "frappe.desk.notifications.clear_doctype_notifications",
            # ...
        ],
        "on_trash": [
            "frappe.desk.notifications.clear_doctype_notifications",
            # ...
        ],
        "on_update_after_submit": [
            "frappe.workflow.doctype.workflow_action.workflow_action.process_workflow_actions",
            # ...
        ],
        "on_change": [
            "frappe.automation.doctype.milestone_tracker.milestone_tracker.evaluate_milestone",
        ],
        "after_delete": ["frappe.core.doctype.permission_log.permission_log.make_perm_log"],
    },
    "Event": {
        "after_insert": "frappe.integrations.doctype.google_calendar.google_calendar.insert_event_in_google_calendar",
        # ...
    },
    # ...
}

- Hook tất cả DocType ("*"): khi on_update/on_cancel/on_trash/..., gọi các hàm tương ứng.
- Hook riêng cho Event, Contact, DocType, Page, v.v.
Các event phổ biến:
- before_insert, after_insert
- before_validate, validate, on_update
- before_submit, on_submit
- before_cancel, on_cancel
- on_trash, after_delete
- on_update_after_submit
- on_change, after_rename

14. Scheduler events
scheduler_events = {
    "cron": {
        "0/5 * * * *": [...],
        "0/15 * * * *": [...],
        # ...
    },
    "all": [
    
        "frappe.email.queue.flush",
        # ...
    ],
    "hourly": [],
    "hourly_maintenance": [...],
    "daily": [...],
    "daily_long": [],
    "daily_maintenance": [...],
    "weekly_long": [...],
    "monthly": [...],
}

Định nghĩa các background jobs theo tần suất.
- cron: dạng cron expression.
- all: chạy mỗi lần scheduler tick.
- hourly, daily, weekly_long, monthly, v.v.

15. Sounds
sounds = [
    {"name": "email", "src": "/assets/frappe/sounds/email.mp3", "volume": 0.1},
    {"name": "submit", "src": "/assets/frappe/sounds/submit.mp3", "volume": 0.1},
    # ...
]

- Đặt mapping tên → file âm thanh dùng trong UI (submit, cancel, delete, error,…).

16. Migrate hooks
before_migrate = ["frappe.core.doctype.patch_log.patch_log.before_migrate"]
after_migrate = [

    "frappe.website.doctype.website_theme.website_theme.after_migrate",
    "frappe.search.sqlite_search.build_index_in_background",
]

- Chạy trước/sau khi bench migrate.

17. User data privacy / GDPR
user_data_fields = [
    {"doctype": "Access Log", "strict": True},
    # ...
    {
        "doctype": "User",
        "filter_by": "name",
        "redact_fields": [...],
    },
    # ...
]

- Xác định dữ liệu nào là nhạy cảm, cách ẩn/xóa/redact khi xử lý yêu cầu xóa dữ liệu cá nhân (GDPR).

18. Global search doctypes
global_search_doctypes = {
    "Default": [
    
        {"doctype": "Contact"},
        {"doctype": "Address"},
        # ...
    ]
}

- Quy định DocType nào được index vào global search.

19. override_whitelisted_methods
override_whitelisted_methods = {
    "frappe.utils.file_manager.download_file": "download_file",
    # ...
}

- Cho phép override route/method whitelisted có sẵn bằng function khác (thường để giữ backward-compatibility).

20. ignore_links_on_delete
ignore_links_on_delete = [
    "Communication",
    "ToDo",
    "DocShare",
    # ...
]

- Khi xóa DocType khác, nếu có link từ các DocType trong list này → không chặn xóa.

21. Request / Job hooks
before_request = [

    "frappe.recorder.record",
    "frappe.monitor.start",
    "frappe.rate_limiter.apply",
    "frappe.integrations.oauth2.set_cors_for_privileged_requests",
]
after_request = [

    "frappe.monitor.stop",
]

before_job = [

    "frappe.recorder.record",
    "frappe.monitor.start",
]
after_job = [

    "frappe.recorder.dump",
    "frappe.monitor.stop",
    "frappe.utils.file_lock.release_document_locks",
]

Chạy trước/sau mỗi HTTP request hoặc background job:
 - Ghi log (recorder)
 - Theo dõi performance (monitor)
 - Áp dụng rate limit
 - Thiết lập context Sentry, v.v.

22. extend_bootinfo
extend_bootinfo = [

    "frappe.utils.telemetry.add_bootinfo",
    "frappe.core.doctype.user_permission.user_permission.send_user_permissions",
]

- Hàm được gọi để bổ sung thêm field vào frappe.boot khi user login / load desk.

23. Navbar & Help items
standard_navbar_items = [

    {"item_label": "User Settings", "item_type": "Action", "action": "frappe.ui.toolbar.route_to_user()", "is_standard": 1},
    {"item_label": "Log out", "item_type": "Action", "action": "frappe.app.logout()", "is_standard": 1},
]

standard_help_items = [

    {"item_label": "About", "item_type": "Action", "action": "frappe.ui.toolbar.show_about()", "is_standard": 1},
    # ...
]

- Định nghĩa các mục mặc định trong navbar & menu Help.

24. default_log_clearing_doctypes
default_log_clearing_doctypes = {
    "Error Log": 14,
    "Email Queue": 30,
    # ...
}

- Số ngày giữ log cho từng log doctype, dùng cho job dọn dẹp log.

25. persistent_cache_keys
persistent_cache_keys = [
    "changelog-*",
    "insert_queue_for_*",
    # ...
]

- Các cache key không bị xóa khi chạy frappe.clear_cache().

26. user_invitation
user_invitation = {
    "allowed_roles": {
        "System Manager": [],
    },
}

- Cấu hình ai có quyền gửi User Invitation.

27. add_to_apps_screen
add_to_apps_screen = [

    {
        "name": app_name,
        "logo": app_logo_url,
        "title": app_title,
        "route": app_home,
    }
]

- Xác định cách app frappe hiển thị trên màn hình Apps.
