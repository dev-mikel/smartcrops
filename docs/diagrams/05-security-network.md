# Network & Security Architecture (Zero Trust)
---

This diagram illustrates the complete Zero Trust security model for SmartCrops, including:  
Edge LAN isolation, deny-all inbound rules, cloud VPC segmentation, TLS-only communication, and outbound-only sync.

```
+--------------------------------------------------------------------------------------------------------------------+
|                                 SmartCrops – Network & Security Architecture (Zero Trust)                          |
+--------------------------------------------------------------------------------------------------------------------+

                                     ┌──────────────────────────────────────────┐
                                     │              Local Physical LAN          │
                                     └──────────────────────────────────────────┘
                                                            |
                                   +----------------------------------------------+
                                   |        Edge System – Security Boundary       |
                                   |   (Zero Trust: deny-all inbound traffic)     |
                                   +----------------------------------------------+
                                                            |
                         -------------------------------------------------------------------------------
                        |                                   |                                           |
                        v                                   v                                           v

          +------------------------------+   +-----------------------------+   +------------------------------+
          | Mini-PC (Edge Compute)       |   |   ESP32 MCU Node            |   | Local Devices / UI Clients   |
          |------------------------------|   |-----------------------------|   |------------------------------|
          | - Docker containers          |   | - Sensors/Actuators         |   | - Browser UI (HTTPS LAN)     |
          | - Fastify Backend            |   | - UART/I2C/1-Wire/GPIO      |   | - Technician/Admin access    |
          | - PostgreSQL DB              |   | - MQTT client               |   +------------------------------+
          | - MQTT Broker                |   | - No inbound ports          |
          | - Sync Client (WS outbound)  |   +-----------------------------+
          +------------------------------+
                        |
                        | MQTT (LAN only)
                        v

                [ MQTT Broker – LAN only, not exposed to Internet ]
                        |
                        |
                +---------------------------+
                | Local DB (Postgres)       |
                | - Time-series partitions  |
                | - Local authority         |
                +---------------------------+
                        |
                        | TLS WebSocket (Outbound Only)
                        v
+--------------------------------------------------------------------------------------------------------------------+
|                                        Internet (Untrusted Network)                                                |
|                              All inbound traffic to the Edge system is blocked                                     |
|                               Only SELECT outbound ports allowed (TLS 443)                                         |
+--------------------------------------------------------------------------------------------------------------------+
                        |
                        | HTTPS / WSS (Client → Cloud)
                        v
+--------------------------------------------------------------------------------------------------------------------+
|                                        Cloud VPC (Google Cloud – Private)                                           |
+--------------------------------------------------------------------------------------------------------------------+
                                              |
                              +--------------------------------------+
                              | Cloud Global Firewall (Deny-All)     |
                              | Only allow HTTPS 443 inbound         |
                              +--------------------------------------+
                                              |
                       -------------------------------------------------------------------
                       |                  |                      |                        |
                       v                  v                      v                        v

     +------------------------+   +---------------------+   +------------------------+   +---------------------------+
     | Web Subnet (10.20.10)  |   | Core Subnet         |   | DB Subnet (10.20.30)   |   | Monitoring Subnet         |
     |------------------------|   | (10.20.20)          |   |------------------------|   | (10.20.40)                |
     | - Cloud UI (Next.js)   |   | - Cloud Backend     |   | - Cloud Postgres/SQLite|   | - Prometheus/Grafana      |
     | - Clerk Auth           |   | - Sync Server (WS)  |   | - No public access     |   | - Exporters               |
     +------------------------+   +---------------------+   +------------------------+   +---------------------------+

                                            Security Controls
                                            ------------------
                                            • TLS everywhere (Edge → Cloud)
                                            • Outbound-only communication from Edge
                                            • No direct inbound to Mini-PC or ESP32
                                            • Cloud DB is read-only (no actuator control)
                                            • IAM separation:
                                                - Local = Basic Auth (offline-capable)
                                                - Cloud = Clerk (OAuth2 / JWT)
                                            • HMAC signing of all sync payloads
                                            • VPC subnets: web / core / db / monitoring
                                            • Firewall deny-all rulebase
                                            • Docker container isolation
```
