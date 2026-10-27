# LiteBoardSetup

Thư viện Arduino cho thiết bị ESP32 kết nối [LiteBoard](https://github.com/lavio-oss/liteboard). Repo này gồm đúng 3 thư viện cần có để biên dịch các sketch LiteBoard (`AccessToken`, `AccessTokenNoTls`, `Provisioning`) bằng Arduino IDE:

| Thư viện | Phiên bản | Dùng để |
|---|---|---|
| `LiteBoardSetup` | 1.0.0 | Kết nối Wi-Fi và cấu hình MQTT (TLS có xác minh CA, hoặc không TLS để thử nhanh). Phụ thuộc `PubSubClient`. |
| `PubSubClient` | 2.8 | Client MQTT mà các sketch LiteBoard sử dụng. |
| `ArduinoMqttClient` | 0.1.8 | Client MQTT của Arduino, kèm theo để dùng khi cần. Các sketch LiteBoard hiện không gọi tới. |

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

### 4. Lấy sketch và biên dịch

Sketch nằm ở repo chính [lavio-oss/liteboard](https://github.com/lavio-oss/liteboard), thư mục `ESP32/`:

| Sketch | Dùng khi |
|---|---|
| `AccessToken/AccessToken.ino` | Thiết bị đã có Access Token; kết nối MQTT qua TLS (cổng `8883`). |
| `AccessTokenNoTls/AccessTokenNoTls.ino` | Thử nhanh qua cổng playground `1883` không TLS. Thiết bị phải bật **Allow connection without TLS**. |
| `Provisioning/Provisioning.ino` | Thiết bị tự đăng ký qua HTTPS rồi lấy Access Token. |

1. Mở file `.ino` tương ứng trong Arduino IDE.
2. Sao chép các file `*.example` cùng thư mục thành file thật (bỏ đuôi `.example`) và điền giá trị của bạn:
   - `AccessToken`: `device_config.h` và `mqtt_ca_cert.h`.
   - `AccessTokenNoTls`: `device_config.h`.
   - `Provisioning`: `provisioning_config.h` và `provisioning_ca_cert.h`; sketch này còn dùng `mqtt_ca_cert.h` của `AccessToken`.

   Các file này chứa Wi-Fi, host, Project ID, Device ID, Access Token hoặc provisioning secret. Chỉ nhúng **public CA certificate**, không bao giờ nhúng private key, và không đưa các file thật lên Git.
3. Chọn board ở **Tools → Board → esp32** (ví dụ `ESP32 Dev Module`) và cổng ở **Tools → Port**.
4. Bấm **Upload**. Nếu quá trình nạp dừng ở `Connecting...`, giữ nút **BOOT** trên board cho tới khi bắt đầu nạp.
5. Mở **Tools → Serial Monitor**, đặt tốc độ **115200 baud**. Khi thành công bạn thấy log `[Wi-Fi] Connected` rồi kết nối MQTT; thiết bị hiện **Online** trong LiteBoard.

Chi tiết cấu hình từng sketch có trong [docs/setup/esp32.md](https://github.com/lavio-oss/liteboard/blob/main/docs/setup/esp32.md) của repo chính.

## API của `LiteBoardSetup`

```cpp
#include <LiteBoardSetup.h>

// Kết nối Wi-Fi ở chế độ station; chặn cho tới khi có IP.
LiteBoardSetup::connectWiFi(ssid, password);

// MQTT qua TLS, xác minh chứng chỉ broker bằng CA bạn cung cấp.
LiteBoardSetup::configureMqttTls(tlsClient, mqttClient, caCertificate, host, port, keepAliveSeconds);

// MQTT không TLS (chỉ để thử nhanh qua cổng 1883).
LiteBoardSetup::configureMqtt(mqttClient, host, port, keepAliveSeconds);
```

`keepAliveSeconds` mặc định là `60`. Khi dùng TLS, host của broker phải khớp DNS hoặc IP trong SAN của chứng chỉ MQTT.

## Lỗi thường gặp

| Triệu chứng | Cách xử lý |
|---|---|
| `fatal error: LiteBoardSetup.h: No such file or directory` | Thư viện chưa nằm đúng `<Sketchbook>/libraries/LiteBoardSetup/` hoặc chưa khởi động lại IDE. |
| `fatal error: PubSubClient.h: No such file or directory` | Chưa chép thư mục `PubSubClient`. `LiteBoardSetup` phụ thuộc thư viện này. |
| `multiple libraries were found for "PubSubClient.h"` | Có hai bản `PubSubClient`. Giữ một bản trong `<Sketchbook>/libraries/`. |
| Không có `WiFiClientSecure.h`, `HTTPClient.h` hoặc `cJSON.h` | Chưa chọn board ESP32 hoặc chưa cài gói **esp32 by Espressif Systems**. |
| MQTT TLS báo lỗi chứng chỉ | Host phải nằm trong SAN của chứng chỉ broker và CA nhúng phải đúng CA đã ký. Không tắt xác minh để lách lỗi. |

## Giấy phép

Mỗi thư viện giữ nguyên giấy phép gốc trong thư mục của nó. `PubSubClient` theo MIT (`LICENSE.txt`), `ArduinoMqttClient` theo LGPL 2.1 (`LICENSE.txt`). `LiteBoardSetup` chưa có file giấy phép riêng.
