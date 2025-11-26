# End-to-End Telemetry & Data Flow Diagram
---

This diagram illustrates the complete lifecycle of data within SmartCrops, from sensors → ESP32 → MQTT → Edge processing → Local DB → Cloud sync → Cloud visualization.

```
+----------------------------------------------------------------------------------------------------------------+
|                             SmartCrops – End-to-End Telemetry & Data Flow (ASCII)                              |
+----------------------------------------------------------------------------------------------------------------+

STEP 1 — SENSOR ACQUISITION (REAL-TIME LOOP)
---------------------------------------------

               +---------------------------+
               |        Sensors            |
               |---------------------------|
               | pH (UART)                 |
               | EC (UART)                 |
               | Temp/Humidity (I2C)       |
               | Water Temp (1-Wire)       |
               | Water Level (GPIO/UART)   |
               +---------------------------+
                           |
                           |  (UART / I2C / 1-Wire / GPIO)
                           v
               +---------------------------+
               |        ESP32 MCU          |
               |---------------------------|
               | - Reads sensors           |
               | - Normalizes data         |
               | - Safety checks (HW)      |
               | - Publishes MQTT events   |
               +---------------------------+
                           |
                           |  MQTT Publish (LAN Only)
                           v

STEP 2 — MESSAGE BROKER (IOT EVENT BUS)
----------------------------------------

                   +-------------------------------+
                   |   MQTT Broker (Mosquitto)     |
                   |-------------------------------|
                   |  Topics:                      |
                   |   - sensor/ph                 |
                   |   - sensor/ec                 |
                   |   - sensor/temp_air           |
                   |   - actuator/pump/state       |
                   |   - alerts/...                |
                   +-------------------------------+
                           |
                           |  MQTT Subscribe
                           v

STEP 3 — EDGE BACKEND PROCESSING
---------------------------------

          +--------------------------------------------------+
          |       Local Backend (Fastify + TypeScript)       |
          |--------------------------------------------------|
          | Components:                                      |
          |   • Sensor Manager                               |
          |   • Automation Engine (Flood & Drain)            |
          |   • Actuator Controller                          |
          |   • Scheduler                                    |
          |   • WebSocket Gateway                            |
          |   • Sync Manager                                 |
          |   • Data Access Layer (DAL)                      |
          +--------------------------------------------------+
                           |
                           |  Processed readings, events
                           v

STEP 4 — LOCAL STORAGE (TIME-SERIES DB)
----------------------------------------

                   +-------------------------------+
                   |    PostgreSQL (Local DB)      |
                   |-------------------------------|
                   |  - Time-series partitions     |
                   |  - Multi-year retention       |
                   |  - Raw + normalized tables    |
                   +-------------------------------+
                           |
                           |  Writes (INSERT) / Reads (SELECT)
                           v

STEP 5 — REAL-TIME LOCAL UI
----------------------------

         +----------------------------------------+
         |      Local Dashboard (Next.js UI)      |
         |----------------------------------------|
         |  - WebSocket live updates              |
         |  - Graphs, sensor history              |
         |  - Local admin/technician views        |
         +----------------------------------------+
                           ^
                           |
                           |  WebSocket (LAN)
                           |

STEP 6 — OUTBOUND-ONLY CLOUD SYNCHRONIZATION
---------------------------------------------

                   +-------------------------------+
                   |     Sync Manager (Client)     |
                   |-------------------------------|
                   | - Reads sync_queue            |
                   | - Creates batch payloads      |
                   | - HMAC signing                |
                   | - Retry w/ backoff            |
                   +-------------------------------+
                           |
                           |  WSS (TLS, outbound-only)
                           v

+------------------------------------------------------------------------------------------------------------+
|                                     CLOUD INFRASTRUCTURE (READ-ONLY)                                      |
+------------------------------------------------------------------------------------------------------------+

                    +-------------------------------+
                    |   Sync Server (WebSocket)     |
                    |-------------------------------|
                    | - Validates signatures        |
                    | - De-duplicates batches       |
                    | - Inserts into Cloud DB       |
                    +-------------------------------+
                           |
                           | SQL Inserts
                           v

                    +-------------------------------+
                    |        Cloud Database         |
                    |-------------------------------|
                    |  - Partitioned or single-file |
                    |  - Aggregated views           |
                    |  - Read-only model            |
                    +-------------------------------+
                           |
                           |
                           v

                    +-------------------------------+
                    |  Cloud Dashboard (Next.js)    |
                    |-------------------------------|
                    | - Remote visualization        |
                    | - Read-only data access       |
                    | - Clerk authentication        |
                    +-------------------------------+

SUMMARY
--------
Sensors → ESP32 → MQTT → Local Backend → Local DB → Local Dashboard  
                                          ↓  
                                Sync Queue → WS Outbound → Cloud Backend → Cloud DB → Cloud Dashboard
```
