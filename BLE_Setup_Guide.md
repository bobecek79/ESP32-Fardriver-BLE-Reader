# Step-by-Step Guide: Connect to BLE Characteristic and Get Speed Working

## 1. Find the Speed Characteristic in nRF Connect

* Open **nRF Connect for Mobile** → scan and connect to your device
* Expand the **Unknown Service 0xFFE0** 
* Select the **Unknown Characteristic 0xFFEC** that shows **NOTIFY, READ, WRITE NO RESPONSE**

> **Note:** `0xFFEC` is a 16-bit vendor UUID. The full 128-bit form is:
> **`0000ffec-0000-1000-8000-00805f9b34fb`**
> 
> Bluetooth "short" UUIDs expand into the standard Base UUID: `0000xxxx-0000-1000-8000-00805F9B34FB`

## 2. Copy the UUID for Your Code

* In nRF Connect, tap the **clipboard** icon on the characteristic row to copy the UUID
* You should get: `0000ffec-0000-1000-8000-00805f9b34fb`
* **Important:** Note that the **CCCD** (Client Characteristic Configuration Descriptor) has UUID **0x2902** - you'll need this to enable notifications

## 3. Subscribe to Notifications

* In nRF Connect, tap the **two down-arrows** icon next to the characteristic to **enable notifications**
* You should now see live hex packets (like the `AA-AD-…-66-77` frame in your screenshot)
* This writes `0x0001` to **CCCD (0x2902)** to enable Notify mode

## 4. Implement the UUID in Your Code

Replace the placeholder characteristic UUID in your project:

### Arduino/ESP32 (NimBLE/ESP-IDF style)
```cpp
static BLEUUID speedCharUUID("0000ffec-0000-1000-8000-00805f9b34fb");
```

### Web Bluetooth
```javascript
const SPEED_CHAR_UUID = '0000ffec-0000-1000-8000-00805f9b34fb';
```

## 5. Enable Notifications in Code

Ensure your client subscribes to notifications on the characteristic:
* Write `0x01,0x00` to descriptor `0x2902`, or 
* Use your library's "startNotifications" call

## 6. Configure Speed Calculation Constants

Update these two critical constants:

```cpp
const float wheel_circumference_m = 1.416;  // meters
const int   motor_pole_pairs      = 20;
```

**Explanation:**
- **`wheel_circumference_m`**: Tire circumference in meters
- **`motor_pole_pairs`**: Converts electrical rotation rate to mechanical RPM

> **Speed Calculation Formula:**
> ```
> mechanical RPM = eRPM / pole_pairs
> wheel linear speed (m/s) = (mechanical RPM / 60) × wheel_circumference_m
> ```
> 
> *Note: Include gear ratios if there's gearing between motor and wheel*

## 7. Verify Operation

* With notifications enabled, you should see packets arriving in both the app and your code's notification handler
* Spin the wheel and verify that reported speed changes
* If no updates appear, re-enable notifications (CCCD 0x2902 must be `0x0001`)

## 8. Troubleshooting

### Common Issues & Solutions

| Problem | Solution |
|---------|----------|
| **Wrong characteristic?** | Ensure you're on **Service 0xFFE0 → Characteristic 0xFFEC** |
| **Notifications disabled?** | Tap the **dual-arrow** icon in nRF Connect again, or verify your client wrote `0x01,0x00` to **0x2902** |
| **Speed seems inaccurate?** | Re-measure tire circumference (in meters) and confirm your **pole pair** count |

### Quick Verification Checklist

- [ ] Connected to correct device
- [ ] Selected Service 0xFFE0
- [ ] Selected Characteristic 0xFFEC  
- [ ] Notifications enabled (CCCD = 0x0001)
- [ ] UUID copied correctly to code
- [ ] Constants updated with your values
- [ ] Speed changes when wheel spins

---

**References:** [1] Novel Bits, [2] MDN Web Docs, [3] Nordic Developer Academy, [4] TI Software, [5] Arduino Forum, [6] Stack Overflow, [7] Bluetooth Technology Website, [8] Microchip, [9] Sciencing, [10] u-blox Content