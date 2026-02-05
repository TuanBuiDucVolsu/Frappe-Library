# Hướng dẫn sử dụng POS Next

Tài liệu hướng dẫn chi tiết cho người dùng cuối sử dụng ứng dụng Point of Sale (POS) POS Next trên ERPNext.

---

## Mục lục

1. [Giới thiệu và truy cập](#1-giới-thiệu-và-truy-cập)
2. [Đăng nhập](#2-đăng-nhập)
3. [Mở ca làm việc](#3-mở-ca-làm-việc)
4. [Giao diện màn hình bán hàng](#4-giao-diện-màn-hình-bán-hàng)
5. [Thêm hàng vào giỏ](#5-thêm-hàng-vào-giỏ)
6. [Quản lý giỏ hàng](#6-quản-lý-giỏ-hàng)
7. [Khuyến mãi và Coupon](#7-khuyến-mãi-và-coupon)
8. [Thanh toán](#8-thanh-toán)
9. [Draft và lịch sử hóa đơn](#9-draft-và-lịch-sử-hóa-đơn)
10. [Trả hàng](#10-trả-hàng)
11. [Chế độ Offline và đồng bộ](#11-chế-độ-offline-và-đồng-bộ)
12. [Cài đặt POS và đóng ca](#12-cài-đặt-pos-và-đóng-ca)
13. [Xử lý lỗi thường gặp](#13-xử-lý-lỗi-thường-gặp)

---

## 1. Giới thiệu và truy cập

### POS Next là gì?

POS Next là ứng dụng bán hàng (Point of Sale) chạy trên nền ERPNext, hỗ trợ:

- **Bán hàng nhanh**: chọn hàng, thêm vào giỏ, chọn khách, thanh toán.
- **Làm việc offline**: khi mất mạng vẫn bán được, hóa đơn tự đồng bộ khi có mạng.
- **Khuyến mãi**: áp dụng offer, coupon theo cấu hình trên ERPNext.
- **Đa thiết bị**: dùng trên máy tính, tablet, điện thoại.

### Cách truy cập

- **Trên trình duyệt**: mở địa chỉ do quản trị cung cấp, thường dạng `https://<tên-site>/pos` hoặc `http://localhost:8102` (khi chạy dev).
- **PWA (ứng dụng cài đặt)**: nếu đã cài “Add to Home Screen” / “Cài đặt ứng dụng”, mở app POS Next từ màn hình chính.

Sau khi vào, nếu chưa đăng nhập bạn sẽ được chuyển tới màn hình **Đăng nhập**.

---

## 2. Đăng nhập

1. Trên màn hình **Sign in to POS Next**:
   - **User ID / Email**: nhập tên đăng nhập hoặc email tài khoản ERPNext.
   - **Password**: nhập mật khẩu.
2. (Tùy chọn) Bấm biểu tượng mắt để hiện/ẩn mật khẩu.
3. Bấm **Sign in**.
4. Nếu sai thông tin, thông báo **Login Failed** sẽ hiện — kiểm tra lại và thử lại.
5. Đăng nhập thành công → chuyển sang màn hình bán hàng. Nếu chưa mở ca, hộp thoại **Open POS Shift** sẽ tự mở.

---

## 3. Mở ca làm việc

Trước khi bán hàng, bạn phải **mở ca** (Open Shift) một lần trong phiên làm việc.

### Bước 1: Chọn POS Profile

1. Hộp thoại **Open POS Shift** hiện ra.
2. Chọn **một POS Profile** (ví dụ: "Main Store", "Counter 1").
3. Mỗi profile hiển thị: tên profile, công ty, đơn vị tiền tệ.
4. Bấm vào profile cần dùng → chuyển sang bước 2.

### Bước 2: Số dư mở ca (tùy chọn)

1. Có thể nhập **Opening Balance** cho từng **Mode of Payment** (Tiền mặt, Thẻ, v.v.) — thường là số tiền có trong quầy khi bắt đầu ca.
2. Để trống nếu không cần.
3. Bấm **Open Shift** để hoàn tất.

Sau khi mở ca:

- Header hiển thị tên profile, thời gian ca, trạng thái online/offline.
- Bạn có thể bắt đầu thêm hàng và bán.

**Lưu ý**: Nếu đã có ca đang mở (ví dụ mở từ lần trước chưa đóng), ứng dụng sẽ không bắt buộc mở ca lại; màn hình bán hàng hiện ngay.

---

## 4. Giao diện màn hình bán hàng

### Desktop

- **Header**: thời gian, thời lượng ca, tên profile, user, nút Sync, Refresh, Clear Cache, menu (View Shift, Draft Invoices, Invoice History, Return Invoice, Offline Invoices, Close Shift).
- **Bên trái**: danh sách/sử dụng hàng (**Items**): tìm kiếm, lọc, barcode, grid hàng.
- **Thanh chia**: kéo để thay đổi độ rộng cột Items vs Cart.
- **Bên phải**: **Giỏ hàng (Cart)**: danh sách hàng, khách hàng, tổng tiền, nút Thanh toán, Clear, Save Draft, Coupon, Offers.

### Mobile / Tablet

- **Tab Items / Cart**: chuyển qua lại giữa “xem hàng” và “xem giỏ”.
- **Nút giỏ nổi**: khi đang ở tab Items và giỏ có hàng, bấm nút tròn có số lượng để nhảy sang Cart.
- Header và menu tương tự desktop (có thể gộp trong menu).

---

## 5. Thêm hàng vào giỏ

### Cách thêm nhanh

1. **Tìm hàng**: gõ tên, mã hàng vào ô tìm kiếm hoặc quét **barcode**.
2. **Bấm vào hàng** (hoặc quét xong chọn hàng tương ứng):
   - Nếu hàng **không có biến thể, không bắt buộc chọn UOM/batch/serial** → hàng được thêm thẳng vào giỏ với số lượng 1.
   - Nếu có **nhiều UOM** (đơn vị): màn hình chọn UOM/số lượng hiện → chọn xong bấm thêm.
   - Nếu có **biến thể** (size, màu…): chọn biến thể → có thể tiếp tục chọn UOM hoặc thêm luôn.
   - Nếu có **batch/serial**: hộp thoại chọn Batch/Serial hiện → nhập hoặc chọn rồi xác nhận.

### Các trường hợp đặc biệt

- **Hàng hết tồn (và cấu hình không cho bán âm)**: ứng dụng báo lỗi, không thêm được — cần kiểm tra kho hoặc cấu hình “Allow Negative Stock”.
- **Barcode cân / giá**: nếu hệ thống hỗ trợ barcode cân (weight/price), sau khi quét có thể tự điền số lượng/đơn giá — kiểm tra và bấm thêm.
- **Số lượng**: với hàng đơn giản có thể tăng số lượng ngay trên ô tìm kiếm hoặc trong giỏ (xem phần Giỏ hàng).

---

## 6. Quản lý giỏ hàng

### Xem và chỉnh sửa

- **Tăng/giảm số lượng**: dùng nút +/- hoặc ô nhập số lượng bên cạnh từng dòng.
- **Xóa dòng**: bấm nút xóa (thùng rác) trên dòng hàng.
- **Đổi đơn vị (UOM)**: chọn UOM khác cho dòng — nếu đã có cùng hàng cùng UOM trong giỏ, hệ thống có thể gộp (merge) số lượng.
- **Chỉnh giảm giá dòng**: một số giao diện cho phép nhập % hoặc số tiền giảm theo dòng — tùy cấu hình POS.

### Khách hàng

- Bấm **Chọn khách** / **Customer**:
  - Tìm theo tên, số điện thoại, email.
  - Chọn **Walk-in** hoặc khách mặc định (nếu POS Profile có cấu hình) để không bắt buộc chọn khách.
- Khách ảnh hưởng đến:
  - Giá (price list theo khách).
  - Khuyến mãi (một số offer theo nhóm khách).
  - In hóa đơn (tên, địa chỉ).

### Xóa toàn bộ giỏ

- Bấm **Clear** / **Xóa giỏ** → xác nhận → toàn bộ hàng và khuyến mãi/coupon áp dụng trên giỏ sẽ bị xóa.

### Lưu tạm (Draft)

- Bấm **Save Draft** / **Lưu nháp**: lưu giỏ hiện tại thành draft để load lại sau (xem mục 9).

---

## 7. Khuyến mãi và Coupon

### Coupon (mã giảm giá)

1. Bấm **Coupon** / **Áp dụng Coupon** trên giỏ.
2. Nhập **mã coupon** (do quản trị tạo trên ERPNext).
3. Hệ thống kiểm tra điều kiện (đơn tối thiểu, khách, thời hạn…) và áp dụng giảm giá.
4. Để **bỏ coupon**: mở lại dialog Coupon và bỏ áp dụng, hoặc dùng nút gỡ coupon trên giỏ (nếu có).

### Offers (Khuyến mãi)

- **Tự động**: nhiều khuyến mãi (giảm %, giảm tiền, tặng hàng) được **tự áp dụng** khi giỏ thỏa điều kiện (số lượng, số tiền, nhóm hàng…). Thông báo dạng “Offer applied: …” sẽ hiện.
- **Thủ công**: bấm **Offers** / **Khuyến mãi** → chọn offer muốn áp dụng. Có thể bỏ từng offer đã áp dụng trong dialog.
- **Offline**: khi không có mạng, POS Next dùng dữ liệu offer đã cache để áp dụng giảm giá/ tặng hàng trên máy; khi lên mạng lại sẽ dùng đúng API để đồng bộ.

---

## 8. Thanh toán

### Mở màn hình thanh toán

1. Đảm bảo giỏ có ít nhất một mặt hàng.
2. (Tùy cấu hình) Chọn khách hàng nếu POS Profile bắt buộc.
3. Bấm **Proceed to Payment** / **Thanh toán**.

### Nhập tiền thanh toán

- Chọn **Mode of Payment** (Tiền mặt, Thẻ, Chuyển khoản…) và nhập **số tiền** tương ứng.
- **Thanh toán chia nhiều hình thức**: thêm nhiều dòng (ví dụ: 100k tiền mặt + 50k thẻ).
- **Partial payment** (nếu được bật): có thể thanh toán một phần, phần còn lại ghi nợ (Credit).
- **Write-off** (nếu được bật): có thể ghi nhận số tiền làm hao hụt/khuyến mãi nhỏ (thường có giới hạn theo ca).

### Hoàn tất

- Bấm **Submit** / **Xác nhận**:
  - **Online**: hóa đơn được tạo trên server, tồn kho cập nhật, có thể in hóa đơn ngay.
  - **Offline**: hóa đơn lưu local, chờ đồng bộ khi có mạng (xem mục 11).
- Sau khi tạo hóa đơn thành công:
  - Giỏ được xóa sạch.
  - Có thể bật in tự động (Auto Print) hoặc bấm in từ dialog thành công / từ lịch sử hóa đơn.

### Sales Order (đơn đặt hàng)

- Nếu POS Profile cho phép tạo **Sales Order** thay vì Sales Invoice, có thể chọn loại đơn (Invoice / Order) và nhập **Delivery Date** trước khi thanh toán — luồng thanh toán tương tự, chỉ khác doctype tạo ra.

---

## 9. Draft và lịch sử hóa đơn

### Draft (Hóa đơn nháp)

- **Lưu draft**: từ giỏ hàng bấm **Save Draft** — giỏ hiện tại được lưu với tên/ID draft.
- **Mở draft**: từ menu Header chọn **Draft Invoices** → chọn một draft → **Load** — nội dung giỏ được khôi phục (hàng, khách, offer đã áp dụng sẽ được load lại và có thể tính lại offer).
- **Xóa draft**: trong danh sách draft có thể xóa từng bản nháp khi không cần.

### Lịch sử hóa đơn

- **Invoice History**: từ Header → **Invoice History** (hoặc **Invoice Management** tùy phiên bản) — xem danh sách hóa đơn đã tạo trong ca/profile.
- **Xem chi tiết**: chọn một hóa đơn → xem nội dung, in lại.
- **In lại**: từ dialog chi tiết hoặc danh sách, chọn **Print** — in theo Print Format của POS Profile.

---

## 10. Trả hàng

- Từ Header chọn **Return Invoice**.
- Nhập **số hóa đơn gốc** (hoặc tìm theo khách/ngày) → chọn hóa đơn.
- Chọn **mặt hàng và số lượng** cần trả → xác nhận.
- Hệ thống tạo **Credit Note / Return** tương ứng; tồn kho được cập nhật.

Chi tiết trả hàng phụ thuộc cấu hình ERPNext (Return, Credit Note, link với Sales Invoice).

---

## 11. Chế độ Offline và đồng bộ

### Khi đang offline

- Header hiển thị trạng thái **Offline** (biểu tượng hoặc chữ).
- Bạn vẫn có thể:
  - Mở ca (nếu đã từng mở ca online và dữ liệu được cache).
  - Tìm hàng từ cache, thêm vào giỏ, áp dụng offer (theo dữ liệu đã cache).
  - Chọn khách từ cache, tạo hóa đơn.
- Hóa đơn tạo khi offline được lưu **local** với ID tạm (ví dụ `OFFLINE-...`). Chúng **chưa** có trên server.

### Đồng bộ khi lên mạng lại

- Khi kết nối lại, có thể bấm **Sync** (hoặc mở **Offline Invoices**) để xem danh sách **Pending Invoices**.
- Bấm **Sync All** (hoặc đồng bộ từng hóa đơn): ứng dụng gửi từng hóa đơn lên server. Thành công → hóa đơn có tên thật trên ERPNext; thất bại → giữ trong pending, có thể thử lại hoặc xóa (tùy chính sách).

**Lưu ý**: Nên đồng bộ khi mạng ổn định; tránh trùng lặp bằng cách không tạo lại hóa đơn đã sync.

---

## 12. Cài đặt POS và đóng ca

### Cài đặt POS (Settings)

- Từ menu (Management / Settings) mở **POS Settings** (hoặc **Settings** trong POS).
- Có thể chỉnh:
  - **Warehouse**: kho mặc định cho ca — ảnh hưởng tồn kho hiển thị và trừ khi bán.
  - **Tax Inclusive / Exclusive**: giá hiển thị đã gồm thuế hay chưa.
  - **Allow Negative Stock**, **Allow Partial Payment**, **Allow Credit Sale**, **Allow Write Off**, **Silent Print**, v.v. — tùy phiên bản và quyền.

### Đóng ca (Close Shift)

- Khi kết thúc ca: từ Header chọn **Close Shift**.
- Hệ thống có thể yêu cầu **đếm tiền thực tế** theo từng Mode of Payment và so với số dư hệ thống (tùy cấu hình).
- Xác nhận đóng ca → ca được đóng trên server; không thể thêm giao dịch vào ca đó nữa.

### Đăng xuất

- Bấm **User menu** (avatar/tên) → **Sign Out** (hoặc **Logout**).
- Nếu **đang có ca mở**, hệ thống có thể nhắc **đóng ca trước** rồi mới cho đăng xuất; hoặc cho chọn “Close Shift & Sign Out” / “Skip & Sign Out” tùy cấu hình.

---

## 13. Xử lý lỗi thường gặp

| Tình huống | Gợi ý xử lý |
|------------|-------------|
| **Trang trắng / không load** | Kiểm tra mạng; mở Console (F12) xem lỗi; thử refresh hoặc clear cache (Clear Cache trong menu). |
| **Không đăng nhập được** | Kiểm tra User ID và mật khẩu; xác nhận tài khoản ERPNext còn hoạt động và có quyền POS. |
| **Không có POS Profile** | Liên hệ quản trị tạo và gán POS Profile cho user/role. |
| **Hàng không thêm được – “Insufficient Stock”** | Kiểm tra kho đã chọn và tồn thực tế; hoặc bật “Allow Negative Stock” (nếu chính sách cho phép). |
| **Offer/Coupon không áp dụng** | Kiểm tra điều kiện (số lượng, số tiền, nhóm hàng, khách); thử bỏ coupon/offer và áp dụng lại. |
| **Thanh toán báo lỗi** | Kiểm tra đã chọn khách (nếu bắt buộc); kiểm tra số tiền và mode of payment; xem thông báo lỗi chi tiết trong dialog. |
| **Offline – đồng bộ thất bại** | Kiểm tra mạng và đăng nhập còn hợp lệ; mở Offline Invoices xem lỗi từng hóa đơn; thử Sync lại hoặc liên hệ quản trị. |
| **In không ra** | Kiểm tra Print Format và máy in trong POS Profile; thử in từ Invoice History; kiểm tra quyền in trên ERPNext. |
