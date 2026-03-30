# Unified Solar + Battery + Smart Energy Platform  
**Product Requirements Specification (v0.1)**  
**Date:** 2026‑03‑29  

---

## 0. Scope & Glossary

### 0.1 Scope

This document defines high‑level requirements for a **unified, modular power‑electronics + software platform** that starts as a **Solar + Battery + Smart Control** appliance and can be configured (without redesign) to also serve:

- **Residential projects** (towers/townships, common‑area loads, optional EV)  
- **Charge Point Operator (CPO) sites** (fast‑charging stations with solar + storage)

The platform must share the same hardware and software building blocks across variants, differing mainly by **module count and configuration**.

### 0.2 Glossary

- **CCM** – Common Control Module (central controller + comms)  
- **PMU** – Power Module Unit (10–25 kW building block)  
- **BESS** – Battery Energy Storage System  
- **EV** – Electric Vehicle  
- **CPO** – Charge Point Operator  
- **ToD Tariff** – Time‑of‑Day electricity tariff  

---

## 1. Product Goals

**P‑1 Purpose**  
Provide a **multi‑input → multi‑output energy appliance** that can integrate grid, solar PV, and battery on a common DC bus and feed AC loads and optional DC EV outputs, controlled by a unified software stack.

**P‑2 Target Segments**

- **Residential:**  
  - New and existing residential projects (towers, gated communities, townships)  
  - Use cases: common‑area bills reduction, backup power, green‑building compliance, EV‑ready infrastructure  

- **CPO:**  
  - Public and fleet fast‑charging sites (highways, hubs, depots)  
  - Use cases: lower energy cost per kWh, peak‑shaving, faster and modular expansion of site power

**P‑3 Design Principles**

- **Modular**: Scale from ~10 kW to ≥150 kW by adding PMUs.  
- **Configurable**: Same CCU/PMU hardware, different product behaviour via configuration.  
- **De‑populatable**: Solar‑only, Solar+Battery, full EV‑ready variants by omitting boards, not redesigning.  
- **Vertical**: Tight HW+SW integration for reliability, cost, and grid‑service capability.

---

## 2. Hardware Requirements

### 2.1 Common Control Module (CCM)

**R‑HW‑1**  
A single CCM design shall be used across all product variants (Home / Residential Project / CPO).

**R‑HW‑2**  
CCM shall include at minimum:

- Real‑time controller (MCU/DSP) for power‑conversion loops  
- Communications: Ethernet, RS‑485/CAN; expansion port for LTE/5G modem  
- Hardware secure element for device identity, keys, and configuration profile  
- Local HMI: status LEDs and service port; optional small display connector

**R‑HW‑3**  
Firmware shall be field‑upgradeable via both:

- Local service port  
- Remote over‑the‑air (OTA) updates  

### 2.2 Power Module Units (PMU)

**R‑HW‑4**  
A single PMU rating (10–25 kW) shall be defined as the primary power building block.

**R‑HW‑5**  
PMUs shall be connectable in parallel on the common DC bus to scale system power from:

- **Minimum**: 10 kW  
- **Target upper bound (per cabinet)**: ≥150 kW  

**R‑HW‑6**  
Each PMU shall be assignable in software to perform any of the following roles:

- PV DC‑DC conversion (MPPT role)  
- Bi‑directional BESS DC‑DC conversion  
- Grid‑tied inverter / AC link  
- EV DC output stage (DC fast‑charging)

### 2.3 Common DC Bus

**R‑HW‑7**  
The system shall provide a single high‑voltage DC bus to which all PMUs and interface modules connect.

- Nominal design point: around 800 V (with a 400 V variant achievable by BOM changes, not PCB redesign)

**R‑HW‑8**  
A low‑voltage DC rail (e.g., 48 V) may be used for auxiliaries and control but shall not be used as a main energy bus.

### 2.4 Interface Modules

**R‑HW‑9 Grid AC Interface**

- 3‑phase 415 V AC, bidirectional  
- Include necessary protections: breakers, contactors, anti‑islanding, metering  

**R‑HW‑10 Solar PV Interface**

- Accept PV string inputs up to ~1 000 V DC  
- Provide MPPT capability (either integrated into PMUs or via dedicated front‑end)  

**R‑HW‑11 BESS Interface**

- Connect high‑voltage battery via contactors and pre‑charge circuitry  
- Communicate with BMS using CAN; configurable for different pack capacities and chemistries  

**R‑HW‑12 EV Interface (Optional)**

- Provide DC EV outputs using CCS2 initially (via external or integrated controller)  
- Optionally provide AC Type‑2 outputs for lower‑power EV charging  
- EV interface modules must be field‑addable to an existing Solar+Battery cabinet without modifying core PMUs or CCM

### 2.5 De‑population & Variants

**R‑HW‑13**  
The same core PCB set shall support at least these variants:

- Solar‑only  
- Solar+Battery  
- Solar+Battery+EV (full CPO SKU)

By **not** populating certain interface boards or components, rather than re‑spinning the main hardware.

---

## 3. Software Requirements

### 3.1 Architecture & Configuration

**R‑SW‑1**  
A single firmware codebase shall serve all product variants.

**R‑SW‑2**  
Runtime behaviour shall be controlled by a **configuration profile** stored in secure memory, including:

- Product mode: Home / Residential‑Block / CPO  
- Maximum site power (kW)  
- Enabled interfaces (PV, BESS, EV, demand‑response)  
- Tariff model (flat, ToD, custom)

### 3.2 Core Control & Optimisation

**R‑SW‑3 Real‑Time Control**

The firmware shall implement:

- MPPT for PV inputs  
- Bi‑directional BESS charging with SoC/SoH constraints  
- Grid interconnection logic (voltage/frequency support, anti‑islanding)

**R‑SW‑4 Energy Management**

The system shall:

- Maximise on‑site solar self‑consumption  
- Minimise electricity cost under flat and ToD tariffs by scheduling:  
  - Battery charging (from solar vs grid)  
  - Battery discharging (into loads / EVs / grid export)  
- Implement configurable peak‑shaving: enforce a site peak‑power cap (kW)

**R‑SW‑5 Forecasting**

The system shall provide:

- Simple day‑ahead and intra‑day forecasts of:  
  - Site load (AC + EV where present)  
  - PV generation  

Using recent historical data and, where available, weather API inputs.

**R‑SW‑6 Predictive Maintenance**

The system shall:

- Monitor PMU and key component metrics (temperatures, switching patterns, fault counters)  
- Detect deviations from normal behaviour and raise early‑warning maintenance alerts

### 3.3 CPO‑Specific Logic

**R‑SW‑7 Power Allocation**

For CPO mode (EV outputs enabled), the system shall:

- Allocate available DC power across EV connectors based on:  
  - Queue position and session age  
  - Vehicle SoC and requested energy  
  - Expected dwell time (if provided)  
- Enforce connector‑level and site‑level power limits

**R‑SW‑8 Revenue‑Aware Operation**

In CPO mode, the system shall:

- Use tariff and session‑pricing inputs to minimise **energy cost per kWh delivered** at target utilisation  
- Support product differentiation (e.g., “standard” vs “priority” charging speeds)

**R‑SW‑9 Protocols & Integration**

- Support OCPP 1.6/2.0.1 for EV session management when EV modules are present  
- Expose northbound REST/gRPC APIs for:  
  - Billing and CRM systems  
  - Fleet dashboards  
- Be architected for future integration with **India Energy Stack / Utility Intelligence Platform** APIs (meter data, DR signals) as they become available

### 3.4 Fleet & Cloud

**R‑SW‑10 Fleet Management**

Each unit shall be able to connect to a central fleet platform to:

- Send periodic telemetry (energy by source, state, faults, utilisation)  
- Receive configuration updates and firmware updates  
- Provide aggregated reporting for residential customers and CPOs

---

## 4. Performance & Business Requirements

### 4.1 Technical Performance

**R‑PERF‑1**  
PV→BESS→AC/EV round‑trip efficiency shall be ≥ **89–92 %** at representative operating points.

**R‑PERF‑2**  
Per‑site availability (excluding utility‑side outages) shall target ≥ **99 %**.

**R‑PERF‑3**  
Cold‑start (power restored to system ready) shall be ≤ **60 seconds**.

### 4.2 Residential Projects

**R‑BUS‑RES‑1**  
For typical PV sizing and tariffs, the system should target **≥ 20 % reduction** in common‑area electricity spend vs grid‑only baseline when BESS is installed.

**R‑BUS‑RES‑2**  
Provide data granularity and export required for popular green‑building certifications (e.g., GRIHA/LEED energy monitoring credits).

**R‑BUS‑RES‑3**  
Adding EV interface modules to an existing Solar+Battery installation shall not require changes to CCM or PMU hardware and shall minimise site downtime.

### 4.3 CPO Sites

**R‑BUS‑CPO‑1**  
For medium‑to‑high utilisation fast‑charging sites, Solar+Battery deployment should target **40–60 % reduction** in effective grid‑energy cost per kWh delivered vs a grid‑only fast‑charging baseline.

**R‑BUS‑CPO‑2**  
The Solar+Battery add‑on CAPEX shall be sized so that a typical CPO site can achieve **payback in ≤ 5–7 years**, assuming representative Indian tariffs and utilisation.

**R‑BUS‑CPO‑3**  
The platform must support **incremental scale‑up** by:

- Adding PMUs to increase site power (e.g., from 60 kW to 120 kW)  
- Adding EV interface modules as demand grows  

Without redesigning or re‑certifying the core CCM/PMU hardware.

**R‑BUS‑CPO‑4**  
New CPO sites using standardised layouts should be deployable (post‑utility approvals) within **≤ 8 weeks** from order to commissioning, assuming local EPC availability.

---

## 5. Non‑Functional Requirements

**R‑NF‑1 Safety & Compliance**

- Design to comply with relevant IEC/IS standards for PV inverters, BESS, and EV charging equipment.  
- Implement all mandatory protections (OV/UV, OC, ground fault, isolation monitoring, anti‑islanding).

**R‑NF‑2 Security**

- All external communications shall be encrypted (e.g., TLS).  
- Device identities and configuration shall be anchored in a secure element; firmware shall be signed and verified at boot and update.

**R‑NF‑3 Maintainability**

- PMUs and interface boards shall be field‑replaceable modules.  
- The software shall support versioning, remote diagnostics, and safe roll‑back of failed updates.

---

## 6. Open Questions (to refine in next iteration)

- Final PMU rating (10, 15, 20, or 25 kW) and voltage levels (400 vs 800 V nominal)  
- Exact list of target standards (IEC/IS numbers) for each market  
- Specific tariff / utilisation assumptions for CPO payback calculations  
- Priority order for segment roll‑out: residential vs CPO vs mixed sites  
