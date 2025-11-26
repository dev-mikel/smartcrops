# **📄 Solution Architecture Document (SAD)**

# **SmartCrops Platform**

**Version:** 1.0
**Document Type:** Solution Architecture Document (SAD)
**Author:** Miguel Ladines
**Based on FSR:** *SmartCrops Functional Specification Requirements* 
**Classification:** MVP

# **Table of Contents**

* [1. Introduction and Purpose of the Architecture](#1-introduction-and-purpose-of-the-architecture)
* [2. Solution Scope](#2-solution-scope)
* [3. High-Level Context](#3-high-level-context)
* [4. Business Architecture](#4-business-architecture)
* [5. Logical Architecture](#5-logical-architecture)
* [6. Physical / Infrastructure Architecture](#6-physical--infrastructure-architecture)
* [7. Application Architecture](#7-application-architecture)
* [8. Integration Architecture](#8-integration-architecture)
* [9. Data Architecture](#9-data-architecture)
* [10. Security Architecture](#10-security-architecture)
* [11. DevOps & Environments Architecture](#11-devops--environments-architecture)
* [12. Architectural Patterns Applied](#12-architectural-patterns-applied)
* [13. Architectural Decisions (ADR Summaries)](#13-architectural-decisions-adr-summaries)
* [14. Architectural Risks & Mitigation](#14-architectural-risks--mitigation)
* [15. Technical Assumptions](#15-technical-assumptions)
* [16. Limitations / Exclusions](#16-limitations--exclusions)
* [17. C4 Diagrams (Narrated)](#17-c4-diagrams-narrated)
* [18. Technical Annexes](#18-technical-annexes)

---

## **1. Introduction and Purpose of the Architecture**

The purpose of this Solution Architecture Document (SAD) is to provide a complete and authoritative technical architecture for the **SmartCrops Hydroponic Automation Platform**, defining how the system is structured, deployed, secured, integrated, and operated from an enterprise-grade architectural standpoint.

SmartCrops is a **fully autonomous hydroponic control system** for Flood & Drain installations, with **local-first operation** and **optional cloud visualization**, as described in its FSR .
This SAD defines:

* The architectural vision and guiding principles
* Logical and physical decomposition
* Application, data, security, integration, and DevOps architectures
* C4 views (Context, Container, Component)
* Key architectural decisions (ADR references)
* Risks, assumptions, and constraints

This document is designed for review by architects, tech leads, developers, security teams, and cloud/DevOps engineers.

## **2. Solution Scope**

This SAD covers **only architectural concerns** and excludes all functional descriptions.
The scope includes:

### **In-Scope Architectural Areas**

* Local on-premise hydroponic controller architecture (Mini-PC + ESP32)
* Containerized backend, frontend, MQTT broker, PostgreSQL
* Cloud environment (Google Cloud VPS) for remote dashboards
* Data synchronization architecture (WebSockets-based push model)
* Internal communication (MQTT + UART/I2C/1-Wire)
* C4 logical and physical diagrams (text-based)
* Network topology, VCN, subnets, routing and firewalling
* Security baseline (authN, authZ, encryption, secrets, roles)
* DevOps build/deploy model with Docker Compose
* Observability architecture (logging, metrics, auditing)
* Patterns and ADRs
* Architectural risks and mitigations

### **Explicitly Out of Scope**

* All functional requirements (FSR already covers them)
* Roadmaps, planning, WBS, Sprints
* UI/UX design
* Low-level electrical schematics
* Predictive analytics (Phase II)
* Business logic descriptions
* Hardware implementation details
* Algorithms and firmware logic specifics

## **3. High-Level Context**

### **3.1 Vision Overview**

SmartCrops is architected as a **Local-First Autonomous Hydroponic Control System**, where:

* **All control, automation, logic, and safety protections run entirely onsite**
* **A Mini-PC (Intel i5) acts as the local orchestration and backend server**
* **An ESP32 microcontroller handles all sensor acquisition and actuator control**
* **Inter-module communication uses local MQTT + UART/I2C/1-Wire**
* **Local Web UI (Next.js) provides real-time control and dashboards**
* **PostgreSQL stores multi-year historical sensor data**
* **A Cloud VPS is optional and used only for synchronized data visualization**

The system continues operating even if the Internet is unavailable, as required in the FSR (offline autonomy) .

### **3.2 Architectural Drivers**

#### **Functional Drivers (from FSR)**

* Full local autonomy and continuous operation offline
* Real-time sensor monitoring (pH, EC, climate, water level)
* Flood & Drain irrigation automation
* Local-only control and override safety model
* Long-term historical data retention
* Cloud dashboard for read-only visualization
* Asynchronous synchronization

#### **Technical Drivers**

* High reliability and deterministic behavior for crop safety
* Modular monolithic deployment (per your tech-stack baseline)
* Hardware abstraction via ESP32
* Low-latency message distribution (MQTT)
* Secure network segmentation
* Simple deploy/rollback pipelines
* Multi-year storage efficiency (time-series)
* Future expandability (Phase II analytics in Python)

#### **Quality Attribute Drivers**

* **Reliability:** Autonomous operation, no cloud dependency
* **Safety:** Strict override rules, actuator monitoring
* **Performance:** Sensor update cycles 10–60 seconds (tomato hydroponics best practices)
* **Security:** Zero-trust networking, local auth, Clerk in cloud
* **Maintainability:** Modular code, container-based deployment
* **Scalability:** Cloud dashboard can serve unlimited installations

### **3.3 Constraints**

* MVP does **not include AI/analytics** (Phase II enhancement)
* System supports **one hydroponic rig per installation** (FSR BR-001)
* No remote control from cloud (security constraint)
* No managed services in cloud (Compute Engine only)
* No load balancers in MVP
* No Kubernetes (Docker Compose only)
* Local-only API is private and not exposed publicly
* ESP32 must interface sensors using UART + I2C + 1-Wire

### **3.4 High-Level System Context (C4 – Level 1)**

**Primary External Actors**

* **Administrator**
* **Technician**
* **Viewer**

**Primary Systems**

* **SmartCrops Local System** (Mini-PC + ESP32)
* **SmartCrops Cloud System** (VPS on Google Cloud)

#### **Context Summary (text-based diagram)**

```
[Administrator] ──\
[Technician] ------> [Local Web UI (Next.js)]
[Viewer] ─────────/               |
                                V
                        [Local Backend (Fastify TS)]
                                |
                                v
                        [MQTT Broker (Mosquitto)]
                                |
                 +--------------+--------------+
                 |                             |
        [ESP32 Sensor/Actuator Node]     [PostgreSQL Storage]
                 |
                 v
         Sensors/Actuators (pH, EC, Temp, Fans, Pumps, Level)

Cloud Context:
[Local Backend] ---> (WebSockets Sync) ---> [Cloud Backend + DB]
[Users] -----------------> [Cloud Dashboard]
```

### **3.5 High-Level Limitations**

* Cloud is **read-only** and receives periodic or event-based state snapshots.
* Cloud cannot trigger actuators (safety constraint).
* Local PostgreSQL may grow large; retention strategies apply.
* ESP32 must use isolated circuits for pH/EC probes.

### **3.6 Environmental Assumptions**

* Facility has stable power; Mini-PC is UPS-protected.
* ESP32 firmware implements watchdogs and safe restart routines.
* Hydroponic installation uses electrical safety standards (GFCI, isolation).
* Local network uses dedicated SSID/VLAN or wired Ethernet.

## **4. Business Architecture**

This section describes SmartCrops' *business-capability-oriented* architecture from a technical perspective, without functional details.
It maps the system into **domains** and **subdomains**, following TOGAF and domain-driven architecture.

SmartCrops is composed of the following business-technical capabilities:

### **4.1 Capability Map (High-Level)**

#### **A. Hydroponic Automation**

* Irrigation automation (Flood & Drain)
* Actuator coordination (pump, fans, extractor)
* Sensor-driven decision loop
* Growth phase-driven parameterization

#### **B. Environmental Monitoring**

* Acquisition of environmental variables (air temp, humidity)
* Water-level and tray-state verification
* Nutrient parameters (pH, EC)

#### **C. Local Operations**

* Local UI (Next.js)
* Local backend (Fastify)
* MQTT event distribution
* Local-only authentication (Basic login)

#### **D. Data Management**

* Local PostgreSQL data store
* High-frequency time-series retention
* Cloud sync pipeline (WebSockets push)
* Event metadata catalog

#### **E. Cloud Visualization**

* Cloud dashboard (read-only)
* Cloud backend
* Multi-installation data separation
* Cloud query API (private)

#### **F. System Management**

* Override logic
* Calibration workflows
* Maintenance task management
* Diagnostic tests
* Logging & observability

### **4.2 Domains and Subdomains**

#### **Domain 1 — Automation Domain**

**Subdomains:**

1. **Irrigation Control Subdomain**
2. **Climate Control Subdomain**
3. **Actuator Scheduling Subdomain**
4. **Hydro-Safety Subdomain**

#### **Domain 2 — Sensor Telemetry Domain**

**Subdomains:**

1. **Analog/Nutrient Sensors (pH, EC)**
2. **Environmental Sensors (Temp/Humidity)**
3. **Water-Level/Flow Sensors**
4. **Actuator State Telemetry**

#### **Domain 3 — Local Runtime Domain**

**Subdomains:**

1. **Local Web Application**
2. **Local Backend**
3. **MQTT Messaging Layer**
4. **Local Auth & Permissions**

#### **Domain 4 — Data & Sync Domain**

**Subdomains:**

1. **Local DB Storage**
2. **Cloud Sync Engine**
3. **Long-Term Archival**
4. **Event Queues**

#### **Domain 5 — Cloud Visualization Domain**

**Subdomains:**

1. **Cloud Dashboard UI**
2. **Cloud Backend**
3. **Cloud Data API**
4. **User Authentication (Clerk)**

## **5. Logical Architecture**

This section describes how SmartCrops is structured logically: modules, layers, boundaries, interdependencies, and responsibilities.

The system follows a **modular monolithic architecture**, deployed via Docker Compose, with **strong domain separation** and **clear inter-module contracts**.

### **5.1 Logical Layering**

#### **Layer 1 — Presentation Layer**

* Local Next.js Web UI
* Cloud Next.js Web UI
* Authentication UI components

#### **Layer 2 — API & Application Layer**

* Fastify-based REST API (local)
* Fastify-based REST API (cloud)
* Cloud Sync Controller (WebSocket client/server)

#### **Layer 3 — Domain Logic Layer**

* Automation Orchestrator
* Sensor Manager
* Scheduler Engine
* Safety Controller
* Override Manager
* Calibration Manager
* Maintenance Manager

#### **Layer 4 — Integration Layer**

* MQTT broker
* MQTT event dispatcher
* Serial drivers / UART / I2C / 1-Wire interfaces
* ESP32 communication handlers

#### **Layer 5 — Data Layer**

* PostgreSQL (local)
* PostgreSQL or SQLite (cloud VPS)
* ORM / Query Layer
* Time-series partitions
* Data retention layer

### **5.2 Logical Modules**

#### **Module A — Local Web UI**

* Next.js application
* Real-time dashboards via WebSockets
* Auth (Basic local)
* User roles (Admin, Technician, Viewer)

#### **Module B — Local Backend (Fastify TS)**

* REST API endpoints
* Automation engine
* Growth phase controller
* Scheduler (Cron-like job runner)
* WebSocket server
* Sync Manager
* Sensor/Actuator service abstraction
* Logging & metrics exporters

#### **Module C — Messaging Layer**

* Mosquitto MQTT broker
* Pub/sub channels:

  * `sensors/#`
  * `actuators/#`
  * `system/events`
  * `sync/outbound`
* Decouples backend and ESP32

#### **Module D — ESP32 Firmware**

* UART/I2C/1-Wire sensor reading
* Actuator control GPIO/SSR
* Local safety checks
* MQTT publisher/subscriber
* Watchdog & failsafe mechanisms

#### **Module E — Local Database**

* PostgreSQL instance
* Tables for:

  * Sensor readings
  * Irrigation events
  * Actuator events
  * Calibration logs
  * Maintenance events
  * Sync queues
* Partitioned time-series tables

#### **Module F — Cloud Backend**

* Fastify API
* Sync ingestion endpoints
* Clerk authentication
* Cross-installation data separation

#### **Module G — Cloud Dashboard**

* Next.js UI
* Read-only dashboards
* Aggregation views
* Installation switcher

## **6. Physical / Infrastructure Architecture**

This section defines the **actual physical topology**, including compute nodes, networks, subnets, firewalls, and deployment units.

### **6.1 Physical Components**

#### **Local On-Premises Components**

1. **Mini-PC (Intel i5)**

   * Runs Docker Engine + Docker Compose
   * Runs:

     * Local Backend
     * Local Next.js UI
     * MQTT Broker
     * PostgreSQL
     * Sync Client

2. **ESP32 MCU**

   * Runs firmware for sensors & actuators
   * Connected via UART/I2C/1-Wire/GPIO
   * Connects to local MQTT broker

#### **Cloud Components (Google Cloud VPS)**

1. **Cloud VPS (Compute Engine)**

   * Runs Docker Engine + Compose
   * Runs:

     * Cloud Backend (Fastify)
     * Cloud Dashboard (Next.js)
     * Cloud DB (PostgreSQL or SQLite)
     * WebSocket Sync Server
   * One VPS handles all roles for MVP

### **6.2 Network Architecture**

#### **Local Network**

* Local LAN (wired preferred)
* ESP32 connects via WiFi or wired UART-to-USB bridge
* Local system has no public ingress
* Only HTTPS exposed is **Cloud VPS**, not local

#### **Cloud Network**

* Google Cloud VCN (10.20.0.0/16)
* Subnets:

  * `10.20.10.0/24` Web
  * `10.20.20.0/24` Core
  * `10.20.30.0/24` DB
  * `10.20.40.0/24` Monitoring
* Firewall: deny-by-default

### **6.3 Connectivity Model**

#### **Local Connectivity**

```
Mini-PC ↔ ESP32 (MQTT)
Mini-PC ↔ PostgreSQL (localhost)
Mini-PC ↔ Local Web UI (localhost:3000)
```

#### **Cloud Connectivity**

```
Local Backend → Cloud Sync Server (WebSockets)
Cloud Dashboard → Cloud Backend
Cloud Backend → Cloud DB
```

#### **No inbound access from cloud to local.**

### **6.4 Deployment Units (per VPS / per device)**

#### **Local Mini-PC Containers**

* `sc_local_backend`
* `sc_local_ui`
* `sc_mqtt`
* `sc_postgres`
* `sc_sync_client`

#### **ESP32**

* Firmware binary (UART/I2C/1-Wire drivers + MQTT)

#### **Cloud VPS Containers**

* `sc_cloud_backend`
* `sc_cloud_ui`
* `sc_cloud_db`
* `sc_sync_server`

### **6.5 Infrastructure Constraints**

* No Kubernetes
* No load balancer
* No autoscaling
* Only docker-compose
* MVP cloud uses a single VPS

### **6.6 Physical Architecture Summary (Text Diagram)**

```
                 ┌───────────────────────────────────────────────┐
                 │         LOCAL HYDROPONIC SYSTEM (On-Prem)     │
                 │                                               │
                 │  ┌───────────┐   MQTT   ┌──────────────────┐  │
                 │  │  ESP32    │<-------->│  Mini-PC (Intel) │  │
                 │  │ Sensors & │          │  Docker Host     │  │
                 │  │ Actuators │          ├──────────────────┤  │
                 │  └───────────┘          │Local Backend     │  │
                 │                         │Local UI          │  │
                 │                         │PostgreSQL        │  │
                 │                         │MQTT Broker       │  │
                 │                         │Sync Client       │  │
                 │                         └──────────────────┘  │
                 └───────────────────────────────────────────────┘

                               Internet (Outbound Only)
                                       ↑ WebSockets
                                       |

               ┌────────────────────────────────────────────────────┐
               │               GOOGLE CLOUD VPS (1 node)            │
               │                                                    │
               │  ┌──────────────────┐   ┌──────────────────────┐   │
               │  │ Cloud Backend    │   │ Cloud Dashboard (UI) │   │
               │  ├──────────────────┤   └──────────────────────┘   │
               │  │ Sync Server (WS) │   │ PostgreSQL/SQLite    │   │
               │  └──────────────────┘   └──────────────────────┘   │
               └────────────────────────────────────────────────────┘
```

## **7. Application Architecture**

This section describes how the application is structured internally, covering frontends, backends, services, modules, workers, and execution flows.

The system follows a **modular-monolithic application architecture** deployed through Docker Compose across local and cloud environments.

### **7.1 Application Overview**

SmartCrops consists of two fully independent operational applications:

#### **A. Local Application (Authoritative Runtime)**

* Runs all real-time automation logic
* Interfaces directly with sensors/actuators via ESP32
* Ensures safe, autonomous operation
* Exposes a local Web UI for configuration, monitoring, and diagnostics
* Stores all data locally
* Synchronizes with cloud in an asynchronous, outbound-only manner

#### **B. Cloud Application (Read-Only Visualization)**

* Receives synchronized state & historical data
* Provides a web dashboard for remote viewing
* Does **not** control actuators (safety constraint)
* Uses Clerk for user authentication

### **7.2 Local Application Components**

#### **1. Local Web UI (Next.js)**

Responsibilities:

* Real-time dashboards
* Control interface (manual flood, override, etc.)
* Alerts visualization
* Calibration workflows
* Maintenance management
* Local auth (Basic login)
* Offline operation

**Communication:**

* WebSockets with Local Backend
* REST calls for configuration changes

#### **2. Local Backend (Fastify + TypeScript)**

Core center of the system.

**Responsibilities:**

* Automation & event orchestration
* Growth-phase engine
* Flood & Drain scheduler
* Climate control engine
* Data storage orchestrator
* MQTT subscriber/publisher
* WebSocket server for UI
* Cloud sync client
* Validation & safety guards
* Logging (Pino)
* Metrics exporters

#### **3. Automation Engine**

Implements real-time logic based on sensor input:

* Flood & Drain cycle engine
* Pump control
* Water-level validation
* EC and pH threshold evaluators
* Environmental threshold enforcement
* Actuator failsafe mechanisms

#### **4. Sensor Manager (via MQTT)**

* Receives raw sensor messages from ESP32
* Normalizes units, checks validity
* Stores readings in PostgreSQL
* Emits events for the Automation Engine

#### **5. Sync Manager (WebSocket Client)**

* Bundles outbound datasets
* Sends them to cloud in intervals or events
* Manages retry logic, exponential backoff
* Queues unsent batches locally

#### **6. Local Database Interface**

* Postgres connector
* ORM or query builder
* Time-series partitioning
* Storage of events, logs, sync states

### **7.3 Cloud Application Components**

#### **1. Cloud Web UI (Next.js)**

* Displays synchronized data
* Installation selector
* Historical charts
* Alerts viewer
* Clerk-based authentication

#### **2. Cloud Backend (Fastify)**

Responsibilities:

* Receives sync payloads
* Stores cloud-side data
* Handles metadata for installations
* Queries for Cloud UI
* Verifies installation tokens

#### **3. Sync Server (WebSocket Server)**

* Receives batches from local Sync Client
* Validates schema
* Inserts into cloud DB
* Acknowledges reception

### **7.4 Application Layer Boundaries**

Clear module boundaries are enforced:

```
UI Layer  →  API Layer  →  Domain Logic  →  Integration Layer → DB/MQTT
```

All cross-domain interactions go through **Application Services**, never directly.

## **8. Integration Architecture**

This section defines all communication protocols, message flows, interfaces, APIs, queues, events, and data contracts.

### **8.1 Integration Principles**

* All integration is **local-first**
* ESP32 communicates via **MQTT**
* Cloud communication is **outbound-only (WebSockets)**
* No public APIs on local system
* Cloud provides only read-only APIs

### **8.2 Local Integration**

#### **Primary Integration Bus: MQTT (Mosquitto)**

Topics include:

| Topic                | Direction       | Purpose              |
| -------------------- | --------------- | -------------------- |
| `sensors/ph`         | ESP32 → Backend | pH readings          |
| `sensors/ec`         | ESP32 → Backend | EC readings          |
| `sensors/temp_air`   | ESP32 → Backend | Temp/Humidity        |
| `sensors/temp_water` | ESP32 → Backend | DS18B20 readings     |
| `sensors/level`      | ESP32 → Backend | Water level state    |
| `actuators/pump`     | Backend → ESP32 | Pump commands        |
| `actuators/fan`      | Backend → ESP32 | Fan/extractor        |
| `system/events`      | Both             | Diagnostics & events |

MQTT ensures decoupling between timing-sensitive MCU tasks and backend logic.

### **8.3 Microcontroller Integration**

#### **ESP32 Interfaces**

* **UART** → pH, EC modules
* **I2C** → air sensors (SHT31/AHT20)
* **1-Wire** → DS18B20 water temp
* **GPIO/Relays** → Pump, fan, extractor
* **MQTT client** → event publication

Drivers managed with a firmware abstraction layer.

### **8.4 Local API (Private)**

The Local Backend exposes REST endpoints:

```
GET /status
POST /actuators/<device>
GET /sensors/recent
POST /config/irrigation
POST /config/thresholds
POST /override
POST /calibration
POST /maintenance
```

This API is **not exposed to internet**.

### **8.5 Local WebSocket Interface**

Used for UI:

* Live sensor updates
* Actuator state updates
* Irrigation cycle state
* Alerts
* Logs (tailing)

### **8.6 Cloud Synchronization Integration**

#### **Protocol: WebSockets**

Chosen for reliability and push-efficiency.

**Sync Flow**:

1. Local Sync Manager queues events/readings
2. Establishes WS connection to Cloud
3. Sends compressed JSON batches
4. Cloud Sync Server validates and stores
5. Acknowledgement returned
6. Local clears batch from queue

No inbound communication is allowed from cloud.

### **8.7 Cloud REST APIs (Read-Only)**

Endpoints (secured with Clerk):

```
GET /installations
GET /installations/:id/sensors
GET /installations/:id/events
GET /installations/:id/alerts
GET /installations/:id/cycles
```

Dashboards pull data from these endpoints.

## **9. Data Architecture**

This section defines data models, logical/physical schemas, retention, indexing, and storage patterns.

### **9.1 Data Architecture Principles**

* Local PostgreSQL is the **source of truth**
* Cloud DB is a **replica for visualization**
* Data sync is **asynchronous**
* Storage must support **multi-year history**
* Data ingestion is continuous (10–60 sec for most readings)
* Data models must be optimized for time-series workloads
* Cloud storage is optimized for read queries

### **9.2 Logical Data Model**

#### **Core Entities**

| Entity               | Description                              |
| -------------------- | ---------------------------------------- |
| **SensorReading**    | pH, EC, temp, humidity, water level      |
| **ActuatorEvent**    | pump/fan/extractor commands and outcomes |
| **IrrigationCycle**  | flood/drain cycle logs                   |
| **GrowthPhase**      | current/previous phase info              |
| **AlertEvent**       | threshold deviations and safety events   |
| **CalibrationEvent** | sensor calibration logs                  |
| **MaintenanceTask**  | scheduled tasks                          |
| **SyncBatch**        | pending/processed sync payloads          |

### **9.3 Physical Data Model (Local PostgreSQL)**

#### **Tables**

```
sensor_readings
    id (pk)
    timestamp (ts indexed)
    sensor_type (ph, ec, temp_air,…)
    value
    metadata (jsonb)

irrigation_cycles
    id
    start_ts
    end_ts
    flood_duration
    drain_duration
    status (success/failure)
    reason

actuator_events
    id
    timestamp
    actuator (pump, fan,…)
    command (on/off)
    duration

alerts
    id
    timestamp
    type
    severity
    details jsonb

calibrations
    id
    sensor_type
    timestamp
    user
    notes

maintenance_events
    id
    task_id
    timestamp
    notes

sync_queue
    id
    payload jsonb
    created_at
    status (pending/sent)
```

### **9.4 Time-Series Storage Strategy**

Because tomatoes require high-frequency monitoring (10–60s intervals), the DB must handle **millions of rows per year**.

Recommended approach:

* Partition `sensor_readings` table by **month**
* Index `(sensor_type, timestamp)`
* Enable gzip compression for JSONB payloads
* Use sequential IDs for ingestion speed


### **9.5 Cloud Data Model**

Mirrors the local schema but optimized for read-side:

* Aggregated views
* Daily partitions instead of monthly
* Slimmer tables: fewer metadata fields

### **9.6 Data Retention Policies**

#### **Local**

* Raw sensor data retained for **3–5 years**
* Older partitions archived to external storage (optional)

#### **Cloud**

* Last 12 months of full data
* Older data aggregated (min/max/avg per hour)

### **9.7 Data Access Flows**

#### **Local Operation Flow**

```
ESP32 → MQTT → Backend → PostgreSQL → UI (WebSocket)
```

#### **Cloud Sync Flow**

```
PostgreSQL (local) → Sync Manager → WebSocket → Cloud Server → Cloud DB → Cloud UI
```

## **10. Security Architecture**

SmartCrops follows a **Zero-Trust, Local-First Security Model** aligned with OWASP, NIST SP 800-53, and cloud/network security best practices.

Security is divided into:

* Local security
* Cloud security
* Communication security
* Identity & access
* Credential management
* Data protection
* Operational hardening

### **10.1 Security Principles**

1. **No inbound access** to the local system
2. **All automation stays on-premise**
3. **Cloud is read-only**
4. **Least privilege** everywhere
5. **Deny-by-default networking**
6. **All channels encrypted** (TLS 1.2+)
7. **APIs are private**
8. **Role-Based Access** enforced locally and in cloud
9. **Secure partition of duties** between local & cloud

### **10.2 Identity & Access Management**

#### **Local IAM**

* Authentication: **Basic login** (username/password), stored locally
* Authorization: 3 roles per FSR 

  * Administrator
  * Technician
  * Viewer
* The Local UI enforces role-based visibility and access
* Privileged actions require explicit confirmation (override, manual flood)

#### **Cloud IAM**

* **Clerk Authentication Provider** (OAuth-compliant)
* Roles defined per installation:

  * Owner
  * Viewer
* Multi-installation enforced via scoped JWT (installationId)

## **10.3 Network Security**

#### **Local Network**

* Local backend/UI not exposed to Internet
* Access only through LAN
* Optional firewall rules to restrict based on MAC or IP
* ESP32 communicates only to MQTT broker on Mini-PC
* Mini-PC runs Debian 13 with:

  * UFW enabled
  * SSH disabled or restricted
  * No root login
  * No open ports externally

#### **Cloud Network**

* Google Cloud VCN (10.20.0.0/16)
* Subnets per your template: Web / Core / DB / Monitoring
* Firewall deny-all except:

  * HTTPS 443 (Cloud UI + API)
  * WebSocket sync endpoint
* No SSH exposed publicly (tunnel or private key restricted)

### **10.4 Communication Security**

#### **Local communication**

* HTTP → Localhost only
* WebSocket → Localhost
* MQTT → LAN only (port 1883)
* ESP32 uses **MQTT with authentication** (username/password)
* Optionally TLS MQTT if performance allows

#### **Local → Cloud Sync**

* WebSockets over TLS (wss://)
* Installation-level API token included in headers
* Sync payloads signed via HMAC (ADR-SEC-003)

### **10.5 Data Security & Encryption**

#### **At Rest**

* PostgreSQL local → Data directory protected with filesystem permissions
* Cloud DB → Encrypted via Linux filesystem encryption or disk-level encryption
* Passwords hashed with **bcrypt**
* Secrets stored in **.env** not committed to Git

#### **In Transit**

* TLS-only endpoints
* MQTT optional TLS depending on MCU capabilities
* WebSockets secured via modern cipher suites

### **10.6 Secrets & Credential Management**

* Local `.env` loaded via Docker Compose

* Cloud `.env` stored in VPS (root-owned, restricted)

* API Tokens:

  * InstallationToken (local→cloud)
  * Clerk tokens (cloud UI)

* ESP32 credentials baked at flash time:

  * MQTT user/password
  * WiFi credentials
  * Optional deviceId

### **10.7 Hardening**

* Disable unneeded services on Mini-PC
* Fail2ban for repeated login attempts
* File system permission lockdown
* Regular package updates (scripted)
* Resource limits on containers (CPU/mem)
* Cloud VM hardened with CIS benchmark baseline

### **10.8 Security Monitoring**

* Pino structured logs for audit
* Prometheus metrics on auth failures
* Alerting via cloud dashboards (future phase)

## **11. DevOps & Environments Architecture**

This section covers containers, deployment flows, versioning, CI, CD, observability, and environment segregation.

### **11.1 Environments**

#### **Local Environment (Authoritative Runtime)**

* Runs on-prem
* All containers deployed on Mini-PC
* No external dependencies
* Real-time logic runs here

#### **Cloud Environment (Visualization Runtime)**

* Runs in Google Cloud VPS
* Independent of local availability
* Stores synchronized data
* Read-only dashboards

### **11.2 Deployment Model**

#### **Local Mini-PC**

* Docker Engine + docker-compose
* Services run in isolated containers
* Restart policy: always
* Local `.env` mounted at runtime

```
docker-compose.local.yml
  - local_backend
  - local_ui
  - mqtt
  - postgres
  - sync_client
```

#### **Cloud VPS**

```
docker-compose.cloud.yml
  - cloud_backend
  - cloud_ui
  - cloud_db
  - sync_server
```

Cloud DB may use PostgreSQL or SQLite depending on VPS resources.

### **11.3 CI/CD Pipeline**

For MVP: **Manual CI/CD**, simple and reliable.

#### **Local System**

* No automated CI/CD
* Deployment triggered manually via SSH or VSCode Remote
* UI and Backend built locally

#### **Cloud System**

* Manual build + deploy
* Steps:

  1. Pull code
  2. Build containers
  3. Restart services

Future improvement: GitHub Actions.

## **11.4 Observability Architecture**

#### **Local**

* Logging: Pino (JSON)
* Metrics: Prometheus Node Exporter + custom backend metrics
* UI: Internal monitoring dashboard (minimal MVP)

#### **Cloud**

* Logs: Cloud Backend logs
* Metrics: Node Exporter
* Grafana (optional on cloud VPS)

### **11.5 Backup & Recovery**

### **Local**

* DB snapshots stored daily
* Manual off-device backup recommended

#### **Cloud**

* VPS disk snapshots weekly
* Manual trigger via cloud console

### **11.6 Release Strategy**

* **Blue/Green** per VPS (your template)
* Containers versioned via tags
* Configs externalized

### **11.7 Infrastructure-as-Code**

Not required for MVP since no managed services or Kubernetes.
Optional future: Terraform for VPS provisioning.

## **12. Architectural Patterns Applied**

SmartCrops adopts several industry-standard architectural patterns.

### **12.1 Modular Monolith Pattern**

* The entire local backend is a monolithic Fastify app
* Strict domain modularization inside codebase
* Reduces deployment complexity

### **12.2 Event-Driven Architecture**

* MQTT acts as the internal event bus
* ESP32 sensor readings → published as events
* Backend automation reacts to events

### **12.3 CQRS-Lite Pattern**

* Separation between:

  * Commands (actuator operations, override, config)
  * Queries (read-only sensor/history data)

### **12.4 Gateway Aggregator Pattern**

Cloud backend acts as a lightweight API gateway for dashboards.

### **12.5 Local-First Architecture**

All core logic is onsite, ensuring resilience and offline autonomy.

### **12.6 Outbound-Only Sync Pattern**

Inspired by secure ICS/SCADA architectures.
Local → Cloud only, never reverse.

### **12.7 Time-Series Data Architecture**

* Partitioned sensor readings
* Writes optimized for fast ingestion
* Read-optimized aggregates on cloud

## **13. Architectural Decisions (ADR Summaries)**

All decisions use the ADR tag format requested: **ADR-XXX-###**, where XXX = category (ARC, SEC, INT, DAT, OPS, etc.).

Below are **summary versions**; full ADRs would be stored separately.

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

## **14. Architectural Risks & Mitigation**

Complies with TOGAF risk evaluation.

### **14.1 Local System Risks**

#### **R-001 – Sensor Drift (pH/EC) Leading to Incorrect Automation**

* **Impact:** High
* **Likelihood:** High
* **Mitigation:**

  * Scheduled calibrations (FSR-mandated)
  * Alerts for out-of-range drift
  * Use of industrial-grade isolated probes

#### **R-002 – MQTT Broker Failure**

* **Impact:** High
* **Likelihood:** Medium
* **Mitigation:**

  * Automatic restart policies
  * Watchdog monitoring
  * Lightweight broker for minimal crash scenarios

#### **R-003 – Local PostgreSQL Disk Exhaustion**

* **Impact:** High
* **Likelihood:** Medium
* **Mitigation:**

  * Partitioning + retention policies
  * Disk usage alerts
  * Cloud-side aggregation

#### **R-004 – ESP32 WiFi Instability**

* **Impact:** Medium
* **Likelihood:** High
* **Mitigation:**

  * Ethernet UART bridge optional
  * Retry logic
  * Keepalive pings

### **14.2 Cloud System Risks**

#### **R-010 – Cloud VPS Outage**

* **Impact:** Low for operations, Medium for visibility
* **Mitigation:**

  * Local-first design prevents downtime
  * Sync resumes when available

#### **R-011 – Unauthorized Access to Cloud Dashboard**

* **Impact:** Medium
* **Likelihood:** Low
* **Mitigation:**

  * Clerk authentication
  * HTTPS-only
  * JWT scoped tokens

### **14.3 Integration Risks**

#### **R-020 – Loss of Sync Events**

* **Impact:** Low (as system continues locally)
* **Mitigation:**

  * Sync queue
  * Retry mechanism with exponential backoff

#### **R-021 – Network Latency / Unstable Connectivity**

* **Impact:** Low
* **Mitigation:**

  * Local-first logic
  * Time-stamped batching

## **15. Technical Assumptions**

Based on requirements + MVP constraints.

1. Mini-PC has stable power, UPS recommended
2. ESP32 is electrically isolated from pumps (SSR/relay isolation)
3. WiFi network has acceptable coverage or UART bridge is used
4. PostgreSQL uses SSD storage
5. Sensors operate within expected temperature ranges
6. Mechanical flood/drain system is correctly installed
7. Cloud VPS runs Debian 13 or Ubuntu 22.04
8. Only 1 hydroponic system per installation (FSR BR-001)
9. Local system is installed indoors
10. No AI analytics until Phase II

## **16. Limitations / Exclusions**

These reflect explicit boundaries of the architecture.

1. No cloud → local actuation
2. No Kubernetes or autoscaling
3. No multi-system management per installation
4. No AI-based predictions / analytics in MVP
5. No managed databases (Cloud SQL, Firestore)
6. No real-time global telemetry (cloud is eventually consistent)
7. No role federation between local and cloud
8. No remote firmware updates for ESP32 (manual flash only)
9. Local system does not support multi-tenant users
10. Cloud does not provide high availability (single VPS)

## **17. C4 Diagrams**

Following the C4 Model levels required: Context (L1), Container (L2), Component (L3).
Diagrams are rendered textually using structured notation.

### **17.1 C4 Level 1 – System Context Diagram**

```
+--------------------------------------------------------------+
|                        SmartCrops System                     |
|       Local Autonomous Hydroponic Automation Platform        |
+--------------------------------------------------------------+
|                                                              |
|  - Executes all control logic locally                        |
|  - Provides real-time UI                                     |
|  - Stores all historical data                                |
|  - Synchronizes with optional cloud dashboard                |
|                                                              |
+---------------------+              +-------------------------+
|   Local Users       |              |  Cloud Users (Remote)   |
|  Admin/Tech/View    |              |  Admin/Viewer (Clerk)   |
+----------+----------+              +------------+------------+
           |                                       |
           | HTTP/WebSocket (LAN)                  | HTTPS/Web
           v                                       v
   [Local Web UI]                            [Cloud Dashboard]
           |                                       |
           | WS/MQTT/Internal                      | REST/Graph APIs
           v                                       v
   [Local Backend] ------------------> [Cloud Backend + Sync Server]
           |                                       |
           | PostgreSQL                            | PostgreSQL/SQLite
           v                                       v
   [Local DB]                                  [Cloud DB]
           |
   [ESP32 Sensor/Actuator Node]
```

### **17.2 C4 Level 2 – Container Diagram**

#### **Local System Containers**

```
+-----------------------------------------------------------------------------------+
|                                   LOCAL SYSTEM                                     |
+----------------------------------------+------------------------------------------+
| Container: Local Web UI (Next.js)     | Container: Local Backend (Fastify TS)    |
| - Real-time dashboards                | - Automation engine                       |
| - Local auth                          | - Sensor ingestion via MQTT               |
| - Config interfaces                   | - Actuator control via MQTT               |
|                                        | - Growth phase logic                      |
+----------------------------------------+------------------------------------------+
| Container: MQTT Broker (Mosquitto)     | Container: PostgreSQL (Local)            |
| - Sensor events                        | - Time-series readings                    |
| - Actuator commands                    | - Irrigation and alert logs               |
| - System events                        | - Sync queue                              |
+----------------------------------------+------------------------------------------+
| Container: Sync Client                 | Device: ESP32 MCU                         |
| - Bundles outbound sync payloads       | - UART/I2C/1-Wire sensing                 |
| - WebSocket to cloud sync server       | - MQTT publish/subscribe                  |
+----------------------------------------+------------------------------------------+
```

---

#### **Cloud System Containers**

```
+-----------------------------------------------------------------------------------+
|                                   CLOUD SYSTEM                                     |
+-----------------------------------------------------------------------------------+
| Container: Cloud Dashboard (Next.js)            | Container: Cloud Backend        |
| - Read-only visualization                       | - API for dashboard             |
| - Clerk auth                                     | - Receives sync batches         |
|                                                  | - Installation scoping          |
+--------------------------------------------------+--------------------------------+
| Container: Cloud Sync Server                     | Container: Cloud DB             |
| - WebSocket server                               | - Stores synchronized data      |
| - Batch validation                               | - Query-optimized tables        |
+--------------------------------------------------+--------------------------------+
```

### **17.3 C4 Level 3 – Component Diagram (Local Backend)**

```
+----------------------------------------------------------------------------------+
|                          Local Backend (Fastify, TypeScript)                     |
+----------------------------------------------------------------------------------+
| Components:                                                                       |
|                                                                                   |
| 1. API Controller Layer                                                           |
|    - Exposes REST endpoints                                                       |
|    - Validates incoming requests                                                  |
|                                                                                   |
| 2. WebSocket Gateway                                                              |
|    - Emits live events to Local Web UI                                            |
|    - Subscribes to automation + sensor changes                                    |
|                                                                                   |
| 3. Automation Engine                                                               |
|    - Flood & Drain cycle management                                               |
|    - Climate control decisions                                                    |
|    - Growth phase logic                                                           |
|    - Safety checks                                                                |
|                                                                                   |
| 4. Sensor Manager                                                                  |
|    - MQTT subscriber                                                              |
|    - Normalizes sensor data                                                       |
|    - Stores readings in DB                                                        |
|                                                                                   |
| 5. Actuator Controller                                                             |
|    - MQTT publisher to ESP32                                                      |
|    - Tracks actuator state                                                        |
|                                                                                   |
| 6. Scheduler Engine                                                                |
|    - Cron-like scheduler                                                          |
|    - Irrigation cycles                                                            |
|    - Maintenance reminders                                                         |
|                                                                                   |
| 7. Sync Manager (Client)                                                           |
|    - Reads sync_queue table                                                       |
|    - Sends WebSocket batches                                                      |
|    - Retries + exponential backoff                                                |
|                                                                                   |
| 8. Data Access Layer                                                               |
|    - PostgreSQL queries                                                           |
|    - Time-series partition utilities                                              |
|    - ORM or query builder                                                         |
|                                                                                   |
| 9. Logging & Metrics                                                               |
|    - Pino structured logger                                                       |
|    - Prometheus exporters                                                         |
|                                                                                   |
+----------------------------------------------------------------------------------+
```

## **18. Technical Annexes**

The following annexes consolidate technical parameters, standards, and configuration references used throughout the SAD.

### **18.1 Sensor Specifications (Recommended for Hydroponic Tomato Production)**

| Sensor              | Type                                | Bus              | Frequency    | Notes                                 |
| ------------------- | ----------------------------------- | ---------------- | ------------ | ------------------------------------- |
| pH                  | Analog + Isolation or EZO UART      | UART             | every 30–60s | Avoid noise interference              |
| EC                  | Analog or EZO UART                  | UART             | every 30–60s | Temp compensation recommended         |
| Temp/Humidity (air) | SHT31/AHT20                         | I2C              | every 20–30s | Stable outdoor/hot-house environments |
| Temp (water)        | DS18B20                             | 1-Wire           | every 30s    | Waterproof                            |
| Water Level         | Float Sensor / Ultrasonic JSN-SR04T | GPIO / UART      | event + 10s  | Critical for irrigation safety        |
| Actuators           | Pump/Fans                           | GPIO (SSR/Relay) | 5s           | Status polling                        |

### **18.2 Local Network Configuration**

* Local backend: `127.0.0.1:4000`
* Local UI: `127.0.0.1:3000`
* MQTT Broker: `localhost:1883` (LAN-only)
* DB: `localhost:5432`
* No public IP exposure

### **18.3 Cloud Network Configuration**

Google Cloud VCN:

```
10.20.0.0/16
  ├─ Web Subnet:       10.20.10.0/24
  ├─ Core Subnet:      10.20.20.0/24
  ├─ DB Subnet:        10.20.30.0/24
  └─ Monitoring Subnet:10.20.40.0/24
```

### **18.4 Docker Compose Layouts**

#### **local-compose.yml**

```
services:
  local_backend:
  local_ui:
  mqtt:
  postgres:
  sync_client:
```

#### **cloud-compose.yml**

```
services:
  cloud_backend:
  cloud_ui:
  cloud_db:
  sync_server:
```

### **18.5 Security Standards Referenced**

* OWASP ASVS
* OWASP Top 10
* NIST SP 800-53
* CIS Linux Hardening
* TLS 1.2+ best practices

### **18.6 Time-Series Partition Strategy**

* `sensor_readings`: **monthly partitions**
* Index: `(sensor_type, timestamp DESC)`
* Vacuum tuned for high-ingest workloads
* Gzip compression for JSONB metadata

### **18.7 Default Threshold Configuration (Tomato Hydroponics)**

*(From agricultural reference profiles; not functional logic)*

| Parameter  | Recommended Range                               |
| ---------- | ----------------------------------------------- |
| pH         | 5.8–6.3                                         |
| EC         | 2.0–4.0 mS/cm                                   |
| Temp (air) | 20–28°C                                         |
| RH         | 60–75%                                          |
| Irrigation | Flood 3–6 min every 30–60 min (light-dependent) |

Values not implemented as logic here; FSR defines functional behavior.

## **CLOSING NOTES**

This **SAD v1.0** is a full, enterprise-grade architectural specification aligned with TOGAF, C4, IEEE 42010, your tech-stack template, and the SmartCrops FSR.

It defines:

* End-to-end system architecture
* Logical, physical, data, security, integration and DevOps models
* C4 diagrams
* ADR summaries
* Technical annex references
