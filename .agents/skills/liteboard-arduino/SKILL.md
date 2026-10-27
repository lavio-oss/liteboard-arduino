---
name: liteboard-arduino
description: "Use this skill when working on the LiteBoardSetup Arduino library or an ESP32 sketch that sends data to LiteBoard: installing the library, compiling or flashing with arduino-cli, writing a sketch that connects with an Access Token and publishes telemetry, debugging MQTT connection failures, or changing the library itself. Do not use for the LiteBoard server (Laravel, telemetry-service, web), which lives in the lavio-oss/liteboard repository."
metadata:
  author: liteboard
---

# LiteBoard Arduino

This repository ships one Arduino library, `LiteBoardSetup`, for ESP32 boards that talk to LiteBoard over MQTT. Sketches include it as `#include <LiteBoardSetup.h>`. Its only external dependency is `PubSubClient`, installed from the Library Manager and never vendored here.

```text
libraries/LiteBoardSetup/
├── library.properties      name, version, architectures=esp32, depends=PubSubClient
└── src/
    ├── LiteBoardSetup.h
    └── LiteBoardSetup.cpp
```

Reference sketches (`AccessToken`, `AccessTokenNoTls`, `Provisioning`) and their `*.example` config files are in `lavio-oss/liteboard` under `ESP32/`.

## What the library does

All functions are in the `LiteBoardSetup` namespace. None of them opens the MQTT connection; the sketch calls `mqttClient.connect(...)` itself.

| Function | Behaviour |
|---|---|
| `connectWiFi(ssid, password)` | Station mode. Returns at once if already connected, otherwise **blocks until connected**, logging status every 5 s and IP/RSSI at the end. No timeout. |
| `configureMqttTls(tlsClient, mqttClient, caCertificate, host, port, keepAliveSeconds = 60)` | `tlsClient.setCACert(caCertificate)`, then `configureMqtt`. The `PubSubClient` must have been constructed with that `WiFiClientSecure`. |
| `configureMqtt(mqttClient, host, port, keepAliveSeconds = 60)` | `setServer(host, port)` and `setKeepAlive`. Used directly for plaintext. |

## Build and flash with arduino-cli

Prefer `arduino-cli` over describing IDE clicks; it is scriptable and gives exact errors.

```bash
arduino-cli config add board_manager.additional_urls https://espressif.github.io/arduino-esp32/package_esp32_index.json
arduino-cli core update-index
arduino-cli core install esp32:esp32
arduino-cli lib install PubSubClient

# Compile against this repository's library folder without installing it.
arduino-cli compile --libraries ./libraries --fqbn esp32:esp32:esp32 <sketch-folder>
arduino-cli board list                      # find the serial port
arduino-cli upload -p <port> --fqbn esp32:esp32:esp32 <sketch-folder>
arduino-cli monitor -p <port> -c baudrate=115200
```

`esp32:esp32:esp32` is "ESP32 Dev Module"; use the FQBN of the actual board if it differs. To install permanently instead of `--libraries`, copy `libraries/LiteBoardSetup` into `<sketchbook>/libraries/`. The human-facing Arduino IDE steps are in `README.md`.

If `PubSubClient.h` is reported by more than one library, keep a single copy in the sketchbook.

## LiteBoard device contract

A sketch is correct only if it follows these server rules:

- **Credentials:** MQTT username and password are both the device Access Token. Client id must be unique per device (the reference sketches use `ESP32-<efuse MAC>`).
- **Topic:** `liteboard/<project-id>/devices/<device-id>/telemetry`. Heartbeats go to `.../heartbeat`. Any other topic is refused by the broker ACL.
- **Ports:** `8883` with TLS (default, use `configureMqttTls`). `1883` without TLS (use `configureMqtt` with a plain `WiFiClient`) only works when the device has **Allow connection without TLS** enabled in LiteBoard; a token used there must be rotated before real use.
- **TLS:** pass the **public** CA certificate that signed the broker certificate. The host must match a DNS name or IP in that certificate's SAN. Never call `setInsecure()` and never embed a private key.
- **Do not publish retained messages.** Retained telemetry is rejected.
- **Buffer:** call `mqttClient.setBufferSize(512)` (or larger). PubSubClient's default 256 bytes is too small for the topic plus an envelope, and `publish` then fails silently with `false`.
- **Payload size:** at most 64 KB.

### Payload format depends on the device profile

| Device profile in LiteBoard | Payload to send |
|---|---|
| `Default` (no schema fields) | Any non-empty JSON object, e.g. `{"temperature":25.4,"rpm":1500}`. Stored as sent; receive time is used as the timestamp. |
| A profile version with fields | `{"message_id":"<unique, ≤128 chars>","schema_version":<profile version number>,"occurred_at":"<ISO-8601 UTC>","values":{...}}`. Fields are type- and range-checked; with strict mode, unknown keys are rejected. |

For the envelope, sync the clock first (`configTime(0, 0, "pool.ntp.org")`): `occurred_at` more than 5 minutes in the future is rejected. Generate a new `message_id` per message; reusing one is treated as a duplicate.

## Minimal sketch

```cpp
#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <PubSubClient.h>
#include <LiteBoardSetup.h>
#include "device_config.h"   // LB_WIFI_SSID, LB_WIFI_PASSWORD, LB_HOST, LB_PORT, LB_ACCESS_TOKEN, LB_TELEMETRY_TOPIC
#include "mqtt_ca_cert.h"    // LB_MQTT_CA_CERT (public CA only)

WiFiClientSecure wifiClient;
PubSubClient mqttClient(wifiClient);

void setup() {
  Serial.begin(115200);
  LiteBoardSetup::connectWiFi(LB_WIFI_SSID, LB_WIFI_PASSWORD);
  LiteBoardSetup::configureMqttTls(wifiClient, mqttClient, LB_MQTT_CA_CERT, LB_HOST, LB_PORT);
  mqttClient.setBufferSize(512);
}

void loop() {
  while (!mqttClient.connected()) {
    if (!mqttClient.connect("ESP32-example", LB_ACCESS_TOKEN, LB_ACCESS_TOKEN)) {
      Serial.printf("MQTT failed, state=%d\n", mqttClient.state());
      delay(3000);
    }
  }
  mqttClient.loop();

  static unsigned long lastAt = 0;
  if (millis() - lastAt >= 5000) {
    lastAt = millis();
    mqttClient.publish(LB_TELEMETRY_TOPIC, "{\"temperature\":25.4}");  // Default profile format
  }
}
```

Keep `device_config.h` and `mqtt_ca_cert.h` out of Git; only commit `*.example` files with placeholders.

## Debugging a connection

Read the serial log at 115200 baud and `mqttClient.state()`:

| `state()` | Meaning | Check |
|---|---|---|
| `-2` | TCP/TLS connect failed | Host and port reachable; CA is the right one; host is in the certificate SAN; clock is not years off. |
| `-4` / `-3` | Timeout / connection lost | Network quality, keep-alive, `mqttClient.loop()` called often. |
| `4` | Bad username or password | Token copied exactly; token not rotated or revoked. |
| `5` | Not authorized | Device not approved, inactive, or `1883` used without **Allow connection without TLS**. |

If the device connects but no data appears in LiteBoard, the message was rejected at ingest: wrong topic, retained flag, or a payload that does not match the device profile (see the table above).

## Changing the library

- Keep the folder layout `libraries/LiteBoardSetup/{library.properties,src/}` so it remains a valid Arduino library.
- Keep the public API source-compatible; the reference sketches in `lavio-oss/liteboard` call it.
- Bump `version` in `library.properties` for any behaviour change. Keep `architectures=esp32` and `depends=PubSubClient` accurate.
- Do not vendor `PubSubClient` or other Library Manager libraries into this repository.
- Compile at least one TLS and one plaintext sketch with `arduino-cli compile --libraries ./libraries` before committing.
- Keep `README.md` in English and limited to installing the library.
