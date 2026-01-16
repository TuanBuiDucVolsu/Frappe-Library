# Restrict To Domain - Hướng dẫn Chi tiết

File này giải thích **cách hoạt động của "Restrict To Domain"** trong DocType settings của Frappe.

---

## 📋 Tổng quan

**Restrict To Domain** là một setting trong DocType cho phép **giới hạn DocType chỉ hiển thị và có thể truy cập khi domain tương ứng được kích hoạt (active)** trong Domain Settings.

---

## 🔷 1. Domain là gì?

### 1.1. Khái niệm

**Domain** trong Frappe là một cách để **phân chia và tổ chức functionality** của hệ thống theo từng lĩnh vực/nghề nghiệp cụ thể.

**Ví dụ các Domain phổ biến:**
- `Manufacturing` - Sản xuất
- `Healthcare` - Y tế
- `Education` - Giáo dục
- `Non Profit` - Tổ chức phi lợi nhuận
- `Retail` - Bán lẻ
- `Agriculture` - Nông nghiệp

### 1.2. Domain DocType

Domain là một DocType đơn giản trong Frappe:

```json
{
    "doctype": "Domain",
    "fieldname": "domain",
    "fieldtype": "Data",
    "label": "Domain",
    "reqd": 1,
    "unique": 1
}
```

**Lưu ý:** Domain chỉ có một field duy nhất là `domain` (tên domain).

---

## 🔷 2. Cách Domain hoạt động

### 2.1. Active Domains

**Active Domains** là các domain được kích hoạt trong **Domain Settings**.

**Cách kích hoạt Domain:**
1. Vào **Domain Settings** (Setup > Domain Settings)
2. Thêm domain vào danh sách **Active Domains**
3. Save

**Lưu ý:** Có thể có nhiều active domains cùng lúc.

### 2.2. Domain Settings

Domain Settings là một DocType quản lý các active domains:

```python
# File: frappe/core/doctype/domain_settings/domain_settings.py

def get_active_domains():
    """get the domains set in the Domain Settings as active domain"""
    domains = frappe.get_all(
        "Has Domain",
        filters={"parent": "Domain Settings"},
        fields=["domain"],
        distinct=True,
    )
    active_domains = [row.get("domain") for row in domains]
    active_domains.append("")  # Empty string = no restriction
    return active_domains
```

**Kết quả:** Trả về danh sách các domain đang active, bao gồm cả empty string `""` (không giới hạn).

---

## 🔷 3. Restrict To Domain trong DocType

### 3.1. Cách set Restrict To Domain

**Trong DocType Settings:**
1. Mở DocType cần giới hạn
2. Vào tab **Permissions**
3. Tìm field **Restrict To Domain**
4. Chọn domain từ dropdown (Link field đến DocType "Domain")
5. Save

**Ví dụ:**
```json
{
    "doctype": "Sales Invoice",
    "restrict_to_domain": "Retail"
}
```

### 3.2. Logic hoạt động

**DocType chỉ hiển thị và có thể truy cập khi:**

```python
# File: frappe/utils/user.py

def build_doctype_map(self):
    active_domains = frappe.get_active_domains()
    all_doctypes = frappe.get_all("DocType", fields=["name", "restrict_to_domain"])
    
    for dt in all_doctypes:
        # DocType được hiển thị nếu:
        # 1. Không có restrict_to_domain (null hoặc empty)
        # 2. HOẶC restrict_to_domain nằm trong active_domains
        if not dt.restrict_to_domain or (dt.restrict_to_domain in active_domains):
            self.doctype_map[dt["name"]] = dt
```

**Điều kiện:**
- `restrict_to_domain` là `null` hoặc `""` → **Luôn hiển thị** (không giới hạn)
- `restrict_to_domain` có giá trị → **Chỉ hiển thị khi domain đó active**

---

## 🔷 4. Ví dụ Cụ thể

### 4.1. Ví dụ 1: DocType không giới hạn

```json
{
    "doctype": "Customer",
    "restrict_to_domain": null  // hoặc ""
}
```

**Kết quả:**
- ✅ Luôn hiển thị, bất kể active domains là gì
- ✅ Tất cả users đều thấy

### 4.2. Ví dụ 2: DocType giới hạn cho Manufacturing

```json
{
    "doctype": "Work Order",
    "restrict_to_domain": "Manufacturing"
}
```

**Kịch bản 1: Manufacturing domain KHÔNG active**
- ❌ DocType **KHÔNG hiển thị** trong List View
- ❌ Users **KHÔNG thể truy cập** DocType này
- ❌ DocType **KHÔNG xuất hiện** trong search
- ❌ DocType **KHÔNG xuất hiện** trong desktop icons

**Kịch bản 2: Manufacturing domain ACTIVE**
- ✅ DocType **hiển thị** trong List View
- ✅ Users **có thể truy cập** DocType này
- ✅ DocType **xuất hiện** trong search
- ✅ DocType **xuất hiện** trong desktop icons

### 4.3. Ví dụ 3: Nhiều Domains

**Active Domains:** `["Manufacturing", "Retail", ""]`

**DocTypes:**
```json
[
    {
        "doctype": "Sales Invoice",
        "restrict_to_domain": null  // Không giới hạn
    },
    {
        "doctype": "Work Order",
        "restrict_to_domain": "Manufacturing"  // Giới hạn Manufacturing
    },
    {
        "doctype": "POS Invoice",
        "restrict_to_domain": "Retail"  // Giới hạn Retail
    },
    {
        "doctype": "Patient",
        "restrict_to_domain": "Healthcare"  // Giới hạn Healthcare
    }
]
```

**Kết quả:**
- ✅ `Sales Invoice` - Hiển thị (không giới hạn)
- ✅ `Work Order` - Hiển thị (Manufacturing active)
- ✅ `POS Invoice` - Hiển thị (Retail active)
- ❌ `Patient` - **KHÔNG hiển thị** (Healthcare không active)

---

## 🔷 5. Nơi Restrict To Domain được áp dụng

### 5.1. DocType Map (User Permissions)

```python
# File: frappe/utils/user.py

def build_doctype_map(self):
    active_domains = frappe.get_active_domains()
    all_doctypes = frappe.get_all("DocType", fields=["name", "restrict_to_domain"])
    
    for dt in all_doctypes:
        if not dt.restrict_to_domain or (dt.restrict_to_domain in active_domains):
            self.doctype_map[dt["name"]] = dt
```

**Ảnh hưởng:**
- DocType không có trong `doctype_map` → User không thể truy cập

### 5.2. Desktop Icons

```python
# File: frappe/desk/doctype/desktop_icon/desktop_icon.py

def get_desktop_icons(user=None, bootinfo=None):
    active_domains = frappe.get_active_domains()
    
    DocType = frappe.qb.DocType("DocType")
    if active_domains:
        blocked_condition = (
            (DocType.restrict_to_domain.isnull())
            | (DocType.restrict_to_domain == "")
            | (DocType.restrict_to_domain.notin(active_domains))
        )
    else:
        blocked_condition = (DocType.restrict_to_domain.isnull()) | (DocType.restrict_to_domain == "")
    
    blocked_doctypes = [
        d.get("name")
        for d in frappe.qb.from_(DocType)
        .select(DocType.name)
        .where(blocked_condition)
        .run(as_dict=True)
    ]
    
    for icon in standard_icons:
        if icon._doctype in blocked_doctypes:
            icon.blocked = 1
```

**Ảnh hưởng:**
- Desktop icons của DocType bị blocked → Không hiển thị

### 5.3. Permission Manager

```python
# File: frappe/core/page/permission_manager/permission_manager.py

def get_roles_and_doctypes():
    active_domains = frappe.get_active_domains()
    
    DocType = frappe.qb.DocType("DocType")
    doctype_domain_condition = (DocType.restrict_to_domain.isnull()) | (DocType.restrict_to_domain == "")
    if active_domains:
        doctype_domain_condition = doctype_domain_condition | DocType.restrict_to_domain.isin(active_domains)
    
    doctypes = (
        frappe.qb.from_(DocType)
        .select(DocType.name)
        .where(
            (DocType.istable == 0)
            & (DocType.name.notin(not_allowed_in_permission_manager))
            & doctype_domain_condition
        )
        .run(as_dict=True)
    )
```

**Ảnh hưởng:**
- DocType không xuất hiện trong Permission Manager

### 5.4. Cache

```python
# File: frappe/cache_manager.py

def build_domain_restricted_doctype_cache(*args, **kwargs):
    active_domains = frappe.get_active_domains()
    doctypes = frappe.get_all("DocType", filters={"restrict_to_domain": ("IN", active_domains)})
    doctypes = [doc.name for doc in doctypes]
    frappe.cache.set_value("domain_restricted_doctypes", doctypes)
    return doctypes
```

**Ảnh hưởng:**
- Cache các DocType bị giới hạn để tăng performance

---

## 🔷 6. Use Cases

### 6.1. Use Case 1: Multi-Industry Application

**Tình huống:** Bạn có một ứng dụng ERP hỗ trợ nhiều ngành:
- Manufacturing (Sản xuất)
- Retail (Bán lẻ)
- Healthcare (Y tế)
- Education (Giáo dục)

**Giải pháp:**
- Tạo các DocType riêng cho từng ngành
- Set `restrict_to_domain` cho từng DocType
- User chỉ thấy DocTypes của ngành họ đang dùng

**Ví dụ:**
```json
{
    "doctype": "Work Order",
    "restrict_to_domain": "Manufacturing"
},
{
    "doctype": "POS Invoice",
    "restrict_to_domain": "Retail"
},
{
    "doctype": "Patient",
    "restrict_to_domain": "Healthcare"
}
```

### 6.2. Use Case 2: Module Activation

**Tình huống:** Bạn muốn ẩn một số DocTypes cho đến khi user kích hoạt module tương ứng.

**Giải pháp:**
- Tạo Domain cho module
- Set `restrict_to_domain` cho các DocTypes của module
- Chỉ hiển thị khi user kích hoạt domain

**Ví dụ:**
```json
{
    "doctype": "Advanced Manufacturing",
    "restrict_to_domain": "Advanced Manufacturing Module"
}
```

### 6.3. Use Case 3: Custom App Distribution

**Tình huống:** Bạn tạo một custom app cho một khách hàng cụ thể.

**Giải pháp:**
- Tạo Domain riêng cho khách hàng
- Set `restrict_to_domain` cho các DocTypes custom
- Chỉ khách hàng đó mới thấy DocTypes này

**Ví dụ:**
```json
{
    "doctype": "Custom Invoice",
    "restrict_to_domain": "Client ABC"
}
```

---

## 🔷 7. Lưu ý Quan trọng

### 7.1. Child Tables

**Lưu ý:** `restrict_to_domain` **KHÔNG áp dụng trực tiếp** cho child tables, nhưng child table sẽ bị ảnh hưởng gián tiếp qua parent DocType.

**Ví dụ:**
```json
{
    "doctype": "Sales Invoice",  // Parent
    "restrict_to_domain": "Retail"
},
{
    "doctype": "Sales Invoice Item",  // Child table
    "restrict_to_domain": null  // Không cần set
}
```

**Kết quả:**
- Nếu `Sales Invoice` bị ẩn → `Sales Invoice Item` cũng không thể truy cập

### 7.2. Permissions

**Lưu ý:** `restrict_to_domain` **KHÔNG thay thế** permissions. Nó chỉ **ẩn DocType**, nhưng nếu user có quyền truy cập trực tiếp (qua URL), họ vẫn có thể truy cập.

**Best Practice:**
- Kết hợp `restrict_to_domain` với **Permissions** để đảm bảo an toàn

### 7.3. Cache

**Lưu ý:** Sau khi thay đổi active domains, cần **clear cache**:

```python
frappe.clear_cache()
```

Hoặc restart server để cache được refresh.

### 7.4. Migration

**Lưu ý:** Khi migrate DocType có `restrict_to_domain`, cần đảm bảo Domain đã tồn tại:

```python
# Trong migration
if not frappe.db.exists("Domain", "Manufacturing"):
    frappe.get_doc({
        "doctype": "Domain",
        "domain": "Manufacturing"
    }).insert()
```

### 7.5. Empty String vs Null

**Lưu ý:** Cả `null` và `""` (empty string) đều có nghĩa là **không giới hạn**:

```python
# Cả hai đều được xử lý như nhau
if not dt.restrict_to_domain or (dt.restrict_to_domain == ""):
    # DocType không bị giới hạn
```

---

## 🔷 8. So sánh với Permissions

### 8.1. Restrict To Domain vs Permissions

| Tính năng | Restrict To Domain | Permissions |
|----------|-------------------|-------------|
| **Mục đích** | Ẩn/hiện DocType theo domain | Kiểm soát quyền truy cập |
| **Áp dụng** | Toàn bộ DocType | Từng role/user |
| **Level** | DocType level | Document level |
| **UI** | Ẩn hoàn toàn khỏi UI | Vẫn hiển thị nhưng không truy cập được |
| **Search** | Không xuất hiện trong search | Có thể search nhưng không đọc được |
| **Use Case** | Multi-industry, module activation | Access control |

### 8.2. Kết hợp sử dụng

**Best Practice:** Kết hợp cả hai:

```json
{
    "doctype": "Work Order",
    "restrict_to_domain": "Manufacturing",  // Chỉ hiển thị khi Manufacturing active
    "permissions": [
        {
            "role": "Manufacturing User",
            "read": 1,
            "write": 1,
            "create": 1
        },
        {
            "role": "Manufacturing Manager",
            "read": 1,
            "write": 1,
            "create": 1,
            "delete": 1,
            "submit": 1,
            "cancel": 1
        }
    ]
}
```

---

## 🔷 9. Troubleshooting

### 9.1. DocType không hiển thị

**Vấn đề:** DocType có `restrict_to_domain` nhưng không hiển thị.

**Kiểm tra:**
1. Domain có được kích hoạt trong Domain Settings không?
2. Cache đã được clear chưa?
3. User có permissions không?

**Giải pháp:**
```python
# Kiểm tra active domains
active_domains = frappe.get_active_domains()
print(active_domains)  # ['Manufacturing', 'Retail', '']

# Kiểm tra DocType
doctype = frappe.get_doc("DocType", "Work Order")
print(doctype.restrict_to_domain)  # 'Manufacturing'

# Clear cache
frappe.clear_cache()
```

### 9.2. DocType vẫn hiển thị khi không nên

**Vấn đề:** DocType có `restrict_to_domain` nhưng vẫn hiển thị khi domain không active.

**Kiểm tra:**
1. `restrict_to_domain` có giá trị đúng không?
2. Domain name có match chính xác không?
3. Cache đã được refresh chưa?

**Giải pháp:**
```python
# Kiểm tra giá trị
doctype = frappe.get_doc("DocType", "Work Order")
print(doctype.restrict_to_domain)  # Phải là 'Manufacturing' (chính xác)

# Kiểm tra active domains
active_domains = frappe.get_active_domains()
print("Manufacturing" in active_domains)  # Phải là True

# Clear cache
frappe.clear_cache()
```

### 9.3. Domain không tồn tại

**Vấn đề:** Domain được set trong `restrict_to_domain` nhưng không tồn tại.

**Giải pháp:**
```python
# Tạo Domain nếu chưa có
if not frappe.db.exists("Domain", "Manufacturing"):
    frappe.get_doc({
        "doctype": "Domain",
        "domain": "Manufacturing"
    }).insert()
```

---

## 🔷 10. Code Examples

### 10.1. Kiểm tra DocType có bị giới hạn không

```python
import frappe

def is_doctype_visible(doctype_name):
    """Kiểm tra DocType có hiển thị không dựa trên restrict_to_domain"""
    active_domains = frappe.get_active_domains()
    doctype = frappe.get_doc("DocType", doctype_name)
    
    if not doctype.restrict_to_domain:
        return True  # Không giới hạn
    
    return doctype.restrict_to_domain in active_domains

# Sử dụng
if is_doctype_visible("Work Order"):
    print("Work Order is visible")
else:
    print("Work Order is hidden")
```

### 10.2. Lấy tất cả DocTypes có thể truy cập

```python
import frappe

def get_accessible_doctypes():
    """Lấy danh sách DocTypes có thể truy cập"""
    active_domains = frappe.get_active_domains()
    
    doctypes = frappe.get_all(
        "DocType",
        fields=["name", "restrict_to_domain"],
        filters={"istable": 0}  # Chỉ lấy parent doctypes
    )
    
    accessible = []
    for dt in doctypes:
        if not dt.restrict_to_domain or (dt.restrict_to_domain in active_domains):
            accessible.append(dt.name)
    
    return accessible

# Sử dụng
accessible_doctypes = get_accessible_doctypes()
print(f"Accessible DocTypes: {accessible_doctypes}")
```

### 10.3. Set restrict_to_domain cho DocType

```python
import frappe

def set_doctype_domain(doctype_name, domain_name):
    """Set restrict_to_domain cho DocType"""
    # Đảm bảo Domain tồn tại
    if not frappe.db.exists("Domain", domain_name):
        frappe.get_doc({
            "doctype": "Domain",
            "domain": domain_name
        }).insert()
    
    # Set restrict_to_domain
    doctype = frappe.get_doc("DocType", doctype_name)
    doctype.restrict_to_domain = domain_name
    doctype.save()
    
    # Clear cache
    frappe.clear_cache()
    
    print(f"Set restrict_to_domain for {doctype_name} to {domain_name}")

# Sử dụng
set_doctype_domain("Work Order", "Manufacturing")
```

---

## 📝 Tóm tắt

### Các điểm chính:

1. **Domain** là cách phân chia functionality theo lĩnh vực/nghề nghiệp
2. **Active Domains** được quản lý trong Domain Settings
3. **Restrict To Domain** giới hạn DocType chỉ hiển thị khi domain tương ứng active
4. **Logic:** DocType hiển thị nếu `restrict_to_domain` là null/empty HOẶC nằm trong active_domains
5. **Áp dụng:** DocType Map, Desktop Icons, Permission Manager, Search, Cache
6. **Use Cases:** Multi-industry apps, module activation, custom app distribution
7. **Best Practice:** Kết hợp với Permissions để đảm bảo an toàn

### Công thức:

```
DocType hiển thị = (restrict_to_domain == null || restrict_to_domain == "") 
                   || (restrict_to_domain IN active_domains)
```

---

## 🔗 Tài liệu tham khảo

- **File source:** 
  - `apps/frappe/frappe/utils/user.py` (build_doctype_map)
  - `apps/frappe/frappe/core/doctype/domain_settings/domain_settings.py` (get_active_domains)
  - `apps/frappe/frappe/desk/doctype/desktop_icon/desktop_icon.py` (get_desktop_icons)
  - `apps/frappe/frappe/cache_manager.py` (build_domain_restricted_doctype_cache)
