# Cashave — IoT Smart Cash Savings & Monitoring System

An **ESP32-S3**-based smart piggy bank that identifies Indonesian Rupiah (IDR) paper banknotes using an **FC-51 Infrared trigger**, **GY-31 (TCS3200) RGB color sensor**, and an on-device **K-Nearest Neighbor (KNN)** classifier. Savings balances, transaction history, and personalized savings goals are displayed locally on a **3.2" ILI9341 TFT LCD** and synchronized in real time with **Firebase Realtime Database** and a companion mobile application.

---

## 1. Key Features

* **Automated Banknote Detection & Denomination Classification:** Uses an FC-51 IR sensor to wake/trigger RGB sampling on the GY-31 sensor, averaging 10 consecutive readings before running a Euclidean-distance **KNN classifier** to identify IDR banknote denominations.
* **Dual-Core FreeRTOS Architecture:** Decouples real-time hardware sensing (`Core 1`) from network/cloud synchronization (`Core 0`) via a thread-safe **FreeRTOS Queue** (`msgQueue`), keeping the UI responsive even during network latency.
* **Local TFT Dashboard:** Displays current balance, detected banknote RGB telemetry, and savings goal progress bars on a 3.2-inch ILI9341 SPI TFT display.
* **Real-Time Cloud & Mobile App Sync:** Pushes balance updates and transaction logs to **Firebase Realtime Database**, enabling users to track balances and manage wishlist goals from their smartphone.

---

## 2. System Architecture & Dual-Core FreeRTOS Design

```text
┌───────────────────┐     Trigger     ┌───────────────────────────────┐
│  FC-51 IR Sensor  │ ──────────────► │   TaskSensing (ESP32 Core 1)  │
└───────────────────┘                 │  • Enable GY-31 White LEDs    │
                                      │  • Sample 10x RGB & Average   │
┌───────────────────┐    RGB Data     │  • Run KNN Classification     │
│ GY-31 (TCS3200)   │ ──────────────► │  • Update ILI9341 TFT Display │
└───────────────────┘                 └───────────────┬───────────────┘
                                                      │ xQueueSend(msgQueue)
                                                      ▼
┌───────────────────┐   Real-Time     ┌───────────────────────────────┐
│   Cashave Mobile  │ ◄─────────────► │  TaskFirebase (ESP32 Core 0)  │
│    Application    │     Sync        │  • xQueueReceive(msgQueue)    │
└─────────┬─────────┘                 │  • Fetch Initial Balance/Goals│
          │                           │  • Push Updated Saldo/History │
          ▼                           └───────────────┬───────────────┘
┌─────────────────────────────────────────────────────▼───────────────┐
│                     Firebase Realtime Database                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Task Breakdown
1. **`TaskSensing` (Pinned to Core 1):**
   * Polls the FC-51 IR sensor for banknote insertion.
   * Activates the GY-31 illumination LEDs, collects **10 RGB samples**, and computes the mean $(R, G, B)$ vector to minimize ambient light noise.
   * Computes Euclidean distance against calibrated IDR currency training samples using **KNN** to classify the nominal value.
   * Updates the ILI9341 TFT LCD and dispatches the detected amount via `xQueueSend(msgQueue, &nominal)`.
2. **`TaskFirebase` (Pinned to Core 0):**
   * Monitors Wi-Fi connectivity and listens on `xQueueReceive(msgQueue, &receivedNominal)`.
   * Synchronizes the updated balance (`saldo`), transaction history, and goal targets with **Firebase Realtime Database** without blocking hardware sensing.

---

## 3. Hardware Components & ESP32-S3 Pinout

### Bill of Materials (BOM)
* **Microcontroller:** Freenove ESP32-S3-WROOM CAM (N16R8)
* **Color Recognition Sensor:** GY-31 (TCS3200) Programmable Color Light-to-Frequency Converter
* **Object Detection Sensor:** FC-51 Infrared Obstacle Avoidance Sensor
* **Display:** 3.2-inch ILI9341 SPI TFT LCD

### GPIO Wiring Mapping (Derived from Schematic)

| Peripheral Module | Module Pin | ESP32-S3 GPIO | Notes |
| :--- | :---: | :---: | :--- |
| **FC-51 IR Sensor** | `OUT` | `GPIO 14` | Digital trigger input (`VCC`: 3.3V, `GND`: GND) |
| **GY-31 Color Sensor** | `S0` | `GPIO 19` | Frequency scaling control |
| **GY-31 Color Sensor** | `S1` | `GPIO 20` | Frequency scaling control |
| **GY-31 Color Sensor** | `S2` | `GPIO 21` | Photodiode color filter select |
| **GY-31 Color Sensor** | `S3` | `GPIO 4` | Photodiode color filter select |
| **GY-31 Color Sensor** | `OUT` | `GPIO 5` | Frequency output (`pulseIn` measurement) |
| **GY-31 Color Sensor** | `LED` | `GPIO 6` | White LED illumination control |
| **ILI9341 TFT LCD** | `D/C` | `GPIO 1` | Data / Command select |
| **ILI9341 TFT LCD** | `CS` | `GPIO 2` | SPI Chip Select |
| **ILI9341 TFT LCD** | `RST` | `GPIO 3` | Hardware Reset |
| **ILI9341 TFT LCD** | `SCK` | `GPIO 45` | SPI Clock |
| **ILI9341 TFT LCD** | `MOSI` | `GPIO 47` | SPI Master Out Slave In |
| **ILI9341 TFT LCD** | `MISO` | `GPIO 48` | SPI Master In Slave Out |

---

## 4. Software Stack & Libraries

* **IDE & Build System:** Visual Studio Code + PlatformIO
* **RTOS:** FreeRTOS (built-in ESP-IDF / Arduino-ESP32 framework)
* **Cloud Backend:** Firebase Realtime Database (`FirebaseESP32` library)
* **Display Drivers:** `Adafruit_ILI9341` & `Adafruit_GFX`

---

## 5. Firebase Realtime Database Structure

```json
{
  "cashave": {
    "saldo": 560000,
    "history": {
      "-txnId1": {
        "nominal": 10000,
        "timestamp": "2026-01-15T10:20:00Z"
      }
    }
  },
  "targets": {
    "-OuI8VbsclrXLMsTUYNF": {
      "nama": "Mac mini",
      "harga": 100000
    },
    "-OuNizs2wprUhl10LQe7": {
      "nama": "denim",
      "harga": 10000
    }
  }
}
```

---

## 6. Current Limitations & Future Roadmap

* **Current Scope & Constraints:**
  * Supports Indonesian paper banknotes only (coins are not supported).
  * Relies on RGB color classification; ambient lighting or folded banknotes can affect accuracy, and counterfeit verification is not performed.
  * Detects insertion at the slot but does not verify physical drop into the lower vault.
* **Planned Enhancements:**
  * Upgrade from RGB color sensing to **ESP32-S3 CAM computer vision / TinyML** classification for higher accuracy and counterfeit pattern checks.
  * Add a secondary break-beam drop sensor inside the storage chamber to confirm successful deposit.

---
