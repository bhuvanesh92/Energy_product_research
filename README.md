# India Electricity Rate Calculator & EV Business Analysis Tool

A Python-based tool for querying electricity rates across Indian states, analyzing EV charging station economics, and generating comprehensive business reports for EV charging infrastructure.

## Features

- 🔌 **Electricity Rates**: Query real-time electricity rates for all Indian states
- ⚡ **EV Charging Rates**: DC fast charging and AC slow charging rates by state
- 📊 **Revenue Analysis**: Calculate EV charging station revenue potential
- ☀️ **Solar Economics**: Solar offset calculations and hybrid system analysis
- 📝 **Report Generation**: Auto-generate comprehensive markdown business reports
- 💾 **Smart Caching**: Local caching with configurable expiry (7 days default)

## Installation

```bash
# Clone or navigate to the project directory
cd Energy_product_research

# Install dependencies
pip install -r requirements.txt
```

## Quick Start

```bash
# Show all electricity and EV charging rates
python Query_web_electricity_rate.py

# Generate full business analysis report
python Query_web_electricity_rate.py --generate-report
```

## Command Reference

### Basic Commands

| Command | Description |
|---------|-------------|
| `python Query_web_electricity_rate.py` | Show all electricity & EV rates |
| `python Query_web_electricity_rate.py --status` | Show data source status |
| `python Query_web_electricity_rate.py -h` | Show help message |

### Report Generation

```bash
# Generate full EV revenue report (saves to ev_revenue_report.md)
python Query_web_electricity_rate.py --generate-report

# Generate report with custom output path
python Query_web_electricity_rate.py --generate-report --output my_report.md
python Query_web_electricity_rate.py -g -o /path/to/custom_report.md
```

### IPCU Cost Targets (Technical Document Integration)

```bash
# Generate IPCU cost targets as standalone file
python Query_web_electricity_rate.py --generate-ipcu-costs

# Update Section 7.0 in existing IPCU document
python Query_web_electricity_rate.py --generate-ipcu-costs --ipcu-file integrated_power_conversion_unit.md

# Force refresh data and update IPCU document
python Query_web_electricity_rate.py -f --generate-ipcu-costs --ipcu-file integrated_power_conversion_unit.md
```

> **Note:** The `--generate-ipcu-costs` option derives cost targets from the revenue model, ensuring the IPCU technical document stays in sync with business analysis.

### State-Specific Queries

```bash
# Get rate for a specific state
python Query_web_electricity_rate.py --state "Maharashtra"
python Query_web_electricity_rate.py -s "Karnataka"

# Calculate bill for specific units consumed
python Query_web_electricity_rate.py --state "Delhi" --calculate 500
python Query_web_electricity_rate.py -s "Tamil Nadu" -c 350
```

### EV Data Commands

```bash
# Show EV DC fast charging rates
python Query_web_electricity_rate.py --ev

# Show EV charging statistics summary
python Query_web_electricity_rate.py --ev-stats

# Show EV charging station revenue estimates by state
python Query_web_electricity_rate.py --ev-revenue
python Query_web_electricity_rate.py -r
```

### Cache Control

```bash
# Force refresh all data from web sources (ignore cache)
python Query_web_electricity_rate.py --force-refresh
python Query_web_electricity_rate.py -f

# Use only cached/default data (no web updates)
python Query_web_electricity_rate.py --no-update
python Query_web_electricity_rate.py -n

# Force refresh and generate report
python Query_web_electricity_rate.py -f -g
```

### API Key (Optional)

```bash
# Use API key for data.gov.in
python Query_web_electricity_rate.py --api-key YOUR_API_KEY
python Query_web_electricity_rate.py -k YOUR_API_KEY

# Or set as environment variable
export DATA_GOV_IN_API_KEY="your_api_key"
python Query_web_electricity_rate.py
```

## Command-Line Options Summary

| Option | Short | Description |
|--------|-------|-------------|
| `--generate-report` | `-g` | Generate full EV revenue report in markdown |
| `--generate-ipcu-costs` | | Generate IPCU cost targets from revenue model |
| `--ipcu-file FILE` | | Path to IPCU document (for `--generate-ipcu-costs`) |
| `--output FILE` | `-o` | Output file path for generated report |
| `--state STATE` | `-s` | Get rate for specific state |
| `--calculate UNITS` | `-c` | Calculate bill for given units |
| `--ev` | `-e` | Show EV DC Fast Charging rates |
| `--ev-stats` | | Show EV charging statistics summary |
| `--ev-revenue` | `-r` | Show EV charging station revenue estimates |
| `--status` | | Show data source status |
| `--force-refresh` | `-f` | Force refresh rates from web (ignore cache) |
| `--no-update` | `-n` | Don't auto-update, use cached/default rates |
| `--api-key KEY` | `-k` | API key for data.gov.in |
| `--help` | `-h` | Show help message |

## Generated Files

### Reports
- `ev_revenue_report.md` - Comprehensive EV charging business analysis
- `ipcu_cost_targets_section.md` - IPCU cost targets (standalone, when no --ipcu-file specified)

### Cache Files (auto-generated)
- `electricity_rates_cache.json` - Grid electricity rates by state
- `ev_charging_rates_cache.json` - EV charging rates (DC/AC)
- `ev_data_cache.json` - EV population and traffic data
- `land_prices_cache.json` - Commercial land prices by city
- `equipment_prices_cache.json` - Charger and equipment costs
- `solar_costs_cache.json` - Solar panel and battery costs

### Related Documentation
- `integrated_power_conversion_unit.md` - IPCU technical architecture (Section 7.0 can be auto-updated)
- `ipcu_home_dev_kit_projects.md` - Development kit project ideas

## Example Workflows

### 1. Generate Fresh Business Report

```bash
# Force refresh all data and generate report
python Query_web_electricity_rate.py --force-refresh --generate-report

# Check the generated report
cat ev_revenue_report.md
```

### 2. Quick State Comparison

```bash
# Compare rates for different states
python Query_web_electricity_rate.py -s "Maharashtra"
python Query_web_electricity_rate.py -s "Gujarat"
python Query_web_electricity_rate.py -s "Karnataka"
```

### 3. Estimate Monthly Bill

```bash
# Calculate bill for 500 units in Delhi
python Query_web_electricity_rate.py -s "Delhi" -c 500

# Calculate bill for 1000 units in Maharashtra
python Query_web_electricity_rate.py -s "Maharashtra" -c 1000
```

### 4. Revenue Analysis

```bash
# Show revenue potential by state
python Query_web_electricity_rate.py --ev-revenue

# Generate detailed report for business planning
python Query_web_electricity_rate.py -g -o business_plan_report.md
```

### 5. Update IPCU Technical Document

```bash
# Generate both reports (EV Revenue + IPCU Cost Targets) with fresh data
python Query_web_electricity_rate.py --force-refresh --generate-report
python Query_web_electricity_rate.py --generate-ipcu-costs --ipcu-file integrated_power_conversion_unit.md

# The IPCU document's Section 7.0 will now have:
# - Cost targets derived from actual revenue model
# - PMU/product tier pricing based on payback analysis
# - Auto-generated timestamp showing data freshness
```

## Data Flow Architecture

```
┌──────────────────────────────────────────────────────────────────────────┐
│                      DATA FLOW ARCHITECTURE                               │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   CACHE FILES (JSON)                  SCRIPT                             │
│   ──────────────────                  ──────                             │
│   electricity_rates_cache.json  ──┐                                      │
│   ev_charging_rates_cache.json  ──┼──► IndiaElectricityRateQuery         │
│   ev_data_cache.json            ──┤         │                            │
│   solar_costs_cache.json        ──┘         ▼                            │
│                                        ┌────────────┐                    │
│                                        │ Calculate  │                    │
│                                        │ Economics  │                    │
│                                        └─────┬──────┘                    │
│                                              │                           │
│                            ┌─────────────────┼─────────────────┐         │
│                            ▼                 ▼                 ▼         │
│                    ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│                    │EVReportGen  │  │IPCUCostGen   │  │ CLI Output   │  │
│                    └──────┬───────┘  └──────┬───────┘  └──────────────┘  │
│                           │                 │                            │
│                           ▼                 ▼                            │
│                    ev_revenue_report.md    integrated_power_             │
│                    (fully generated)        conversion_unit.md           │
│                                            (Section 7.0 updated)         │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

## Data Sources

- **Electricity Rates**: data.gov.in, state DISCOM websites
- **EV Charging Rates**: OpenChargeMap API, operator data
- **EV Population**: VAHAN database, FADA reports
- **Land Prices**: Market index estimates, housing APIs
- **Equipment Costs**: Manufacturer catalogs
- **Solar Costs**: MNRE benchmarks

## Report Contents

The generated `ev_revenue_report.md` includes:

1. **Executive Summary** - Key findings and recommendations
2. **Part 1: Demand Story** - EV population growth, infrastructure gap
3. **Part 2: Revenue Model** - Charging rates, station economics
4. **Part 3: Solar Offset** - Solar sizing, ROI calculations
5. **Part 4: Hybrid Systems** - Solar + battery economics
6. **Part 5: Component Costs** - BOM and build recommendations

## Requirements

- Python 3.8 or higher
- Internet connection (for data refresh)
- ~50MB disk space (for cache files)

## License

MIT License - Free for personal and commercial use.

## Contributing

Feel free to submit issues and pull requests for:
- Additional data sources
- New analysis features
- Bug fixes and improvements
