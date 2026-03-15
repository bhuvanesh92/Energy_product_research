# India EV Charging Infrastructure: Market Analysis & Investment Thesis

**Version:** 2.0 — VC-Ready  
**Updated:** 2026-03-15  
**Data Sources:** VAHAN, Ministry of Power (MoP), BEE/EESL, CEEW, ICCT, IEEFA, MNRE Benchmark 2025, State DISCOM tariff orders (2023–2025), public operator data

> **How to read this report:** All market sizing and unit economics use a three-scenario framework — **Conservative, Base, Aggressive** — so assumptions are explicit and defensible. Percentages marked *(I)* are *illustrative estimates based on expert judgement and secondary research*; they are not directly measured national statistics and should be treated as working hypotheses for modelling.

---

# EXECUTIVE SUMMARY

| # | Headline Finding |
|---|------------------|
| 1 | India's EV fleet has grown from **237K (2020) to ~4.5–5.5M (2025)** — a 20–25x expansion in five years *(VAHAN/FADA)* |
| 2 | Public charging infrastructure lags badly: **~25,000 installed / ~12,000+ operational stations** as of 2025 against a structural need of **60,000–80,000** |
| 3 | The infra gap is **not uniform** — 2-wheelers (62% of fleet) mostly charge at home; the real crunch is for **3W commercial, fleet 4W, and commercial trucks** |
| 4 | A fully-ramped public station (2×DC 60kW + 2×AC 22kW) generates **Rs. 3.5–10 L/month gross revenue** (Conservative–Aggressive) at gross margins of **50–65%** before rent and O&M |
| 5 | Adding solar (40% offset) improves net margin by ~7pp; solar + battery by ~11pp — with payback of **7.6–10 years** depending on state tariff |
| 6 | **No domestic integrated product** (battery + bi-directional inverter + DC charger, 100–500 kWh) exists at scale; buying components separately costs **Rs. 47–169 L** per unit |
| 7 | An integrated unit priced at a **27% discount to bill-of-materials** targets a **Rs. 250+ Cr revenue opportunity at 5% market share by 2030** |

---

# PART 1: DEMAND STORY

## 1.1 India's EV Fleet — What the Numbers Actually Show

### EV Registrations 2020–2026 *(VAHAN/FADA)*

| Year | Total EVs | YoY Growth | 2W | 3W | 4W | E-Buses | E-Trucks |
|------|-----------|------------|-----|-----|-----|---------|----------|
| 2020 | 237K | — | 152K | 59K | 25K | 802 | ~500 |
| 2021 | 430K | +82% | 287K | 98K | 44K | 1,121 | ~900 |
| 2022 | 1.00M | +133% | 683K | 214K | 103K | 1,987 | ~1,500 |
| 2023 | 1.83M | +83% | 1,214K | 398K | 219K | 3,623 | ~3,000 |
| 2024 | 2.85M | +55% | 1,847K | 612K | 382K | 6,500 | ~7,000 |
| 2025 | ~4.57M* | ~60% | ~2,900K | ~920K | ~680K | ~15K | ~13K |
| 2026E | ~5.5M | ~20% | ~3,400K | ~1,100K | ~850K | ~25K | ~20K |

*2025 total is VAHAN-based mid-year figure extrapolated; treat as Base estimate. 2026E is a forward projection.*

**Growth reality check:** EV sales hit **~1.9M units in 2024** and **~2.3M in 2025** *(ICCT/FADA)*. Cumulative stock above is directionally correct, though precise annualised stock depends on scrapping rates which are currently near zero for EVs.

---

## 1.2 Charging Infrastructure: The Real Gap

### What the Data Actually Shows *(MoP/BEE/EESL releases 2024–2025)*

| Metric | Original Report | Vetted (2025) | Source |
|--------|----------------|---------------|--------|
| Public stations (2025) | 2,738 | **~25,000 installed; ~12,000–15,000 operational** | MoP Feb 2024 press release; EESL data |
| EV:charger ratio (2025) | 1,401:1 | **~180:1–380:1** depending on metric | Calculated from above |
| DC charger share | Not stated | **~30–35% of public chargers are DC** | Industry reports |
| Stations needed (2025) | 74,012 | **~45,000–80,000** (conservative–aggressive) | Scenario-modelled |

**Key nuance:** Even at 25,000 stations, India is at **3–6x the 'comfort' EV:charger ratio of ~50:1** — the infra gap thesis is real, but the starting point is materially higher than the original report assumed.

### Infrastructure Gap: Scenario View

| Metric (End-2025) | Conservative | Base | Aggressive |
|-------------------|-------------|------|------------|
| EVs in fleet | 4.5M | 5.0M | 5.5M |
| Operational public stations | 12,000 | 15,000 | 20,000 |
| EV : charger ratio | 375:1 | 333:1 | 275:1 |
| "Comfort" target ratio | 80:1 | 60:1 | 50:1 |
| Additional stations needed | ~44,000 | ~68,000 | ~90,000 |

---

## 1.3 State-wise Distribution *(VAHAN estimates; charger counts are approximate)*

| Rank | State | EVs (2025E) | % of India | Est. Public Stations | Indicative Gap |
|------|-------|-------------|------------|---------------------|----------------|
| 1 | Maharashtra | ~570K | ~12.5% | ~2,000–2,500 | ~9,000–11,000 |
| 2 | Uttar Pradesh | ~520K | ~11.4% | ~600–900 | ~9,000–10,000 |
| 3 | Karnataka | ~500K | ~11.0% | ~1,200–1,600 | ~8,000–9,000 |
| 4 | Delhi | ~420K | ~9.2% | ~1,400–1,800 | ~7,000–8,000 |
| 5 | Tamil Nadu | ~415K | ~9.1% | ~1,000–1,300 | ~7,000–8,000 |
| 6 | Telangana | ~350K | ~7.7% | ~700–1,000 | ~6,000–7,000 |
| 7 | Gujarat | ~310K | ~6.8% | ~700–1,000 | ~5,000–6,000 |
| 8 | Kerala | ~195K | ~4.3% | ~600–800 | ~3,000–4,000 |
| 9 | Rajasthan | ~165K | ~3.6% | ~300–450 | ~3,000–3,500 |
| 10 | Haryana | ~130K | ~2.9% | ~350–500 | ~2,000–2,500 |

*State-level charger counts are approximate; official BEE/EESL data does not publish real-time state breakdowns consistently.*

---

## 1.4 Segment-by-Segment: Who Actually Needs Public Charging?

This is the most important analytical lens for the investment thesis. **Not all EVs create equal public-charging demand.** The segment hierarchy below determines where infra investment is most constrained — and therefore where revenue is most defensible.

### Charging Dependency Framework

> Segments are classified by: (a) share of energy from public/semi-public infra, (b) whether missing public infra causes income loss (commercial) or inconvenience (private), and (c) whether the constraint is public chargers, depot capacity, or home/residential access.

| Segment | Fleet (2025E) | Home / Depot share *(I)* | Public AC share *(I)* | Public DC share *(I)* | Underserved by public infra? | Primary constraint |
|---------|---------------|--------------------------|----------------------|----------------------|------------------------------|--------------------|
| **2W – private** | ~2,600K | 75–85% | 5–10% | <5% | **No** — home-charging dominant | Residential wiring, RWA access |
| **2W – commercial delivery** | ~300K | 40–55% | 5% | 5% | **Swap-network gaps** | Swap station density on delivery routes |
| **3W – commercial (e-rick/auto)** | ~920K | 10–30% | 50–65% (AC cluster) | 5–10% | **Yes — structurally** | AC cluster hubs at stands/markets |
| **4W – private (with parking)** | ~450K | 60–75% | 10–15% | 15–25% | **Partially** — highway DC gaps | Highway DC corridors |
| **4W – private (apartment/no parking)** | ~180K | 10–25% | 30–40% | 30–40% | **Yes** | Urban public AC + DC density |
| **4W – fleet / ride-hailing** | ~50K | 20–35% | 20–25% | 40–55% | **Yes** | Depot DC + urban hub availability |
| **E-buses** | ~15K | 90–100% (depot) | ~0% | <5% (opportunity) | **No (public); Yes (depot grid)** | Depot MW-scale grid infra |
| **E-trucks / LCVs** | ~13K | 50–65% (depot/hub) | 5% | 30–45% | **Yes — highway/hub DC** | 150–350kW highway & logistics hub DC |

*(I) = Illustrative shares based on secondary research (CEEW 3W studies, TERI/IEEFA bus reports, operator data), expert judgement. Not directly measured national statistics.*

### Segment Detail

#### 2-Wheelers

| Spec | Entry | Mid | Premium |
|------|-------|-----|---------|
| Battery | 2.0–2.5 kWh | 2.9–3.5 kWh | 4.0–5.2 kWh |
| Range | 80–100 km | 120–150 km | 150–200 km |
| Charge time | 4–5 hrs (5A) | 3–4 hrs | 2–3 hrs |

**Investment implication:** Public charging capex for 2W is low-priority. The real gap is **battery swap network density** for commercial fleets (Zomato/Swiggy/Amazon Last Mile), and **RWA/housing board regulations** for home charging access.

#### 3-Wheelers (e-Rickshaws and e-Autos)

| Spec | E-Rickshaw | E-Auto | E-Cargo 3W |
|------|-----------|--------|------------|
| Battery | 3.5–5 kWh | 7–10 kWh | 10–15 kWh |
| Daily km | 60–80 | 100–150 | 80–120 |
| Charge pattern | 1–2x/day at stand | 1x overnight + 1 midday | 1x overnight |
| Primary infra need | AC 3.3–7 kW cluster | AC 7 kW cluster | AC 7–22 kW depot |

**Investment implication:** The binding constraint is **cluster AC hubs at auto stands and market clusters** — not highway DC. This is a **high-density, low-power** infra play, most relevant in UP, Bihar, West Bengal, Delhi where e-rick density is highest.

#### 4-Wheelers (Private + Fleet)

| Spec | Entry (Tiago EV) | Mid (Nexon EV) | Premium (Ioniq 5) |
|------|-----------------|----------------|-------------------|
| Battery | 24–30 kWh | 40–50 kWh | 60–80 kWh |
| Range | 250–315 km | 350–450 km | 450–600 km |
| DC fast charge (10–80%) | ~56 min @ 50kW | 30–45 min @ 50–120kW | 18–30 min @ 150–350kW |

**Investment implication — split by user type:**
- **Private with home parking:** public charging is supplemental; **highway DC corridors** (every 50–100 km on NH) are the critical gap.
- **Apartment dwellers + fleet operators:** public charging is primary; need **urban DC hubs** in high-density residential zones and fleet depots.

#### E-Buses

| Spec | City 9m | City 12m | Intercity |
|------|---------|----------|-----------|
| Battery | 150–200 kWh | 250–350 kWh | 350–450 kWh |
| Daily route | 150–200 km | 180–250 km | 400–600 km |
| Depot charge power | 60–120 kW DC | 120–240 kW DC | 150–350 kW DC |

**Investment implication:** The market here is **depot-scale energy infrastructure** (MW-level grid connections, solar + storage at depots), not retail public chargers. Underserved by commercial public charging networks but addressable via B2B depot packages.

#### E-Trucks and LCVs

| Spec | Light (<2T) | Medium (2–7T) | Heavy (>7T) |
|------|------------|--------------|-------------|
| Battery | 20–40 kWh | 60–150 kWh | 200–400 kWh |
| Daily km | 80–120 | 150–300 | 300–500 |
| Primary infra | Warehouse AC | Depot DC 60kW | Highway DC 150kW+ |

**Investment implication:** The commercial vehicle segment is **chronically underserved** on highway corridors and logistics hubs — and it is a high-willingness-to-pay segment because charging downtime = direct revenue loss. Priority: **150–350 kW DC at highway truck stops and logistics parks**.

### Recommended Station Configurations by Use Case

| Station Type | Target Segments | Configuration | Daily Sessions (Base) |
|--------------|-----------------|---------------|-----------------------|
| Urban Hub | 4W + 3W | 2× DC 60kW + 4× AC 22kW + 4× AC 7kW | 120–180 |
| Highway Corridor | 4W + Trucks | 4× DC 120kW + 2× DC 60kW | 80–140 |
| Auto Stand Cluster | 3W focused | 1× DC 30kW + 6× AC 7kW | 70–110 |
| Mall / Destination | Private 4W | 2× DC 60kW + 6× AC 22kW | 50–90 |
| Bus Depot | E-buses | 4× DC 150kW + 2× Pantograph | 30–50 |
| Logistics Hub | Trucks + LCVs | 2× DC 150kW + 4× DC 60kW | 40–70 |

---

# PART 2: REVENUE MODEL

## 2.1 Tariff Reality Check

> The original report used DC selling prices of Rs. 17–20/kWh and grid buy rates of Rs. 2.25–3.5/kWh. Vetted state tariff orders (2023–2025) show a different picture.

### Actual EV Tariff Ranges by State *(State ERC orders, DISCOM EV categories, 2023–2025)*

| State | C&I / Dedicated EV buy rate (Rs/kWh) | Typical public DC sell price (Rs/kWh) | Typical public AC sell price (Rs/kWh) | Gross spread (mid DC) |
|-------|--------------------------------------|---------------------------------------|---------------------------------------|-----------------------|
| Karnataka | 5.0–7.0 | 10–18 | 7–14 | ~8–10 |
| Maharashtra | 5.0–7.5 | 10–17 | 7–13 | ~7–9 |
| Delhi | 4.5–6.5 | 9–16 | 6–12 | ~6–8 |
| Gujarat | 4.5–6.5 | 9–16 | 6–12 | ~6–8 |
| Tamil Nadu | 4.0–6.0 | 8–14 | 6–11 | ~5–7 |
| Telangana | 6.0–8.0 | 10–16 | 7–12 | ~5–7 |
| Uttar Pradesh | 4.5–6.5 | 9–15 | 6–11 | ~5–8 |
| Kerala | 4.5–6.5 | 9–15 | 7–12 | ~5–7 |

**Why this matters for the model:** The original report's gross margins of 80–87% assumed DC sell prices of Rs. 17–20 and buy rates of Rs. 2.25–3.5 — an implied spread of Rs. 14–18/kWh. The realistic mid-case spread is **Rs. 6–9/kWh**, producing gross margins closer to **50–65%** rather than 80–87%.

### Scenario Price Assumptions (used in all P&L models below)

| Scenario | Blended DC sell price | Blended AC sell price | Effective grid buy rate |
|----------|-----------------------|-----------------------|-------------------------|
| Conservative | Rs. 11.5/kWh | Rs. 8.0/kWh | Rs. 6.0/kWh equiv. |
| Base | Rs. 13.5/kWh | Rs. 9.5/kWh | Rs. 6.0/kWh equiv. |
| Aggressive | Rs. 15.5/kWh | Rs. 11.0/kWh | Rs. 5.5/kWh equiv. |

*Aggressive aligns with the original report's assumptions and is only realistic for select hub locations with volume contracts and dynamic pricing.*

---

## 2.2 Station Unit Economics

### Charger Throughput Assumptions

| Charger | Session length | Realistic sessions/day | kWh/session |
|---------|---------------|----------------------|-------------|
| DC 60kW | 30 min | 30–40 | 20–24 kWh |
| DC 120kW | 20 min | 40–55 | 28–35 kWh |
| AC 22kW | 3 hrs | 4–6 | 30–40 kWh |
| AC 7kW | 5–6 hrs | 2–3 | 28–35 kWh |

### Standard Station: Daily Throughput Scenarios

**Configuration: 2× DC 60kW + 2× AC 22kW**

| Metric | Conservative | Base | Aggressive |
|--------|-------------|------|------------|
| DC sessions/day | 40 | 60 | 80 |
| AC sessions/day | 6 | 10 | 14 |
| kWh/day (DC) | 840 | 1,320 | 1,800 |
| kWh/day (AC) | 180 | 330 | 490 |
| **Total kWh/day** | **1,020** | **1,650** | **2,290** |

### Monthly P&L: Three Scenarios (Blended Cross-State)

| Line Item | Conservative | Base | Aggressive |
|-----------|-------------|------|------------|
| Energy sold (kWh/month) | 30,600 | 49,500 | 68,700 |
| DC revenue | Rs. 2.76 L | Rs. 5.34 L | Rs. 8.37 L |
| AC revenue | Rs. 0.72 L | Rs. 1.26 L | Rs. 1.93 L |
| **Total Revenue** | **Rs. 3.48 L** | **Rs. 6.60 L** | **Rs. 10.30 L** |
| Grid power cost | Rs. 1.84 L | Rs. 2.97 L | Rs. 3.78 L |
| **Gross Profit** | **Rs. 1.64 L** | **Rs. 3.63 L** | **Rs. 6.52 L** |
| **Gross Margin** | **~47%** | **~55%** | **~63%** |

> **Note:** Gross profit is before rent, O&M (~Rs. 0.5–1.0 L/month), depreciation, and working capital. The original report's Rs. 9–10 L/month at 80%+ gross margin is re-classified as **Aggressive + low-tariff state benchmark** — achievable at fully ramped highway hubs in Tamil Nadu or Goa, not a typical current-market figure.

### State-wise Revenue Variance (Base Scenario)

| State | Grid buy rate | Monthly Revenue | Power cost | Gross Margin |
|-------|--------------|-----------------|------------|---------------|
| Karnataka | Rs. 6.0/kWh | Rs. 6.60 L | Rs. 2.97 L | ~55% |
| Maharashtra | Rs. 6.0/kWh | Rs. 6.60 L | Rs. 2.97 L | ~55% |
| Tamil Nadu | Rs. 5.0/kWh | Rs. 6.00 L | Rs. 2.48 L | ~59% |
| Delhi | Rs. 5.5/kWh | Rs. 6.30 L | Rs. 2.72 L | ~57% |
| Telangana | Rs. 7.0/kWh | Rs. 6.60 L | Rs. 3.47 L | ~47% |
| Gujarat | Rs. 5.5/kWh | Rs. 6.30 L | Rs. 2.72 L | ~57% |

---

## 2.3 National Market Size: Scenario View

> The original report cited Rs. 3,153 Cr as current market size. This figure used both understated charger counts and overstated per-kWh revenues. A scenario-based rebuild gives:

| Metric | Conservative | Base | Aggressive |
|--------|-------------|------|------------|
| Operational public stations | 12,000 | 15,000 | 20,000 |
| Avg energy sold/station/day | 500 kWh | 900 kWh | 1,300 kWh |
| Total daily MWh (public) | 6,000 | 13,500 | 26,000 |
| Avg realised revenue (Rs/kWh) | Rs. 11.5 | Rs. 13.0 | Rs. 14.5 |
| **Yearly public charging revenue** | **~Rs. 252 Cr** | **~Rs. 641 Cr** | **~Rs. 1,376 Cr** |

**Scope note:** These figures cover **public charging only**. Including semi-public (workplace, fleet depot), captive (private home), and private fleet charging, the **extended TAM is Rs. 2,000–6,000+ Cr** in 2025 — consistent with the original report's Rs. 3,153–6,300 Cr range when all segments are included.

---

# PART 3: SOLAR COST OFFSET

## 3.1 Why Solar Makes Sense for EV Stations

| Factor | Data point | Source |
|--------|------------|--------|
| Solar capex (2025) | Rs. 42,000–48,000/kWp (C&I rooftop) | MNRE Benchmark 2025 |
| Generation (pan-India average) | 3.5–5.0 kWh/kWp/day | MNRE/NIWE |
| Grid parity | Solar LCOE < grid tariff in all major states | IEEFA 2024 |
| C&I rooftop payback | 5–10 years depending on state tariff | CEEW, IEEFA |

### Regional Solar Generation *(MNRE/NIWE)*

| Region | States | kWh/kWp/day (avg) | Best Months |
|--------|--------|-------------------|-------------|
| Northwest | Rajasthan, Gujarat | 4.5–5.0 | Mar–Jun |
| West | Maharashtra, Goa | 4.0–4.5 | Feb–May |
| South | TN, Karnataka, Kerala, AP | 4.5–5.0 | Feb–May |
| North | Delhi, Haryana, Punjab, UP | 4.0–4.5 | Mar–Jun |
| East | WB, Odisha, Bihar | 3.5–4.0 | Mar–May |

---

## 3.2 Solar Sizing: The 40% Offset Logic

> **Assumptions:** Rs. 45,000/kWp capex | 4.0 kWh/kWp/day | Standard station: 1,650 kWh/day (Base)

| Offset Level | Solar Size | Area | Capex | Annual Savings (Base) | Payback |
|--------------|------------|------|-------|-----------------------|---------|
| 20% | 93 kWp | 466 sqm | Rs. 41.9 L | Rs. 5.9 L | ~7.1 yrs |
| **40%** | **186 kWp** | **931 sqm** | **Rs. 83.8 L** | **Rs. 11.9 L** | **~7.0 yrs** |
| 60% | 279 kWp | 1,396 sqm | Rs. 125.7 L | Rs. 17.8 L | ~7.1 yrs |
| 80% | 372 kWp | 1,862 sqm | Rs. 167.6 L | Rs. 23.7 L | ~7.1 yrs |

**40% is operationally optimal** — higher offsets need proportionally more roof/land area with no change in payback ratio, but land scarcity makes >40% impractical at most urban/highway locations.

### Solar Payback by State (Base tariffs, 40% offset)

| State | Grid buy rate | Annual solar savings | Solar capex | Payback |
|-------|--------------|---------------------|-------------|----------|
| Karnataka | Rs. 6.0/kWh | Rs. 12.9 L | Rs. 83.8 L | **6.5 yrs** |
| Maharashtra | Rs. 6.0/kWh | Rs. 12.9 L | Rs. 83.8 L | **6.5 yrs** |
| Tamil Nadu | Rs. 5.0/kWh | Rs. 10.7 L | Rs. 83.8 L | **7.8 yrs** |
| Delhi | Rs. 5.5/kWh | Rs. 11.8 L | Rs. 83.8 L | **7.1 yrs** |
| Telangana | Rs. 7.0/kWh | Rs. 15.1 L | Rs. 83.8 L | **5.6 yrs** |
| Gujarat | Rs. 5.5/kWh | Rs. 11.8 L | Rs. 83.8 L | **7.1 yrs** |

*Payback uses Base-scenario throughput and updated tariff ranges; figures will vary with actual utilisation.*

---

# PART 4: HYBRID SYSTEMS (SOLAR + BATTERY)

## 4.1 Why Add Battery Storage?

| Challenge | Battery Solution | Operational Benefit |
|-----------|-----------------|--------------------|
| Solar only generates during day | Store solar, dispatch at peak | 24/7 solar utilisation |
| Peak-hour tariffs (Time-of-Day) | Discharge battery at peak, charge at off-peak | Reduce effective buy rate |
| Grid instability / outages | Battery backup | Revenue continuity |
| Excess daytime solar | Store instead of export at low rates | Capture full solar value |

## 4.2 Hybrid Configurations

| Config | Solar | Battery | Total Capex | Grid Offset *(I)* | Payback (Base) |
|--------|-------|---------|-------------|-------------------|----------------|
| Solar only | 186 kWp | — | Rs. 83.8 L | ~40% | ~7.0 yrs |
| Solar + 250 kWh LFP | 186 kWp | 250 kWh | Rs. 124.5 L | ~60–65% | ~7.6 yrs |
| Solar + 500 kWh LFP | 186 kWp | 500 kWh | Rs. 165.3 L | ~65–72% | ~8.5–10 yrs |

> **Battery cost note:** LFP pack costs have been declining globally. At 2025 Indian C&I pricing (~Rs. 13,000/kWh all-in), these capex figures are reasonable; by 2027–28, costs may improve payback by 1–2 years. Refresh annually.

## 4.3 P&L Impact: Grid vs Solar vs Hybrid (Base Scenario, Blended State)

| Line Item | Grid Only | + Solar 40% | + Solar + 500kWh Battery |
|-----------|-----------|-------------|---------------------------|
| Monthly Revenue | Rs. 6.60 L | Rs. 6.60 L | Rs. 6.70 L |
| Grid power cost | Rs. 2.97 L | Rs. 1.78 L | Rs. 0.89 L |
| Battery degradation | — | — | Rs. 0.20 L |
| Solar O&M | — | Rs. 0.10 L | Rs. 0.15 L |
| **Gross Profit** | **Rs. 3.63 L** | **Rs. 4.72 L** | **Rs. 5.46 L** |
| **Gross Margin** | **~55%** | **~71%** | **~82%** |

---

# PART 5: THE PRODUCT OPPORTUNITY

## 5.1 Component Costs (2025–26)

### Solar Components (186 kWp, 40% offset)

| Component | Unit Cost | Total |
|-----------|-----------|-------|
| Solar panels (550W Mono PERC) | Rs. 18,000/kWp | Rs. 33.5 L |
| Mounting + civil | Rs. 8,000/kWp | Rs. 14.9 L |
| String inverter | Rs. 5,000/kWp | Rs. 9.3 L |
| Cables, protection, metering | Rs. 5,000/kWp | Rs. 9.3 L |
| Installation | Rs. 9,000/kWp | Rs. 16.7 L |
| **Solar Total** | **Rs. 45,000/kWp** | **Rs. 83.7 L** |

### Battery Components (500 kWh LFP)

| Component | Unit Cost | Total |
|-----------|-----------|-------|
| LFP cells (CATL/BYD Grade A) | Rs. 13,000/kWh | Rs. 65.0 L |
| Rack, housing (IP65) | Rs. 1,500/kWh | Rs. 7.5 L |
| BMS | Rs. 1,000/kWh | Rs. 5.0 L |
| Thermal management | Rs. 800/kWh | Rs. 4.0 L |
| **Battery Total** | **Rs. 16,300/kWh** | **Rs. 81.5 L** |

### Power Electronics

| Component | Spec | Cost |
|-----------|------|------|
| Bi-directional hybrid inverter | 200 kVA | Rs. 50.0 L |
| Grid-tie inverter (backup) | 100 kW | Rs. 4.0 L |
| Transformer | 250 kVA | Rs. 5.0 L |
| **Power Electronics Total** | | **Rs. 59.0 L** |

### EV Chargers

| Component | Spec | Cost |
|-----------|------|------|
| DC fast charger × 2 | 60 kW CCS2 | Rs. 30.0 L |
| AC charger × 2 | 22 kW Type 2 | Rs. 4.3 L |
| OCPP backend + payment | — | Rs. 3.0 L |
| **Charger Total** | | **Rs. 37.3 L** |

---

## 5.2 The Integration Opportunity

### OEM Landscape Reality Check

India has multiple domestic providers of standalone DC fast chargers and a few battery + charger bundles. However, **high-power bi-directional inverters and fully integrated "battery + inverter + DC charger" blocks in the 100–500 kWh / 60–240 kW range** are:

- Scarce in domestic supply; rely on imported power-electronics sub-systems
- Bought separately and integrated at site (adding 15–25% integration cost and complexity)
- Fragmented across 3–4 vendors per station with no single accountability

**Opportunity:** A domestically manufactured, modular, integrated unit that combines battery, bi-directional inverter, and DC charger — delivered as a single SKU with a single warranty.

### Bill of Materials vs Target Integrated Price

| Unit Size | BOM (separate) | Target integrated price | Saving to buyer | Gross margin target |
|-----------|---------------|------------------------|-----------------|---------------------|
| **Small** — 100kWh + 60kW + 50kVA | Rs. 46.9 L | Rs. 34.2 L | 27% | 25–30% |
| **Medium** — 250kWh + 60kW + 100kVA | Rs. 86.5 L | Rs. 63.1 L | 27% | 25–30% |
| **Large** — 500kWh + 120kW + 200kVA | Rs. 168.7 L | Rs. 123.2 L | 27% | 25–30% |
| **XL Fleet** — 1000kWh + 240kW + 400kVA | Rs. 310 L | Rs. 226 L | 27% | 25–30% |

### Build Priority

| Priority | Component | Rationale |
|----------|-----------|----------|
| **1** | Bi-directional inverter | All-imported today; highest margin; enables V2G future | 
| **2** | Full integrated unit | Bundles inverter + battery + charger; single SKU for CPOs |
| **3** | Smart BMS | Needed for warranty on integrated unit |
| **4** | Battery pack | Commoditising fast; build only if integration cost justifies |

---

## 5.3 Market Sizing & Revenue Roadmap

### Total Addressable Market for Integrated Units

| Metric | 2025 | 2028E | 2030E |
|--------|------|-------|-------|
| EVs in India | ~4.6M | ~9M | ~15M |
| Public + semi-public stations needed | ~45–80K | ~120K | ~300K |
| New stations/year (Base) | ~15,000 | ~30,000 | ~50,000 |
| Integrated units market (Rs.) | ~Rs. 750 Cr | ~Rs. 2,500 Cr | ~Rs. 5,000 Cr |
| **5% share target** | **Rs. 37.5 Cr** | **Rs. 125 Cr** | **Rs. 250 Cr** |

### Product Roadmap & Revenue Plan

| Phase | Year | Product | Unit Price | Volume Target | Revenue |
|-------|------|---------|------------|---------------|---------|
| 1 | 2026 | 100 kWh integrated | Rs. 34 L | 500 units | Rs. 171 Cr |
| 2 | 2027 | 250 kWh integrated | Rs. 63 L | 800 units | Rs. 505 Cr |
| 3 | 2028 | 500 kWh integrated | Rs. 123 L | 500 units | Rs. 616 Cr |
| 4 | 2029 | Fleet XL (1000 kWh) | Rs. 226 L | 300 units | Rs. 678 Cr |

*These revenue figures are top-line only; achieving them requires a full sales, manufacturing, and service organisation. They serve as a directional market-sizing exercise, not an operating plan.*

---

# PART 6: INVESTMENT THESIS SUMMARY

## Why Now

| Signal | Data |
|--------|------|
| Fleet growth | 4.57M EVs by 2025, sales of 2.3M/year and accelerating |
| Policy tailwind | FAME III expected; PM e-Bus Seva; state EV policies in 14+ states |
| Infrastructure lag | Even with 25K stations, EV:charger ratio is 3–6x the comfort target |
| No domestic integrated product | Rs. 47–169 L per station in separate components today |
| Declining input costs | Solar –44% since 2020; LFP battery costs declining further |

## The Case in One Paragraph

India's EV market has reached an inflection point — fleet size is large enough that the infrastructure gap is becoming a commercial pain point, not just a policy discussion. The most capital-efficient entry is not operating charging stations (low margin, real-estate-intensive) but **manufacturing the integrated hardware that station operators need**. No domestic vendor offers a modular battery + inverter + charger block; all components are sourced separately with 15–25% integration cost and fragmented accountability. A product that delivers a 27% total cost saving to operators, with a domestic manufacturing and service model, can capture a Rs. 250+ Cr revenue position by 2030 at a modest 5% market share.

## Key Risks & Mitigants

| Risk | Severity | Mitigant |
|------|----------|----------|
| Station utilisation lower than modelled | High | Anchor first customers in high-traffic highway/hub locations |
| Grid tariff changes erode solar ROI | Medium | Long-term DISCOM agreements; model sensitivity to ±1 Rs/kWh |
| Chinese integrated product imports | High | BIS certification requirements; PLI scheme for domestic mfg |
| Battery cost decline erodes margin | Medium | Design for modularity; upgrade battery sub-system independently |
| Capital intensity of manufacturing | Medium | Asset-light: outsource cell/panel; own BMS + inverter IP |

---

*Report v2.0 — Updated 2026-03-15*
*Sources: VAHAN, FADA, Ministry of Power, BEE, EESL, CEEW, ICCT, IEEFA, TERI, MNRE, State DISCOM tariff orders (2023–2025)*
*Scenarios marked (I) are illustrative estimates based on secondary research and expert judgement, not directly measured statistics.*
