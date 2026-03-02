# Integrated Power Conversion Unit (IPCU) - Planning Document

**Project Start:** 2026-02-24  
**Status:** Planning Phase  
**Author:** EV Charging Infrastructure Team

---

## 1. Executive Summary

### 1.1 Document Purpose

This document presents **two distinct product architectures** based on a common Integrated Power Conversion Unit (IPCU) platform. Both share the same DC-bus-centric design philosophy but target different markets and use cases.

### 1.2 The Two Architecture Options

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                     TWO IPCU ARCHITECTURE OPTIONS                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│   ┌────────────────────────────────┐      ┌────────────────────────────────┐   │
│   │     OPTION A: IPCU-STATION     │      │     OPTION B: VIES/UPM-HOME    │   │
│   │     (Commercial/Highway)       │      │     (Residential/Small Biz)    │   │
│   ├────────────────────────────────┤      ├────────────────────────────────┤   │
│   │                                │      │                                │   │
│   │   ┌────┐  ┌────┐  ┌────┐      │      │   ┌────┐  ┌────┐  ┌────┐      │   │
│   │   │Grid│  │Solar│  │BESS│      │      │   │Grid│  │Solar│  │ EV │      │   │
│   │   │250k│  │100k │  │500k│      │      │   │ 10k│  │ 10k │  │V2H │      │   │
│   │   └──┬─┘  └──┬─┘  └──┬─┘      │      │   └──┬─┘  └──┬─┘  └──┬─┘      │   │
│   │      │       │       │        │      │      │       │       │        │   │
│   │      └───────┼───────┘        │      │      └───────┼───────┘        │   │
│   │              ▼                │      │              ▼                │   │
│   │     ┌────────────────┐        │      │     ┌────────────────┐        │   │
│   │     │  800V DC BUS   │        │      │     │ 48-400V DC BUS │        │   │
│   │     └───────┬────────┘        │      │     └───────┬────────┘        │   │
│   │             │                 │      │             │                 │   │
│   │      ┌──────┴──────┐          │      │      ┌──────┴──────┐          │   │
│   │      ▼             ▼          │      │      ▼             ▼          │   │
│   │  ┌───────┐    ┌───────┐       │      │  ┌───────┐    ┌───────┐       │   │
│   │  │DC Fast│    │AC+V2G │       │      │  │AC Type│    │ Home  │       │   │
│   │  │60-180k│    │ 75kW  │       │      │  │  2    │    │ Loads │       │   │
│   │  └───────┘    └───────┘       │      │  └───────┘    └───────┘       │   │
│   │                               │      │                               │   │
│   │  STORAGE: Fixed 100-500 kWh   │      │  STORAGE: EV Battery (V2H)    │   │
│   │  BATTERY: Stationary BESS     │      │  + Optional Aux 5-20 kWh      │   │
│   │                               │      │                               │   │
│   │  TARGET COST: Rs. 30-40 Lakhs │      │  TARGET COST: Rs. 1.5-4 Lakhs │   │
│   └────────────────────────────────┘      └────────────────────────────────┘   │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 1.3 Quick Comparison

| Aspect | **OPTION A: IPCU-STATION** | **OPTION B: VIES/UPM-HOME** |
|--------|---------------------------|----------------------------|
| **Market** | Commercial, Highway, Fleet | Residential, Small Business |
| **Power Rating** | 60-250 kW | 5-30 kW |
| **DC Bus Voltage** | 800V (or 400V/1000V) | 48V or 400V |
| **Energy Storage** | Fixed BESS (100-500 kWh) | EV Battery (V2H) + Optional Aux |
| **Primary Output** | DC Fast Charging (CCS2) | Home loads, EV charging (AC) |
| **Solar Integration** | 50-200 kWp | 2-10 kWp |
| **Key Innovation** | Unified AC+V2G inverter | EV as storage (zero battery cost) |
| **Target Price** | Rs. 30-40 Lakhs | Rs. 1.5-4 Lakhs |
| **Installation** | Dedicated land, permits | Residential, plug-and-play |
| **Typical Customer** | CPO, Fleet operator, Mall | Homeowner, SME, Apartment |

### 1.4 Why Two Options?

| Need | IPCU-STATION Answer | VIES/UPM Answer |
|------|---------------------|-----------------|
| **High-power DC charging** | ✅ 60-180 kW per port | ❌ Not designed for this |
| **Grid services/V2G revenue** | ✅ Large-scale dispatch | ✅ Aggregated V2G |
| **Zero battery investment** | ❌ Requires BESS purchase | ✅ Uses existing EV battery |
| **Home backup power** | ❌ Overspec'd for homes | ✅ Purpose-built |
| **Scalability** | ✅ Add charger modules | ✅ Add aux battery modules |
| **Solar self-consumption** | ✅ Peak shaving | ✅ Store for evening use |

### 1.5 Product Family Overview

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           IPCU PRODUCT FAMILY                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│      COMMERCIAL                              RESIDENTIAL                        │
│      ──────────                              ───────────                        │
│                                                                                 │
│  ┌─────────────────┐                    ┌─────────────────┐                     │
│  │  IPCU-STATION   │                    │  UPM-HOME-5     │                     │
│  │  250 kW         │                    │  5 kW           │                     │
│  │  Rs.30-40L      │                    │  Rs.1.5L        │                     │
│  │  Highway/Fleet  │                    │  Apartment      │                     │
│  └────────┬────────┘                    └────────┬────────┘                     │
│           │                                      │                              │
│  ┌────────┴────────┐                    ┌────────┴────────┐                     │
│  │  IPCU-MINI      │                    │  UPM-HOME-10    │                     │
│  │  60-120 kW      │                    │  10 kW          │                     │
│  │  Rs.15-25L      │                    │  Rs.2.5L        │                     │
│  │  Retail/Office  │                    │  Most homes     │                     │
│  └────────┬────────┘                    └────────┴────────┘                     │
│           │                                      │                              │
│  ┌────────┴────────┐                    ┌────────┴────────┐                     │
│  │  IPCU-HOME      │◄───────────────────│  UPM-HOME-15    │                     │
│  │  10-25 kW       │   Bridge Product   │  15 kW          │                     │
│  │  Rs.8-12L       │   (Fixed BESS)     │  Rs.3.5L        │                     │
│  │  Villa/SME      │                    │  Villa/SME      │                     │
│  └─────────────────┘                    └────────┬────────┘                     │
│                                                  │                              │
│                                         ┌────────┴────────┐                     │
│                                         │  UPM-BIZ-30     │                     │
│                                         │  30 kW          │                     │
│                                         │  Rs.6L          │                     │
│                                         │  Small business │                     │
│                                         └─────────────────┘                     │
│                                                                                 │
│  ═══════════════════════════════════════════════════════════════════════════    │
│  KEY DIFFERENCE:                                                                │
│  IPCU-* = Fixed stationary battery (BESS) included                              │
│  UPM-*  = EV battery (V2H) as primary storage, aux battery optional             │
│  ═══════════════════════════════════════════════════════════════════════════    │
│                                                                                 │
└────────────────────────────────────────────────────────────────────────────────-┘
```

### 1.6 Document Structure

| Part | Section | Content |
|------|---------|---------|
| **PART I** | 2-7 | IPCU-STATION: Commercial architecture, hardware, costs |
| **PART II** | 8 | VIES/UPM-HOME: Residential architecture, EV-as-storage |
| **PART III** | 9-12 | Common: Development roadmap, risks, next steps |

---

## 1.7 UNIFIED PLATFORM ARCHITECTURE: One Hardware, Many Configurations

### The Platform Concept

Instead of building two separate product lines, we design a **single modular platform** with:
- **Common hardware building blocks** (power modules, interfaces, control)
- **Software-defined product configuration** (home vs station behavior)
- **Scalability through module count** (add more bricks = more power)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    UNIFIED MODULAR PLATFORM CONCEPT                             │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│                        ┌──────────────────────────────┐                        │
│                        │    SOFTWARE CONFIGURATION    │                        │
│                        │    ────────────────────────  │                        │
│                        │  • Product Mode (Home/Stn)   │                        │
│                        │  • Power Limits              │                        │
│                        │  • Charging Protocols        │                        │
│                        │  • Grid Services Enable      │                        │
│                        │  • Billing/OCPP              │                        │
│                        └──────────────┬───────────────┘                        │
│                                       │                                         │
│                                       ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                     COMMON CONTROL MODULE (CCM)                          │   │
│  │           TI C2000 / STM32H7 DSP + Linux Gateway (Jetson/RPi)           │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                       │                                         │
│              ┌────────────────────────┼────────────────────────┐               │
│              │                        │                        │               │
│              ▼                        ▼                        ▼               │
│  ┌───────────────────┐   ┌───────────────────┐   ┌───────────────────┐        │
│  │   POWER MODULE    │   │   POWER MODULE    │   │   POWER MODULE    │  ...   │
│  │   UNIT (PMU)      │   │   UNIT (PMU)      │   │   UNIT (PMU)      │        │
│  │   25kW Brick      │   │   25kW Brick      │   │   25kW Brick      │        │
│  └─────────┬─────────┘   └─────────┬─────────┘   └─────────┬─────────┘        │
│            │                       │                       │                   │
│            └───────────────────────┼───────────────────────┘                   │
│                                    │                                            │
│                    ┌───────────────┴───────────────┐                           │
│                    │      COMMON DC BUS            │                           │
│                    │   48V / 400V / 800V           │                           │
│                    │   (Jumper/SW Selectable)      │                           │
│                    └───────────────┬───────────────┘                           │
│                                    │                                            │
│         ┌──────────────────────────┼──────────────────────────┐                │
│         │                          │                          │                │
│         ▼                          ▼                          ▼                │
│  ┌─────────────┐           ┌─────────────┐           ┌─────────────┐          │
│  │ INTERFACE   │           │ INTERFACE   │           │ INTERFACE   │          │
│  │ MODULE      │           │ MODULE      │           │ MODULE      │          │
│  │             │           │             │           │             │          │
│  │ • Grid AC   │           │ • EV Port   │           │ • Battery   │          │
│  │ • Solar DC  │           │   (CCS2)    │           │   (BESS/V2H)│          │
│  │             │           │ • AC Type-2 │           │ • Aux Batt  │          │
│  └─────────────┘           └─────────────┘           └─────────────┘          │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 1.7.1 Common Hardware Building Blocks

#### Core Module: Power Module Unit (PMU) - 25kW Brick

The **PMU** is the fundamental building block - a 25kW bidirectional power converter:

```
┌─────────────────────────────────────────────────────────────────┐
│              POWER MODULE UNIT (PMU) - 25kW BRICK               │
│              Dimensions: 300 × 200 × 150 mm                     │
│              Weight: 12 kg │ Cost Target: Rs. 2.5L              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DC IN/OUT ◄───┐       ┌─────────────────┐       ┌───► AC OUT  │
│  (Bus Side)    │       │                 │       │    (Load)   │
│                │       │   SiC MOSFET    │       │             │
│  400-900V DC ◄─┼──────►│   Full Bridge   │◄─────►├──► 3φ 415V  │
│                │       │   + LCL Filter  │       │    or       │
│                │       │                 │       │    1φ 230V  │
│                │       └────────┬────────┘       │             │
│                │                │                │             │
│                │       ┌────────┴────────┐       │             │
│                │       │  LOCAL DSP      │       │             │
│                │       │  (TMS320F28)    │       │             │
│                │       │  PWM, Protection│       │             │
│                │       └────────┬────────┘       │             │
│                │                │ CAN            │             │
│                └────────────────┼────────────────┘             │
│                                 ▼                               │
│                          TO: CCM (Master)                       │
│                                                                 │
│  SPECIFICATIONS:                                                │
│  ├─ Bidirectional: AC↔DC or DC↔DC                              │
│  ├─ Efficiency: 97-98%                                         │
│  ├─ Switching: 50-100 kHz (SiC)                                │
│  ├─ Isolation: 4kV galvanic (optional)                         │
│  ├─ Cooling: Liquid-ready (G1/4 fittings)                      │
│  └─ Comms: CAN-FD to master controller                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Interface Modules (Plug-in)

| Module | Function | Voltage | Power | Cost Target |
|--------|----------|---------|-------|-------------|
| **IFM-GRID** | Grid AC interface (AFE) | 3φ 415V / 1φ 230V | 25-100kW | Rs. 80k |
| **IFM-SOLAR** | Solar MPPT input | 150-600V DC | 10-25kW | Rs. 40k |
| **IFM-BESS** | Stationary battery DC-DC | 48-800V DC | 25-100kW | Rs. 60k |
| **IFM-EV-DC** | CCS2/CHAdeMO DC output | 200-920V DC | 25-180kW | Rs. 1.5L |
| **IFM-EV-AC** | Type-2 AC output | 1φ/3φ 230/415V | 7-22kW | Rs. 30k |
| **IFM-V2H** | V2H bidirectional (CCS2) | 200-800V DC | 7-11kW | Rs. 60k |
| **IFM-AUX** | Aux battery slot (48V LFP) | 48V DC | 5-20kWh | Rs. 20k |

### 1.7.2 Product Configurations from Common Modules

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                   PRODUCT CONFIGURATIONS FROM COMMON MODULES                    │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  UPM-HOME-5 (5kW)                              Cost: Rs. 1.5L           │   │
│  │  ──────────────────                                                      │   │
│  │  ┌─────┐                                                                 │   │
│  │  │ PMU │──┬──[IFM-GRID]──► Grid (1φ 230V, 25A)                          │   │
│  │  │ 25kW│  │                                                              │   │
│  │  │ @20%│  ├──[IFM-SOLAR]──► Solar (2-5 kWp)                             │   │
│  │  │     │  │                                                              │   │
│  │  └─────┘  └──[IFM-V2H]──► EV Battery (V2H) OR [IFM-AUX]──► 5kWh Aux     │   │
│  │                                                                          │   │
│  │  SW Config: HOME_MODE, 5kW_LIMIT, V2H_ENABLED, OCPP_DISABLED            │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  UPM-HOME-10 (10kW)                            Cost: Rs. 2.5L           │   │
│  │  ───────────────────                                                     │   │
│  │  ┌─────┐                                                                 │   │
│  │  │ PMU │──┬──[IFM-GRID]──► Grid (1φ 230V, 50A or 3φ 415V)               │   │
│  │  │ 25kW│  │                                                              │   │
│  │  │ @40%│  ├──[IFM-SOLAR]──► Solar (5-10 kWp)                            │   │
│  │  │     │  │                                                              │   │
│  │  └─────┘  ├──[IFM-V2H]──► EV Battery (V2H, 7-11kW bidirectional)        │   │
│  │           │                                                              │   │
│  │           └──[IFM-AUX]──► 5-10 kWh Aux Battery (optional)               │   │
│  │                                                                          │   │
│  │  SW Config: HOME_MODE, 10kW_LIMIT, V2H_ENABLED, DEPARTURE_SCHED         │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  IPCU-MINI (60kW)                              Cost: Rs. 15-20L         │   │
│  │  ────────────────                                                        │   │
│  │  ┌─────┐ ┌─────┐ ┌─────┐                                                │   │
│  │  │ PMU │ │ PMU │ │ PMU │──┬──[IFM-GRID]──► Grid (3φ 415V, 100A)         │   │
│  │  │ 25kW│ │ 25kW│ │ 25kW│  │                                              │   │
│  │  └──┬──┘ └──┬──┘ └──┬──┘  ├──[IFM-SOLAR]×2──► Solar (20-50 kWp)         │   │
│  │     │       │       │     │                                              │   │
│  │     └───────┼───────┘     ├──[IFM-BESS]──► 100 kWh BESS                  │   │
│  │             │             │                                              │   │
│  │        [DC BUS 800V]      └──[IFM-EV-DC]──► CCS2 60kW DC Charger         │   │
│  │                                                                          │   │
│  │  SW Config: STATION_MODE, 60kW_LIMIT, OCPP_2.0, BILLING_ENABLED         │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  IPCU-STATION (250kW)                          Cost: Rs. 35-45L         │   │
│  │  ────────────────────                                                    │   │
│  │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐       │   │
│  │  │ PMU │ │ PMU │ │ PMU │ │ PMU │ │ PMU │ │ PMU │ │ PMU │ │ PMU │       │   │
│  │  │ 25kW│ │ 25kW│ │ 25kW│ │ 25kW│ │ 25kW│ │ 25kW│ │ 25kW│ │ 25kW│       │   │
│  │  └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘       │   │
│  │     └───────┴───────┴───────┼───────┴───────┴───────┴───────┘           │   │
│  │                             │                                            │   │
│  │                      [DC BUS 800V]                                       │   │
│  │                             │                                            │   │
│  │  ┌──────────────────────────┼──────────────────────────┐                │   │
│  │  │                          │                          │                │   │
│  │  ▼                          ▼                          ▼                │   │
│  │  [IFM-GRID]              [IFM-EV-DC]×2             [IFM-BESS]           │   │
│  │  250kW Grid              2× 120kW CCS2             500 kWh BESS         │   │
│  │                                                                          │   │
│  │  +[IFM-SOLAR]×4 (100kWp)  +[IFM-EV-AC]×4 (22kW Type-2)                  │   │
│  │                                                                          │   │
│  │  SW Config: STATION_MODE, 250kW_LIMIT, OCPP_2.0, V2G_ENABLED,           │   │
│  │             GRID_SERVICES, DEMAND_RESPONSE, FLEET_MGMT                   │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 1.7.3 Software Configuration Parameters

The **same hardware** behaves differently based on software configuration:

| Parameter | UPM-HOME | IPCU-STATION | Description |
|-----------|----------|--------------|-------------|
| `PRODUCT_MODE` | `HOME` | `STATION` | Primary operating mode |
| `POWER_LIMIT_KW` | 5-15 | 60-250 | Maximum power throughput |
| `DC_BUS_VOLTAGE` | 48/400 | 800 | Internal DC bus voltage |
| `GRID_EXPORT_ENABLE` | `TRUE` | `TRUE` | Net metering/feed-in |
| `V2H_ENABLE` | `TRUE` | `FALSE` | Vehicle-to-Home mode |
| `V2G_ENABLE` | `FALSE` | `TRUE` | Vehicle-to-Grid (commercial) |
| `OCPP_ENABLE` | `FALSE` | `TRUE` | Charging network protocol |
| `BILLING_ENABLE` | `FALSE` | `TRUE` | Payment processing |
| `DEPARTURE_SCHEDULE` | `TRUE` | `FALSE` | EV ready-by-time feature |
| `DEMAND_RESPONSE` | `FALSE` | `TRUE` | Utility DR participation |
| `SOLAR_PRIORITY` | `SELF_CONSUME` | `PEAK_SHAVE` | Solar usage strategy |
| `BACKUP_MODE` | `SEAMLESS` | `MANUAL` | Power outage behavior |
| `UI_MODE` | `SIMPLE` | `COMMERCIAL` | User interface complexity |

### 1.7.4 Scaling Math: Module Count by Product

| Product | PMU Count | Power | Interface Modules | DC Bus | Est. Cost |
|---------|-----------|-------|-------------------|--------|-----------|
| **UPM-HOME-5** | 1 (20%) | 5 kW | GRID + SOLAR + V2H | 48/400V | Rs. 1.5L |
| **UPM-HOME-10** | 1 (40%) | 10 kW | GRID + SOLAR + V2H + AUX | 400V | Rs. 2.5L |
| **UPM-HOME-15** | 1 (60%) | 15 kW | GRID + SOLAR×2 + V2H + AUX | 400V | Rs. 3.5L |
| **UPM-BIZ-30** | 2 | 30 kW | GRID + SOLAR×2 + V2H×2 + AUX | 400V | Rs. 6L |
| **IPCU-HOME** | 1 | 25 kW | GRID + SOLAR + BESS(50kWh) | 400V | Rs. 10L |
| **IPCU-MINI** | 3 | 60 kW | GRID + SOLAR×2 + BESS + EV-DC | 800V | Rs. 18L |
| **IPCU-STANDARD** | 5 | 120 kW | GRID + SOLAR×4 + BESS + EV-DC×2 | 800V | Rs. 28L |
| **IPCU-STATION** | 8-10 | 250 kW | GRID + SOLAR×4 + BESS + EV-DC×2 + EV-AC×4 | 800V | Rs. 40L |

### 1.7.5 Common Control Module (CCM) Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    COMMON CONTROL MODULE (CCM)                                  │
│                    Same hardware for all products                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  APPLICATION LAYER (Linux Gateway - Jetson Nano / RPi CM4)              │   │
│  │  ─────────────────────────────────────────────────────────────────────  │   │
│  │  • Product Configuration Manager (reads SW config from EEPROM/cloud)    │   │
│  │  • Energy Management System (EMS) - mode-dependent algorithms           │   │
│  │  • User Interface (Web/App) - HOME_UI or STATION_UI based on config     │   │
│  │  • OCPP Stack (enabled/disabled by config)                              │   │
│  │  • Cloud Connectivity (MQTT/REST)                                       │   │
│  │  • OTA Firmware Updates                                                 │   │
│  │  • Data Logging & Analytics                                             │   │
│  └────────────────────────────────────┬────────────────────────────────────┘   │
│                                       │ Ethernet/SPI                           │
│                                       ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  REAL-TIME LAYER (TI C2000 F28388D or STM32H7)                          │   │
│  │  ─────────────────────────────────────────────────────────────────────  │   │
│  │  • Power Flow Controller (receives setpoints from EMS)                  │   │
│  │  • PMU Coordinator (CAN master to all PMU slaves)                       │   │
│  │  • Protection Manager (hardware + software protection)                  │   │
│  │  • Grid Synchronization (PLL, anti-islanding)                           │   │
│  │  • Metering (V, I, P, E for billing)                                    │   │
│  └────────────────────────────────────┬────────────────────────────────────┘   │
│                                       │ CAN-FD Bus                              │
│                                       ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  PMU SLAVE CONTROLLERS (TMS320F280 per PMU)                             │   │
│  │  ─────────────────────────────────────────────────────────────────────  │   │
│  │  • PWM Generation (50-100 kHz)                                          │   │
│  │  • Local Current/Voltage Control Loops                                  │   │
│  │  • Gate Driver Interface                                                │   │
│  │  • Local Protection (OCP, OVP, OTP)                                     │   │
│  │  • CAN Slave (receives power/voltage setpoints)                         │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  CONFIGURATION FLOW:                                                            │
│  ───────────────────                                                            │
│  1. Factory programs PRODUCT_ID to EEPROM (e.g., "UPM-HOME-10")                │
│  2. CCM reads PRODUCT_ID at boot → loads corresponding SW config               │
│  3. EMS algorithm, UI skin, enabled features all derive from config            │
│  4. Customer can upgrade (e.g., HOME-5 → HOME-10) via license key + OTA        │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 1.7.6 Benefits of Unified Platform Approach

| Benefit | Description | Impact |
|---------|-------------|--------|
| **Volume Pricing** | Single PMU design at high volume | 20-30% component cost reduction |
| **Shared R&D** | One control platform, one firmware base | 50% engineering cost savings |
| **Inventory Efficiency** | Common modules across products | Lower working capital |
| **Field Upgradability** | Add PMU or interface module to upgrade | Customer lifetime value |
| **Software Licensing** | Same HW, unlock features via license | Recurring revenue model |
| **Faster Time-to-Market** | New product = new config, not new design | Weeks vs months |
| **Quality & Reliability** | Higher volume = more field data | Better MTBF |
| **Service & Support** | Technicians learn one platform | Lower support costs |

### 1.7.7 Upgrade Paths

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          CUSTOMER UPGRADE PATHS                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  HOME CUSTOMER JOURNEY:                                                         │
│  ──────────────────────                                                         │
│                                                                                 │
│  Year 1: Buy UPM-HOME-5 (Rs. 1.5L)                                             │
│          └─► 5kW solar, EV charging, basic backup                               │
│                    │                                                            │
│                    ▼ (Add IFM-AUX module: Rs. 75k)                              │
│  Year 2: Upgrade to UPM-HOME-5 + 5kWh Aux                                       │
│          └─► Better backup when EV is away                                      │
│                    │                                                            │
│                    ▼ (SW license + add IFM-SOLAR: Rs. 50k)                      │
│  Year 3: Upgrade to UPM-HOME-10 equivalent                                      │
│          └─► 10kW throughput, larger solar                                      │
│                    │                                                            │
│                    ▼ (Add PMU + IFM-V2H: Rs. 3L)                                │
│  Year 5: Upgrade to UPM-BIZ-30 for home office                                  │
│          └─► 2 EVs, higher backup capacity                                      │
│                                                                                 │
│  ═══════════════════════════════════════════════════════════════════════════   │
│                                                                                 │
│  COMMERCIAL CUSTOMER JOURNEY:                                                   │
│  ────────────────────────────                                                   │
│                                                                                 │
│  Year 1: Buy IPCU-MINI (Rs. 18L)                                               │
│          └─► 60kW charger, 100kWh BESS, 20kWp solar                            │
│                    │                                                            │
│                    ▼ (Add 2× PMU + IFM-EV-DC: Rs. 8L)                           │
│  Year 2: Upgrade to IPCU-STANDARD                                               │
│          └─► 120kW, 2 DC charging ports                                         │
│                    │                                                            │
│                    ▼ (Add 3× PMU + IFM-BESS + IFM-EV-AC×4: Rs. 12L)            │
│  Year 4: Upgrade to IPCU-STATION                                                │
│          └─► 250kW, full hub with AC charging                                   │
│                    │                                                            │
│                    ▼ (SW license for V2G + Grid Services)                       │
│  Year 5: Enable V2G revenue stream                                              │
│          └─► Participate in grid ancillary services                             │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 1.7.8 Bill of Materials: Platform vs. Discrete

| Approach | UPM-HOME-10 | IPCU-STATION | Total Dev Cost |
|----------|-------------|--------------|----------------|
| **Discrete (2 products)** | Rs. 3.5L | Rs. 55L | Rs. 8 Cr |
| **Platform (common PMU)** | Rs. 2.5L | Rs. 40L | Rs. 5 Cr |
| **Savings** | **29%** | **27%** | **37%** |

---

# PART I: OPTION A - IPCU-STATION (Commercial)

> **Target Market:** Highway charging stations, Fleet depots, Commercial parking, Malls  
> **Power Range:** 60-250 kW  
> **Storage:** Fixed stationary BESS (100-500 kWh)  
> **Key Value:** High-power DC fast charging with integrated solar and grid services

---

## 2. Technical Architecture - IPCU-STATION

### 2.1 Power Flow Topology

```
                         IPCU BLOCK DIAGRAM
    ════════════════════════════════════════════════════════════

    INPUTS                    DC BUS                    OUTPUTS
    ──────                    ──────                    ───────

    ┌──────────┐         ┌──────────────┐         ┌──────────────┐
    │ GRID AC  │         │              │         │  DC FAST     │
    │ 3φ 415V  │──►[AFE]───►│            │──►[DC/DC]──►│  CCS2/CHAdeMO │
    │ 50Hz     │         │              │         │  60-180kW    │
    └──────────┘         │              │         └──────────────┘
                         │   COMMON     │
    ┌──────────┐         │   DC BUS     │         ┌──────────────────────┐
    │ SOLAR DC │         │              │         │  UNIFIED AC INVERTER │
    │ 300-600V │──►[MPPT]──►│   800V     │◄──►[BI-DIR]◄──►│  (75kW Bidirectional) │
    │ Array    │         │   NOMINAL    │         │                      │
    └──────────┘         │              │         │  ┌────────┬────────┐ │
                         │              │         │  │Type-2  │Grid/V2G│ │
    ┌──────────┐         │              │         │  │AC Out  │Export  │ │
    │ BATTERY  │         │              │         │  │(22kW)  │(75kW)  │ │
    │ LFP/NMC  │◄──►[DCDC]◄──►│            │         │  └────────┴────────┘ │
    │ 400-800V │         │              │         └──────────────────────┘
    └──────────┘         └──────────────┘

    ════════════════════════════════════════════════════════════
    
    Note: Single 75kW bidirectional inverter serves AC charging, grid-tie,
    and V2G/V2H functions via contactor switching. Saves Rs.1-3L vs separate.
```

### 2.2 DC Bus Voltage Selection

| Voltage Level | Pros | Cons | Use Case |
|---------------|------|------|----------|
| **400V** | Common, mature tech, cheaper components | Limited to 150kW charging | Small stations, retrofit |
| **800V** | Higher power (350kW+), lower currents, smaller cables | Newer tech, costlier | New builds, highway stations |
| **1000V** | Future-proof, highest efficiency | Very limited components | R&D, mega stations |

**Recommendation:** **800V DC Bus** - optimal balance for 2025-2030 infrastructure.

### 2.3 Configurable DC Bus Voltage Architecture

The IPCU supports **hardware and software configurable** intermediate DC bus voltage levels to enable product classification across different market segments.

#### 2.3.1 Product Classification Tiers

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    IPCU PRODUCT CLASSIFICATION MATRIX                            │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│   TIER 1: RESIDENTIAL (B2C)              TIER 2: FLEET (B2B)                    │
│   ┌─────────────────────────┐            ┌─────────────────────────┐            │
│   │  IPCU-HOME              │            │  IPCU-FLEET             │            │
│   │  DC Bus: 400V           │            │  DC Bus: 600V           │            │
│   │  Power: 10-50kW         │            │  Power: 50-150kW        │            │
│   │  Target: Homes, Villas  │            │  Target: Depots, Fleets │            │
│   └─────────────────────────┘            └─────────────────────────┘            │
│                                                                                  │
│   TIER 3: COMMERCIAL (B2B2C)             TIER 4: UTILITY (B2B)                  │
│   ┌─────────────────────────┐            ┌─────────────────────────┐            │
│   │  IPCU-STATION           │            │  IPCU-MEGA              │            │
│   │  DC Bus: 800V           │            │  DC Bus: 1000V          │            │
│   │  Power: 150-350kW       │            │  Power: 350-1000kW      │            │
│   │  Target: Public Charging│            │  Target: Highway Hubs   │            │
│   │  Revenue-as-a-Service   │            │  Grid Services          │            │
│   └─────────────────────────┘            └─────────────────────────┘            │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

#### 2.3.2 Hardware Configurability

| Configuration Element | 400V Class | 600V Class | 800V Class | 1000V Class |
|-----------------------|------------|------------|------------|-------------|
| **DC Bus Capacitors** | 450V rated | 700V rated | 900V rated | 1100V rated |
| **SiC MOSFET Modules** | 650V/200A | 1200V/200A | 1200V/300A | 1700V/300A |
| **Bus Bar Insulation** | Standard | Enhanced | Enhanced | High-voltage |
| **Creepage Distance** | 8mm | 12mm | 16mm | 20mm |
| **Fuse Rating** | 500V DC | 700V DC | 1000V DC | 1200V DC |
| **Contactor Rating** | 500V/200A | 700V/300A | 1000V/400A | 1200V/500A |

**Modular Hardware Approach:**
```
┌─────────────────────────────────────────────────────────────────────────┐
│                    CONFIGURABLE POWER STAGE                              │
│                                                                          │
│   ┌──────────────┐     ┌──────────────┐     ┌──────────────┐           │
│   │  CAPACITOR   │     │   SWITCH     │     │  PROTECTION  │           │
│   │   MODULE     │     │   MODULE     │     │   MODULE     │           │
│   │              │     │              │     │              │           │
│   │ ○ 400V Slot  │     │ ○ 650V SiC   │     │ ○ 500V Fuse  │           │
│   │ ○ 600V Slot  │     │ ○ 1200V SiC  │     │ ○ 700V Fuse  │           │
│   │ ○ 800V Slot  │     │ ○ 1700V SiC  │     │ ○ 1000V Fuse │           │
│   │ ○ 1000V Slot │     │              │     │ ○ 1200V Fuse │           │
│   └──────────────┘     └──────────────┘     └──────────────┘           │
│                                                                          │
│   Hardware Config ID stored in EEPROM → Auto-detected at boot           │
└─────────────────────────────────────────────────────────────────────────┘
```

#### 2.3.3 Software Configurability

| Parameter | Range | Default | Config Method | Access Level |
|-----------|-------|---------|---------------|---------------|
| **DC Bus Setpoint** | 350-1050V | HW-dependent | Register/OCPP | Installer |
| **Bus Voltage Tolerance** | ±2-10% | ±5% | Register | Factory |
| **Soft-Start Ramp Rate** | 10-100V/s | 50V/s | Register | Installer |
| **OV Protection Threshold** | 105-115% | 110% | Register | Factory |
| **UV Protection Threshold** | 75-90% | 80% | Register | Factory |
| **Power Limit per Tier** | 0-100% | 100% | OCPP/Cloud | Operator |
| **Operating Mode** | Grid/Island/Hybrid | Hybrid | OCPP | Operator |

**Software Configuration Interface:**
```python
# IPCU Configuration API Example
class IPCUConfig:
    def __init__(self, hw_tier: str):
        self.hw_tier = hw_tier  # AUTO-DETECTED from EEPROM
        self.voltage_limits = self._get_hw_limits()
    
    def set_dc_bus_voltage(self, target_v: int) -> bool:
        """Set DC bus voltage within HW-allowed range"""
        if self.voltage_limits['min'] <= target_v <= self.voltage_limits['max']:
            self._write_register(REG_DC_BUS_SETPOINT, target_v)
            return True
        return False  # Reject if outside HW capability
    
    def get_product_tier(self) -> dict:
        """Return product classification based on HW config"""
        return {
            '400V': {'tier': 'HOME', 'market': 'B2C', 'max_power': 50},
            '600V': {'tier': 'FLEET', 'market': 'B2B', 'max_power': 150},
            '800V': {'tier': 'STATION', 'market': 'B2B2C', 'max_power': 350},
            '1000V': {'tier': 'MEGA', 'market': 'B2B', 'max_power': 1000}
        }.get(self.hw_tier)
```

#### 2.3.4 Product Tier Specifications

| Specification | IPCU-HOME (B2C) | IPCU-FLEET (B2B) | IPCU-STATION (B2B2C) | IPCU-MEGA (B2B) |
|---------------|-----------------|------------------|----------------------|-----------------|
| **DC Bus Voltage** | 400V nominal | 600V nominal | 800V nominal | 1000V nominal |
| **Max Power** | 50 kW | 150 kW | 350 kW | 1000 kW |
| **Solar Input** | 10-30 kW | 30-100 kW | 100-200 kW | 200-500 kW |
| **Battery Capacity** | 20-50 kWh | 100-300 kWh | 300-1000 kWh | 1-5 MWh |
| **DC Charging** | 1×25kW | 2×60kW | 4×90kW | 8×120kW |
| **AC Charging** | 1×7kW | 4×22kW | 8×22kW | 16×22kW |
| **V2G/V2H** | V2H only | V2G basic | V2G full | Grid services |
| **Cooling** | Air-cooled | Liquid-cooled | Liquid-cooled | Liquid + chiller |
| **Enclosure** | Wall-mount | Floor cabinet | Outdoor kiosk | Container |
| **Target Price** | Rs. 8-12 L | Rs. 20-30 L | Rs. 35-50 L | Rs. 80-150 L |

#### 2.3.5 Use Case Mapping

| Market Segment | Product Tier | Primary Use Case | Revenue Model |
|----------------|--------------|------------------|---------------|
| **Residential (B2C)** | IPCU-HOME | Home solar + EV + backup | Self-consumption savings |
| **Housing Society** | IPCU-HOME × N | Shared EV charging | Per-kWh billing to residents |
| **Fleet Depot (B2B)** | IPCU-FLEET | Bus/truck overnight charging | TCO reduction, fleet uptime |
| **Corporate Campus** | IPCU-FLEET | Employee EV charging | Employee benefit, ESG |
| **Fuel Station (B2B2C)** | IPCU-STATION | Public fast charging | Per-session revenue |
| **Mall/Retail (B2B2C)** | IPCU-STATION | Customer charging | Footfall + charging revenue |
| **Highway Hub (B2B)** | IPCU-MEGA | Ultra-fast charging | High-margin corridor revenue |
| **Grid Operator (B2B)** | IPCU-MEGA | Ancillary services | Frequency regulation, capacity |

#### 2.3.6 Configuration Workflow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    IPCU CONFIGURATION WORKFLOW                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   FACTORY                    INSTALLATION                 OPERATION     │
│   ───────                    ────────────                 ─────────     │
│                                                                          │
│   ┌──────────────┐          ┌──────────────┐          ┌──────────────┐ │
│   │ 1. HW Config │          │ 3. Site      │          │ 5. Runtime   │ │
│   │    Selection │          │    Commissioning│        │    Tuning    │ │
│   │              │          │              │          │              │ │
│   │ • Tier select│          │ • Grid params│          │ • Load balance│ │
│   │ • Voltage    │          │ • Solar size │          │ • TOU optimize│ │
│   │ • Power rating│         │ • Battery SOC│          │ • V2G events │ │
│   └──────┬───────┘          └──────┬───────┘          └──────┬───────┘ │
│          │                         │                         │          │
│          ▼                         ▼                         ▼          │
│   ┌──────────────┐          ┌──────────────┐          ┌──────────────┐ │
│   │ 2. EEPROM    │          │ 4. Local     │          │ 6. Cloud/OCPP│ │
│   │    Programming│         │    Config    │          │    Updates   │ │
│   │              │          │              │          │              │ │
│   │ • HW ID      │          │ • Installer  │          │ • OTA firmware│ │
│   │ • Cal data   │          │    app/HMI   │          │ • Remote config│ │
│   │ • Limits     │          │ • Modbus/CAN │          │ • Analytics  │ │
│   └──────────────┘          └──────────────┘          └──────────────┘ │
│                                                                          │
│   [LOCKED]                  [INSTALLER KEY]           [OPERATOR ACCESS] │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.4 Module Breakdown

#### 2.4.1 Input Stage Modules

| Module | Input | Output | Power | Efficiency | Est. Cost |
|--------|-------|--------|-------|------------|-----------|
| **Grid Rectifier (AFE)** | 3φ 415V AC | 800V DC | 250kW | 97.5% | Rs. 8-12 L |
| **Solar MPPT** | 300-600V DC | 800V DC | 100kW | 99% | Rs. 3-5 L |
| **Battery DC-DC** | 400-800V DC | 800V DC | 200kW | 98% | Rs. 6-10 L |

#### 2.4.2 Output Stage Modules

| Module | Input | Output | Power | Efficiency | Est. Cost |
|--------|-------|--------|-------|------------|-----------|
| **DC Fast Charger** | 800V DC | 200-920V DC | 60-180kW | 97% | Rs. 10-18 L |
| **Unified AC Inverter (Bidirectional)** | 800V DC | 3φ 415V AC | 75kW bi-dir | 95% | Rs. 6-9 L |

> **Design Note:** The Unified AC Inverter replaces separate AC charger and V2G modules. A single 75kW 4-quadrant inverter with contactor switching serves:
> - **AC Charging Output** (Type-2, up to 22kW to EV's onboard charger)
> - **Grid Tie / V2G Export** (up to 75kW bidirectional)
> - **Building Backup** (V2H mode)
> 
> **Savings:** Rs. 1-3 L vs. separate modules, simpler thermal management, reduced enclosure size.

---

## 3. Operating Modes

### 3.1 Mode Matrix

| Mode | Solar | Grid | Battery | EV Charging | V2G Export | Use Case |
|------|-------|------|---------|-------------|------------|----------|
| **Daytime Peak** | ✓ Max | Minimal | Charging | ✓ | - | Solar priority |
| **Evening Peak** | - | Minimal | Discharging | ✓ | - | Battery priority |
| **Night Off-Peak** | - | ✓ Max | Charging | ✓ | - | Grid arbitrage |
| **Grid Failure** | ✓ | - | ✓ | Limited | - | Islanded mode |
| **Peak Shaving** | ✓ | Limited | Discharging | ✓ | ✓ | Demand response |
| **V2G Event** | - | - | ✓ | - | ✓ Max | Grid support |

### 3.2 Power Flow Priority Logic

```
┌─────────────────────────────────────────────────────────────────┐
│                    ENERGY MANAGEMENT SYSTEM                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   PRIORITY 1: Use Solar First                                   │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ IF solar_power > 0:                                      │   │
│   │     → Route to EV charging (primary)                     │   │
│   │     → Excess to battery (secondary)                      │   │
│   │     → Surplus to grid export (tertiary)                  │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   PRIORITY 2: Battery Arbitrage                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ IF grid_rate < threshold AND battery_soc < 80%:         │   │
│   │     → Charge battery from grid                           │   │
│   │ IF grid_rate > threshold AND battery_soc > 20%:         │   │
│   │     → Discharge battery for EV charging                  │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   PRIORITY 3: V2G Revenue                                       │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ IF grid_demand_response_event AND parked_ev_soc > 50%:  │   │
│   │     → Export EV power to grid (with owner consent)       │   │
│   │     → Credit EV owner account                            │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Hardware Design Considerations

### 4.1 Power Electronics Stack

```
IPCU INTERNAL LAYOUT (Top View) - OPTIMIZED DESIGN
═══════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────┐
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   GRID      │  │   SOLAR     │  │   BATTERY   │         │
│  │   AFE       │  │   MPPT      │  │   DC-DC     │         │
│  │   250kW     │  │   100kW     │  │   200kW     │         │
│  │             │  │             │  │             │         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘         │
│         │                │                │                 │
│         └────────────────┼────────────────┘                 │
│                          │                                   │
│  ┌───────────────────────┼───────────────────────────────┐  │
│  │                DC BUS BAR (800V)                       │  │
│  │              [Bus Capacitors Bank]                     │  │
│  └───────────────────────┼───────────────────────────────┘  │
│                          │                                   │
│         ┌────────────────┴────────────────┐                 │
│         │                                 │                 │
│  ┌──────┴───────────┐  ┌──────────────────┴───────────┐    │
│  │   DC FAST        │  │   UNIFIED AC INVERTER        │    │
│  │   OUTPUT         │  │   (75kW Bidirectional)       │    │
│  │   60-180kW       │  │                              │    │
│  │                  │  │   ┌────────────────────────┐ │    │
│  │   [CCS2]         │  │   │ [Type-2]    [Grid/V2G] │ │    │
│  │   [CHAdeMO]      │  │   │  AC Out      Export    │ │    │
│  └──────────────────┘  │   │  (22kW)      (75kW)    │ │    │
│                        │   └────────────────────────┘ │    │
│                        └──────────────────────────────┘    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              CONTROL & COMMUNICATION                 │    │
│  │  [Main Controller] [HMI] [OCPP Gateway] [Metering]  │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              THERMAL MANAGEMENT                      │    │
│  │  [Liquid Cooling Loop] [Heat Exchanger] [Pumps]     │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────┘

Dimensions: 1800mm (L) × 1100mm (W) × 2100mm (H) [10% smaller!]
Weight: ~1350 kg (fully loaded) [10% lighter!]
```

### 4.2 Key Components Specification

| Component | Specification | Qty | Est. Unit Cost | Notes |
|-----------|---------------|-----|----------------|-------|
| **SiC MOSFET Module** | 1200V/300A, 3-level NPC | 12 | Rs. 45,000 | Main switching devices |
| **DC Bus Capacitor** | 450V/6800µF Film | 8 | Rs. 8,000 | 2 series × 4 parallel |
| **Gate Driver** | Isolated, 2W output | 12 | Rs. 3,500 | Matched to SiC |
| **Current Sensor** | Hall-effect, 500A | 8 | Rs. 2,500 | Closed-loop |
| **DSP Controller** | TI TMS320F28379D | 2 | Rs. 4,000 | Main + backup |
| **Cooling Pump** | 24V, 30L/min | 2 | Rs. 8,000 | Redundant |
| **Liquid-Air HEX** | 50kW capacity | 1 | Rs. 35,000 | Roof-mounted |
| **Contactor** | 800V DC, 400A | 6 | Rs. 12,000 | Input/Output isolation |
| **Fuse** | 800V DC, 400A gR | 6 | Rs. 5,000 | Semiconductor fuses |

### 4.3 Thermal Design

```
COOLING SYSTEM SCHEMATIC (OPTIMIZED - 4 MODULES)
═════════════════════════════════════════════════

     ┌──────────────────────────────────────────────────────┐
     │                   ROOF-MOUNTED                        │
     │                  HEAT EXCHANGER                       │
     │                   (Air-Cooled, 40kW)                  │
     └──────────────────────┬───────────────────────────────┘
                            │
              ┌─────────────┴─────────────┐
              │    COOLANT LOOP (50/50    │
              │    Glycol/Water, 12L)     │
              └─────────────┬─────────────┘
                            │
     ┌──────────────────────┼────────────────────────────┐
     │                      │                             │
┌────┴────┐    ┌────┴────┐    ┌────┴────┐    ┌──────┴──────┐
│  Grid   │    │ Solar   │    │ Battery │    │  DC Fast +  │
│  AFE    │    │ MPPT    │    │ DC-DC   │    │  Unified AC │
│  250kW  │    │ 100kW   │    │ 200kW   │    │  180+75kW   │
│ (Cold   │    │ (Cold   │    │ (Cold   │    │ (Cold Plate)│
│  Plate) │    │  Plate) │    │  Plate) │    │             │
└─────────┘    └─────────┘    └─────────┘    └─────────────┘

Design Point: 250kW continuous, 45°C ambient
Coolant Flow: 25 L/min (reduced from 30, fewer modules)
Max Coolant Temp: 65°C
Cold Plate ΔT: <10°C junction-coolant
Benefit: Simpler plumbing, smaller pump, lower parasitic load
```

---

## 5. Control System Architecture

### 5.1 Control Hierarchy

```
┌─────────────────────────────────────────────────────────────────┐
│                     LEVEL 3: CLOUD/SCADA                         │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ • Fleet Management     • Predictive Maintenance         │    │
│  │ • Energy Trading       • Remote Diagnostics             │    │
│  │ • Analytics Dashboard  • OTA Updates                    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              │ MQTT/HTTPS                        │
├──────────────────────────────┼──────────────────────────────────┤
│                     LEVEL 2: SITE CONTROLLER                     │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ • OCPP 2.0.1 Gateway   • Energy Management System       │    │
│  │ • User Authentication  • Payment Processing             │    │
│  │ • Load Balancing       • Grid Interface                 │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              │ Modbus TCP / CANopen             │
├──────────────────────────────┼──────────────────────────────────┤
│                     LEVEL 1: IPCU CONTROLLER                     │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ • Power Flow Control   • DC Bus Regulation              │    │
│  │ • MPPT Algorithm       • Battery SOC Management         │    │
│  │ • Fault Detection      • Protection Coordination        │    │
│  │ • Thermal Management   • Efficiency Optimization        │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              │ PWM / Gate Signals               │
├──────────────────────────────┼──────────────────────────────────┤
│                     LEVEL 0: POWER ELECTRONICS                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ • SiC MOSFETs          • Gate Drivers                   │    │
│  │ • Sensors              • Contactors                     │    │
│  │ • Protection Devices   • Cooling System                 │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Control Loop Timing

| Control Loop | Update Rate | Latency | Processor |
|--------------|-------------|---------|-----------|
| PWM Generation | 20 kHz | <1 µs | DSP (hardware) |
| Current Control | 10 kHz | <50 µs | DSP |
| Voltage Control | 2 kHz | <500 µs | DSP |
| MPPT | 100 Hz | <10 ms | DSP |
| Thermal Control | 1 Hz | <1 s | MCU |
| Energy Management | 0.1 Hz | <10 s | Linux SBC |
| OCPP Communication | Event-driven | <1 s | Linux SBC |

---

## 6. Safety & Compliance

### 6.1 Standards Compliance Matrix

| Standard | Description | Applicable Sections |
|----------|-------------|---------------------|
| **IEC 61851-1** | EV Charging - General | All charging outputs |
| **IEC 61851-23** | DC Charging | DC fast charger output |
| **IEC 62196-3** | CCS Combo 2 connector | Physical interface |
| **IS 17017** | India AC Charging | AC output, Bharat AC-001 |
| **IS 17017** | India DC Charging | DC output, Bharat DC-001 |
| **IEC 62109-1/2** | PV Inverter Safety | Solar MPPT module |
| **IEC 62619** | Battery Safety | Battery interface |
| **IEC 61000-6-2/4** | EMC Immunity/Emissions | Full system |
| **IP54** | Ingress Protection | Enclosure |
| **IK10** | Impact Protection | Enclosure |

### 6.2 Protection Features

| Protection | Trigger | Response Time | Action |
|------------|---------|---------------|--------|
| **DC Bus OV** | >900V | <10 µs | Shutdown all modules |
| **DC Bus UV** | <650V | <100 µs | Reduce power, alarm |
| **Overcurrent** | >120% rated | <5 µs | Gate block, contactor open |
| **Short Circuit** | di/dt > limit | <2 µs | Desaturation protection |
| **Ground Fault** | >30mA | <30 ms | Contactor open, alarm |
| **Over Temperature** | >85°C junction | <1 s | Derate, then shutdown |
| **Arc Fault** | Arc signature | <100 ms | Contactor open |
| **Grid Loss** | Voltage <80% | <2 s | Islanding protection |

---

## 7. Cost Targets & Business Case Analysis

> **Source:** Cost targets are derived from the India EV Charging Revenue Analysis (`ev_revenue_report.md`) which establishes station economics, payback requirements, and market pricing benchmarks.

### 7.0 Cost Target Derivation from Revenue Model

#### 7.0.1 The Business Case: Station Economics Summary

Based on our revenue analysis of India's EV charging market (3.84M EVs, 4,494 stations as of 2026):

| Metric | Standard Station (2 DC + 2 AC) | Value |
|--------|--------------------------------|-------|
| **Daily kWh Delivered** | 1,862 kWh | - |
| **Daily Sessions** | 82 sessions | - |
| **Monthly Revenue** | Rs. 9.7-10.3 Lakhs | Average Rs. 10 L |
| **Monthly Grid Cost** | Rs. 2.6-4.3 Lakhs | 30-42% of revenue |
| **Monthly Gross Profit** | Rs. 5.9-6.8 Lakhs | 57-70% margin |
| **Problem** | Power costs eat 30-45% of revenue | Key pain point |

#### 7.0.2 The Value Proposition: Hybrid Systems Payback

| System Configuration | Investment | Monthly Savings | Payback | Final Margin |
|---------------------|------------|-----------------|---------|--------------|
| Grid Only | Rs. 0 | - | - | 57-58% |
| Solar 40% (177 kWp) | Rs. 67 L | Rs. 1.73 L | **3.2 years** | 73-74% |
| Solar + 250 kWh Battery | Rs. 102 L | Rs. 2.90 L | **2.9 years** | 80% |
| Solar + 500 kWh Battery | Rs. 137 L | Rs. 2.90 L | **3.9 years** | **83-84%** |

**Key Insight:** Operators are willing to invest Rs. 100-140 Lakhs if payback is under 4 years. This sets our **maximum system cost ceiling**.

#### 7.0.3 IPCU-STATION: Target Cost Derivation

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│              COST TARGET DERIVATION: IPCU-STATION                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│   MARKET PRICING (BUYING SEPARATELY):                                           │
│   ────────────────────────────────────                                          │
│   │ Component                    │ Market Price │                               │
│   ├──────────────────────────────┼──────────────┤                               │
│   │ 500 kWh LFP Battery          │ Rs. 65 L     │                               │
│   │ 200 kVA Bi-directional Inv   │ Rs. 50 L     │                               │
│   │ 120 kW DC Fast Charger       │ Rs. 25 L     │                               │
│   │ BMS + Controller             │ Rs. 6 L      │                               │
│   │ Integration & Wiring         │ Rs. 8 L      │                               │
│   ├──────────────────────────────┼──────────────┤                               │
│   │ TOTAL (DISCRETE)             │ Rs. 154 L    │                               │
│   └──────────────────────────────┴──────────────┘                               │
│                                                                                 │
│   ▼ PAYBACK CONSTRAINT (< 4 years) ▼                                            │
│                                                                                 │
│   ┌──────────────────────────────────────────────────────────────────┐          │
│   │ Monthly Savings with 67% offset: Rs. 2.90 Lakhs                  │          │
│   │ Target Payback: 3.5 years (42 months)                            │          │
│   │ Maximum Allowable Investment: Rs. 2.9L × 42 = Rs. 122 Lakhs      │          │
│   │                                                                  │          │
│   │ SYSTEM COST = IPCU + Solar + Installation                        │          │
│   │ Rs. 122 L = IPCU + Rs. 67 L (Solar 177 kWp) + Rs. 15 L (Install) │          │
│   │ IPCU TARGET = Rs. 122 L - 67 L - 15 L = Rs. 40 Lakhs (Max)       │          │
│   │                                                                  │          │
│   │ BUT: Market disruption requires 27% savings vs discrete          │          │
│   │ IPCU TARGET PRICE: Rs. 100 L × 0.73 = Rs. 73 Lakhs (excl solar)  │          │
│   │ IPCU BOM TARGET: Rs. 73 L × 0.65 (35% margin) = Rs. 47 Lakhs     │          │
│   └──────────────────────────────────────────────────────────────────┘          │
│                                                                                 │
│   ▼ FINAL TARGETS ▼                                                             │
│                                                                                 │
│   ┌───────────────────────────────────────────────────────────────────┐         │
│   │              IPCU-STATION COST TARGETS (500 kWh)                  │         │
│   ├───────────────────────────────────────────────────────────────────┤         │
│   │  METRIC              │  TARGET         │  JUSTIFICATION           │         │
│   ├──────────────────────┼─────────────────┼──────────────────────────┤         │
│   │  BOM Cost            │  Rs. 30-47 L    │  35-50% gross margin     │         │
│   │  Selling Price       │  Rs. 73-100 L   │  27% below discrete      │         │
│   │  Customer Payback    │  < 4 years      │  Investment hurdle rate  │         │
│   │  ROI to Customer     │  > 25% annual   │  Attractive investment   │         │
│   │  Our Gross Margin    │  35-50%         │  Sustainable business    │         │
│   └──────────────────────┴─────────────────┴──────────────────────────┘         │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

#### 7.0.4 IPCU-STATION: Product Tier Cost Targets

| Configuration | Battery | DC Charger | Inverter | BOM Target | Sell Price | Market Price | Savings |
|---------------|---------|------------|----------|------------|------------|--------------|---------|
| **IPCU-MINI** | 100 kWh | 60 kW | 50 kVA | Rs. 18 L | **Rs. 28 L** | Rs. 38 L | 26% |
| **IPCU-STD** | 250 kWh | 60 kW | 100 kVA | Rs. 36 L | **Rs. 55 L** | Rs. 75 L | 27% |
| **IPCU-HUB** | 500 kWh | 120 kW | 200 kVA | Rs. 30.5 L | **Rs. 100 L** | Rs. 137 L | 27% |
| **IPCU-FLEET** | 1000 kWh | 240 kW | 400 kVA | Rs. 117 L | **Rs. 180 L** | Rs. 250 L | 28% |

> **Note:** BOM target for IPCU-HUB matches our detailed Section 7.1 breakdown of Rs. 30.5 Lakhs.

#### 7.0.5 UPM-HOME: Target Cost Derivation

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│              COST TARGET DERIVATION: UPM-HOME (Residential)                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│   COMPETITOR LANDSCAPE:                                                         │
│   ─────────────────────                                                         │
│   │ Product                      │ Price         │ Capability               │   │
│   ├──────────────────────────────┼───────────────┼──────────────────────────┤   │
│   │ 5 kVA Solar Inverter         │ Rs. 50,000    │ Solar only               │   │
│   │ 5 kVA Hybrid Inverter        │ Rs. 80,000    │ Solar + Battery          │   │
│   │ 7 kW AC EV Charger           │ Rs. 60,000    │ Charging only            │   │
│   │ 10 kVA Servo Stabilizer      │ Rs. 25,000    │ Voltage only             │   │
│   │ 5 kVA Home UPS + Battery     │ Rs. 1,20,000  │ Backup only              │   │
│   ├──────────────────────────────┼───────────────┼──────────────────────────┤   │
│   │ TOTAL (Separate)             │ Rs. 3,35,000  │ 5 devices                │   │
│   └──────────────────────────────┴───────────────┴──────────────────────────┘   │
│                                                                                 │
│   UPM-HOME VALUE PROPOSITION:                                                   │
│   ──────────────────────────────                                                │
│   Replaces 5 devices: Solar Inverter + Hybrid Inverter + EV Charger +          │
│                       Stabilizer + UPS                                          │
│                                                                                 │
│   ┌───────────────────────────────────────────────────────────────────┐         │
│   │              UPM-HOME COST TARGETS                                │         │
│   ├───────────────────────────────────────────────────────────────────┤         │
│   │  TIER           │  BOM TARGET  │  SELL PRICE  │  REPLACES        │         │
│   ├─────────────────┼──────────────┼──────────────┼──────────────────┤         │
│   │  UPM-HOME-5     │  Rs. 65,000  │  Rs. 1.2 L   │  Rs. 2.0 L equiv │         │
│   │  UPM-HOME-10    │  Rs. 95,000  │  Rs. 1.8 L   │  Rs. 2.8 L equiv │         │
│   │  UPM-HOME-15    │  Rs. 1.4 L   │  Rs. 2.5 L   │  Rs. 3.5 L equiv │         │
│   │  UPM-HOME-25    │  Rs. 2.0 L   │  Rs. 3.5 L   │  Rs. 5.0 L equiv │         │
│   └─────────────────┴──────────────┴──────────────┴──────────────────┘         │
│                                                                                 │
│   MARGIN STRUCTURE:                                                             │
│   │ BOM Cost       │  40-50% of Sell Price                                │     │
│   │ Gross Margin   │  45-55%                                              │     │
│   │ Customer Value │  30-40% cheaper than buying 5 separate devices       │     │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

#### 7.0.6 Cost Target Summary Table

| Product | Target Market | Power | Storage | BOM Target | Sell Price | Customer Payback |
|---------|---------------|-------|---------|------------|------------|------------------|
| **UPM-HOME-5** | Budget Home | 5 kW | V2H only | Rs. 0.65 L | Rs. 1.2 L | 2-3 years |
| **UPM-HOME-10** | Mid Home | 10 kW | V2H + 5kWh | Rs. 0.95 L | Rs. 1.8 L | 2-3 years |
| **UPM-HOME-15** | Large Home | 15 kW | V2H + 10kWh | Rs. 1.4 L | Rs. 2.5 L | 2.5-3 years |
| **UPM-HOME-25** | Villa/Sm Biz | 25 kW | V2H + 20kWh | Rs. 2.0 L | Rs. 3.5 L | 3-4 years |
| **IPCU-MINI** | Highway Stop | 60 kW | 100 kWh | Rs. 18 L | Rs. 28 L | 3-3.5 years |
| **IPCU-STD** | Urban Station | 100 kW | 250 kWh | Rs. 36 L | Rs. 55 L | 3-3.5 years |
| **IPCU-HUB** | Large Hub | 200 kW | 500 kWh | Rs. 30.5 L | Rs. 100 L | 3.5-4 years |
| **IPCU-FLEET** | Fleet Depot | 400 kW | 1000 kWh | Rs. 117 L | Rs. 180 L | 3.5-4 years |

#### 7.0.7 PMU (Power Module Unit) Cost Target

The 25 kW PMU is our core building block. Target cost must enable all product tiers:

| PMU Cost Element | Target Cost | Notes |
|------------------|-------------|-------|
| SiC Full-Bridge (25 kW) | Rs. 1,50,000 | Wolfspeed/Infineon SiC modules |
| Gate Drivers + Sensors | Rs. 25,000 | Isolated drivers, current sensors |
| EMI Filter + Magnetics | Rs. 35,000 | CM choke, filter inductors |
| DC Bus Capacitors | Rs. 20,000 | Film caps, 900V rated |
| Local DSP Controller | Rs. 15,000 | TI C2000 or equivalent |
| Heatsink + Coldplate | Rs. 25,000 | Liquid-cooled interface |
| Housing + Connectors | Rs. 15,000 | IP65 rated, quick-connect |
| Assembly + Testing | Rs. 15,000 | Labor, calibration |
| **PMU BOM TOTAL** | **Rs. 3,00,000** | Per 25 kW module |

**Scaling Math:**
- UPM-HOME-5: 1 PMU @ 20% = Rs. 60,000 power electronics
- UPM-HOME-25: 1 PMU @ 100% = Rs. 3,00,000 power electronics  
- IPCU-HUB: 8 PMUs = Rs. 24,00,000 power electronics (80% of our Rs. 30.5L BOM)

#### 7.0.8 Cost Reduction Roadmap

| Phase | Volume | PMU Cost | IPCU-HUB BOM | IPCU-HUB Price |
|-------|--------|----------|--------------|----------------|
| **Phase 1 (2026)** | 100-500/yr | Rs. 3.0 L | Rs. 30.5 L | Rs. 100 L |
| **Phase 2 (2027)** | 500-2000/yr | Rs. 2.5 L | Rs. 26 L | Rs. 85 L |
| **Phase 3 (2028)** | 2000-5000/yr | Rs. 2.0 L | Rs. 22 L | Rs. 72 L |
| **Phase 4 (2030)** | 5000+/yr | Rs. 1.5 L | Rs. 18 L | Rs. 60 L |

> **Target:** 40% cost reduction over 4 years through volume, vertical integration, and component optimization.

---

### 7.1 Bill of Materials (Target) - Optimized Design

| Category | Components | Target Cost | % of Total |
|----------|------------|-------------|------------|
| **Power Electronics** | SiC modules, gate drivers, capacitors | Rs. 10,50,000 | 34% |
| **Magnetics** | Inductors, transformers | Rs. 3,50,000 | 11% |
| **Control System** | DSP, MCU, sensors, HMI | Rs. 2,80,000 | 9% |
| **Thermal System** | Coldplates, HEX, pumps, coolant (simplified) | Rs. 2,20,000 | 7% |
| **Enclosure** | Cabinet, busbars, wiring (10% smaller) | Rs. 3,20,000 | 10% |
| **Connectors & Cables** | CCS2, Type-2, DC cables | Rs. 1,80,000 | 6% |
| **Protection Devices** | Fuses, contactors, SPD | Rs. 1,80,000 | 6% |
| **Assembly & Testing** | Labor, calibration, burn-in | Rs. 2,70,000 | 9% |
| **Certification** | Type testing, approvals | Rs. 2,00,000 | 6% |
| **TOTAL BOM** | | **Rs. 30,50,000** | 100% |

> **Optimization Note:** Merging AC Charger + V2G into a single 75kW bidirectional inverter saves ~Rs. 1-3L in components, reduces thermal complexity, and shrinks enclosure by 10%.

### 7.2 Cost Comparison: Integrated vs. Discrete

| Component | Discrete Approach | IPCU Approach | Savings |
|-----------|-------------------|---------------|---------|
| Solar Inverter (100kW) | Rs. 10,00,000 | Included | - |
| Battery Inverter (200kW) | Rs. 15,00,000 | Included | - |
| DC Charger (120kW) | Rs. 25,00,000 | Included | - |
| AC Charger (22kW) + V2G (50kW) | Rs. 7,00,000 | Rs. 6,00,000 (unified 75kW) | Rs. 1,00,000 |
| Integration & Wiring | Rs. 5,00,000 | Included | - |
| **TOTAL** | **Rs. 62,00,000** | **Rs. 30,50,000** | **Rs. 31,50,000 (51%)** |

> **Key Savings from Unified Design:**
> - Merged AC + V2G inverter: Rs. 1-3 Lakhs saved
> - Simplified thermal system (4 modules vs 5): Rs. 30k saved
> - Smaller enclosure (10% reduction): Rs. 30k saved
> - Reduced wiring complexity: Rs. 20k saved

### 7.3 Efficiency Gains

| Scenario | Discrete Efficiency | IPCU Efficiency | Energy Saved/Year |
|----------|---------------------|-----------------|-------------------|
| Solar → EV | 97% × 97% = 94.1% | 99% × 97% = 96.0% | +2,000 kWh |
| Solar → Battery → EV | 97% × 98% × 98% × 97% = 90.3% | 99% × 98% × 97% = 94.1% | +4,500 kWh |
| Grid → Battery → EV | 97% × 98% × 98% × 97% = 90.3% | 97.5% × 98% × 97% = 92.7% | +2,800 kWh |

**Annual Efficiency Benefit:** ~₹50,000-80,000 per station at ₹6/kWh

### 7.4 IPCU-HOME: Voltage Stabilizer Context (India)

Indian residential supply faces significant voltage fluctuations (170-270V typical range vs 230V nominal). The IPCU-HOME tier integrates stabilization functionality, replacing standalone stabilizers.

#### 7.4.1 Indian Voltage Stabilizer Market Overview

| Type | Technology | Input Range | Rating | Typical Price | Efficiency |
|------|------------|-------------|--------|---------------|------------|
| **Relay-based** | Tap-changing transformer | 170-270V | 1-10 kVA | Rs. 2,000-15,000 | 95-97% |
| **Servo-controlled** | Motorized autotransformer | 150-280V | 3-25 kVA | Rs. 8,000-50,000 | 96-98% |
| **Static (SCR)** | Thyristor tap-changer | 170-270V | 5-50 kVA | Rs. 15,000-1,00,000 | 97-98% |
| **Online UPS** | Double-conversion | 100-300V | 1-10 kVA | Rs. 25,000-2,00,000 | 85-92% |
| **IPCU-HOME** | Active rectifier + inverter | 170-280V | 10-50 kVA | Rs. 8-12 L | 94-96% |

**Major Indian Stabilizer Brands:** V-Guard, Microtek, Luminous, Livguard, Servokon, Consul Neowatt

#### 7.4.2 House Supply Rating: Component Breakdown (5 kVA Servo Stabilizer)

| Component | Specification | Qty | Unit Cost (Rs.) | Total (Rs.) | % of BOM |
|-----------|---------------|-----|-----------------|-------------|----------|
| **Autotransformer (Toroidal)** | 5 kVA, copper wound, 0-270V | 1 | 2,500 | 2,500 | 28% |
| **Servo Motor** | 24V DC, 10 RPM, geared | 1 | 800 | 800 | 9% |
| **Carbon Brush Assembly** | Graphite, spring-loaded | 2 | 150 | 300 | 3% |
| **Control PCB** | MCU-based, voltage sensing | 1 | 600 | 600 | 7% |
| **Voltage Sensing Circuit** | Resistive divider + ADC | 1 | 100 | 100 | 1% |
| **Relay (Bypass/Cutoff)** | 30A, 250V AC | 2 | 120 | 240 | 3% |
| **Contactor (Input)** | 25A, 240V AC | 1 | 350 | 350 | 4% |
| **MOV (Surge Protection)** | 275V, 20kA | 3 | 25 | 75 | 1% |
| **Capacitors (Filter)** | 450V, 10µF film | 2 | 40 | 80 | 1% |
| **LED Display** | 3-digit, voltage readout | 1 | 150 | 150 | 2% |
| **Enclosure (Metal)** | Powder-coated, ventilated | 1 | 800 | 800 | 9% |
| **Wiring & Terminals** | 4 sq mm, lugs, connectors | 1 set | 300 | 300 | 3% |
| **Cooling Fan** | 80mm, 230V AC | 1 | 120 | 120 | 1% |
| **PCB Assembly & Testing** | Labor + QC | 1 | 400 | 400 | 4% |
| **Packaging** | Thermocol, carton | 1 | 150 | 150 | 2% |
| **TOTAL BOM** | | | | **Rs. 6,965** | 78% |
| **Overhead (22%)** | Margin, warranty, logistics | | | Rs. 1,965 | 22% |
| **MRP** | | | | **Rs. 8,930** | 100% |

#### 7.4.3 House Supply Rating: Component Breakdown (10 kVA Static/SCR Stabilizer)

| Component | Specification | Qty | Unit Cost (Rs.) | Total (Rs.) | % of BOM |
|-----------|---------------|-----|-----------------|-------------|----------|
| **Autotransformer (Toroidal)** | 10 kVA, copper wound, tapped | 1 | 5,500 | 5,500 | 22% |
| **SCR/Thyristor Module** | 50A, 800V, with heatsink | 4 | 450 | 1,800 | 7% |
| **TRIAC (Tap Selection)** | 40A, 600V | 6 | 180 | 1,080 | 4% |
| **Gate Driver IC** | Optocoupler-isolated | 6 | 35 | 210 | 1% |
| **DSP/MCU Controller** | STM32 or equivalent | 1 | 250 | 250 | 1% |
| **Voltage/Current Sensors** | Hall-effect or CT | 2 | 200 | 400 | 2% |
| **Filter Inductor** | 5mH, 50A, iron core | 1 | 1,200 | 1,200 | 5% |
| **DC Bus Capacitor** | 450V, 2200µF electrolytic | 4 | 180 | 720 | 3% |
| **EMI Filter** | Common-mode choke + caps | 1 | 600 | 600 | 2% |
| **Relay (Bypass)** | 50A, 250V AC | 2 | 250 | 500 | 2% |
| **Contactor (Main)** | 40A, 240V AC, 2-pole | 1 | 650 | 650 | 3% |
| **MOV Array** | 275V, 40kA surge | 6 | 40 | 240 | 1% |
| **SMPS (Control Power)** | 12V/24V, 2A | 1 | 350 | 350 | 1% |
| **LCD Display** | 16x2 or segment | 1 | 200 | 200 | 1% |
| **Enclosure (Metal)** | Floor-standing, IP21 | 1 | 2,500 | 2,500 | 10% |
| **Wiring & Busbars** | 10 sq mm, copper bus | 1 set | 800 | 800 | 3% |
| **Cooling (Fan + Heatsink)** | Forced air, Al heatsink | 1 set | 600 | 600 | 2% |
| **PCB Assembly & Testing** | Labor + burn-in | 1 | 1,200 | 1,200 | 5% |
| **Certification (BIS)** | Amortized per unit | 1 | 500 | 500 | 2% |
| **Packaging** | Heavy-duty carton | 1 | 300 | 300 | 1% |
| **TOTAL BOM** | | | | **Rs. 19,600** | 78% |
| **Overhead (22%)** | Margin, warranty, logistics | | | Rs. 5,530 | 22% |
| **MRP** | | | | **Rs. 25,130** | 100% |

#### 7.4.4 IPCU-HOME vs Standalone Stabilizer Comparison

| Aspect | 5 kVA Servo Stabilizer | 10 kVA Static Stabilizer | IPCU-HOME (10 kVA) |
|--------|------------------------|--------------------------|---------------------|
| **Cost** | Rs. 8,000-12,000 | Rs. 20,000-35,000 | Rs. 8-12 L (integrated) |
| **Voltage Regulation** | ±1% (slow) | ±1% (fast) | ±0.5% (instantaneous) |
| **Response Time** | 500-2000 ms | 10-50 ms | <1 ms |
| **Waveform** | Unchanged | Minor distortion | Pure sine |
| **Efficiency** | 96-98% | 97-98% | 94-96% |
| **Backup Power** | No | No | Yes (with battery) |
| **Solar Integration** | No | No | Yes (MPPT) |
| **EV Charging** | No | No | Yes (7-25 kW) |
| **Smart Features** | Basic | LCD/remote | IoT, app, OCPP |
| **Lifespan** | 8-10 years | 10-12 years | 15-20 years |

**Key Insight:** IPCU-HOME eliminates the need for separate stabilizer, inverter/UPS, solar inverter, and EV charger—consolidating 4 devices into 1.

> **📌 Note:** IPCU-HOME (fixed battery) is the foundation for the VIES/UPM architecture. Part II extends this concept with **modular, EV-based storage** that eliminates the need to purchase a stationary battery.

---

# PART II: OPTION B - VIES/UPM-HOME (Residential)

> **Target Market:** Homes, Apartments, Small businesses, Rental properties  
> **Power Range:** 5-30 kW  
> **Storage:** EV battery via V2H (50-100 kWh) + Optional aux battery (5-20 kWh)  
> **Key Value:** Zero battery cost if you own an EV, modular scalability, home backup

---

## 8. Vehicle-Integrated Energy Storage (VIES) Architecture

### 8.1 The Vision: Vehicle-Integrated Energy Storage

The core insight: **Why buy a large stationary battery when your EV already has 50-100 kWh sitting in the driveway?**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    MODULAR ENERGY STORAGE CONCEPT                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────┐      ┌─────────────────────────────────┐                  │
│   │    SOLAR    │      │     UNIVERSAL POWER MODULE      │                  │
│   │   PANELS    │─────▶│          (UPM)                  │                  │
│   │  3-10 kWp   │      │                                 │                  │
│   └─────────────┘      │   ┌─────────┐   ┌─────────┐    │   ┌──────────┐   │
│                        │   │ DC Bus  │   │  Smart  │    │   │  HOME    │   │
│   ┌─────────────┐      │   │ 48-400V │◀─▶│ Energy  │────┼──▶│  LOADS   │   │
│   │    GRID     │◀────▶│   │         │   │ Manager │    │   │  5-20kW  │   │
│   │  1φ/3φ AC   │      │   └────┬────┘   └─────────┘    │   └──────────┘   │
│   └─────────────┘      │        │                       │                  │
│                        │        ▼                       │                  │
│                        │   ┌─────────────────────┐      │                  │
│                        │   │  STORAGE INTERFACE  │      │                  │
│                        │   │    (Modular Slot)   │      │                  │
│                        │   └──────────┬──────────┘      │                  │
│                        └──────────────┼─────────────────┘                  │
│                                       │                                    │
│              ┌────────────────────────┼────────────────────────┐           │
│              │                        │                        │           │
│              ▼                        ▼                        ▼           │
│   ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐    │
│   │   OPTION A:      │    │   OPTION B:      │    │   OPTION C:      │    │
│   │   EV Battery     │    │   Aux Battery    │    │   EV + Aux       │    │
│   │   (V2H/V2G)      │    │   (Stationary)   │    │   (Hybrid)       │    │
│   │                  │    │                  │    │                  │    │
│   │  ┌──────────┐    │    │  ┌──────────┐    │    │  ┌────┐ ┌────┐   │    │
│   │  │    EV    │    │    │  │   LFP    │    │    │  │ EV │ │Aux │   │    │
│   │  │  50-100  │    │    │  │  5-20    │    │    │  │    │ │    │   │    │
│   │  │   kWh    │    │    │  │   kWh    │    │    │  │    │ │    │   │    │
│   │  └──────────┘    │    │  └──────────┘    │    │  └────┘ └────┘   │    │
│   │                  │    │                  │    │                  │    │
│   │  Cost: Rs.0      │    │  Cost: Rs.1.5-3L │    │  Cost: Rs.1.5-3L │    │
│   │  (already own)   │    │  (5-10 kWh)      │    │  + EV already    │    │
│   └──────────────────┘    └──────────────────┘    └──────────────────┘    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 8.2 Use Case Scenarios

| Scenario | Storage Source | Capacity | Power | Best For |
|----------|---------------|----------|-------|----------|
| **EV Owner (Primary)** | EV only | 50-100 kWh | 7-11 kW | Most homes, backup, solar storage |
| **No EV Yet** | Aux battery only | 5-15 kWh | 3-5 kW | Solar self-consumption, UPS |
| **EV + Backup** | EV + Aux | 60-120 kWh | 10-15 kW | Extended outages, high loads |
| **Rental/Shared** | Visitor EV | Variable | 7-11 kW | Apartments, shared housing |
| **Small Business** | Multiple EVs + Aux | 100-300 kWh | 20-50 kW | Peak shaving, demand response |

### 8.3 Universal Power Module (UPM) - Home Version

#### 8.3.1 Product Variants

| Model | Power | Storage Interface | Target User | Est. Price |
|-------|-------|-------------------|-------------|------------|
| **UPM-Home-5** | 5 kW | 1× EV OR 1× Aux (5kWh) | Apartment/Small home | Rs. 1.5 L |
| **UPM-Home-10** | 10 kW | 1× EV + 1× Aux (10kWh) | Medium home | Rs. 2.5 L |
| **UPM-Home-15** | 15 kW | 2× EV OR 1× EV + Aux | Large home/Villa | Rs. 3.5 L |
| **UPM-Biz-30** | 30 kW | 3× EV + Aux (50kWh) | Small business | Rs. 6.0 L |

#### 8.3.2 UPM-Home-10 Architecture (Recommended)

```
┌────────────────────────────────────────────────────────────────────────────┐
│                         UPM-HOME-10 (10kW Bidirectional)                   │
│                         Dimensions: 600 × 400 × 800 mm                     │
│                         Weight: ~80 kg (without aux battery)               │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                            │
│  ┌──────────────────┐                           ┌──────────────────────┐   │
│  │   GRID INPUT     │                           │    HOME OUTPUT       │   │
│  │   1φ 230V 50A    │◀─────────────────────────▶│    1φ 230V 45A       │   │
│  │   or 3φ 415V     │         AC BUS            │    + Critical Load   │   │
│  └────────┬─────────┘                           └──────────┬───────────┘   │
│           │                                                │               │
│           ▼                                                ▼               │
│  ┌────────────────────────────────────────────────────────────────────┐   │
│  │                    BIDIRECTIONAL INVERTER                          │   │
│  │                    10 kW / 4-Quadrant                              │   │
│  │                    Efficiency: 96%                                  │   │
│  └────────────────────────────────────┬───────────────────────────────┘   │
│                                       │                                    │
│                              ┌────────┴────────┐                          │
│                              │    DC BUS       │                          │
│                              │    48V / 400V   │                          │
│                              │    (Selectable) │                          │
│                              └───┬─────────┬───┘                          │
│                                  │         │                               │
│           ┌──────────────────────┘         └────────────────────── ┐       │
│           │                                                        │       │
│           ▼                                                        ▼       │
│  ┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐   │
│  │  SOLAR MPPT     │      │  EV INTERFACE   │      │  AUX BATTERY    │   │
│  │  2× 5kW inputs  │      │  (Bidirectional)│      │  INTERFACE      │   │
│  │                 │      │                 │      │                 │   │
│  │  MPPT Range:    │      │  CCS2 Combo     │      │  48V/400V       │   │
│  │  150-500V DC    │      │  7-11 kW Bidir  │      │  Modular Slot   │   │
│  │                 │      │  V2H Protocol   │      │                 │   │
│  └─────────────────┘      └────────┬────────┘      └────────┬────────┘   │
│                                    │                        │             │
│                                    ▼                        ▼             │
│                           ┌───────────────┐        ┌───────────────┐     │
│                           │  TO: EV       │        │  OPTIONAL:    │     │
│                           │  (Parked)     │        │  LFP Module   │     │
│                           │               │        │  5-20 kWh     │     │
│                           │  ┌─────────┐  │        │  ┌─────────┐  │     │
│                           │  │   🚗    │  │        │  │ ░░░░░░░ │  │     │
│                           │  │ 50-100  │  │        │  │  5-10   │  │     │
│                           │  │  kWh    │  │        │  │  kWh    │  │     │
│                           │  └─────────┘  │        │  └─────────┘  │     │
│                           └───────────────┘        └───────────────┘     │
│                                                                            │
├────────────────────────────────────────────────────────────────────────────┤
│  CONTROL: ESP32 + Energy Meter │ DISPLAY: 4.3" Touch │ COMMS: WiFi/4G/CAN │
└────────────────────────────────────────────────────────────────────────────┘
```

### 8.4 Smart Energy Management Logic

#### 8.4.1 Decision Tree: Which Battery to Use?

```
                        ┌──────────────────┐
                        │  POWER NEEDED?   │
                        └────────┬─────────┘
                                 │
                    ┌────────────┴────────────┐
                    │                         │
              ┌─────▼─────┐             ┌─────▼─────┐
              │  CHARGE   │             │ DISCHARGE │
              │  (Solar/  │             │  (Home/   │
              │   Grid)   │             │   Grid)   │
              └─────┬─────┘             └─────┬─────┘
                    │                         │
         ┌──────────┴──────────┐    ┌────────┴─────────┐
         │                     │    │                  │
    ┌────▼────┐          ┌────▼────▼───┐         ┌────▼────┐
    │ EV      │          │ EV Plugged? │         │ EV      │
    │ Plugged?│          └──────┬──────┘         │ Plugged?│
    └────┬────┘                 │                └────┬────┘
         │                      │                     │
    ┌────┴────┐           ┌─────┴─────┐         ┌────┴────┐
    │YES  │NO │           │ YES │ NO  │         │YES  │NO │
    └──┬──┴─┬─┘           └──┬──┴──┬──┘         └──┬──┴─┬─┘
       │    │                │     │                │    │
       ▼    ▼                ▼     ▼                ▼    ▼
  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
  │Charge  │ │Charge  │ │Use EV  │ │Use Aux │ │Use EV  │ │Use Aux │
  │EV First│ │Aux Batt│ │(if SOC │ │Battery │ │First   │ │Only    │
  │(to 80%)│ │        │ │> 30%) │ │        │ │(>20%)  │ │        │
  └────────┘ └────────┘ └────────┘ └────────┘ └────────┘ └────────┘
```

#### 8.4.2 Operating Modes

| Mode | Trigger | EV Battery | Aux Battery | Grid |
|------|---------|------------|-------------|------|
| **Solar Self-Consume** | Solar > Load | Charge (if <80%) | Charge (if <90%) | Export excess |
| **Peak Shaving** | ToD Peak hours | Discharge (if >30%) | Discharge (if >20%) | Minimize import |
| **Backup (Outage)** | Grid fails | Primary source | Secondary source | Disconnected |
| **EV Priority Charge** | EV SOC <20% | Charge from grid | Hold | Import |
| **Grid Services** | Utility signal | Discharge to grid | Discharge to grid | Export (V2G) |
| **Night Storage** | Off-peak hours | N/A (EV likely away) | Charge from grid | Import |

#### 8.4.3 EV Departure Prediction

```
┌─────────────────────────────────────────────────────────────────┐
│           SMART CHARGING WITH DEPARTURE PREDICTION              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  User sets: "I leave at 7:30 AM, need 80% SOC"                  │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    NIGHT SCHEDULE                        │   │
│  │                                                          │   │
│  │  10PM    12AM    2AM     4AM     6AM     7:30AM         │   │
│  │   │       │       │       │       │        │            │   │
│  │   ▼       ▼       ▼       ▼       ▼        ▼            │   │
│  │  ┌───────────────────────────────┐ ┌─────┐              │   │
│  │  │  V2H DISCHARGE (if needed)   │ │CHARGE│  🚗 DEPART   │   │
│  │  │  Use EV for home loads       │ │ EV   │              │   │
│  │  │  (Grid outage backup)        │ │to 80%│              │   │
│  │  └───────────────────────────────┘ └─────┘              │   │
│  │                                                          │   │
│  │  Off-peak rate: Rs.4/kWh  │  Peak rate: Rs.8/kWh        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Logic:                                                         │
│  1. Calculate energy needed: (80% - current%) × battery kWh    │
│  2. Calculate charging time: energy / charge_rate              │
│  3. Start charging at: departure_time - charging_time - buffer │
│  4. Until then: EV available for V2H discharge                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 8.5 Aux Battery Module Options

#### 8.5.1 Plug-and-Play Battery Modules

| Module | Chemistry | Capacity | Voltage | Dimensions | Weight | Price |
|--------|-----------|----------|---------|------------|--------|-------|
| **AUX-5** | LFP | 5 kWh | 48V | 500×400×200mm | 45 kg | Rs. 75,000 |
| **AUX-10** | LFP | 10 kWh | 48V | 500×400×350mm | 85 kg | Rs. 1,40,000 |
| **AUX-10HV** | LFP | 10 kWh | 400V | 600×400×300mm | 80 kg | Rs. 1,50,000 |
| **AUX-20HV** | LFP | 20 kWh | 400V | 600×400×500mm | 150 kg | Rs. 2,80,000 |

> **Modularity:** Stack up to 4 modules for 20-80 kWh total capacity.

#### 8.5.2 Why Aux Battery Even with EV?

| Scenario | Problem with EV-Only | Aux Battery Solution |
|----------|---------------------|---------------------|
| EV not home | No backup during day | Aux provides 4-8 hrs backup |
| Long outage | Drains EV, can't drive | Aux depletes first, EV preserved |
| Solar midday | EV at work, solar wasted | Aux stores solar for evening |
| EV degradation concern | Frequent V2H cycles | Aux handles daily cycling |
| Multiple EVs | Only 1 V2H-capable | Aux + 1 EV = sufficient |

### 8.6 Home System Sizing Guide

#### 8.6.1 Quick Sizing Calculator

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    HOME ENERGY STORAGE SIZING                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  STEP 1: Determine Daily Energy Need                                    │
│  ────────────────────────────────────                                   │
│  Average daily consumption: _______ kWh (check electricity bill)        │
│  Typical Indian home: 8-15 kWh/day                                      │
│                                                                         │
│  STEP 2: Determine Backup Duration Needed                               │
│  ──────────────────────────────────────                                 │
│  Hours of backup needed: _______ hours                                  │
│  Critical loads only (lights, fans, fridge, WiFi): 1-2 kW              │
│  Full home (+ AC): 3-5 kW                                              │
│                                                                         │
│  STEP 3: Calculate Storage Capacity                                     │
│  ─────────────────────────────────                                      │
│  Backup Energy = Critical Load (kW) × Backup Hours                      │
│  Example: 2 kW × 8 hours = 16 kWh                                       │
│                                                                         │
│  STEP 4: Choose Configuration                                           │
│  ──────────────────────────────                                         │
│                                                                         │
│  ┌─────────────────┬─────────────────┬─────────────────┐               │
│  │    EV OWNER     │   NO EV (YET)   │   EV + PEACE    │               │
│  │                 │                 │   OF MIND       │               │
│  ├─────────────────┼─────────────────┼─────────────────┤               │
│  │  EV: 50-100 kWh │  Aux: 10 kWh    │  EV: 50+ kWh    │               │
│  │  Aux: 0         │  (Rs. 1.4 L)    │  Aux: 5 kWh     │               │
│  │                 │                 │  (Rs. 75k)      │               │
│  │  Total: 50+ kWh │  Total: 10 kWh  │  Total: 55+ kWh │               │
│  │  Cost: Rs.0*    │  Cost: Rs.1.4L  │  Cost: Rs.75k   │               │
│  │  Backup: 25+ hr │  Backup: 5 hrs  │  Backup: 27+ hr │               │
│  └─────────────────┴─────────────────┴─────────────────┘               │
│  * EV already purchased                                                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

#### 8.6.2 Recommended Configurations by Home Type

| Home Type | Daily Use | Solar | Storage | UPM Model | Total Cost |
|-----------|-----------|-------|---------|-----------|------------|
| **1 BHK Apartment** | 6-8 kWh | 2 kWp | EV only | UPM-Home-5 | Rs. 2.3 L |
| **2 BHK Apartment** | 10-12 kWh | 3 kWp | EV + 5kWh Aux | UPM-Home-10 | Rs. 4.0 L |
| **3 BHK Flat** | 12-15 kWh | 5 kWp | EV + 10kWh Aux | UPM-Home-10 | Rs. 5.0 L |
| **Independent House** | 15-20 kWh | 7 kWp | EV + 10kWh Aux | UPM-Home-15 | Rs. 6.5 L |
| **Villa with Pool** | 25-35 kWh | 10 kWp | EV + 20kWh Aux | UPM-Home-15 | Rs. 8.5 L |

### 8.7 EV Compatibility for V2H

#### 8.7.1 V2H-Capable EVs in India (2025-26)

| Vehicle | Battery | V2H Power | Protocol | Availability |
|---------|---------|-----------|----------|--------------|
| **Hyundai Ioniq 5** | 72.6 kWh | 3.6 kW (V2L) | Proprietary | Available |
| **Kia EV6** | 77.4 kWh | 3.6 kW (V2L) | Proprietary | Available |
| **BYD Atto 3** | 60.5 kWh | 3.3 kW (V2L) | Proprietary | Available |
| **MG ZS EV** | 50.3 kWh | - | Planned | 2025 |
| **Tata Curvv EV** | 55 kWh | - | Planned | 2025 |
| **Mahindra XUV.e8** | 80 kWh | 7-11 kW (V2H) | CCS2 | 2025-26 |

> **Note:** True bidirectional V2H (via CCS2) requires vehicle OEM support. V2L (outlet on car) is limited to 3.6 kW.

#### 8.7.2 Protocol Support in UPM

| Protocol | Power | Vehicles | UPM Support |
|----------|-------|----------|-------------|
| **V2L (AC Outlet)** | 1.5-3.6 kW | Ioniq 5, EV6, Atto 3 | Direct plug |
| **CHAdeMO V2H** | 6-10 kW | Nissan Leaf (not in India) | Optional module |
| **CCS2 Bidirectional** | 7-11 kW | Future EVs (2025+) | Native support |
| **ISO 15118-20** | Up to 22 kW | Next-gen EVs | Firmware update |

### 8.8 Cost-Benefit Analysis: Home System

#### 8.8.1 Investment vs. Savings

| Configuration | Investment | Monthly Savings | Payback | 10-Year Benefit |
|---------------|------------|-----------------|---------|-----------------|
| **Solar Only (5kWp)** | Rs. 2.25 L | Rs. 3,500 | 5.4 yrs | Rs. 1.95 L |
| **Solar + EV V2H** | Rs. 4.0 L | Rs. 5,500 | 6.1 yrs | Rs. 2.60 L |
| **Solar + Aux 10kWh** | Rs. 4.9 L | Rs. 5,000 | 8.2 yrs | Rs. 1.10 L |
| **Solar + EV + Aux 5kWh** | Rs. 4.75 L | Rs. 6,000 | 6.6 yrs | Rs. 2.45 L |

> **Best Value:** Solar + EV V2H (if you own a V2H-capable EV)  
> **Best Reliability:** Solar + EV + Small Aux (covers all scenarios)

#### 8.8.2 Value of Backup Power

| Outage Duration | Inverter + Battery Cost | Generator Cost | UPM + EV Value |
|-----------------|------------------------|----------------|----------------|
| 2 hours/day | Rs. 30,000 (1kVA) | N/A | Included |
| 4 hours/day | Rs. 60,000 (2kVA) | Rs. 50,000 + fuel | Included |
| 8+ hours/day | Rs. 1,20,000 (5kVA) | Rs. 1,00,000 + fuel | Included |

**Hidden Value:** No fuel, no noise, no maintenance, no fumes!

### 8.9 Commercial Application: Fleet Charging Hub

For commercial applications (fleet depots, malls, offices), the same modular concept scales up:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                  COMMERCIAL FLEET HUB - MODULAR STORAGE                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌──────────────────────────────────────────────────────────────────┐     │
│   │                    IPCU-STATION (250 kW)                         │     │
│   │                    + Modular Battery Slots                       │     │
│   └────────────────────────────┬─────────────────────────────────────┘     │
│                                │                                            │
│              ┌─────────────────┼─────────────────┐                          │
│              │                 │                 │                          │
│              ▼                 ▼                 ▼                          │
│   ┌────────────────┐  ┌────────────────┐  ┌────────────────┐               │
│   │  SLOT 1: V2G   │  │  SLOT 2: V2G   │  │  SLOT 3: AUX   │               │
│   │  Fleet EV #1   │  │  Fleet EV #2   │  │  100 kWh LFP   │               │
│   │  (60 kWh)      │  │  (60 kWh)      │  │  (Buffer)      │               │
│   └────────────────┘  └────────────────┘  └────────────────┘               │
│                                                                             │
│   Total Storage Pool: 60 + 60 + 100 = 220 kWh                              │
│   Peak Shaving: Discharge during 6-10 PM peak                               │
│   V2G Revenue: Sell to grid during high prices                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 8.10 Key Advantages of Modular VIES Architecture

| Advantage | Description | Benefit |
|-----------|-------------|---------|
| **Zero Battery Cost (EV Owners)** | Use existing EV battery | Save Rs. 1.5-5L on stationary storage |
| **Scalable** | Add aux modules as needed | Pay-as-you-grow |
| **Redundant** | Multiple storage sources | Higher reliability |
| **Future-Proof** | Supports upcoming V2H EVs | No obsolescence |
| **Grid Services Ready** | Aggregated V2G from EVs | Potential revenue stream |
| **Lower Lifecycle Cost** | EV battery replaced by OEM anyway | No separate battery maintenance |

---

---

# PART III: COMMON ELEMENTS & ROADMAP

> **Applies to:** Both IPCU-STATION and VIES/UPM products  
> **Shared:** DC bus topology, SiC power electronics, control architecture, safety standards

---

## 9. Development Roadmap

### 9.1 Phase Plan

```
2026                          2027                          2028
═══════════════════════════════════════════════════════════════════════

Q1    Q2    Q3    Q4    Q1    Q2    Q3    Q4    Q1    Q2
────────────────────────────────────────────────────────────────────

PHASE 1: DESIGN & SIMULATION
├─────────────────────┤
│ • Topology finalization
│ • Component selection  
│ • Thermal simulation
│ • Control algorithm dev
│ • PCB design
└─────────────────────

            PHASE 2: PROTOTYPE
            ├──────────────────────┤
            │ • 50kW lab prototype
            │ • Module testing
            │ • Integration testing
            │ • Safety testing
            │ • Control tuning
            └──────────────────────

                          PHASE 3: PILOT
                          ├─────────────────────┤
                          │ • 3 field units
                          │ • Real-world testing
                          │ • Customer feedback
                          │ • Design refinement
                          │ • Certification
                          └─────────────────────

                                        PHASE 4: PRODUCTION
                                        ├─────────────────────►
                                        │ • Manufacturing setup
                                        │ • Supply chain
                                        │ • Volume production
                                        │ • After-sales support
                                        └─────────────────────►
```

### 9.2 Key Milestones

| Milestone | Target Date | Deliverable | Go/No-Go Criteria |
|-----------|-------------|-------------|-------------------|
| **M1: Design Freeze** | Q2 2026 | Complete design package | Simulation meets spec |
| **M2: Lab Prototype** | Q4 2026 | Working 50kW unit | 95% efficiency achieved |
| **M3: Full Power Test** | Q2 2027 | 250kW validated | All modes functional |
| **M4: Type Test Pass** | Q3 2027 | Certification | IEC/IS compliance |
| **M5: Field Pilot** | Q4 2027 | 3 units deployed | 6-month MTBF >2000 hrs |
| **M6: Production Start** | Q2 2028 | First 10 units | Cost target met |

---

## 10. Risk Assessment

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| **SiC supply shortage** | Medium | High | Dual-source, buffer stock |
| **Thermal design failure** | Low | High | Early prototyping, margin |
| **EMC non-compliance** | Medium | Medium | Expert review, pre-compliance |
| **Certification delays** | Medium | Medium | Early engagement with labs |
| **Cost overrun** | Medium | Medium | Value engineering, DFM |
| **Grid code changes** | Low | High | Modular firmware, flexibility |

---

## 11. Next Steps

### Immediate Actions (Next 30 Days)

- [ ] **Finalize DC bus voltage** (800V recommended)
- [ ] **Select SiC module vendor** (Wolfspeed, Infineon, or ROHM)
- [ ] **Complete thermal simulation** (ANSYS Icepak)
- [ ] **Define control architecture** (TI vs. Infineon DSP)
- [ ] **Engage certification body** (TÜV, NABL lab)
- [ ] **Draft preliminary BOM** with 3 vendor quotes each
- [ ] **IP strategy** - file provisional patents

### Team Requirements

| Role | FTE | Skills Required |
|------|-----|-----------------|
| Power Electronics Engineer | 2 | SiC, high-voltage design |
| Embedded Systems Engineer | 1 | DSP, real-time control |
| Mechanical Engineer | 1 | Thermal, enclosure design |
| Test Engineer | 1 | Power testing, certification |
| Project Manager | 0.5 | EV infrastructure experience |

---

## 12. Appendix

### A. Reference Designs

- Texas Instruments: TIDA-010210 (Bidirectional EV Charger)
- Infineon: REF_DAB_500W_SIC (DAB Converter)
- Wolfspeed: CRD-60DD12N (60kW SiC Charger)

### B. Vendor Contacts

| Component | Vendor | Contact |
|-----------|--------|---------|
| SiC Modules | Wolfspeed India | ev-sales@wolfspeed.com |
| SiC Modules | Infineon India | power-india@infineon.com |
| Gate Drivers | Analog Devices | india_sales@analog.com |
| Capacitors | TDK India | capacitors@tdk.com |
| Cooling | Aavid/Boyd | thermal@boydcorp.com |

### C. Glossary

| Term | Definition |
|------|------------|
| **AFE** | Active Front End - grid-side converter with power factor correction |
| **MPPT** | Maximum Power Point Tracking - solar optimization |
| **BESS** | Battery Energy Storage System |
| **V2G** | Vehicle-to-Grid - bidirectional power flow |
| **V2H** | Vehicle-to-Home - using EV battery to power home loads |
| **V2L** | Vehicle-to-Load - AC outlet on EV for portable appliances |
| **VIES** | Vehicle-Integrated Energy Storage - modular storage using EV batteries |
| **UPM** | Universal Power Module - integrated home energy management unit |
| **SiC** | Silicon Carbide - wide bandgap semiconductor |
| **DAB** | Dual Active Bridge - isolated DC-DC topology |
| **NPC** | Neutral Point Clamped - 3-level inverter topology |

---

**Document Version:** 1.3  
**Last Updated:** 2026-03-01  
**Review Cycle:** Monthly during development  
**Change Log:**  
- v1.3 - Added Section 1.7: Unified Platform Architecture (common PMU building blocks, SW-driven config)  
- v1.2 - Reorganized into Part I (IPCU-STATION) and Part II (VIES/UPM-HOME) as two distinct architecture options  
- v1.1 - Added Section 8 (VIES Modular Energy Storage Architecture), unified AC+V2G inverter design  
- v1.0 - Initial IPCU-STATION architecture
