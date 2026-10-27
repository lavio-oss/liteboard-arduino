# LiteBoardSetup

`LiteBoardSetup` is an Arduino library for ESP32 devices that connect to [LiteBoard](https://github.com/lavio-oss/liteboard). LiteBoard sketches use it through:

```cpp
#include <LiteBoardSetup.h>
```

The library handles the Wi-Fi connection and MQTT setup for the sketches. It depends on `PubSubClient`, which is available in the Arduino IDE Library Manager, so this repository does not bundle it.

```text
libraries/
└── LiteBoardSetup/
    ├── library.properties
    └── src/
        ├── LiteBoardSetup.h
        └── LiteBoardSetup.cpp
```

## Installing in the Arduino IDE

These steps are for Arduino IDE 2.x. Arduino IDE 1.8.x works the same way and uses the same menu names.

### 1. Install ESP32 board support

1. Open **File → Preferences** (macOS: **Arduino IDE → Settings**).
2. In **Additional boards manager URLs**, add the following line and click **OK**:

   ```text
   https://espressif.github.io/arduino-esp32/package_esp32_index.json
   ```

3. Open **Tools → Board → Boards Manager**, search for `esp32` and install **esp32 by Espressif Systems**.

This package provides `WiFi` and `WiFiClientSecure`, which `LiteBoardSetup` uses.

### 2. Install `PubSubClient` from the Library Manager

Open **Tools → Manage Libraries…** (or **Sketch → Include Library → Manage Libraries…**), search for `PubSubClient` by **Nick O'Leary** and click **Install**.

### 3. Copy `LiteBoardSetup` into your sketchbook

1. In **File → Preferences**, note the **Sketchbook location**. The defaults are:

   | Operating system | Path |
   |---|---|
   | Windows | `C:\Users\<your name>\Documents\Arduino` |
   | macOS | `~/Documents/Arduino` |
   | Linux | `~/Arduino` |

2. Download this repository (**Code → Download ZIP**, then extract it) or clone it:

   ```bash
   git clone https://github.com/lavio-oss/LiteBoardSetup.git
   ```

3. Copy the `libraries/LiteBoardSetup` folder into the `libraries` folder of your sketchbook. The result should be:

   ```text
   <Sketchbook>/libraries/LiteBoardSetup/library.properties
   ```

   Create the `libraries` folder if it does not exist. Copy the `LiteBoardSetup` folder itself; do not copy the repository's whole `libraries/` folder, which would add an extra nested level.

4. **Quit and reopen the Arduino IDE** so it picks up the new library.

### 4. Check that the library is installed

Open **Sketch → Include Library**. `LiteBoardSetup` should appear under **Contributed libraries**, and `PubSubClient` should be in the list of installed libraries. If either is missing, check the sketchbook path and the folder structure from step 3.

The LiteBoard sketches and the device configuration guide live in the main repository, [lavio-oss/liteboard](https://github.com/lavio-oss/liteboard), in the `ESP32/` folder.
