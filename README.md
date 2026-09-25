# Digital Blossom - Smart Connected Flower Pot

An interactive, long-distance relationship desktop art piece and IoT device. **Digital Blossom** features a circular color IPS display mounted on a hollow metallic stem resembling a flower blossom, paired with a smart electronics hub, physical controls, an integrated battery charger, a secondary dashboard status display, and a two-way digital voice mailbox—all nestled inside a compact desktop planter pot.

This repository contains the complete firmware, mechanical specifications, and assembly instructions for building, flashing, and configuring the Digital Blossom.

---

## 🌸 Key Features

* **Simultaneous "Heartbeat" Presence:** A capacitive touch sensor wired to a brass leaf or pot rim detects physical interaction. Touching the leaf registers a timestamp on a shared real-time database. If your partner touches theirs within 10 seconds, both devices trigger a synchronized, vibrant "Together Now" golden blooming petal animation accompanied by a harmonic chime.
* **Two-Way Audio Mailbox:** Record and stream 10-second digital voice notes directly between pots. Pressing and holding the rotary encoder records audio via an I2S digital microphone, packaging it as a `.wav` file before posting it to a Telegram channel. An incoming note flashes a classic cassette tape icon on the OLED dashboard; tapping the rotary switch plays the note through an integrated I2S Class-D amplifier and mini speaker.
* **Remote Photo & Art Streaming:** Easily stream photos or custom illustrations to your partner's blossom. Images uploaded to a dedicated Telegram Bot are automatically fetched, decoded in high-capacity PSRAM, and rendered directly onto the circular IPS screen.
* **Atmospheric & Multi-View Engine:** Use the physical rotary dial to scrub through multiple system modes:
  * **Living Blossom:** Interactive real-time digital art and smooth visual animations driven by the LVGL graphics library.
  * **Circadian Sky & Time Zone Dial:** The background sky dynamically shifts (dawn, day, golden hour, starry night) based on your partner's local time zone, calculated using a high-precision DS3231 Real-Time Clock.
  * **Milestone Time Capsule:** Scrub through custom relationship milestones (e.g., "Our First Trip - 428 Days Ago") on the base OLED while displaying corresponding pictures on the circular LCD.
  * **Focus Bloom:** A productivity Pomodoro timer that displays a closed flower bud slowly opening petal by petal over 25 minutes.

---

## 🛠️ Hardware Architecture & BOM

To successfully handle dual-buffered graphic layouts, secure webhooks, and digital audio streams, staying within the **ESP32-S3 family with external PSRAM** is required. 

Below is the complete bill of materials optimized to stay within a **€30 budget**.

### 1. Core Processor & Power
* **Microcontroller:** ESP32-S3 DevKitC-1 N16R8 (16MB Flash / 8MB Octal PSRAM)
* **Battery:** 3.7V XTAR 18650 Li-ion Battery (2600mAh)
* **Battery Holder:** Single-slot 18650 plastic battery holder with pre-attached wire leads
* **Charger Module:** TP4056 USB-C Li-ion Charging Module with Protection (must have 6 solder pads: `B+`, `B-`, `OUT+`, `OUT-`)
* **Power Switch:** Mini SPDT slide switch (cuts battery connection during USB coding/programming)
* **Voltage Divider Resistors:** Two 100kΩ resistors (for safe battery level monitoring via ESP32 ADC)

### 2. Audio & Interfaces
* **Visual Display (Blossom):** 1.28" Round IPS LCD Display (GC9A01 driver over SPI, 240x240 resolution)
* **Visual Display (Dashboard):** 0.91" Monochrome OLED Display (SSD1306 driver over I2C, 128x32 resolution)
* **Microphone (Audio In):** INMP441 I2S Digital MEMS Microphone Module
* **Audio Amplifier (Audio Out):** MAX98357A I2S Class-D 3W Amplifier Breakout
* **Speaker:** 8 Ohm, 1W to 2W Micro Speaker (diameter ≤ 28mm to fit enclosure base)
* **Rotary Encoder:** KY-040 Rotary Encoder Module (with integrated push-button)
* **Real-Time Clock:** DS3231 High-Precision RTC Module
* **Touch Sensor:** Solid brass wire, brass rod, or a decorative brass leaf

### 3. Structural & Prototyping
* **Planter Enclosure:** 3D-printed, plastic, or wood desktop planter pot (with cutouts for screens, USB, and dials)
* **Flower Stem:** Hollow metal tube (8mm to 10mm brass, copper, or aluminum pipe) to route 7 thin wires to the circular screen
* **Prototyping Gear:** Solderless breadboard, Dupont jumper wires, 30 AWG thin solid core wire, soldering iron, and solder

---

## 📌 ESP32-S3 System Pinout Mapping

The abundant GPIO capacity of the ESP32-S3 DevKitC-1 allows all components to connect simultaneously. Refer to the pinout below:

| Peripheral / Component | Pin Name | GPIO | Bus / Function |
| :--- | :--- | :--- | :--- |
| **GC9A01 SCL (SCK)** | D8 | GPIO 9 | Hardware FSPI SCK |
| **GC9A01 SDA (MOSI)** | D10 | GPIO 10 | Hardware FSPI MOSI |
| **GC9A01 CS** | D1 | GPIO 2 | SPI Chip Select |
| **GC9A01 DC** | D2 | GPIO 3 | Data / Command |
| **GC9A01 RES** | D0 | GPIO 1 | Display Hardware Reset |
| **Shared I2C SDA** | D4 | GPIO 5 | SSD1306 (`0x3C`) & DS3231 (`0x68`) SDA |
| **Shared I2C SCL** | D5 | GPIO 6 | SSD1306 (`0x3C`) & DS3231 (`0x68`) SCL |
| **Rotary CLK** | D6 | GPIO 7 | Encoder Phase A |
| **Rotary DT** | D7 | GPIO 8 | Encoder Phase B |
| **Rotary SW** | D9 | GPIO 44 | Push-button (Internal Pullup, Active LOW) |
| **Capacitive Touch** | D3 | GPIO 4 | Brass Leaf Capacitive Touch Interrupt |
| **MAX98357A DIN** | D13 | GPIO 13 | I2S Channel 0: Serial Data Out |
| **MAX98357A BCLK** | A0 | GPIO 15 | I2S Channel 0: Bit Clock |
| **MAX98357A LRCLK** | A1 | GPIO 16 | I2S Channel 0: Word Select (Left/Right Clock) |
| **INMP441 SD** | D11 | GPIO 11 | I2S Channel 1: Serial Data In |
| **INMP441 SCK** | D12 | GPIO 12 | I2S Channel 1: Bit Clock |
| **INMP441 WS** | D14 | GPIO 14 | I2S Channel 1: Word Select |
| **Battery Gauge ADC** | A4 | GPIO 17 | Analog voltage read via 100kΩ/100kΩ divider |

---

## ⚡ Power Distribution & Charging Wiring Diagram

Ensure your XTAR 18650 cell is routed through the TP4056 protection board. Use a slide switch to protect your computer's USB port from power feedback during programming.

```text
                    +------------------------------------+
                    |   TP4056 Charger with Protection   |
                    |                                    |
                    | [USB-C Input]   B+    B-  OUT+ OUT-|
                    +-----------------|-----|----+----+--+
                                      |     |    |    |
               +----------------------+     |    |    |
               |                            |    |    |
       (Red Wire +)                         |    |    |
      +-------------+                       |    |    |
      |             |                       |    |    |
      | 18650 Cell  |                       |    |    |
      |  (Battery)  |                       |    |    |
      |             |                       |    |    |
      +-------------+                       |    |    |
       (Black Wire -)                       |    |    |
               +----------------------------+    |    |
                                                 |    |
                                           [Slide Switch]
                                                 |    |
                                                 v    v
                                               (5V) (GND)
                                                 |    |
                                                 v    v
                                            ESP32-S3 Board
