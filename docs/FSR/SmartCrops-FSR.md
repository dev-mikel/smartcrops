# **Functional Specification Requirements (FSR)**
**Project** SmartCrops
**Version:** 1.0
**Author:** Miguel Ladines

Perfecto. Aquí tienes **una Tabla de Contenidos completamente funcional (GitHub-compatible)** para tu **FSR**, con las anclas corregidas según el contenido real del archivo
.

La puedes **copiar/pegar directamente** al inicio del FSR.

---

# **📄 Tabla de Contenidos — FSR (Corregida, 100% funcional)**

> Anclas generadas según reglas de GitHub Markdown:
>
> * Minúsculas
> * Sin caracteres especiales
> * Espacios → guiones
> * Títulos con numeración → la numeración cuenta para el slug

---

# **Table of Contents**

* [1. Introduction and Scope](#1-introduction-and-scope)

  * [1.1 Purpose](#11-purpose)
  * [1.2 Functional Scope](#12-functional-scope)
  * [1.3 Out of Scope](#13-out-of-scope)
* [2. Glossary](#2-glossary)
* [3. Actors / Users](#3-actors--users)

  * [3.1 Roles](#31-roles)
  * [3.2 Permission Matrix](#32-permission-matrix)
* [4. Functional Requirements (FR)](#4-functional-requirements-fr)

  * [4.1 Core System Behavior](#41-core-system-behavior)
  * [4.2 Growth Phases](#42-growth-phases)
  * [4.3 Nutrient Management](#43-nutrient-management)
  * [4.4 Flood & Drain Irrigation](#44-flood--drain-irrigation)
  * [4.5 Environmental Control](#45-environmental-control)
  * [4.6 Manual Control and Override](#46-manual-control-and-override)
  * [4.7 Alerts & Notifications](#47-alerts--notifications)
  * [4.8 Calibration & Diagnostics](#48-calibration--diagnostics)
  * [4.9 Maintenance Management](#49-maintenance-management)
  * [4.10 Data Storage and Handling](#410-data-storage-and-handling)
* [5. Non-Functional Requirements (User-Visible Only)](#5-non-functional-requirements-user-visible-only)
* [6. Business Rules](#6-business-rules)
* [7. Use Cases](#7-use-cases)

  * [UC-01 — Start a New Crop](#uc-01--start-a-new-crop)
  * [UC-02 — Configure Irrigation Schedule](#uc-02--configure-irrigation-schedule)
  * [UC-03 — Monitor System Status](#uc-03--monitor-system-status)
  * [UC-04 — Manual Flood Activation](#uc-04--manual-flood-activation)
  * [UC-05 — Respond to Irrigation Failure](#uc-05--respond-to-irrigation-failure)
  * [UC-06 — Calibrate Sensor](#uc-06--calibrate-sensor)
  * [UC-07 — Perform Maintenance Task](#uc-07--perform-maintenance-task)
* [8. Functional Flows](#8-functional-flows)

  * [Flow 1 — Standard Flood & Drain Cycle](#flow-1--standard-flood--drain-cycle)
  * [Flow 2 — Phase Change](#flow-2--phase-change)
  * [Flow 3 — Offline Operation](#flow-3--offline-operation)
* [9. Wireframes (Textual)](#9-wireframes-textual)

  * [9.1 Dashboard](#91-dashboard)
  * [9.2 Irrigation Page](#92-irrigation-page)
  * [9.3 Nutrient Management Page](#93-nutrient-management-page)
  * [9.4 Environmental Control Page](#94-environmental-control-page)
* [10. Interactions Between Modules](#10-interactions-between-modules)
* [11. Assumptions](#11-assumptions)
* [12. Exclusions](#12-exclusions)

---

## **1. Introduction and Scope**

### **1.1 Purpose**

This document defines the functional behavior of the **SmartCrops Platform**, a system designed to manage, automate, and monitor **Flood & Drain hydroponic environments**.
It specifies *what the system must do* from a user-visible and operational standpoint, covering local and cloud behavior, control logic, user interactions, data flows, alerts, history, and offline autonomy.

### **1.2 Functional Scope**

The system provides:

* Fully autonomous local operation of a hydroponic installation.
* Local real-time monitoring through a Web App accessible on-site.
* Cloud interface for remote visibility and historical analytics.
* Configurable crop profiles and growth phases.
* Control of irrigation, environment, and nutrient management.
* Alerts, maintenance scheduling, calibration, and reporting.
* Long-term high-frequency sensor data retention.
* User roles with differentiated permissions.

### **1.3 Out of Scope**

The following are not included in this FSR:

* Hardware schematics, electrical wiring, embedded firmware.
* Network protocols, communication technologies, or architecture.
* Database engines, schemas, or physical storage structures.
* Deployment models, infrastructure, containers, and security stacks.
* Business models, pricing, monetization, or commercial logic.
* Technical design, internal algorithms, or implementation details.

## **2. Glossary**

| Term                    | Definition                                                                |
| ----------------------- | ------------------------------------------------------------------------- |
| **SmartCrops**          | Complete platform for hydroponic automation and monitoring.               |
| **Local Server**        | On-site gateway providing full offline functionality and local Web App.   |
| **Cloud Platform**      | Remote interface for viewing synchronized data and historical analytics.  |
| **Control Loop**        | Sequence of reading sensors, evaluating rules, and controlling actuators. |
| **Growth Phase**        | Development stage of a crop (Vegetative, Pre-Flowering, Fruiting).        |
| **Flood & Drain Cycle** | Sequence of filling and draining the grow tray.                           |
| **Override Mode**       | Manual mode where automatic actions are disabled.                         |
| **Maintenance Task**    | Scheduled operations such as cleaning or solution renewal.                |
| **Sensor Event**        | A measurement recorded at a point in time.                                |

## **3. Actors / Users**

### **3.1 Roles**

#### **Administrator**

* Full access to all system modules.
* Configures operational parameters, crop profiles, and schedules.
* Manages users and permissions.
* Performs maintenance, calibration, reporting, and override actions.

#### **Technician**

* Performs operational actions such as manual control, maintenance, and calibration (when permitted).
* Views dashboards, alerts, and history.

#### **Viewer**

* Read-only access to dashboards, alerts, and historical records.

### **3.2 Permission Matrix**

| Function                      | Admin | Technician | Viewer |
| ----------------------------- | :---: | :--------: | :----: |
| View dashboards & alerts      |   ✔   |      ✔     |    ✔   |
| View history & reports        |   ✔   |      ✔     |    ✔   |
| Modify crop profiles & phases |   ✔   |      ✘     |    ✘   |
| Modify irrigation parameters  |   ✔   |      ✘     |    ✘   |
| Modify environment thresholds |   ✔   |      ✘     |    ✘   |
| Manual actuator control       |   ✔   |  Optional  |    ✘   |
| Activate/Deactivate override  |   ✔   |  Optional  |    ✘   |
| Calibration                   |   ✔   |  Optional  |    ✘   |
| Maintenance scheduling        |   ✔   |  Optional  |    ✘   |
| User management               |   ✔   |      ✘     |    ✘   |

## **4. Functional Requirements (FR)**

### **4.1 Core System Behavior**

#### **FR-001 — Local Autonomy**

The platform shall continue functioning fully, including the control loop and Local Web App, when disconnected from the Internet.

#### **FR-002 — Cloud Visibility**

The platform shall provide a Cloud Web App where users can view synchronized historical data and the last known state of the local system.

#### **FR-003 — Continuous Operation**

The platform shall execute all monitoring, automation, and control functions continuously without requiring user interaction.

#### **FR-004 — Data Persistence**

The platform shall store measurement history, events, configurations, and logs locally for long-term retention.

### **4.2 Growth Phases**

#### **FR-GP-001 — Phase Definitions**

The system shall support at least three configurable growth phases:

* Vegetative
* Pre-Flowering
* Flowering/Fruiting

Each phase shall support:

* Target pH range
* Target EC range
* Irrigation frequency and duration
* Environmental thresholds

#### **FR-GP-002 — Automatic Progression**

The system shall automatically determine the current phase based on configured duration.

#### **FR-GP-003 — Manual Phase Selection**

Administrators shall be able to manually switch the active growth phase.

#### **FR-GP-004 — Parameter Application**

When the phase changes, all relevant system parameters shall update immediately.

### **4.3 Nutrient Management**

#### **FR-NM-001 — Monitoring**

The system shall display real-time and historical pH and EC values.

#### **FR-NM-002 — Threshold Alerts**

The system shall generate alerts when pH or EC values remain outside the configured target range beyond a defined tolerance period.

#### **FR-NM-003 — Solution Renewal**

The system shall allow scheduling periodic nutrient solution renewals and generate reminders when due or overdue.

### **4.4 Flood & Drain Irrigation**

#### **FR-FD-001 — Cycle Configuration**

The system shall allow configuration of:

* Flood duration
* Flood frequency
* Active scheduling window
* Maximum allowed fill and drain times

#### **FR-FD-002 — Automatic Execution**

The system shall execute Flood & Drain cycles according to schedule.

#### **FR-FD-003 — Manual Activation**

Authorized users shall be able to manually initiate a Flood cycle.

#### **FR-FD-004 — Water Level Validation**

The system shall monitor tray water levels to validate successful fill and drain actions.

#### **FR-FD-005 — Cycle Failure Alerts**

The system shall generate alerts when:

* Flood fails to reach the expected level
* Drain does not complete in time
* Pump exceeds safe runtime

### **4.5 Environmental Control**

#### **FR-CL-001 — Monitoring**

The system shall monitor air temperature and humidity in real time.

#### **FR-CL-002 — Threshold Configuration**

Users shall configure high and low thresholds for environmental variables.

#### **FR-CL-003 — Automatic Climate Control**

The system shall activate fans and extractors according to threshold logic.

#### **FR-CL-004 — Environmental Alerts**

Alerts shall be generated when values remain above or below thresholds for a sustained period.

### **4.6 Manual Control and Override**

#### **FR-MC-001 — Manual Actuator Control**

Authorized users shall be able to manually operate:

* Pump
* Fans
* Extractor
* Valves (if present)

#### **FR-MC-002 — Override Mode**

Enabling override shall disable all automatic control functions until manually deactivated.

#### **FR-MC-003 — Visual Indicator**

The interface shall clearly indicate when override mode is active.

### **4.7 Alerts & Notifications**

#### **FR-AL-001 — Alert Types**

The system shall support alerts for:

* Out-of-range sensor values
* Irrigation failures
* Environmental deviations
* Overdue maintenance
* Calibration requirements

#### **FR-AL-002 — Presentation**

Alerts shall be displayed:

* On the dashboard
* On a dedicated alerts page
* On relevant module pages

### **4.8 Calibration & Diagnostics**

#### **FR-CAL-001 — Sensor Calibration**

The system shall provide guided calibration procedures for supported sensors.

#### **FR-CAL-002 — Calibration Records**

Calibration events shall be logged with timestamp, user, and outcome.

#### **FR-CAL-003 — Diagnostics**

The system shall provide diagnostic tests for actuators, enabling short test activations.

### **4.9 Maintenance Management**

#### **FR-MA-001 — Task Scheduling**

The system shall allow recurring maintenance tasks to be defined.

#### **FR-MA-002 — Reminders**

The system shall generate reminders when tasks are due.

#### **FR-MA-003 — Task Completion Logging**

Users shall record completion with timestamp and optional notes.

### **4.10 Data Storage and Handling**

#### **FR-DS-001 — High-Frequency Data Support**

The system shall be capable of storing high-frequency sensor data for long-term retention.

#### **FR-DS-002 — Local Storage Capacity**

The system shall provide sufficient local storage to maintain multiple years of historical data.

#### **FR-DS-003 — Cloud Synchronization**

The system shall perform asynchronous synchronization of historical data and configurations when connectivity is available.

#### **FR-DS-004 — Connection Status**

Users shall be able to view the last known synchronization timestamp and connection status.

## **5. Non-Functional Requirements (User-Visible Only)**

### **NFR-001 — Offline Availability**

All critical monitoring and control functions shall remain available without Internet connectivity.

### **NFR-002 — User Interface Responsiveness**

Real-time values displayed in the Local Web App shall update with minimal perceptible delay.

### **NFR-003 — Safety Visibility**

Critical states shall be distinctly highlighted (e.g., irrigation failure, high temperature).

### **NFR-004 — Usability**

Actions capable of causing crop damage (override, manual flood, deletion of profiles) shall require confirmation.

### **NFR-005 — Long-Term Data Handling**

The system shall support multi-year historical visualization without performance degradation from a user perspective.

## **6. Business Rules**

### **BR-001 — Single System Operation**

The platform manages one hydroponic system per installation.

### **BR-002 — Automation Precedence**

Automatic logic is disabled when override mode is active.

### **BR-003 — Safety Priority**

Irrigation safety checks take precedence over all other automated functions.

### **BR-004 — Irreversible Deletions**

Deleting crop profiles or associated history is irreversible.

## **7. Use Cases**

### **UC-01 — Start a New Crop**

**Actor:** Administrator
**Goal:** Initialize a crop using a chosen profile.

**Main Flow:**

1. User opens Crop Setup.
2. Selects a crop profile.
3. Enters start date and confirms.
4. System activates the first growth phase and applies parameters.
5. Dashboard displays the active crop.

### **UC-02 — Configure Irrigation Schedule**

**Actor:** Administrator
**Goal:** Set Flood & Drain behavior.

**Main Flow:**

1. User opens irrigation settings.
2. Configures flood duration and frequency.
3. Sets schedule constraints.
4. Saves configuration.
5. System applies updated schedule.

### **UC-03 — Monitor System Status**

**Actor:** All roles
**Goal:** View real-time operational status.

**Main Flow:**

1. User opens dashboard.
2. System displays sensor values, actuator states, alerts, and phase.

### **UC-04 — Manual Flood Activation**

**Actor:** Administrator or authorized Technician
**Goal:** Trigger an immediate Flood cycle.

**Main Flow:**

1. User selects Manual Flood.
2. Confirms the action.
3. System starts flood cycle.
4. System monitors tray levels.
5. Event is logged.

### **UC-05 — Respond to Irrigation Failure**

**Actor:** Administrator or Technician
**Goal:** Address a flood or drain failure.

**Main Flow:**

1. System raises alert.
2. User opens alert details.
3. User acknowledges alert.
4. User performs physical inspection.

### **UC-06 — Calibrate Sensor**

**Actor:** Administrator or authorized Technician

**Main Flow:**

1. User opens Calibration.
2. Selects a sensor type.
3. Follows guided steps.
4. System logs calibration.

### **UC-07 — Perform Maintenance Task**

**Actor:** Administrator or Technician

**Main Flow:**

1. User accesses Maintenance section.
2. Reviews tasks due.
3. Completes physical work.
4. Marks task as completed.
5. System records completion.

## **8. Functional Flows**

### **Flow 1 — Standard Flood & Drain Cycle**

1. System scheduler triggers flood.
2. Pump activates.
3. Water level increases until safe threshold.
4. Pump deactivates.
5. Drain phase begins.
6. System verifies full drain.
7. Event is recorded.

### **Flow 2 — Phase Change**

1. System evaluates crop timeline.
2. Phase end date is reached.
3. System switches to next phase.
4. Updated parameters are applied.

### **Flow 3 — Offline Operation**

1. Internet connectivity is lost.
2. Local Web App remains fully functional.
3. All automation continues.
4. History continues to accumulate locally.
5. Sync occurs automatically when connectivity returns.

## **9. Wireframes (Textual)**

### **9.1 Dashboard**

* Crop name and phase
* Connectivity status
* Key sensor tiles (pH, EC, water temp, air temp, humidity)
* Irrigation status
* Climate control status
* Alerts summary
* Buttons for manual flood, override, maintenance, and calibration

### **9.2 Irrigation Page**

* Flood & Drain configuration
* Current schedule
* Last cycles and outcomes
* Manual activation button

### **9.3 Nutrient Management Page**

* pH and EC live readings
* Target ranges
* Alerts
* Solution renewal schedule

### **9.4 Environmental Control Page**

* Temperature and humidity
* Thresholds
* Fan/extractor states
* Climate alerts

## **10. Interactions Between Modules**

* Dashboard ↔ Monitoring: Displays real-time sensor values and alarms.
* Phase Manager → Irrigation / Nutrients / Climate: Applies updated parameters when phases change.
* Alerts Module ↔ All modules: Generates alerts based on deviations.
* Maintenance Module ↔ Alerts: Issues reminders when tasks are due.
* Local Storage ↔ Cloud Sync: Synchronizes data asynchronously.

## **11. Assumptions**

* A single hydroponic system is operated per installation.
* Mechanical and electrical installation is functioning correctly.
* Users understand basic hydroponic processes.
* Local and cloud interfaces access the same logical dataset, with cloud data reflecting the last synchronization point.

## **12. Exclusions**

* Multi-system or multi-zone management.
* Automatic nutrient dosing or chemical injection.
* Predictive analytics or AI-based recommendations.
* Integration with external greenhouse platforms.
* Remote actuation from the cloud beyond safe design constraints.
