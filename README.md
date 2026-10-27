# LiteBoardSetup

Thư viện Arduino cho thiết bị ESP32 kết nối [LiteBoard](https://github.com/lavio-oss/liteboard). Repo này gồm 3 thư viện cần cài trong Arduino IDE trước khi biên dịch các sketch LiteBoard:

| Thư viện | Phiên bản | Dùng để |
|---|---|---|
| `LiteBoardSetup` | 1.0.0 | Kết nối Wi-Fi và cấu hình MQTT cho sketch LiteBoard. Phụ thuộc `PubSubClient`. |
| `PubSubClient` | 2.8 | Client MQTT mà các sketch LiteBoard sử dụng. |
| `ArduinoMqttClient` | 0.1.8 | Client MQTT của Arduino, kèm theo để dùng khi cần. |

Cấu trúc thư mục trong `libraries/` được giữ nguyên, mỗi thư viện là một thư mục độc lập theo đúng định dạng Arduino:

```text
libraries/
├── ArduinoMqttClient/
├── LiteBoardSetup/
│   ├── library.properties
│   └── src/
│       ├── LiteBoardSetup.h
│       └── LiteBoardSetup.cpp
└── PubSubClient/
```

## Cài đặt trên Arduino IDE

Hướng dẫn dùng cho Arduino IDE 2.x (IDE 1.8.x làm tương tự, menu có tên giống nhau).

### 1. Cài hỗ trợ board ESP32

1. Mở **File → Preferences** (macOS: **Arduino IDE → Settings**).
2. Ở ô **Additional boards manager URLs**, thêm dòng sau rồi bấm **OK**:

   ```text
   https://espressif.github.io/arduino-esp32/package_esp32_index.json
   ```

3. Mở **Tools → Board → Boards Manager**, tìm `esp32` và cài gói **esp32 by Espressif Systems**.

Gói này cung cấp sẵn `WiFi`, `WiFiClientSecure`, `HTTPClient`, `Preferences` và `cJSON` mà các sketch LiteBoard dùng, nên bạn không cần cài thêm.

### 2. Chép thư viện vào Sketchbook

1. Trong **File → Preferences**, xem đường dẫn ở ô **Sketchbook location**. Mặc định là:

   | Hệ điều hành | Đường dẫn |
   |---|---|
   | Windows | `C:\Users\<tên bạn>\Documents\Arduino` |
   | macOS | `~/Documents/Arduino` |
   | Linux | `~/Arduino` |

2. Tải repo này (nút **Code → Download ZIP**, giải nén) hoặc clone:

   ```bash
   git clone https://github.com/lavio-oss/LiteBoardSetup.git
   ```

3. Chép **các thư mục bên trong `libraries/`** (`ArduinoMqttClient`, `LiteBoardSetup`, `PubSubClient`) vào thư mục `libraries` của Sketchbook. Kết quả:

   ```text
   <Sketchbook>/libraries/LiteBoardSetup/library.properties
   <Sketchbook>/libraries/PubSubClient/library.properties
   <Sketchbook>/libraries/ArduinoMqttClient/library.properties
   ```

   Nếu thư mục `libraries` chưa có, hãy tạo nó. Chép từng thư mục thư viện, không chép nguyên thư mục `libraries/` của repo thành một lớp lồng thêm.

4. **Thoát hẳn và mở lại Arduino IDE** để IDE quét thư viện mới.

> Nếu bạn đã cài `PubSubClient` hoặc `ArduinoMqttClient` qua Library Manager, hãy gỡ bản đó (hoặc ghi đè bằng bản trong repo này) để không có hai bản cùng tên.

### 3. Kiểm tra thư viện đã nhận

Mở **Sketch → Include Library**. Bạn phải thấy `LiteBoardSetup`, `PubSubClient` và `ArduinoMqttClient` trong phần **Contributed libraries**. Nếu thiếu thư viện nào, kiểm tra lại đường dẫn Sketchbook và cấu trúc thư mục ở bước 2.

Sketch LiteBoard và hướng dẫn cấu hình thiết bị nằm ở repo chính [lavio-oss/liteboard](https://github.com/lavio-oss/liteboard), thư mục `ESP32/`.

## Giấy phép

Mỗi thư viện giữ nguyên giấy phép gốc trong thư mục của nó. `PubSubClient` theo MIT (`LICENSE.txt`), `ArduinoMqttClient` theo LGPL 2.1 (`LICENSE.txt`). `LiteBoardSetup` chưa có file giấy phép riêng.
