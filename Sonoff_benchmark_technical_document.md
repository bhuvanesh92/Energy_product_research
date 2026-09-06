# Recreating the SONOFF POWCT Energy-Monitoring System

## End-to-End Architecture, Metrology, Firmware, UART Protocol, Cloud Telemetry and Implementation Guide

**Audience:** System architects, embedded developers, firmware engineers, electrical engineers and developers building a functionally equivalent energy-monitoring system.

**Status:** Engineering reconstruction based on publicly available documentation, reverse engineering, firmware implementations and observed hardware architecture.

---

## 1. Executive Summary

This document describes how to recreate the architecture of a SONOFF POWCT-class smart power-monitoring system from first principles.

The system can be understood as four major layers:

```text
┌──────────────────────────────────────────────────────────────┐
│                    USER / APPLICATION                       │
│                                                              │
│                  eWeLink / Mobile App                       │
└──────────────────────────────┬───────────────────────────────┘
                               │
                         Internet / Wi-Fi
                               │
┌──────────────────────────────▼───────────────────────────────┐
│                    CONNECTIVITY / CONTROL                    │
│                                                              │
│                         ESP32                                 │
│          ┌───────────────┼────────────────┐                  │
│          │               │                │                  │
│       Wi-Fi          Local LCD         Relay Control         │
│          │                                │                  │
└──────────┼────────────────────────────────┼───────────────────┘
           │                                │
           │ UART                            ▼
           │                         External Contactor
           ▼
┌──────────────────────────────────────────────────────────────┐
│                     METROLOGY ASIC                           │
│                                                              │
│                         CSE7761                              │
│                                                              │
│     ADC → waveform processing → RMS → power → energy        │
└───────────────┬───────────────────────────────┬──────────────┘
                │                               │
                ▼                               ▼
          Voltage sensing                 CT current sensing
                │                               │
                └───────────────┬───────────────┘
                                ▼
                           AC MAINS / LOAD
```

The most important architectural insight is:

> **The ESP32 is not the energy-metering engine. The CSE7761 is.**

The CSE7761 receives analog voltage/current information, performs high-speed sampling and metrology calculations, and exposes measurements through a digital UART interface.

The ESP32 acts primarily as the:

* system controller,
* CSE7761 host,
* calibration/scaling layer,
* connectivity processor,
* telemetry publisher,
* local display controller,
* relay/contactor controller,
* configuration manager.

This separation is extremely useful when recreating the system because the design naturally divides into two independently testable subsystems:

```text
                    METROLOGY DOMAIN
                ┌─────────────────────┐
                │ Voltage / Current   │
                │ sensing             │
                │        ↓            │
                │ CSE7761             │
                │        ↓            │
                │ Digital measurements│
                └──────────┬──────────┘
                           │ UART
                           ▼
                ┌─────────────────────┐
                │     ESP32           │
                │                     │
                │ Scale               │
                │ Validate            │
                │ Aggregate           │
                │ Publish             │
                │ Display             │
                │ Control             │
                └─────────────────────┘
```

---

# 2. System Objective

The recreated system should be capable of:

1. Measuring AC voltage.
2. Measuring AC current.
3. Measuring active power.
4. Measuring apparent power.
5. Measuring power factor.
6. Measuring accumulated energy.
7. Providing local measurement display.
8. Providing Wi-Fi connectivity.
9. Publishing telemetry to a cloud/application backend.
10. Receiving control commands.
11. Controlling a load through an appropriate switching stage.
12. Supporting calibration.
13. Recovering from communication or sensor faults.
14. Maintaining safe isolation between mains and low-voltage electronics.

A representative user-visible measurement might be:

```text
Voltage       231.4 V
Current         8.72 A
Active power  2010 W
Power factor   0.996
Energy        12.84 kWh
```

These values are not necessarily calculated from each other by the ESP32.

The metrology ASIC independently derives quantities from sampled electrical waveforms.

For example:

```text
Voltage waveform ─┐
                  ├──► CSE7761 ──► Active Power
Current waveform ─┘

Voltage waveform ─┐
                  ├──► CSE7761 ──► RMS Voltage
                  │
Current waveform ─┘               └──► RMS Current
```

This distinction is fundamental.

---

# 3. High-Level Architecture

## 3.1 Complete End-to-End Architecture

```text
                         ┌──────────────────────┐
                         │      MOBILE APP      │
                         │                      │
                         │ Voltage              │
                         │ Current              │
                         │ Power                │
                         │ Energy               │
                         │ Status               │
                         │ Controls             │
                         └──────────┬───────────┘
                                    │
                                    │ HTTPS / MQTT /
                                    │ proprietary cloud protocol
                                    │
                         ┌──────────▼───────────┐
                         │    CLOUD BACKEND     │
                         │                      │
                         │ Device identity      │
                         │ Telemetry            │
                         │ State                │
                         │ Commands             │
                         │ Historical data      │
                         └──────────┬───────────┘
                                    │
                              Internet
                                    │
                              Wi-Fi / IP
                                    │
                         ┌──────────▼───────────┐
                         │        ESP32         │
                         │                      │
                         │ Application          │
                         │ Connectivity         │
                         │ Device state         │
                         │ CSE driver           │
                         │ Display driver       │
                         │ Relay driver         │
                         └─────┬──────┬─────────┘
                               │      │
                            UART      │ GPIO
                               │      │
                ┌──────────────▼─┐    ▼
                │    CSE7761     │  Relay
                │                │
                │ Metrology      │
                │ ADC             │
                │ RMS             │
                │ Power           │
                │ Energy          │
                └──────┬─────────┘
                       │
              Analog measurement
                       │
              ┌────────┴─────────┐
              │                  │
        Voltage sensing      CT sensing
              │                  │
              └────────┬─────────┘
                       │
                    AC LOAD
```

---

# 4. Hardware Architecture

## 4.1 Major Components

A POWCT-class implementation consists conceptually of:

| Component             | Responsibility                        |
| --------------------- | ------------------------------------- |
| AC voltage sensing    | Measures mains voltage                |
| Split-core CT         | Measures load current                 |
| CSE7761               | Performs metrology                    |
| ESP32                 | Application/control/connectivity      |
| Wi-Fi antenna         | RF connectivity                       |
| LCD driver            | Local measurement display             |
| LCD                   | Local user interface                  |
| Relay                 | Low-power switching/control           |
| External contactor    | High-current load switching           |
| Isolated power supply | Converts mains to safe low-voltage DC |
| Push button           | Local configuration/reset             |
| Status LEDs           | Device state indication               |

The important distinction is between the **measurement path** and the **control path**.

```text
                 MEASUREMENT PATH

 Mains ──► Voltage sensor ────────────────┐
                                          │
 Load ──► CT ─────────────────────────────┤
                                          ▼
                                      CSE7761
                                          │
                                          │ UART
                                          ▼
                                        ESP32
                                          │
                              ┌───────────┴───────────┐
                              ▼                       ▼
                           Display                Cloud


                  CONTROL PATH

 Mobile App
     │
     ▼
 Cloud
     │
     ▼
 ESP32
     │
     ▼
 Relay
     │
     ▼
 External contactor
     │
     ▼
 Load
```

---

# 5. Electrical Measurement Architecture

## 5.1 Current Measurement

The POWCT architecture uses a split-core current transformer (CT).

Conceptually:

```text
          High-current conductor
                 │
                 │
        ┌────────┴────────┐
        │                 │
        │       CT        │
        │    ┌──────┐     │
        │    │      │     │
        │    │  ⟳   │     │
        │    │      │     │
        │    └──────┘     │
        │                 │
        └─────────────────┘
                 │
                 ▼
           Analog current
                 │
                 ▼
              CSE7761
```

The CT provides galvanic isolation between the high-current conductor and the measurement electronics.

The CSE7761 then digitizes and processes the resulting signal.

---

# 6. Voltage Measurement

Voltage is measured using an appropriately scaled and protected sensing network.

Conceptually:

```text
       AC mains
          │
          │
       Protection
          │
          ▼
    Voltage divider
          │
          ▼
      Analog input
          │
          ▼
       CSE7761
```

The actual implementation must account for:

* mains voltage range,
* transient protection,
* creepage,
* clearance,
* resistor voltage rating,
* resistor power dissipation,
* isolation requirements,
* PCB layout,
* EMC,
* measurement accuracy.

This section is deliberately architectural rather than a construction recipe because mains measurement is a safety-critical design domain.

---

# 7. The CSE7761 Metrology ASIC

## 7.1 Architectural Role

The CSE7761 is the heart of the measurement system.

It performs substantially more than simple ADC conversion.

Its conceptual internal architecture is:

```text
             Analog Voltage
                   │
                   ▼
              ┌─────────┐
              │ Voltage │
              │   ADC   │
              └────┬────┘
                   │
                   │
                   ▼
              ┌─────────┐
              │         │
Current ─────►│ Digital │
   ADC        │ Signal  │
              │Processing
              │         │
              └────┬────┘
                   │
        ┌──────────┼───────────┐
        │          │           │
        ▼          ▼           ▼
       RMS       Power       Energy
        │          │           │
        ▼          ▼           ▼
    Voltage      Active      kWh
    Current      Power
                   │
                   ▼
              Power factor
                   │
                   ▼
                UART
                   │
                   ▼
                 ESP32
```

---

# 8. ADC and Signal Processing

The CSE7761 contains sigma-delta ADC channels and internal digital processing.

The architecture can be viewed as:

```text
Analog waveform
      │
      ▼
┌───────────────┐
│ Sigma-delta   │
│ ADC           │
└───────┬───────┘
        │
        ▼
Digital samples
        │
        ├──────────────► RMS calculation
        │
        ├──────────────► Instantaneous values
        │
        ├──────────────► Active power
        │
        ├──────────────► Apparent power
        │
        ├──────────────► Power factor
        │
        ├──────────────► Energy accumulation
        │
        └──────────────► Waveform information
```

This means the ESP32 does **not** need to sample mains waveforms at several kilohertz itself.

That work is already performed inside the metrology ASIC.

---

# 9. Why Active Power Is Not Simply V × I

Suppose the system reports:

```text
V = 231.4 V
I = 8.72 A
P = 2010 W
```

Multiplying RMS voltage and RMS current gives:

```text
231.4 × 8.72 = 2017.8 VA
```

This is apparent power, not necessarily active power.

The relationship is:

```text
                Active Power
Power Factor = ──────────────
              Apparent Power
```

Therefore:

```text
PF ≈ 2010 / 2017.8
   ≈ 0.996
```

The CSE7761 can derive active power directly from the relationship between voltage and current waveforms.

Conceptually:

```text
v(t) ─────┐
          │
          ├──► v(t) × i(t) ──► averaging ──► Active Power
          │
i(t) ─────┘
```

This is one of the most important architectural reasons to use a dedicated metrology ASIC.

---

# 10. CSE7761 Register Map

The following registers are particularly relevant to a POWCT implementation.

| Address | Register     | Size | Function                             |
| ------: | ------------ | ---: | ------------------------------------ |
|  `0x00` | SYSCON       |  2 B | System configuration                 |
|  `0x01` | EMUCON       |  2 B | Measurement configuration            |
|  `0x13` | EMUCON2      |  2 B | Additional measurement configuration |
|  `0x1D` | PULSE1SEL    |  2 B | Pulse configuration                  |
|  `0x20` | PFCnt_PA     |  2 B | Power pulse count                    |
|  `0x21` | PFCnt_PB     |  2 B | Power pulse count                    |
|  `0x22` | Angle        |  2 B | Phase angle                          |
|  `0x23` | Ufreq        |  2 B | Voltage frequency measurement        |
|  `0x24` | RmsIA        |  3 B | RMS current channel A                |
|  `0x25` | RmsIB        |  3 B | RMS current channel B                |
|  `0x26` | RmsU         |  3 B | RMS voltage                          |
|  `0x27` | PowerFactor  |  3 B | Power factor                         |
|  `0x28` | Energy_PA    |  3 B | Energy channel A                     |
|  `0x29` | Energy_PB    |  3 B | Energy channel B                     |
|  `0x2C` | PowerPA      |  4 B | Active power A                       |
|  `0x2D` | PowerPB      |  4 B | Active power B                       |
|  `0x2E` | PowerS       |  4 B | Apparent power                       |
|  `0x2F` | EMUStatus    |  3 B | Measurement status                   |
|  `0x30` | PeakIA       |  3 B | Peak current A                       |
|  `0x31` | PeakIB       |  3 B | Peak current B                       |
|  `0x32` | PeakU        |  3 B | Peak voltage                         |
|  `0x33` | InstanIA     |  3 B | Instantaneous current A              |
|  `0x34` | InstanIB     |  3 B | Instantaneous current B              |
|  `0x35` | InstanU      |  3 B | Instantaneous voltage                |
|  `0x36` | WaveIA       |  3 B | Current waveform                     |
|  `0x37` | WaveIB       |  3 B | Current waveform                     |
|  `0x38` | WaveU        |  3 B | Voltage waveform                     |
|  `0x3C` | InstanP      |  4 B | Instantaneous active power           |
|  `0x3D` | InstanS      |  4 B | Instantaneous apparent power         |
|  `0x43` | SYSSTATUS    |  1 B | System status                        |
|  `0x6F` | Coeff_chksum |  2 B | Calibration coefficient checksum     |
|  `0x70` | RmsIAC       |  2 B | Current calibration coefficient      |
|  `0x71` | RmsIBC       |  2 B | Current-B coefficient                |
|  `0x72` | RmsUC        |  2 B | Voltage calibration coefficient      |
|  `0x73` | PowerPAC     |  2 B | Power-A coefficient                  |
|  `0x74` | PowerPBC     |  2 B | Power-B coefficient                  |
|  `0x75` | PowerSC      |  2 B | Apparent-power coefficient           |
|  `0x76` | EnergyAC     |  2 B | Energy-A coefficient                 |
|  `0x77` | EnergyBC     |  2 B | Energy-B coefficient                 |
|  `0x7F` | DeviceID     |  3 B | Device identifier                    |

The CSE7761 device ID is documented as:

```text
0x776110
```

---

# 11. Measurement Update Rates

Different measurements have different update characteristics.

The CSE7761 supports configurable update rates for several RMS/average measurements, including approximately:

```text
3.4 Hz
6.8 Hz
13.6 Hz
27.2 Hz
```

The POWCT implementation uses measurement information suitable for a responsive smart-device telemetry system.

Instantaneous and waveform-related data operate at substantially higher rates, approximately several kilohertz.

Conceptually:

```text
                  Sampling / processing hierarchy

         ┌───────────────────────────────────────┐
         │ High-speed waveform processing        │
         │ ~kHz                                  │
         └──────────────────┬────────────────────┘
                            │
                            ▼
              ┌─────────────────────────┐
              │ RMS / power calculations│
              │ ~Hz                     │
              └────────────┬────────────┘
                           │
                           ▼
              ┌─────────────────────────┐
              │ ESP32 telemetry         │
              │ ~sub-second / seconds   │
              └────────────┬────────────┘
                           │
                           ▼
                       Cloud/App
```

The important architectural principle is:

> **Do not confuse sensor sampling rate with application telemetry rate.**

The metrology engine may process thousands of samples per second while the mobile application only needs a handful of measurements per second.

---

# 12. CSE7761 UART Interface

## 12.1 Electrical Interface

The ESP32 communicates with the CSE7761 over UART.

The interface is:

```text
ESP32                          CSE7761
─────                          ───────

TX ──────────────────────────► RX

RX ◄────────────────────────── TX

             UART
```

The documented UART configuration is:

```text
Baud rate:   38400
Data bits:   8
Parity:      Even
Stop bits:   1
```

The interface is effectively operated as a command/response, half-duplex protocol.

---

# 13. Physical GPIO Mapping

A reverse-engineered POWCT implementation identifies the following ESP32 GPIO assignments:

| ESP32 GPIO | Function            |
| ---------: | ------------------- |
|      GPIO0 | Push button         |
|      GPIO5 | TM1621 display data |
|     GPIO13 | Status LED          |
|     GPIO15 | Wi-Fi LED           |
|     GPIO17 | TM1621 chip select  |
|     GPIO18 | TM1621 write        |
|     GPIO21 | Relay               |
|     GPIO23 | TM1621 read         |
|     GPIO25 | CSE7761 RX          |
|     GPIO26 | CSE7761 TX          |

Therefore the central metrology path is:

```text
CSE7761 TX ─────────► ESP32 GPIO25
CSE7761 RX ◄───────── ESP32 GPIO26
```

The exact board revision should always be verified before treating this mapping as universal.

---

# 14. UART Packet Structure

The CSE7761 protocol is simple and deterministic.

A transaction starts with:

```text
0xA5
```

The next byte is the command.

The command byte contains:

```text
bit 7       = Read / Write
bits 6..0   = Register address
```

Conceptually:

```text
        Command byte

        7 6 5 4 3 2 1 0
       ┌─┬───────────────┐
       │R│ Register Addr │
       └─┴───────────────┘

       R = 0 → read
       R = 1 → write
```

Therefore:

```text
Read register 0x26:

0x26
```

and:

```text
Write register 0x26:

0xA6
```

because:

```text
0x26 | 0x80 = 0xA6
```

---

# 15. Read Transaction

A read transaction has the conceptual structure:

```text
ESP32 → CSE7761

A5 CMD


CSE7761 → ESP32

DATA[0]
DATA[1]
...
DATA[n]
CHECKSUM
```

For example, reading RMS voltage:

```text
Request:

A5 26
```

Suppose the raw register contains:

```text
22 CB 67
```

The CSE7761 response is:

```text
22 CB 67 E0
```

where:

```text
22 CB 67 = measurement
E0       = checksum
```

---

# 16. Checksum

The checksum is calculated as the bitwise complement of the 8-bit sum of:

```text
0xA5
+ command
+ all data bytes
```

Formally:

```text
checksum = ~(sum & 0xFF) & 0xFF
```

For:

```text
A5 26 22 CB 67
```

the calculation is:

```text
A5 + 26 + 22 + CB + 67
```

The low 8 bits are then complemented.

Result:

```text
E0
```

Thus:

```text
A5 26 22 CB 67 E0
```

is a valid example response.

---

# 17. Write Transaction

A write transaction has the form:

```text
A5
COMMAND
DATA...
CHECKSUM
```

For example:

```text
Write register 0x02 = 0x1234
```

The command is:

```text
0x02 | 0x80 = 0x82
```

Therefore:

```text
A5 82 12 34 92
```

The checksum is:

```text
~((A5 + 82 + 12 + 34) & FF)
= 92
```

---

# 18. Register Width Matters

The register width varies.

Examples:

```text
RmsU      = 3 bytes
RmsIA     = 3 bytes
PowerPA   = 4 bytes
EnergyPA  = 3 bytes
SYSCON    = 2 bytes
SYSSTATUS = 1 byte
```

The firmware therefore needs a register metadata table rather than assuming every measurement has the same width.

Example:

```text
Register     Width      Signed?

RmsU          3          No*
RmsIA         3          No*
PowerPA       4          Yes
EnergyPA      3          No
```

`*` The MSB has validity/sign semantics that must be handled according to the CSE7761 register definition.

---

# 19. Byte Ordering

Multi-byte register values are transmitted most-significant byte first.

For example:

```text
0x22CB67
```

is transmitted as:

```text
22 CB 67
```

The ESP32 reconstructs the integer:

```text
raw = (byte0 << 16)
    | (byte1 << 8)
    | byte2
```

For a 32-bit power register:

```text
raw = (byte0 << 24)
    | (byte1 << 16)
    | (byte2 << 8)
    | byte3
```

---

# 20. RMS Voltage Decoding

The voltage RMS register is:

```text
RmsU = 24-bit
```

The calibration equation is:

```text
Voltage =
    RmsU × RmsUC
    ─────────────────
       K2 × 2^22
```

The resulting unit is 10 mV according to the CSE7761 calibration model.

A practical firmware implementation can normalize this into volts.

A commonly used ESPHome-style formulation is:

```text
voltage_divisor =
    0x400000 × 100
    ───────────────
        RmsUC
```

Then:

```text
Voltage = RmsU / voltage_divisor
```

---

# 21. RMS Current Decoding

The current RMS register is:

```text
RmsIA = 24-bit
```

The calibration equation is:

```text
Current =
    RmsIA × RmsIAC
    ─────────────────
       K1 × 2^23
```

The normalized implementation can use:

```text
current_divisor =
    2^23 × 1000
    ─────────────
       RmsIAC
```

Then:

```text
Current[A] = RmsIA / current_divisor
```

The conversion must preserve the appropriate scaling from the CSE7761 representation.

---

# 22. Active Power Decoding

The active power register is:

```text
PowerPA = 32-bit signed
```

The calibration equation is:

```text
Power =
    PowerPA × PowerPAC
    ─────────────────────
       K1 × K2 × 2^31
```

A normalized implementation can use:

```text
power_divisor =
    2^31
    ───────
    PowerPAC
```

and:

```text
Power[W] = PowerPA / power_divisor
```

The raw value is signed.

Therefore the firmware must perform two's-complement interpretation before scaling.

---

# 23. Constructed End-to-End Measurement Example

Consider the desired output:

```text
Voltage:       231.4 V
Current:         8.72 A
Active power:  2010 W
```

Using representative fallback/reference calibration coefficients:

```text
UREF = 42563
IREF = 52241
PREF = 44513
```

the corresponding illustrative raw values are:

```text
Voltage raw:
2,280,295 = 0x22CB67

Current raw:
1,400,216 = 0x155D98

Power raw:
96,970,371 = 0x05C7A683
```

These values are **constructed mathematical examples**, not claims that these exact bytes were captured from a physical POWCT.

---

## 23.1 Voltage Transaction

Request:

```text
A5 26
```

Response:

```text
22 CB 67 E0
```

Decode:

```text
raw = 0x22CB67
    = 2,280,295
```

Then:

```text
Voltage =
    2,280,295 × 42,563
    ────────────────────
       2^22 × 100

Voltage = 231.4 V
```

---

## 23.2 Current Transaction

Request:

```text
A5 24
```

Response:

```text
15 5D 98 2C
```

Decode:

```text
raw = 0x155D98
    = 1,400,216
```

Then:

```text
Current =
    1,400,216 × 52,241
    ─────────────────────
       2^23 × 1000

Current = 8.72 A
```

---

## 23.3 Active Power Transaction

Request:

```text
A5 2C
```

Response:

```text
05 C7 A6 83 39
```

Decode:

```text
raw = 0x05C7A683
    = 96,970,371
```

Then:

```text
Power =
    96,970,371 × 44,513
    ────────────────────
             2^31

Power = 2010 W
```

---

# 24. From Raw Bytes to Mobile Application

The complete transformation can therefore be represented as:

```text
                 PHYSICAL WORLD

        231.4 V AC / 8.72 A load
                    │
                    ▼
            Analog waveforms
                    │
                    ▼
              CSE7761 ADC
                    │
                    ▼
          Digital signal processing
                    │
          ┌─────────┼─────────┐
          │         │         │
          ▼         ▼         ▼
        RMS V     RMS I    Active P
          │         │         │
          └─────────┼─────────┘
                    │
                    ▼
              CSE registers
                    │
                    ▼
                UART bytes
                    │
                    ▼
                  ESP32
                    │
            ┌───────┼────────┐
            │       │        │
            ▼       ▼        ▼
         Decode   Scale    Validate
            │       │        │
            └───────┼────────┘
                    ▼
             Application model
                    │
                    ▼
             Wi-Fi telemetry
                    │
                    ▼
               Cloud backend
                    │
                    ▼
                Mobile app
                    │
                    ▼
                "231.4 V"
                 "8.72 A"
                 "2.01 kW"
```

This is the essential end-to-end data pipeline.

---

# 25. ESP32 Software Architecture

The ESP32 firmware should be architected as independent services/modules rather than one monolithic loop.

Recommended architecture:

```text
┌─────────────────────────────────────────────┐
│                  ESP32 APP                  │
├─────────────────────────────────────────────┤
│ Device State Machine                        │
├──────────┬──────────┬──────────┬────────────┤
│ Metrology│ Display  │ Relay    │ Network    │
│ Manager  │ Manager  │ Manager  │ Manager    │
├────┬─────┴──────────┴──────────┴──────┬─────┤
│    │                                   │     │
│ CSE7761 Driver                         │ Wi-Fi
│    │                                   │     │
└────┼───────────────────────────────────┼─────┘
     │                                   │
     ▼                                   ▼
 CSE7761                              Cloud/API
```

---

# 26. Recommended Firmware Layering

A clean implementation should use approximately the following layers.

## Layer 1 — Hardware Abstraction

Responsible for:

* UART
* GPIO
* timers
* Wi-Fi
* nonvolatile storage
* watchdog
* GPIO interrupts

```text
HAL
├── UART
├── GPIO
├── Timer
├── Wi-Fi
├── NVS
└── Watchdog
```

---

## Layer 2 — CSE7761 Driver

Responsible only for:

* reset
* read register
* write register
* checksum
* register decoding
* communication timeout
* CRC/checksum validation
* raw measurement retrieval

It should not know anything about:

* MQTT,
* mobile apps,
* LCD layout,
* cloud authentication.

---

## Layer 3 — Metrology Service

Responsible for:

* voltage conversion,
* current conversion,
* power conversion,
* energy conversion,
* power factor,
* frequency,
* sanity checking,
* calibration management.

Example internal object:

```text
Measurement
{
    voltage_V
    current_A
    active_power_W
    apparent_power_VA
    power_factor
    energy_kWh
    frequency_Hz
    valid
    timestamp
}
```

---

## Layer 4 — Device State Manager

Responsible for:

```text
BOOT
  ↓
INITIALIZING
  ↓
SELF_TEST
  ↓
READY
  ↓
MEASURING
  ↓
CONNECTED
```

with fault states such as:

```text
CSE_ERROR
WIFI_ERROR
CLOUD_ERROR
CALIBRATION_ERROR
```

---

## Layer 5 — Connectivity

Responsible for:

* Wi-Fi connection,
* provisioning,
* device identity,
* cloud connection,
* telemetry,
* commands,
* reconnect logic.

---

## Layer 6 — Application

Responsible for:

* display,
* user controls,
* load switching,
* configuration,
* telemetry policies.

---

# 27. ESP32 ↔ CSE7761 Driver State Machine

A robust implementation should look conceptually like:

```text
             ┌─────────────┐
             │    RESET    │
             └──────┬──────┘
                    ▼
             ┌─────────────┐
             │ READ CONFIG │
             └──────┬──────┘
                    ▼
          ┌────────────────────┐
          │ READ CALIBRATION   │
          │ COEFFICIENTS       │
          └─────────┬──────────┘
                    ▼
             ┌─────────────┐
             │ VALIDATE    │
             │ CHECKSUM    │
             └──────┬──────┘
                    │
              ┌─────┴─────┐
              │           │
            VALID       INVALID
              │           │
              │           ▼
              │      Use fallback
              │      coefficients
              │           │
              └─────┬─────┘
                    ▼
             ┌─────────────┐
             │ WRITE ENABLE│
             └──────┬──────┘
                    ▼
             ┌─────────────┐
             │ CONFIGURE   │
             │ CSE7761     │
             └──────┬──────┘
                    ▼
             ┌─────────────┐
             │ WRITE       │
             │ PROTECT     │
             └──────┬──────┘
                    ▼
             ┌─────────────┐
             │ PERIODIC    │
             │ MEASUREMENT │
             └──────┬──────┘
                    │
                    └──────────► repeat
```

---

# 28. CSE7761 Reset

A special reset command is used by implementations of the driver.

Representative sequence:

```text
A5 EA 96 DA
```

This should be treated as a special command rather than an ordinary register write.

After reset, the driver should allow the ASIC to reach a known state before proceeding.

---

# 29. Reading Factory Calibration

The calibration registers occupy:

```text
0x70 ... 0x77
```

The driver reads:

```text
RmsIAC
RmsIBC
RmsUC
PowerPAC
PowerPBC
PowerSC
EnergyAC
EnergyBC
```

and validates them against:

```text
Coeff_chksum
```

at:

```text
0x6F
```

Conceptually:

```text
              CSE7761
                 │
      ┌──────────┴───────────┐
      │                      │
      ▼                      ▼
0x70...0x77              0x6F
coefficients            checksum
      │                      │
      └──────────┬───────────┘
                 ▼
          Validate coefficients
                 │
          ┌──────┴──────┐
          │             │
        VALID         INVALID
          │             │
          ▼             ▼
       Use them     Use fallback
```

---

# 30. Calibration Coefficients

A commonly implemented fallback/reference set is:

```text
UREF = 42563
IREF = 52241
PREF = 44513
```

These values are useful for reproducing the software scaling behavior found in open-source drivers.

However:

> They should not automatically be assumed to be the factory calibration values of every production POWCT unit.

A production recreation should preferably:

1. read the device's calibration coefficients,
2. validate them,
3. use them when valid,
4. use known fallback coefficients only when appropriate.

---

# 31. Generic Driver Initialization

A commonly implemented CSE7761 initialization sequence performs:

```text
1. Reset
2. Read SYSCON
3. Read calibration coefficients
4. Validate coefficient checksum
5. Enable register writes
6. Check system status
7. Configure SYSCON
8. Configure EMUCON
9. Configure EMUCON2
10. Configure PULSE1SEL
11. Write-protect configuration
12. Begin periodic measurement
```

One generic ESPHome-derived implementation uses:

```text
SYSCON   = 0xFF04
EMUCON   = 0x1183
EMUCON2  = 0x0FC1
PULSE1SEL= 0x3290
```

The corresponding write transactions are:

```text
A5 80 FF 04 D7
A5 81 11 83 45
A5 93 0F C1 F7
A5 9D 32 90 FB
```

Again, these values belong to a **generic CSE7761 driver configuration** and must not be blindly treated as the exact production POWCT configuration.

---

# 32. POWCT-Specific Configuration Caveat

This distinction is particularly important when recreating the actual POWCT.

Reverse engineering of POWCT firmware indicates that its CSE7761 configuration differs from configurations used in other products such as the Dual R3.

The CSE7761 `SYSCON` register controls, among other things:

```text
ADC2 enable
Current-channel-B PGA
Voltage PGA
Current-channel-A PGA
```

The relevant conceptual bit allocation is:

```text
bit 10       ADC2ON
bits 8..6    PGAIB
bits 5..3    PGAU
bits 2..0    PGAIA
```

The POWCT is a single-channel measurement implementation, so its configuration should reflect:

```text
Current channel A: enabled
Current channel B: disabled
Voltage channel:    enabled
```

The reverse-engineered Tasmota implementation contains POWCT-specific configuration handling.

Therefore, when reproducing the device:

> **Use the actual POWCT-specific configuration rather than copying an unrelated CSE7761 product's initialization sequence.**

This is one of the areas that should be validated against captured UART traffic or the target firmware.

---

# 33. Example Special Commands

The generic driver uses the following special sequences:

### Reset

```text
A5 EA 96 DA
```

### Write enable

```text
A5 EA E5 8B
```

### Write protect

```text
A5 EA DC 94
```

These are protocol-level control sequences.

They should be represented explicitly in the driver:

```c
cse_reset();
cse_write_enable();
configure_registers();
cse_write_protect();
```

rather than being hidden as unexplained byte arrays.

---

# 34. Why Write Protection Exists

The metrology configuration should not be continuously writable.

A reasonable security/robustness model is:

```text
NORMAL OPERATION
       │
       ▼
Configuration protected
       │
       │
configuration required
       ▼
Write enable
       │
       ▼
Modify registers
       │
       ▼
Write protect
       │
       ▼
NORMAL OPERATION
```

This minimizes accidental configuration corruption.

---

# 35. Measurement Acquisition Strategy

The ESP32 does not need to continuously read every CSE7761 register.

A sensible acquisition policy is:

```text
Fast loop
─────────
Voltage
Current
Active Power


Medium loop
───────────
Power factor
Frequency
Apparent power


Slow loop
─────────
Energy
Diagnostics
Calibration status
```

For example:

```text
Every ~100 ms:
    read voltage
    read current
    read power

Every ~500 ms:
    read PF
    read frequency

Every ~1-5 s:
    read energy
    publish telemetry
```

The exact cadence should be determined by:

* desired UI responsiveness,
* cloud bandwidth,
* device power consumption,
* measurement update rate,
* backend requirements.

---

# 36. Local Display Architecture

The POWCT uses a TM1621-style LCD controller interface.

The ESP32 therefore has another independent digital subsystem:

```text
ESP32
  │
  ├── CSE7761 UART
  │
  └── TM1621 interface
          │
          ▼
        LCD
```

The display should consume the same normalized measurement model used by the cloud layer.

For example:

```text
CSE7761
   ↓
Measurement object
   ├── voltage
   ├── current
   ├── power
   └── energy
        │
        ├────────► LCD
        │
        └────────► Cloud
```

This avoids duplicating measurement calculations.

---

# 37. Relay and Contactor Architecture

A critical architectural point is that the small onboard relay should not be assumed to carry the full load current.

A robust high-current architecture is:

```text
ESP32
  │
  ▼
Onboard relay
  │
  ▼
Contactor coil
  │
  ▼
Power contactor
  │
  ▼
High-current load
```

This allows:

* low-power logic control,
* galvanic/functional separation,
* high current switching,
* better thermal management,
* replaceable power switching components.

The relay output therefore belongs to the **control domain**, not the measurement domain.

---

# 38. Cloud Architecture

The cloud side can be reconstructed independently of the metrology subsystem.

Conceptually:

```text
                 ESP32
                   │
                   │ Wi-Fi
                   ▼
             Internet
                   │
                   ▼
        ┌────────────────────┐
        │ Device Gateway     │
        │                    │
        │ Authentication     │
        │ Device routing     │
        │ Telemetry          │
        │ Commands           │
        └─────────┬──────────┘
                  │
        ┌─────────┴───────────┐
        │                     │
        ▼                     ▼
 Telemetry database       Command service
        │                     │
        └──────────┬──────────┘
                   ▼
               Mobile App
```

---

# 39. Telemetry Data Model

The ESP32 should convert raw CSE7761 measurements into a device-level telemetry object.

Example:

```json
{
  "timestamp": 1750000000,
  "voltage": 231.4,
  "current": 8.72,
  "active_power": 2010,
  "apparent_power": 2017.8,
  "power_factor": 0.996,
  "frequency": 50.0,
  "energy": 12.84,
  "relay": true
}
```

The cloud does not need to know anything about:

```text
CSE7761
RmsU
RmsIA
PowerPA
RmsUC
RmsIAC
PowerPAC
```

Those are device-internal implementation details.

The cloud should receive semantic measurements.

---

# 40. Separation of Concerns

A recreated architecture should preserve the following boundaries:

```text
┌─────────────────────────────────────────────┐
│ CLOUD                                       │
│ Device telemetry / user state / commands    │
└──────────────────────┬──────────────────────┘
                       │
              Device-level API
                       │
┌──────────────────────▼──────────────────────┐
│ ESP32 APPLICATION                           │
│ Measurement model / state / connectivity    │
└──────────────────────┬──────────────────────┘
                       │
              normalized metrics
                       │
┌──────────────────────▼──────────────────────┐
│ METROLOGY DRIVER                            │
│ Register access / calibration / conversion  │
└──────────────────────┬──────────────────────┘
                       │
                    UART
                       │
┌──────────────────────▼──────────────────────┐
│ CSE7761                                    │
│ ADC / DSP / RMS / power / energy           │
└─────────────────────────────────────────────┘
```

This separation makes the system significantly easier to test.

---

# 41. Recommended Software Interfaces

A clean C/C++ implementation could expose interfaces such as:

```cpp
class CSE7761 {
public:
    bool reset();
    bool readRegister(uint8_t address,
                      uint8_t *data,
                      size_t length);

    bool writeRegister(uint8_t address,
                       const uint8_t *data,
                       size_t length);

    bool initialize();
    bool readCalibration();
};
```

Then a higher-level service:

```cpp
class EnergyMeter {
public:
    bool initialize();

    Measurement readMeasurement();

    float voltage();
    float current();
    float activePower();
    float apparentPower();
    float powerFactor();
    float energy();
    float frequency();
};
```

And application logic:

```cpp
class DeviceController {
public:
    void update();
    void publishTelemetry();
    void updateDisplay();
    void processCommand();
};
```

---

# 42. Error Handling

A recreation should not assume every UART transaction succeeds.

The CSE driver should detect:

```text
UART timeout
Invalid checksum
Unexpected response length
Invalid register data
Out-of-range measurement
Sensor reset
```

Example:

```text
                 UART transaction
                       │
                       ▼
                Response received?
                  /           \
                NO             YES
                │               │
                ▼               ▼
             timeout       checksum valid?
                              /      \
                            NO        YES
                            │          │
                            ▼          ▼
                         retry      decode
                                      │
                                      ▼
                                   validate
                                      │
                                ┌─────┴─────┐
                                │           │
                              valid       invalid
                                │           │
                                ▼           ▼
                            publish      fault
```

---

# 43. Retry Strategy

A simple retry strategy is:

```text
Attempt 1
   │
   ├── success → continue
   │
   └── failure
          ↓
      Attempt 2
          │
          ├── success → continue
          │
          └── failure
                 ↓
             Attempt 3
                 │
                 └── failure → sensor fault
```

The firmware should avoid retry storms.

A failed sensor should not cause:

```text
100% CPU
continuous UART traffic
watchdog reset
network starvation
```

Instead:

```text
FAULT → backoff → retry → recover
```

---

# 44. Measurement Validity

Each measurement should carry validity information.

For example:

```cpp
struct Measurement {
    float voltage;
    float current;
    float activePower;
    float apparentPower;
    float powerFactor;
    float energy;
    float frequency;

    bool voltageValid;
    bool currentValid;
    bool powerValid;
};
```

This is preferable to returning zero for every failure.

Otherwise:

```text
Sensor failure
      ↓
Voltage = 0
Current = 0
Power = 0
      ↓
Cloud assumes genuine zero-load condition
```

which is semantically incorrect.

---

# 45. Calibration Architecture

Calibration should be treated as a first-class subsystem.

The chain is:

```text
Physical system
      │
      ▼
Sensor gain
      │
      ▼
CSE7761 raw count
      │
      ▼
Factory coefficient
      │
      ▼
Engineering unit
      │
      ▼
Application value
```

For example:

```text
CT ratio
   ↓
analog gain
   ↓
ADC response
   ↓
RmsIAC
   ↓
raw RmsIA
   ↓
8.72 A
```

---

# 46. Calibration Types

A complete implementation may require:

### Voltage calibration

```text
Raw RmsU → Volts
```

### Current calibration

```text
Raw RmsIA → Amps
```

### Power calibration

```text
Raw PowerPA → Watts
```

### Energy calibration

```text
Energy register → kWh
```

Calibration should ideally be traceable to known reference equipment.

---

# 47. Energy Measurement

Energy is fundamentally an integration problem:

```text
              t
Energy = ∫ P(t) dt
              0
```

The metrology ASIC performs the accumulation.

The ESP32 therefore does not need to implement:

```text
energy += power × dt
```

for primary metering if it trusts the CSE7761 energy register.

Instead:

```text
CSE7761
   │
   ▼
Energy register
   │
   ▼
ESP32
   │
   ▼
Scale
   │
   ▼
kWh
```

This is preferable because the energy calculation remains synchronized with the metrology engine.

---

# 48. Frequency Measurement

The CSE7761 exposes a voltage-frequency measurement register:

```text
Ufreq = 0x23
```

The frequency is derived approximately from:

```text
f = CLKI / (8 × Ufreq)
```

with the relevant clock approximately:

```text
CLKI ≈ 3.579545 MHz
```

Thus the firmware can derive the mains frequency from the register rather than independently sampling the voltage waveform.

---

# 49. Instantaneous Versus RMS Measurements

The architecture exposes several distinct classes of measurement.

```text
Instantaneous
─────────────
InstanU
InstanIA
InstanP


RMS
───
RmsU
RmsIA


Peak
────
PeakU
PeakIA


Power
─────
PowerPA
PowerS
PowerFactor


Energy
──────
EnergyPA
```

These should not be conflated.

For example:

```text
Instantaneous voltage:
v(t)

RMS voltage:
sqrt(mean(v(t)^2))

Active power:
mean(v(t) × i(t))

Energy:
integral(P(t)dt)
```

---

# 50. End-to-End Timing Model

A useful system timing model is:

```text
                    HIGH SPEED
                       │
                       ▼
                 Analog waveform
                       │
                  ADC / DSP
                  ~kHz domain
                       │
                       ▼
                 Metrology result
                       │
                  ~Hz domain
                       │
                       ▼
                ESP32 polling
                 ~10–100 ms
                       │
                       ▼
                Device telemetry
                 ~100 ms–seconds
                       │
                       ▼
                 Cloud update
                 seconds-scale
                       │
                       ▼
                 Mobile display
```

This explains why a user can see a stable display even though the underlying electrical waveform is changing thousands of times per second.

---

# 51. Full Example Data Flow

Consider a load drawing:

```text
231.4 V
8.72 A
~2.01 kW
```

The complete process is:

```text
STEP 1
──────
Mains voltage creates analog voltage waveform.

STEP 2
──────
Load current passes through CT.

STEP 3
──────
CT produces proportional analog current signal.

STEP 4
──────
CSE7761 digitizes voltage/current.

STEP 5
──────
CSE7761 calculates RMS and active power.

STEP 6
──────
Results are stored in registers.

STEP 7
──────
ESP32 sends:

A5 26

STEP 8
──────
CSE7761 returns:

22 CB 67 E0

STEP 9
──────
ESP32 reconstructs:

RmsU = 0x22CB67

STEP 10
──────
ESP32 applies calibration:

231.4 V

STEP 11
──────
ESP32 repeats for current:

A5 24
→ 15 5D 98 2C
→ 8.72 A

STEP 12
──────
ESP32 reads power:

A5 2C
→ 05 C7 A6 83 39
→ 2010 W

STEP 13
──────
ESP32 updates internal state.

STEP 14
──────
ESP32 updates LCD.

STEP 15
──────
ESP32 publishes telemetry over Wi-Fi.

STEP 16
──────
Cloud stores/routes telemetry.

STEP 17
──────
Mobile application displays:

231.4 V
8.72 A
2.01 kW
```

---

# 52. Architecture for a Recreated System

A clean recreated product could use:

```text
                         ┌─────────────────┐
                         │   Mobile App    │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │ Cloud Backend   │
                         └────────┬────────┘
                                  │
                               MQTT/HTTPS
                                  │
                                  ▼
┌────────────────────────────────────────────────────────────┐
│                         ESP32                              │
│                                                            │
│  ┌────────────┐ ┌────────────┐ ┌────────────────────────┐ │
│  │ Device     │ │ Telemetry  │ │ Command / State        │ │
│  │ Manager    │ │ Manager    │ │ Manager                │ │
│  └─────┬──────┘ └─────┬──────┘ └───────────┬────────────┘ │
│        │              │                    │              │
│        └──────────────┼────────────────────┘              │
│                       │                                   │
│                ┌──────▼──────┐                            │
│                │ Metrology   │                            │
│                │ Service     │                            │
│                └──────┬──────┘                            │
│                       │                                   │
│                ┌──────▼──────┐                            │
│                │ CSE7761     │                            │
│                │ Driver      │                            │
│                └──────┬──────┘                            │
└───────────────────────┼────────────────────────────────────┘
                        │ UART
                        ▼
                  ┌───────────┐
                  │ CSE7761   │
                  └─────┬─────┘
                        │
                ┌───────┴────────┐
                │                │
             Voltage            CT
             sensing          sensing
                │                │
                └───────┬────────┘
                        ▼
                     AC LOAD
```

---

# 53. Development Strategy

The safest development approach is to build the system in layers.

## Phase 1 — CSE7761 Standalone

Ignore Wi-Fi.

Implement:

```text
ESP32
  │
 UART
  │
CSE7761
```

Verify:

* reset,
* register reads,
* checksum,
* voltage,
* current,
* power,
* energy.

---

## Phase 2 — Calibration

Add:

```text
factory coefficients
       ↓
scaling
       ↓
engineering units
```

Compare against calibrated laboratory equipment.

---

## Phase 3 — Local Display

Add:

```text
Measurement object
        │
        ▼
       LCD
```

No cloud yet.

---

## Phase 4 — Relay Control

Add:

```text
GPIO
 ↓
relay
 ↓
contactor
```

Test control independently of metrology.

---

## Phase 5 — Wi-Fi

Add:

```text
ESP32
 ↓
Wi-Fi
 ↓
local network
```

---

## Phase 6 — Cloud

Add:

```text
Telemetry
Commands
Device identity
Authentication
Historical data
```

---

## Phase 7 — Mobile Application

Finally expose:

```text
Voltage
Current
Power
Energy
Status
Control
```

This sequence dramatically simplifies debugging.

---

# 54. Test Strategy

A recreated system should have four independent test domains.

## 54.1 Protocol Tests

Test:

```text
checksum
register addressing
byte ordering
read/write
timeouts
reset
configuration
```

A protocol test can use a UART simulator instead of real mains.

---

## 54.2 Metrology Tests

Use controlled loads and reference instruments.

Test:

```text
Voltage accuracy
Current accuracy
Power accuracy
Power factor
Energy accumulation
Frequency
```

---

## 54.3 Connectivity Tests

Test:

```text
Wi-Fi loss
router restart
Internet loss
cloud unavailable
reconnect
credential changes
```

---

## 54.4 System Tests

Test:

```text
Power-on
Power-off
Brownout
Sensor failure
Network failure
Relay switching
Long-duration operation
```

---

# 55. Fault Containment

The architecture should ensure that failures remain local.

```text
CSE7761 failure
     ↓
Metrology unavailable
     │
     ├── LCD → error indication
     ├── Cloud → sensor fault
     └── Control → safe state
```

A cloud outage should not prevent local measurement:

```text
Cloud failure
     ↓
ESP32 continues
     │
     ├── measurement
     ├── display
     └── local control
```

Similarly:

```text
Wi-Fi failure
     ↓
No telemetry
     │
     └── device remains locally functional
```

This is an important architectural principle:

> **Connectivity should be an enhancement to the device, not a prerequisite for basic device safety and measurement.**

---

# 56. Security Architecture

A production implementation should separate:

```text
Device identity
        │
        ▼
Authentication
        │
        ▼
Encrypted transport
        │
        ▼
Cloud authorization
        │
        ▼
Command validation
```

Commands should not directly manipulate GPIOs from network packets.

Instead:

```text
Network command
      ↓
Parse
      ↓
Authenticate
      ↓
Authorize
      ↓
Validate
      ↓
Application command
      ↓
Relay manager
      ↓
GPIO
```

---

# 57. Why This Architecture Scales

The architecture is highly reusable.

The metrology subsystem can be treated as a reusable platform:

```text
               ┌─────────────────────┐
               │  Metrology Platform │
               │                     │
               │ CSE7761             │
               │ Calibration         │
               │ Measurements        │
               └─────────┬───────────┘
                         │
             ┌───────────┼────────────┐
             ▼           ▼            ▼
          Smart       Industrial    Energy
          switch      monitor       gateway
```

Similarly, the ESP32 platform can be reused for:

```text
Energy meter
Smart relay
EV charger monitor
Solar monitor
Heat-pump monitor
Industrial load monitor
```

Only the sensing and application layer needs to change.

---

# 58. Key Engineering Insights

## Insight 1 — Separate Metrology From Application

The CSE7761 is a dedicated metrology engine.

The ESP32 should not duplicate its job.

```text
CSE7761 = "measure"
ESP32   = "manage"
Cloud   = "serve"
App     = "present"
```

---

## Insight 2 — Raw Registers Are Not Engineering Values

A register such as:

```text
RmsU = 0x22CB67
```

is not:

```text
231.4 V
```

until the calibration model is applied.

Therefore:

```text
raw register
      ↓
signedness / validity
      ↓
calibration coefficient
      ↓
scaling
      ↓
engineering unit
```

---

## Insight 3 — The CSE7761 Is Doing DSP

The system is not:

```text
ADC → ESP32 → V × I
```

It is:

```text
ADC → DSP/metrology ASIC → RMS/power/energy
                             ↓
                           UART
                             ↓
                           ESP32
```

That is a substantially more sophisticated architecture.

---

## Insight 4 — Telemetry Is a Separate Rate Domain

The electrical waveform may be processed at approximately kHz rates.

The user interface operates at approximately Hz rates.

The cloud operates at an even slower application timescale.

Therefore:

```text
kHz → Hz → sub-Hz/seconds
```

is a natural architecture.

---

## Insight 5 — Factory Calibration Is Part of the Device

The CSE7761 calibration coefficients should be considered part of the physical device identity.

Two otherwise identical boards can have different calibration coefficients.

Therefore:

```text
Device
 ├── Hardware
 ├── CSE7761
 └── Calibration data
```

rather than:

```text
One universal conversion constant
```

---

# 59. Minimal Reimplementation

A minimum viable recreation does not need the complete cloud stack.

The smallest useful implementation is:

```text
AC voltage/current
       │
       ▼
    CSE7761
       │
      UART
       │
       ▼
     ESP32
       │
       ▼
 Serial console

231.4 V
8.72 A
2010 W
```

The development progression can then be:

```text
                 MVP

CSE7761
   │
 UART
   │
 ESP32
   │
 Serial output


                  ↓


               Device

CSE7761
   │
 ESP32 ──────► LCD
   │
   └─────────► Relay


                  ↓


              Connected

CSE7761
   │
 ESP32 ──────► LCD
   │
   ├─────────► Relay
   │
   └─────────► Wi-Fi
                   │
                   ▼
                 Cloud
                   │
                   ▼
                  App
```

---

# 60. Recommended Repository Structure

A recreated implementation could be organized as:

```text
powct-reimplementation/
│
├── firmware/
│   ├── main.cpp
│   │
│   ├── hal/
│   │   ├── uart.cpp
│   │   ├── gpio.cpp
│   │   ├── wifi.cpp
│   │   └── storage.cpp
│   │
│   ├── cse7761/
│   │   ├── cse7761.cpp
│   │   ├── cse7761.h
│   │   ├── registers.h
│   │   ├── protocol.cpp
│   │   └── calibration.cpp
│   │
│   ├── metrology/
│   │   ├── energy_meter.cpp
│   │   ├── energy_meter.h
│   │   └── measurement.h
│   │
│   ├── display/
│   │   ├── tm1621.cpp
│   │   └── display_manager.cpp
│   │
│   ├── relay/
│   │   └── relay_manager.cpp
│   │
│   ├── connectivity/
│   │   ├── wifi_manager.cpp
│   │   ├── telemetry.cpp
│   │   └── command_manager.cpp
│   │
│   └── device/
│       ├── device_state.cpp
│       └── watchdog.cpp
│
├── cloud/
│   ├── device_gateway/
│   ├── telemetry/
│   ├── commands/
│   └── database/
│
├── app/
│
├── tests/
│   ├── protocol/
│   ├── calibration/
│   ├── metrology/
│   └── integration/
│
└── docs/
    └── architecture.md
```

---

# 61. Protocol Test Vectors

The following test vectors are useful for validating the implementation.

## Reset

```text
TX:
A5 EA 96 DA
```

---

## Write Enable

```text
TX:
A5 EA E5 8B
```

---

## Write Protect

```text
TX:
A5 EA DC 94
```

---

## Read Voltage

```text
TX:
A5 26
```

Example response:

```text
22 CB 67 E0
```

Expected raw value:

```text
0x22CB67
```

---

## Read Current

```text
TX:
A5 24
```

Example response:

```text
15 5D 98 2C
```

Expected raw value:

```text
0x155D98
```

---

## Read Active Power

```text
TX:
A5 2C
```

Example response:

```text
05 C7 A6 83 39
```

Expected raw value:

```text
0x05C7A683
```

---

## Example Register Write

```text
Register:
0x02

Value:
0x1234

TX:
A5 82 12 34 92
```

---

# 62. Architecture Summary

The complete system can be reduced to one diagram:

```text
                         ┌──────────────────────────┐
                         │       MOBILE APP         │
                         │                          │
                         │ V / A / W / kWh / state │
                         └────────────┬─────────────┘
                                      │
                                      ▼
                         ┌──────────────────────────┐
                         │      CLOUD BACKEND       │
                         │                          │
                         │ telemetry / commands     │
                         └────────────┬─────────────┘
                                      │
                                  Internet
                                      │
                                      ▼
┌────────────────────────────────────────────────────────────────┐
│                            ESP32                               │
│                                                                │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────────┐ │
│  │ Metrology │ │ Display   │ │ Relay     │ │ Connectivity  │ │
│  │ Manager   │ │ Manager   │ │ Manager   │ │ Manager       │ │
│  └─────┬─────┘ └─────┬─────┘ └─────┬─────┘ └───────┬───────┘ │
│        │              │             │               │         │
└────────┼──────────────┼─────────────┼───────────────┼─────────┘
         │              │             │               │
        UART           GPIO          GPIO            Wi-Fi
         │              │             │               │
         ▼              ▼             ▼               ▼
    ┌─────────┐       LCD          Relay           Router
    │CSE7761  │         │             │
    └────┬────┘         │             ▼
         │                         Contactor
         │                             │
    ┌────┴─────┐                       │
    │          │                       ▼
 Voltage      CT                      LOAD
 sensing    sensing
    │          │
    └────┬─────┘
         │
       MAINS
```

---

# 63. Final Engineering Model

The entire product can ultimately be understood as five transformations:

```text
1. PHYSICAL
   Electricity
       ↓

2. METROLOGY
   Analog waveforms
       ↓
   CSE7761
       ↓
   Raw digital measurements

3. EMBEDDED
   UART
       ↓
   ESP32
       ↓
   Calibration
       ↓
   Engineering values

4. CONNECTIVITY
   Wi-Fi
       ↓
   Cloud telemetry

5. USER EXPERIENCE
   Mobile application
       ↓
   "231.4 V / 8.72 A / 2.01 kW"
```

The central architectural principle is therefore:

> **Do not recreate the product as one system. Recreate it as a chain of bounded systems with explicit interfaces.**

```text
┌──────────────┐
│ ELECTRICAL   │
└──────┬───────┘
       │ analog
       ▼
┌──────────────┐
│ METROLOGY    │
│ CSE7761      │
└──────┬───────┘
       │ UART
       ▼
┌──────────────┐
│ EMBEDDED     │
│ ESP32        │
└──────┬───────┘
       │ IP
       ▼
┌──────────────┐
│ CLOUD        │
└──────┬───────┘
       │ API
       ▼
┌──────────────┐
│ APPLICATION  │
└──────────────┘
```

Once these boundaries are understood, the apparent complexity of the product becomes manageable.

The CSE7761 solves the hard **electrical metrology** problem.

The ESP32 solves the **embedded orchestration and connectivity** problem.

The cloud solves the **device fleet and telemetry** problem.

The application solves the **human interaction** problem.

That separation is the key to reproducing the system cleanly, testing it systematically, and extending it into a new product architecture.
