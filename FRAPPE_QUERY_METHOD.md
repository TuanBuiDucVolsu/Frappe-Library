# Tất Cả Các Cách Query Lấy Dữ Liệu Trong Frappe Version 16

Tài liệu này tổng hợp tất cả các phương thức để query và lấy dữ liệu trong Frappe Framework version 16.

---

## Mục Lục

1. [frappe.db Methods (Low-level Database Methods)](#1-frappedb-methods-low-level-database-methods)
2. [frappe.get_* Methods (High-level Methods)](#2-frappeget_-methods-high-level-methods)
3. [Query Builder (frappe.qb) - Modern Approach](#3-query-builder-frappeqb---modern-approach)
4. [Raw SQL (frappe.db.sql)](#4-raw-sql-frappedbsql)
5. [Document Methods (frappe.get_doc)](#5-document-methods-frappeget_doc)
6. [So Sánh và Lựa Chọn Phương Thức](#6-so-sánh-và-lựa-chọn-phương-thức)

---

## 1. frappe.db Methods (Low-level Database Methods)

### 1.1. `frappe.db.get_value()`

Lấy một giá trị hoặc nhiều giá trị từ một document.

**Cú pháp:**
```python
frappe.db.get_value(
    doctype: str,
    filters: dict | str | None = None,
    fieldname: str | list[str] = "name",
    ignore: bool = False,
    as_dict: bool = False,
    debug: bool = False,
    order_by: str = DefaultOrderBy,
    cache: bool = False,
    for_update: bool = False,
    pluck: bool = False,
    distinct: bool = False,
    skip_locked: bool = False,
    wait: bool = True,
)
```

**Ví dụ:**
```python
# Lấy một giá trị đơn
last_login = frappe.db.get_value("User", "test@example.com", "last_login")

# Lấy nhiều giá trị
last_login, last_ip = frappe.db.get_value(
    "User", 
    "test@example.com",
    ["last_login", "last_ip"]
)

# Lấy giá trị với filter
customer_name = frappe.db.get_value(
    "Customer",
    {"name": ("like", "A%")},
    "customer_name"
)

# Lấy giá trị từ Single DocType
date_format = frappe.db.get_value("System Settings", None, "date_format")

# Lấy giá trị dạng dict
user_data = frappe.db.get_value(
    "User",
    "test@example.com",
    ["name", "email", "full_name"],
    as_dict=True
)

# Lấy giá trị với for_update (lock row)
user = frappe.db.get_value(
    "User",
    "test@example.com",
    "name",
    for_update=True
)
```

**Lưu ý:**
- Trả về `None` nếu không tìm thấy
- Nếu chỉ lấy 1 field, trả về giá trị trực tiếp (không phải list)
- Nếu lấy nhiều field, trả về tuple hoặc dict (tùy `as_dict`)

---

### 1.2. `frappe.db.get_values()`

Lấy nhiều giá trị từ nhiều documents (tương tự `get_value` nhưng có thể lấy nhiều rows).

**Cú pháp:**
```python
frappe.db.get_values(
    doctype: str,
    filters: dict | list | None = None,
    fieldname: str | list[str] = "name",
    ignore: bool = False,
    as_dict: bool = False,
    debug: bool = False,
    order_by: str = DefaultOrderBy,
    update: dict | None = None,
    cache: bool = False,
    for_update: bool = False,
    run: bool = True,
    pluck: bool = False,
    distinct: bool = False,
    limit: int | None = None,
    skip_locked: bool = False,
    wait: bool = True,
)
```

**Ví dụ:**
```python
# Lấy danh sách tên user
user_names = frappe.db.get_values(
    "User",
    {"enabled": 1},
    "name",
    pluck=True  # Trả về list các giá trị thay vì list các tuple
)

# Lấy nhiều fields từ nhiều documents
users = frappe.db.get_values(
    "User",
    {"enabled": 1},
    ["name", "email", "full_name"],
    as_dict=True
)

# Lấy với limit
recent_users = frappe.db.get_values(
    "User",
    {},
    ["name", "creation"],
    order_by="creation desc",
    limit=10
)
```

---

### 1.3. `frappe.db.get_single_value()`

Lấy giá trị từ Single DocType (DocType chỉ có 1 record duy nhất).

**Cú pháp:**
```python
frappe.db.get_single_value(
    doctype: str,
    fieldname: str,
    cache: bool = True,
    debug: bool = False,
    for_update: bool = False,
    run: bool = True,
)
```

**Ví dụ:**
```python
# Lấy default company từ Global Defaults
company = frappe.db.get_single_value("Global Defaults", "default_company")

# Lấy robots.txt từ Website Settings
robots_txt = frappe.db.get_single_value("Website Settings", "robots_txt")

# Lấy language từ System Settings
language = frappe.db.get_single_value("System Settings", "language")
```

**Lưu ý:**
- Mặc định sử dụng cache (`cache=True`)
- Chỉ dùng cho Single DocType

---

### 1.4. `frappe.db.get()`

Alias của `get_value` với `fieldname='*'` và `as_dict=True`.

**Cú pháp:**
```python
frappe.db.get(
    doctype: str,
    filters: dict | None = None,
    as_dict: bool = True,
    cache: bool = False,
)
```

**Ví dụ:**
```python
# Lấy toàn bộ document dạng dict
user = frappe.db.get("User", "test@example.com")
# Tương đương với:
# frappe.db.get_value("User", "test@example.com", "*", as_dict=True)
```

---

### 1.5. `frappe.db.exists()`

Kiểm tra document có tồn tại hay không.

**Cú pháp:**
```python
frappe.db.exists(
    dt: str,
    dn: str | dict | None = None,
    cache: bool = False,
    debug: bool = False,
)
```

**Ví dụ:**
```python
# Kiểm tra bằng docname
if frappe.db.exists("User", "test@example.com"):
    print("User exists")

# Kiểm tra bằng filters
if frappe.db.exists("User", {"email": "test@example.com"}):
    print("User exists")

# Kiểm tra với cache
if frappe.db.exists("User", "test@example.com", cache=True):
    print("User exists")
```

**Lưu ý:**
- Trả về `docname` nếu tồn tại, `None` nếu không
- Có thể cache kết quả nếu `dt` và `dn` là string

---

### 1.6. `frappe.db.count()`

Đếm số lượng documents thỏa mãn điều kiện.

**Cú pháp:**
```python
frappe.db.count(
    dt: str,
    filters: dict | None = None,
    debug: bool = False,
    cache: bool = False,
    distinct: bool = True,
)
```

**Ví dụ:**
```python
# Đếm tổng số users
total_users = frappe.db.count("User")

# Đếm users enabled
enabled_users = frappe.db.count("User", {"enabled": 1})

# Đếm với cache
total = frappe.db.count("User", cache=True)
```

---

### 1.7. `frappe.db.get_all()` (Static Method)

Wrapper cho `frappe.get_all()` (xem phần 2.1).

---

### 1.8. `frappe.db.get_list()` (Static Method)

Wrapper cho `frappe.get_list()` (xem phần 2.2).

---

## 2. frappe.get_* Methods (High-level Methods)

### 2.1. `frappe.get_all()`

Lấy danh sách documents **KHÔNG kiểm tra permissions**.

**Cú pháp:**
```python
frappe.get_all(
    doctype: str,
    fields: list[str] | str = None,
    filters: dict | list = None,
    or_filters: dict | list = None,
    order_by: str = None,
    group_by: str = None,
    limit_start: int = 0,
    limit_page_length: int = 0,  # 0 = không giới hạn
    limit: int = None,
    offset: int = None,
    as_list: bool = False,
    as_dict: bool = True,
    debug: bool = False,
    ignore_permissions: bool = True,  # Mặc định True
    user: str = None,
    distinct: bool = False,
    pluck: str = None,
)
```

**Ví dụ:**
```python
# Lấy tất cả users
all_users = frappe.get_all("User")

# Lấy users với filters
enabled_users = frappe.get_all(
    "User",
    fields=["name", "email", "full_name"],
    filters={"enabled": 1}
)

# Lấy với order_by
recent_users = frappe.get_all(
    "User",
    fields=["name", "creation"],
    order_by="creation desc",
    limit=10
)

# Lấy với pagination
page_1 = frappe.get_all(
    "User",
    limit_start=0,
    limit_page_length=20
)

page_2 = frappe.get_all(
    "User",
    limit_start=20,
    limit_page_length=20
)

# Lấy với or_filters
users = frappe.get_all(
    "User",
    filters={"enabled": 1},
    or_filters=[
        ["email", "like", "%@example.com"],
        ["full_name", "like", "Admin%"]
    ]
)

# Lấy với pluck (chỉ lấy 1 field)
user_names = frappe.get_all("User", pluck="name")

# Lấy dạng list (không phải dict)
users_list = frappe.get_all("User", as_list=True)
```

**Lưu ý:**
- **KHÔNG kiểm tra permissions** (mặc định `ignore_permissions=True`)
- Mặc định `limit_page_length=0` (không giới hạn)
- Trả về list các dict (hoặc list nếu `as_list=True`)

---

### 2.2. `frappe.get_list()`

Lấy danh sách documents **CÓ kiểm tra permissions**.

**Cú pháp:**
```python
frappe.get_list(
    doctype: str,
    fields: list[str] | str = None,
    filters: dict | list = None,
    or_filters: dict | list = None,
    order_by: str = None,
    group_by: str = None,
    limit_start: int = 0,
    limit_page_length: int = 20,  # Mặc định 20
    limit: int = None,
    offset: int = None,
    as_list: bool = False,
    as_dict: bool = True,
    debug: bool = False,
    ignore_permissions: bool = False,  # Mặc định False
    user: str = None,
    distinct: bool = False,
    pluck: str = None,
)
```

**Ví dụ:**
```python
# Lấy danh sách (có kiểm tra permissions)
my_todos = frappe.get_list("ToDo", filters={"owner": frappe.session.user})

# Lấy với pagination (mặc định 20 records/page)
todos = frappe.get_list(
    "ToDo",
    fields=["name", "description", "status"],
    filters={"status": "Open"},
    limit_start=0,
    limit_page_length=20
)

# Lấy với order_by
recent_todos = frappe.get_list(
    "ToDo",
    order_by="modified desc",
    limit=10
)
```

**Lưu ý:**
- **CÓ kiểm tra permissions** (mặc định `ignore_permissions=False`)
- Mặc định `limit_page_length=20`
- Chỉ trả về documents mà user có quyền đọc

---

### 2.3. `frappe.get_value()`

Alias của `frappe.db.get_value()`.

**Ví dụ:**
```python
last_login = frappe.get_value("User", "test@example.com", "last_login")
```

---

### 2.4. `frappe.get_doc()`

Lấy document object (có đầy đủ methods và properties).

**Cú pháp:**
```python
frappe.get_doc(
    doctype: str,
    name: str = None,
    for_update: bool = None,
    check_permission: str | bool | None = None,
)
```

**Ví dụ:**
```python
# Lấy document
user = frappe.get_doc("User", "test@example.com")

# Truy cập fields
print(user.email)
print(user.full_name)

# Lấy child table
for role in user.roles:
    print(role.role)

# Lấy Single DocType
system_settings = frappe.get_doc("System Settings")

# Lấy với for_update (lock row)
user = frappe.get_doc("User", "test@example.com", for_update=True)

# Lấy với check_permission
user = frappe.get_doc(
    "User",
    "test@example.com",
    check_permission="read"
)

# Tạo document mới từ dict
new_user = frappe.get_doc({
    "doctype": "User",
    "email": "new@example.com",
    "first_name": "New",
    "roles": [
        {"role": "System User"}
    ]
})
```

**Lưu ý:**
- Trả về `Document` object (có methods như `save()`, `delete()`, `submit()`, etc.)
- Có thể truy cập child tables
- Có thể validate và trigger document events

---

## 3. Query Builder (frappe.qb) - Modern Approach

Query Builder là cách hiện đại và type-safe để query dữ liệu trong Frappe v16.

### 3.1. Basic Query với `frappe.qb.from_()`

**Cú pháp:**
```python
from frappe.query_builder import DocType, Field

# Tạo DocType object
User = frappe.qb.DocType("User")

# Query cơ bản
query = (
    frappe.qb.from_(User)
    .select(User.name, User.email, User.full_name)
    .where(User.enabled == 1)
    .orderby(User.creation, order=frappe.qb.desc)
    .limit(10)
)

# Chạy query
results = query.run(as_dict=True)
```

**Ví dụ:**
```python
from frappe.query_builder import DocType, Field, functions

User = frappe.qb.DocType("User")
Customer = frappe.qb.DocType("Customer")

# Select với conditions
query = (
    frappe.qb.from_(User)
    .select(User.name, User.email)
    .where(User.enabled == 1)
    .where(User.email.like("%@example.com"))
)

# Select với join
query = (
    frappe.qb.from_(Customer)
    .left_join(User).on(Customer.email == User.email)
    .select(Customer.name, Customer.customer_name, User.full_name)
)

# Select với aggregate functions
query = (
    frappe.qb.from_(User)
    .select(functions.Count(User.name).as_("total"))
    .where(User.enabled == 1)
)

# Select với group_by
query = (
    frappe.qb.from_(Customer)
    .select(Customer.territory, functions.Count(Customer.name).as_("count"))
    .groupby(Customer.territory)
)

# Select với having
query = (
    frappe.qb.from_(Customer)
    .select(Customer.territory, functions.Count(Customer.name).as_("count"))
    .groupby(Customer.territory)
    .having(functions.Count(Customer.name) > 10)
)
```

---

### 3.2. Update Query

**Ví dụ:**
```python
User = frappe.qb.DocType("User")

# Update single field
(
    frappe.qb.update(User)
    .set(User.enabled, 0)
    .where(User.email == "old@example.com")
).run()

# Update multiple fields
(
    frappe.qb.update(User)
    .set(User.enabled, 1)
    .set(User.modified, frappe.utils.now())
    .where(User.email == "test@example.com")
).run()
```

---

### 3.3. Delete Query

**Ví dụ:**
```python
User = frappe.qb.DocType("User")

# Delete với condition
(
    frappe.qb.from_(User)
    .delete()
    .where(User.enabled == 0)
).run()
```

---

### 3.4. Insert Query

**Ví dụ:**
```python
User = frappe.qb.DocType("User")

# Insert single record
(
    frappe.qb.into(User)
    .columns(User.name, User.email, User.enabled)
    .insert("user1", "user1@example.com", 1)
).run()

# Insert multiple records
(
    frappe.qb.into(User)
    .columns(User.name, User.email, User.enabled)
    .insert(
        ("user1", "user1@example.com", 1),
        ("user2", "user2@example.com", 1),
        ("user3", "user3@example.com", 0)
    )
).run()
```

---

### 3.5. Sử dụng `frappe.qb.get_query()`

Wrapper method để tạo query từ filters và fields.

**Cú pháp:**
```python
frappe.qb.get_query(
    table: str | Table,
    fields: list | tuple | None = None,
    filters: dict | list | None = None,
    order_by: str | None = None,
    group_by: str | None = None,
    limit: int | None = None,
    offset: int | None = None,
    distinct: bool = False,
    for_update: bool = False,
    update: bool = False,
    into: bool = False,
    delete: bool = False,
    ignore_permissions: bool = False,
    ignore_user_permissions: bool = False,
    user: str | None = None,
    reference_doctype: str | None = None,
    parent_doctype: str | None = None,
    validate_filters: bool = False,
    skip_locked: bool = False,
    wait: bool = True,
) -> QueryBuilder
```

**Ví dụ:**
```python
# Query với filters dict
query = frappe.qb.get_query(
    table="User",
    fields=["name", "email", "full_name"],
    filters={"enabled": 1},
    order_by="creation desc",
    limit=10
)
results = query.run(as_dict=True)

# Query với filters list
query = frappe.qb.get_query(
    table="User",
    fields=["name", "email"],
    filters=[
        ["enabled", "=", 1],
        ["email", "like", "%@example.com"]
    ]
)
results = query.run(as_dict=True)

# Query với for_update
query = frappe.qb.get_query(
    table="User",
    filters={"email": "test@example.com"},
    for_update=True
)
results = query.run(as_dict=True)
```

---

### 3.6. Functions và Operators

**Ví dụ:**
```python
from frappe.query_builder import functions

User = frappe.qb.DocType("User")

# Count
query = (
    frappe.qb.from_(User)
    .select(functions.Count(User.name).as_("total"))
)

# Sum, Avg, Max, Min
query = (
    frappe.qb.from_("Sales Invoice")
    .select(
        functions.Sum("grand_total").as_("total"),
        functions.Avg("grand_total").as_("average"),
        functions.Max("grand_total").as_("max"),
        functions.Min("grand_total").as_("min")
    )
)

# Date functions
from frappe.query_builder.functions import Date, Now, Extract

query = (
    frappe.qb.from_(User)
    .select(
        Date(User.creation).as_("creation_date"),
        Extract("year", User.creation).as_("year")
    )
    .where(User.creation >= Now() - frappe.query_builder.Interval(days=30))
)

# String functions
from frappe.query_builder.functions import Concat, Upper, Lower

query = (
    frappe.qb.from_(User)
    .select(
        Concat(User.first_name, " ", User.last_name).as_("full_name"),
        Upper(User.email).as_("email_upper")
    )
)

# Case when
from frappe.query_builder.terms import Case

query = (
    frappe.qb.from_(User)
    .select(
        Case()
        .when(User.enabled == 1, "Active")
        .else_("Inactive")
        .as_("status")
    )
)
```

---

## 4. Raw SQL (frappe.db.sql)

Sử dụng SQL trực tiếp (không khuyến khích, chỉ dùng khi cần thiết).

### 4.1. `frappe.db.sql()`

**Cú pháp:**
```python
frappe.db.sql(
    query: str,
    values: tuple | dict = (),
    as_dict: bool = False,
    as_list: bool = False,
    debug: bool = False,
    autocommit: bool = False,
    update: dict = None,
)
```

**Ví dụ:**
```python
# Query đơn giản
results = frappe.db.sql("SELECT name, email FROM `tabUser` WHERE enabled = 1")

# Query với parameters (an toàn hơn)
results = frappe.db.sql(
    "SELECT name, email FROM `tabUser` WHERE enabled = %s",
    (1,),
    as_dict=True
)

# Query với named parameters
results = frappe.db.sql(
    "SELECT name, email FROM `tabUser` WHERE enabled = %(enabled)s",
    {"enabled": 1},
    as_dict=True
)

# Query phức tạp
results = frappe.db.sql("""
    SELECT 
        u.name,
        u.email,
        COUNT(si.name) as invoice_count
    FROM `tabUser` u
    LEFT JOIN `tabSales Invoice` si ON si.owner = u.name
    WHERE u.enabled = 1
    GROUP BY u.name, u.email
    HAVING invoice_count > 10
""", as_dict=True)

# Query với update
frappe.db.sql("""
    UPDATE `tabUser`
    SET enabled = 0
    WHERE last_login < DATE_SUB(NOW(), INTERVAL 1 YEAR)
""", autocommit=True)
```

**Lưu ý:**
- **Luôn sử dụng parameterized queries** để tránh SQL injection
- Không khuyến khích dùng raw SQL trừ khi thực sự cần thiết
- Không tự động kiểm tra permissions
- Cần escape table names với backticks (`` `tabUser` ``)

---

### 4.2. `frappe.db.sql_list()`

Tương tự `sql()` nhưng trả về list đơn giản.

**Ví dụ:**
```python
user_names = frappe.db.sql_list("SELECT name FROM `tabUser` WHERE enabled = 1")
```

---

### 4.3. `frappe.db.sql_ddl()`

Thực thi DDL statements (CREATE, ALTER, DROP).

**Ví dụ:**
```python
frappe.db.sql_ddl("ALTER TABLE `tabUser` ADD COLUMN test_field VARCHAR(255)")
```

---

## 5. Document Methods (frappe.get_doc)

### 5.1. `frappe.get_doc()`

Đã được mô tả ở phần 2.4.

### 5.2. `frappe.get_lazy_doc()`

Lấy document nhưng không load ngay (lazy loading).

**Ví dụ:**
```python
user = frappe.get_lazy_doc("User", "test@example.com")
# Document chỉ được load khi truy cập properties
print(user.email)  # Load tại đây
```

---

## 6. So Sánh và Lựa Chọn Phương Thức

### 6.1. Khi Nào Dùng Gì?

| Mục Đích | Phương Thức Khuyến Nghị | Lý Do |
|----------|------------------------|-------|
| Lấy 1 giá trị đơn | `frappe.db.get_value()` | Đơn giản, nhanh |
| Lấy nhiều giá trị từ 1 document | `frappe.db.get_value()` với list fields | Đơn giản |
| Lấy nhiều documents (không cần permissions) | `frappe.get_all()` | Nhanh, không check permissions |
| Lấy nhiều documents (cần permissions) | `frappe.get_list()` | Tự động check permissions |
| Lấy document object (cần methods) | `frappe.get_doc()` | Có đầy đủ methods và events |
| Query phức tạp với joins | `frappe.qb` (Query Builder) | Type-safe, dễ đọc |
| Query với aggregate functions | `frappe.qb` | Hỗ trợ tốt functions |
| Query động (filters từ user input) | `frappe.get_all()` hoặc `frappe.qb.get_query()` | Dễ xử lý filters |
| Query tối ưu performance | `frappe.qb` | Query Builder tối ưu tốt |
| Query cực kỳ phức tạp | `frappe.db.sql()` | Chỉ khi thực sự cần |
| Kiểm tra tồn tại | `frappe.db.exists()` | Nhanh, có cache |
| Đếm số lượng | `frappe.db.count()` | Tối ưu cho COUNT |
| Lấy từ Single DocType | `frappe.db.get_single_value()` | Có cache, tối ưu |

### 6.2. Performance Tips

1. **Sử dụng cache khi có thể:**
   ```python
   # Cache cho get_value
   value = frappe.db.get_value("User", "test@example.com", "email", cache=True)
   
   # Cache cho get_single_value (mặc định đã cache)
   value = frappe.db.get_single_value("System Settings", "language")
   
   # Cache cho exists
   exists = frappe.db.exists("User", "test@example.com", cache=True)
   ```

2. **Sử dụng `pluck` khi chỉ cần 1 field:**
   ```python
   # Thay vì
   users = frappe.get_all("User", fields=["name"])
   names = [u["name"] for u in users]
   
   # Dùng
   names = frappe.get_all("User", pluck="name")
   ```

3. **Sử dụng `limit` để giới hạn kết quả:**
   ```python
   # Chỉ lấy 10 records đầu tiên
   users = frappe.get_all("User", limit=10)
   ```

4. **Sử dụng `distinct` khi cần:**
   ```python
   # Loại bỏ duplicates
   territories = frappe.get_all("Customer", fields=["territory"], distinct=True)
   ```

5. **Tránh `select *` khi không cần:**
   ```python
   # Tốt: chỉ lấy fields cần thiết
   users = frappe.get_all("User", fields=["name", "email"])
   
   # Không tốt: lấy tất cả
   users = frappe.get_all("User", fields=["*"])
   ```

### 6.3. Security Tips

1. **Luôn kiểm tra permissions khi cần:**
   ```python
   # Dùng get_list thay vì get_all khi cần check permissions
   my_docs = frappe.get_list("ToDo", filters={"owner": frappe.session.user})
   ```

2. **Sử dụng parameterized queries với SQL:**
   ```python
   # Tốt
   frappe.db.sql("SELECT * FROM `tabUser` WHERE email = %s", (email,))
   
   # Không tốt (SQL injection risk)
   frappe.db.sql(f"SELECT * FROM `tabUser` WHERE email = '{email}'")
   ```

3. **Validate user input trước khi query:**
   ```python
   # Validate filters từ user input
   filters = frappe.parse_json(frappe.form_dict.filters)
   # Sanitize filters...
   results = frappe.get_all("User", filters=filters)
   ```

---

## 7. Ví Dụ Tổng Hợp

### 7.1. Lấy danh sách customers với tổng doanh thu

```python
from frappe.query_builder import functions

Customer = frappe.qb.DocType("Customer")
SalesInvoice = frappe.qb.DocType("Sales Invoice")

query = (
    frappe.qb.from_(Customer)
    .left_join(SalesInvoice).on(
        (SalesInvoice.customer == Customer.name) & 
        (SalesInvoice.docstatus == 1)
    )
    .select(
        Customer.name,
        Customer.customer_name,
        functions.Sum(SalesInvoice.grand_total).as_("total_revenue")
    )
    .groupby(Customer.name, Customer.customer_name)
    .orderby("total_revenue", order=frappe.qb.desc)
    .limit(10)
)

results = query.run(as_dict=True)
```

### 7.2. Lấy users chưa login trong 30 ngày

```python
from frappe.query_builder.functions import Now
from frappe.query_builder import Interval

User = frappe.qb.DocType("User")

query = (
    frappe.qb.from_(User)
    .select(User.name, User.email, User.last_login)
    .where(User.enabled == 1)
    .where(
        (User.last_login < Now() - Interval(days=30)) |
        (User.last_login.isnull())
    )
)

results = query.run(as_dict=True)
```

### 7.3. Update batch với Query Builder

```python
User = frappe.qb.DocType("User")

(
    frappe.qb.update(User)
    .set(User.enabled, 0)
    .set(User.modified, frappe.utils.now())
    .where(User.last_login < frappe.utils.add_days(frappe.utils.today(), -365))
).run()
```

---

## 8. Kết Luận

Frappe v16 cung cấp nhiều cách để query dữ liệu:

1. **Low-level methods** (`frappe.db.*`): Cho các thao tác đơn giản, nhanh
2. **High-level methods** (`frappe.get_*`): Cho các thao tác thông thường, có kiểm tra permissions
3. **Query Builder** (`frappe.qb`): Cho các query phức tạp, type-safe, hiện đại
4. **Raw SQL** (`frappe.db.sql`): Chỉ dùng khi thực sự cần thiết

**Khuyến nghị:**
- Ưu tiên sử dụng **Query Builder** (`frappe.qb`) cho các query mới
- Sử dụng `frappe.get_all()` / `frappe.get_list()` cho các query đơn giản
- Sử dụng `frappe.db.get_value()` cho các thao tác đơn giản
- Tránh raw SQL trừ khi thực sự cần thiết

---

**Tài liệu tham khảo:**
- Frappe Framework Documentation: https://frappeframework.com/docs
- Query Builder Guide: https://frappeframework.com/docs/v14/user/en/api/query-builder
- Database API: https://frappeframework.com/docs/v14/user/en/api/database
