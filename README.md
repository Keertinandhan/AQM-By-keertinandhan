# ESP32 Air Quality Monitor — Track B (Wokwi)

Firmware prototype for the **Aerion Technology Labs Engineering Intern** practical task, built using ESP32 Arduino C++ and the Wokwi online simulator.

**Wokwi project:** https://wokwi.com/projects/479947453409433601

> This prototype measures the MPU6050's **internal temperature** and generates **simulated humidity**. It is not a calibrated ambient air-quality monitor and does not measure pollutants.

## Track and hardware

- **Track:** B — no physical ESP32 required
- **Controller:** ESP32 DevKit, Arduino framework
- **Sensor:** MPU6050 on I2C address `0x68`
- **Serial baud:** `115200`
- **Default sampling interval:** 2 seconds; configurable from 1–3600 seconds
- **Filter:** moving average over the most recent five valid samples
- **BLE:** compiled out in Wokwi (`ENABLE_BLE 0`), implementation included for physical ESP32 (`ENABLE_BLE 1`)

## Wiring

| ESP32 | MPU6050 | Purpose |
|---|---|---|
| 3V3 | VCC | Power |
| GND | GND | Ground |
| GPIO 21 | SDA | I2C data |
| GPIO 22 | SCL | I2C clock |

`diagram.json` is a reconstructed wiring configuration based on the project screenshot; compare it with the original Wokwi `diagram.json` before uploading.

## How the sensor reading works

1. Initialize I2C with `Wire.begin(21, 22)`.
2. Probe the MPU6050 at `0x68`.
3. Write `0x00` to power-management register `0x6B` to wake the sensor.
4. Point to register `0x41`, then read two bytes (`TEMP_OUT_H` and `TEMP_OUT_L`).
5. Combine them into a signed 16-bit value and convert using `temperature_C = raw / 340.0 + 36.53`.
6. Simulate humidity via the clearly named `readSimulatedHumidity()` function (40.0–60.0%).
7. Push readings into circular buffers and average up to five recent samples.

The first four averages use the number of samples available so far. Once the buffer fills, the oldest sample is overwritten each interval.

## Run in Wokwi

1. Open the [public Wokwi project](https://wokwi.com/projects/479947453409433601).
2. Use `sketch.ino` with `#define ENABLE_BLE 0`.
3. Start simulation and open the Serial Monitor at 115200 baud.
4. Observe automatically printed filtered temperature and humidity values.
5. Enter each command followed by Enter:

```text
GET TEMP
GET HUM
SET INTERVAL 5
GET TEMP
GET HUM
```

The expected confirmation for the third command is `Sampling interval updated to 5 seconds`. Subsequent automatic reports should occur about every five seconds of simulation time.

## BLE GATT specification (physical ESP32 only)

**Advertised name:** `AQM-Keertinandhan`

| Type | UUID | Properties | Value encoding |
|---|---|---|---|
| Custom service | `12345678-1234-1234-1234-123456789000` | Service | — |
| Temperature | `12345678-1234-1234-1234-123456789001` | Read, Notify | UTF-8 decimal Celsius, e.g. `24.00` |
| Humidity | `12345678-1234-1234-1234-123456789002` | Read, Notify | UTF-8 decimal percent, e.g. `50.20` |
| Sampling interval | `12345678-1234-1234-1234-123456789003` | Read, Write | UTF-8 integer seconds, e.g. `5` |

Temperature and humidity have client-configuration descriptors (`BLE2902`) to support notification subscriptions. Writing a valid interval changes the firmware's sampling period. Accepted range: 1–3600 seconds.

## Build for a physical ESP32

1. Open `sketch.ino` in an Arduino ESP32 environment with the ESP32 board package installed.
2. Change `#define ENABLE_BLE 0` to `#define ENABLE_BLE 1`.
3. Select the matching ESP32 board and port, then compile and upload.
4. Connect the MPU6050 as shown above.
5. In nRF Connect, find `AQM-Keertinandhan`, read temperature/humidity, enable notifications, and write a UTF-8 decimal string (for example `5`) to the interval characteristic.

**Important:** BLE compilation and behavior on a physical ESP32 have not yet been verified. ESP32 Arduino core versions can differ in BLE API signatures; check compilation and adapt the callback value type if necessary. This is planned for the review session.

## Testing and evidence
<img width="1920" height="972" alt="Screenshot 2026-11-06 at 19-15-43 AQM-keertinandhan - Wokwi ESP32 STM32 Arduino Simulator" src="https://github.com/user-attachments/assets/f9715475-4337-4e95-8bf2-a41ec69cac0b" />


- **Observed:** Wokwi screenshot shows repeated filtered temperature and simulated humidity reports, with the filter reaching five samples.
- **To capture before submission:** screenshot showing `GET TEMP`, `GET HUM`, `SET INTERVAL 5`, the confirmation, and subsequent automatic reports.
- **Not tested:** real BLE connection, notifications, and characteristic writes on hardware.

Add the final Serial Monitor evidence image to `screenshots/` and update this section with its filename.

## Limitations

- Wokwi does not emulate BLE, so BLE is disabled during simulation.
- Humidity is simulated, not measured by the MPU6050.
- MPU6050 internal temperature is not a calibrated ambient air-temperature measurement.
- I2C read errors are reported but not retried; the sensor probe stops the program if the device is absent.
- Notification/connection behavior and BLE library compatibility need physical-board validation.
- No air-pollutant sensor is present, so the project does not calculate an air-quality index.

## Project files

- `sketch.ino` — sensor reading, filtering, serial commands and conditional BLE GATT implementation
- `diagram.json` — reconstructed Wokwi wiring; verify against original project
- `AI_USAGE.md` — AI usage disclosure (review and personalize before submission)
- `screenshots/` — testing evidence

## Author

Keertinandhan — Engineering Intern practical task submission.
