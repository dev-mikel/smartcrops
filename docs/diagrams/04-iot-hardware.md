# IoT Hardware Integration Diagram
---

This diagram shows the complete hardware integration model for the SmartCrops MVP, including:  
ESP32 MCU, sensor interfaces, actuator relays, wiring buses (UART/I2C/1-Wire/GPIO), and MQTT link to the Edge System.

```
+-----------------------------------------------------------------------------------------------------------+
|                                SmartCrops – IoT Hardware Integration (ESP32)                              |
+-----------------------------------------------------------------------------------------------------------+

                                      ┌────────────────────────────────────┐
                                      │              ESP32 MCU             │
                                      │------------------------------------│
                                      │  - MQTT Client                     │
                                      │  - UART / I2C / 1-Wire / GPIO      │
                                      │  - PWM/GPIO Relay Control          │
                                      │  - Hardware Watchdog               │
                                      │  - Sensor Acquisition Loop         │
                                      └────────────────────────────────────┘
                                                        |
          -----------------------------------------------------------------------------------------------------
          |                        |                          |                         |                      |
          v                        v                          v                         v                      v

+----------------------+   +-----------------------+   +-----------------------+   +----------------------+   +---------------------+
|   pH Sensor (EZO)    |   |   EC Sensor (EZO)     |   | Air Temp/Humidity     |   | Water Temp Sensor    |   | Water Level         |
|----------------------|   |-----------------------|   | Sensor (SHT31/AHT20)  |   | (DS18B20 - 1-Wire)   |   | (Float / JSN-SR04T) |
|  - UART (TX/RX)      |   |  - UART (TX/RX)       |   |  - I2C (SDA/SCL)      |   |  - Single-wire bus   |   |  - GPIO / UART      |
|  - Isolated Board    |   |  - Isolated Board     |   |  - 3.3V logic         |   |  - Waterproof probe  |   |  - Trigger/Echo     |
+----------------------+   +-----------------------+   +-----------------------+   +----------------------+   +---------------------+

                                                        |
                                                        |
                                                        v
                                     ┌────────────────────────────────────┐
                                     │        Actuator Control Layer      │
                                     │------------------------------------│
                                     │   - GPIO outputs → SSR/Relays      │
                                     │   - Electrical isolation           │
                                     └────────────────────────────────────┘
                                                        |
                                 -------------------------------------------------------
                                 |                          |                           |
                                 v                          v                           v
                     +--------------------+     +---------------------+     +----------------------+
                     |  Water Pump (SSR)  |     |   Fans / Extractor  |     | Nutrient Pump (SSR) |
                     +--------------------+     +---------------------+     +----------------------+
                                                        |
                                                        v
                                                        MQTT (Publish readings & events)
+-----------------------------------------------------------------------------------------------------------+
|                                Edge System (Mini-PC – MQTT Broker)                                        |
+-----------------------------------------------------------------------------------------------------------+
```
