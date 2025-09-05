Detailed Description: ESP32 Fardriver BLE Controller Reader
What This Code Does
This Arduino sketch lets an ESP32 microcontroller connect to a Fardriver controller (used in electric bikes or scooters) over Bluetooth Low Energy (BLE). It reads data like voltage, current, power, RPM, gear, speed, temperatures, state of charge (SOC), and whether the bike is in regeneration mode (regen, when the battery is being charged during braking or coasting). The data is sent to the Serial Monitor in the Arduino IDE, so you can see it in real-time on your computer. This is a simplified version of my project, with the display code removed, perfect for testing or building your own eBike projects.
The code is designed to be easy to use, even if you’re new to coding. I built it with help from AI tools, so don’t worry if you’re a beginner—you can make it work too!
How It Works
Here’s a step-by-step breakdown of what the code does:

Start the ESP32: The code sets up the ESP32 to act as a Bluetooth client, looking for your Fardriver controller.
Find the Controller: It scans for a device broadcasting a specific Bluetooth service ID (UUID).
Connect and Listen: Once connected, it listens for data from the controller’s characteristic UUID.
Decode Data: The code checks each packet to make sure it’s valid, then pulls out useful information like voltage, speed, and more.
Calculate Values: It uses the raw data to calculate things like speed (based on your bike’s wheel size and motor settings) and power.
Show Results: Every half-second, it prints the data to the Serial Monitor, so you can see what’s happening with your bike.
Detect Regen: If the current is negative (up to -100A), it flags that the bike is in regen mode, meaning the battery is charging.

What You’ll See in the Serial Monitor
When you run the code and connect to your controller, the Serial Monitor (in the Arduino IDE) will show output like this every 500ms:
--- Fardriver Controller Data ---
Voltage: 72.5 V
Line Current: 10.2 A
Power: 738 W
RPM: 1200
Gear: 2
Speed: 35.6 km/h
Controller Temp: 45 C
Motor Temp: 60 C
SOC: 85 %
Regen (Current): No
--------------------------------

If the current goes negative (e.g., -20.5 A), Regen (Current): Yes will show, indicating regen mode.
Requirements

Hardware: An ESP32 development board (like the ESP32 DevKitC).
Software: Arduino IDE (free, download from arduino.cc).
Libraries: The code uses the built-in ESP32 BLE libraries, included when you set up the ESP32 board in the Arduino IDE.
Setup: A computer to run the Arduino IDE and a USB cable to connect the ESP32.
Fardriver Controller: Your eBike’s Fardriver controller must have BLE enabled.

Getting Started

Install Arduino IDE: Download and install it from arduino.cc.
Add ESP32 Support: In the Arduino IDE, go to File > Preferences, add this URL to the Additional Boards Manager URLs: https://raw.githubusercontent.com/espressif/arduino-esp32/master/package_esp32_index.json, then go to Tools > Board > Boards Manager, search for “ESP32,” and install the ESP32 package.
Connect Your ESP32: Plug your ESP32 into your computer via USB.
Open the Code: Download the FardriverBLE.ino file from the GitHub repository (https://github.com/bobecek79/ESP32-Fardriver-BLE-Reader) and open it in the Arduino IDE.
Make Changes: You need to tweak a few parts of the code to match your bike (see below).
Upload the Code: Select your ESP32 board under Tools > Board (e.g., “ESP32 Dev Module”), choose the correct port under Tools > Port, and click the upload button.
Open Serial Monitor: Go to Tools > Serial Monitor and set the baud rate to 115200 to see the data.

Parts of the Code You Need to Change
To make the code work with your Fardriver controller and bike, you need to modify three settings: the Bluetooth service UUID, characteristic UUID, wheel circumference, and motor pole pairs. These changes are simple, even if you’re new to coding. Below, I’ll show you exactly where and how to update them.
1. Bluetooth Service and Characteristic UUIDs
The code uses two Bluetooth IDs (UUIDs): one for the service and one for the characteristic. These identify your Fardriver controller and the specific data stream it sends. The default UUIDs in the code are placeholders and won’t work for your controller. You need to find your controller’s UUIDs using the nRF Connect app and update them in the code. Note that the service UUID and characteristic UUID are different, even if they look similar.
Where to Change:In the code, look for these lines near the top (around line 20):
#define SERVICE_UUID        "0000ffe0-0000-1000-8000-00805f9b34fb"
#define CHARACTERISTIC_UUID "0000ffec-0000-1000-8000-00805f9b34fb"

How to Find Your UUIDs:

Download the nRF Connect app on your smartphone (available for iOS or Android).
Turn on your Fardriver controller (make sure your bike is powered on).
Open nRF Connect and scan for devices. Look for your Fardriver controller in the list (it might show as “Fardriver” or a similar name).
Connect to the controller and view the available services.
Find the service with properties NOTIFY, READ, and WRITE NO RESPONSE. Press and hold on the service UUID to copy it (it looks like 0000xxxx-0000-1000-8000-00805f9b34fb, where xxxx is unique).
Within that service, find the characteristic with the same properties (NOTIFY, READ, WRITE NO RESPONSE). Press and hold on the characteristic UUID to copy it (it will be different from the service UUID, e.g., 0000yyyy-0000-1000-8000-00805f9b34fb).
Save both UUIDs.

How to Update:Replace the placeholder UUIDs with your controller’s UUIDs. For example, if your service UUID is 00001234-0000-1000-8000-00805f9b34fb and your characteristic UUID is 00005678-0000-1000-8000-00805f9b34fb, change the lines to:
#define SERVICE_UUID        "00001234-0000-1000-8000-00805f9b34fb"
#define CHARACTERISTIC_UUID "00005678-0000-1000-8000-00805f9b34fb"

Tip: If you can’t find the UUIDs or nRF Connect is confusing, ask on a forum like Endless Sphere or use an AI tool like ChatGPT or Grok for help.
2. Wheel Circumference
The code calculates your bike’s speed based on the wheel size. The default value is set for a wheel circumference of 1.416 meters, but you need to change this to match your bike’s wheel.
Where to Change:Look for this line in the processPacket function (around line 140):
const float wheel_circumference_m = 1.416;

How to Find Your Wheel Circumference:

Measure your wheel’s diameter (in meters) and calculate the circumference using π * diameter (π ≈ 3.1416). For example, a 26-inch wheel is about 0.66 meters in diameter, so the circumference is 3.1416 * 0.66 ≈ 2.073 meters.
Alternatively, roll your wheel one full turn, mark the ground, and measure the distance.

How to Update:Replace the default value with your wheel’s circumference. For example, if it’s 2.073 meters:
const float wheel_circumference_m = 2.073;

Tip: If your speed readings look wrong (e.g., too high or low), double-check this value.
3. Motor Pole Pairs
The code uses the number of pole pairs in your motor to convert RPM to speed accurately. The default is 20, but your motor might be different.
Where to Change:In the processPacket function, look for this line (right below wheel circumference):
const int motor_pole_pairs = 20;

How to Find Your Motor Pole Pairs:

Check your motor’s documentation or label (often provided by the manufacturer or in your eBike’s specs).
If you can’t find it, ask your eBike supplier or search online for your motor model. Common values range from 10 to 30 for eBike motors.

How to Update:Replace the default value with your motor’s pole pairs. For example, if your motor has 16 pole pairs:
const int motor_pole_pairs = 16;

Tip: Incorrect pole pairs will mess up speed and RPM readings, so verify this with your motor’s specs.
Other Notes for Beginners

Why These Changes Matter: The UUIDs ensure the ESP32 connects to your controller. The wheel circumference and pole pairs make sure the speed is accurate. Skipping these steps could mean no connection or wrong data.
Testing: After uploading the code, open the Serial Monitor. If you see “Scanning for Fardriver Controller...” and no data, check your UUIDs or ensure the controller is powered on and in range.
Safety: Make sure your ESP32 is securely connected and your bike is safe to test (e.g., on a stand if powered).
Ask for Help: If you’re stuck, post on forums like Endless Sphere or Arduino, or ask an AI tool like Grok for troubleshooting tips.

What You Can Do with This Code

Monitor Your Bike: Use the Serial Monitor to check your bike’s performance in real-time, great for debugging or tuning.
Build a Display: Add a screen (like an OLED) to show the data on your bike. You can ask AI tools for code to do this.
Log Data: Modify the code to save data to an SD card or send it to a phone app for tracking rides.
Share Ideas: Fork the GitHub repo and add your own features, like alerts for high temperatures or low SOC.

License
This project is licensed under the MIT License, meaning you can use, modify, and share it freely as long as you include the original license file. See the LICENSE file in the repository for details.
