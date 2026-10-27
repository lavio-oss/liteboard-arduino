# LiteBoardSetup

Thư viện Arduino `LiteBoardSetup` cho thiết bị ESP32 kết nối [LiteBoard](https://github.com/lavio-oss/liteboard). Sketch LiteBoard dùng nó qua:

```cpp
#include <LiteBoardSetup.h>
```

Thư viện gom phần kết nối Wi-Fi và cấu hình MQTT cho sketch. Nó phụ thuộc `PubSubClient`, thư viện có sẵn trong Library Manager của Arduino IDE nên repo này không đóng gói lại.

```text
libraries/
└── LiteBoardSetup/
    ├── library.properties
    └── src/
        ├── LiteBoardSetup.h
        └── LiteBoardSetup.cpp
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

Gói này cung cấp sẵn `WiFi` và `WiFiClientSecure` mà `LiteBoardSetup` dùng.

### 2. Cài `PubSubClient` từ Library Manager

Mở **Tools → Manage Libraries…** (hoặc **Sketch → Include Library → Manage Libraries…**), tìm `PubSubClient` của **Nick O'Leary** và bấm **Install**.

### 3. Chép `LiteBoardSetup` vào Sketchbook

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

3. Chép thư mục `libraries/LiteBoardSetup` vào thư mục `libraries` của Sketchbook. Kết quả:

   ```text
   <Sketchbook>/libraries/LiteBoardSetup/library.properties
   ```

   Nếu thư mục `libraries` chưa có, hãy tạo nó. Chép thư mục `LiteBoardSetup`, không chép nguyên thư mục `libraries/` của repo thành một lớp lồng thêm.

4. **Thoát hẳn và mở lại Arduino IDE** để IDE quét thư viện mới.

### 4. Kiểm tra thư viện đã nhận

Mở **Sketch → Include Library**. Bạn phải thấy `LiteBoardSetup` trong phần **Contributed libraries**, và `PubSubClient` trong danh sách thư viện đã cài. Nếu thiếu, kiểm tra lại đường dẫn Sketchbook và cấu trúc thư mục ở bước 3.

Sketch LiteBoard và hướng dẫn cấu hình thiết bị nằm ở repo chính [lavio-oss/liteboard](https://github.com/lavio-oss/liteboard), thư mục `ESP32/`.
