# Tất cả các Settings trong DocType

File này liệt kê **tất cả các settings** có thể cấu hình cho một DocType trong Frappe v16.

---

## 📋 Tổng quan

DocType có rất nhiều settings được tổ chức thành các tab và section khác nhau. File này sẽ liệt kê tất cả các settings theo thứ tự xuất hiện trong UI.

---

## 🔷 Tab: Settings

### 1. Module Settings

#### 1.1. `module` (Link)
**Module** - Module chứa DocType này.

- **Bắt buộc:** Có
- **Options:** `Module Def`
- **Ví dụ:** `"Accounts"`, `"Selling"`, `"Buying"`
- **Lưu ý:** DocType phải thuộc về một module

---

### 2. Document Type Settings

#### 2.1. `is_submittable` (Check)
**Is Submittable** - Cho phép submit document.

- **Default:** `0` (false)
- **Description:** "Once submitted, submittable documents cannot be changed. They can only be Cancelled and Amended."
- **Depends on:** `!istable` (không áp dụng cho child table)
- **Lưu ý:** 
  - Khi enabled, document có thể được submit/cancel/amend
  - Cần có permissions cho submit, cancel, amend
  - Tự động thêm field `amended_from` nếu enabled

#### 2.2. `istable` (Check)
**Is Child Table** - DocType này là child table.

- **Default:** `0` (false)
- **Description:** "Child Tables are shown as a Grid in other DocTypes"
- **Lưu ý:**
  - Child table không thể submit
  - Child table không thể có child table khác
  - Child table hiển thị như grid trong parent document

#### 2.3. `issingle` (Check)
**Is Single** - DocType chỉ có một record duy nhất.

- **Default:** `0` (false)
- **Description:** "Single Types have only one record no tables associated. Values are stored in tabSingles"
- **Depends on:** `!istable` (không áp dụng cho child table)
- **Set only once:** `1` (không thể thay đổi sau khi save)
- **Lưu ý:**
  - Single DocType không có table trong database
  - Dữ liệu lưu trong `tabSingles`
  - Ví dụ: `System Settings`, `Global Defaults`

#### 2.4. `is_tree` (Check)
**Is Tree** - DocType có cấu trúc cây (tree structure).

- **Default:** `0` (false)
- **Description:** "Tree structures are implemented using Nested Set"
- **Depends on:** `!istable` (không áp dụng cho child table)
- **Lưu ý:**
  - Tự động thêm fields: `lft`, `rgt`, `is_group`
  - Cần set `nsm_parent_field` để chỉ định parent field
  - Ví dụ: `Account`, `Department`, `Territory`

#### 2.5. `is_calendar_and_gantt` (Check)
**Is Calendar and Gantt** - Cho phép Calendar và Gantt views.

- **Default:** `0` (false)
- **Description:** "Enables Calendar and Gantt views."
- **Depends on:** `!istable` (không áp dụng cho child table)
- **Lưu ý:** Cần có date fields để hiển thị trong Calendar/Gantt

#### 2.6. `editable_grid` (Check)
**Editable Grid** - Cho phép edit trực tiếp trong grid (cho child table).

- **Default:** `1` (true)
- **Depends on:** `istable` (chỉ áp dụng cho child table)
- **Lưu ý:** Khi enabled, có thể edit trực tiếp trong grid view

#### 2.7. `quick_entry` (Check)
**Quick Entry** - Cho phép tạo document nhanh qua dialog.

- **Default:** `0` (false)
- **Description:** "Open a dialog with mandatory fields to create a new record quickly. There must be at least one mandatory field to show in dialog."
- **Depends on:** `!istable` (không áp dụng cho child table)
- **Lưu ý:** Cần có ít nhất một mandatory field

#### 2.8. `grid_page_length` (Int)
**Grid Page Length** - Số rows hiển thị mỗi trang trong grid (cho child table).

- **Default:** `50`
- **Depends on:** `istable` (chỉ áp dụng cho child table)
- **Non negative:** `1` (phải >= 0)

#### 2.9. `rows_threshold_for_grid_search` (Int)
**Rows Threshold for Grid Search** - Số rows tối thiểu để hiển thị search trong grid.

- **Default:** `20`
- **Depends on:** `istable` (chỉ áp dụng cho child table)
- **Non negative:** `1` (phải >= 0)

---

### 3. Tracking Settings

#### 3.1. `track_changes` (Check)
**Track Changes** - Theo dõi thay đổi trong document.

- **Default:** `0` (false)
- **Description:** "If enabled, changes to the document are tracked and shown in timeline"
- **Depends on:** `!istable` (không áp dụng cho child table)
- **Lưu ý:** Thay đổi sẽ hiển thị trong timeline

#### 3.2. `track_seen` (Check)
**Track Seen** - Đánh dấu document đã được xem.

- **Default:** `0` (false)
- **Description:** "If enabled, the document is marked as seen, the first time a user opens it"
- **Depends on:** `!istable` (không áp dụng cho child table)
- **Lưu ý:** Chỉ đánh dấu lần đầu mở

#### 3.3. `track_views` (Check)
**Track Views** - Theo dõi số lần xem document.

- **Default:** `0` (false)
- **Description:** "If enabled, document views are tracked, this can happen multiple times"
- **Depends on:** `!istable` (không áp dụng cho child table)
- **Lưu ý:** Có thể track nhiều lần (khác với track_seen)

---

### 4. Other Settings

#### 4.1. `custom` (Check)
**Custom?** - DocType này là custom DocType.

- **Default:** `0` (false)
- **Lưu ý:** Custom DocType được tạo bởi user, không phải core

#### 4.2. `beta` (Check)
**Beta** - Đánh dấu DocType đang trong giai đoạn beta.

- **Default:** `0` (false)
- **Depends on:** `!istable` (không áp dụng cho child table)
- **Lưu ý:** Hiển thị warning cho user

#### 4.3. `is_virtual` (Check)
**Is Virtual** - DocType không lưu trong database.

- **Default:** `0` (false)
- **Lưu ý:** Virtual DocType không có table, dữ liệu được tính toán động

#### 4.4. `queue_in_background` (Check)
**Queue in Background (BETA)** - Submit document trong background.

- **Default:** `0` (false)
- **Description:** "Enabling this will submit documents in background"
- **Depends on:** `doc.is_submittable` (chỉ áp dụng cho submittable doctypes)
- **Lưu ý:** Document sẽ được submit bất đồng bộ

#### 4.5. `description` (Small Text)
**Description** - Mô tả về DocType.

- **Bắt buộc:** Không
- **Ví dụ:** "Sales Invoice is used to bill customers"

---

### 5. Form Settings

#### 5.1. `image_field` (Data)
**Image Field** - Tên field chứa image (phải là Attach Image type).

- **Bắt buộc:** Không
- **Description:** "Must be of type 'Attach Image'"
- **Lưu ý:** Image sẽ hiển thị trong form header

#### 5.2. `timeline_field` (Data)
**Timeline Field** - Tên field Link để associate comments/communications.

- **Bắt buộc:** Không
- **Description:** "Comments and Communications will be associated with this linked document"
- **Depends on:** `!istable` (không áp dụng cho child table)
- **Ví dụ:** `"customer"`, `"supplier"`, `"employee"`

#### 5.3. `nsm_parent_field` (Data)
**Parent Field (Tree)** - Tên field chứa parent (cho tree structure).

- **Bắt buộc:** Không (nhưng cần thiết nếu `is_tree = 1`)
- **Depends on:** `is_tree` (chỉ hiển thị khi is_tree = 1)
- **Ví dụ:** `"parent_account"`, `"parent_department"`

#### 5.4. `max_attachments` (Int)
**Max Attachments** - Số lượng attachments tối đa.

- **Bắt buộc:** Không
- **Lưu ý:** Giới hạn số file có thể attach

#### 5.5. `hide_toolbar` (Check)
**Hide Sidebar, Menu, and Comments** - Ẩn sidebar, menu và comments.

- **Default:** `0` (false)
- **Lưu ý:** Ẩn toàn bộ sidebar bên phải

#### 5.6. `allow_copy` (Check)
**Hide Copy** - Ẩn nút Copy.

- **Default:** `0` (false)
- **Lưu ý:** Khi enabled, ẩn nút Copy (tên field hơi confusing)

#### 5.7. `allow_rename` (Check)
**Allow Rename** - Cho phép rename document.

- **Default:** `1` (true)
- **Lưu ý:** Khi disabled, không thể rename document

#### 5.8. `allow_import` (Check)
**Allow Import (via Data Import Tool)** - Cho phép import qua Data Import Tool.

- **Default:** `0` (false)
- **Depends on:** `!issingle` (không áp dụng cho single doctype)

#### 5.9. `allow_events_in_timeline` (Check)
**Allow events in timeline** - Cho phép events trong timeline.

- **Default:** `0` (false)
- **Lưu ý:** Cho phép hiển thị events trong timeline

#### 5.10. `allow_auto_repeat` (Check)
**Allow Auto Repeat** - Cho phép Auto Repeat.

- **Default:** `0` (false)
- **Lưu ý:** Cho phép tự động tạo document lặp lại

#### 5.11. `make_attachments_public` (Check)
**Make Attachments Public by Default** - Attachments mặc định là public.

- **Default:** `0` (false)
- **Lưu ý:** Tất cả attachments sẽ là public khi upload

---

### 6. View Settings

#### 6.1. `title_field` (Data)
**Title Field** - Tên field dùng làm title khi hiển thị trong link.

- **Bắt buộc:** Không (nhưng required nếu `show_title_field_in_link = 1`)
- **Depends on:** `!istable` (không áp dụng cho child table)
- **Mandatory depends on:** `doc.show_title_field_in_link`
- **Ví dụ:** `"customer_name"`, `"item_name"`

#### 6.2. `show_title_field_in_link` (Check)
**Show Title in Link Fields** - Hiển thị title field trong link fields.

- **Default:** `0` (false)
- **Lưu ý:** Khi enabled, cần set `title_field`

#### 6.3. `translated_doctype` (Check)
**Translate Link Fields** - Dịch link fields.

- **Default:** `0` (false)
- **Lưu ý:** Link fields sẽ được dịch

#### 6.4. `search_fields` (Data)
**Search Fields** - Danh sách fields dùng để search (comma-separated).

- **Bắt buộc:** Không
- **Depends on:** `!istable` (không áp dụng cho child table)
- **Ví dụ:** `"customer_name,email_id,phone"`

#### 6.5. `default_print_format` (Data)
**Default Print Format** - Print format mặc định.

- **Bắt buộc:** Không
- **Ví dụ:** `"Standard"`, `"Invoice"`

#### 6.6. `sort_field` (Data)
**Default Sort Field** - Field mặc định để sort trong List View.

- **Default:** `"creation"`
- **Depends on:** `!istable` (không áp dụng cho child table)
- **Ví dụ:** `"creation"`, `"modified"`, `"name"`

#### 6.7. `sort_order` (Select)
**Default Sort Order** - Thứ tự sort mặc định.

- **Default:** `"DESC"`
- **Depends on:** `!istable` (không áp dụng cho child table)
- **Options:** `"ASC"`, `"DESC"`

#### 6.8. `default_view` (Select)
**Default View** - View mặc định khi mở List View.

- **Bắt buộc:** Không
- **Lưu ý:** Có thể là List, Calendar, Gantt, Kanban, etc.

#### 6.9. `force_re_route_to_default_view` (Check)
**Force Re-route to Default View** - Bắt buộc chuyển về default view.

- **Default:** `0` (false)
- **Lưu ý:** Luôn redirect về default view

---

### 7. Display Settings

#### 7.1. `document_type` (Select)
**Show in Module Section** - Hiển thị trong section nào của module.

- **Bắt buộc:** Không
- **Options:** `""`, `"Document"`, `"Setup"`, `"System"`, `"Other"`
- **Lưu ý:** Ảnh hưởng đến cách hiển thị trong module

#### 7.2. `icon` (Data)
**Icon** - Icon cho DocType.

- **Bắt buộc:** Không
- **Ví dụ:** `"fa fa-file"`, `"octicon octicon-file"`

#### 7.3. `color` (Data)
**Color** - Màu cho DocType.

- **Bắt buộc:** Không
- **Ví dụ:** `"#FF5733"`, `"blue"`

#### 7.4. `show_preview_popup` (Check)
**Show Preview Popup** - Hiển thị preview popup khi click vào document.

- **Default:** `0` (false)
- **Lưu ý:** Hiển thị popup preview thay vì mở form

#### 7.5. `show_name_in_global_search` (Check)
**Make "name" searchable in Global Search** - Cho phép search "name" trong Global Search.

- **Default:** `0` (false)
- **Lưu ý:** Field "name" sẽ được index cho Global Search

---

### 8. Email Settings

#### 8.1. `email_append_to` (Check)
**Allow document creation via Email** - Cho phép tạo document qua email.

- **Default:** `0` (false)
- **Lưu ý:** Khi enabled, có thể tạo document bằng cách reply email

#### 8.2. `default_email_template` (Link)
**Default Email Template** - Email template mặc định.

- **Bắt buộc:** Không
- **Options:** `Email Template`
- **Lưu ý:** Template dùng khi gửi email

#### 8.3. `sender_field` (Data)
**Sender Email Field** - Tên field chứa sender email.

- **Bắt buộc:** Không (nhưng required nếu `email_append_to = 1`)
- **Depends on:** `email_append_to`
- **Mandatory depends on:** `email_append_to`
- **Ví dụ:** `"sender_email"`

#### 8.4. `sender_name_field` (Data)
**Sender Name Field** - Tên field chứa sender name.

- **Bắt buộc:** Không
- **Depends on:** `email_append_to`
- **Ví dụ:** `"sender_name"`

#### 8.5. `recipient_account_field` (Data)
**Recipient Account Field** - Tên field chứa recipient account.

- **Bắt buộc:** Không
- **Depends on:** `email_append_to`
- **Ví dụ:** `"recipient_account"`

#### 8.6. `subject_field` (Data)
**Subject Field** - Tên field chứa email subject.

- **Bắt buộc:** Không
- **Depends on:** `email_append_to`
- **Ví dụ:** `"subject"`

---

## 🔷 Tab: Naming

### 1. Naming Settings

#### 1.1. `naming_rule` (Select)
**Naming Rule** - Quy tắc đặt tên cho document.

- **Bắt buộc:** Không
- **Options:**
  - `""` - Empty
  - `"Set by user"` - User tự đặt tên
  - `"Autoincrement"` - Tự động tăng số
  - `"By fieldname"` - Theo tên field
  - `"By \"Naming Series\" field"` - Theo Naming Series field
  - `"Expression"` - Theo expression
  - `"Expression (old style)"` - Expression kiểu cũ
  - `"Random"` - Random
  - `"UUID"` - UUID
  - `"By script"` - Theo script

#### 1.2. `autoname` (Data)
**Auto Name** - Expression hoặc format cho auto naming.

- **Bắt buộc:** Không (nhưng cần thiết nếu `naming_rule` không phải "Set by user")
- **Ví dụ:**
  - `"naming_series"` - Cho Naming Series
  - `"PROJ-.#####"` - Format với số
  - `"format:{customer}-{date}"` - Expression
  - `"hash"` - Hash

---

## 🔷 Tab: Permissions

### 1. Permission Settings

#### 1.1. `permissions` (Table)
**Permissions** - Danh sách permissions (DocPerm).

- **Options:** `DocPerm`
- **Lưu ý:** Định nghĩa quyền truy cập cho các roles

#### 1.2. `restrict_to_domain` (Link)
**Restrict To Domain** - Giới hạn DocType cho domain cụ thể.

- **Bắt buộc:** Không
- **Options:** `Domain`
- **Lưu ý:** Chỉ domain được chỉ định mới thấy DocType này

#### 1.3. `read_only` (Check)
**User Cannot Search** - User không thể search.

- **Default:** `0` (false)
- **Lưu ý:** Khi enabled, user không thể search document này

#### 1.4. `in_create` (Check)
**User Cannot Create** - User không thể tạo.

- **Default:** `0` (false)
- **Lưu ý:** Khi enabled, user không thể tạo document mới

#### 1.5. `protect_attached_files` (Check)
**Protect Attached Files** - Bảo vệ attached files.

- **Default:** `0` (false)
- **Description:** "Users are only able to delete attached files if the document is either in draft or if the document is canceled and they are also able to delete the document."
- **Lưu ý:** Chỉ cho phép xóa file khi document ở draft hoặc cancelled

---

## 🔷 Tab: Fields

### 1. Fields

#### 1.1. `fields` (Table)
**Fields** - Danh sách fields trong DocType.

- **Options:** `DocField`
- **Lưu ý:** Định nghĩa tất cả fields của DocType

---

## 🔷 Tab: Actions

### 1. Document Actions

#### 1.1. `actions` (Table)
**Document Actions** - Danh sách actions có thể thực hiện trên document.

- **Options:** `DocType Action`
- **Lưu ý:** Định nghĩa các action buttons và workflows

---

## 🔷 Tab: Links

### 1. Document Links

#### 1.1. `links` (Table)
**Document Links** - Danh sách links đến DocTypes khác.

- **Options:** `DocType Link`
- **Lưu ý:** Định nghĩa các links hiển thị trong sidebar

---

## 🔷 Tab: States

### 1. Document States

#### 1.1. `states` (Table)
**Document States** - Danh sách states của document.

- **Options:** `DocType State`
- **Lưu ý:** Định nghĩa các states và màu sắc tương ứng

---

## 🔷 Tab: Web View

### 1. Web View Settings

#### 1.1. `has_web_view` (Check)
**Has Web View** - DocType có web view.

- **Default:** `0` (false)
- **Depends on:** `doc.custom === 0 && !doc.istable` (chỉ áp dụng cho non-custom, non-child table)

#### 1.2. `allow_guest_to_view` (Check)
**Allow Guest to View** - Cho phép guest xem web view.

- **Default:** `0` (false)
- **Depends on:** `has_web_view` (chỉ hiển thị khi has_web_view = 1)

#### 1.3. `index_web_pages_for_search` (Check)
**Index Web Pages for Search** - Index web pages cho search.

- **Default:** `1` (true)
- **Lưu ý:** Web pages sẽ được index cho search engine

#### 1.4. `route` (Data)
**Route** - Route cho web view.

- **Bắt buộc:** Không
- **Depends on:** `!istable` (không áp dụng cho child table)
- **Ví dụ:** `"blog"`, `"news"`

#### 1.5. `is_published_field` (Data)
**Is Published Field** - Tên field xác định document có được publish không.

- **Bắt buộc:** Không
- **Depends on:** `has_web_view`
- **Ví dụ:** `"published"`, `"is_published"`

#### 1.6. `website_search_field` (Data)
**Website Search Field** - Tên field dùng để search trên website.

- **Bắt buộc:** Không
- **Depends on:** `has_web_view`
- **Ví dụ:** `"title"`, `"content"`

---

## 🔷 Advanced Settings (Hidden)

### 1. Database Settings

#### 1.1. `engine` (Select)
**Database Engine** - Database engine cho table.

- **Default:** `"InnoDB"`
- **Depends on:** `!issingle` (không áp dụng cho single doctype)
- **Options:** `"InnoDB"`, `"MyISAM"`
- **Hidden:** `1` (ẩn trong UI, chỉ hiển thị khi enable advanced)

#### 1.2. `row_format` (Select)
**Row Format** - Row format cho table.

- **Default:** `"Dynamic"`
- **Hidden:** `1` (ẩn trong UI)
- **Options:** `"Dynamic"`, `"Compressed"`

#### 1.3. `migration_hash` (Data)
**Migration Hash** - Hash cho migration.

- **Hidden:** `1` (ẩn trong UI)
- **Lưu ý:** Tự động set bởi Frappe

---

## 🔷 Tab: Connections

### 1. Connections

Tab này hiển thị các connections liên quan đến DocType (Workflows, Reports, Print Formats, etc.).

- **Show Dashboard:** `1` (hiển thị dashboard)

---

## 📝 Tóm tắt theo Nhóm

### Nhóm Module & Type:
- `module`, `is_submittable`, `istable`, `issingle`, `is_tree`, `is_calendar_and_gantt`, `is_virtual`

### Nhóm Grid & Entry:
- `editable_grid`, `quick_entry`, `grid_page_length`, `rows_threshold_for_grid_search`

### Nhóm Tracking:
- `track_changes`, `track_seen`, `track_views`

### Nhóm Form Display:
- `image_field`, `timeline_field`, `nsm_parent_field`, `max_attachments`, `hide_toolbar`, `allow_copy`, `allow_rename`

### Nhóm Import & Export:
- `allow_import`, `allow_auto_repeat`, `allow_events_in_timeline`

### Nhóm View Settings:
- `title_field`, `show_title_field_in_link`, `translated_doctype`, `search_fields`, `default_print_format`, `sort_field`, `sort_order`, `default_view`, `force_re_route_to_default_view`

### Nhóm Display:
- `document_type`, `icon`, `color`, `show_preview_popup`, `show_name_in_global_search`

### Nhóm Email:
- `email_append_to`, `default_email_template`, `sender_field`, `sender_name_field`, `recipient_account_field`, `subject_field`

### Nhóm Naming:
- `naming_rule`, `autoname`

### Nhóm Permissions:
- `permissions`, `restrict_to_domain`, `read_only`, `in_create`, `protect_attached_files`

### Nhóm Web View:
- `has_web_view`, `allow_guest_to_view`, `index_web_pages_for_search`, `route`, `is_published_field`, `website_search_field`

### Nhóm Advanced:
- `engine`, `row_format`, `migration_hash`

### Nhóm Other:
- `custom`, `beta`, `queue_in_background`, `description`, `documentation`, `make_attachments_public`

---

## 🔗 Tài liệu tham khảo

- **File source:** `apps/frappe/frappe/core/doctype/doctype/doctype.json`
- **Python class:** `apps/frappe/frappe/core/doctype/doctype/doctype.py`
- **Frappe Documentation:** [DocType Settings](https://frappeframework.com/docs/user/en/desk/customize/customize-doctype)
