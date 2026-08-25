# BME280 Environmental Sensor :— Technical Specification

The <strong>BME280</strong> is a precision digital environmental sensor combining relative humidity, barometric pressure, and ambient temperature sensing capabilities in an 8-pin metal-lid Land Grid Array (LGA) package (2.5 mm &times; 2.5 mm &times; 0.93 mm). It communicates over standard I<sup>2</sup>C and SPI digital interfaces with low power consumption for battery-constrained mobile and IoT devices.

<hr/>

## 1. Key Features &amp; Operating Ranges

<ul>
  <li><strong>Sensing Parameters:</strong> Relative humidity, barometric pressure, and ambient temperature.</li>
  <li><strong>Supply Voltage:</strong>
    <ul>
      <li>Main analog supply (V<sub>DD</sub>): 1.71 V to 3.6 V.</li>
      <li>Digital interface supply (V<sub>DDIO</sub>): 1.2 V to 3.6 V.</li>
    </ul>
  </li>
  <li><strong>Low Current Consumption:</strong>
    <ul>
      <li>0.1 &micro;A in Sleep mode.</li>
      <li>1.8 &micro;A @ 1 Hz (Humidity + Temperature).</li>
      <li>3.6 &micro;A @ 1 Hz (Humidity + Pressure + Temperature).</li>
    </ul>
  </li>
  <li><strong>Operating Range:</strong> -40 &deg;C to +85 &deg;C, 0% to 100% relative humidity, 300 hPa to 1100 hPa.</li>
  <li><strong>Digital Interfaces:</strong> I<sup>2</sup>C (up to 3.4 MHz) and SPI (3-wire and 4-wire, up to 10 MHz).</li>
</ul>

<hr/>

## 2. Pin Configuration

| Pin # | Name | Type | Description |
| :---: | :--- | :--- | :--- |
| **1** | `GND` | Supply | Ground connection. |
| **2** | `CSB` | Digital Input | Chip Select (active low). Connect to `V>DDIO` for I<sup>2</sup>C mode. |
| **3** | `SDI` | Digital I/O | Serial Data Input. Functions as `SDA` in I<sup>2</sup>C mode and `MOSI` in SPI mode. |
| **4** | `SCK` | Digital Input | Serial Clock. Functions as `SCL` in I<sup>2</sup>C mode and `SCK` in SPI mode. |
| **5** | `SDO` | Digital Output | Serial Data Output (`MISO` in SPI mode). Sets bit 0 of the I<sup>2</sup>C slave address. |
| **6** | `VDDIO`| Supply | Digital interface power supply (1.2 V to 3.6 V). |
| **7** | `GND` | Supply | Ground connection. |
| **8** | `VDD` | Supply | Main analog power supply (1.71 V to 3.6 V). |

<hr/>

## 3. Absolute Maximum Ratings

> **Caution:** Exceeding any parameter in Table 2 can cause permanent silicon damage. Extended exposure to absolute maximum rating conditions can degrade overall device reliability.

**Table 2: Absolute Maximum Ratings**

| Parameter | Condition | Min | Max | Unit |
| :--- | :--- | :---: | :---: | :---: |
| Supply Voltage (`VDD`, `VDDIO`) | Voltage at any supply pin | -0.3 | 4.25 | V |
| Interface Pin Voltage | Voltage at `SDI`, `SDO`, `SCK`, `CSB` | -0.3 | V<sub>DDIO</sub> + 0.3 | V |
| Storage Temperature | &le; 65% Relative Humidity | -45 | +85 | &deg;C |
| Overpressure Tolerance | Static pressure | 0 | 20,000 | hPa |
| ESD (Human Body Model) | HBM, all pins | &mdash; | &plusmn;2 | kV |
| ESD (Charge Device Model) | CDM | &mdash; | &plusmn;500 | V |
| ESD (Machine Model) | MM | &mdash; | &plusmn;200 | V |

<hr/>

## 4. Digital Interfaces

The BME280 operates as a slave device on both I<sup>2</sup>C and SPI digital buses. The active interface is selected automatically depending on the logic state of the `CSB` pin at startup:

<ul>
  <li><strong>I<sup>2</sup>C mode:</strong> Connect `CSB` directly to the `VDDIO` rail.</li>
  <li><strong>SPI mode:</strong> Drive `CSB` low to initialize SPI communication. Once pulled low, the device remains locked in SPI mode until the next Power-On Reset (POR).</li>
</ul>

### 4.1 I<sup>2</sup>C Interface

The I<sup>2</sup>C interface complies with the Philips I<sup>2</sup>C Specification version 2.1 and supports Standard Mode, Fast Mode, and High-Speed (HS) mode (up to 3.4 MHz).

<ul>
  <li><strong>Bus Lines:</strong>
    <ul>
      <li>`SCK` acts as serial clock line (`SCL`).</li>
      <li>`SDI` acts as serial data line (`SDA`) and requires an external pull-up resistor (typically 4.7 k&Omega;) to `VDDIO`.</li>
    </ul>
  </li>
  <li><strong>Slave Addressing:</strong> The 7-bit device address is defined by the hardware logic level of the `SDO` pin:
    <ul>
      <li>`SDO` tied to `GND`: <code>0x76</code> (<code>1110110b</code>)</li>
      <li>`SDO` tied to `VDDIO`: <code>0x77</code> (<code>1110111b</code>)</li>
    </ul>
  </li>
</ul>

> **Important:** Never leave the `SDO` pin floating, as this results in an undefined I<sup>2</sup>C bus address.

<ul>
  <li><strong>Burst Read Operation:</strong> Always read measurement registers (<code>0xF7</code> through <code>0xFE</code>) in a continuous multi-byte burst read transaction. Individual byte reads can cause race conditions where sensor data updates mid-read, resulting in inconsistent telemetry.</li>
</ul>

### 4.2 SPI Interface

The sensor supports both SPI Mode 00 (<code>CPOL = 0, CPHA = 0</code>) and SPI Mode 11 (<code>CPOL = 1, CPHA = 1</code>) with clock frequencies up to 10 MHz.

<ul>
  <li><strong>4-Wire SPI (Default):</strong>
    <ul>
      <li>`CSB`: Chip Select (active low).</li>
      <li>`SCK`: Serial Clock.</li>
      <li>`SDI`: Serial Data Input / MOSI (latched on the rising edge of `SCK`).</li>
      <li>`SDO`: Serial Data Output / MISO (shifted out on the falling edge of `SCK`).</li>
    </ul>
  </li>
  <li><strong>3-Wire SPI:</strong> Set the <code>spi3w_en</code> bit to <code>1</code> in the <code>config</code> register (<code>0xF5</code>, bit 0) to enable bidirectional data transfer on `SDI`. In this configuration, `SDO` remains in a high-impedance (high-Z) state.</li>
  <li><strong>SPI Addressing Convention:</strong> Bit 7 of the 8-bit SPI transaction header controls read/write access (<code>0</code> for Write, <code>1</code> for Read), followed by the 7-bit target register address.</li>
</ul>