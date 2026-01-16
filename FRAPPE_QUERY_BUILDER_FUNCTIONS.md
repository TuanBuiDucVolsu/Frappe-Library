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
- `Count(term, alias=None)` - COUNT()
- `Sum(term, alias=None)` - SUM()
- `Avg(term, alias=None)` - AVG()
- `Max(term, alias=None)` - MAX()
- `Min(term, alias=None)` - MIN()
- `StdDev(term, alias=None)` - STDDEV()
- `Variance(term, alias=None)` - VARIANCE()

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
- `Abs(term, alias=None)` - ABS()
- `Round(term, decimals=0, alias=None)` - ROUND()
- `Floor(term, alias=None)` - FLOOR()
- `Ceil(term, alias=None)` - CEIL()
- `Sqrt(term, alias=None)` - SQRT()
- `Power(term, exponent, alias=None)` - POWER()
- `Mod(term, divisor, alias=None)` - MOD()
- `Sign(term, alias=None)` - SIGN()

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
- `Upper(term, alias=None)` - UPPER()
- `Lower(term, alias=None)` - LOWER()
- `Length(term, alias=None)` - LENGTH()
- `Trim(term, alias=None)` - TRIM()
- `LTrim(term, alias=None)` - LTRIM()
- `RTrim(term, alias=None)` - RTRIM()
- `Replace(term, old, new, alias=None)` - REPLACE()
- `Substring(term, start, length=None, alias=None)` - SUBSTRING()
- `Concat(*terms, alias=None)` - CONCAT()
- `Coalesce(*terms, alias=None)` - COALESCE()
- `IfNull(term, default, alias=None)` - IFNULL()
- `NullIf(term1, term2, alias=None)` - NULLIF()

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
- `Now(alias=None)` - NOW()
- `CurrentDate(alias=None)` - CURRENT_DATE()
- `CurrentTime(alias=None)` - CURRENT_TIME()
- `CurrentTimestamp(alias=None)` - CURRENT_TIMESTAMP()
- `Date(term, alias=None)` - DATE()
- `Time(term, alias=None)` - TIME()
- `DateAdd(term, interval, alias=None)` - DATE_ADD()
- `DateSub(term, interval, alias=None)` - DATE_SUB()
- `DateDiff(term1, term2, alias=None)` - DATEDIFF()
- `Year(term, alias=None)` - YEAR()
- `Month(term, alias=None)` - MONTH()
- `Day(term, alias=None)` - DAY()
- `Hour(term, alias=None)` - HOUR()
- `Minute(term, alias=None)` - MINUTE()
- `Second(term, alias=None)` - SECOND()
- `Week(term, alias=None)` - WEEK()
- `Weekday(term, alias=None)` - WEEKDAY()
- `DayOfWeek(term, alias=None)` - DAYOFWEEK()
- `DayOfYear(term, alias=None)` - DAYOFYEAR()
- `Extract(part, term, alias=None)` - EXTRACT()
- `Format(term, format_string, alias=None)` - FORMAT()

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
- `Cast(term, as_type, alias=None)` - CAST()
- `Convert(term, as_type, alias=None)` - CONVERT()

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
- `Case()` - CASE WHEN statement

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
CONCAT với separator (CONCAT_WS).

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
Tìm vị trí của `needle` trong `haystack`.

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
IFNULL function (backward compatibility alias).

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
TIMESTAMP function.

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
ROUND function với số chữ số thập phân.

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
TRUNCATE function.

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
GROUP_CONCAT function (MariaDB) hoặc STRING_AGG (PostgreSQL).

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
MATCH AGAINST function (MariaDB) hoặc TO_TSVECTOR (PostgreSQL).

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
Kết hợp date và time thành datetime.

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
Format date theo format string.

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
YEARWEEK function.

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
UNIX_TIMESTAMP function.

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
CAST function với xử lý đặc biệt cho MariaDB (VARCHAR cast).

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
Tính MAX của một field trong DocType.

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
Tính MIN của một field trong DocType.

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
Tính AVG của một field trong DocType.

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
Tính SUM của một field trong DocType.

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
