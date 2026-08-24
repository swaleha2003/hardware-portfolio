
# Board Bring-Up Guide: Interfacing BME280 with ESP32-S3 via I2C

This guide provides step-by-step instructions to wire, flash, and verify I2C communication between an **ESP32-S3 DevKit** and a **BME280 Environmental Sensor** using the ESP-IDF framework.

---

## 1. Prerequisites

### 1.1 Hardware Requirements
* 1x ESP32-S3 DevKit-C board.
* 1x BME280 breakout module.
* 4x Female-to-female jumper wires.
* 1x USB-C communication cable.
* 2x $4.7\text{ k}\Omega$ pull-up resistors (if not integrated on the sensor breakout board).

### 1.2 Software Requirements
* **Toolchain:** ESP-IDF v5.x installed and sourced in your terminal environment.
* **Serial Terminal:** `idf.py monitor`, PuTTY, or Minicom (configured to 115200 baud).

---

## 2. Hardware Wiring & Pin Mapping

Connect the sensor module to the microcontroller development board according to the pin connections below:

| BME280 Sensor Pin | ESP32-S3 GPIO Pin | Function | Notes |
| :--- | :--- | :--- | :--- |
| `VDD` / `VIN` | `3V3` | Power Supply | Nominal 3.3 V DC rail. |
| `GND` | `GND` | Common Ground | System reference ground. |
| `SCK` / `SCL` | `GPIO 9` | I2C Clock | Pull up to 3.3 V via 4.7 kΩ. |
| `SDI` / `SDA` | `GPIO 8` | I2C Data | Pull up to 3.3 V via 4.7 kΩ. |
| `CSB` | `3V3` | Bus Mode Select | Tied to 3.3 V to enforce I2C mode. |
| `SDO` | `GND` | Address Select | Tied to GND sets I2C address to `0x76`. |

> **Warning:** Do not connect the BME280 `VDD` pin to the `5V` rail of the ESP32 board. Applying 5 V will permanently damage the silicon.

---

## 3. Step-by-Step Bring-Up Procedure

### Step 1: Set Up the Target Environment
Open your terminal and configure the build target for the ESP32-S3 architecture:

```bash
cd ~/esp/bme280_telemetry
idf.py set-target esp32s3

```

### Step 2: Configure I2C Peripherals

Open the project configuration menu to verify peripheral clock speeds and pin assignments:

```bash
idf.py menuconfig

```

1. Navigate to **Component config** $\rightarrow$ **Driver configurations** $\rightarrow$ **I2C Configuration**.
2. Set **I2C Master Port** to `I2C_NUM_0`.
3. Set **SDA GPIO Pin** to `8` and **SCL GPIO Pin** to `9`.
4. Set **I2C Clock Speed** to `100000` (100 kHz Standard Mode).
5. Save changes (`S`) and exit (`Q`).

### Step 3: Build the Firmware

Compile the application binary and partition table:

```bash
idf.py build

```

Verify that the build outputs `Project build complete` with zero compilation errors.

### Step 4: Flash and Monitor Output

Hold the `BOOT` button on the ESP32-S3 board, connect the USB cable, and execute the flash command:

```bash
idf.py -p /dev/ttyUSB0 flash monitor

```

---

## 4. Expected Console Telemetry

Upon boot, the ESP32 initializes the I2C driver, verifies the sensor Chip ID (`0x60`), and streams cyclic environmental measurements:

```text
I (312) main: Initializing I2C bus at port 0 (SDA: GPIO 8, SCL: GPIO 9)...
I (322) bme280: Probing device at address 0x76...
I (330) bme280: Device identified successfully. Chip ID: 0x60
I (335) bme280: Calibration registers loaded into memory.
I (1340) telemetry: [T: 24.85 °C] [P: 1013.25 hPa] [H: 48.20 %RH]
I (2340) telemetry: [T: 24.86 °C] [P: 1013.22 hPa] [H: 48.23 %RH]
I (3340) telemetry: [T: 24.85 °C] [P: 1013.28 hPa] [H: 48.18 %RH]

```

---

## 5. Troubleshooting Common Failures

| Symptom / Error | Root Cause | Corrective Action |
| --- | --- | --- |
| `ESP_ERR_TIMEOUT: I2C bus timeout` | Missing pull-up resistors or open lines. | Check physical jumper wires on `GPIO 8` and `GPIO 9`. Verify 4.7 kΩ pull-ups to 3.3 V. |
| `Invalid Chip ID: 0xFF` or `0x00` | SDO pin floating or wrong I2C address. | Verify `SDO` is securely tied to `GND` for `0x76`. If tied to `3V3`, change target address to `0x77`. |
| `A fatal error occurred: Failed to connect to ESP32` | Bootloader strap pin not asserted. | Press and hold `BOOT`, tap `RESET`, release `BOOT`, then re-run `idf.py flash`. |
