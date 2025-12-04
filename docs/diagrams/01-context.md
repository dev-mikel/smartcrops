# C4 Level 1 – System Context Diagram
---

This diagram provides a high-level view of all external actors, the Edge system, the Cloud system, and how they interact.

```
+--------------------------------------------------------------------------------------+
|                             SmartCrops – System Context (C4 L1)                      |
+--------------------------------------------------------------------------------------+

                            [ Administrator / Technician / Viewer ]
                                           |
                                           |  (HTTPS / Web, LAN or Internet)
                                           v

+-----------------------------------+                      +---------------------------+
|      SmartCrops Local System      |                      |   SmartCrops Cloud System |
|              (Edge)               |                      |         (Cloud)           |
|-----------------------------------|                      |---------------------------|
|                                   |                      |                           |
|   +---------------------------+   |                      |   +-------------------+   |
|   |   Local Web UI (Next.js)  |<-----------------------------| Cloud Dashboard   |   |
|   +---------------------------+   |        (HTTPS)       |   |    (Next.js)      |   |
|               |                   |                      |   +-------------------+   |
|               |  (WebSocket/HTTP) |                      |           |               |
|               v                   |                      |           | (HTTPS/REST)  |
|   +---------------------------+   |         Sync         |           v               |
|   | Local Backend (Fastify+TS)|------------------------->|  +---------------------+  |
|   |   - Automation Engine     |   | (WebSocket outbound) |  | Cloud Backend       |  |
|   |   - Business Logic        |   |                      |  | + Sync Server (WS)  |  |
|   +---------------------------+   |                      |  +---------------------+  |
|               |                   |                      |           |               |
|               | (SQL)             |                      |           | (SQL)         |
|               v                   |                      |           v               |
|      +--------------------+       |                      |   +-------------------+   |
|      | Local DB (Postgres)|       |                      |   |   Cloud DB        |   |
|      +--------------------+       |                      |   | (Postgres/SQLite) |   |
|                                   |                      |   +-------------------+   |
+-----------------------------------+                      +---------------------------+
                ^
                |   (MQTT, LAN / WiFi)
                |
        +------------------------+
        |   ESP32 MCU Node       |
        |  Sensors + Actuators   |
        +------------------------+
```
