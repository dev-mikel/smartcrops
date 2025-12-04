---

# **Architecture Decision Records (ADR) – SmartCrops**

### *Version: 1.0 — Public Reduced Version*

### *Scope: FSR + SAD Architecture Decisions*

---

# **ADR List**

| ADR ID      | Area                 | Title                                                |
| ----------- | -------------------- | ---------------------------------------------------- |
| ADR-ARC-001 | Architecture         | Modular Monolithic Backend                           |
| ADR-ARC-002 | Architecture         | ESP32 as Dedicated Sensor/Actuator Controller        |
| ADR-ARC-003 | Integration          | MQTT as Internal Event Bus                           |
| ADR-ARC-004 | Integration/Cloud    | Outbound-Only WebSocket Sync                         |
| ADR-ARC-005 | Frontend             | Next.js for Local & Cloud UI                         |
| ADR-DAT-010 | Data                 | PostgreSQL Partitioned Time-Series Store             |
| ADR-DAT-011 | Data                 | Mirrored Cloud Schema (Read-Only)                    |
| ADR-SEC-001 | Security             | Cloud is Read-Only                                   |
| ADR-SEC-002 | Security             | Dual IAM Model (Local Basic Auth + Clerk Cloud Auth) |
| ADR-SEC-003 | Security             | HMAC-Signed Synchronization Payloads                 |
| ADR-INT-020 | Hardware/Integration | Mixed Sensor Buses (UART + I2C + 1-Wire + GPIO)      |
| ADR-INT-021 | Security/Integration | No Public Local API                                  |
| ADR-OPS-030 | Operations           | Docker Compose for All Deployments                   |
| ADR-OPS-031 | Operations           | Blue/Green Deployment on Cloud VPS                   |

---

# **📘 ADR-ARC-001 – Modular Monolithic Backend**

**Status:** Accepted
**Date:** YYYY-MM-DD
**Tags:** backend, architecture, modular-monolith

## **Context**

The SmartCrops system requires a low-latency, resilient, real-time control loop with limited compute resources running on a Mini-PC. The system must remain operational even with network outages.

## **Decision**

Use a **modular monolithic architecture** implemented in **Fastify + TypeScript**.

## **Alternatives Considered**

* Microservices
* Python-only backend
* Multi-process architecture

## **Justification**

* Lowest operational overhead
* Deterministic latency (no network hops internally)
* Easier deployment on edge hardware
* Better for real-time event sequencing
* Aligns with Docker Compose, not Kubernetes

## **Consequences**

* Clear module boundaries must be enforced internally
* Less horizontal scalability on edge (acceptable for MVP)

---

# **📘 ADR-ARC-002 – ESP32 as Dedicated Sensor/Actuator Controller**

**Status:** Accepted
**Tags:** IoT, MCU, hardware

## **Context**

Sensors and actuators require isolation, real-time handling, watchdogs, and protection against electrical noise.

## **Decision**

Use an **ESP32** as the microcontroller responsible for all sensor readings and actuator control.

## **Alternatives**

* USB sensors into Mini-PC
* Raspberry Pi GPIO

## **Justification**

* Industrial viability
* Dedicated hardware for real-time tasks
* Built-in WiFi + MQTT
* Electrical isolation safer than Mini-PC GPIO

## **Consequences**

* Requires firmware development
* Introduces a messaging layer (MQTT)

---

# **📘 ADR-ARC-003 – MQTT as Internal Event Bus**

**Status:** Accepted
**Tags:** mqtt, integration, messaging

## **Context**

Sensors publish frequently, actuators require quick commands, and decoupling MCU/Backend is mandatory.

## **Decision**

Use **Mosquitto MQTT** as internal event bus.

## **Alternatives**

* HTTP polling
* WebSockets only
* CoAP

## **Justification**

* Lightweight and perfect for MCU-level devices
* Decoupled event-driven architecture
* Industry-standard in IoT

## **Consequences**

* Adds broker configuration
* Requires topic governance

---

# **📘 ADR-ARC-004 – Outbound-Only WebSocket Synchronization**

**Status:** Accepted
**Tags:** cloud, security, networking

## **Context**

SmartCrops must not allow cloud control over critical actuators. Only local decision-making is allowed.

## **Decision**

Local → Cloud **WebSocket outbound-only** sync.

## **Alternatives**

* REST polling
* Remote MQTT broker
* Bidirectional WebSockets

## **Justification**

* Ensures cloud cannot trigger local actions
* Works even under intermittent connectivity
* Efficient streaming for time-series data

## **Consequences**

* Cloud dashboards are read-only
* Requires reliable local queue/retry logic

---

# **📘 ADR-ARC-005 – Next.js for Local and Cloud UI**

**Status:** Accepted
**Tags:** frontend, ui, consistency

## **Context**

Dashboards are needed locally (real-time) and in the cloud.

## **Decision**

Use **Next.js** for both UIs.

## **Justification**

* Shared UI patterns
* SSR for cloud, CSR for local
* Mature TS ecosystem

## **Consequences**

* Two builds to maintain
* Requires WebSocket integration on local UI

---

# **📘 ADR-DAT-010 – PostgreSQL Time-Series Partitioned Store**

**Status:** Accepted
**Tags:** database, time-series, postgres

## **Context**

Sensor data is high frequency and long-term (multi-year).

## **Decision**

Use **PostgreSQL** with **monthly partitioning**.

## **Alternatives**

* InfluxDB
* TimescaleDB
* SQLite

## **Justification**

* No extra overhead or new tech
* Better operational simplicity
* Partitioning = efficient queries + pruning
* Integrates easily with Docker

## **Consequences**

* Requires partition maintenance scripts
* Schema management is critical

---

# **📘 ADR-DAT-011 – Mirrored Cloud Schema (Read-Only)**

**Status:** Accepted
**Tags:** cloud, database

## **Decision**

Cloud database mirrors local schema but optimized for read-only queries.

---

# **📘 ADR-SEC-001 – Cloud is Read-Only**

**Status:** Accepted
**Tags:** security, safety, cloud

## **Decision**

No actuator commands or configuration changes allowed from cloud.

## **Justification**

* Safety-critical environment
* Avoid accidental or malicious cloud control
* Matches ICS/SCADA patterns

---

# **📘 ADR-SEC-002 – Dual IAM Model**

**Status:** Accepted
**Tags:** iam, auth, security

## **Decision**

* Local = Basic auth (offline capable)
* Cloud = Clerk (OAuth2 identity)

## **Justification**

* Local must work offline
* Cloud requires modern authentication

---

# **📘 ADR-SEC-003 – HMAC-Signed Sync Payloads**

**Status:** Accepted
**Tags:** security, crypto

## **Decision**

All outbound sync batches are **HMAC-signed** using an installation token.

## **Justification**

* Prevents tampering
* Prevents replay attacks
* Simple for MCU/edge environments

---

# **📘 ADR-INT-020 – Mixed Sensor Buses (UART + I2C + 1-Wire + GPIO)**

**Status:** Accepted
**Tags:** hardware, IoT, integration

## **Decision**

Use the bus required by each class of sensor:

* UART → pH, EC
* I2C → Temperature/Humidity
* 1-Wire → Water temperature
* GPIO → Float switches, SSR relays

---

# **📘 ADR-INT-021 – No Public Local API**

**Status:** Accepted
**Tags:** security, networking

## **Decision**

Local backend APIs are LAN-only and never exposed to the Internet.

---

# **📘 ADR-OPS-030 – Docker Compose as Deployment Model**

**Status:** Accepted
**Tags:** devops, deployment

## **Decision**

Use Docker Compose on both local and cloud environments.

## **Justification**

* No Kubernetes overhead
* Easier local debugging
* Works offline

---

# **📘 ADR-OPS-031 – Blue/Green Deployment on Cloud VPS**

**Status:** Accepted
**Tags:** devops, release-engineering

## **Decision**

Cloud deployments use Blue/Green strategy to ensure no downtime and easy rollback.

---
