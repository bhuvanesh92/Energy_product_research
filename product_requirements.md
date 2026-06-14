***

# HEMS Product Requirements Document (with Acceptance Criteria)

> Version: 0.2 – with acceptance criteria for hardware, firmware, cloud and app teams  

***

## 1. Product overview

### 1.1 Vision

Build an India‑first Home & Industrial Energy Management System (HEMS) box that:

- Starts as a **non‑intrusive energy monitor** (usage statistics, education, bill insights). [ijraset](https://www.ijraset.com/best-journal/smart-home-energy-monitoring-and-management-system)
- Evolves into a **solar + battery + grid orchestration controller** that optimizes when to draw from grid, when to use stored solar, and when to export. [sciencedirect](https://www.sciencedirect.com/science/article/abs/pii/S0196890424003108)
- Extends to **industrial energy management** (sub‑metering, predictive maintenance) and later **EV charging sites** (load balancing, EV charging management). [akraniq](https://akraniq.com/solutions/industrial-iot)

**Acceptance criteria**

- A single hardware family (shared PCB/platform) can serve at least:
  - 1‑phase homes.  
  - 3‑phase homes / small businesses.  
  - A C&I pilot site (≥1 three‑phase panel) without redesign.  
- Roadmap shows documented upgrade path to:
  - Solar + battery optimization.  
  - EV‑site load management (via software and integrations).  

### 1.2 Primary personas

- Residential homeowner (high electricity bill, considering or using solar). [indem](https://indem.in)
- Small business / MSME owner (shops, workshops, small factories). [iotdashboard](https://iotdashboard.in)
- Industrial facility manager (multi‑panel, multi‑line plant). [irjet](https://www.irjet.net/archives/V12/i1/IRJET-V12I1103.pdf)
- Solar EPC / installer seeking an integrated monitoring/control solution. [enerman](https://enerman.in)
- Future: EV charge point operator (CPO) / fleet operator. [pulseenergy](https://pulseenergy.io)

**Acceptance criteria**

- UX team provides at least one persona‑driven journey per persona (flow + screens).  
- At least one field pilot is run for:
  - 5+ residential sites.  
  - 2+ MSME / industrial sites.  
  - 1+ EPC partner.  

***

## 2. Regulatory and standards requirements (India)

### 2.1 Legal and regulatory framework

- Comply with **Electricity Act 2003** and **CEA (Measures relating to Safety and Electric Supply) Regulations** for all consumer‑side work. [en.wikipedia](https://en.wikipedia.org/wiki/Central_Electricity_Authority_Regulations)
- Only **licensed contractors** and competent persons perform installation/alteration where required by CEAR and state rules. [pib.gov](https://www.pib.gov.in/PressReleasePage.aspx?PRID=2041636)
- When interfacing to DISCOM meters / AMI, meet DISCOM’s published specs and approvals in addition to BIS/CEA. [services.bis.gov](https://www.services.bis.gov.in/tmp/tbl5_2024-11-10_1153.pdf)

**Acceptance criteria**

- Legal/regulatory checklist is completed and signed off by a qualified electrical engineer for each target state.  
- Installation SOP explicitly states:
  - Roles allowed to perform install.  
  - Tests and documentation required before energizing.  
- For any DISCOM integration, written approval or email confirmation from DISCOM is archived.  

### 2.2 Installation standards (wiring, earthing, panels)

- **IS 732 – Electrical Wiring Installations**: installation, wiring, protection, testing must comply. [mace.ac](https://www.mace.ac.in/files/INDIAN%20STANDARDS%20BUILDING%20SERVICE%20-%20ESD.pdf)
- **NEC‑2011 (India)**: design practices for wiring, panels, clearances, protective devices. [electrician.iti](https://electrician.iti.directory/book/export/html/50)
- **IS 3043 – Earthing**: all required earthing and bonding to consumer’s system. [archive](https://archive.org/details/gov.in.is.3043.2018)

**Acceptance criteria**

- Reference design (single‑line) reviewed and stamped “compliant with IS 732/NEC‑2011” by external or internal chief electrical engineer. [linkedin](https://www.linkedin.com/pulse/national-electrical-code-india-basic-understanding-6lk9c)
- On pilot installs, pre‑commissioning report includes:
  - Insulation resistance test results.  
  - Earth continuity and earth resistance measurements.  
  - Functional test of protective devices (RCCB/MCB) where touched. [mace.ac](https://www.mace.ac.in/files/INDIAN%20STANDARDS%20BUILDING%20SERVICE%20-%20ESD.pdf)
- Site photos confirm:
  - Correct cable routing and segregation.  
  - Panel clearances per NEC‑2011.  

### 2.3 Product / equipment standards (HEMS device)

- **IS / IEC 61439** for any board‑like assembly. [scribd](https://www.scribd.com/document/752659341/IEC-61439-Low-Voltage-Switchgear-Control-Gear-Assemblies)
- **IS 13252** for IT‑class electronics. [services.bis.gov](https://www.services.bis.gov.in/tmp/tbl5_2024-11-08_1193.pdf)
- **IS 1293** for any socket outlets. [services.bis.gov](https://www.services.bis.gov.in/tmp/tbl5_2024-12-02-11-19.pdf)
- **IS 16444** where the product is part of revenue metering. [scribd](https://www.scribd.com/document/799297854/16444-2015)
- EES/BESS and new CEA BESS regulations if integrated batteries are involved. [energetica-india](https://www.energetica-india.net/news/cea-moves-to-upgrade-indias-power-sector-with-new-draft-rules-on-infrastructure-and-cybersecurity)

**Acceptance criteria**

- Final BOM and drawings are accepted by an accredited BIS test lab as suitable for testing against the above standards (where applicable).  
- At least one hardware revision passes:
  - Type tests for enclosure/assembly as per IEC/IS 61439 (if applicable). [assets1.sc.hager](https://assets1.sc.hager.com/exports/files/Guide_Normes_IEC_61439_GB_web_sRGB.pdf)
  - Safety tests for IT equipment per IS 13252 (creepage, clearances, dielectric strength, leakage). [services.bis.gov](https://www.services.bis.gov.in/tmp/tbl5_2024-11-08_1193.pdf)
- If sockets are present, the socket part has a conformity report to IS 1293 from supplier or lab. [tuv](https://www.tuv.com/regulations-and-standards/en/india-standard-update-is-1293-for-plugs-and-socket-outlets.html)
- If a smart‑meter function is integrated, test lab confirms compliance to IS 16444 for that function. [services.bis.gov](https://www.services.bis.gov.in/tmp/tbl5_2024-11-10_1153.pdf)

***

## 3. Functional requirements

### 3.1 Core monitoring and analytics

**F1. Whole‑site monitoring**

- Measure V, I, P, kWh, PF, f per phase; optional Q and THD. [ijraset](https://www.ijraset.com/best-journal/smart-home-energy-monitoring-and-management-system)
- Support:
  - 1‑phase 230 V up to 100–200 A.  
  - 3‑phase 415 V up to ≥200 A/phase (extendable via CTs). [indiamart](https://www.indiamart.com/proddetail/emporia-3-phase-smart-electricity-monitor-23366067688.html)
- Data resolution: 1–60 s locally, 1–15 min to cloud. [sciencedirect](https://www.sciencedirect.com/science/article/abs/pii/S0196890424003108)

**Acceptance criteria**

- In lab tests vs a calibrated reference meter:
  - Active power and energy error ≤1 % from 10 % to 100 % of rated current, at PF 0.5 lag to 1.0, for both 1‑phase and 3‑phase. [irjet](https://www.irjet.net/archives/V12/i1/IRJET-V12I1103.pdf)
- Device successfully measures and uploads data at configured intervals (e.g., 10 s local sampling, 1‑min cloud aggregation) continuously for 7 days with <0.1 % sample loss.  
- System can be configured for:
  - 1‑phase mode.  
  - 3‑phase 4‑wire mode.  
  - Passing configuration tests in both.  

**F2. Circuit / feeder‑level monitoring**

- Multiple CT channels (mains + branch circuits), similar to Emporia (up to 16) and SPM (4‑channel blocks). [shop.sonoff](https://www.shop.sonoff.in/product-category/computer-accessories/energy-saving/)
- User labeling of each circuit in app.  

**Acceptance criteria**

- For the selected SKU, at least:
  - Residential version: 3 mains CT + ≥8 branch CT channels.  
  - C&I version: 3 mains CT + ≥16 branch CT channels.  
- User can:
  - Add/edit labels for each channel in app/web.  
  - See per‑channel power and kWh on dashboards and exports.  
- Integration test: sum of all CT channels is within ±5 % of main CT reading under mixed load in lab.  

**F3. Appliance / machine insights & education**

- Basic breakdown by circuit and equipment category. [inc42](https://inc42.com/buzz/meet-the-12-startups-part-of-panasonics-accelerator-programme/)
- Tips on load shifting and cost impact (tariff‑aware). [ijecs](https://www.ijecs.in/index.php/ijecs/article/view/1927/1781)

**Acceptance criteria**

- For each monitored site, app displays:
  - Top 5 consumers (circuits or inferred appliances) for selected period.  
  - At least 3 generated “insights” per month (e.g., “AC usage increased 20 % vs last month”).  
- For at least one supported utility tariff, a bill comparison view shows:
  - Actual vs “if you adopted recommended schedule” cost difference.  

### 3.2 Solar, grid, and battery integration

**F4. Solar‑ready import/export metering**

- Separate tracking of import, export, and generation. [techhive](https://www.techhive.com/article/1804290/emporia-gen-2-vue-energy-monitor-review.html)

**Acceptance criteria**

- On a test site with net‑metered solar:
  - Import and export totals over a billing cycle differ from DISCOM meter by ≤2 %.  
  - Solar generation measured by HEMS differs from inverter’s internal meter by ≤2 %.  

**F5. Energy‑flow visualization & balancing**

- UI showing flows between grid, solar, battery, EV, loads. [play.google](https://play.google.com/store/apps/details?id=com.oorjan.solar_monitoring&hl=en)
- Optimization respecting tariffs and constraints. [combined](https://www.combined.energy/hems)

**Acceptance criteria**

- Flow diagram correctly reflects sign and magnitude of power on each leg (grid, solar, battery, EV, load) for test scenarios (pure import, pure export, mixed, battery only).  
- In simulation mode with historical data and configured tariffs:
  - Enabling optimization leads to at least 10 % reduction in modeled grid energy or bill vs naive baseline on 3 distinct test datasets.  

**F6. Battery and inverter integration**

- Modbus/SunSpec or vendor API integration. [gridx](https://www.gridx.ai/use-cases/home-energy-management-system)

**Acceptance criteria**

- For at least one inverter + battery pair used in India:
  - HEMS can read SoC, charge/discharge power, and set active power limit and charge window.  
- Changes made from app (e.g., “no export between 18:00–22:00”) are reflected in inverter behaviour in <60 s in lab tests.  

### 3.3 EV charging and site capacity (future)

**F7. EV charger monitoring and control**

- Monitor EV load; share capacity and load info with EV‑CSMS. [auto.economictimes.indiatimes](https://auto.economictimes.indiatimes.com/news/industry/electricpe-free-charger-management-software-to-support-manufacturers-operators/97765020)

**Acceptance criteria**

- At least one pilot site where:
  - HEMS publishes site power limit and current load to a partner CSMS via API.  
  - CSMS adjusts charger power based on HEMS data, keeping total demand below configured limit for a 1‑week test.  

**F8. OCPP/CSMS integration (future)**

**Acceptance criteria**

- For at least one OCPP‑compliant charger:
  - HEMS (or attached service) can connect to it, receive meter values, and send charging profile / remote stop commands in a test environment.  

### 3.4 Predictive maintenance and remote debugging

**F9. Anomaly detection**

- Detect abnormal signatures (standby drift, phase imbalance, motor overload). [cmti.res](https://cmti.res.in/wp-content/uploads/2025/03/SMmodsolCMTI-Jan2025.pdf)

**Acceptance criteria**

- On curated test datasets (simulated or real):
  - At least 80 % of injected anomalies (e.g., 20 % standby rise, 15 % phase imbalance, 25 % current increase on a motor) raise alerts within 24 h.  
  - False‑positive rate <10 % over 1‑month normal dataset.  

**F10. Remote diagnostics**

- Support and installer views with live status, logs, remote config, OTA. [niot](https://niot.in)

**Acceptance criteria**

- Device can be:
  - Remotely rebooted.  
  - Remotely updated with new firmware, with success confirmation and rollback on failure.  
- Support UI shows:
  - Last heartbeat time.  
  - Firmware version.  
  - Connectivity medium (Wi‑Fi/Ethernet/LTE).  
  - Last 100 key events (config change, OTA, reboot, fault).  

***

## 4. Non‑functional & hardware requirements

### 4.1 Electrical and mechanical

**H1. Installation**

- Split‑core CTs where possible; safe voltage taps; proper terminals. [desertcart](https://www.desertcart.in/products/288876350-ct-clamp-energy-monitor-clamp-home-energy-monitor-current-sensors)

**Acceptance criteria**

- Installation of CTs on typical house DB (3‑phase) can be done without disconnecting mains cables, by a trained electrician, within 60–90 minutes.  
- All live‑accessible parts meet finger‑safe (IP2X) requirements when enclosure is closed.  

**H2. Accuracy and performance**

- Class 1 target for active energy and power. [indem](https://indem.in)

**Acceptance criteria**

- See F1 AC – same tests apply.  
- Additional:
  - Frequency reading within ±0.1 Hz from 48–52 Hz.  
  - Voltage reading within ±1.5 % from 180–260 V.  

**H3. Enclosure and environment**

- Indoor IP20–IP30; industrial variant IP54+. [enerman](https://enerman.in)

**Acceptance criteria**

- Enclosure passes:
  - IP test for declared rating (independent lab or in‑house test with documented method).  
  - Thermal test: at 40 °C ambient and 80 % RH, device operates within spec (no derating or shutdown) at full rated current for 24 h.  

### 4.2 Connectivity and security

**N1. Connectivity options**

- Wi‑Fi + Ethernet baseline; LTE/4G optional. [iotdashboard](https://iotdashboard.in)

**Acceptance criteria**

- Device successfully connects and sends data via:
  - Wi‑Fi to home router.  
  - Ethernet to router.  
  - LTE (where module present) via SIM.  
- Connectivity failover:
  - If primary link fails, device retries and, if configured, switches to backup; outage period <5 min under test.  

**N2. Protocols and APIs**

- MQTT over TLS / HTTPS upstream; Modbus/REST locally. [gridx](https://www.gridx.ai/use-cases/home-energy-management-system)

**Acceptance criteria**

- MQTT connection uses TLS 1.2+ with certificate verification; connection fails if certificate is invalid.  
- REST/Modbus interface lets a test client:
  - Read at least: V, I, P, kWh, PF per channel.  
  - Change basic config (e.g., measurement interval) with authentication.  

**N3. Cybersecurity**

- Secure identity, signed firmware, secure boot. [v0-vidyuta.vercel](https://v0-vidyuta.vercel.app)

**Acceptance criteria**

- Each device has unique cryptographic identity; attempts to load firmware not signed by trusted key fail.  
- Pen‑test (internal or external) reports no critical or high‑severity vulnerabilities in exposed device interfaces at launch.  

***

## 5. Cloud, app, and data requirements

### 5.1 Cloud platform

**C1. Multi‑tenant architecture**

- Single backend for all segments; role‑based access. [akraniq](https://akraniq.com/solutions/industrial-iot)

**Acceptance criteria**

- Tenant isolation test: data from Site A is not accessible to users of Site B under any tested role.  
- Roles implemented: Owner, Installer, Support, EPC Admin, CPO Admin; each has a documented permissions matrix and passes manual test.  

**C2. Analytics**

- Bills, solar analytics, industrial KPIs. [enerman](https://www.enerman.in/post/iot-scada-eti-sol-for-solar-pv-plants-rooftops-monitoring-control-and-analysis)

**Acceptance criteria**

- At least one Indian tariff (slab + ToD) is modeled correctly; QA verifies sample bills from raw data.  
- For solar sites, daily plant generation on dashboard matches inverter total within ±2 % over test period.  
- For an industrial pilot, dashboard shows:
  - kWh per machine.  
  - One derived KPI (e.g., kWh per produced unit) based on integrated external data.  

### 5.2 Mobile and web UX

**C3. Apps**

- Real‑time flow dashboard, historical charts, exports, alert centre. [play.google](https://play.google.com/store/apps/details?id=com.oorjan.solar_monitoring&hl=en)

**Acceptance criteria**

- iOS and Android builds pass:
  - Real‑time view refresh under 10 s latency from last data point in >95 % of cases (good connectivity).  
  - User can export at least 3 months of 15‑min data to CSV from app or web.  
- Alert centre shows:
  - All anomalies from F9.  
  - Device faults (connectivity loss, OTA failure).  

**C4. Education layer**

- What‑if simulations and tips around solar/battery and usage. [inc42](https://inc42.com/buzz/meet-the-12-startups-part-of-panasonics-accelerator-programme/)

**Acceptance criteria**

- “Add solar” simulation: user picks plant size and tariff; system outputs:
  - Estimated annual generation.  
  - Estimated self‑consumption.  
  - Approximate payback period, based on last 6–12 months of usage.  
- At least 5 localized “tip” templates exist (e.g., AC, water heater, pumps), triggered by specific patterns in Indian homes.  

***

## 6. Competitor feature comparison (for reference)

(As before; kept for context, no extra AC – used mainly as positioning input.)  

***

## 7. Cost and pricing targets (initial hypotheses)

### 7.1 Hardware price bands

- Residential HEMS:
  - Entry: ₹8k–₹12k.  
  - Multi‑CT: ₹12k–₹18k. [eshop.se](https://eshop.se.com/in/shop-by-category/easy-homes.html)
- Small commercial: ₹20k–₹35k.  
- Industrial: likely ₹50k+ per site depending on CTs and gateways (project‑based). [enerman](https://www.enerman.in/post/iot-scada-eti-sol-for-solar-pv-plants-rooftops-monitoring-control-and-analysis)

**Acceptance criteria**

- Initial BoM plus expected manufacturing and channel margins allow selling:
  - Entry home SKU at ≤ upper bound of the target range while meeting margin targets.  
- Competitive analysis confirms:
  - Residential SKUs undercut Schneider Wiser energy monitor and stay competitive vs Emporia and SONOFF bundles for similar functionality. [indiamart](https://www.indiamart.com/proddetail/emporia-3-phase-smart-electricity-monitor-23366067688.html)

### 7.2 Subscription / SaaS (optional)

**Acceptance criteria**

- At least two tiers:
  - Free/basic (core monitoring for N devices).  
  - Paid (advanced analytics, industrial/EV features) with clear value differentiation.  
- Pricing model approved by product + finance; trials tested with at least 10 live customers.  

***

## 8. Competitor feature comparison (for reference)

### 8.1 Key competitors covered

- Emporia Vue / Indem (India distributor). [indiamart](https://www.indiamart.com/proddetail/emporia-3-phase-smart-electricity-monitor-23366067688.html)
- SONOFF POWCT and stackable power meters. [sonoff](https://sonoff.tech/products/sonoff-pow-ring-smart-power-meter-powct)
- Picostone Basic and related home‑automation modules. [indiamart](https://www.indiamart.com/proddetail/picostone-basic-21367430448.html)
- Schneider Electric Wiser energy‑monitoring device & eco‑system. [eshop.se](https://eshop.se.com/in/shop-by-category/easy-homes.html)
- EnerMAN ETi‑SOL / IoT SCADA for solar plants. [enerman](https://enerman.in)

### 8.2 High‑level comparison

| Aspect | Emporia Vue (Indem) | SONOFF POWCT / SPM | Picostone Basic | Schneider Wiser (Energy Device) | EnerMAN ETi‑SOL / SCADA |
|---|---|---|---|---|---|
| Target segment | Home, small C&I [indem](https://indem.in) | Home, small commercial, small boards [shop.sonoff](https://www.shop.sonoff.in/product-category/computer-accessories/energy-saving/) | Residential smart‑home [indiamart](https://www.indiamart.com/proddetail/picostone-basic-21367430448.html) | Premium residential and small commercial [eshop.se](https://eshop.se.com/in/shop-by-category/easy-homes.html) | Utility‑scale & C&I solar plants [enerman](https://enerman.in) |
| Phases & current | 1‑φ & 3‑φ, up to 200 A mains [indem](https://indem.in) | 1‑φ up to 100 A (POWCT); 20 A per channel (SPM‑4Relay) [sonoff](https://sonoff.tech/products/sonoff-pow-ring-smart-power-meter-powct) | 1‑φ; 4 loads (typ. 6–16 A) [indiamart](https://www.indiamart.com/proddetail/picostone-basic-21367430448.html) | 1‑φ device; ecosystem spans more [eshop.se](https://eshop.se.com/in/shop-by-category/easy-homes.html) | Multi‑φ, high‑current (MW‑scale) [enerman](https://enerman.in) |
| Channels/granularity | 3 main + up to 16 branch CTs [indem](https://indem.in) | 1 CT per POWCT; SPM‑Main + up to 32×4‑relay modules [shop.sonoff](https://www.shop.sonoff.in/product-category/computer-accessories/energy-saving/) | 4 control channels, limited metering [indiamart](https://www.indiamart.com/proddetail/picostone-basic-21367430448.html) | 1 energy‑monitor device, plus many automation modules [eshop.se](https://eshop.se.com/in/shop-by-category/easy-homes.html) | Many meters feeding gateway/SCADA [enerman](https://enerman.in) |
| Connectivity | Wi‑Fi, cloud [indem](https://indem.in) | Wi‑Fi (POWCT), RS‑485 + Wi‑Fi (SPM) [shop.sonoff](https://www.shop.sonoff.in/product-category/computer-accessories/energy-saving/) | Wi‑Fi [indiamart](https://www.indiamart.com/proddetail/picostone-basic-21367430448.html) | Wi‑Fi/Zigbee via Wiser hub [eshop.se](https://eshop.se.com/in/shop-by-category/easy-homes.html) | RS‑485, Ethernet, 4G, Wi‑Fi [enerman](https://enerman.in) |
| Solar support | Bi‑directional CTs, net‑solar bundles [indem](https://indem.in) | Basic export/solar capable, but no full PV EMS [sonoff](https://sonoff.tech/products/sonoff-pow-ring-smart-power-meter-powct) | None specific [indiamart](https://www.indiamart.com/proddetail/picostone-basic-21367430448.html) | Possible as part of home automation; not a dedicated PV EMS [se](https://www.se.com/in/en/residential/save/) | Full PV monitoring, control and analytics [enerman](https://enerman.in) |
| Industrial/EV | Light C&I only; no EV CMS [indem](https://indem.in) | Some C&I sub‑metering; no EV CMS [shop.sonoff](https://www.shop.sonoff.in/product-category/computer-accessories/energy-saving/) | None [indiamart](https://www.indiamart.com/proddetail/picostone-basic-21367430448.html) | Residential‑centric; industrial via other Schneider offerings [se](https://www.se.com/in/en/residential/save/) | C&I and utility; not home HEMS, EV not explicit [enerman](https://enerman.in) |
| Indicative device prices (India) | ~₹19,000 per 3‑phase kit [indiamart](https://www.indiamart.com/proddetail/emporia-3-phase-smart-electricity-monitor-23366067688.html) | POWCT ≈₹4,576; SPM‑Main ≈₹3,487; SPM‑4Relay ≈₹5,450 [shop.sonoff](https://www.shop.sonoff.in/product-category/computer-accessories/energy-saving/) | ≈₹7,800 per module (range 4,999–12,999) [indiamart](https://www.indiamart.com/proddetail/picostone-basic-21367430448.html) | Single‑phase energy device ~₹23,200–₹29,000 [eshop.se](https://eshop.se.com/in/shop-by-category/easy-homes.html) | Project‑based, no list price [enerman](https://www.enerman.in/post/iot-scada-eti-sol-for-solar-pv-plants-rooftops-monitoring-control-and-analysis) |

Your HEMS should **outperform Emporia and SONOFF** in integration breadth (solar + battery + EV + industrial) while remaining significantly more affordable than Wiser for mass Indian deployment. [shop.sonoff](https://www.shop.sonoff.in/product-category/computer-accessories/energy-saving/)

***

## 9. Cost and pricing targets (initial hypotheses)

These are **market‑based targets** informed by competitor retail prices (not BOM), to be refined once BoM is defined.

### 9.1 Hardware price bands

- **Residential HEMS (baseline CT‑monitor + Wi‑Fi + app)**  
  - Market reference: SONOFF single‑circuit ~₹4.5k, Emporia 3‑phase CT kit ~₹19k, Wiser energy device ~₹23–29k. [eshop.se](https://eshop.se.com/in/shop-by-category/easy-homes.html)
  - Target retail (India):  
    - Entry home HEMS (main CTs only): **₹8k–₹12k**.  
    - Home HEMS with multi‑CT and better analytics: **₹12k–₹18k** (aim to sit between Emporia and Wiser). [indiamart](https://www.indiamart.com/proddetail/emporia-3-phase-smart-electricity-monitor-23366067688.html)

- **Small commercial / MSME variant**  
  - More CT channels, optional LTE, industrial‑grade enclosure.  
  - Target retail: **₹20k–₹35k** depending on CT count and connectivity, still below typical industrial SCADA offerings. [iotdashboard](https://iotdashboard.in)

- **Industrial / solar‑plant gateway variant**  
  - Competes with ETi‑SOL‑class hardware, which is sold project‑wise without list prices. [enerman](https://www.enerman.in/post/iot-scada-eti-sol-for-solar-pv-plants-rooftops-monitoring-control-and-analysis)
  - Target: price per site comparable to a typical industrial IoT gateway plus some CTs, set via pilots and margin expectations (likely **₹50k+** per site, depending on scale).  
