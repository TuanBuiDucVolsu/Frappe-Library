# `frappe.query_builder.functions` - Tất cả các thành phần

File này liệt kê **tất cả các thành phần** có thể import từ `frappe.query_builder.functions` trong Frappe v16.

---

## 📦 Import cơ bản

```python
from frappe.query_builder.functions import *
# hoặc
from frappe.query_builder import functions as fn
```

---

## 🔷 1. Từ `pypika.functions` (được re-export)

File `functions.py` import `from pypika.functions import *`, nghĩa là **tất cả** các SQL functions từ PyPika đều có sẵn:

### 1.1. Aggregate Functions
- `Count(term, alias=None)` - COUNT() - **Đếm số lượng bản ghi** (hoặc số giá trị khác NULL)
- `Sum(term, alias=None)` - SUM() - **Tính tổng** các giá trị số
- `Avg(term, alias=None)` - AVG() - **Tính trung bình** các giá trị số
- `Max(term, alias=None)` - MAX() - **Tìm giá trị lớn nhất**
- `Min(term, alias=None)` - MIN() - **Tìm giá trị nhỏ nhất**
- `StdDev(term, alias=None)` - STDDEV() - **Tính độ lệch chuẩn** (standard deviation)
- `Variance(term, alias=None)` - VARIANCE() - **Tính phương sai** (variance)

**Ví dụ:**
```python
from frappe.query_builder import DocType, Count, Sum, Avg, Max, Min

Order = DocType("Sales Order")
q = frappe.qb.from_(Order).select(
    Count(Order.name).as_("total_orders"),
    Sum(Order.grand_total).as_("total_amount"),
    Avg(Order.grand_total).as_("avg_amount"),
    Max(Order.grand_total).as_("max_amount"),
    Min(Order.grand_total).as_("min_amount")
)
```

### 1.2. Mathematical Functions
- `Abs(term, alias=None)` - ABS() - **Lấy giá trị tuyệt đối** (bỏ dấu âm)
- `Round(term, decimals=0, alias=None)` - ROUND() - **Làm tròn** số đến số chữ số thập phân chỉ định
- `Floor(term, alias=None)` - FLOOR() - **Làm tròn xuống** (số nguyên nhỏ hơn gần nhất)
- `Ceil(term, alias=None)` - CEIL() - **Làm tròn lên** (số nguyên lớn hơn gần nhất)
- `Sqrt(term, alias=None)` - SQRT() - **Tính căn bậc hai**
- `Power(term, exponent, alias=None)` - POWER() - **Tính lũy thừa** (term^exponent)
- `Mod(term, divisor, alias=None)` - MOD() - **Tính số dư** của phép chia (modulo)
- `Sign(term, alias=None)` - SIGN() - **Trả về dấu** của số (-1 nếu âm, 0 nếu bằng 0, 1 nếu dương)

**Ví dụ:**
```python
from frappe.query_builder import DocType, Abs, Round, Floor, Ceil, Sqrt

Item = DocType("Item")
q = frappe.qb.from_(Item).select(
    Item.standard_rate,
    Abs(Item.standard_rate).as_("abs_rate"),
    Round(Item.standard_rate, 2).as_("rounded_rate"),
    Floor(Item.standard_rate).as_("floor_rate"),
    Ceil(Item.standard_rate).as_("ceil_rate"),
    Sqrt(Item.standard_rate).as_("sqrt_rate")
)
```

### 1.3. String Functions
- `Upper(term, alias=None)` - UPPER() - **Chuyển chuỗi thành chữ HOA**
- `Lower(term, alias=None)` - LOWER() - **Chuyển chuỗi thành chữ thường**
- `Length(term, alias=None)` - LENGTH() - **Đếm số ký tự** trong chuỗi
- `Trim(term, alias=None)` - TRIM() - **Xóa khoảng trắng** ở đầu và cuối chuỗi
- `LTrim(term, alias=None)` - LTRIM() - **Xóa khoảng trắng** ở đầu chuỗi (bên trái)
- `RTrim(term, alias=None)` - RTRIM() - **Xóa khoảng trắng** ở cuối chuỗi (bên phải)
- `Replace(term, old, new, alias=None)` - REPLACE() - **Thay thế** tất cả chuỗi `old` bằng `new` trong `term`
- `Substring(term, start, length=None, alias=None)` - SUBSTRING() - **Cắt chuỗi con** từ vị trí `start` với độ dài `length`
- `Concat(*terms, alias=None)` - CONCAT() - **Nối nhiều chuỗi** lại với nhau
- `Coalesce(*terms, alias=None)` - COALESCE() - **Trả về giá trị đầu tiên khác NULL** trong danh sách
- `IfNull(term, default, alias=None)` - IFNULL() - **Nếu `term` là NULL thì trả về `default`, ngược lại trả về `term`**
- `NullIf(term1, term2, alias=None)` - NULLIF() - **Nếu `term1` = `term2` thì trả về NULL, ngược lại trả về `term1`**

**Ví dụ:**
```python
from frappe.query_builder import DocType, Upper, Lower, Length, Concat, Coalesce, IfNull

Customer = DocType("Customer")
q = frappe.qb.from_(Customer).select(
    Customer.customer_name,
    Upper(Customer.customer_name).as_("upper_name"),
    Lower(Customer.customer_name).as_("lower_name"),
    Length(Customer.customer_name).as_("name_length"),
    Concat(Customer.first_name, " ", Customer.last_name).as_("full_name"),
    Coalesce(Customer.email_id, Customer.mobile_no, "N/A").as_("contact"),
    IfNull(Customer.status, "Unknown").as_("status")
)
```

### 1.4. Date/Time Functions
- `Now(alias=None)` - NOW() - **Trả về ngày giờ hiện tại** (datetime)
- `CurrentDate(alias=None)` - CURRENT_DATE() - **Trả về ngày hiện tại** (chỉ date, không có time)
- `CurrentTime(alias=None)` - CURRENT_TIME() - **Trả về giờ hiện tại** (chỉ time, không có date)
- `CurrentTimestamp(alias=None)` - CURRENT_TIMESTAMP() - **Trả về timestamp hiện tại** (giống NOW())
- `Date(term, alias=None)` - DATE() - **Trích xuất phần date** từ datetime
- `Time(term, alias=None)` - TIME() - **Trích xuất phần time** từ datetime
- `DateAdd(term, interval, alias=None)` - DATE_ADD() - **Cộng thêm** một khoảng thời gian vào date/datetime
- `DateSub(term, interval, alias=None)` - DATE_SUB() - **Trừ đi** một khoảng thời gian từ date/datetime
- `DateDiff(term1, term2, alias=None)` - DATEDIFF() - **Tính số ngày chênh lệch** giữa 2 date (term1 - term2)
- `Year(term, alias=None)` - YEAR() - **Trích xuất năm** từ date/datetime
- `Month(term, alias=None)` - MONTH() - **Trích xuất tháng** (1-12) từ date/datetime
- `Day(term, alias=None)` - DAY() - **Trích xuất ngày** (1-31) từ date/datetime
- `Hour(term, alias=None)` - HOUR() - **Trích xuất giờ** (0-23) từ time/datetime
- `Minute(term, alias=None)` - MINUTE() - **Trích xuất phút** (0-59) từ time/datetime
- `Second(term, alias=None)` - SECOND() - **Trích xuất giây** (0-59) từ time/datetime
- `Week(term, alias=None)` - WEEK() - **Trích xuất số tuần** trong năm (1-53)
- `Weekday(term, alias=None)` - WEEKDAY() - **Trả về thứ trong tuần** (0=Monday, 6=Sunday)
- `DayOfWeek(term, alias=None)` - DAYOFWEEK() - **Trả về thứ trong tuần** (1=Sunday, 7=Saturday)
- `DayOfYear(term, alias=None)` - DAYOFYEAR() - **Trả về ngày thứ mấy trong năm** (1-366)
- `Extract(part, term, alias=None)` - EXTRACT() - **Trích xuất phần cụ thể** (year, month, day, hour, ...) từ date/datetime
- `Format(term, format_string, alias=None)` - FORMAT() - **Format số** theo định dạng (ví dụ: 1000.5 → "1,000.50")

**Ví dụ:**
```python
from frappe.query_builder import DocType, Now, Year, Month, Day, DateDiff, Extract

Order = DocType("Sales Order")
q = frappe.qb.from_(Order).select(
    Order.posting_date,
    Year(Order.posting_date).as_("year"),
    Month(Order.posting_date).as_("month"),
    Day(Order.posting_date).as_("day"),
    DateDiff(Now(), Order.posting_date).as_("days_ago"),
    Extract("YEAR", Order.posting_date).as_("extracted_year")
)
```

### 1.5. Type Conversion Functions
- `Cast(term, as_type, alias=None)` - CAST() - **Chuyển đổi kiểu dữ liệu** của `term` sang `as_type` (ví dụ: VARCHAR, INT, DATE)
- `Convert(term, as_type, alias=None)` - CONVERT() - **Chuyển đổi kiểu dữ liệu** (tương tự CAST, nhưng syntax khác một chút)

**Ví dụ:**
```python
from frappe.query_builder import DocType, Cast

Item = DocType("Item")
q = frappe.qb.from_(Item).select(
    Item.item_code,
    Cast(Item.standard_rate, "VARCHAR(50)").as_("rate_str")
)
```

### 1.6. Conditional Functions
- `Case()` - CASE WHEN statement - **Câu lệnh điều kiện** (if-else trong SQL), trả về giá trị khác nhau tùy theo điều kiện

**Ví dụ:**
```python
from frappe.query_builder import DocType, Case

Order = DocType("Sales Order")
q = frappe.qb.from_(Order).select(
    Order.name,
    Case()
        .when(Order.grand_total > 1000, "High")
        .when(Order.grand_total > 500, "Medium")
        .else_("Low")
        .as_("order_category")
)
```

---

## 🔷 2. Custom Functions của Frappe

### 2.1. `Concat_ws(*terms, **kwargs)`
CONCAT với separator (CONCAT_WS) - **Nối nhiều chuỗi lại với nhau, cách nhau bởi separator** (ví dụ: "A", "B", "C" → "A-B-C" với separator là "-")

**Signature:**
```python
class Concat_ws(Function):
    def __init__(self, *terms, **kwargs):
        super().__init__("CONCAT_WS", *terms, **kwargs)
```

**Ví dụ:**
```python
from frappe.query_builder import DocType, Concat_ws

Customer = DocType("Customer")
q = frappe.qb.from_(Customer).select(
    Concat_ws(" ", Customer.first_name, Customer.last_name).as_("full_name")
)
# SQL: CONCAT_WS(' ', first_name, last_name) AS full_name
```

---

### 2.2. `Locate(needle, haystack, **kwargs)`
Tìm vị trí của `needle` trong `haystack` - **Trả về vị trí (index) đầu tiên** mà `needle` xuất hiện trong `haystack` (bắt đầu từ 1, trả về 0 nếu không tìm thấy)

**Tự động chọn function phù hợp theo database:**
- **MariaDB**: `LOCATE(needle, haystack)`
- **PostgreSQL**: `STRPOS(haystack, needle)`
- **SQLite**: `INSTR(haystack, needle)`

**Signature:**
```python
Locate = ImportMapper({
    db_type_is.MARIADB: Locate,      # LOCATE(needle, haystack)
    db_type_is.POSTGRES: Strpos,     # STRPOS(haystack, needle)
    db_type_is.SQLITE: Instr          # INSTR(haystack, needle)
})
```

**Ví dụ:**
```python
from frappe.query_builder import DocType, Locate

Customer = DocType("Customer")
q = frappe.qb.from_(Customer).select(
    Customer.customer_name,
    Locate("test", Customer.customer_name).as_("position")
)
# MariaDB: LOCATE('test', customer_name) AS position
# PostgreSQL: STRPOS(customer_name, 'test') AS position
# SQLite: INSTR(customer_name, 'test') AS position
```

---

### 2.3. `Ifnull(term, default, **kwargs)` / `IfNull(term, default, **kwargs)`
IFNULL function (backward compatibility alias) - **Nếu `term` là NULL thì trả về `default`, ngược lại trả về `term`** (giống Coalesce với 2 tham số)

**Signature:**
```python
Ifnull = IfNull  # Alias cho backward compatibility
```

**Ví dụ:**
```python
from frappe.query_builder import DocType, Ifnull, IfNull

Customer = DocType("Customer")
q = frappe.qb.from_(Customer).select(
    Ifnull(Customer.email_id, "N/A").as_("email"),
    IfNull(Customer.mobile_no, "N/A").as_("mobile")
)
```

---

### 2.4. `Timestamp(term, time=None, alias=None)`
TIMESTAMP function - **Kết hợp date và time thành datetime**, hoặc chuyển đổi string thành datetime

**Signature:**
```python
class Timestamp(Function):
    def __init__(self, term: str, time=None, alias=None):
        if time:
            super().__init__("TIMESTAMP", term, time, alias=alias)
        else:
            super().__init__("TIMESTAMP", term, alias=alias)
```

**Ví dụ:**
```python
from frappe.query_builder import DocType, Timestamp

Order = DocType("Sales Order")
q = frappe.qb.from_(Order).select(
    Timestamp(Order.posting_date, Order.posting_time).as_("datetime")
)
# SQL: TIMESTAMP(posting_date, posting_time) AS datetime
```

---

### 2.5. `Round(term, decimal=0, **kwargs)`
ROUND function với số chữ số thập phân - **Làm tròn số** đến số chữ số thập phân chỉ định (ví dụ: Round(3.14159, 2) → 3.14)

**Signature:**
```python
class Round(Function):
    def __init__(self, term, decimal=0, **kwargs):
        super().__init__("ROUND", term, decimal, **kwargs)
```

**Ví dụ:**
```python
from frappe.query_builder import DocType, Round

Item = DocType("Item")
q = frappe.qb.from_(Item).select(
    Item.standard_rate,
    Round(Item.standard_rate, 2).as_("rounded_rate")
)
# SQL: ROUND(standard_rate, 2) AS rounded_rate
```

---

### 2.6. `Truncate(term, decimal, **kwargs)`
TRUNCATE function - **Cắt bỏ** các chữ số thập phân sau vị trí `decimal` (không làm tròn, chỉ cắt bỏ) (ví dụ: Truncate(3.14159, 2) → 3.14)

**Signature:**
```python
class Truncate(Function):
    def __init__(self, term, decimal, **kwargs):
        super().__init__("TRUNCATE", term, decimal, **kwargs)
```

**Ví dụ:**
```python
from frappe.query_builder import DocType, Truncate

Item = DocType("Item")
q = frappe.qb.from_(Item).select(
    Item.standard_rate,
    Truncate(Item.standard_rate, 0).as_("truncated_rate")
)
# SQL: TRUNCATE(standard_rate, 0) AS truncated_rate
```

---

### 2.7. `GroupConcat(column, alias=None)`
GROUP_CONCAT function (MariaDB) hoặc STRING_AGG (PostgreSQL) - **Nối tất cả giá trị** của một cột trong nhóm thành một chuỗi, cách nhau bởi dấu phẩy (dùng trong GROUP BY)

**Tự động chọn function phù hợp:**
- **MariaDB**: `GROUP_CONCAT(column)`
- **PostgreSQL**: `STRING_AGG(column, ',')`

**Signature:**
```python
GroupConcat = ImportMapper({
    db_type_is.MARIADB: GROUP_CONCAT,    # GROUP_CONCAT(column)
    db_type_is.POSTGRES: STRING_AGG      # STRING_AGG(column, ',')
})
```

**Ví dụ:**
```python
from frappe.query_builder import DocType, GroupConcat

Order = DocType("Sales Order")
Item = DocType("Sales Order Item")

q = (
    frappe.qb.from_(Order)
    .left_join(Item).on(Order.name == Item.parent)
    .select(
        Order.name,
        GroupConcat(Item.item_code).as_("items")
    )
    .groupby(Order.name)
)
# MariaDB: GROUP_CONCAT(item_code) AS items
# PostgreSQL: STRING_AGG(item_code, ',') AS items
```

---

### 2.8. `Match(column, *args, **kwargs)`
MATCH AGAINST function (MariaDB) hoặc TO_TSVECTOR (PostgreSQL) - **Tìm kiếm full-text** trong cột (tìm kiếm nhanh trong văn bản dài, cần index full-text)

**Tự động chọn function phù hợp:**
- **MariaDB**: `MATCH(column) AGAINST(...)`
- **PostgreSQL**: `TO_TSVECTOR(column) @@ PLAINTO_TSQUERY(...)`

**Signature:**
```python
Match = ImportMapper({
    db_type_is.MARIADB: MATCH,        # MATCH(column) AGAINST(...)
    db_type_is.POSTGRES: TO_TSVECTOR   # TO_TSVECTOR(column) @@ PLAINTO_TSQUERY(...)
})
```

**Ví dụ:**
```python
from frappe.query_builder import DocType, Match

Item = DocType("Item")
q = frappe.qb.from_(Item).select("*").where(
    Match(Item.description).Against("laptop")
)
# MariaDB: MATCH(description) AGAINST('+laptop*' IN BOOLEAN MODE)
# PostgreSQL: TO_TSVECTOR(description) @@ PLAINTO_TSQUERY('laptop')
```

---

### 2.9. `CombineDatetime(datepart, timepart, alias=None)`
Kết hợp date và time thành datetime - **Ghép 2 cột date và time** thành một giá trị datetime (ví dụ: "2024-01-01" + "10:30:00" → "2024-01-01 10:30:00")

**Tự động chọn function phù hợp:**
- **MariaDB**: `TIMESTAMP(date, time)`
- **PostgreSQL**: `CAST(date AS date) + CAST(time AS time)`

**Signature:**
```python
CombineDatetime = ImportMapper({
    db_type_is.MARIADB: CustomFunction("TIMESTAMP", ["date", "time"]),
    db_type_is.POSTGRES: _PostgresTimestamp  # date + time
})
```

**Ví dụ:**
```python
from frappe.query_builder import DocType, CombineDatetime

Order = DocType("Sales Order")
q = frappe.qb.from_(Order).select(
    CombineDatetime(Order.posting_date, Order.posting_time).as_("datetime")
)
# MariaDB: TIMESTAMP(posting_date, posting_time) AS datetime
# PostgreSQL: (CAST(posting_date AS date) + CAST(posting_time AS time)) AS datetime
```

---

### 2.10. `DateFormat(date, format, alias=None)`
Format date theo format string - **Định dạng ngày tháng** theo pattern chỉ định (ví dụ: "%Y-%m-%d" → "2024-01-01", "%d/%m/%Y" → "01/01/2024")

**Tự động chọn function phù hợp:**
- **MariaDB**: `DATE_FORMAT(date, format)`
- **PostgreSQL**: `TO_CHAR(date, format)`

**Signature:**
```python
DateFormat = ImportMapper({
    db_type_is.MARIADB: CustomFunction("DATE_FORMAT", ["date", "format"]),
    db_type_is.POSTGRES: ToChar  # TO_CHAR(date, format)
})
```

**Ví dụ:**
```python
from frappe.query_builder import DocType, DateFormat

Order = DocType("Sales Order")
q = frappe.qb.from_(Order).select(
    DateFormat(Order.posting_date, "%Y-%m-%d").as_("formatted_date")
)
# MariaDB: DATE_FORMAT(posting_date, '%Y-%m-%d') AS formatted_date
# PostgreSQL: TO_CHAR(posting_date, 'YYYY-MM-DD') AS formatted_date
```

---

### 2.11. `YearWeek(term)`
YEARWEEK function - **Trả về năm và tuần** dưới dạng số (ví dụ: 202401 = năm 2024, tuần 1)

**Signature:**
```python
class YearWeek(Function):
    def __init__(self, term):
        super().__init__("YEARWEEK", term, 1)
```

**Ví dụ:**
```python
from frappe.query_builder import DocType, YearWeek

Order = DocType("Sales Order")
q = frappe.qb.from_(Order).select(
    YearWeek(Order.posting_date).as_("year_week")
)
# SQL: YEARWEEK(posting_date, 1) AS year_week
```

---

### 2.12. `UnixTimestamp(field, alias=None)`
UNIX_TIMESTAMP function - **Chuyển đổi date/datetime thành Unix timestamp** (số giây tính từ 1/1/1970 00:00:00 UTC)

**Tự động chọn function phù hợp:**
- **MariaDB**: `UNIX_TIMESTAMP(date)`
- **PostgreSQL**: `EXTRACT(epoch FROM date)`

**Signature:**
```python
UnixTimestamp = ImportMapper({
    db_type_is.MARIADB: CustomFunction("unix_timestamp", ["date"]),
    db_type_is.POSTGRES: _PostgresUnixTimestamp  # EXTRACT(epoch FROM date)
})
```

**Ví dụ:**
```python
from frappe.query_builder import DocType, UnixTimestamp

Order = DocType("Sales Order")
q = frappe.qb.from_(Order).select(
    UnixTimestamp(Order.posting_date).as_("unix_timestamp")
)
# MariaDB: UNIX_TIMESTAMP(posting_date) AS unix_timestamp
# PostgreSQL: EXTRACT(epoch FROM posting_date) AS unix_timestamp
```

---

### 2.13. `Cast_(value, as_type, alias=None)`
CAST function với xử lý đặc biệt cho MariaDB (VARCHAR cast) - **Chuyển đổi kiểu dữ liệu** (giống Cast, nhưng có xử lý đặc biệt cho VARCHAR trong MariaDB vì MariaDB không hỗ trợ VARCHAR cast trực tiếp)

**Signature:**
```python
class Cast_(Function):
    def __init__(self, value, as_type, alias=None):
        if frappe.db.db_type == "mariadb" and (
            (hasattr(as_type, "get_sql") and as_type.get_sql().lower() == "varchar")
            or str(as_type).lower() == "varchar"
        ):
            # MariaDB không có VARCHAR cast, dùng CONCAT thay thế
            super().__init__("CONCAT", value, "", alias=alias)
        else:
            super().__init__("CAST", value, alias=alias)
            self.as_type = as_type
```

**Ví dụ:**
```python
from frappe.query_builder import DocType, Cast_

Item = DocType("Item")
q = frappe.qb.from_(Item).select(
    Cast_(Item.item_code, "VARCHAR(100)").as_("item_code_str")
)
# MariaDB: CONCAT(item_code, '') AS item_code_str (vì không có VARCHAR cast)
# PostgreSQL: CAST(item_code AS VARCHAR(100)) AS item_code_str
```

---

## 🔷 3. Aggregate Helper Functions

Các hàm helper để tính aggregate trên một DocType với filters:

### 3.1. `_max(dt, fieldname, filters=None, **kwargs)`
Tính MAX của một field trong DocType - **Helper function để tìm giá trị lớn nhất** của một field trong DocType với filters (trả về số, không phải query object)

**Signature:**
```python
def _max(dt, fieldname, filters=None, **kwargs):
    return _aggregate(Max, dt, fieldname, filters, **kwargs)
```

**Ví dụ:**
```python
from frappe.query_builder.functions import _max

max_rate = _max("Item", "standard_rate", filters={"disabled": 0})
# Tương đương với:
# frappe.qb.get_query("Item", filters={"disabled": 0}, 
#                     fields=[Max(PseudoColumn("standard_rate"))]).run()[0][0] or 0
```

---

### 3.2. `_min(dt, fieldname, filters=None, **kwargs)`
Tính MIN của một field trong DocType - **Helper function để tìm giá trị nhỏ nhất** của một field trong DocType với filters (trả về số, không phải query object)

**Signature:**
```python
def _min(dt, fieldname, filters=None, **kwargs):
    return _aggregate(Min, dt, fieldname, filters, **kwargs)
```

**Ví dụ:**
```python
from frappe.query_builder.functions import _min

min_rate = _min("Item", "standard_rate", filters={"disabled": 0})
```

---

### 3.3. `_avg(dt, fieldname, filters=None, **kwargs)`
Tính AVG của một field trong DocType - **Helper function để tính trung bình** của một field trong DocType với filters (trả về số, không phải query object)

**Signature:**
```python
def _avg(dt, fieldname, filters=None, **kwargs):
    return _aggregate(Avg, dt, fieldname, filters, **kwargs)
```

**Ví dụ:**
```python
from frappe.query_builder.functions import _avg

avg_rate = _avg("Item", "standard_rate", filters={"disabled": 0})
```

---

### 3.4. `_sum(dt, fieldname, filters=None, **kwargs)`
Tính SUM của một field trong DocType - **Helper function để tính tổng** của một field trong DocType với filters (trả về số, không phải query object)

**Signature:**
```python
def _sum(dt, fieldname, filters=None, **kwargs):
    return _aggregate(Sum, dt, fieldname, filters, **kwargs)
```

**Ví dụ:**
```python
from frappe.query_builder.functions import _sum

total_qty = _sum("Bin", "actual_qty", filters={"warehouse": "Main Warehouse"})
```

---

## 🔷 4. Enum: `SqlFunctions`

Enum chứa tên các SQL functions (dùng nội bộ).

**Signature:**
```python
class SqlFunctions(Enum):
    DayOfYear = "dayofyear"
    Extract = "extract"
    Locate = "locate"
    Count = "count"
    Sum = "sum"
    Avg = "avg"
    Max = "max"
    Min = "min"
    Abs = "abs"
    Timestamp = "timestamp"
    IfNull = "ifnull"
```

**Không cần sử dụng trực tiếp**, chỉ dùng nội bộ trong Frappe.

---

## 🔷 5. Internal Classes (không nên dùng trực tiếp)

### 5.1. `_PostgresTimestamp`
Internal class để xử lý TIMESTAMP cho PostgreSQL (dùng trong `CombineDatetime`).

### 5.2. `_PostgresUnixTimestamp`
Internal class để xử lý UNIX_TIMESTAMP cho PostgreSQL (dùng trong `UnixTimestamp`).

### 5.3. `_aggregate(function, dt, fieldname, filters, **kwargs)`
Internal function để tính aggregate (dùng trong `_max`, `_min`, `_avg`, `_sum`).

---

## 📝 Lưu ý quan trọng

1. **Database-specific functions**: Một số functions (`Locate`, `GroupConcat`, `Match`, `CombineDatetime`, `DateFormat`, `UnixTimestamp`) tự động chọn implementation phù hợp với database hiện tại.

2. **ImportMapper**: Frappe sử dụng `ImportMapper` để tự động chọn function phù hợp với database type (MariaDB, PostgreSQL, SQLite).

3. **Backward compatibility**: `Ifnull` là alias của `IfNull` để tương thích với code cũ.

4. **VARCHAR cast trong MariaDB**: `Cast_()` có xử lý đặc biệt cho MariaDB vì MariaDB không hỗ trợ VARCHAR cast trực tiếp.

5. **Aggregate helpers**: Các hàm `_max()`, `_min()`, `_avg()`, `_sum()` là helper functions tiện lợi, nhưng có thể dùng trực tiếp `Max()`, `Min()`, `Avg()`, `Sum()` trong query builder.

---

## 🔗 Ví dụ tổng hợp

```python
import frappe
from frappe.query_builder import DocType
from frappe.query_builder.functions import (
    Count, Sum, Avg, Max, Min,
    Concat_ws, Locate, Round, Truncate,
    GroupConcat, Match, CombineDatetime,
    DateFormat, UnixTimestamp, Cast_
)

# Aggregate functions
Order = DocType("Sales Order")
q1 = frappe.qb.from_(Order).select(
    Count(Order.name).as_("total"),
    Sum(Order.grand_total).as_("total_amount"),
    Avg(Order.grand_total).as_("avg_amount"),
    Max(Order.grand_total).as_("max_amount"),
    Min(Order.grand_total).as_("min_amount")
)

# String functions
Customer = DocType("Customer")
q2 = frappe.qb.from_(Customer).select(
    Concat_ws(" ", Customer.first_name, Customer.last_name).as_("full_name"),
    Locate("test", Customer.customer_name).as_("position")
)

# Math functions
Item = DocType("Item")
q3 = frappe.qb.from_(Item).select(
    Round(Item.standard_rate, 2).as_("rounded"),
    Truncate(Item.standard_rate, 0).as_("truncated")
)

# Date functions
q4 = frappe.qb.from_(Order).select(
    CombineDatetime(Order.posting_date, Order.posting_time).as_("datetime"),
    DateFormat(Order.posting_date, "%Y-%m-%d").as_("formatted_date"),
    UnixTimestamp(Order.posting_date).as_("unix_ts")
)

# Type conversion
q5 = frappe.qb.from_(Item).select(
    Cast_(Item.item_code, "VARCHAR(100)").as_("item_code_str")
)

# Aggregate helpers
max_rate = _max("Item", "standard_rate", filters={"disabled": 0})
min_rate = _min("Item", "standard_rate", filters={"disabled": 0})
avg_rate = _avg("Item", "standard_rate", filters={"disabled": 0})
total_qty = _sum("Bin", "actual_qty", filters={"warehouse": "Main Warehouse"})
```

---

## 🔗 Tài liệu tham khảo

- File source: `apps/frappe/frappe/query_builder/functions.py`
- [PyPika Functions Documentation](https://pypika.readthedocs.io/en/latest/2_tutorial.html#functions)
- [Frappe Query Builder Guide](./FRAPPE_QUERY_BUILDER.md)
