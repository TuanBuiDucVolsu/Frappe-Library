# Hướng Dẫn Thiết Lập Thị Giác Máy Tính

> **Dành cho:** Máy ấp trứng IoT — Raspberry Pi 5
> **Mục tiêu:** Dự đoán tỷ lệ trứng nở bằng USB webcam + OpenCV

---

## Danh Sách Thiết Bị Cần Mua

### Bắt buộc

| Thiết bị | Gợi ý sản phẩm | Giá tham khảo |
|----------|---------------|---------------|
| **USB Webcam** | Logitech C270 / C920 hoặc webcam USB bất kỳ hỗ trợ Linux | 200k – 800k VNĐ |
| **Đèn soi trứng** | Hộp soi trứng chuyên dụng (egg candler) có lỗ tròn vừa trứng | 150k – 350k VNĐ |

> **Có thể thay thế đèn soi bằng:** Đèn pin LED nhỏ ≥ 500 lumen (loại nhỏ gọn, chiếu tập trung). Tạm thời dùng được nhưng kết quả kém hơn hộp soi chuyên dụng.

### Tùy chọn (để tăng độ chính xác)

| Thiết bị | Mục đích | Giá tham khảo |
|----------|----------|---------------|
| **Giá đỡ webcam nhỏ** | Giữ webcam cố định, cách trứng đúng khoảng cách | 50k – 150k VNĐ |
| **Hộp che sáng** | Che ánh sáng phòng khi chụp, tăng độ tương phản | Tự làm bằng thùng carton |
| **Đèn LED strip trắng** | Chiếu đều bề mặt trứng khi phân tích surface | 50k – 100k VNĐ |

---

## Yêu Cầu Phần Mềm

- Raspberry Pi OS (64-bit, Bookworm trở lên)
- Python 3.10+
- Project `egg_incubator_iot` đã cài đặt và chạy được

---

## Các Bước Thiết Lập

### Bước 1 — Cài OpenCV

Mở terminal trên Raspberry Pi, chạy:

```bash
pip install opencv-python-headless
```

Kiểm tra cài đặt thành công:

```bash
python -c "import cv2; print('OpenCV OK:', cv2.__version__)"
```

Kết quả mong đợi: `OpenCV OK: 4.x.x`

---

### Bước 2 — Kết Nối Webcam

1. Cắm webcam vào cổng **USB-A** của Raspberry Pi 5
2. Kiểm tra Pi nhận ra webcam:

```bash
ls /dev/video*
```

Kết quả mong đợi: `/dev/video0` (nếu thấy nhiều hơn thì webcam thường là `video0`)

Xem tên webcam đang kết nối:

```bash
v4l2-ctl --list-devices
```

---

### Bước 3 — Test Camera Qua Terminal

```bash
python -c "
import cv2
cap = cv2.VideoCapture(0)
ok, frame = cap.read()
print('Camera OK:', ok)
if ok:
    print('Độ phân giải:', frame.shape[1], 'x', frame.shape[0])
cap.release()
"
```

Kết quả mong đợi:
```
Camera OK: True
Độ phân giải: 640 x 480
```

---

### Bước 4 — Test Camera Qua Web UI

1. Mở trình duyệt, vào `http://<IP-Pi>:8010`
2. Click **⚙️** góc trên phải → chọn **Thị giác máy tính**
3. Click nút **🔍 Test camera**

Kết quả mong đợi: `Camera OK — 640×480`

---

### Bước 5 — Thiết Lập Vị Trí Chụp

Đây là bước quan trọng nhất — ánh sáng quyết định chất lượng kết quả.

#### Sơ đồ bố trí (nhìn từ trên xuống)

```
┌─────────────────────────────────┐
│                                 │
│   [Đèn soi]  [Trứng]  [Webcam] │
│                                 │
│   ◄── 5cm ──►◄── 10-15cm ──►   │
│                                 │
└─────────────────────────────────┘
            (phòng tối)
```

**Lưu ý quan trọng:**
- Đặt webcam cách trứng **10–15 cm**
- Đèn soi đặt **phía sau trứng**, webcam nhìn **xuyên qua** trứng
- Phòng tối hoặc dùng hộp carton che ánh sáng xung quanh
- Trứng phải **che kín miệng đèn** để ánh sáng xuyên qua, không bị tán ra ngoài

#### Cách cầm khi chụp thủ công

```
Tay trái giữ đèn soi → áp trứng vào miệng đèn → webcam gắn cố định nhìn vào trứng
```

---

### Bước 6 — Chụp Ảnh Và Xem Kết Quả

1. Mở modal **Thị giác máy tính** (⚙️ → Thị giác máy tính)
2. Đặt trứng trước đèn soi đúng vị trí
3. Click **📷 Chụp & phân tích**
4. Đợi **3–5 giây** → kết quả hiện ra

**Giải thích kết quả:**

| Kết quả | Ý nghĩa |
|---------|---------|
| Giai đoạn: Không thụ tinh — 0% | Trứng không được thụ tinh, sẽ không nở |
| Giai đoạn: Phát triển sớm — ~65% | Phôi đang bắt đầu phát triển (ngày 1–7) |
| Giai đoạn: Phát triển giữa — ~76% | Phôi phát triển tốt (ngày 8–14) |
| Giai đoạn: Phát triển muộn — ~88% | Gần nở (ngày 15+) |
| Cảnh báo ⚠️ Vết nứt / Nấm mốc | Trứng có vấn đề về vỏ, tỷ lệ nở giảm |

**Độ tin cậy theo ngày ấp:**

| Ngày ấp | Độ tin cậy | Ghi chú |
|---------|-----------|---------|
| Ngày 1–3 | Thấp | Quá sớm, phôi chưa đủ lớn để nhận diện |
| Ngày 4–7 | Trung bình | Bắt đầu thấy mạch máu mờ |
| Ngày 8+ | Cao | Kết quả đáng tin cậy nhất |

> **Khuyến nghị:** Chụp lần đầu vào **ngày 7** — đây là mốc phát triển quan trọng nhất. Nếu trứng vẫn hiện "Không thụ tinh" vào ngày 7 thì khả năng cao trứng đó sẽ không nở.

---

### Bước 7 — Lịch Chụp Tự Động

Hệ thống tự động chụp ở 2 thời điểm mà không cần thao tác thủ công:

| Loại | Thời điểm | Ghi chú |
|------|-----------|---------|
| **Hàng ngày** | 8:00 sáng | 1 lần/ngày |
| **Mốc quan trọng** | Ngày 7, ngày 14, ngày lockdown | Tự kích hoạt 1 lần/mốc |

Để lịch tự động hoạt động, webcam phải được kết nối với Pi **trước khi khởi động app**.

---

## Nâng Cao — Train Model Riêng

Mặc định hệ thống dùng **OpenCV heuristic** (phân tích màu sắc và cạnh ảnh). Kết quả đủ dùng nhưng chưa tối ưu. Để tăng độ chính xác, có thể train một model AI riêng.

### Thu Thập Dữ Liệu

Trong quá trình ấp, chụp và lưu ảnh vào thư mục theo cấu trúc sau:

```
data/
├── infertile/       ← Chụp ngày 1-2, trứng soi thấy trong suốt
├── early_dev/       ← Chụp ngày 5-7, thấy mạch máu mờ
├── mid_dev/         ← Chụp ngày 10-14, phôi rõ ràng
└── late_dev/        ← Chụp ngày 17-20, gần như tối hoàn toàn
```

**Số lượng tối thiểu:** 50 ảnh/thư mục. Càng nhiều ảnh thì model càng chính xác.

### Chạy Training

```bash
# Cài TensorFlow (chỉ cần khi train, không cần khi chạy app)
pip install tensorflow

# Train model (mất 10-30 phút tùy số lượng ảnh)
python scripts/train_model.py
```

Model được xuất ra `vision/model/egg_classifier.tflite`. Khởi động lại app là hệ thống tự dùng model mới.

### Kiểm Tra Model

```bash
# Test model trên 1 ảnh bất kỳ
python scripts/train_model.py --test vision/model/egg_classifier.tflite duong_dan_anh.jpg
```

---

## Xử Lý Sự Cố

| Lỗi | Nguyên nhân | Cách sửa |
|-----|-------------|----------|
| `Camera lỗi: không kết nối được` | Webcam chưa cắm hoặc Pi chưa nhận | Kiểm tra `ls /dev/video*`, thử rút cắm lại |
| `Không chụp được ảnh` | OpenCV chưa cài | Chạy `pip install opencv-python-headless` |
| Tỷ lệ luôn hiện 45% | Không đủ ánh sáng, heuristic không nhận diện được | Tăng cường độ đèn soi, chụp trong phòng tối hơn |
| Ảnh quá tối / quá sáng | Webcam chưa tự chỉnh exposure kịp | Bình thường — code đã tự chụp 5 frame warmup. Nếu vẫn lỗi thì chờ thêm 2-3 giây trước khi nhấn nút |
| Giai đoạn nhận diện sai | Chưa có model, heuristic còn thô | Train model riêng theo hướng dẫn Bước 7 |

---

## Tóm Tắt Nhanh

```
1. pip install opencv-python-headless
2. Cắm webcam USB vào Pi
3. Chuẩn bị đèn soi trứng
4. Mở UI → ⚙️ → Thị giác máy tính → Test camera
5. Đặt trứng trước đèn → Chụp & phân tích
6. Đọc kết quả (tốt nhất từ ngày 7 trở đi)
```
