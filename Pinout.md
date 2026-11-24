# Hardware Pinout & Wiring Documentation

## 1. Power Distribution
* **ESP32-S3-WROOM** powered via USB-C (`5V` regulated onboard to `3.3V`).
* **FC-51 IR Sensor**, **GY-31 Color Sensor**, and **ILI9341 TFT LCD** share a common `3.3V` rail and common `GND`.

## 2. Signal Connections

### FC-51 Infrared Sensor (`U4`)
* `VCC` $\rightarrow$ `3.3V`
* `GND` $\rightarrow$ `GND`
* `OUT` $\rightarrow$ `GPIO 14`

### GY-31 / TCS3200 Color Sensor (`U3`)
* `VCC` $\rightarrow$ `3.3V`
* `GND` $\rightarrow$ `GND`
* `S0` $\rightarrow$ `GPIO 19`
* `S1` $\rightarrow$ `GPIO 20`
* `S2` $\rightarrow$ `GPIO 21`
* `S3` $\rightarrow$ `GPIO 4`
* `OUT` $\rightarrow$ `GPIO 5`
* `LED` $\rightarrow$ `GPIO 6`

### ILI9341 3.2" SPI TFT Display (`U2`)
* `VCC` & `LED` $\rightarrow$ `3.3V`
* `GND` $\rightarrow$ `GND`
* `D/C` $\rightarrow$ `GPIO 1`
* `CS` $\rightarrow$ `GPIO 2`
* `RST` $\rightarrow$ `GPIO 3`
* `SCK` $\rightarrow$ `GPIO 45`
* `MOSI` $\rightarrow$ `GPIO 47`
* `MISO` $\rightarrow$ `GPIO 48`