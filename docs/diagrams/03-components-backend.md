# C4 Level 3 – Local Backend Components
---

This diagram shows the internal architecture of the SmartCrops Edge Backend (Fastify + TypeScript).  
It illustrates how automation logic, sensor handling, actuator control, data management, and cloud synchronization interact inside the backend.

```
+------------------------------------------------------------------------------------------------------+
|                          SmartCrops – Local Backend (C4 Level 3: Components)                         |
+------------------------------------------------------------------------------------------------------+

                                     [ Edge System / Mini-PC ]
                                                |
                                                |
                                   +--------------------------------+
                                   |  Local Backend (Fastify + TS)  |
                                   |--------------------------------|
                                   |  Handles automation, data,     |
                                   |  messaging, and local UI       |
                                   |  communication.                |
                                   +--------------------------------+
                                                |
                                                |
      ----------------------------------------------------------------------------------------------------
      |                                         |                        |                               |
      v                                         v                        v                               v

+---------------------------+     +---------------------------+     +---------------------------+     +---------------------------+
|  API Controller Layer     |     |  WebSocket Gateway        |     |  Sensor Manager           |     |  Actuator Controller      |
|---------------------------|     |---------------------------|     |---------------------------|     |---------------------------|
| - Validates HTTP calls    |     | - Push real-time events   |     | - Subscribes to MQTT      |     | - Publishes MQTT commands |
| - Routes UI/API requests  |     | - UI <-> Backend channel  |     | - Normalizes sensor data  |     | - Controls pumps/fans     |
| - Calls automation logic  |     | - Broadcasts state        |     | - Stores readings         |     | - State tracking          |
+---------------------------+     +---------------------------+     +---------------------------+     +---------------------------+
              |                                  |                       |                                  |
              v                                  v                       v                                  v
                                        +----------------------------------------------+
                                        |             Automation Engine                |
                                        |----------------------------------------------|
                                        | - Flood & Drain logic                        |
                                        | - Phase-based control                        |
                                        | - Rule-based thresholds                      |
                                        | - Safety checks (dry run, overflow, etc.)    |
                                        | - Reacts to sensor events                    |
                                        | - Commands Actuator Controller               |
                                        +----------------------------------------------+
                                                             |
                                                             v
                                        +----------------------------------------------+
                                        |        Data Access Layer (DAL)               |
                                        |----------------------------------------------|
                                        | - SQL queries (PostgreSQL)                   |
                                        | - Time-series partitions                     |
                                        | - Insert sensor logs                         |
                                        | - Fetch state, events, stats                 |
                                        +----------------------------------------------+
                                                             |
                                                             v
                                               [ PostgreSQL – Local DB ]

      -------------------------------------------------------------------------------------------------------------------------
      |                                                                                                                       |
      |                                       Sync (Outbound Only)                                                            |
      |                                                                                                                       |
      v                                                                                                                       v

+---------------------------+                                                                 +-------------------------------+
|  Sync Manager (Client)    | --------------------------------------------------------------> | Cloud Sync Server (WebSocket) |
|---------------------------|                         (WebSocket outbound-only)               |  *Read-only cloud system*     |
| - Reads sync_queue        |                                                                 +-------------------------------+
| - Creates signed batches  |
| - HMAC validation         |
| - Retries + backoff       |
+---------------------------+
```
