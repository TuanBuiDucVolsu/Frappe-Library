# Frappe Query Builder - Tất cả các thành phần

File này liệt kê **tất cả các thành phần** có thể import từ `frappe.query_builder` trong Frappe v16.

---

## 📦 Import cơ bản

```python
from frappe.query_builder import *
# hoặc
import frappe.query_builder as qb
```

---

## 🔷 1. Từ `pypika` (được re-export)

Frappe re-export **tất cả** các thành phần từ `pypika`, bao gồm:

### 1.1. Query Builders (Dialects)
- `MySQLQuery` - Query builder cho MySQL/MariaDB
- `PostgreSQLQuery` - Query builder cho PostgreSQL
- `SQLLiteQuery` - Query builder cho SQLite
- `MSSQLQuery` - Query builder cho SQL Server
- `OracleQuery` - Query builder cho Oracle
- `ClickHouseQuery` - Query builder cho ClickHouse
- `RedshiftQuery` - Query builder cho Amazon Redshift
- `VerticaQuery` - Query builder cho Vertica

### 1.2. Core Classes
- `Query` - Class chính để build queries
- `Table` - Đại diện cho một bảng trong database
- `Schema` - Đại diện cho schema/database
- `Column` - Đại diện cho một cột
- `Field` - Đại diện cho một field/column trong query
- `Database` - Đại diện cho database connection
- `AliasedQuery` - Query có alias

### 1.3. Terms (Các thành phần của query)
- `Array` - Mảng trong SQL
- `Bracket` - Dấu ngoặc
- `Case` - CASE WHEN statement
- `Criterion` - Điều kiện WHERE
- `EmptyCriterion` - Điều kiện rỗng
- `Index` - Index trong SQL
- `Interval` - Khoảng thời gian
- `JSON` - JSON value
- `Not` - NOT operator
- `NullValue` - NULL value
- `SystemTimeValue` - System time value
- `Parameter` - SQL parameter
- `QmarkParameter` - `?` parameter
- `NumericParameter` - `:1` parameter
- `NamedParameter` - `:name` parameter
- `FormatParameter` - `%s` parameter
- `PyformatParameter` - `%(name)s` parameter
- `Rollup` - ROLLUP trong GROUP BY
- `Tuple` - Tuple trong SQL
- `CustomFunction` - Custom SQL function

### 1.4. Enums
- `DatePart` - Enum cho các phần của date (year, month, day, ...)
- `JoinType` - Enum cho các loại JOIN (inner, left, right, full)
- `Order` - Enum cho ORDER BY (asc, desc)

### 1.5. Functions (SQL Functions)
- `Count` - COUNT()
- `Sum` - SUM()
- `Avg` - AVG()
- `Max` - MAX()
- `Min` - MIN()
- `Abs` - ABS()
- `Round` - ROUND()
- `Floor` - FLOOR()
- `Ceil` - CEIL()
- `Sqrt` - SQRT()
- `Power` - POWER()
- `Mod` - MOD()
- `Upper` - UPPER()
- `Lower` - LOWER()
- `Length` - LENGTH()
- `Trim` - TRIM()
- `LTrim` - LTRIM()
- `RTrim` - RTRIM()
- `Replace` - REPLACE()
- `Substring` - SUBSTRING()
- `Concat` - CONCAT()
- `Coalesce` - COALESCE()
- `IfNull` - IFNULL()
- `NullIf` - NULLIF()
- `Cast` - CAST()
- `Extract` - EXTRACT()
- `Now` - NOW()
- `CurrentDate` - CURRENT_DATE()
- `CurrentTime` - CURRENT_TIME()
- `CurrentTimestamp` - CURRENT_TIMESTAMP()
- `Date` - DATE()
- `Time` - TIME()
- `Timestamp` - TIMESTAMP()
- `DateAdd` - DATE_ADD()
- `DateSub` - DATE_SUB()
- `DateDiff` - DATEDIFF()
- `Year` - YEAR()
- `Month` - MONTH()
- `Day` - DAY()
- `Hour` - HOUR()
- `Minute` - MINUTE()
- `Second` - SECOND()
- `Week` - WEEK()
- `Weekday` - WEEKDAY()
- `DayOfWeek` - DAYOFWEEK()
- `DayOfYear` - DAYOFYEAR()
- `Quarter` - QUARTER()
- `Format` - FORMAT()
- `GroupConcat` - GROUP_CONCAT() (MySQL)
- `StringAgg` - STRING_AGG() (PostgreSQL)

### 1.6. Utilities
- `Tables` - Helper để tạo nhiều Table objects
- `Columns` - Helper để tạo nhiều Column objects
- `make_tables` - Alias của `Tables`
- `make_columns` - Alias của `Columns`

### 1.7. Exceptions
- `CaseException`
- `GroupingException`
- `JoinException`
- `QueryException`
- `RollupException`
- `SetOperationException`
- `FunctionException`

---

## 🔷 2. Từ `frappe.query_builder.utils`

### 2.1. `DocType(doctype_name, *args, **kwargs)`
Tạo Table object cho một DocType trong Frappe.

**Ví dụ:**
```python
from frappe.query_builder import DocType

Customer = DocType("Customer")
q = frappe.qb.from_(Customer).select("*")
```

**Lưu ý:** Tự động thêm prefix `tab` vào tên bảng (ví dụ: `Customer` → `tabCustomer`).

### 2.2. `Table(table_name, *args, **kwargs)`
Tạo Table object cho một bảng thông thường (không phải DocType).

**Ví dụ:**
```python
from frappe.query_builder import Table

MyTable = Table("my_table")
q = frappe.qb.from_(MyTable).select("*")
```

### 2.3. `Column`
Re-export từ `pypika.queries.Column`.

### 2.4. `get_query(doctype, filters=None, fields=None, ...)`
Tạo QueryBuilder object cho một DocType với filters và fields.

**Ví dụ:**
```python
from frappe.query_builder import get_query

q = get_query("Customer", filters={"status": "Active"})
results = q.run(as_dict=True)
```

### 2.5. `get_query_builder(type_of_db)`
Lấy Query Builder class phù hợp với loại database.

**Ví dụ:**
```python
from frappe.query_builder import get_query_builder

MariaDB = get_query_builder("mariadb")
Postgres = get_query_builder("postgres")
SQLite = get_query_builder("sqlite")
```

**Giá trị hợp lệ:** `"mariadb"`, `"postgres"`, `"sqlite"`

---

## 🔷 3. Từ `frappe.query_builder.terms`

### 3.1. `ParameterizedFunction`
Class được patch vào `pypika.terms.Function` để hỗ trợ parameterized queries.

**Không cần import trực tiếp**, được sử dụng tự động khi dùng các SQL functions.

### 3.2. `ParameterizedValueWrapper`
Class được patch vào `pypika.terms.ValueWrapper` để hỗ trợ parameterized queries.

**Không cần import trực tiếp**, được sử dụng tự động khi dùng values trong queries.

### 3.3. `SubQuery(subq, alias=None)` / `subqry(subq, alias=None)`
Tạo subquery trong WHERE clause.

**Ví dụ:**
```python
from frappe.query_builder import DocType, subqry

Customer = DocType("Customer")
Order = DocType("Sales Order")

subq = frappe.qb.from_(Order).select("customer").where(Order.status == "Completed")
q = frappe.qb.from_(Customer).select("*").where(
    Customer.name.isin(subqry(subq))
)
```

### 3.4. `NamedParameterWrapper`
Utility class để quản lý named parameters trong queries (dùng nội bộ).

---

## 🔷 4. Từ `frappe.query_builder.functions`

### 4.1. `Locate(needle, haystack, **kwargs)`
Tìm vị trí của `needle` trong `haystack`.

**Tự động chọn function phù hợp theo database:**
- MariaDB: `LOCATE(needle, haystack)`
- PostgreSQL: `STRPOS(haystack, needle)`
- SQLite: `INSTR(haystack, needle)`

**Ví dụ:**
```python
from frappe.query_builder import DocType, Locate

Customer = DocType("Customer")
q = frappe.qb.from_(Customer).select(
    Locate("test", Customer.customer_name).as_("position")
)
```

### 4.2. `Concat_ws(*terms, **kwargs)`
CONCAT với separator (CONCAT_WS).

**Ví dụ:**
```python
from frappe.query_builder import DocType, Concat_ws

Customer = DocType("Customer")
q = frappe.qb.from_(Customer).select(
    Concat_ws(" ", Customer.first_name, Customer.last_name).as_("full_name")
)
```

### 4.3. `Ifnull(term, default, **kwargs)` / `IfNull(term, default, **kwargs)`
IFNULL function (backward compatibility alias).

### 4.4. `Timestamp(term, time=None, alias=None)`
TIMESTAMP function.

**Ví dụ:**
```python
from frappe.query_builder import DocType, Timestamp

Order = DocType("Sales Order")
q = frappe.qb.from_(Order).select(
    Timestamp(Order.posting_date, Order.posting_time).as_("datetime")
)
```

### 4.5. `Round(term, decimal=0, **kwargs)`
ROUND function với số chữ số thập phân.

**Ví dụ:**
```python
from frappe.query_builder import DocType, Round

Item = DocType("Item")
q = frappe.qb.from_(Item).select(
    Round(Item.standard_rate, 2).as_("rounded_rate")
)
```

### 4.6. `Truncate(term, decimal, **kwargs)`
TRUNCATE function.

**Ví dụ:**
```python
from frappe.query_builder import DocType, Truncate

Item = DocType("Item")
q = frappe.qb.from_(Item).select(
    Truncate(Item.standard_rate, 0).as_("truncated_rate")
)
```

### 4.7. `GroupConcat(column, alias=None)`
GROUP_CONCAT function (MariaDB) hoặc STRING_AGG (PostgreSQL).

**Tự động chọn function phù hợp:**
- MariaDB: `GROUP_CONCAT(column)`
- PostgreSQL: `STRING_AGG(column, ',')`

**Ví dụ:**
```python
from frappe.query_builder import DocType, GroupConcat

Order = DocType("Sales Order")
Item = DocType("Sales Order Item")

q = (
    frappe.qb.from_(Order)
    .left_join(Item).on(Order.name == Item.parent)
    .select(Order.name, GroupConcat(Item.item_code).as_("items"))
    .groupby(Order.name)
)
```

### 4.8. `Match(column, *args, **kwargs)`
MATCH AGAINST function (MariaDB) hoặc TO_TSVECTOR (PostgreSQL).

**Tự động chọn function phù hợp:**
- MariaDB: `MATCH(column) AGAINST(...)`
- PostgreSQL: `TO_TSVECTOR(column) @@ PLAINTO_TSQUERY(...)`

**Ví dụ:**
```python
from frappe.query_builder import DocType, Match

Item = DocType("Item")
q = frappe.qb.from_(Item).select("*").where(
    Match(Item.description).Against("laptop")
)
```

### 4.9. `CombineDatetime(datepart, timepart, alias=None)`
Kết hợp date và time thành datetime.

**Tự động chọn function phù hợp:**
- MariaDB: `TIMESTAMP(date, time)`
- PostgreSQL: `CAST(date AS date) + CAST(time AS time)`

**Ví dụ:**
```python
from frappe.query_builder import DocType, CombineDatetime

Order = DocType("Sales Order")
q = frappe.qb.from_(Order).select(
    CombineDatetime(Order.posting_date, Order.posting_time).as_("datetime")
)
```

### 4.10. `DateFormat(date, format, alias=None)`
Format date theo format string.

**Tự động chọn function phù hợp:**
- MariaDB: `DATE_FORMAT(date, format)`
- PostgreSQL: `TO_CHAR(date, format)`

**Ví dụ:**
```python
from frappe.query_builder import DocType, DateFormat

Order = DocType("Sales Order")
q = frappe.qb.from_(Order).select(
    DateFormat(Order.posting_date, "%Y-%m-%d").as_("formatted_date")
)
```

### 4.11. `YearWeek(term)`
YEARWEEK function.

**Ví dụ:**
```python
from frappe.query_builder import DocType, YearWeek

Order = DocType("Sales Order")
q = frappe.qb.from_(Order).select(
    YearWeek(Order.posting_date).as_("year_week")
)
```

### 4.12. `UnixTimestamp(field, alias=None)`
UNIX_TIMESTAMP function.

**Tự động chọn function phù hợp:**
- MariaDB: `UNIX_TIMESTAMP(date)`
- PostgreSQL: `EXTRACT(epoch FROM date)`

**Ví dụ:**
```python
from frappe.query_builder import DocType, UnixTimestamp

Order = DocType("Sales Order")
q = frappe.qb.from_(Order).select(
    UnixTimestamp(Order.posting_date).as_("unix_timestamp")
)
```

### 4.13. `Cast_(value, as_type, alias=None)`
CAST function với xử lý đặc biệt cho MariaDB (VARCHAR cast).

**Ví dụ:**
```python
from frappe.query_builder import DocType, Cast_

Item = DocType("Item")
q = frappe.qb.from_(Item).select(
    Cast_(Item.item_code, "VARCHAR(100)").as_("item_code_str")
)
```

### 4.14. Aggregate Functions (được patch vào Base)
- `_max(dt, fieldname, filters=None, **kwargs)` - MAX aggregate
- `_min(dt, fieldname, filters=None, **kwargs)` - MIN aggregate
- `_avg(dt, fieldname, filters=None, **kwargs)` - AVG aggregate
- `_sum(dt, fieldname, filters=None, **kwargs)` - SUM aggregate

**Ví dụ:**
```python
from frappe.query_builder import _max, _min, _avg, _sum

max_rate = _max("Item", "standard_rate", filters={"disabled": 0})
min_rate = _min("Item", "standard_rate", filters={"disabled": 0})
avg_rate = _avg("Item", "standard_rate", filters={"disabled": 0})
total_qty = _sum("Bin", "actual_qty", filters={"warehouse": "Main Warehouse"})
```

---

## 🔷 5. Từ `frappe.query_builder.custom`

### 5.1. `GROUP_CONCAT(column, alias=None)`
GROUP_CONCAT function cho MariaDB.

**Ví dụ:**
```python
from frappe.query_builder import DocType, GROUP_CONCAT

Order = DocType("Sales Order")
Item = DocType("Sales Order Item")

q = (
    frappe.qb.from_(Order)
    .left_join(Item).on(Order.name == Item.parent)
    .select(Order.name, GROUP_CONCAT(Item.item_code).as_("items"))
    .groupby(Order.name)
)
```

### 5.2. `STRING_AGG(column, separator=",", alias=None)`
STRING_AGG function cho PostgreSQL.

**Ví dụ:**
```python
from frappe.query_builder import DocType, STRING_AGG

Order = DocType("Sales Order")
Item = DocType("Sales Order Item")

q = (
    frappe.qb.from_(Order)
    .left_join(Item).on(Order.name == Item.parent)
    .select(Order.name, STRING_AGG(Item.item_code, ",").as_("items"))
    .groupby(Order.name)
)
```

### 5.3. `MATCH(column, *args, **kwargs)`
MATCH AGAINST function cho MariaDB.

**Ví dụ:**
```python
from frappe.query_builder import DocType, MATCH

Item = DocType("Item")
q = frappe.qb.from_(Item).select("*").where(
    MATCH(Item.description).Against("laptop")
)
```

### 5.4. `TO_TSVECTOR(column, *args, **kwargs)`
TO_TSVECTOR function cho PostgreSQL.

**Ví dụ:**
```python
from frappe.query_builder import DocType, TO_TSVECTOR

Item = DocType("Item")
q = frappe.qb.from_(Item).select("*").where(
    TO_TSVECTOR(Item.description).Against("laptop")
)
```

### 5.5. `Month(field, alias=None)`
MONTH function.

**Ví dụ:**
```python
from frappe.query_builder import DocType, Month

Order = DocType("Sales Order")
q = frappe.qb.from_(Order).select(
    Month(Order.posting_date).as_("month")
)
```

### 5.6. `MonthName(field, alias=None)`
MONTHNAME function.

**Ví dụ:**
```python
from frappe.query_builder import DocType, MonthName

Order = DocType("Sales Order")
q = frappe.qb.from_(Order).select(
    MonthName(Order.posting_date).as_("month_name")
)
```

### 5.7. `Quarter(field, alias=None)`
QUARTER function.

**Ví dụ:**
```python
from frappe.query_builder import DocType, Quarter

Order = DocType("Sales Order")
q = frappe.qb.from_(Order).select(
    Quarter(Order.posting_date).as_("quarter")
)
```

### 5.8. `ConstantColumn(value)`
Tạo một cột constant với giá trị cố định trong tất cả các rows.

**Ví dụ:**
```python
from frappe.query_builder import DocType, ConstantColumn

Customer = DocType("Customer")
q = frappe.qb.from_(Customer).select(
    Customer.name,
    ConstantColumn("Active").as_("status")
)
```

---

## 🔷 6. Từ `frappe.query_builder.builder`

### 6.1. `Base`
Base class cho tất cả các Query Builder classes.

**Attributes:**
- `terms` - Access đến pypika.terms
- `desc` - Alias cho `Order.desc`
- `asc` - Alias cho `Order.asc`
- `Schema` - Schema class
- `Table` - Table class

**Methods:**
- `functions(name, *args, **kwargs)` - Tạo custom function
- `DocType(table_name, *args, **kwargs)` - Tạo DocType table
- `into(table, *args, **kwargs)` - INSERT INTO
- `update(table, *args, **kwargs)` - UPDATE

### 6.2. `MariaDB`
Query Builder cho MariaDB/MySQL.

**Inherits from:** `Base`, `MySQLQuery`

**Methods:**
- `from_(table, *args, **kwargs)` - FROM clause (tự động convert string → DocType)
- `Field` - Field class

**Ví dụ:**
```python
from frappe.query_builder import DocType

Customer = DocType("Customer")
q = frappe.qb.from_(Customer).select("*")
```

### 6.3. `Postgres`
Query Builder cho PostgreSQL.

**Inherits from:** `Base`, `PostgreSQLQuery`

**Methods:**
- `from_(table, *args, **kwargs)` - FROM clause (tự động convert string → DocType)
- `Field(field_name, *args, **kwargs)` - Field class với field translation

**Ví dụ:**
```python
from frappe.query_builder import DocType

Customer = DocType("Customer")
q = frappe.qb.from_(Customer).select("*")
```

### 6.4. `SQLite`
Query Builder cho SQLite.

**Inherits from:** `Base`, `SQLLiteQuery`

**Methods:**
- `from_(table, *args, **kwargs)` - FROM clause (tự động convert string → DocType)
- `Field` - Field class

**Ví dụ:**
```python
from frappe.query_builder import DocType

Customer = DocType("Customer")
q = frappe.qb.from_(Customer).select("*")
```

---

## 🔷 7. Cách sử dụng `frappe.qb`

Frappe cung cấp global object `frappe.qb` để truy cập Query Builder:

```python
import frappe

# frappe.qb tự động chọn builder phù hợp với database hiện tại
Customer = frappe.qb.DocType("Customer")
Order = frappe.qb.DocType("Sales Order")

# SELECT query
q = (
    frappe.qb.from_(Customer)
    .select(Customer.name, Customer.customer_name)
    .where(Customer.status == "Active")
    .orderby(Customer.name)
)

# Chạy query
results = q.run(as_dict=True)

# INSERT query
frappe.qb.into("Customer").insert("CUST-001", "Test Customer").run()

# UPDATE query
frappe.qb.update("Customer").set("status", "Inactive").where(
    Customer.name == "CUST-001"
).run()

# DELETE query
frappe.qb.from_("Customer").delete().where(
    Customer.name == "CUST-001"
).run()
```

---

## 🔷 8. Ví dụ tổng hợp

### 8.1. SELECT với JOIN
```python
import frappe
from frappe.query_builder import DocType, Count, Sum

Customer = DocType("Customer")
Order = DocType("Sales Order")

q = (
    frappe.qb.from_(Customer)
    .left_join(Order).on(Customer.name == Order.customer)
    .select(
        Customer.name,
        Customer.customer_name,
        Count(Order.name).as_("order_count"),
        Sum(Order.grand_total).as_("total_amount")
    )
    .where(Customer.status == "Active")
    .groupby(Customer.name)
    .orderby(Customer.name)
)

results = q.run(as_dict=True)
```

### 8.2. SELECT với Subquery
```python
import frappe
from frappe.query_builder import DocType, subqry

Customer = DocType("Customer")
Order = DocType("Sales Order")

# Subquery: lấy danh sách customers có order
subq = (
    frappe.qb.from_(Order)
    .select(Order.customer)
    .where(Order.status == "Completed")
    .distinct()
)

# Main query: lấy thông tin customers từ subquery
q = (
    frappe.qb.from_(Customer)
    .select("*")
    .where(Customer.name.isin(subqry(subq)))
)

results = q.run(as_dict=True)
```

### 8.3. SELECT với Functions
```python
import frappe
from frappe.query_builder import DocType, Concat_ws, DateFormat, Year, Month

Customer = DocType("Customer")
Order = DocType("Sales Order")

q = (
    frappe.qb.from_(Order)
    .left_join(Customer).on(Order.customer == Customer.name)
    .select(
        Order.name,
        Concat_ws(" ", Customer.first_name, Customer.last_name).as_("full_name"),
        DateFormat(Order.posting_date, "%Y-%m-%d").as_("order_date"),
        Year(Order.posting_date).as_("year"),
        Month(Order.posting_date).as_("month"),
        Order.grand_total
    )
    .where(Order.status == "Submitted")
    .orderby(Order.posting_date, order=frappe.qb.Order.desc)
)

results = q.run(as_dict=True)
```

### 8.4. INSERT
```python
import frappe

Customer = frappe.qb.DocType("Customer")

frappe.qb.into(Customer).insert(
    "CUST-001",
    "Test Customer",
    "Active"
).run()
```

### 8.5. UPDATE
```python
import frappe

Customer = frappe.qb.DocType("Customer")

frappe.qb.update(Customer).set(
    Customer.status, "Inactive"
).where(
    Customer.name == "CUST-001"
).run()
```

### 8.6. DELETE
```python
import frappe

Customer = frappe.qb.DocType("Customer")

frappe.qb.from_(Customer).delete().where(
    Customer.name == "CUST-001"
).run()
```

---

## 📝 Lưu ý quan trọng

1. **Tự động parameterization**: Tất cả queries được tự động parameterize để tránh SQL injection.

2. **Database-specific functions**: Một số functions (như `Locate`, `GroupConcat`, `Match`) tự động chọn implementation phù hợp với database.

3. **DocType vs Table**: 
   - `DocType()` tự động thêm prefix `tab` vào tên bảng
   - `Table()` dùng trực tiếp tên bảng

4. **Query execution**: 
   - Gọi `.run()` để execute query
   - Có thể truyền `as_dict=True` để trả về dict thay vì tuple
   - Có thể truyền `as_list=True` để trả về list thay vì dict

5. **Permissions**: Query Builder **KHÔNG** tự động kiểm tra permissions. Cần kiểm tra permissions thủ công nếu cần.

6. **Safe execution**: Trong server scripts, chỉ SELECT queries được phép.

---

## 🔗 Tài liệu tham khảo

- [PyPika Documentation](https://pypika.readthedocs.io/)
- [Frappe Query Builder Docs](https://frappeframework.com/docs/user/en/api/query-builder)
- File source: `apps/frappe/frappe/query_builder/`
