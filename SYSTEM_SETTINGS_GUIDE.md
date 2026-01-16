# System Settings - Hướng dẫn đầy đủ

File này giải thích **tất cả các trường (fields) và chức năng của chúng** trong **System Settings** của Frappe Framework.

---

## 📋 Tổng quan

**System Settings** là một Single DocType (chỉ có 1 record duy nhất) dùng để cấu hình các thiết lập hệ thống cơ bản của Frappe Framework.

**Vị trí:** Setup > Settings > System Settings

---

## 🔷 1. Localization Section

### 1.1. Country (`country`)

**Field Type:** Link (Country)  
**Default:** None

**Chức năng:**
- Thiết lập quốc gia mặc định cho hệ thống
- Ảnh hưởng đến các thiết lập localization khác

**Ví dụ:**
- Country = "Vietnam"
- Ảnh hưởng đến date format, number format, currency, etc.

---

### 1.2. Language (`language`)

**Field Type:** Link (Language)  
**Default:** None  
**Required:** Yes

**Chức năng:**
- Thiết lập ngôn ngữ mặc định cho hệ thống
- Được lưu vào `frappe.defaults` và set làm default language

**Code:**
```python
# File: system_settings.py

def set_defaults(self):
    from frappe.translate import set_default_language
    
    if self.language:
        set_default_language(self.language)
```

**Ví dụ:**
- Language = "vi" (Vietnamese)
- Tất cả UI sẽ hiển thị bằng tiếng Việt

---

### 1.3. Time Zone (`time_zone`)

**Field Type:** Select  
**Default:** None  
**Required:** Yes  
**Read Only:** Yes

**Chức năng:**
- Thiết lập timezone mặc định cho hệ thống
- Timezone được set trong site_config.json và không thể thay đổi từ UI

**Ví dụ:**
- Time Zone = "Asia/Ho_Chi_Minh"
- Tất cả datetime sẽ được hiển thị theo timezone này

---

### 1.4. Currency (`currency`)

**Field Type:** Link (Currency)  
**Default:** None

**Chức năng:**
- Thiết lập currency (tiền tệ) mặc định cho hệ thống
- Được lưu vào `frappe.defaults`

**Ví dụ:**
- Currency = "VND" (Vietnamese Dong)
- Tất cả transactions sẽ sử dụng VND làm currency mặc định

---

### 1.5. Application Name (`app_name`)

**Field Type:** Data  
**Default:** "Frappe"  
**Hidden:** Yes

**Chức năng:**
- Tên ứng dụng hiển thị trên Login page
- Mặc định là "Frappe"

**Ví dụ:**
- `app_name` = "ERPNext"
- Login page sẽ hiển thị "ERPNext" thay vì "Frappe"

---

### 1.6. Enable Onboarding (`enable_onboarding`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, hiển thị onboarding wizard cho users mới
- Hướng dẫn users cách sử dụng hệ thống

**Ví dụ:**
- `enable_onboarding` = 1 → Hiển thị onboarding wizard
- `enable_onboarding` = 0 → Không hiển thị

---

### 1.7. Setup Complete (`setup_complete`)

**Field Type:** Check  
**Default:** 0  
**Hidden:** Yes  
**Read Only:** Yes

**Chức năng:**
- Đánh dấu hệ thống đã hoàn tất setup
- Được set tự động sau khi hoàn tất setup wizard

---

### 1.8. Disable Document Sharing (`disable_document_sharing`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **vô hiệu hóa** tính năng chia sẻ documents
- Users không thể share documents với nhau

**Ví dụ:**
- `disable_document_sharing` = 1 → Không thể share documents
- `disable_document_sharing` = 0 → Có thể share documents

---

## 🔷 2. Date and Number Format Section

### 2.1. Date Format (`date_format`)

**Field Type:** Select  
**Default:** None  
**Required:** Yes  
**Options:**
- `yyyy-mm-dd` - 2024-01-15
- `dd-mm-yyyy` - 15-01-2024
- `dd/mm/yyyy` - 15/01/2024
- `dd.mm.yyyy` - 15.01.2024
- `mm/dd/yyyy` - 01/15/2024
- `mm-dd-yyyy` - 01-15-2024

**Chức năng:**
- Xác định format hiển thị ngày tháng trong hệ thống
- Được lưu vào `frappe.defaults`

**Ví dụ:**
- `date_format` = "dd/mm/yyyy" → Ngày hiển thị: 15/01/2024
- `date_format` = "yyyy-mm-dd" → Ngày hiển thị: 2024-01-15

---

### 2.2. Time Format (`time_format`)

**Field Type:** Select  
**Default:** "HH:mm:ss"  
**Required:** Yes  
**Options:**
- `HH:mm:ss` - 14:30:45
- `HH:mm` - 14:30

**Chức năng:**
- Xác định format hiển thị thời gian trong hệ thống
- Được lưu vào `frappe.defaults`

**Ví dụ:**
- `time_format` = "HH:mm:ss" → Thời gian: 14:30:45
- `time_format` = "HH:mm" → Thời gian: 14:30

---

### 2.3. Number Format (`number_format`)

**Field Type:** Select  
**Default:** None  
**Required:** Yes  
**Options:**
- `#,###.##` - 1,234.56
- `#.###,##` - 1.234,56
- `# ###.##` - 1 234.56
- `# ###,##` - 1 234,56
- `#'###.##` - 1'234.56
- `#, ###.##` - 1, 234.56
- `#,##,###.##` - 12,34,567.89
- `#,###.###` - 1,234.567
- `#.###` - 1234.567
- `#,###` - 1,234

**Chức năng:**
- Xác định format hiển thị số trong hệ thống
- Được lưu vào `frappe.defaults`

**Ví dụ:**
- `number_format` = "#,###.##" → Số: 1,234.56
- `number_format` = "#.###,##" → Số: 1.234,56

---

### 2.4. Use Number Format from Currency (`use_number_format_from_currency`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, sử dụng number format từ Currency thay vì System Settings
- Mỗi currency có thể có number format riêng

**Ví dụ:**
- Currency "USD": Number Format = "#,###.##"
- Currency "EUR": Number Format = "#.###,##"
- `use_number_format_from_currency` = 1 → Sử dụng format từ currency
- `use_number_format_from_currency` = 0 → Sử dụng format từ System Settings

---

### 2.5. First Day of the Week (`first_day_of_the_week`)

**Field Type:** Select  
**Default:** "Sunday"  
**Options:**
- `Sunday`, `Monday`, `Tuesday`, `Wednesday`, `Thursday`, `Friday`, `Saturday`

**Chức năng:**
- Xác định ngày đầu tiên của tuần
- Ảnh hưởng đến calendar và weekly reports

**Ví dụ:**
- `first_day_of_the_week` = "Monday" → Tuần bắt đầu từ Thứ 2
- `first_day_of_the_week` = "Sunday" → Tuần bắt đầu từ Chủ nhật

---

### 2.6. Float Precision (`float_precision`)

**Field Type:** Select  
**Default:** None  
**Options:**
- `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9`

**Chức năng:**
- Xác định số chữ số thập phân mặc định cho Float fields
- Nếu không set, sử dụng precision mặc định

**Ví dụ:**
- `float_precision` = 2 → Float: 123.45
- `float_precision` = 4 → Float: 123.4567

---

### 2.7. Currency Precision (`currency_precision`)

**Field Type:** Select  
**Default:** None  
**Options:**
- `0`, `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9`

**Chức năng:**
- Xác định số chữ số thập phân cho Currency fields
- Nếu không set, sẽ phụ thuộc vào number format

**Ví dụ:**
- `currency_precision` = 2 → Currency: 1,234.56
- `currency_precision` = 0 → Currency: 1,235

---

### 2.8. Rounding Method (`rounding_method`)

**Field Type:** Select  
**Default:** "Banker's Rounding (legacy)"  
**Options:**
- `Banker's Rounding (legacy)` - Làm tròn theo chuẩn cũ
- `Banker's Rounding` - Làm tròn theo chuẩn Banker's (IEEE 754)
- `Commercial Rounding` - Làm tròn thương mại (0.5 → 1)

**Chức năng:**
- Xác định phương pháp làm tròn số trong hệ thống
- Ảnh hưởng đến tất cả các phép tính làm tròn

**Ví dụ:**
- Số: 1.5
- `Banker's Rounding` → 2 (nếu số trước là chẵn) hoặc 2 (nếu số trước là lẻ)
- `Commercial Rounding` → 2 (luôn làm tròn lên)

---

### 2.9. Show Absolute Datetime in Timeline (`show_absolute_datetime_in_timeline`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, hiển thị **absolute datetime** (ngày giờ đầy đủ) trong timeline
- Khi disabled, hiển thị relative time (ví dụ: "2 hours ago")

**Ví dụ:**
- `show_absolute_datetime_in_timeline` = 1 → Timeline: "2024-01-15 14:30:45"
- `show_absolute_datetime_in_timeline` = 0 → Timeline: "2 hours ago"

---

## 🔷 3. Permissions Section

### 3.1. Apply Strict User Permissions (`apply_strict_user_permissions`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **strict mode** cho User Permissions
- Nếu User Permission được định nghĩa cho một DocType, tất cả documents có link field = blank sẽ **không được hiển thị** cho user đó

**Ví dụ:**
- User Permission: User A chỉ thấy Customers của Company A
- Document: Customer có Company = blank
- `apply_strict_user_permissions` = 1 → User A **không thấy** Customer này
- `apply_strict_user_permissions` = 0 → User A **có thể thấy** Customer này

---

## 🔷 4. Security Tab (Login)

### 4.1. Security Section

#### 4.1.1. Session Expiry (idle timeout) (`session_expiry`)

**Field Type:** Data  
**Default:** "170:00"  
**Format:** "hh:mm"

**Chức năng:**
- Xác định thời gian **idle timeout** cho user sessions
- Nếu user không hoạt động trong khoảng thời gian này, session sẽ hết hạn và user sẽ bị logout

**Code:**
```python
# File: system_settings.py

if self.session_expiry:
    parts = self.session_expiry.split(":")
    if len(parts) != 2 or not (cint(parts[0]) or cint(parts[1])):
        frappe.throw(_("Session Expiry must be in format {0}").format("hh:mm"))
```

**Ví dụ:**
- `session_expiry` = "24:00" → User sẽ bị logout nếu không hoạt động trong 24 giờ
- `session_expiry` = "02:00" → User sẽ bị logout nếu không hoạt động trong 2 giờ

---

#### 4.1.2. Document Share Key Expiry (in Days) (`document_share_key_expiry`)

**Field Type:** Int  
**Default:** 30

**Chức năng:**
- Xác định số ngày mà document Web View link được share sẽ hết hạn
- Sau khi hết hạn, link sẽ không còn hoạt động

**Ví dụ:**
- `document_share_key_expiry` = 30 → Link share sẽ hết hạn sau 30 ngày
- `document_share_key_expiry` = 7 → Link share sẽ hết hạn sau 7 ngày

---

#### 4.1.3. Allow only one session per user (`deny_multiple_sessions`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **chỉ cho phép** 1 session per user
- Khi user login ở một nơi khác, session cũ sẽ bị logout
- **Lưu ý:** Multiple sessions vẫn được phép trên mobile devices

**Ví dụ:**
- `deny_multiple_sessions` = 1 → User chỉ có thể login ở 1 nơi
- `deny_multiple_sessions` = 0 → User có thể login ở nhiều nơi

---

#### 4.1.4. Disable Username/Password Login (`disable_user_pass_login`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **vô hiệu hóa** login bằng username/password
- Chỉ cho phép login qua Social Login, LDAP, hoặc Email Link
- **Phải** có ít nhất 1 phương thức login khác được enable trước khi disable

**Code:**
```python
# File: system_settings.py

def validate_user_pass_login(self):
    if not self.disable_user_pass_login:
        return
    
    social_login_enabled = frappe.db.exists("Social Login Key", {"enable_social_login": 1})
    ldap_enabled = frappe.db.get_single_value("LDAP Settings", "enabled")
    
    if not (social_login_enabled or ldap_enabled or self.login_with_email_link):
        frappe.throw("Please enable atleast one Social Login Key or LDAP or Login With Email Link...")
```

**Ví dụ:**
- `disable_user_pass_login` = 1 → Không thể login bằng username/password
- `disable_user_pass_login` = 0 → Có thể login bằng username/password

---

#### 4.1.5. Max signups allowed per hour (`max_signups_allowed_per_hour`)

**Field Type:** Int  
**Default:** 300  
**Non-negative:** Yes

**Chức năng:**
- Giới hạn số lượng user signups được phép trong 1 giờ
- Ngăn chặn spam và abuse

**Ví dụ:**
- `max_signups_allowed_per_hour` = 300 → Tối đa 300 signups/giờ
- `max_signups_allowed_per_hour` = 10 → Tối đa 10 signups/giờ

---

### 4.2. Login Methods Section

#### 4.2.1. Allow Login using Mobile Number (`allow_login_using_mobile_number`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, user có thể login bằng **Email hoặc Mobile Number**
- Khi disabled, chỉ có thể login bằng Email

**Ví dụ:**
- `allow_login_using_mobile_number` = 1 → Login bằng email hoặc mobile
- `allow_login_using_mobile_number` = 0 → Chỉ login bằng email

---

#### 4.2.2. Allow Login using User Name (`allow_login_using_user_name`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, user có thể login bằng **Email hoặc User Name**
- Khi disabled, chỉ có thể login bằng Email

**Ví dụ:**
- `allow_login_using_user_name` = 1 → Login bằng email hoặc username
- `allow_login_using_user_name` = 0 → Chỉ login bằng email

---

#### 4.2.3. Login with email link (`login_with_email_link`)

**Field Type:** Check  
**Default:** 1 (True)

**Chức năng:**
- Khi enabled, cho phép user login **không cần password**, sử dụng login link được gửi qua email
- User click vào link trong email để login

**Ví dụ:**
- `login_with_email_link` = 1 → User có thể login bằng email link
- `login_with_email_link` = 0 → Không thể login bằng email link

---

#### 4.2.4. Login with email link expiry (in minutes) (`login_with_email_link_expiry`)

**Field Type:** Int  
**Default:** 10

**Dependency:** Chỉ hiển thị khi `login_with_email_link` = 1

**Chức năng:**
- Xác định thời gian hết hạn của login link (tính bằng phút)
- Sau khi hết hạn, link sẽ không còn hoạt động

**Ví dụ:**
- `login_with_email_link_expiry` = 10 → Link hết hạn sau 10 phút
- `login_with_email_link_expiry` = 60 → Link hết hạn sau 60 phút

---

#### 4.2.5. Rate limit for email link login (`rate_limit_email_link_login`)

**Field Type:** Int  
**Default:** None

**Dependency:** Chỉ hiển thị khi `login_with_email_link` = 1

**Chức năng:**
- Giới hạn số lượng email link login requests trong một khoảng thời gian
- Có thể set giá trị cao nếu nhiều users login từ cùng một network

**Ví dụ:**
- `rate_limit_email_link_login` = 10 → Tối đa 10 requests/giờ
- `rate_limit_email_link_login` = 100 → Tối đa 100 requests/giờ

---

### 4.3. Brute Force Security Section

#### 4.3.1. Allow Consecutive Login Attempts (`allow_consecutive_login_attempts`)

**Field Type:** Int  
**Default:** 10

**Chức năng:**
- Xác định số lần login thất bại liên tiếp được phép trước khi bị block
- Ngăn chặn brute force attacks

**Ví dụ:**
- `allow_consecutive_login_attempts` = 10 → Cho phép 10 lần thử sai liên tiếp
- `allow_consecutive_login_attempts` = 5 → Cho phép 5 lần thử sai liên tiếp

---

#### 4.3.2. Allow Login After Fail (`allow_login_after_fail`)

**Field Type:** Int  
**Default:** 60  
**Unit:** Seconds

**Chức năng:**
- Xác định thời gian (tính bằng giây) phải chờ sau khi login thất bại trước khi có thể thử lại
- Ngăn chặn brute force attacks

**Ví dụ:**
- `allow_login_after_fail` = 60 → Phải chờ 60 giây sau khi login thất bại
- `allow_login_after_fail` = 300 → Phải chờ 300 giây (5 phút) sau khi login thất bại

---

### 4.4. Two Factor Authentication Section

#### 4.4.1. Enable Two Factor Auth (`enable_two_factor_auth`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **bật** Two Factor Authentication (2FA) cho tất cả users
- Users sẽ phải nhập OTP sau khi nhập password

**Code:**
```python
# File: system_settings.py

if self.has_value_changed("enable_two_factor_auth"):
    if self.enable_two_factor_auth:
        if self.two_factor_method == "SMS":
            if not frappe.db.get_single_value("SMS Settings", "sms_gateway_url"):
                frappe.throw("Please setup SMS before setting it as an authentication method...")
        toggle_two_factor_auth(True, roles=["All"])
    else:
        self.bypass_2fa_for_retricted_ip_users = 0
        self.bypass_restrict_ip_check_if_2fa_enabled = 0
```

**Ví dụ:**
- `enable_two_factor_auth` = 1 → Bật 2FA cho tất cả users
- `enable_two_factor_auth` = 0 → Tắt 2FA

---

#### 4.4.2. Bypass Two Factor Auth for users who login from restricted IP Address (`bypass_2fa_for_retricted_ip_users`)

**Field Type:** Check  
**Default:** 0 (False)

**Dependency:** Chỉ hiển thị khi `enable_two_factor_auth` = 1

**Chức năng:**
- Khi enabled, users login từ Restricted IP Address sẽ **không bị yêu cầu** Two Factor Auth
- Khi disabled, tất cả users đều phải qua 2FA

**Ví dụ:**
- User có Restricted IP = "192.168.1.100"
- `bypass_2fa_for_retricted_ip_users` = 1 → User login từ IP này không cần 2FA
- `bypass_2fa_for_retricted_ip_users` = 0 → User vẫn phải qua 2FA

---

#### 4.4.3. Bypass restricted IP Address check If Two Factor Auth Enabled (`bypass_restrict_ip_check_if_2fa_enabled`)

**Field Type:** Check  
**Default:** 0 (False)

**Dependency:** Chỉ hiển thị khi `enable_two_factor_auth` = 1

**Chức năng:**
- Khi enabled, **tất cả users** có thể login từ bất kỳ IP Address nào nếu có 2FA enabled
- Có thể set riêng cho từng user trong User Page

**Ví dụ:**
- User có Restricted IP = "192.168.1.100"
- `bypass_restrict_ip_check_if_2fa_enabled` = 1 → User có thể login từ IP khác nếu có 2FA
- `bypass_restrict_ip_check_if_2fa_enabled` = 0 → User chỉ có thể login từ IP được phép

---

#### 4.4.4. Two Factor Authentication method (`two_factor_method`)

**Field Type:** Select  
**Default:** "OTP App"  
**Options:**
- `OTP App` - Sử dụng OTP App (Google Authenticator, Authy, etc.)
- `SMS` - Sử dụng SMS
- `Email` - Sử dụng Email

**Dependency:** Chỉ hiển thị khi `enable_two_factor_auth` = 1

**Chức năng:**
- Xác định phương thức xác thực 2FA
- `OTP App`: User scan QR code và nhập OTP từ app
- `SMS`: User nhận OTP qua SMS
- `Email`: User nhận OTP qua Email

**Ví dụ:**
- `two_factor_method` = "OTP App" → Sử dụng OTP App
- `two_factor_method` = "SMS" → Sử dụng SMS (cần setup SMS Settings)

---

#### 4.4.5. Expiry time of QR Code Image Page (`lifespan_qrcode_image`)

**Field Type:** Int  
**Default:** None  
**Unit:** Seconds  
**Minimum:** 240

**Dependency:** Chỉ hiển thị khi `enable_two_factor_auth` = 1 VÀ `two_factor_method` = "OTP App"

**Chức năng:**
- Xác định thời gian (tính bằng giây) QR code image được lưu trên server
- Sau khi hết hạn, QR code sẽ không còn hoạt động

**Ví dụ:**
- `lifespan_qrcode_image` = 240 → QR code hết hạn sau 240 giây (4 phút)
- `lifespan_qrcode_image` = 600 → QR code hết hạn sau 600 giây (10 phút)

---

#### 4.4.6. OTP Issuer Name (`otp_issuer_name`)

**Field Type:** Data  
**Default:** "Frappe Framework"

**Dependency:** Chỉ hiển thị khi `enable_two_factor_auth` = 1

**Chức năng:**
- Tên hiển thị trong OTP App khi user scan QR code
- Giúp user nhận biết OTP từ hệ thống nào

**Ví dụ:**
- `otp_issuer_name` = "ERPNext" → OTP App hiển thị "ERPNext"
- `otp_issuer_name` = "My Company" → OTP App hiển thị "My Company"

---

#### 4.4.7. OTP SMS Template (`otp_sms_template`)

**Field Type:** Small Text  
**Default:** None

**Dependency:** Chỉ hiển thị khi `enable_two_factor_auth` = 1 VÀ `two_factor_method` = "SMS"

**Chức năng:**
- Template SMS để gửi OTP
- **Phải** chứa placeholder `{{otp}}` để insert OTP

**Code:**
```python
# File: system_settings.py

def validate_otp_sms_template(self):
    if not self.enable_two_factor_auth or self.two_factor_method != "SMS" or not self.otp_sms_template:
        return
    
    if "{{otp}}" not in self.otp_sms_template.replace(" ", ""):
        frappe.throw("OTP SMS Template must contain <code>{{otp}}</code> placeholder...")
```

**Ví dụ:**
- `otp_sms_template` = "Your OTP is {{otp}}. Valid for 10 minutes."
- SMS gửi: "Your OTP is 123456. Valid for 10 minutes."

---

## 🔷 5. Password Tab

### 5.1. Password Settings Section

#### 5.1.1. Logout All Sessions on Password Reset (`logout_on_password_reset`)

**Field Type:** Check  
**Default:** 1 (True)

**Chức năng:**
- Khi enabled, **logout tất cả sessions** khi user reset password
- Đảm bảo security khi password bị thay đổi

**Ví dụ:**
- User reset password
- `logout_on_password_reset` = 1 → Tất cả sessions bị logout
- `logout_on_password_reset` = 0 → Sessions vẫn active

---

#### 5.1.2. Force User to Reset Password (`force_user_to_reset_password`)

**Field Type:** Int  
**Default:** None  
**Unit:** Days

**Chức năng:**
- Xác định số ngày mà user **phải reset password**
- Sau số ngày này, user sẽ bị yêu cầu reset password khi login

**Code:**
```python
# File: system_settings.py

frappe.flags.update_last_reset_password_date = False
if self.force_user_to_reset_password and not cint(
    frappe.db.get_single_value("System Settings", "force_user_to_reset_password")
):
    frappe.flags.update_last_reset_password_date = True
```

**Ví dụ:**
- `force_user_to_reset_password` = 90 → User phải reset password sau 90 ngày
- `force_user_to_reset_password` = 30 → User phải reset password sau 30 ngày

---

#### 5.1.3. Reset Password Link Expiry Duration (`reset_password_link_expiry_duration`)

**Field Type:** Duration  
**Default:** 1200 (20 minutes)  
**Non-negative:** Yes

**Chức năng:**
- Xác định thời gian hết hạn của reset password link
- Sau khi hết hạn, link sẽ không còn hoạt động

**Ví dụ:**
- `reset_password_link_expiry_duration` = 1200 → Link hết hạn sau 20 phút
- `reset_password_link_expiry_duration` = 3600 → Link hết hạn sau 60 phút

---

#### 5.1.4. Password Reset Link Generation Limit (`password_reset_limit`)

**Field Type:** Int  
**Default:** 3

**Chức năng:**
- Giới hạn số lượng password reset links được tạo trong 1 giờ
- Ngăn chặn abuse và spam

**Ví dụ:**
- `password_reset_limit` = 3 → Tối đa 3 reset links/giờ
- `password_reset_limit` = 10 → Tối đa 10 reset links/giờ

---

#### 5.1.5. Enable Password Policy (`enable_password_policy`)

**Field Type:** Check  
**Default:** 1 (True)

**Chức năng:**
- Khi enabled, **bắt buộc** password phải đạt minimum password score
- Password strength được đánh giá từ 1 (rất yếu) đến 4 (rất mạnh)

**Code:**
```python
# File: system_settings.py

enable_password_policy = cint(self.enable_password_policy)
minimum_password_score = cint(getattr(self, "minimum_password_score", 0))
if enable_password_policy and minimum_password_score <= 0:
    frappe.throw(_("Please select Minimum Password Score"))
```

**Ví dụ:**
- `enable_password_policy` = 1 → Bắt buộc password đạt minimum score
- `enable_password_policy` = 0 → Không bắt buộc

---

#### 5.1.6. Minimum Password Score (`minimum_password_score`)

**Field Type:** Select  
**Default:** "2"  
**Options:**
- `1` - Very guessable: protection from throttled online attacks
- `2` - Somewhat guessable: protection from unthrottled online attacks
- `3` - Safely unguessable: moderate protection from offline slow-hash scenario
- `4` - Very unguessable: strong protection from offline slow-hash scenario

**Dependency:** Chỉ hiển thị khi `enable_password_policy` = 1

**Chức năng:**
- Xác định độ mạnh tối thiểu của password
- Password phải đạt score này mới được chấp nhận

**Ví dụ:**
- `minimum_password_score` = 2 → Password phải đạt score >= 2
- `minimum_password_score` = 4 → Password phải đạt score = 4 (rất mạnh)

---

## 🔷 6. Email Tab

### 6.1. Email Section

#### 6.1.1. Email Footer Address (`email_footer_address`)

**Field Type:** Small Text  
**Default:** None

**Chức năng:**
- Địa chỉ tổ chức hiển thị trong email footer
- Được thêm vào tất cả emails được gửi từ hệ thống

**Ví dụ:**
- `email_footer_address` = "My Company\n123 Main St\nCity, Country"
- Email footer sẽ hiển thị địa chỉ này

---

#### 6.1.2. Email Retry Limit (`email_retry_limit`)

**Field Type:** Int  
**Default:** 3

**Chức năng:**
- Số lần hệ thống sẽ **retry** gửi email nếu thất bại
- Sau khi đạt limit, email sẽ được đánh dấu là failed

**Ví dụ:**
- `email_retry_limit` = 3 → Retry 3 lần nếu gửi email thất bại
- `email_retry_limit` = 5 → Retry 5 lần nếu gửi email thất bại

---

#### 6.1.3. Disable Standard Email Footer (`disable_standard_email_footer`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **tắt** standard email footer mặc định
- Chỉ hiển thị custom footer nếu có

**Ví dụ:**
- `disable_standard_email_footer` = 1 → Không hiển thị standard footer
- `disable_standard_email_footer` = 0 → Hiển thị standard footer

---

#### 6.1.4. Hide footer in auto email reports (`hide_footer_in_auto_email_reports`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **ẩn** footer trong auto email reports
- Chỉ áp dụng cho auto email reports

**Ví dụ:**
- `hide_footer_in_auto_email_reports` = 1 → Không hiển thị footer trong reports
- `hide_footer_in_auto_email_reports` = 0 → Hiển thị footer trong reports

---

#### 6.1.5. Include Web View Link in Email (`attach_view_link`)

**Field Type:** Check  
**Default:** 1 (True)

**Chức năng:**
- Khi enabled, **thêm** Web View Link vào email
- User có thể xem document trực tiếp từ email mà không cần login

**Ví dụ:**
- `attach_view_link` = 1 → Email có link "View Document"
- `attach_view_link` = 0 → Email không có link

---

#### 6.1.6. Store Attached PDF Document (`store_attached_pdf_document`)

**Field Type:** Check  
**Default:** 1 (True)

**Chức năng:**
- Khi enabled, **lưu** PDF document vào Communication khi gửi email
- **Cảnh báo:** Có thể tăng storage usage

**Ví dụ:**
- `store_attached_pdf_document` = 1 → Lưu PDF vào Communication
- `store_attached_pdf_document` = 0 → Không lưu PDF

---

#### 6.1.7. Welcome Email Template (`welcome_email_template`)

**Field Type:** Link (Email Template)  
**Default:** None

**Chức năng:**
- Email Template được sử dụng để gửi welcome email cho users mới
- Nếu không set, sử dụng template mặc định

**Ví dụ:**
- `welcome_email_template` = "Welcome Email" → Sử dụng template "Welcome Email"
- `welcome_email_template` = None → Sử dụng template mặc định

---

#### 6.1.8. Reset Password Template (`reset_password_template`)

**Field Type:** Link (Email Template)  
**Default:** None

**Chức năng:**
- Email Template được sử dụng để gửi reset password email
- Nếu không set, sử dụng template mặc định

**Ví dụ:**
- `reset_password_template` = "Reset Password" → Sử dụng template "Reset Password"
- `reset_password_template` = None → Sử dụng template mặc định

---

## 🔷 7. Files Tab

### 7.1. Files Section

#### 7.1.1. Max File Size (MB) (`max_file_size`)

**Field Type:** Int  
**Default:** None  
**Non-negative:** Yes

**Chức năng:**
- Xác định kích thước file tối đa được phép upload (tính bằng MB)
- Nếu không set, sử dụng limit mặc định của server

**Ví dụ:**
- `max_file_size` = 10 → Tối đa 10 MB
- `max_file_size` = 100 → Tối đa 100 MB

---

#### 7.1.2. Allow Guests to Upload Files (`allow_guests_to_upload_files`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, cho phép **guests** (users chưa login) upload files
- Hữu ích cho web forms như job applications

**Ví dụ:**
- `allow_guests_to_upload_files` = 1 → Guests có thể upload files
- `allow_guests_to_upload_files` = 0 → Chỉ users đã login mới có thể upload

---

#### 7.1.3. Force Web Capture Mode for Uploads (`force_web_capture_mode_for_uploads`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **bắt buộc** sử dụng web-based image capture khi upload
- Khi disabled, sử dụng mobile native camera nếu detect mobile device

**Ví dụ:**
- `force_web_capture_mode_for_uploads` = 1 → Luôn dùng web camera
- `force_web_capture_mode_for_uploads` = 0 → Dùng native camera trên mobile

---

#### 7.1.4. Strip EXIF tags from uploaded images (`strip_exif_metadata_from_uploaded_images`)

**Field Type:** Check  
**Default:** 1 (True)

**Chức năng:**
- Khi enabled, **xóa** EXIF metadata từ uploaded images
- Bảo vệ privacy (EXIF có thể chứa location, camera info, etc.)

**Ví dụ:**
- `strip_exif_metadata_from_uploaded_images` = 1 → Xóa EXIF metadata
- `strip_exif_metadata_from_uploaded_images` = 0 → Giữ nguyên EXIF metadata

---

#### 7.1.5. Only allow System Managers to upload public files (`only_allow_system_managers_to_upload_public_files`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **chỉ System Managers** mới có thể upload public files
- Users khác không thấy checkbox "Is Private" trong upload dialog

**Ví dụ:**
- `only_allow_system_managers_to_upload_public_files` = 1 → Chỉ System Managers upload public files
- `only_allow_system_managers_to_upload_public_files` = 0 → Tất cả users có thể upload public files

---

#### 7.1.6. Delete Background Exported Reports After (Hours) (`delete_background_exported_reports_after`)

**Field Type:** Int  
**Default:** 48  
**Non-negative:** Yes

**Chức năng:**
- Xác định thời gian (tính bằng giờ) mà exported reports được giữ trong hệ thống
- Sau thời gian này, files sẽ được tự động xóa

**Ví dụ:**
- `delete_background_exported_reports_after` = 48 → Xóa sau 48 giờ
- `delete_background_exported_reports_after` = 24 → Xóa sau 24 giờ

---

#### 7.1.7. Allowed File Extensions (`allowed_file_extensions`)

**Field Type:** Small Text  
**Default:** None

**Chức năng:**
- Danh sách các file extensions được phép upload
- Mỗi dòng chứa một extension
- Nếu không set, tất cả extensions đều được phép

**Code:**
```python
# File: system_settings.py

def validate_file_extensions(self):
    if not self.allowed_file_extensions:
        return
    
    self.allowed_file_extensions = "\n".join(
        ext.strip().upper() for ext in self.allowed_file_extensions.strip().splitlines()
    )
```

**Ví dụ:**
- `allowed_file_extensions` = "CSV\nJPG\nPNG\nPDF" → Chỉ cho phép CSV, JPG, PNG, PDF
- `allowed_file_extensions` = None → Cho phép tất cả extensions

---

## 🔷 8. App Tab

### 8.1. Default App (`default_app`)

**Field Type:** Select  
**Default:** None

**Chức năng:**
- Xác định app mặc định được redirect sau khi login
- Nếu không set, redirect đến app đầu tiên trong list

**Ví dụ:**
- `default_app` = "ERPNext" → Redirect đến ERPNext sau khi login
- `default_app` = None → Redirect đến app đầu tiên

---

## 🔷 9. Updates Tab (Display)

### 9.1. System Updates Section

#### 9.1.1. Disable System Update Notification (`disable_system_update_notification`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **tắt** thông báo về system updates
- Users sẽ không thấy notification khi có update mới

**Ví dụ:**
- `disable_system_update_notification` = 1 → Không hiển thị update notification
- `disable_system_update_notification` = 0 → Hiển thị update notification

---

#### 9.1.2. Disable Change Log Notification (`disable_change_log_notification`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **tắt** thông báo về change log
- Users sẽ không thấy notification về changes

**Ví dụ:**
- `disable_change_log_notification` = 1 → Không hiển thị change log notification
- `disable_change_log_notification` = 0 → Hiển thị change log notification

---

#### 9.1.3. Hide Empty Read-Only Fields (`hide_empty_read_only_fields`)

**Field Type:** Check  
**Default:** 1 (True)

**Chức năng:**
- Khi enabled, **ẩn** các read-only fields nếu chúng empty
- Giúp form gọn gàng hơn

**Ví dụ:**
- `hide_empty_read_only_fields` = 1 → Ẩn empty read-only fields
- `hide_empty_read_only_fields` = 0 → Hiển thị tất cả fields

---

#### 9.1.4. Disable Product Suggestion (`disable_product_suggestion`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **tắt** product suggestions
- Users sẽ không thấy suggestions về products

**Ví dụ:**
- `disable_product_suggestion` = 1 → Tắt product suggestions
- `disable_product_suggestion` = 0 → Bật product suggestions

---

#### 9.1.5. Show External Link Warning (`show_external_link_warning`)

**Field Type:** Select  
**Default:** "Never"  
**Options:**
- `Never` - Không bao giờ hiển thị warning
- `Ask` - Hỏi user trước khi mở external link
- `Always` - Luôn hiển thị warning

**Chức năng:**
- Xác định khi nào hiển thị warning khi click external links
- Bảo vệ users khỏi phishing và malicious links

**Ví dụ:**
- `show_external_link_warning` = "Always" → Luôn hiển thị warning
- `show_external_link_warning` = "Never" → Không hiển thị warning

---

## 🔷 10. Backups Tab

### 10.1. Number of Backups (`backup_limit`)

**Field Type:** Int  
**Default:** 3

**Chức năng:**
- Xác định số lượng backups được giữ trong hệ thống
- Backups cũ hơn sẽ được tự động xóa

**Code:**
```python
# File: system_settings.py

def validate_backup_limit(self):
    if not self.backup_limit or self.backup_limit < 1:
        frappe.msgprint(_("Number of backups must be greater than zero."), alert=True)
        self.backup_limit = 1
```

**Ví dụ:**
- `backup_limit` = 3 → Giữ 3 backups gần nhất
- `backup_limit` = 10 → Giữ 10 backups gần nhất

---

### 10.2. Encrypt Backups (`encrypt_backup`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **mã hóa** backups
- Đảm bảo security cho backup files

**Ví dụ:**
- `encrypt_backup` = 1 → Backups được mã hóa
- `encrypt_backup` = 0 → Backups không được mã hóa

---

## 🔷 11. Advanced Tab

### 11.1. Reports Section

#### 11.1.1. Max auto email report per user (`max_auto_email_report_per_user`)

**Field Type:** Int  
**Default:** 20

**Chức năng:**
- Giới hạn số lượng auto email reports mỗi user có thể tạo
- Ngăn chặn abuse và spam

**Ví dụ:**
- `max_auto_email_report_per_user` = 20 → Tối đa 20 reports/user
- `max_auto_email_report_per_user` = 50 → Tối đa 50 reports/user

---

#### 11.1.2. Max Report Rows (`max_report_rows`)

**Field Type:** Int  
**Default:** 100000

**Chức năng:**
- Xác định số dòng tối đa có thể render trong report view
- Nếu report có nhiều dòng hơn, sẽ bị giới hạn

**Ví dụ:**
- `max_report_rows` = 100000 → Tối đa 100,000 dòng
- `max_report_rows` = 50000 → Tối đa 50,000 dòng

---

### 11.2. Background Workers Section

#### 11.2.1. Enable Scheduled Jobs (`enable_scheduler`)

**Field Type:** Check  
**Default:** 0 (False)  
**Hidden:** Yes

**Chức năng:**
- Khi enabled, **chạy** scheduled jobs (scheduler)
- Khi disabled, scheduled jobs sẽ không chạy

**Ví dụ:**
- `enable_scheduler` = 1 → Scheduled jobs chạy
- `enable_scheduler` = 0 → Scheduled jobs không chạy

---

#### 11.2.2. Run Jobs only Daily if Inactive For (Days) (`dormant_days`)

**Field Type:** Int  
**Default:** 4

**Chức năng:**
- Xác định số ngày inactive trước khi scheduler chỉ chạy jobs 1 lần/ngày
- Set = 0 để tránh tự động disable scheduler

**Ví dụ:**
- `dormant_days` = 4 → Nếu inactive 4 ngày, scheduler chỉ chạy 1 lần/ngày
- `dormant_days` = 0 → Scheduler luôn chạy bình thường

---

### 11.3. Telemetry Section

#### 11.3.1. Allow Sending Usage Data for Improving Applications (`enable_telemetry`)

**Field Type:** Check  
**Default:** 1 (True)

**Chức năng:**
- Khi enabled, **gửi** usage data cho developers để cải thiện ứng dụng
- Data được gửi ẩn danh

**Ví dụ:**
- `enable_telemetry` = 1 → Gửi usage data
- `enable_telemetry` = 0 → Không gửi usage data

---

#### 11.3.2. Show Full Error and Allow Reporting of Issues to the Developer (`allow_error_traceback`)

**Field Type:** Check  
**Default:** 1 (True)

**Chức năng:**
- Khi enabled, hiển thị **full error traceback** và cho phép report issues
- Khi disabled, chỉ hiển thị error message đơn giản

**Ví dụ:**
- `allow_error_traceback` = 1 → Hiển thị full traceback
- `allow_error_traceback` = 0 → Chỉ hiển thị error message

---

### 11.4. Search Section

#### 11.4.1. Link Field Results Limit (`link_field_results_limit`)

**Field Type:** Int  
**Default:** 10  
**Non-negative:** Yes  
**Maximum:** 50

**Chức năng:**
- Giới hạn số lượng kết quả hiển thị trong Link field dropdown
- Tối đa 50, mặc định 10

**Code:**
```python
# File: system_settings.py

if not self.link_field_results_limit:
    self.link_field_results_limit = 10

if self.link_field_results_limit > 50:
    self.link_field_results_limit = 50
    frappe.msgprint("Link Field Results Limit can not be more than 50", alert=True, indicator="yellow")
```

**Ví dụ:**
- `link_field_results_limit` = 10 → Hiển thị 10 kết quả
- `link_field_results_limit` = 50 → Hiển thị 50 kết quả

---

### 11.5. API Logging Section

#### 11.5.1. Log API Requests (`log_api_requests`)

**Field Type:** Check  
**Default:** 0 (False)

**Chức năng:**
- Khi enabled, **log** tất cả API requests
- Hữu ích cho debugging và monitoring

**Ví dụ:**
- `log_api_requests` = 1 → Log tất cả API requests
- `log_api_requests` = 0 → Không log API requests

---

## 🔷 12. Các Methods trong system_settings.py

### 12.1. `validate()`

**Chức năng:**
- Validate password policy, session expiry, 2FA, user pass login, backup limit, file extensions, OTP SMS template
- Set link_field_results_limit

---

### 12.2. `on_update()`

**Chức năng:**
- Set defaults cho tất cả fields
- Clear system settings cache
- Update last reset password date nếu cần

---

### 12.3. `set_defaults()`

**Chức năng:**
- Lưu tất cả field values vào `frappe.defaults`
- Set default language

---

### 12.4. `validate_user_pass_login()`

**Chức năng:**
- Validate không thể disable user/pass login nếu không có phương thức login khác

---

### 12.5. `validate_backup_limit()`

**Chức năng:**
- Validate backup limit phải >= 1

---

### 12.6. `validate_file_extensions()`

**Chức năng:**
- Format và validate allowed file extensions

---

### 12.7. `validate_otp_sms_template()`

**Chức năng:**
- Validate OTP SMS template phải chứa `{{otp}}` placeholder

---

## 📝 Tóm tắt

### Các nhóm settings chính:

1. **Localization** - Country, Language, Time Zone, Currency
2. **Date and Number Format** - Date format, Time format, Number format, Precision
3. **Permissions** - Strict User Permissions
4. **Security (Login)** - Session expiry, Login methods, Brute force security, 2FA
5. **Password** - Password policy, Reset password settings
6. **Email** - Email footer, Templates, Retry limit
7. **Files** - File size limit, Allowed extensions, Upload settings
8. **App** - Default app
9. **Display** - Update notifications, UI settings
10. **Backups** - Backup limit, Encryption
11. **Advanced** - Reports, Scheduler, Telemetry, Search, API logging

### Các settings quan trọng nhất:

1. **`session_expiry`** - Thời gian hết hạn session
2. **`enable_two_factor_auth`** - Bật 2FA
3. **`enable_password_policy`** - Bật password policy
4. **`apply_strict_user_permissions`** - Strict user permissions
5. **`language`** - Ngôn ngữ mặc định
6. **`date_format`** - Format ngày tháng
7. **`number_format`** - Format số

---

## 🔗 Tài liệu tham khảo

- **File source:**
  - `apps/frappe/frappe/core/doctype/system_settings/system_settings.json`
  - `apps/frappe/frappe/core/doctype/system_settings/system_settings.py`
