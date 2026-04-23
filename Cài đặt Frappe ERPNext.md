# Cài đặt ERPNext

## Cài đặt môi trường Frappe

Làm theo hướng dẫn **Step 1 → Step 12** tại:

- [Frappe ERPNext Version 15 trên Ubuntu 24.04 LTS]([https://github.com/TuanBuiDucVolsu/-Frappe-ERPNext-Version-15--in-Ubuntu-24.04-LTS])
- [Frappe Documentation]([https://docs.frappe.io/framework/user/en/introduction])
- [ERPNext Documentation]([https://docs.frappe.io/erpnext/getting-started-with-erpnext])

## Kiến thức Frappe & ERPNext

| Tài liệu | Mô tả |
|----------|--------|
| [FRAPPE_DOCTYPE_SETTINGS](https://github.com/TuanBuiDucVolsu/Frappe-Library/blob/main/FRAPPE_DOCTYPE_SETTINGS.md) | Tìm hiểu về Doctype trong Frappe (1 Doctype tương ứng với 1 table trong Database) |
| [FRAPPE_DOCFIELD_PROPERTIES](https://github.com/TuanBuiDucVolsu/Frappe-Library/blob/main/FRAPPE_DOCFIELD_PROPERTIES.md) | Tất cả các thuộc tính của Field trong DocType |
| [FRAPPE_DEPENDENCY_PROPERTIES_GUIDE](https://github.com/TuanBuiDucVolsu/Frappe-Library/blob/main/FRAPPE_DEPENDENCY_PROPERTIES_GUIDE.md) | Cách sử dụng Dependency Properties trong Frappe |
| [HOOKS_GUIDE](https://github.com/TuanBuiDucVolsu/Frappe-Library/blob/main/HOOKS_GUIDE.md) | Hooks trong Frappe/ERPNext |
| [FRAPPE_QUERY_METHOD](https://github.com/TuanBuiDucVolsu/Frappe-Library/blob/main/FRAPPE_QUERY_METHOD.md) | ORM cơ bản hay dùng trong Frappe |
| [FRAPPE_QUERY_BUILDER](https://github.com/TuanBuiDucVolsu/Frappe-Library/blob/main/FRAPPE_QUERY_BUILDER.md) | Frappe Query Builder |
| [FRAPPE_QUERY_BUILDER_FUNCTIONS](https://github.com/TuanBuiDucVolsu/Frappe-Library/blob/main/FRAPPE_QUERY_BUILDER_FUNCTIONS.md) | Các function chính của Query Builder |
| [FRAPPE_JS_GUIDE](https://github.com/TuanBuiDucVolsu/Frappe-Library/blob/main/FRAPPE_JS_GUIDE.md) | Frappe JS |

## Các phân hệ trong ERPNext

| Tài liệu | Mô tả |
|----------|--------|
| [SYSTEM_SETTINGS_GUIDE](https://github.com/TuanBuiDucVolsu/Frappe-Library/blob/main/SYSTEM_SETTINGS_GUIDE.md) | Giải thích các trường và chức năng trong **System Settings** của Frappe Framework |
| [STOCK_SETTINGS_GUIDE](https://github.com/TuanBuiDucVolsu/Frappe-Library/blob/main/STOCK_SETTINGS_GUIDE.md) | Giải thích các trường và chức năng trong **Stock Settings** của ERPNext |
| [STOCK_SETTINGS_GUIDE](https://github.com/TuanBuiDucVolsu/Frappe-Library/blob/main/STOCK.md) | Giải thích các trường và chức năng trong **Stock** của ERPNext |
| [BUYING_SETTINGS_GUIDE](https://github.com/TuanBuiDucVolsu/Frappe-Library/blob/main/BUYING_SETTINGS_GUIDE.md) | Giải thích các trường và chức năng trong **Buying Settings** của ERPNext |
| [BUYING_SETTINGS_GUIDE](https://github.com/TuanBuiDucVolsu/Frappe-Library/blob/main/BUYING.md) | Giải thích các trường và chức năng trong **Buying** của ERPNext |
| [SELLING_SETTINGS_GUIDE](https://github.com/TuanBuiDucVolsu/Frappe-Library/blob/main/SELLING_SETTINGS_GUIDE.md) | Giải thích các trường và chức năng trong **Selling Settings** của ERPNext |
| [SELLING_SETTINGS_GUIDE](https://github.com/TuanBuiDucVolsu/Frappe-Library/blob/main/SELLING.md) | Giải thích các trường và chức năng trong **Selling** của ERPNext |

## Khởi tạo bench để chạy ERPNext

### Khởi tạo dự án

Thay `"name"` bằng tên dự án:

```bash
bench init "name" --frappe-branch version-15
```

Ví dụ:

```bash
bench init mbw_rtg --frappe-branch version-15
```

### Tạo site mới (1 site sẽ tương ứng với 1 database)

- `--admin-password`: mật khẩu admin (tùy chọn)
- `--db-root-password`: mật khẩu MariaDB

```bash
bench new-site mbw.com --admin-password 123 --db-root-password 123 --set-default
```

(Có thể thay `mbw.com` bằng tên site bạn muốn.)

### Gán site mặc định cho bench

Sau khi gán, không cần `bench --site name_site` nữa — chỉ cần `bench` + lệnh:

```bash
bench use site_name
```

Ví dụ:

```bash
bench use mbw.com
```

### Bật developer mode

```bash
bench set-config -g developer_mode 1
```

## Cài module ERPNext (version-15)

```bash
bench get-app erpnext --branch version-15
```

## Tạo custom app

Custom app dùng để test và tùy biến; nên chỉnh trên app này, **không sửa core** Frappe và ERPNext.

```bash
bench new-app tên_app
```

Ví dụ:

```bash
bench new-app test_erp
```

## Cài app lên site

Chạy `bench start` trước khi dùng các lệnh cài app trong terminal.

```bash
bench install-app tên_app
```

Ví dụ cài ERPNext và app tùy chỉnh:

```bash
bench install-app erpnext
bench install-app test_erp
```

### Lệnh bổ trợ (khi cần)

```bash
bench --site all reinstall
bench update --reset
```
