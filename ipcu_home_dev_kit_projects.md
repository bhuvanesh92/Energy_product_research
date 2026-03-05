# IPCU-HOME Dev Kit Projects

**Document Created:** 2026-02-24  
**Status:** Planning Phase  
**Target Audience:** Engineers, Makers, Students  
**Prerequisite:** Basic electronics, microcontroller programming

---

## 1. Overview

This document outlines a progressive series of development projects to build skills toward a full **IPCU-HOME** (Integrated Power Conversion Unit for residential applications). Each project uses commonly available dev kits and builds upon the previous one.

### 1.1 Learning Path

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         IPCU-HOME DEVELOPMENT PATH                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│   PHASE 1: FUNDAMENTALS          PHASE 2: POWER STAGES        PHASE 3: SYSTEM  │
│   ─────────────────────          ──────────────────────        ───────────────  │
│                                                                                  │
│   ┌─────────────┐               ┌─────────────┐               ┌─────────────┐  │
│   │ Project 1   │               │ Project 4   │               │ Project 7   │  │
│   │ Voltage     │──────────────►│ Buck/Boost  │──────────────►│ Solar MPPT  │  │
│   │ Sensing     │               │ Converter   │               │ Charger     │  │
│   └─────────────┘               └─────────────┘               └─────────────┘  │
│         │                             │                             │          │
│         ▼                             ▼                             ▼          │
│   ┌─────────────┐               ┌─────────────┐               ┌─────────────┐  │
│   │ Project 2   │               │ Project 5   │               │ Project 8   │  │
│   │ PWM &       │──────────────►│ H-Bridge    │──────────────►│ Battery     │  │
│   │ Gate Drive  │               │ Inverter    │               │ BMS         │  │
│   └─────────────┘               └─────────────┘               └─────────────┘  │
│         │                             │                             │          │
│         ▼                             ▼                             ▼          │
│   ┌─────────────┐               ┌─────────────┐               ┌─────────────┐  │
│   │ Project 3   │               │ Project 6   │               │ Project 9   │  │
│   │ Current     │──────────────►│ PFC         │──────────────►│ Full IPCU   │  │
│   │ Sensing     │               │ Rectifier   │               │ Integration │  │
│   └─────────────┘               └─────────────┘               └─────────────┘  │
│                                                                                  │
│   [Low Voltage]                 [Medium Voltage]              [Full Voltage]   │
│   [5-24V DC]                    [48-100V DC]                  [400V DC Bus]    │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Recommended Dev Kits

| Dev Kit | Manufacturer | Purpose | Est. Cost (Rs.) |
|---------|--------------|---------|-----------------|
| **STM32G4 Nucleo** | STMicroelectronics | Motor/power control MCU | Rs. 2,500 |
| **TI C2000 LaunchPad** | Texas Instruments | Real-time control DSP | Rs. 3,000 |
| **ESP32-S3** | Espressif | IoT, HMI, communication | Rs. 800 |
| **Arduino Mega** | Arduino | Prototyping, learning | Rs. 1,200 |
| **Raspberry Pi 4** | RPi Foundation | Energy management, UI | Rs. 5,000 |

---

## 2. Phase 1: Fundamentals (Low Voltage)

### Project 1: Voltage Sensing & Monitoring System

**Objective:** Build a multi-channel voltage monitoring system with display and logging.

#### 1.1 Learning Goals
- ADC configuration and calibration
- Voltage divider design for high-voltage sensing
- Real-time display updates
- Data logging to SD card

#### 1.2 Components

| Component | Specification | Qty | Est. Cost (Rs.) |
|-----------|---------------|-----|-----------------|
| STM32G4 Nucleo-G474RE | 170 MHz, 12-bit ADC | 1 | 2,500 |
| Voltage Divider Resistors | 1% tolerance, 1/4W | 10 | 50 |
| 16x2 LCD Display | I2C interface | 1 | 200 |
| SD Card Module | SPI interface | 1 | 150 |
| Bench Power Supply | 0-30V, 3A | 1 | 3,000 |
| Breadboard + Jumpers | Standard | 1 set | 300 |
| **TOTAL** | | | **Rs. 6,200** |

#### 1.3 Circuit Diagram

```
                    VOLTAGE SENSING CIRCUIT
    ════════════════════════════════════════════════════

    V_INPUT (0-50V)
         │
         │
        ┌┴┐
        │ │ R1 = 47kΩ (1%)
        │ │
        └┬┘
         │
         ├──────────────► ADC_IN (STM32G4)
         │                (0-3.3V range)
        ┌┴┐
        │ │ R2 = 3.3kΩ (1%)
        │ │
        └┬┘
         │
        ═╧═ GND

    Voltage Divider Ratio: V_ADC = V_INPUT × (R2 / (R1 + R2))
                          V_ADC = V_INPUT × (3.3 / 50.3)
                          V_ADC = V_INPUT × 0.0656

    For V_INPUT = 50V → V_ADC = 3.28V ✓
```

#### 1.4 Firmware Outline

```c
// Project 1: Voltage Sensing - STM32G4
// File: main.c

#include "stm32g4xx_hal.h"

#define ADC_CHANNELS    4
#define VREF            3.3f
#define ADC_RESOLUTION  4096
#define DIVIDER_RATIO   0.0656f

typedef struct {
    float voltage[ADC_CHANNELS];
    uint32_t timestamp;
} VoltageReading;

void ADC_Init(void);
float ADC_ReadVoltage(uint8_t channel);
void LCD_DisplayVoltages(VoltageReading *reading);
void SD_LogData(VoltageReading *reading);

int main(void) {
    HAL_Init();
    SystemClock_Config();
    ADC_Init();
    LCD_Init();
    SD_Init();
    
    VoltageReading reading;
    
    while (1) {
        // Read all channels
        for (int i = 0; i < ADC_CHANNELS; i++) {
            uint16_t adc_raw = ADC_Read(i);
            float v_adc = (adc_raw * VREF) / ADC_RESOLUTION;
            reading.voltage[i] = v_adc / DIVIDER_RATIO;
        }
        reading.timestamp = HAL_GetTick();
        
        LCD_DisplayVoltages(&reading);
        SD_LogData(&reading);
        
        HAL_Delay(100);  // 10 Hz update rate
    }
}
```

#### 1.5 Success Criteria
- [ ] Measure 4 voltage channels simultaneously
- [ ] Accuracy within ±1% of multimeter reading
- [ ] Display updates at 10 Hz
- [ ] Log data to SD card in CSV format
- [ ] Overvoltage alarm at configurable threshold

---

### Project 2: PWM Generation & Gate Driver Interface

**Objective:** Generate complementary PWM signals with dead-time for driving power MOSFETs.

#### 2.1 Learning Goals
- Hardware timer configuration for PWM
- Dead-time insertion for shoot-through protection
- Gate driver IC interfacing
- Oscilloscope measurement techniques

#### 2.2 Components

| Component | Specification | Qty | Est. Cost (Rs.) |
|-----------|---------------|-----|-----------------|
| STM32G4 Nucleo-G474RE | HRTIM peripheral | 1 | (from Project 1) |
| IR2110 Gate Driver | Bootstrap, 600V | 2 | 200 |
| IRFZ44N MOSFET | 55V, 49A, N-channel | 4 | 160 |
| Bootstrap Capacitor | 10µF, 25V ceramic | 2 | 20 |
| Bootstrap Diode | UF4007, fast recovery | 2 | 10 |
| Gate Resistors | 10Ω, 1/2W | 4 | 20 |
| 12V Power Supply | For gate drive | 1 | 500 |
| Oscilloscope | 2-channel minimum | 1 | (lab equipment) |
| **TOTAL** | | | **Rs. 910** |

#### 2.3 Circuit Diagram

```
                    HALF-BRIDGE GATE DRIVER CIRCUIT
    ════════════════════════════════════════════════════════════

                          VCC (12V)
                             │
                             │
              ┌──────────────┴──────────────┐
              │                             │
              │         IR2110              │
              │    ┌─────────────────┐      │
    PWM_H ────┼───►│ HIN         VB  │──────┼───► To High-Side Gate
              │    │                 │      │
    PWM_L ────┼───►│ LIN         HO  │──────┼───► (via 10Ω resistor)
              │    │                 │      │
              │    │ VCC        VS   │──────┼───► Half-Bridge Output
              │    │                 │      │
              │    │ COM        LO   │──────┼───► To Low-Side Gate
              │    └─────────────────┘      │
              │           │                 │
              │          ═╧═                │
              │          GND                │
              └─────────────────────────────┘

    Dead-Time: 100ns minimum (configured in HRTIM)
```

#### 2.4 Firmware Outline

```c
// Project 2: PWM Generation - STM32G4 HRTIM
// File: pwm_config.c

#include "stm32g4xx_hal.h"

#define PWM_FREQUENCY   20000   // 20 kHz
#define DEAD_TIME_NS    200     // 200 ns dead-time

HRTIM_HandleTypeDef hhrtim1;

void HRTIM_PWM_Init(void) {
    // Configure HRTIM for complementary PWM with dead-time
    
    HRTIM_TimeBaseCfgTypeDef timebase = {0};
    timebase.Period = 0xFFFF;  // Will be calculated
    timebase.RepetitionCounter = 0;
    timebase.PrescalerRatio = HRTIM_PRESCALERRATIO_MUL32;
    timebase.Mode = HRTIM_MODE_CONTINUOUS;
    
    HAL_HRTIM_TimeBaseConfig(&hhrtim1, HRTIM_TIMERINDEX_TIMER_A, &timebase);
    
    // Configure dead-time
    HRTIM_DeadTimeCfgTypeDef deadtime = {0};
    deadtime.Prescaler = HRTIM_TIMDEADTIME_PRESCALERRATIO_MUL8;
    deadtime.RisingValue = DEAD_TIME_NS / 8;   // Rising edge delay
    deadtime.FallingValue = DEAD_TIME_NS / 8;  // Falling edge delay
    
    HAL_HRTIM_DeadTimeConfig(&hhrtim1, HRTIM_TIMERINDEX_TIMER_A, &deadtime);
}

void PWM_SetDutyCycle(float duty) {
    // duty: 0.0 to 1.0
    if (duty < 0.0f) duty = 0.0f;
    if (duty > 0.95f) duty = 0.95f;  // Max 95% for bootstrap charging
    
    uint32_t compare = (uint32_t)(duty * hhrtim1.Instance->sTimerxRegs[0].PERxR);
    __HAL_HRTIM_SETCOMPARE(&hhrtim1, HRTIM_TIMERINDEX_TIMER_A, 
                           HRTIM_COMPAREUNIT_1, compare);
}
```

#### 2.5 Success Criteria
- [ ] Generate 20 kHz complementary PWM
- [ ] Dead-time visible on oscilloscope (200 ns)
- [ ] Duty cycle adjustable 0-95%
- [ ] No shoot-through (verify with current probe)
- [ ] Gate waveforms clean, <50 ns rise/fall

---

### Project 3: Current Sensing & Protection

**Objective:** Implement high-bandwidth current sensing with overcurrent protection.

#### 3.1 Learning Goals
- Hall-effect sensor interfacing
- Shunt resistor current sensing
- Analog comparator for fast protection
- Software overcurrent detection

#### 3.2 Components

| Component | Specification | Qty | Est. Cost (Rs.) |
|-----------|---------------|-----|-----------------|
| ACS712-20A | Hall-effect, ±20A | 2 | 300 |
| INA240 | High-side shunt amplifier | 1 | 250 |
| Shunt Resistor | 10mΩ, 5W, 1% | 1 | 100 |
| LM393 Comparator | Dual, for OCP | 1 | 30 |
| Potentiometer | 10kΩ, for threshold | 2 | 40 |
| LEDs (Status) | Red, Green | 4 | 20 |
| **TOTAL** | | | **Rs. 740** |

#### 3.3 Circuit Diagram

```
                    CURRENT SENSING CIRCUIT
    ════════════════════════════════════════════════════════════

    METHOD 1: HALL-EFFECT (ACS712)
    ───────────────────────────────
                                    VCC (5V)
                                       │
    CURRENT PATH                       │
    ═══════════╦═══════════     ┌──────┴──────┐
               ║                │   ACS712    │
               ║                │             │
               ║ ◄──────────────│ IP+    VOUT │───► ADC (0-5V)
               ║                │             │     2.5V = 0A
               ║ ◄──────────────│ IP-    GND  │     66mV/A
               ║                └──────┬──────┘
    ═══════════╩═══════════            │
                                      ═╧═

    METHOD 2: SHUNT + INA240
    ─────────────────────────
                    R_SHUNT (10mΩ)
    CURRENT ═══════╦═══╦═══════════►
                   │   │
                   │   │
              ┌────┴───┴────┐
              │  IN+   IN-  │
              │             │
              │   INA240    │
              │             │
              │    OUT      │───► ADC (0-3.3V)
              │             │     Gain = 50 V/V
              └─────────────┘     1A = 0.5V output
```

#### 3.4 Success Criteria
- [ ] Measure current with <2% accuracy
- [ ] Bandwidth >50 kHz for transient detection
- [ ] Hardware OCP trips within 1 µs
- [ ] Software OCP with configurable threshold
- [ ] Current waveform logging for analysis

---

## 3. Phase 2: Power Stages (Medium Voltage)

### Project 4: Synchronous Buck/Boost Converter

**Objective:** Build a bidirectional DC-DC converter for battery charging/discharging.

#### 4.1 Learning Goals
- Inductor selection and saturation
- Synchronous rectification
- Voltage/current mode control
- Efficiency measurement

#### 4.2 Components

| Component | Specification | Qty | Est. Cost (Rs.) |
|-----------|---------------|-----|-----------------|
| TI C2000 LaunchPad | F28379D, real-time control | 1 | 3,000 |
| IPP075N15N3G MOSFET | 150V, 100A, OptiMOS | 4 | 800 |
| Gate Driver (UCC21520) | Isolated, dual-channel | 2 | 600 |
| Inductor | 100µH, 20A, ferrite | 1 | 500 |
| DC Bus Capacitor | 100V, 1000µF electrolytic | 4 | 400 |
| Output Capacitor | 63V, 2200µF | 2 | 200 |
| Current Sensor (ACS770) | 50A, Hall-effect | 2 | 600 |
| Heatsink | 1°C/W, aluminum | 1 | 300 |
| 48V Battery (or PSU) | 48V, 20Ah LFP | 1 | 8,000 |
| **TOTAL** | | | **Rs. 14,400** |

#### 4.3 Specifications

| Parameter | Buck Mode | Boost Mode |
|-----------|-----------|------------|
| Input Voltage | 48-60V DC | 24-48V DC |
| Output Voltage | 24-48V DC | 48-60V DC |
| Max Current | 20A | 15A |
| Switching Freq | 50 kHz | 50 kHz |
| Target Efficiency | >95% | >94% |

#### 4.4 Control Block Diagram

```
                    BUCK/BOOST CONTROL LOOP
    ════════════════════════════════════════════════════════════

    V_REF ──►(+)──►┌─────────┐    ┌─────────┐    ┌─────────┐
              │    │ Voltage │    │ Current │    │  PWM    │
              │    │   PI    │───►│   PI    │───►│  Gen    │───► Gate
              │    │Controller│   │Controller│   │         │     Drive
              │    └─────────┘    └────┬────┘    └─────────┘
              │         ▲              │
              │         │              │
    V_OUT ────┴─────────┘              │
    (Feedback)                         │
                                       │
    I_INDUCTOR ────────────────────────┘
    (Feedback)

    Outer Loop: Voltage control (slow, ~1 kHz bandwidth)
    Inner Loop: Current control (fast, ~10 kHz bandwidth)
```

#### 4.5 Success Criteria
- [ ] Bidirectional power flow demonstrated
- [ ] Efficiency >94% at 500W
- [ ] Stable voltage regulation (±1%)
- [ ] Soft-start implemented
- [ ] Overcurrent protection functional

---

### Project 5: Single-Phase H-Bridge Inverter

**Objective:** Convert DC to AC with pure sine wave output.

#### 5.1 Learning Goals
- SPWM (Sinusoidal PWM) generation
- LC filter design
- THD measurement and reduction
- Grid synchronization basics

#### 5.2 Components

| Component | Specification | Qty | Est. Cost (Rs.) |
|-----------|---------------|-----|-----------------|
| TI C2000 LaunchPad | (from Project 4) | 1 | - |
| IRFP4668 MOSFET | 200V, 130A | 4 | 1,200 |
| Gate Driver (IR2113) | High/low side | 2 | 300 |
| Filter Inductor | 2mH, 10A, iron core | 2 | 800 |
| Filter Capacitor | 10µF, 400V film | 2 | 300 |
| Isolation Transformer | 1:1, 1kVA | 1 | 2,500 |
| AC Load (Resistive) | 500W bulbs | 1 set | 500 |
| Oscilloscope | 4-channel preferred | 1 | (lab) |
| **TOTAL** | | | **Rs. 5,600** |

#### 5.3 Specifications

| Parameter | Value |
|-----------|-------|
| DC Input | 100V DC (from bus capacitor) |
| AC Output | 230V RMS, 50 Hz |
| Power Rating | 1 kVA |
| THD | <5% target |
| Switching Frequency | 20 kHz |

#### 5.4 SPWM Generation

```
                    SINUSOIDAL PWM WAVEFORMS
    ════════════════════════════════════════════════════════════

    Carrier (Triangle, 20 kHz)
         /\    /\    /\    /\    /\    /\    /\    /\
        /  \  /  \  /  \  /  \  /  \  /  \  /  \  /  \
       /    \/    \/    \/    \/    \/    \/    \/    \

    Reference (Sine, 50 Hz)
                        ___
                    ___/   \___
                 __/           \__
              __/                 \__
           __/                       \__
          /                             \

    PWM Output (Leg A)
    ████  ████  ████████  ████████  ████████  ████  ████
        ██    ██        ██        ██        ██    ██

    After LC Filter → Clean 50 Hz Sine Wave
```

#### 5.5 Success Criteria
- [ ] 230V RMS output at 50 Hz
- [ ] THD <5% measured with power analyzer
- [ ] Stable under 0-100% load step
- [ ] Efficiency >90% at rated load
- [ ] No audible noise from filter

---

### Project 6: Power Factor Correction (PFC) Rectifier

**Objective:** Build an active front-end rectifier with unity power factor.

#### 6.1 Learning Goals
- Boost PFC topology
- Grid current shaping
- Harmonic reduction
- EMI filter design

#### 6.2 Components

| Component | Specification | Qty | Est. Cost (Rs.) |
|-----------|---------------|-----|-----------------|
| STM32G4 Nucleo | (or C2000) | 1 | - |
| Bridge Rectifier | 600V, 25A | 1 | 150 |
| PFC MOSFET | IPW65R045C7, 650V | 1 | 400 |
| PFC Diode | C3D10065A, SiC | 1 | 350 |
| Boost Inductor | 500µH, 15A, ferrite | 1 | 800 |
| DC Bus Capacitor | 450V, 470µF | 2 | 600 |
| EMI Filter | Common-mode choke + X/Y caps | 1 | 500 |
| Voltage Sensor | Isolated, for AC & DC | 2 | 400 |
| **TOTAL** | | | **Rs. 3,200** |

#### 6.3 Specifications

| Parameter | Value |
|-----------|-------|
| AC Input | 230V ±15%, 50 Hz |
| DC Output | 400V DC |
| Power Rating | 3 kW |
| Power Factor | >0.99 |
| THD (Input Current) | <5% |

#### 6.4 Success Criteria
- [ ] Power factor >0.99 at rated load
- [ ] Input current THD <5%
- [ ] DC bus regulation ±2%
- [ ] Soft-start without inrush
- [ ] EMI compliance (conducted)

---

## 4. Phase 3: System Integration (Full Voltage)

### Project 7: Solar MPPT Charge Controller

**Objective:** Build a maximum power point tracker for solar panels.

#### 7.1 Learning Goals
- MPPT algorithms (P&O, Incremental Conductance)
- Solar panel I-V characteristics
- Battery charging profiles (CC-CV)
- Efficiency optimization

#### 7.2 Components

| Component | Specification | Qty | Est. Cost (Rs.) |
|-----------|---------------|-----|-----------------|
| Solar Panel | 300W, 36V Vmp | 1 | 12,000 |
| DC-DC Converter | (from Project 4, upgraded) | 1 | - |
| LiFePO4 Battery | 48V, 50Ah | 1 | 25,000 |
| Current/Voltage Sensors | High precision | 4 | 1,000 |
| ESP32 (for monitoring) | WiFi, BLE | 1 | 800 |
| LCD Display | 3.5" TFT | 1 | 600 |
| **TOTAL** | | | **Rs. 39,400** |

#### 7.3 MPPT Algorithm

```
                    PERTURB & OBSERVE ALGORITHM
    ════════════════════════════════════════════════════════════

                         START
                           │
                           ▼
                    ┌─────────────┐
                    │ Measure V, I│
                    │ Calculate P │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │ ΔP = P - P_prev │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
         ΔP > 0       ΔP = 0       ΔP < 0
              │            │            │
              ▼            │            ▼
    ┌─────────────┐        │    ┌─────────────┐
    │ ΔV > 0?     │        │    │ ΔV > 0?     │
    └──────┬──────┘        │    └──────┬──────┘
           │               │           │
      ┌────┴────┐          │      ┌────┴────┐
      │         │          │      │         │
      ▼         ▼          │      ▼         ▼
   V += ΔV   V -= ΔV       │   V -= ΔV   V += ΔV
      │         │          │      │         │
      └────┬────┘          │      └────┬────┘
           │               │           │
           └───────────────┴───────────┘
                           │
                           ▼
                    ┌─────────────┐
                    │ Update PWM  │
                    │ Wait Δt     │
                    └──────┬──────┘
                           │
                           └──────► LOOP
```

#### 7.4 Success Criteria
- [ ] MPPT efficiency >99%
- [ ] Track MPP within 5 seconds
- [ ] Battery charging CC-CV profile
- [ ] WiFi monitoring dashboard
- [ ] Data logging (daily yield)

---

### Project 8: Battery Management System (BMS)

**Objective:** Build a BMS for a 48V LiFePO4 pack (16S configuration).

#### 8.1 Learning Goals
- Cell voltage monitoring
- Passive/active balancing
- SOC estimation
- Thermal management

#### 8.2 Components

| Component | Specification | Qty | Est. Cost (Rs.) |
|-----------|---------------|-----|-----------------|
| BQ76952 | 16S AFE IC | 1 | 800 |
| Cell Voltage Sense | Precision resistors | 16 | 200 |
| Balance Resistors | 10Ω, 2W | 16 | 160 |
| Balance MOSFETs | 2N7002 | 16 | 80 |
| NTC Thermistors | 10kΩ | 8 | 80 |
| STM32L4 MCU | Low power | 1 | 1,500 |
| CAN Transceiver | MCP2551 | 1 | 100 |
| Contactor Driver | For main disconnect | 1 | 300 |
| **TOTAL** | | | **Rs. 3,220** |

#### 8.3 Success Criteria
- [ ] Monitor 16 cells with <5mV accuracy
- [ ] Passive balancing functional
- [ ] SOC estimation within ±5%
- [ ] Over/under voltage protection
- [ ] CAN communication to main controller

---

### Project 9: Full IPCU-HOME Integration

**Objective:** Integrate all subsystems into a complete IPCU-HOME unit.

#### 9.1 System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           IPCU-HOME INTEGRATED SYSTEM                            │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│   ┌─────────────┐     ┌─────────────────────────────┐     ┌─────────────┐      │
│   │   SOLAR     │     │                             │     │    GRID     │      │
│   │   PANEL     │────►│                             │◄───►│   230V AC   │      │
│   │   300W      │     │                             │     │             │      │
│   └─────────────┘     │                             │     └─────────────┘      │
│                       │                             │                          │
│   ┌─────────────┐     │        400V DC BUS          │     ┌─────────────┐      │
│   │   BATTERY   │     │                             │     │   EV        │      │
│   │   48V LFP   │◄───►│    (Project 4,5,6,7,8)      │────►│   CHARGER   │      │
│   │   50Ah      │     │                             │     │   7kW       │      │
│   └─────────────┘     │                             │     └─────────────┘      │
│                       │                             │                          │
│   ┌─────────────┐     │                             │     ┌─────────────┐      │
│   │   BMS       │────►│                             │────►│   HOME      │      │
│   │   16S       │     │                             │     │   LOADS     │      │
│   └─────────────┘     └─────────────────────────────┘     └─────────────┘      │
│                                     │                                          │
│                                     │                                          │
│                       ┌─────────────┴─────────────┐                            │
│                       │     ENERGY MANAGEMENT     │                            │
│                       │     SYSTEM (EMS)          │                            │
│                       │                           │                            │
│                       │  • Raspberry Pi 4         │                            │
│                       │  • Node-RED / Python      │                            │
│                       │  • MQTT / Modbus          │                            │
│                       │  • Web Dashboard          │                            │
│                       │  • Mobile App             │                            │
│                       └───────────────────────────┘                            │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

#### 9.2 Integration Checklist

| Subsystem | Project Reference | Status |
|-----------|-------------------|--------|
| Voltage/Current Sensing | Project 1, 3 | ☐ |
| PWM & Gate Drivers | Project 2 | ☐ |
| DC-DC Converter | Project 4 | ☐ |
| Inverter | Project 5 | ☐ |
| PFC Rectifier | Project 6 | ☐ |
| Solar MPPT | Project 7 | ☐ |
| Battery BMS | Project 8 | ☐ |
| EMS Software | New | ☐ |
| Enclosure & Thermal | New | ☐ |
| Safety & Protection | All | ☐ |

#### 9.3 Total Development Cost Estimate

| Phase | Projects | Estimated Cost (Rs.) |
|-------|----------|----------------------|
| Phase 1 | 1, 2, 3 | Rs. 7,850 |
| Phase 2 | 4, 5, 6 | Rs. 23,200 |
| Phase 3 | 7, 8, 9 | Rs. 45,000 |
| **TOTAL** | | **Rs. 76,050** |

*Note: Costs assume some component reuse across projects.*

---

## 5. Resources

### 5.1 Reference Designs

| Design | Manufacturer | Description |
|--------|--------------|-------------|
| TIDA-010210 | Texas Instruments | 11kW Bidirectional EV Charger |
| TIDA-01606 | Texas Instruments | 3.3kW PFC + LLC |
| REF_DAB_500W | Infineon | Dual Active Bridge |
| STEVAL-ISV009V1 | STMicroelectronics | 3kW Solar Inverter |

### 5.2 Learning Resources

- **Power Electronics Specialization** - University of Colorado (Coursera)
- **TI Power Supply Design Seminars** - ti.com/psds
- **Infineon Application Notes** - infineon.com/power
- **STM32 Motor Control SDK** - st.com/motor-control

### 5.3 Safety Notes

⚠️ **WARNING: High Voltage Hazard**

- Projects 4-9 involve voltages >50V DC and 230V AC
- Always use isolated measurement equipment
- Implement proper grounding and enclosures
- Never work on energized circuits alone
- Use fuses and circuit breakers appropriately
- Discharge capacitors before servicing

---

**Document Version:** 1.0  
**Last Updated:** 2026-02-24  
**Next Review:** After completing Phase 1 projects
