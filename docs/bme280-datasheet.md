# BME280 Environmental Sensor — Technical Specification

The **BME280** is a precision digital environmental sensor combining relative humidity, barometric pressure, and ambient temperature sensing capabilities in an 8-pin metal-lid Land Grid Array (LGA) package ($2.5 \text{ mm} \times 2.5 \text{ mm} \times 0.93 \text{ mm}$)[cite: 1]. It communicates over standard I2C and SPI digital interfaces with low power consumption for battery-constrained mobile and IoT devices[cite: 1].

---

## 1. Key Features & Operating Ranges

* **Sensing Parameters:** Relative humidity, barometric pressure, and ambient temperature[cite: 1].
* **Supply Voltage:**
  * Main analog supply ($V_{DD}$): $1.71\text{ V}$ to $3.6\text{ V}$[cite: 1].
  * Digital interface supply ($V_{DDIO}$): $1.2\text{ V}$ to $3.6\text{ V}$[cite: 1].
* **Low Current Consumption:**
  * $0.1\ \mu\text{A}$ in Sleep mode[cite: 1].
  * $1.8\ \mu\text{A}$ @ 1 Hz (Humidity + Temperature)[cite: 1].
  * $3.6\ \mu\text{A}$ @ 1 Hz (Humidity + Pressure + Temperature)[cite: 1].
* **Operating Range:** $-40^\circ\text{C}$ to $+85^\circ\text{C}$, $0\%$ to $100\%$ relative humidity, $300\text{ hPa}$ to $1100\text{ hPa}$[cite: 1].
* **Digital Interfaces:** I2C (up to $3.4\text{ MHz}$) and SPI (3-wire and 4-wire, up to $10\text{ MHz}$)[cite: 1].

---

## 2. Pin Configuration

| Pin # | Name | Type | Description |
| :---: | :--- | :--- | :--- |
| **1** | `GND` | Supply | Ground connection[cite: 1]. |
| **2** | `CSB` | Digital Input | Chip Select (active low)[cite: 1]. Connect to `VDDIO` for I2C interface mode[cite: 1]. |
| **3** | `SDI` | Digital I/O | Serial Data Input[cite: 1]. Functions as `SDA` in I2C mode and `MOSI` in SPI mode[cite: 1]. |
| **4** | `SCK` | Digital Input | Serial Clock[cite: 1]. Functions as `SCL` in I2C mode and `SCK` in SPI mode[cite: 1]. |
| **5** | `SDO` | Digital Output | Serial Data Output (`MISO` in SPI mode)[cite: 1]. Sets bit 0 of the I2C slave address[cite: 1]. |
| **6** | `VDDIO`| Supply | Digital interface power supply ($1.2\text{ V}$ to $3.6\text{ V}$)[cite: 1]. |
| **7** | `GND` | Supply | Ground connection[cite: 1]. |
| **8** | `VDD` | Supply | Main analog power supply ($1.71\text{ V}$ to $3.6\text{ V}$)[cite: 1]. |

---

## 3. Absolute Maximum Ratings

> **Caution:** Exceeding any parameter in Table 2 can cause permanent silicon damage[cite: 1]. Extended exposure to absolute maximum rating conditions can degrade overall device reliability[cite: 1].

**Table 2: Absolute Maximum Ratings**

| Parameter | Condition | Min | Max | Unit |
| :--- | :--- | :---: | :---: | :---: |
| Supply Voltage (`VDD`, `VDDIO`) | Voltage at any supply pin[cite: 1] | -0.3[cite: 1] | 4.25[cite: 1] | V[cite: 1] |
| Interface Pin Voltage | Voltage at `SDI`, `SDO`, `SCK`, `CSB`[cite: 1] | -0.3[cite: 1] | VDDIO + 0.3[cite: 1] | V[cite: 1] |
| Storage Temperature | $\le 65\%$ Relative Humidity[cite: 1] | -45[cite: 1] | +85[cite: 1] | °C[cite: 1] |
| Overpressure Tolerance | Static pressure[cite: 1] | 0[cite: 1] | 20,000[cite: 1] | hPa[cite: 1] |
| ESD (Human Body Model) | HBM, all pins[cite: 1] | — | $\pm 2$[cite: 1] | kV[cite: 1] |
| ESD (Charge Device Model) | CDM[cite: 1] | — | $\pm 500$[cite: 1] | V[cite: 1] |
| ESD (Machine Model) | MM[cite: 1] | — | $\pm 200$[cite: 1] | V[cite: 1] |

---

## 4. Digital Interfaces

The BME280 operates as a slave device on both I2C and SPI digital buses[cite: 1]. The active interface is selected automatically depending on the logic state of the `CSB` pin at startup[cite: 1]:

* **I2C mode:** Connect `CSB` directly to the `VDDIO` rail[cite: 1].
* **SPI mode:** Drive `CSB` low to initialize SPI communication[cite: 1]. Once pulled low, the device remains locked in SPI mode until the next Power-On Reset (POR)[cite: 1].

### 4.1 I2C Interface

The I2C interface complies with the Philips I2C Specification version 2.1 and supports Standard Mode, Fast Mode, and High-Speed (HS) mode (up to 3.4 MHz)[cite: 1].

* **Bus Lines:**
  * `SCK` acts as serial clock line (`SCL`)[cite: 1].
  * `SDI` acts as serial data line (`SDA`) and requires an external pull-up resistor (typically $4.7\text{ k}\Omega$) to `VDDIO`[cite: 1].
* **Slave Addressing:** The 7-bit device address is defined by the hardware logic level of the `SDO` pin[cite: 1]:
  * `SDO` tied to `GND`: `0x76` (`1110110b`)[cite: 1]
  * `SDO` tied to `VDDIO`: `0x77` (`1110111b`)[cite: 1]

> **Important:** Never leave the `SDO` pin floating, as this results in an undefined I2C bus address[cite: 1].

* **Burst Read Operation:** Always read measurement registers (`0xF7` through `0xFE`) in a continuous multi-byte burst read transaction[cite: 1]. Individual byte reads can cause race conditions where sensor data updates mid-read, resulting in inconsistent telemetry[cite: 1].

### 4.2 SPI Interface

The sensor supports both SPI Mode 00 (`CPOL = 0, CPHA = 0`) and SPI Mode 11 (`CPOL = 1, CPHA = 1`) with clock frequencies up to 10 MHz[cite: 1].

* **4-Wire SPI (Default):**
  * `CSB`: Chip Select (active low)[cite: 1].
  * `SCK`: Serial Clock[cite: 1].
  * `SDI`: Serial Data Input / MOSI (latched on the rising edge of `SCK`)[cite: 1].
  * `SDO`: Serial Data Output / MISO (shifted out on the falling edge of `SCK`)[cite: 1].
* **3-Wire SPI:** Set the `spi3w_en` bit to `1` in the `config` register (`0xF5`, bit 0) to enable bidirectional data transfer on `SDI`[cite: 1]. In this configuration, `SDO` remains in a high-impedance (high-Z) state[cite: 1].
* **SPI Addressing Convention:** Bit 7 of the 8-bit SPI transaction header controls read/write access (`0` for Write, `1` for Read), followed by the 7-bit target register address[cite: 1].