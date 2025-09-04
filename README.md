# ESP32 Fardriver BLE Controller Reader

## Overview

This Arduino sketch enables an ESP32 to connect to a Fardriver controller via Bluetooth Low Energy (BLE), receive and decode real-time data, and output it to the Serial Monitor. It is designed to monitor key metrics from an electric bike or scooter, such as voltage, current, power, RPM, gear, speed, controller temperature, motor temperature, state of charge (SOC), and regeneration status. Regeneration (regen) status is identified based on negative current readings.

## Functionality

- **BLE Connection**: Connects to a Fardriver controller using a specified BLE UUID for both service and characteristic.
- **Data Parsing**: Decodes data packets from the controller, extracting metrics like:
  - Voltage (V)
  - Line Current (A, range: -100 to 300)
  - Power (W)
  - RPM
  - Gear
  - Speed (km/h)
  - Controller Temperature (°C, up to 100°C)
  - Motor Temperature (°C, up to 200°C)
  - State of Charge (SOC, %)
  - Regeneration Status (based on negative current)
- **Serial Output**: Prints the parsed data to the Serial Monitor every 500ms for easy monitoring and debugging.
- **Minimal Dependencies**: Uses only the Arduino core and ESP32 BLE libraries, making it lightweight and portable.

## How It Works

1. **Initialization**: The ESP32 initializes as a BLE client and starts scanning for a device advertising the specified UUID.
2. **Connection**: Upon finding the Fardriver controller, it connects and subscribes to notifications from the specified characteristic UUID.
3. **Data Reception**: The controller sends data packets, which are received via BLE notifications.
4. **Packet Processing**: Each packet is validated using a CRC check, then parsed based on its address to update the relevant data fields (e.g., voltage, current, RPM).
5. **Speed Calculation**: Speed is calculated from RPM using the wheel circumference and motor pole pairs, which must be configured for accuracy.
6. **Regeneration Detection**: Negative line current (up to -100A) indicates regen mode, which is reported in the Serial output.
7. **Serial Monitoring**: Every 500ms, the parsed data is formatted and printed to the Serial Monitor for real-time monitoring.

## Configuration Notes

- **BLE UUID**: The `SERVICE_UUID` and `CHARACTERISTIC_UUID` in the code must be updated to match your Fardriver controller's BLE configuration. The provided UUID (`0000ffe0-0000-1000-8000-00805f9b34fb`) is a placeholder and may not work with your controller. To obtain the correct UUID:
  1. Download the **nRF Connect** app on your smartphone (available for iOS and Android).
  2. Turn on your Fardriver controller to enable its BLE advertising.
  3. Open nRF Connect, scan for devices, and locate your Fardriver controller in the list.
  4. Connect to the controller and inspect the available services. Look for a service with properties **NOTIFY**, **READ**, and **WRITE NO RESPONSE**. This service’s UUID is used for `CHARACTERISTIC_UUID`.
  5. Copy this UUID into both the `SERVICE_UUID` and `CHARACTERISTIC_UUID` fields in the code.
- **Wheel Circumference**: The `wheel_circumference_m` (default: 1.416m) must be adjusted to match your vehicle's wheel size for accurate speed calculations. Measure your wheel's circumference or calculate it as `π * diameter` (in meters).
- **Motor Pole Pairs**: The `motor_pole_pairs` (default: 20) must be set to your motor's pole pair count for accurate RPM-to-speed conversion. Check your motor's specifications.
- **Usage**: Upload the sketch to an ESP32, open the Serial Monitor at 115200 baud, and observe the data output. Ensure the ESP32 is within BLE range of the controller.

## Requirements

- **Hardware**: ESP32 development board (e.g., ESP32 DevKitC)
- **Software**: Arduino IDE with ESP32 board support installed
- **Libraries**: Arduino core for ESP32 (includes BLE libraries)
