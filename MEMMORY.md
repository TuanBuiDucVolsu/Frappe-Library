# Bài Giảng: Ứng Dụng IoT Trong Hệ Thống Máy Ấp Trứng Thông Minh

> **Dự án thực tế:** `egg_incubator_iot`
> **Nền tảng:** Raspberry Pi 5 + Python 3
> **Mức độ:** Trung cấp — Sinh viên đã biết lập trình Python cơ bản

---

## Mục Lục

1. [IoT Là Gì? Tại Sao Cần IoT Cho Máy Ấp Trứng?](#1-iot-là-gì)
2. [Kiến Trúc Tổng Thể](#2-kiến-trúc-tổng-thể)
3. [Lớp Phần Cứng — Thu Thập Dữ Liệu](#3-lớp-phần-cứng)
4. [Lớp Điều Khiển — Xử Lý & Ra Quyết Định](#4-lớp-điều-khiển)
5. [Lớp Kết Nối IoT — MQTT](#5-lớp-kết-nối-mqtt)
6. [Lớp Thông Báo — Telegram Bot](#6-telegram-bot)
7. [Lớp Ứng Dụng — Web UI & REST API](#7-web-ui--rest-api)
8. [Truy Cập Từ Xa — Cloudflare Tunnel](#8-cloudflare-tunnel)
9. [Lưu Trữ Dữ Liệu — SQLite](#9-lưu-trữ-dữ-liệu)
10. [Các Pattern Thiết Kế IoT Quan Trọng](#10-các-pattern-thiết-kế-iot)
11. [Thị Giác Máy Tính — Dự Đoán Tỷ Lệ Trứng Nở](#11-thị-giác-máy-tính)

---

## 1. IoT Là Gì?

**IoT (Internet of Things)** — "Internet Vạn Vật" — là mạng lưới các thiết bị vật lý có khả năng:
- **Thu thập dữ liệu** từ thế giới thực (cảm biến)
- **Xử lý** và **ra quyết định** tự động
- **Kết nối internet** để gửi/nhận dữ liệu từ xa
- **Điều khiển** thiết bị vật lý dựa trên dữ liệu đó

### Tại sao máy ấp trứng cần IoT?

Máy ấp trứng truyền thống yêu cầu người dùng phải:
- Ở gần máy để theo dõi nhiệt độ, độ ẩm
- Thủ công đảo trứng mỗi 4 giờ
- Không biết khi nào có sự cố nếu vắng nhà

Với IoT, hệ thống này giải quyết được:

| Vấn đề | Giải pháp IoT |
|--------|---------------|
| Phải ở gần máy để theo dõi | Web UI truy cập qua điện thoại bất kỳ đâu |
| Không biết khi có sự cố | Telegram gửi cảnh báo ngay lập tức |
| Đảo trứng thủ công | Động cơ bước tự động đảo mỗi 4 giờ |
| Không có lịch sử dữ liệu | SQLite ghi log mỗi 10 giây |
| Khó tích hợp với hệ thống khác | MQTT cho phép kết nối Node-RED, Home Assistant... |

---

## 2. Kiến Trúc Tổng Thể

Dự án này áp dụng **kiến trúc 4 lớp** phổ biến trong IoT:

```
┌─────────────────────────────────────────────────────────────────┐
│  LAYER 4: ỨNG DỤNG & TRỰC QUAN HÓA                             │
│  Web UI (HTML/JS) │ Node-RED Dashboard │ Telegram Bot           │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 3: KẾT NỐI & TRUYỀN THÔNG                               │
│  MQTT (paho)  │  HTTP/REST (Flask)  │  Cloudflare Tunnel        │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 2: XỬ LÝ & ĐIỀU KHIỂN  [Raspberry Pi 5 - Python]        │
│  IncubatorController  │  Bang-bang Control  │  SQLite Logger     │
│  EggAnalyzer (CV)     │  OpenCV + TFLite    │  VisionScheduler   │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 1: PHẦN CỨNG & CẢM BIẾN                                 │
│  DHT22 │ BME280 │ DS18B20 │ Relay x3 │ Step Motor │ LCD │ Nút  │
│  USB Webcam (Computer Vision)                                   │
└─────────────────────────────────────────────────────────────────┘
                        │ Raspberry Pi 5 │
                        └────────────────┘
```

**Điểm đặc biệt:** Toàn bộ 4 lớp đều chạy trên **một thiết bị duy nhất** — Raspberry Pi 5. Đây là mô hình **Edge Computing** — xử lý ngay tại thiết bị, không cần server trung tâm.

---

## 3. Lớp Phần Cứng

### 3.1 Sơ đồ kết nối GPIO

```
Raspberry Pi 5 (BCM Numbering)
│
├── GPIO26 ──── DHT22 (Data)          ← Cảm biến nhiệt độ + độ ẩm
├── GPIO2/3 ─── BME280 (SDA/SCL)      ← Cảm biến I2C (tùy chọn)
├── GPIO4 ───── DS18B20 (1-Wire)      ← Cảm biến nhiệt độ (tùy chọn)
│
├── GPIO23 ──── Relay → Thanh nhiệt   ← Bộ sưởi
├── GPIO18 ──── Relay → Máy phun sương← Tạo độ ẩm
├── GPIO17 ──── Relay → Quạt          ← Lưu thông khí
│
├── GPIO20 ──── Step Motor (STEP)     ← Đảo trứng
├── GPIO21 ──── Step Motor (DIR)      ← Hướng quay
│
├── GPIO2/3 ─── LCD 16x2 (I2C)       ← Hiển thị cục bộ
│
├── GPIO7  ──── Nút Gà               ← Chọn loài
├── GPIO8  ──── Nút Vịt
├── GPIO9  ──── Nút Bồ câu
└── GPIO10 ──── Nút Start/Stop
```

### 3.2 Cảm biến — Thu thập dữ liệu từ thế giới thực

File: `hardware/sensors.py`

Dự án hỗ trợ **3 loại cảm biến** với giao thức khác nhau:

#### DHT22 — Giao thức GPIO Bitbang
```python
# Dùng Adafruit Blinka thay vì RPi.GPIO (tương thích Pi 5)
import adafruit_dht

# use_pulseio=False → chế độ bitbang, ổn định hơn trên Pi 5
self._dht = adafruit_dht.DHT22(board.D26, use_pulseio=False)

temp = self._dht.temperature    # °C
humidity = self._dht.humidity   # %
```

> **Lưu ý quan trọng cho sinh viên:** DHT22 cần **tối thiểu 2 giây** giữa hai lần đọc. Đọc quá nhanh sẽ trả về lỗi. Code xử lý bằng cách thử lại sau 2.1 giây.

#### BME280 — Giao thức I2C
```python
import adafruit_bme280

i2c = board.I2C()                          # SDA=GPIO2, SCL=GPIO3
bme = adafruit_bme280.Adafruit_BME280_I2C(i2c)
temp = bme.temperature
humidity = bme.relative_humidity
```

I2C (Inter-Integrated Circuit) dùng chỉ 2 dây (SDA + SCL) để kết nối nhiều thiết bị — địa chỉ I2C phân biệt thiết bị (LCD dùng 0x27, BME280 thường dùng 0x76).

#### DS18B20 — Giao thức 1-Wire qua Kernel
```python
# Đọc trực tiếp từ filesystem Linux — kernel xử lý giao thức
device_file = "/sys/bus/w1/devices/28-XXXX/w1_slave"
with open(device_file, "r") as f:
    lines = f.readlines()
# Dòng kết thúc "YES" = CRC hợp lệ
temp_c = float(lines[1].split("t=")[1]) / 1000.0
```

> **Điểm thú vị:** DS18B20 không cần thư viện Python đặc biệt — Linux kernel đã xử lý giao thức 1-Wire và expose dữ liệu dưới dạng file text!

### 3.3 Actuators — Điều khiển thiết bị vật lý

File: `hardware/actuators.py`

#### Relay (Bộ điện từ bật/tắt)

```python
GPIO.setmode(GPIO.BCM)
GPIO.setup(pin, GPIO.OUT, initial=GPIO.HIGH)  # initial=HIGH = tắt (active-low)

# Bật relay: kéo chân xuống LOW (active-low logic)
GPIO.output(pin, GPIO.LOW)   # BẬT
GPIO.output(pin, GPIO.HIGH)  # TẮT
```

**Tại sao active-low?** Nhiều module relay phổ biến (như relay 4 kênh) hoạt động theo logic đảo ngược: cấp GND (LOW) → relay đóng, cấp 3.3V (HIGH) → relay mở. Dự án này cấu hình riêng từng relay để linh hoạt.

#### Động cơ bước (Stepper Motor) — Đảo trứng

```python
# A4988 driver: gửi xung STEP để motor quay từng bước
step_pin = GPIO20   # Mỗi xung HIGH→LOW = 1 bước
dir_pin  = GPIO21   # HIGH=thuận, LOW=ngược

# Đảo 1 lần = 1600 bước = 180° (nửa vòng)
GPIO.output(dir_pin, GPIO.HIGH)   # chọn chiều
for _ in range(1600):
    GPIO.output(step_pin, GPIO.HIGH)
    time.sleep(delay)              # tốc độ = 1/delay
    GPIO.output(step_pin, GPIO.LOW)
    time.sleep(delay)
```

Mỗi lần đảo đổi chiều (xen kẽ 0 ↔ 1) để trứng không bị lệch một phía sau nhiều chu kỳ.

### 3.4 Simulation Mode — Học mà không cần phần cứng

```python
def _read_simulation(self):
    temp     = self._target_temp + random.uniform(-0.5, 0.5)
    humidity = self._target_humidity + random.uniform(-3, 3)
    return (round(temp, 1), round(humidity, 1))
```

Khi không có phần cứng, hệ thống tự động chuyển sang simulation — tạo dữ liệu giả dao động quanh mục tiêu. **Rất hữu ích để test giao diện và logic điều khiển mà không cần phần cứng thật.**

---

## 4. Lớp Điều Khiển

File: `incubator_controller.py`

### 4.1 Vòng lặp điều khiển — Control Loop

```
Mỗi 10 giây:
┌──────────────────────────────────────────────────────┐
│  1. Đọc cảm biến (ngoài lock, có thể block 2.1s)    │
│  2. Vào RLock                                        │
│     a. Cập nhật trạng thái nội bộ                   │
│     b. Kiểm tra ngưỡng → bật/tắt relay              │
│     c. Kiểm tra lockdown                            │
│     d. Kiểm tra có cần đảo trứng không              │
│  3. Ra khỏi RLock                                   │
│  4. Ghi SQLite                                      │
│  5. Publish MQTT                                    │
│  6. Đảo trứng nếu cần (ngoài lock, block 6.4s)     │
└──────────────────────────────────────────────────────┘
```

> **Pattern quan trọng:** Các thao tác chậm (đọc cảm biến, đảo trứng, gửi mạng) được thực hiện **ngoài lock** để không block Web API trong thời gian đó.

### 4.2 Thuật toán Bang-Bang Control

Đây là thuật toán điều khiển nhiệt độ đơn giản nhất trong kỹ thuật điều khiển:

```python
# Ngưỡng ±0.2°C quanh mục tiêu (37.5°C cho gà)
if temp < target - 0.2:     # < 37.3°C
    heat_on()               # Bật nhiệt
elif temp > target + 0.2:   # > 37.7°C
    heat_off()              # Tắt nhiệt
# Trong khoảng 37.3°C - 37.7°C: giữ nguyên trạng thái hiện tại
```

```
Nhiệt độ
 38.0 ─────────────────────────── temp_max
 37.7 ── ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ── TẮTF nhiệt  ←┐
        ___     ___     ___                     │ Hysteresis
 37.5 ─    ─   ─   ─   ─   ──── target         │ (0.4°C)
 37.3 ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─── BẬT nhiệt  ←┘
 37.0 ─────────────────────────── temp_min
```

**Dải hysteresis (0.4°C)** ngăn relay bật/tắt liên tục khi nhiệt độ dao động nhỏ — bảo vệ relay khỏi hao mòn.

### 4.3 Giai đoạn Lockdown

```python
lockdown_day = profile["lockdown_day"]  # Gà: ngày 18/21
day = (datetime.now() - start_time).days

if day >= lockdown_day:
    is_lockdown = True
    # 1. Dừng đảo trứng (gà con bắt đầu định vị)
    # 2. Tăng độ ẩm: 50-55% → 65-70%
    # 3. Gửi thông báo Telegram + MQTT (chỉ 1 lần)
```

---

## 5. Lớp Kết Nối MQTT

File: `iot/mqtt_client.py`

### 5.1 MQTT Là Gì?

**MQTT (Message Queuing Telemetry Transport)** là giao thức nhắn tin nhẹ theo mô hình **Publish/Subscribe**:

```
┌──────────────┐   publish   ┌─────────────┐   deliver   ┌──────────────┐
│  Máy ấp trứng│ ──────────► │ MQTT Broker │ ──────────► │ Node-RED /   │
│  (Publisher) │             │ (HiveMQ)    │             │ Home Assist  │
└──────────────┘             └─────────────┘             └──────────────┘
                                    ▲
                   subscribe        │ publish cmd
              ┌────────────────────┘
              │
┌──────────────┐
│ App di động  │  {"cmd": "start", "bird_type": "chicken"}
│ (Controller) │
└──────────────┘
```

**Tại sao MQTT thay vì HTTP?**
- HTTP: client phải liên tục hỏi server ("polling") — tốn tài nguyên
- MQTT: server tự đẩy dữ liệu khi có sự kiện ("push") — hiệu quả hơn nhiều
- MQTT dùng ít bandwidth hơn → phù hợp IoT với kết nối yếu

### 5.2 Cấu trúc Topics

```
egg_incubator/              ← Topic prefix (cấu hình được)
├── status   → JSON đầy đủ  [retain=True, QoS 1]
├── temp     → "37.5"       [QoS 0]
├── humidity → "62.3"       [QoS 0]
├── running  → "true"       [QoS 0]
├── day      → "14"         [QoS 0]
├── alert    → JSON cảnh báo [QoS 1]
└── cmd      ← JSON lệnh    [Subscribe, QoS 1]
```

**Giải thích QoS (Quality of Service):**

| QoS | Đảm bảo | Dùng khi |
|-----|---------|----------|
| 0 | Gửi 1 lần, không xác nhận | Dữ liệu cảm biến liên tục (mất 1 gói không sao) |
| 1 | Ít nhất 1 lần (có thể trùng) | Alert, lệnh điều khiển quan trọng |
| 2 | Đúng 1 lần (đắt nhất) | Thanh toán, không dùng ở đây |

**`retain=True` trên `/status`:** Broker lưu message cuối cùng. Khi client mới kết nối, nhận ngay trạng thái mà không cần chờ publish tiếp theo — rất quan trọng cho dashboard.

### 5.3 Triển khai code MQTT

#### Kết nối không block với auto-reconnect

```python
def _setup(self):
    # Tạo client với unique ID (tránh xung đột trên broker)
    client_id = f"egg_incubator_{int(time.time())}"
    self._client = mqtt.Client(client_id=client_id)

    # Đăng ký callbacks
    self._client.on_connect    = self._on_connect
    self._client.on_disconnect = self._on_disconnect
    self._client.on_message    = self._on_message

    # Kết nối trong thread riêng — không block app khởi động
    threading.Thread(target=self._connect_loop, daemon=True).start()

def _connect_loop(self):
    while True:
        try:
            self._client.connect(broker, port, keepalive=60)
            self._client.loop_forever()  # paho tự xử lý reconnect
        except Exception as e:
            time.sleep(30)  # Thử lại sau 30 giây nếu exception
```

#### Subscribe và nhận lệnh từ xa

```python
def _on_connect(self, client, userdata, flags, rc):
    if rc == 0:
        self._connected = True
        client.subscribe(f"{self._prefix}/cmd", qos=1)

def _on_message(self, client, userdata, msg):
    payload = json.loads(msg.payload.decode("utf-8"))
    self.on_command(payload)  # callback → IncubatorController

# Trong IncubatorController:
def _on_mqtt_command(self, payload):
    if payload["cmd"] == "start":
        self.start_incubation(payload["bird_type"])
    elif payload["cmd"] == "stop":
        self.stop_incubation()
```

#### Publish dữ liệu sau mỗi chu kỳ

```python
def publish_status(self, status: dict):
    # /status: JSON đầy đủ, retain để client mới nhận ngay
    self._client.publish(
        f"{self._prefix}/status",
        json.dumps(status),
        qos=1, retain=True
    )
    # Các topic đơn để Node-RED/Home Assistant đọc trực tiếp
    self._client.publish(f"{self._prefix}/temp",     str(status["temp"]),     qos=0)
    self._client.publish(f"{self._prefix}/humidity", str(status["humidity"]), qos=0)
    self._client.publish(f"{self._prefix}/running",  str(status["running"]).lower(), qos=0)
    self._client.publish(f"{self._prefix}/day",      str(status["day"]),      qos=0)
```

### 5.4 Luồng dữ liệu MQTT hoàn chỉnh

```
Mỗi 10 giây:
Cảm biến → Controller → publish_status()
                      → egg_incubator/status  (JSON đầy đủ)
                      → egg_incubator/temp    (37.5)
                      → egg_incubator/humidity (62)
                      → egg_incubator/running  (true)
                      → egg_incubator/day      (14)

Khi có sự kiện:
Nhiệt độ vượt ngưỡng → publish_alert("temp_high", "Nhiệt độ 38.5°C...")
Cảm biến lỗi         → publish_alert("sensor_error", "...")
Vào lockdown         → publish_alert("lockdown", "Bắt đầu lockdown ngày 18")

Từ xa gửi lệnh:
{"cmd": "start", "bird_type": "duck"} → egg_incubator/cmd
    → _on_message() → _on_mqtt_command() → start_incubation("duck")
```

---

## 6. Telegram Bot

File: `iot/telegram_notifier.py`

### 6.1 Cơ chế hoạt động

Telegram Bot API sử dụng **HTTP POST** đơn giản:

```
https://api.telegram.org/bot{TOKEN}/sendMessage
Body: {"chat_id": "...", "text": "...", "parse_mode": "HTML"}
```

### 6.2 Rate Limiting — Tránh spam

```python
def _can_send(self, alert_type: str) -> bool:
    last = self._last_sent.get(alert_type, 0)
    if time.time() - last >= self.cooldown_sec:   # cooldown = 300s
        self._last_sent[alert_type] = time.time()
        return True
    return False                                   # Chưa đủ 5 phút → bỏ qua
```

**Tại sao cần rate limiting?** Nếu nhiệt độ dao động quanh ngưỡng báo động, không có rate limiting sẽ gửi hàng trăm tin nhắn/giờ → spam.

### 6.3 Phân loại thông báo: force vs rate-limited

```python
# Sự kiện quan trọng (force=True): gửi ngay, bỏ qua rate limit
notify_start()      → force=True   # Bắt đầu ấp
notify_stop()       → force=True   # Dừng ấp
notify_lockdown()   → force=True   # Vào giai đoạn nở

# Cảnh báo liên tục (force=False): áp dụng cooldown 5 phút
alert_temp_high()   → force=False
alert_temp_low()    → force=False
alert_sensor_error()→ force=False
```

### 6.4 Gửi bất đồng bộ (Non-blocking)

```python
def send(self, message: str, ...):
    threading.Thread(
        target=self._send_request,
        args=(token, chat_id, message),
        daemon=True,          # thread tự kết thúc khi app thoát
    ).start()
    # Trả về ngay, không chờ HTTP response
```

**Tại sao quan trọng?** Nếu gọi HTTP POST đồng bộ trong control loop, mạng chậm hoặc Telegram API timeout sẽ block toàn bộ hệ thống điều khiển nhiệt độ — nguy hiểm!

---

## 7. Web UI & REST API

File: `app.py`

### 7.1 Các endpoint chính

```
GET  /                    → index.html (giao diện web)
GET  /api/status          → {"temp": 37.5, "humidity": 62, "running": true, ...}
POST /api/start           → {"bird_type": "chicken"}
POST /api/stop            → {}
GET  /api/history?limit=100 → [{time, temp, humidity, day, sensor_ok}, ...]

GET  /api/mqtt/config     → cấu hình MQTT hiện tại
POST /api/mqtt/config     → cập nhật cấu hình MQTT

GET  /api/telegram/bots   → danh sách bot
POST /api/telegram/bots   → thêm bot mới
POST /api/telegram/bots/test → gửi tin nhắn test

# --- Thị giác máy tính ---
GET  /api/vision/status          → trạng thái module + kết quả gần nhất
POST /api/vision/capture         → chụp & phân tích ngay (chạy nền)
GET  /api/vision/history?limit=N → lịch sử N lần phân tích gần nhất
GET  /api/vision/camera/test     → kiểm tra webcam kết nối
GET  /captures/<filename>        → phục vụ ảnh đã chụp
```

### 7.2 Web UI polling mỗi 5 giây

```javascript
// Client-side JavaScript trong index.html
setInterval(async () => {
    const resp = await fetch('/api/status');
    const data = await resp.json();
    updateDisplay(data.temp, data.humidity, data.running);
}, 5000);
```

---

## 8. Cloudflare Tunnel

File: `setup_cloudflare.sh`

**Vấn đề:** Raspberry Pi ở nhà thường không có IP public. Muốn truy cập từ ngoài internet?

**Giải pháp Cloudflare Tunnel:**

```
Người dùng           Cloudflare Edge        Raspberry Pi (LAN)
    │                      │                       │
    │ HTTPS request         │                       │
    │──────────────────────►│   WebSocket tunnel    │
    │                       │──────────────────────►│
    │                       │    cloudflared agent  │
    │◄──────────────────────│◄──────────────────────│
    │   response            │                       │
```

```bash
# Tạo tunnel miễn phí (URL ngẫu nhiên, không cần tài khoản)
cloudflared tunnel --url http://localhost:8010

# Kết quả: https://random-name.trycloudflare.com → Pi:8010
```

**Ưu điểm:** Không cần mở port router, không cần IP tĩnh, HTTPS tự động — phù hợp cho prototype và học tập.

---

## 9. Lưu Trữ Dữ Liệu

File: `data_logger.py`

### SQLite Schema

```sql
CREATE TABLE sessions (
    id TEXT PRIMARY KEY,        -- UUID, tạo mỗi lần start
    bird_type TEXT,
    start_time TEXT,
    end_time TEXT,
    notes TEXT
);

CREATE TABLE measurements (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    session_id TEXT,
    timestamp TEXT,
    bird_type TEXT,
    day INTEGER,
    temp REAL,
    humidity REAL,
    heat_on INTEGER,           -- 0/1
    humidity_on INTEGER,
    fan_on INTEGER,
    turn_position INTEGER,
    is_lockdown INTEGER,
    sensor_ok INTEGER
);
```

**Ghi mỗi 10 giây** → 1 lần ấp gà 21 ngày = 21 × 24 × 360 = **181,440 bản ghi**.

Dữ liệu này có thể export CSV để phân tích:
- Biểu đồ nhiệt độ theo ngày
- Tỷ lệ thời gian relay bật (hiệu suất năng lượng)
- So sánh giữa các lần ấp

---

## 10. Các Pattern Thiết Kế IoT

Dự án này minh họa nhiều pattern quan trọng trong thiết kế hệ thống IoT:

### Pattern 1: Graceful Degradation (Suy giảm có kiểm soát)

Hệ thống **không crash** khi thiếu thành phần:

```python
# Không có thư viện GPIO? → Chạy simulation
self.simulation = simulation or not HAS_GPIO

# Không cài paho-mqtt? → Tắt MQTT, app vẫn chạy
self.enabled = config.get("mqtt_enabled", False) and HAS_MQTT

# Cảm biến lỗi? → Bật quạt, tắt nhiệt (fail-safe), không crash
if not sensor_ok:
    heat_off(); fan_on()
    publish_alert("sensor_error", "...")
    return  # skip control logic, giữ trứng an toàn
```

### Pattern 2: Atomic Write (Ghi nguyên tử)

Tránh file hỏng khi mất điện giữa chừng:

```python
# KHÔNG làm thế này (nguy hiểm):
with open("state.json", "w") as f:
    json.dump(state, f)  # ← nếu mất điện ở đây, file hỏng

# ĐÚNG: ghi vào file tạm, rồi đổi tên nguyên tử
tmp = "state.json.tmp"
with open(tmp, "w") as f:
    json.dump(state, f)
os.replace(tmp, "state.json")  # atomic trên POSIX
```

`os.replace()` là thao tác nguyên tử — file cũ hoặc file mới, không bao giờ có trạng thái nửa vời.

### Pattern 3: Thread Safety với RLock

```python
self._lock = threading.RLock()  # Reentrant lock

def get_status(self):
    with self._lock:              # Tự động unlock kể cả khi exception
        return {
            "temp": self._last_sensor["temp"],
            "running": self.is_running,
            ...
        }
```

RLock (Reentrant Lock) cho phép cùng một thread lock nhiều lần — tránh deadlock khi hàm A gọi hàm B, cả hai đều cần lock.

### Pattern 4: Dependency Injection cho Testability

```python
# MQTTClient không gọi trực tiếp controller — nhận callback qua tham số
class MQTTClient:
    def __init__(self, config: dict, on_command: Callable[[dict], None]):
        self.on_command = on_command  # inject từ ngoài vào

# Controller inject callback vào:
self.mqtt = MQTTClient(
    config=IOT_CONFIG,
    on_command=self._on_mqtt_command,  # callback
)
```

Khi test, có thể inject mock callback thay vì controller thật — không cần khởi động toàn bộ hệ thống.

### Pattern 5: Configuration over Code

```python
# config.py — tham số kỹ thuật theo từng loài
BIRD_PROFILES = {
    "chicken": {"temp_target": 37.5, "lockdown_day": 18, ...},
    "duck":    {"temp_target": 37.5, "lockdown_day": 25, ...},
    "pigeon":  {"temp_target": 37.2, "lockdown_day": 14, ...},
}

# Thêm loài mới: chỉ cần thêm vào dict, không sửa logic điều khiển
```

### Pattern 6: Non-blocking I/O cho Real-time System

```python
# SAI: block control loop khi gửi Telegram
def _control_once(self):
    requests.post(telegram_url, ...)  # ← có thể block 10 giây
    self.mqtt.publish_status(...)     # ← trễ nghiêm trọng

# ĐÚNG: gửi trong thread nền
threading.Thread(target=self._send_request, daemon=True).start()
# Trả về ngay, control loop không bị block
```

---

## Tóm Tắt

| Thành phần | Công nghệ | Vai trò trong IoT |
|------------|-----------|-------------------|
| Raspberry Pi 5 | Linux + Python | Edge device — xử lý tại chỗ |
| DHT22/BME280/DS18B20 | GPIO/I2C/1-Wire | Thu thập dữ liệu vật lý |
| Relay + Step Motor | GPIO | Điều khiển thiết bị vật lý |
| MQTT (paho) | TCP/IP | Pub/Sub, tích hợp hệ sinh thái IoT |
| Telegram Bot | HTTPS | Thông báo sự kiện real-time |
| Flask | HTTP/REST | Giao diện web local |
| Cloudflare Tunnel | HTTPS | Truy cập từ xa an toàn |
| SQLite | SQL | Lưu trữ dữ liệu nghiên cứu |
| Node-RED | MQTT | Dashboard trực quan (tùy chọn) |
| OpenCV + TFLite | USB Webcam | Thị giác máy tính — dự đoán tỷ lệ nở |

Dự án này là ví dụ hoàn chỉnh về một hệ thống IoT thực tế, áp dụng đầy đủ các nguyên tắc: **thu thập dữ liệu → xử lý cục bộ → kết nối cloud → thông báo → lưu trữ** — tất cả trên một thiết bị nhúng duy nhất với chi phí thấp.

---

## 11. Thị Giác Máy Tính

Module: `vision/` — Tích hợp Computer Vision vào hệ thống IoT

### 12.1 Tổng quan

Thị giác máy tính (Computer Vision — CV) cho phép máy ấp trứng **"nhìn"** trứng và dự đoán tỷ lệ nở mà không cần cảm biến chuyên dụng — chỉ cần một USB webcam thông thường.

```
USB Webcam
    │
    ▼
EggCamera.capture()        ← Chụp 2 ảnh (surface + candling)
    │
    ▼
ImageProcessor             ← OpenCV: phân tích mạch máu, mốc, vết nứt
    │
    ▼
EggClassifier              ← TFLite MobileNetV2 (hoặc heuristic fallback)
    │
    ▼
EggAnalyzer                ← Tính hatch_rate, ghi vision_log.json
    │
    ▼
API /api/vision/status     ← Web UI cập nhật card dự đoán
```

### 12.2 Hai loại phân tích ảnh

#### Candling (Soi trứng)
Chiếu đèn vào trứng để thấy sự phát triển bên trong — mô phỏng kỹ thuật truyền thống của người nông dân.

```python
# image_processor.py — analyze_candling()
gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
blurred = cv2.GaussianBlur(gray, (5,5), 0)

# Phát hiện mạch máu = cạnh trong ảnh mờ
edges = cv2.Canny(blurred, threshold1=30, threshold2=80)
vessel_density = np.count_nonzero(edges) / total_pixels

# Phần tối = phôi chiếm chỗ
dark_ratio = np.sum(gray < 80) / total_pixels

# Điểm màu mỡ tổng hợp
fertility_score = brightness_score*0.3 + vessel_density*0.5 + dark_ratio*0.2
```

**Giải thích:** Trứng thụ tinh có mạch máu → nhiều cạnh (edges) hơn khi xử lý Canny. Trứng phát triển muộn có phôi lớn chiếm phần tối → dark_ratio cao.

#### Surface Analysis (Phân tích bề mặt)
Kiểm tra vỏ trứng — phát hiện vết nứt và nấm mốc.

```python
# analyze_surface()

# Phát hiện vết nứt (Canny ngưỡng cao hơn)
crack_edges = cv2.Canny(gray, 100, 200)
crack_ratio = np.count_nonzero(crack_edges) / total_pixels

# Phát hiện nấm mốc (màu xanh/xanh lá trong HSV)
hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
mold_mask = cv2.inRange(hsv,
    np.array([35, 40, 40]),    # Hue 35-85 = xanh lá
    np.array([85, 255, 255]))
has_mold = np.count_nonzero(mold_mask) > 500

surface_ok = not has_crack and not has_mold
```

### 12.3 Phân loại giai đoạn phát triển

File: `vision/classifier.py`

#### TFLite Model (khi có file model)
```python
# MobileNetV2 — pretrained trên ImageNet, fine-tune trên dữ liệu trứng
interpreter = tflite.Interpreter(model_path="vision/model/egg_classifier.tflite")
interpreter.allocate_tensors()

# Resize + normalize ảnh
img = cv2.resize(frame, (224, 224))
img = img.astype(np.float32) / 255.0
img = np.expand_dims(img, axis=0)   # shape: (1, 224, 224, 3)

interpreter.set_tensor(input_index, img)
interpreter.invoke()
output = interpreter.get_tensor(output_index)[0]  # [p0, p1, p2, p3, p4]

LABELS = ["infertile", "early_development", "mid_development",
          "late_development", "dead"]
predicted_class = LABELS[np.argmax(output)]
confidence = float(np.max(output))
```

#### OpenCV Heuristic Fallback (khi chưa có model)
Khi chưa có dữ liệu để train, hệ thống tự động dùng luật heuristic:

```python
# Dựa trên vessel_density và dark_ratio từ ImageProcessor
if vessel_density > 0.08 and dark_ratio > 0.35:
    return "late_development"      # Phôi lớn, nhiều mạch máu
elif vessel_density > 0.05 and dark_ratio > 0.20:
    return "mid_development"
elif vessel_density > 0.025:
    return "early_development"     # Bắt đầu có mạch máu
elif mean_brightness > 190:
    return "infertile"             # Sáng đều = trứng trắng (không thụ tinh)
else:
    return "unknown"
```

> **Điểm quan trọng:** Hệ thống **không crash** khi chưa có model — Graceful Degradation. Kết quả kém chính xác hơn nhưng vẫn hoạt động.

### 12.4 Tính tỷ lệ nở dự đoán

File: `vision/egg_analyzer.py`

```python
# Tỷ lệ cơ sở theo giai đoạn (dựa trên thống kê thực tế)
_STAGE_BASE_RATE = {
    "infertile":         0.00,
    "early_development": 0.65,
    "mid_development":   0.76,
    "late_development":  0.88,
    "dead":              0.00,
    "unknown":           0.45,
}

base_rate = _STAGE_BASE_RATE[predicted_class]

# Khi model kém tin cậy (confidence < 0.6): blend thêm OpenCV score
if confidence < 0.6:
    opencv_rate = fertility_score * 0.88
    base_rate = base_rate * 0.5 + opencv_rate * 0.5

# Penalty cho vấn đề vỏ trứng
if has_crack: base_rate *= 0.70   # Vết nứt: giảm 30%
if has_mold:  base_rate *= 0.60   # Nấm mốc: giảm 40%

# Độ tin cậy dự đoán theo ngày ấp
if day <= 3:   confidence_level = "low"    # Quá sớm
elif day <= 7: confidence_level = "medium"
else:          confidence_level = "high"   # Độ chính xác tốt nhất
```

### 12.5 Lịch chụp tự động

File: `vision/scheduler.py`

```python
CHECK_INTERVAL = 60  # giây

# Chụp hàng ngày lúc 08:00 sáng
if now.hour == capture_hour and now.date() != last_capture_date:
    trigger_capture("scheduled")
    last_capture_date = now.date()

# Chụp tự động ở các mốc quan trọng (1 lần/session)
MILESTONES = [7, 14, lockdown_day]
if current_day in MILESTONES and current_day not in triggered_milestones:
    trigger_capture("milestone")
    triggered_milestones.add(current_day)
```

Ngày 7 và 14 là các mốc quan trọng trong quá trình phát triển phôi gà — thay đổi rõ rệt nhất khi soi trứng.

### 12.6 Train model TFLite riêng

File: `scripts/train_model.py`

Khi đã thu thập đủ ảnh thực tế:

```
data/
├── infertile/       ← ≥50 ảnh trứng không thụ tinh
├── early_dev/       ← ≥50 ảnh ngày 1-7
├── mid_dev/         ← ≥50 ảnh ngày 8-14
└── late_dev/        ← ≥50 ảnh ngày 15-21
```

```bash
# Train và export TFLite
python scripts/train_model.py

# Test model trên 1 ảnh
python scripts/train_model.py --test vision/model/egg_classifier.tflite anh_mau.jpg
```

Quá trình train 2 giai đoạn:
1. **Freeze base** — chỉ train lớp top (Dense) với LR=1e-4
2. **Fine-tune** — unfreeze 20 lớp cuối MobileNetV2, LR=1e-5

### 12.7 Tích hợp vào Web UI

Kết quả phân tích hiển thị trực tiếp trên giao diện:

- **Card nhỏ** trên trang chính: tỷ lệ %, giai đoạn, ảnh thumbnail (hiện sau lần phân tích đầu tiên)
- **Modal đầy đủ** (⚙️ → Thị giác máy tính): ảnh 140×140, tỷ lệ lớn, 4 ô chi tiết, cảnh báo nứt/mốc, lịch sử 20 lần gần nhất
- **Nút "Chụp & phân tích"**: chạy nền, cập nhật sau ~4 giây
- **Poll tự động** mỗi 30 giây: card cập nhật nếu có kết quả mới từ scheduler

```
Kết quả API /api/vision/status:
{
  "hatch_rate":       0.76,          // 76%
  "stage":            "mid_development",
  "fertility_score":  0.71,
  "confidence_level": "high",
  "note":             "Giai đoạn giữa — độ chính xác tốt nhất.",
  "surface": {
    "has_crack": false,
    "has_mold":  false
  },
  "image":     "candling_day7_20260609_080012.jpg",
  "timestamp": "2026-06-09T08:00:12.345678"
}
```

### 12.8 Graceful Degradation trong Vision Module

```python
# camera.py — không có OpenCV? Trả về None an toàn
try:
    import cv2
    HAS_CV2 = True
except ImportError:
    HAS_CV2 = False

def capture(self):
    if not HAS_CV2:
        return None   # caller kiểm tra is not None

# classifier.py — không có TFLite model? Dùng heuristic
try:
    import tflite_runtime.interpreter as tflite
    HAS_TFLITE = True
except ImportError:
    HAS_TFLITE = False

def predict(self, frame):
    if self.model_available and HAS_TFLITE:
        return self._predict_tflite(frame)
    return self._predict_heuristic(frame)  # fallback luôn có
```

**Bảng trạng thái hoạt động:**

| OpenCV | TFLite Model | Kết quả |
|--------|-------------|---------|
| ✓ | ✓ | Chính xác nhất — model + CV |
| ✓ | ✗ | Heuristic OpenCV — hoạt động tốt sau ngày 7 |
| ✗ | - | Trả về empty dict — hatch_rate = 0.45 (unknown) |
