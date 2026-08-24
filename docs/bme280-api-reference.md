# BME280 Embedded C-Driver API Reference

This document details the C-language Application Programming Interface (API) for configuring, initializing, and acquiring telemetry from the BME280 environmental sensor.

---

## 1. Data Structures & Types

### 1.1 Sensor Operating Modes (`bme280_mode_t`)
Defines the power state of the sensor.

```c
typedef enum {
    BME280_MODE_SLEEP  = 0x00, /* Low-power sleep mode (default at reset) */
    BME280_MODE_FORCED = 0x01, /* Single-measurement mode, returns to sleep */
    BME280_MODE_NORMAL = 0x03  /* Continuous cyclic measurement mode */
} bme280_mode_t;

```

### 1.2 Oversampling Options (`bme280_osrs_t`)

Configures noise reduction and resolution for temperature, pressure, and humidity channels.

```c
typedef enum {
    BME280_OVERSAMPLING_SKIPPED = 0x00, /* Channel output skipped (0x80000) */
    BME280_OVERSAMPLING_1X      = 0x01,
    BME280_OVERSAMPLING_2X      = 0x02,
    BME280_OVERSAMPLING_4X      = 0x03,
    BME280_OVERSAMPLING_8X      = 0x04,
    BME280_OVERSAMPLING_16X     = 0x05
} bme280_osrs_t;

```

### 1.3 Telemetry Output Struct (`bme280_data_t`)

Stores compensated physical measurement data.

```c
typedef struct {
    float temperature; /* Temperature in degrees Celsius (°C) */
    float pressure;    /* Barometric pressure in hectopascals (hPa) */
    float humidity;    /* Relative humidity in percentage (%RH) */
} bme280_data_t;

```

---

## 2. API Functions

### `bme280_init`

Initializes the sensor interface, verifies the silicon Chip ID, and loads non-volatile factory calibration parameters.

```c
int8_t bme280_init(uint8_t i2c_addr);

```

* **Parameters:**
* `i2c_addr`: I2C slave address (`0x76` or `0x77`).


* **Returns:**
* `0`: Success.
* `-1`: Communication error / device not responding.
* `-2`: Invalid Chip ID (expected `0x60`).



---

### `bme280_set_config`

Applies oversampling, filter coefficients, and inactive standby duration.

```c
int8_t bme280_set_config(bme280_osrs_t osrs_h, 
                         bme280_osrs_t osrs_p, 
                         bme280_osrs_t osrs_t, 
                         bme280_mode_t mode);

```

* **Parameters:**
* `osrs_h`: Humidity oversampling setting.
* `osrs_p`: Pressure oversampling setting.
* `osrs_t`: Temperature oversampling setting.
* `mode`: Target operational mode (`BME280_MODE_NORMAL` or `BME280_MODE_FORCED`).


* **Returns:**
* `0`: Success.
* `-1`: Write failure on digital bus.



> **Note:** The driver writes `ctrl_hum` (`0xF2`) prior to `ctrl_meas` (`0xF4`) to ensure humidity configuration changes latch properly into internal logic.

---

### `bme280_read_data`

Executes a multi-byte burst read to acquire raw ADC data, then applies factory trimming formulas to compute compensated physical values.

```c
int8_t bme280_read_data(bme280_data_t *data);

```

* **Parameters:**
* `data`: Pointer to a `bme280_data_t` structure to store results.


* **Returns:**
* `0`: Success.
* `-1`: Bus read failure.



---

## 3. Worked Example: Initialization & Cyclic Readout

```c
#include <stdio.h>
#include "bme280.h"

int main(void) {
    bme280_data_t sensor_data;

    /* 1. Initialize sensor at default address 0x76 */
    if (bme280_init(0x76) != 0) {
        printf("Error: Failed to identify BME280 sensor.\n");
        return -1;
    }

    /* 2. Configure for standard weather monitoring: 1x oversampling, normal mode */
    bme280_set_config(BME280_OVERSAMPLING_1X, 
                      BME280_OVERSAMPLING_1X, 
                      BME280_OVERSAMPLING_1X, 
                      BME280_MODE_NORMAL);

    /* 3. Continuously acquire compensated telemetry */
    while (1) {
        if (bme280_read_data(&sensor_data) == 0) {
            printf("Temp: %.2f °C | Press: %.2f hPa | Humidity: %.2f %%\n",
                   sensor_data.temperature, 
                   sensor_data.pressure, 
                   sensor_data.humidity);
        }
        sleep_ms(1000);
    }

    return 0;
}

```


