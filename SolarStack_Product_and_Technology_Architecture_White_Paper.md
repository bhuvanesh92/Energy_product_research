# SolarStack India
## Product and Technology Architecture White Paper
### From Home Energy Intelligence to Solar, Storage, and Charge-Point Energy Orchestration

**Version:** 0.1  
**Date:** 3 October 2026  
**Status:** Product architecture discussion draft  
**Audience:** Founders, product leaders, power-electronics engineers, embedded/cloud architects, EPC partners, and prospective investors

---

## Executive summary

SolarStack should be designed as a **distributed-energy operating system** rather than as a sequence of unrelated HEMS, battery, and EV-charging products.

Its end-state is a Charge Point Operator (CPO) site energy-management platform that coordinates grid connection, transformer capacity, solar PV, stationary battery energy storage systems (BESS), building/site loads, and multiple EV charging stations. The same data model, security model, forecasting services, optimiser, installer workflow, and fleet operations platform can begin at the household level, where the first product measures energy use, identifies viable solar and storage opportunities, and creates a qualified installation pipeline.

The essential architectural principle is:

Cloud / Control Plane + Edge Products by Power Class + Open Asset Adapters

SolarStack should **reuse software and operational capabilities**, not force a single physical controller or power-electronics architecture across 3 kW homes and 500 kW charging sites.

The proposed product family is:

1. **SolarStack Insight** — vendor-neutral monitoring, energy assessment, solar/BESS sizing, and installer workflow for homes and small sites.
2. **SolarStack Home Control** — supervisory control for solar, BESS, backup loads, and optional home EV charging.
3. **SolarStack Site Control** — industrial energy management for multi-asset building, depot, and CPO sites.
4. **SolarStack Fleet OS** — cloud fleet operations, optimisation, service, and future grid/market integration.

The core strategic asset is not a battery or a charger. It is the **vendor-neutral control plane and operational dataset** that can measure, predict, optimise, and safely coordinate assets behind a meter.

---

## 1. Strategic scope and design principles

### 1.1 Product mission

SolarStack helps asset owners turn distributed electrical assets into measurable economic and reliability outcomes.

For a home, the question is:

> How much can this household save or improve resilience by adding solar, BESS, or smart charging — and what exact configuration is justified?

For a CPO site, the question is:

> How can this site complete more charging sessions and improve uptime while respecting grid limits, transformer capacity, tariffs, battery health, and solar availability?

### 1.2 Product boundaries

SolarStack should own the following layers:

- Asset data model and digital twin.
- Edge telemetry, control, configuration, and diagnostics.
- Vendor adapters for meters, PV inverters, BMS/PCS, EVSEs, and building loads.
- Forecasting and optimisation logic.
- Customer/operator UX, installer tools, service workflows, and fleet analytics.
- Cybersecurity, identity, OTA lifecycle management, and auditability.

SolarStack should initially **buy, integrate, or partner for** the following:

- PV modules.
- Certified string inverters, hybrid inverters, and power-conversion systems.
- LFP cells and battery racks.
- Certified EVSE hardware.
- Protection relays, switchgear, meters, contactors, and high-voltage assemblies.
- Payment rails and customer acquisition where partners are better placed.

### 1.3 Non-negotiable principles

1. **Vendor-neutral at the asset boundary.** Support existing solar and charging assets through standards and OEM adapters.
2. **Local autonomy.** Cloud loss must not cause unsafe, uncontrolled, or commercially unacceptable site behaviour.
3. **Safety functions stay in certified hardware.** BMS, PCS, EVSE, relays, and protection systems retain hard safety authority.
4. **Optimisation is bounded.** SolarStack sends validated setpoints inside declared asset capabilities; it does not bypass protection logic.
5. **Security by default.** Device identity, mutually authenticated communications, signed OTA, secure boot, least privilege, and immutable audit history are baseline requirements.
6. **Measure before selling.** Recommendations must be tied to observed load, production, tariff, outage, and site constraints whenever possible.
7. **Explain recommendations.** A consumer or site operator must understand why a proposed battery, backup reserve, demand cap, or charger schedule is recommended.
8. **Productise installation.** Site survey, commissioning, quality assurance, and service workflows are part of the product, not field-operations afterthoughts.

---

## 2. Common reference architecture

```text
                         ┌─────────────────────────────────────────────┐
                         │          SOLARSTACK FLEET OS / CLOUD        │
                         │                                             │
                         │ Identity • asset registry • digital twins   │
                         │ Time-series data • forecasting • optimiser  │
                         │ Tariff engine • OTA • service • APIs        │
                         │ NOC • analytics • reporting • permissions   │
                         └──────────────────────┬──────────────────────┘
                                                │
                       Secure telemetry, commands, OTA, and APIs
                                                │
         ┌──────────────────────┬───────────────┼──────────────────────┐
         │                      │               │                      │
         ▼                      ▼               ▼                      ▼
┌─────────────────┐   ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│ SolarStack      │   │ SolarStack      │ │ SolarStack      │ │ Partner /       │
│ Insight         │   │ Home Control    │ │ Site Control    │ │ utility systems │
│ Monitoring      │   │ Home EMS        │ │ CPO / building  │ │ DISCOM / CSMS   │
└───────┬─────────┘   └───────┬─────────┘ └───────┬─────────┘ └─────────────────┘
        │                     │                   │
        ▼                     ▼                   ▼
 Meter, CTs, PV          PV, BESS, EVSE,      Transformer, grid,
 inverter APIs,          critical loads,       PV, BESS, EVSEs,
 bills, weather          grid interface        building/site loads
```

### 2.1 Control-loop allocation

| Loop | Typical latency | Owner | Examples |
|---|---:|---|---|
| Protection and safety | microseconds to milliseconds | Certified PCS/BMS/EVSE/protection relay | Overcurrent, short circuit, insulation fault, overtemperature, anti-islanding |
| Local equipment control | milliseconds to seconds | Device firmware / real-time edge layer | Charger derating, BESS setpoint tracking, contactor sequencing |
| Site dispatch | seconds to minutes | SolarStack edge controller | Demand cap, battery dispatch, charger power allocation |
| Economic scheduling | 5 minutes to hours | Cloud and edge optimiser | Tariff response, PV forecast schedule, EV ready-by schedule |
| Portfolio optimisation | hours to days | Fleet OS | Capacity planning, maintenance prioritisation, fleet energy policy |

This separation is analogous to a safety-critical distributed embedded system: the cloud is a supervisory planner, while the site retains safe operation and deterministic fallback behaviour.

### 2.2 Common digital-twin model

```text
Organisation
 └── Portfolio
      └── Site
           ├── Grid connection / transformer / sanctioned demand
           ├── Revenue meter / smart meter / feeder meters
           ├── Solar PV system
           │    ├── inverter(s)
           │    └── strings / production meters
           ├── Stationary BESS
           │    ├── battery racks and BMS
           │    └── PCS / hybrid inverter
           ├── Loads
           │    ├── critical loads
           │    ├── controllable loads
           │    └── auxiliary/site loads
           ├── EVSEs
           │    └── connectors / charging sessions
           ├── Controllers, firmware, certificates, and network state
           ├── Tariff / commercial rules
           └── Alarms, work orders, warranty, and audit events
```

A home is a small site. A CPO is a large, high-availability site. The entities remain consistent; the number of assets, power class, operational constraints, and commercial workflows change.

---

## 3. Product 1 — SolarStack Insight

### 3.1 Purpose

SolarStack Insight is a vendor-neutral monitoring and recommendation product for:

- Homes without solar or BESS.
- Existing solar homes without BESS.
- Existing solar homes with poor visibility or suspected underperformance.
- Small commercial customers considering solar, storage, backup, or EV charging.
- EPCs and installers needing a repeatable assessment, proposal, and post-installation monitoring workflow.

Its primary role is **measurement-led qualification**. It should generate trusted recommendations and high-quality conversion opportunities for solar, storage, and later Home Control installations.

### 3.2 Customer outcomes

| Customer state | Insight outcome |
|---|---|
| No solar / no BESS | Solar readiness, recommended PV size, BESS viability, backup requirement, savings range |
| Solar / no BESS | Self-consumption, export profile, evening-load gap, BESS opportunity score, retrofit design |
| Solar underperforming | Expected versus actual production, fault/anomaly indicators, service recommendation |
| High-bill household | Load-shape analysis, major demand windows, bill forecast, energy-saving actions |
| Small business | Peak-demand profile, resilience requirement, solar/BESS/EV opportunity model |

### 3.3 Hardware stack

#### v1: Monitoring gateway

| Subsystem | Recommended function | Design notes |
|---|---|---|
| Current sensing | Split-core CT clamps; 1P and 3P support | Non-invasive install where feasible; CT orientation/phase mapping checks are critical |
| Voltage sensing | Isolated voltage measurement or certified meter interface | Needed for real power, PF, voltage events, and phase validation |
| Energy metering | Class-appropriate metering IC / DIN-rail meter | Revenue-grade metering is not required for v1 consumer insight, but accuracy must support recommendations |
| Main controller | MCU plus optional Linux-class communications module | MCU performs local sampling and fail-safe collection; Linux module supports containers, richer adapters, and OTA |
| Communications | Wi-Fi, Ethernet where available, LTE fallback | Store-and-forward queue is mandatory; consumer Wi-Fi cannot be assumed reliable |
| Local storage | eMMC/industrial flash | Buffer telemetry and event logs during outages |
| Interfaces | RS-485/Modbus, CAN, optical/IR, GPIO, BLE | Supports inverter, BMS, meter, and installer commissioning adapters |
| Power | Isolated AC-DC supply plus surge protection | Must tolerate local power variation; brownout detection and clean recovery required |
| Mechanical | DIN-rail or compact wall enclosure | Installer-friendly, tamper-aware, basic IP protection, labelled terminals |
| Security | Secure element / TPM-like device identity | Certificate storage, secure boot keying, device attestation |

#### Hardware variants

| SKU | Target | Hardware difference |
|---|---|---|
| Insight Lite | Bill-led or limited monitoring | Single-phase CTs, Wi-Fi, basic app onboarding |
| Insight Home | Most households | 1P/3P sensing, inverter interface, LTE fallback, DIN-rail install |
| Insight Solar Pro | Solar EPC servicing | Solar production meter/Modbus, inverter fault capture, technician diagnostics |
| Insight Small Site | Shops, clinics, small commercial | Three-phase meter support, multiple feeder CTs, demand profile and PQ events |

### 3.4 Firmware and edge software

**Real-time firmware responsibilities:**

- Sample current, voltage, real/reactive/apparent power, PF, frequency, and energy counters.
- Detect CT reversal, phase mismatch, invalid sensor state, and communications failure.
- Timestamp, aggregate, and buffer telemetry.
- Execute secure boot and signed firmware validation.
- Provide a local installer commissioning interface.
- Support local event rules: voltage excursion, outage/recovery, sustained abnormal base load, sensor fault.

**Gateway responsibilities:**

- Protocol adapters for Modbus RTU/TCP, CAN, selected inverter clouds/APIs, and meter interfaces.
- Local time-series cache and store-and-forward.
- Device provisioning, certificate rotation, remote command handling, and OTA update management.
- Local health checks, watchdog, log bundling, and remote support tunnel with explicit authorization.

### 3.5 Cloud software stack

| Service | Role |
|---|---|
| Device registry / PKI | Identity, certificates, firmware channels, device lifecycle |
| Telemetry ingestion | MQTT/HTTPS ingestion, validation, schema enforcement, queueing |
| Time-series platform | High-resolution electrical telemetry, aggregation, retention policies |
| Asset registry | Site, meter, inverter, PV, battery, customer, installer relationships |
| Tariff engine | State/DISCOM-specific tariff representation, slabs, time windows, fixed charges, export-credit assumptions |
| Forecasting engine | Load forecast, PV forecast, bill forecast, weather-normalised production baseline |
| Recommendation engine | PV/BESS sizing, backup circuit recommendation, opportunity score, scenario analysis |
| Anomaly engine | Production anomaly, load deviation, connectivity/fault alerts, confidence scoring |
| Installer portal | Survey, proposal, commissioning, photos, QA checklist, work orders |
| Customer app | Usage, bills, savings opportunity, alerts, explainable recommendations |

### 3.6 Core analytics

The basic home economics calculation is not a single “savings percentage.” It should calculate:

Self-consumed solar energy:
```text
E_self_use = sum over each time interval t of:

    min(P_PV(t), P_load(t)) × delta_t
```
Where:
- E_self_use = solar electricity consumed directly on site
- P_PV(t) = solar generation power at time t
- P_load(t) = household or site demand at time t
- delta_t = duration of the measurement interval

Exported solar energy:
```text
E_export = sum over each time interval t of:

    max(P_PV(t) - P_load(t), 0) × delta_t
```

Where:
- E_export = solar electricity exported to the grid
- P_PV(t) = solar generation power at time t
- P_load(t) = household or site electricity demand at time t
- max(..., 0) ensures exported energy cannot be negative
- delta_t = duration of the measurement interval

For a simulated battery:

Battery state of charge (SoC) update:
```text
SoC(t + 1) =
    SoC(t)
    + [charge_efficiency × charging_power(t) × delta_t]
    - [discharging_power(t) × delta_t / discharge_efficiency]
```

With units:
```text
SoC_kWh(t + 1) =
    SoC_kWh(t)
    + [eta_charge × P_charge_kW(t) × delta_t_hours]
    - [P_discharge_kW(t) × delta_t_hours / eta_discharge]
```

Where:
- SoC(t) = battery energy available at the start of interval t
- SoC(t + 1) = battery energy available at the end of interval t
- charge_efficiency = charging efficiency, typically below 1.0
- discharge_efficiency = discharging efficiency, typically below 1.0
- charging_power(t) = power sent into the battery during interval t
- discharging_power(t) = power drawn from the battery during interval t
- delta_t = duration of the measurement interval

subject to battery state-of-charge, charge/discharge power, reserve, and cycle constraints.

The recommended output should include **conservative, base, and upside** scenarios rather than a single payback claim.

### 3.7 User experiences

**Homeowner:**

- “How much electricity did I import, export, and use from solar?”
- “What will my bill likely be this month?”
- “Would a 5/7/10 kWh battery improve my economics?”
- “What appliances or time windows drive my evening peak?”
- “Is my solar system producing within expected range?”

**Installer/EPC:**

- Remote site pre-qualification.
- Digital survey and single-line-diagram capture.
- BESS sizing recommendation with assumptions visible.
- Commissioning checklist and photos.
- Remote alarm triage before sending a technician.

### 3.8 v1 success metrics

- Installation time below a defined target, ideally under 60–90 minutes for standard homes.
- >95% valid interval-data completeness after commissioning.
- Known sensor/phase configuration errors detected during commissioning.
- Monthly load and PV forecast error within a predefined band for qualified homes.
- Recommendation-to-install conversion rate tracked by segment.
- Predicted versus observed post-BESS savings tracked for every conversion.

---

## 4. Product 2 — SolarStack Home Control

### 4.1 Purpose

SolarStack Home Control is the supervisory energy-management layer for homes that have solar PV, stationary BESS, backup requirements, and optionally EV charging.

It must be **vendor-neutral**, AC-coupled or hybrid-inverter compatible where feasible, and designed as a retrofit-first system. It does not attempt to replace the certified BMS, inverter, or charger safety functions.

### 4.2 Customer outcomes

- Higher PV self-consumption.
- Reduced high-tariff or evening grid purchases.
- Reliable backup behaviour for selected critical loads.
- Battery reserve aligned with outage tolerance.
- EV charging scheduled around PV, tariff, and readiness requirements.
- Visibility into energy flows, BESS state, degradation indicators, and system faults.
- Remote diagnostic capability and lower service time.

### 4.3 Hardware stack

| Subsystem | Function | Implementation guidance |
|---|---|---|
| Home controller | Supervisory control and local optimisation | Industrialised gateway/IPC plus real-time MCU or safety-oriented control board |
| Grid meter / CTs | Import/export and phase data | 1P/3P support; validate phase association during commissioning |
| PV interface | Read PV generation and inverter state | Modbus/SunSpec where available; selected OEM APIs; CT fallback |
| BESS interface | Read SoC/SoH/capability; command power/setpoints | CAN/Modbus/vendor API; command abstraction by OEM |
| PCS/hybrid inverter | Execute bounded charge/discharge / mode commands | Only certified interfaces; never bypass inverter protections |
| EVSE interface | Set charging schedule/current/power cap | OCPP, Modbus, or vendor APIs depending on EVSE class |
| Backup switching | Critical-load panel / ATS / contactor interface | Certified equipment; state feedback mandatory; site-specific electrical design |
| Controllable loads | Optional relay/contactor interface | Water heater, pumps, HVAC enable/disable where appropriate; load shedding must be conservative |
| Connectivity | Ethernet/Wi-Fi + LTE fallback | Local operation persists without cloud |
| Power / enclosure | DIN rail or wall-mounted control panel | Surge protection, isolation, watchdog, serviceable wiring |

### 4.4 Local control architecture

```text
                    ┌─────────────────────────────────────┐
                    │        HOME CONTROL EDGE            │
                    │                                     │
                    │ State estimator                     │
                    │ Local policy engine                 │
                    │ Constraint validator                │
                    │ BESS / EVSE command adapters        │
                    │ Fallback and outage manager         │
                    └───────┬──────────┬──────────┬───────┘
                            │          │          │
                         PV inverter  BESS       EVSE
                            │          │          │
Grid meter / CTs ───────────┴──────────┴──────────┴──────► Loads / critical panel
```

### 4.5 Software functions

#### Local, offline-capable functions

- Maintain minimum BESS reserve.
- Enforce maximum grid-import/demand limit where configured.
- Follow approved EV charging schedule.
- Apply PV-self-consumption policy.
- Detect grid outage and transition to equipment-supported backup mode.
- Hold safe fallback commands when cloud or external APIs are unavailable.
- Log state changes, alarms, and command acknowledgements.

#### Cloud-assisted functions

- Day-ahead PV and load forecasts.
- Tariff-aware schedules.
- Battery degradation-aware dispatch policy.
- Customer “ready-by” EV charging optimisation.
- Remote diagnostics, fleet benchmarking, OTA, and model improvement.
- Savings verification against baseline.

### 4.6 Home optimisation policies

| Policy | Objective | Example |
|---|---|---|
| Self-consumption | Use PV locally before exporting | Charge battery from excess afternoon PV; discharge in evening |
| Backup reserve | Preserve resilience | Keep 40% SoC when local outage probability or customer setting is high |
| ToD optimisation | Avoid expensive imports | Charge BESS off-peak only if forecasted PV cannot meet evening demand |
| EV-ready-by | Deliver specified EV energy by deadline | Ensure 12 kWh by 07:00 while respecting import cap |
| Demand cap | Avoid high instantaneous grid draw | Limit EVSE to 2.3 kW when home load rises |
| Battery-care | Protect lifetime | Avoid sustained 100% SoC, high C-rate, or deep cycles beyond warranty policy |

Home optimisation objective:
```text
Minimise:

    Electricity_cost
  + Battery_degradation_cost
  + Critical_load_unserved_cost
```

Interpretation:
The system should minimise the total effective cost of energy, rather than blindly
minimising grid imports. It should consider:

- The retail cost of imported electricity.
- The value of exported solar.
- The long-term wear caused by unnecessary battery cycling.
- The cost or inconvenience of failing to preserve energy for critical loads during an outage.

### 4.7 Safety model

SolarStack Home Control must clearly distinguish between **advisory**, **supervisory control**, and **protection**.

| Layer | SolarStack authority | Must remain in certified equipment |
|---|---|---|
| Advisory | Full | N/A |
| Supervisory scheduling | Bounded commands | Asset capability enforcement |
| Load prioritisation | Bounded relay/controller outputs | Circuit protection and manual override |
| Grid outage response | Observe and request supported modes | Anti-islanding, transfer timing, grid-forming, synchronisation |
| Battery safety | Read and reduce demand on fault | Cell protection, balancing, contactor control, thermal limits |
| EV charging safety | Set allowed schedule/current | Pilot signaling, isolation, fault detection, connector safety |

### 4.8 Minimum integrations

- One priority inverter family or an AC-coupled configuration for v1.
- One BESS/PCS partner with documented controls.
- One home EVSE integration route.
- Certified backup/transfer architecture validated by electrical experts.
- A multi-vendor integration capability matrix published internally and eventually exposed to installers/customers.

### 4.9 Home Control success metrics

- Percentage of time local control continues after cloud loss.
- Command success and acknowledgement rate by asset vendor.
- Reduction in grid imports in eligible homes.
- Backup-transition success in controlled tests.
- Number of service visits per installed system.
- Forecast-to-realised savings variance.
- Battery operations within warranty-safe envelope.

---

## 5. Product 3 — SolarStack Site Control

### 5.1 Purpose

SolarStack Site Control serves:

- Apartment complexes and residential projects.
- Small commercial sites.
- Fleet depots.
- Retail, hospitality, and workplace charging locations.
- CPO sites with solar PV, BESS, and multiple EVSEs.

The product’s objective is to operate constrained electrical infrastructure as an economically optimised site rather than as isolated assets.

### 5.2 Site energy balance

At every control interval:

CPO site power balance:
```text
Grid power
+ Solar PV power
+ Battery discharge power

=

EV charging power
+ Building / site auxiliary-load power
+ Battery charging power
+ Conversion and distribution losses
```

As an equation:
```text
P_grid + P_PV + P_BESS_discharge = P_EVSE + P_site_load + P_BESS_charge + P_losses
```

At every control interval, the site controller must balance incoming and outgoing power.
The controller may shift energy between the grid, PV, stationary battery, EV chargers,
and auxiliary loads, but it cannot violate the physical power balance.

Subject to constraints:

Grid-import constraint:
```text
P_grid <= P_sanctioned_demand
```

Meaning:
The controller must not import more power from the grid than the site's sanctioned
demand limit, except where the electrical and commercial arrangement explicitly permits it.

Transformer-capacity constraint:
```text
P_transformer <= Transformer_rating × Derating_factor
```

Meaning:
The total site load passing through the transformer must remain below the allowed
operating limit after accounting for temperature, ambient conditions, equipment age,
utility requirements, and engineering safety margin.

Battery operating window:
```text
SoC_min <= SoC_BESS <= SoC_max
```

Meaning:
The controller must keep the battery within configured minimum and maximum state-of-charge limits.

Typical reasons:
- Preserve backup reserve.
- Avoid deep discharge.
- Avoid prolonged high state of charge.
- Protect battery warranty and lifetime.
- Maintain a safety margin for unexpected load or outage events.

Aggregate EV-charger power constraint:
```text
Sum of charger power across all active connectors
    <=
Available site capacity
```

Implementation form:
```text
sum(P_EVSE[i] for each active charger i) <= P_available_site_capacity
```

Available site capacity is not fixed. It changes with:
- Building and auxiliary loads.
- Solar generation.
- Battery state and available discharge power.
- Transformer and sanctioned-demand limits.
- Grid quality constraints.
- Reserve policy for outages.

### 5.3 Hardware stack

| Subsystem | Function | Design expectations |
|---|---|---|
| Industrial site controller | Runs local EMS and site interfaces | DIN-rail industrial PC/gateway; industrial temperature; watchdog; redundant power option |
| Real-time controller / PLC | Deterministic IO and fast local sequencing | Separate from cloud/gateway where availability or safety needs justify it |
| Revenue meter interface | Tariff/billing-quality import/export measurement | Use certified meter integration; retain source of truth for billing separately |
| Feeder and sub-metering | PV, BESS, charger banks, auxiliary loads | Three-phase, multi-feeder, interval data; phase quality and CT health diagnostics |
| Power-quality meter | Voltage, frequency, THD, imbalance, events | Required where grid quality impacts uptime or equipment life |
| BESS PCS interface | Read capability and command power/mode | Modbus/IEC/vendor protocol adapters with capability envelopes |
| PV inverter interface | Forecast/monitor/curtail where authorised | Vendor-specific adapters; production meter fallback |
| EVSE controller integration | Charger state, sessions, charging power, fault codes | OCPP 1.6J initially; roadmap to OCPP 2.0.1 |
| Switchgear/relay IO | Status and selected non-safety controls | Direct control only under approved electrical design; feedback interlocks required |
| Network | Ethernet, fibre/WAN, dual-SIM LTE/5G, VPN | Industrial firewall, segmented VLANs, store-and-forward |
| Local HMI | Technician/operations access | Read-only by default; role-controlled local overrides and emergency procedures |
| UPS / control supply | Keep controller, communications, and metering alive | Controlled shutdown and recovery; independent of main BESS policy |

### 5.4 Site controller software

```text
┌───────────────────────────────────────────────────────────┐
│                    SITE CONTROL SOFTWARE                  │
├───────────────────────────────────────────────────────────┤
│ Site state estimator                                      │
│ Asset capability registry                                 │
│ Constraint validator and safety envelope                  │
│ Real-time dispatch engine                                 │
│ Charger power allocator                                   │
│ BESS scheduler                                            │
│ PV / load / EV forecast cache                             │
│ Outage and degraded-grid manager                          │
│ Local alarm correlation                                   │
│ Protocol adapters: OCPP, Modbus, CAN, BACnet, APIs        │
│ Local historian / event recorder                          │
│ Secure communications, OTA agent, remote-support controls │
└───────────────────────────────────────────────────────────┘
```

### 5.5 CPO-specific functions

| Function | Description | Business value |
|---|---|---|
| Dynamic load management | Allocate available site power across active chargers | More connectors supported from same grid connection |
| Demand-cap control | Keep grid draw under sanctioned demand/transformer limit | Avoid penalty, overload, and service interruption |
| BESS peak shaving | Discharge BESS during coincident peak load | Lower demand charge exposure and avoid derating sessions |
| PV self-consumption | Direct PV to charging/site loads or BESS | Lower energy-cost per delivered kWh |
| EV session prioritisation | Allocate power by booking, fleet SLA, departure time, tariff, or paid service tier | Better customer experience and revenue management |
| Resilience mode | Preserve essential systems and selected chargers during outage/grid weakness | Higher charging uptime |
| Charger health correlation | Connect charger fault patterns with power-quality, grid, thermal, network, or site events | Faster root cause and fewer truck rolls |
| Energy-cost attribution | Assign grid/PV/BESS cost to sessions or customer categories | Better pricing and unit economics |
| Capacity planning | Identify when to add chargers, BESS, PV, or grid connection capacity | Capex prioritisation |

### 5.6 Local fallback modes

Cloud connectivity must not be required for continued site operation. The site controller should support at least:

| Mode | Trigger | Local behaviour |
|---|---|---|
| Normal | Cloud and all assets available | Follow optimised schedule and receive updates |
| Cloud disconnected | WAN/API loss | Keep last validated policy; continue local demand cap and session allocation |
| Forecast stale | Weather/data feed loss | Switch to conservative PV/load assumptions |
| BESS unavailable | Fault or SoC constraint | Derate chargers and enforce transformer/demand limits |
| EVSE backend loss | CSMS unavailable | Follow pre-approved local authorization/session policy where legal and supported |
| Grid weak | Voltage/frequency/PQ event | Reduce sensitive loads, preserve control power, protect site assets |
| Grid outage | Utility loss | Transition only as supported by certified equipment; preserve critical loads and selected charging policy |
| Emergency | Safety or electrical fault | Safe stop / load shed / isolate per approved protection and operational procedure |

### 5.7 Site EMS optimisation problem

A simplified site objective is:

CPO site optimisation objective:

Maximise:

    Charging_session_revenue
  - Grid_energy_cost
  - Demand_charge_cost
  - Battery_degradation_cost
  - Unserved_or_delayed_charging_cost
  - Downtime_cost

Interpretation:
The CPO controller should maximise profitable, reliable charging service—not merely
minimise energy use. It must trade off:

- Revenue from completed charging sessions.
- Grid energy purchased under applicable tariffs.
- Demand charges or penalties from exceeding site limits.
- Battery wear created by peak shaving or backup use.
- Lost revenue and customer dissatisfaction from delayed or failed sessions.
- Cost of charger, network, or grid-related downtime.

The optimiser needs hard constraints for asset safety and contractual requirements, with business-policy weights for:

- Charging session priority.
- Minimum battery reserve.
- Maximum grid demand.
- Tariff windows.
- Battery-life policy.
- PV curtailment restrictions.
- Customer fairness.
- Site critical-load commitments.

### 5.8 Standards and protocols

For India, the product should be engineered around open integration where possible. CEA materials reference AIS 138 Parts 1 and 2 for AC/DC conductive charging, while India-focused EV charging guidance identifies OCPP as the protocol linking charging stations and central management systems. The India Energy Stack architecture also anticipates interoperable EV-charging information and transaction flows across operators. [1][2][3]

Recommended protocol roadmap:

| Domain | Initial support | Roadmap |
|---|---|---|
| Charger ↔ backend | OCPP 1.6J | OCPP 2.0.1 |
| Roaming / eMSP | Partner/API-specific | OCPI where needed |
| Inverter/PCS | Modbus RTU/TCP; vendor APIs | SunSpec and richer OEM adapter library |
| BMS | CAN / Modbus / vendor gateway | Common capability abstraction |
| Building loads | Modbus, selected BACnet | Broader BMS integrations |
| Smart/revenue meters | Modbus, DLMS/COSEM where authorised | IES smart-meter exchange adapters |
| Utility / DR | Project-specific APIs | OpenADR and India Energy Stack integrations as access matures |
| Edge-cloud | MQTT over TLS and HTTPS | Device management at scale, certificate rotation, private APN/VPN options |

### 5.9 Site Control success metrics

- Charger uptime and availability.
- Charging-session completion rate.
- Grid-import peak versus sanctioned-demand limit.
- Avoided or deferred transformer/grid-upgrade capacity.
- Energy cost per delivered kWh.
- PV self-consumption rate.
- BESS utilisation and degradation-normalised value.
- Mean time to detect and resolve faults.
- Remote resolution rate versus truck-roll rate.
- Local autonomous-operation duration during cloud loss.

---

## 6. Product 4 — SolarStack Fleet OS

### 6.1 Purpose

Fleet OS is the cloud operating layer for SolarStack Insight, Home Control, and Site Control deployments. It becomes strategically valuable once SolarStack manages a large enough fleet to benchmark performance, improve forecasts, coordinate service, and offer portfolio-level economics.

### 6.2 Core modules

| Module | Function |
|---|---|
| Tenant and identity platform | Organisations, portfolios, users, roles, installers, CPO operators, APIs |
| Asset registry | Canonical inventory, capabilities, firmware, warranty, topology, configuration |
| Device management | Provisioning, certificates, OTA campaigns, remote diagnostics, health monitoring |
| Data platform | Streaming ingestion, historian, aggregation, feature store, data-quality scoring |
| Digital-twin service | Current/forecast state of site and assets, relationships, constraints |
| Forecast engine | PV, load, EV arrival/energy demand, outage-risk proxy, tariff forecast |
| Optimisation service | Day-ahead and intraday schedules; scenario simulation; policy tuning |
| Tariff / cost engine | Electricity tariff, demand charges, export credits, charging cost allocation |
| Service operations | Alarms, ticketing, RMA, technician workflow, installation QA, knowledge base |
| Customer experience | Consumer app, site/operator portal, reports, billing/savings view |
| Partner APIs | EPC, financing, OEM, EVSE, CPO CSMS, payment/eMSP, utility interfaces |
| Security and audit | Access control, logs, command audit trail, data retention, privacy consent |

### 6.3 Data architecture

```text
Devices / asset adapters
        │
        ├── MQTT / HTTPS with mutual TLS
        ▼
Ingestion gateway → schema validation → event bus → time-series store
                                      │              │
                                      │              ├── real-time dashboards
                                      │              ├── alarm engine
                                      │              ├── feature store
                                      │              └── data lake / reporting
                                      ▼
                            Digital twin + capability registry
                                      │
                                      ▼
                 Forecasting + optimisation + recommendation services
                                      │
                                      ▼
                          Command service / policy deployment
                                      │
                                      ▼
                     Edge validates constraints and executes
```

### 6.4 Optimisation hierarchy

| Horizon | Decision | Example |
|---|---|---|
| Day-ahead | Initial battery/EVSE schedule | Preserve BESS for an expected evening demand peak |
| Intra-day | Re-optimise on forecast or session changes | EV arrives early; redistribute power across connectors |
| Real time | Constrain or correct dispatch | Transformer load rises; cap charging or dispatch BESS |
| Portfolio | Plan service/capacity/fleet policy | Identify sites where BESS pays for a grid upgrade deferral |

### 6.5 Explainability requirement

Every recommendation and important control action should have an explainable record:

```text
Action: limit Charger 4 from 60 kW to 30 kW
Reason: Site demand forecast + active load exceeded 95% of transformer policy limit
Alternatives considered: BESS discharge unavailable because reserve floor is 30%
Policy owner: CPO tariff / uptime policy v2.3
Duration: 18:12–18:34
Expected result: Avoid transformer overload while completing priority session on Charger 1
```

This is essential for installer trust, customer support, regulatory conversations, and model debugging.

---

## 7. Cybersecurity, privacy, and functional safety

### 7.1 Security architecture

SolarStack controls energy assets and may access sensitive household/site data. The minimum architecture should include:

- Hardware-rooted device identity and per-device certificates.
- Secure boot and signed firmware/containers.
- Mutually authenticated MQTT/HTTPS, encrypted in transit.
- Encrypted secrets at rest and managed key rotation.
- Role-based access control and tenant isolation.
- Separate operational technology (OT) and IT network zones at CPO sites.
- Command authorization, capability checks, rate limits, and full audit trail.
- OTA staged rollout, canary deployment, signed manifests, automatic rollback.
- Vulnerability management, SBOMs, dependency scanning, and incident response.
- Explicit remote-support approval with time-bound access.

### 7.2 Privacy architecture

- Collect the minimum data required for stated outcomes.
- Separate personally identifiable information from high-frequency telemetry where possible.
- Provide customer consent and clear data-use controls.
- Define data retention by product tier and commercial purpose.
- Use anonymised/aggregated data only under documented governance.
- Design for applicable Indian privacy obligations and contractual requirements; obtain legal advice before launch.

### 7.3 Safety boundary

SolarStack must never make cloud availability a prerequisite for electrical safety. The following remain outside SolarStack’s direct safety authority:

- Battery cell protection and contactor safety.
- Inverter overcurrent, anti-islanding, grid synchronisation, and thermal protection.
- EVSE isolation, connector, earth-fault, and pilot-signal safety.
- Protective relay trip functions.
- Manual emergency isolation and approved electrical safety procedures.

SolarStack may request modes or setpoints only within declared, authenticated asset capabilities.

---

## 8. Installation, commissioning, and service stack

The operational system is a product feature. Poor commissioning will destroy analytics accuracy and trust.

### 8.1 Digital installation workflow

1. **Pre-qualification:** bill upload, location, phase type, sanctioned demand, existing assets, roof/site survey.
2. **Design:** wiring/topology capture, asset compatibility, BOM, backup-circuit definition, proposed policy.
3. **Install:** guided technician workflow, QR-based device binding, photographs, torque/checklist evidence.
4. **Commission:** CT/phase validation, meter sanity checks, inverter/BESS/EVSE handshake, connectivity test, safe fallback test.
5. **Acceptance:** customer/operator review of asset inventory, policy, emergency procedure, and expected outcome.
6. **Post-install validation:** 7/30-day data-quality score, baseline comparison, fault review, savings verification.

### 8.2 Commissioning tests

| Test | Home | CPO site |
|---|---|---|
| CT direction/phase mapping | Mandatory | Mandatory |
| Meter value sanity | Mandatory | Mandatory |
| PV production readout | Where present | Mandatory where PV present |
| BESS capability/status | Where present | Mandatory where BESS present |
| EVSE command path | Optional | Mandatory |
| Network failover | Recommended | Mandatory |
| Cloud-loss local policy | Recommended | Mandatory |
| Demand-cap test | Optional | Mandatory |
| Backup/outage test | Where designed | Site-specific and safety-approved |
| Alarm and ticket path | Mandatory | Mandatory |

### 8.3 Service model

- Remote first-line diagnosis using correlated telemetry, events, and asset status.
- Installer/service partner dispatch only after remote triage.
- Configuration/version history for every site.
- Warranty and RMA workflow linked to device telemetry and diagnostic evidence.
- Root-cause taxonomy that distinguishes power quality, connectivity, electrical wiring, inverter/BESS, EVSE, and software causes.

---

## 9. Roadmap and sequencing

### Phase 0 — Architecture and design partners: 0–6 months

- Define asset model, protocol abstraction, telemetry schema, and security baseline.
- Select one smart-meter/CT gateway architecture and one field-installation partner.
- Recruit EPC, inverter, BESS, and EVSE design partners.
- Build bill-led simulator and installer proposal workflow before full hardware fleet deployment.
- Establish target customer cohorts: solar-only, high-evening-load, outage-sensitive, and small commercial.

### Phase 1 — Insight pilot: 6–12 months

- Deploy 30–50 instrumented Insight units across target cohorts.
- Validate measurement accuracy, telemetry completeness, load/PV forecasting, and BESS recommendations.
- Run installer workflow and remote diagnostics in real sites.
- Track recommendation-to-purchase intent and identify economically viable BESS cohorts.

### Phase 2 — Home Control pilot: 12–24 months

- Integrate one BESS/PCS partner, one inverter route, and one home EVSE route.
- Deploy 10–20 controlled solar+BESS installations.
- Validate backup behaviour, local autonomy, realised savings, service workload, and battery operations.
- Publish an internal compatibility and safety matrix before broad sales.

### Phase 3 — Site Control: 18–36 months

- Pilot one constrained commercial/CPO/depot site.
- Integrate OCPP, PV, BESS, multi-metering, and demand-cap functions.
- Demonstrate avoided peak, charger uptime, session completion, and energy-cost improvement.
- Build NOC/service workflows and multi-site operator portal.

### Phase 4 — Fleet OS and grid-ready services: 30+ months

- Scale portfolio analytics, capacity planning, and cross-site optimisation.
- Evaluate demand-response, aggregation, and utility/India Energy Stack integration only where commercial and regulatory interfaces are available.
- Consider financing, BESS-as-a-service, or performance contracting only after observed unit economics and service reliability are proven.

---

## 10. Metrics that matter

### 10.1 Product metrics

| Product | Leading metrics | Outcome metrics |
|---|---|---|
| Insight | Installation time, data completeness, recommendation confidence, app engagement | Solar/BESS qualified-lead conversion, forecast accuracy, support burden |
| Home Control | Command success, offline continuity, configuration success | Self-consumption improvement, backup success, realised savings, service calls/system |
| Site Control | Asset connectivity, local-policy success, charger-control response | Charger uptime, session completion, demand-cap compliance, cost/kWh |
| Fleet OS | Device health, OTA success, alarm precision, remote resolution | Gross margin/service cost, fleet availability, retention, portfolio value creation |

### 10.2 Commercial metrics

- Customer acquisition cost by channel.
- Installer productivity and rework rate.
- Hardware gross margin after warranty reserve.
- Subscription attach rate and retention.
- Service cost per active asset.
- BESS conversion rate among qualified prospects.
- Predicted versus realised customer savings.
- CPO value captured per site: avoided demand, additional sessions, uptime improvement, and service cost reduction.

---

## 11. What SolarStack should not claim too early

- Guaranteed battery payback for all solar households.
- Predictive maintenance before sufficient labelled fault data exists.
- Utility grid-services revenue before contracts, telemetry, settlement rules, and dispatch rights exist.
- Universal compatibility before integration testing and support maturity.
- Cloud-driven safety functions.
- Full CPO operational capability before local controller reliability and EVSE interoperability are validated.

The credible early claim is narrower:

> SolarStack measures real energy behaviour, identifies where solar/storage/charging investments make sense, and optimises connected assets inside verified safety and capability constraints.

---

## 12. Reference standards and ecosystem direction

Indian EV charging guidance and standards establish the need for interoperable charger and backend communication. CEA materials list AIS 138 Part 1 for AC charging and AIS 138 Part 2 for DC charging, while NITI Aayog charging guidance identifies OCPP as a protocol connecting charge points with central management systems. The India Energy Stack architecture describes interoperable EV-charging discovery, reservation, and charging-session concepts across operators. [1][2][3]

The Electricity Rules and the National Framework for Promoting Energy Storage Systems recognise energy storage as part of the power system and contemplate ownership/operation by consumers and other power-sector entities; the framework also discusses aggregation of distributed BESS. These are enabling signals, not proof of an immediate consumer-grid-services business model. [4][5]

---

## References

[1] Central Electricity Authority, *EV Charging Standards*. https://cea.nic.in/ev-charging-standards/?lang=en

[2] NITI Aayog, *Electric Vehicle Charging Infrastructure and Its Grid Integration in India*. https://www.niti.gov.in/sites/default/files/2021-09/Report1-Fundamentals-ofElectricVehicleChargingTechnology-and-its-Grid-Integration_GIZ-IITB.pdf

[3] REC Limited, *India Energy Stack Architecture Document v0.3*. https://recindia.nic.in/uploads/files/IES-Architecture-Documentv0-3Final.pdf

[4] Ministry of Power, *Electricity (Amendment) Rules, 2022*. https://powermin.gov.in/sites/default/files/Electricity_Amendment_Rules_2022.pdf

[5] Ministry of Power, *National Framework for Promoting Energy Storage Systems*, August 2023. https://powermin.gov.in/sites/default/files/webform/notices/National_Framework_for_promoting_Energy_Storage_Systems_August_2023.pdf

---

## Appendix A — Minimum interface contract

Every connected asset should expose an adapter contract that includes:

```text
Asset identity
- manufacturer, model, serial number, firmware version
- protocol, endpoint, authentication mode

Capabilities
- rated power, allowed modes, min/max setpoints
- response latency, command granularity
- safety restrictions and warranty constraints

Telemetry
- timestamp, quality/confidence, measurement unit
- current power, energy counters, state, fault/alarm code
- health and communication status

Commands
- supported command type
- requested setpoint, valid duration, priority, correlation ID
- acknowledgement, applied value, rejection reason

Lifecycle
- install/commission status
- warranty state, maintenance history, configuration version
```

## Appendix B — Example control-policy hierarchy

```text
1. Safety and certified equipment constraints
2. Grid/protection and contractual limits
3. Critical-load and outage reserve commitments
4. Charger session commitments / customer SLA
5. Asset-health and warranty limits
6. Cost and tariff optimisation
7. PV self-consumption preference
8. Customer comfort and discretionary loads
```

This order prevents an optimiser from sacrificing safety, contractual uptime, or battery health merely to reduce a small amount of energy cost.
