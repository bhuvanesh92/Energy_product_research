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

---

# 64. Experimental Firmware and Reverse-Engineering Plan

## 64.1 Purpose and Scope

The project has two related but distinct goals:

1. **Near-term goal — use an existing SONOFF POW Ring as an experimental platform.**  
   Replace or augment the stock firmware sufficiently to read metrology data, observe device behavior, and run controlled load-management algorithms.

2. **Long-term goal — create an independently engineered POW Ring-class system.**  
   Recreate the relevant architecture from first principles: current measurement, voltage measurement, metrology, ESP32 firmware, local control, telemetry, and safe control of an external contactor.

The first goal is not merely a shortcut to the second. It reduces technical uncertainty before custom hardware is designed. The existing SONOFF unit provides a real enclosure, mature mains-side sensing, CT interface, metering ASIC, display, relay/control output, power supply, and ESP32-based controller. It can therefore serve as a hardware-in-the-loop prototype for algorithm development.

The intended progression is:

```text
Existing SONOFF POW Ring
        │
        ▼
Non-destructive investigation
        │
        ▼
Firmware backup and hardware mapping
        │
        ▼
Minimal replacement firmware
        │
        ▼
Metrology validation
        │
        ▼
Algorithm experimentation
        │
        ▼
Independent POW Ring-class design
```

The objective is **not** to bypass device security, defeat access controls, or distribute modified vendor firmware. The objective is to understand the architecture, preserve the original state, develop original firmware, and use the owned hardware as a controlled research platform.

---

## 64.2 Two-Track Project Model

The work should be organized as two explicit tracks.

| Track | Objective | Primary Output | Main Risk |
| --- | --- | --- | --- |
| Track A — Experimental SONOFF Platform | Use the original device to test metrology handling and control algorithms | Original ESP32 firmware replaced only after backup and validation | Incorrect GPIO mapping, unsafe mains handling, loss of recoverability |
| Track B — Independent Reimplementation | Build a functionally similar architecture from first principles | Custom hardware, firmware, telemetry, and control architecture | Electrical safety, calibration accuracy, production-level reliability |

The tracks share most of the software architecture:

```text
CSE7761 driver
      │
      ▼
Metrology service
      │
      ▼
Measurement model
      │
      ├──► Local display
      ├──► Telemetry
      ├──► Data logging
      └──► Control algorithm
                  │
                  ▼
            Actuator manager
                  │
                  ▼
          Relay / contactor output
```

The experimental device should be treated as a reference hardware platform. The independent implementation should eventually replace each vendor-specific dependency with an explicitly designed equivalent.

---

## 64.3 Safety Boundary

The POW Ring is a mains-connected product. Low-voltage firmware work and mains-side electrical work must be treated as separate engineering domains.

The ESP32 UART pins are low-voltage signals, but the PCB ground reference may not be safe to connect to a grounded computer, oscilloscope, USB hub, or bench instrument unless the power-supply isolation and board grounding architecture have been verified.

The initial rule is:

> **Do not attach a PC, USB-UART adapter, logic analyser, or oscilloscope ground lead to a live mains-powered board until the isolation and ground relationship are understood.**

The preferred development sequence is:

```text
Initial investigation
        │
        ▼
Board disconnected from mains
        │
        ▼
Low-voltage-only access where possible
        │
        ▼
UART identification and firmware backup
        │
        ▼
Bench validation of replacement firmware
        │
        ▼
Mains-connected testing only with an appropriate safety setup
```

For mains-connected experiments:

- Use an enclosure and prevent accidental contact with mains-side conductors.
- Treat all mains-side terminals, PCB regions, and connected conductors as hazardous when energized.
- Do not use a non-isolated USB-UART adapter or grounded oscilloscope connection unless the board-side ground is known to be safely isolated.
- Prefer appropriately isolated instrumentation where live diagnostics are unavoidable.
- Use conservative current limits and a known resistive load for initial behavior verification.
- Keep relay or contactor actuation disabled by default until measurement, GPIO polarity, and fail-safe behavior are verified.
- Do not rely on firmware alone as the safety mechanism for a mains-connected load.

This document remains architectural guidance, not a mains wiring or certification guide.

---

## 64.4 ESP32 Development Access

The expected firmware-development interface is the ESP32 ROM serial bootloader, accessed through a USB-to-3.3 V TTL UART adapter.

The relevant connection is:

```text
Development PC
      │
      │ USB
      ▼
USB-to-UART adapter
      │
      │ 3.3 V UART
      ▼
ESP32 ROM bootloader / application UART
```

This is distinct from the internal ESP32-to-CSE7761 UART:

```text
ESP32 UART for host development
      ≠
ESP32 UART used to communicate with CSE7761
```

The CSE7761 UART is part of the metrology subsystem. Connecting a USB-UART adapter to the CSE7761 interface will not provide ESP32 bootloader access.

A suitable adapter must use **3.3 V logic levels**. Common adapter chip families include:

```text
FTDI
CP2102 / CP210x
CH340
```

The conceptual connection is:

```text
USB-UART adapter                  ESP32

TX  ───────────────────────────►  RX0 / U0RXD
RX  ◄───────────────────────────  TX0 / U0TXD
GND ────────────────────────────  GND
```

The TX and RX lines are crossed:

```text
Adapter TX → ESP32 RX
Adapter RX ← ESP32 TX
Adapter GND ↔ ESP32 GND
```

Do not apply 5 V logic levels to ESP32 UART pins.

Before soldering wires or probing pads, identify the exact ESP32 module or chip, board revision, UART pads or test points, reset/enable signal, and boot strap signal. The GPIO assignments documented elsewhere in this file are useful hypotheses, not a substitute for board-specific verification.

---

## 64.5 ESP32 Bootloader Entry

Classic ESP32 devices include a ROM serial bootloader. The boot mode is selected during reset.

For the original ESP32 family, holding `GPIO0` low while resetting the chip selects serial download mode:

```text
GPIO0 = LOW during reset
        │
        ▼
ESP32 ROM serial bootloader
        │
        ▼
esptool can communicate with the chip
```

Normal application boot is:

```text
GPIO0 = HIGH or released during reset
        │
        ▼
Boot application from external SPI flash
```

The existing board mapping indicates that `GPIO0` may be connected to the physical push button. This creates a plausible manual bootloader-entry method, but it must be confirmed on the actual unit.

A conservative manual procedure is:

```text
1. Hold GPIO0 low.
2. Reset the ESP32 through EN/reset or a controlled power cycle.
3. Keep GPIO0 low during reset release.
4. Release GPIO0 after the ROM bootloader has started.
5. Connect with esptool.
```

Conceptually:

```text
GPIO0 held low
       │
       ▼
ESP32 reset
       │
       ▼
ROM bootloader starts
       │
       ▼
GPIO0 released
       │
       ▼
Serial flashing session
```

If the board exposes both boot and reset signals, a USB-UART adapter with DTR and RTS can potentially automate this process. This should only be attempted after the board signals have been identified, because incorrect connections can interfere with boot strapping or reset behavior.

---

## 64.6 Preserve the Original State First

No erase, flash, or configuration change should be performed before the original device state has been documented and backed up.

The mandatory order is:

```text
IDENTIFY
    ↓
DOCUMENT
    ↓
BACK UP
    ↓
VERIFY BACKUP
    ↓
ANALYZE
    ↓
BUILD REPLACEMENT FIRMWARE
    ↓
FLASH ONLY WHEN READY
```

The prohibited development pattern is:

```text
ERASE
   ↓
FLASH UNKNOWN IMAGE
   ↓
LOSE ORIGINAL REFERENCE
```

The original flash image may contain useful information even if it cannot be directly reused:

```text
Bootloader configuration
Partition table
Application image layout
NVS layout
Wi-Fi and provisioning behavior
GPIO initialization
Display initialization
Relay polarity and defaults
CSE7761 configuration
Calibration handling
Fault handling
Watchdog behavior
Factory-test artifacts
```

A suggested evidence directory is:

```text
evidence/
├── board_photos/
├── board_revision.md
├── pin_mapping.md
├── uart_notes.md
├── flash_backup/
│   ├── original_flash.bin
│   ├── original_flash.sha256
│   ├── flash_metadata.txt
│   └── backup_notes.md
├── captured_uart/
├── logic_analyzer/
└── test_results/
```

The backup should be considered immutable. Copy it to at least two independent storage locations before experimentation begins.

---

## 64.7 Initial esptool Workflow

The preferred host tool is `esptool`, used either directly or indirectly through ESP-IDF.

The first task is to establish communication with the ESP32 ROM bootloader.

Example commands:

```bash
esptool --port <PORT> chip-id
```

Examples of serial-port naming:

```text
Windows:  COM3, COM4, COM5
macOS:    /dev/cu.usbserial-XXXX
macOS:    /dev/cu.SLAB_USBtoUART
Linux:    /dev/ttyUSB0
Linux:    /dev/ttyACM0
```

After communication is established, record the available chip information:

```text
Chip model
Chip revision
MAC address
Crystal frequency
Detected flash size
Flash mode
Flash voltage
```

A full flash read can then be performed using the detected size or `ALL` where supported:

```bash
esptool --port <PORT> read-flash 0 ALL original_flash.bin
```

A fixed-size read can also be used after flash capacity has been established. For example, a 2 MiB flash would be read as:

```bash
esptool --port <PORT> read-flash 0 0x200000 original_flash.bin
```

Afterward, calculate and store a SHA-256 hash:

```bash
sha256sum original_flash.bin
```

On macOS:

```bash
shasum -a 256 original_flash.bin
```

The hash should be written to a file and included in the experiment log.

Example:

```text
File: original_flash.bin
SHA-256: <recorded hash>
Device: <serial number or label>
Board revision: <observed revision>
Date: <date>
Method: ESP32 ROM bootloader via 3.3 V UART
```

A successful flash read proves only that the contents were read. It does not prove that the resulting binary is directly interpretable or reusable. Secure boot, flash encryption, readout protection, or vendor-specific layouts may limit what can be recovered from it.

---

## 64.8 Security and Recoverability Checks

Before replacing any firmware, inspect the target for security features and recovery constraints.

Potential conditions include:

```text
No secure boot and no flash encryption
      │
      └── Conventional backup / replacement workflow may be possible

Secure boot enabled
      │
      └── Unsigned replacement firmware may not boot

Flash encryption enabled
      │
      └── Physical flash readout may not yield useful plaintext firmware

JTAG disabled
      │
      └── Hardware debug access may be unavailable

Custom bootloader or partition scheme
      │
      └── Replacement image must match the expected boot and partition layout
```

An experimental replacement firmware should never assume that the vendor partition table is compatible with a newly built ESP-IDF application.

The safer alternatives are:

- Preserve the existing image and its layout before making changes.
- Start with a minimal known-good ESP-IDF image only after recoverability is understood.
- Use a dedicated partition table for the replacement firmware.
- Keep an explicit recovery procedure and the commands needed to restore the original image.
- Test the recovery procedure on a non-critical unit where possible.

A flash image read through the ROM bootloader may not be sufficient to reconstruct plaintext firmware if flash encryption is enabled. Similarly, secure boot can prevent arbitrary unsigned images from executing. These conditions should be treated as architectural constraints, not obstacles to be bypassed.

---

## 64.9 Firmware Strategy

There are two possible firmware approaches.

### Approach A — Patch or modify vendor firmware

```text
Original firmware
       │
       ▼
Disassemble and analyze
       │
       ▼
Find control logic
       │
       ▼
Patch binary
       │
       ▼
Reflash
```

This approach has significant disadvantages:

```text
Unknown internal architecture
Potential secure boot
Potential flash encryption
Unknown integrity checks
Vendor cloud dependencies
Unknown OTA behavior
Poor maintainability
Hard-to-reproduce modifications
```

### Approach B — Build original replacement firmware

```text
Known hardware
       │
       ▼
Original ESP-IDF firmware
       │
       ▼
CSE7761 driver
       │
       ▼
Measurement model
       │
       ▼
Control algorithm
       │
       ▼
Display / telemetry / actuator control
```

For this project, the preferred approach is **Approach B**.

The existing SONOFF POW Ring is used as a hardware reference and test target. The replacement firmware should be original, modular, version-controlled, and designed so that its hardware-dependent portions can later be moved into an independent POW Ring-class product.

---

## 64.10 Recommended ESP-IDF Architecture

ESP-IDF is preferred over a simplified framework because this project requires explicit control over:

```text
FreeRTOS task structure
UART configuration
GPIO configuration
Watchdogs
Timers
NVS
Wi-Fi provisioning
Logging
Partition tables
OTA update handling
Fault containment
```

The software boundary should be:

```text
┌───────────────────────────────────────────────┐
│                 Application                    │
│  Algorithm experiments / policies / commands  │
└──────────────────────┬────────────────────────┘
                       │
┌──────────────────────▼────────────────────────┐
│              Control / Actuator API            │
│  Safe enable, inhibit, state, interlocks       │
└──────────────────────┬────────────────────────┘
                       │
┌──────────────────────▼────────────────────────┐
│              Measurement Service               │
│  Engineering units, validity, filtering        │
└──────────────────────┬────────────────────────┘
                       │
┌──────────────────────▼────────────────────────┐
│                 CSE7761 Driver                 │
│  UART packets, register access, checksums      │
└──────────────────────┬────────────────────────┘
                       │
┌──────────────────────▼────────────────────────┐
│                     HAL                        │
│  UART, GPIO, timers, NVS, Wi-Fi, watchdog      │
└───────────────────────────────────────────────┘
```

The control algorithm must not access CSE7761 UART registers directly. It should operate only on normalized, timestamped measurements and command an actuator through a controlled interface.

Example:

```cpp
Measurement measurement = energy_meter.read();

ControlCommand command =
    controller.update(measurement);

actuator_manager.apply(command);
```

This separation allows the control algorithm to be tested without hardware.

```text
Recorded measurements
        │
        ▼
Host-side simulation
        │
        ▼
Controller output
        │
        ▼
Compare policy variants
        │
        ▼
Deploy selected version to ESP32
```

---

## 64.11 Experimental Control Design

The purpose of the SONOFF device in the first phase is to validate algorithms, not to immediately automate a high-consequence load.

The actuator path should be modeled explicitly:

```text
Control algorithm
       │
       ▼
Control request
       │
       ▼
Safety and policy checks
       │
       ▼
Actuator manager
       │
       ▼
GPIO output
       │
       ▼
Onboard relay
       │
       ▼
External contactor or controlled load
```

The actuator manager should own the following concerns:

```text
Output polarity
Startup default state
Minimum on-time
Minimum off-time
Switching-rate limits
Manual inhibit
Fault inhibit
Communication-loss behavior
Watchdog behavior
State reporting
```

A control algorithm should request a state; it should not write a relay GPIO directly.

For example:

```cpp
enum class DesiredLoadState {
    OFF,
    ON
};

struct ControlCommand {
    DesiredLoadState desired_state;
    const char *reason;
};

ControlCommand command =
    controller.update(measurement);

actuator_manager.apply(command);
```

A safe initial policy is measurement-only operation:

```text
Algorithm runs
      │
      ▼
Decision is logged
      │
      ▼
Relay remains physically inhibited
```

This enables evaluation of decisions before any physical switching occurs.

---

## 64.12 Algorithm Experimentation Modes

The experimental firmware should support several operating modes.

| Mode | Relay Output | Purpose |
| --- | --- | --- |
| Observe | Forced unchanged or disabled | Validate measurements, timing, logs, and algorithm decisions |
| Shadow | No physical control; command is logged | Compare proposed action with actual device state |
| Manual | Relay state changed only by explicit local/API command | Validate output polarity and contactor behavior |
| Limited automatic | Algorithm may switch within explicit constraints | Controlled low-risk experiments |
| Full automatic | Algorithm controls the output according to configured policy | Only after safety and reliability validation |

The recommended progression is:

```text
Observe
   ↓
Shadow
   ↓
Manual
   ↓
Limited automatic
   ↓
Full automatic
```

Do not begin with unrestricted autonomous switching.

Useful first algorithms include:

```text
Threshold control
Hysteresis control
Debounced threshold control
Time-window control
Minimum on/off duration control
Moving-average control
Peak limiting
Energy-budget control
Load-shedding policy
Price-aware scheduling
PV-surplus response
```

A basic hysteresis controller can be expressed as:

```text
If power remains above upper threshold:
    request OFF

If power remains below lower threshold:
    request ON
```

where:

```text
lower threshold < upper threshold
```

This avoids repeated relay switching when the measured value fluctuates near one threshold.

A more robust implementation adds time qualification:

```text
If power > upper threshold continuously for T_off:
    request OFF

If power < lower threshold continuously for T_on:
    request ON
```

and actuator constraints:

```text
Do not switch ON more often than once per minimum_off_time.
Do not switch OFF more often than once per minimum_on_time.
Do not switch when measurement validity is false.
Do not automatically re-enable after a fault without explicit policy.
```

---

## 64.13 Measurement Model for Algorithms

All control algorithms should receive a stable application-level model rather than raw register values.

Example:

```cpp
struct Measurement {
    float voltage_V;
    float current_A;
    float active_power_W;
    float apparent_power_VA;
    float power_factor;
    float frequency_Hz;
    float energy_kWh;

    bool voltage_valid;
    bool current_valid;
    bool power_valid;
    bool energy_valid;

    uint64_t timestamp_ms;
};
```

The metrology service should perform:

```text
UART transaction
      │
      ▼
Checksum validation
      │
      ▼
Byte reconstruction
      │
      ▼
Signedness handling
      │
      ▼
Calibration scaling
      │
      ▼
Range validation
      │
      ▼
Measurement object
```

The control layer should then enforce a fundamental rule:

```text
Invalid measurement
       │
       ▼
No automatic switching decision
       │
       ▼
Enter configured safe behavior
```

This avoids treating a UART timeout, corrupt packet, or metering failure as a genuine zero-power condition.

---

## 64.14 Minimal Firmware Milestones

The firmware should be introduced in small, recoverable increments.

### Milestone 0 — Hardware evidence

```text
Photograph PCB
Identify ESP32 variant
Identify UART access points
Identify EN/reset and GPIO0
Identify power and ground domains
Record board revision
```

### Milestone 1 — ROM bootloader access

```text
Enter download mode
Run chip identification
Detect flash parameters
Record serial connection settings
```

### Milestone 2 — Backup and recovery plan

```text
Read complete flash
Hash backup
Store metadata
Inspect partition table
Write restoration procedure
```

### Milestone 3 — Minimal replacement firmware

```text
Boot ESP-IDF application
Serial logging works
Watchdog configured
No relay actuation
No mains-side experiment required
```

### Milestone 4 — GPIO discovery

```text
Confirm LED behavior
Confirm button input
Confirm display signals
Keep relay path disabled or inhibited
```

### Milestone 5 — CSE7761 communication

```text
Receive valid UART responses
Validate checksums
Read raw voltage/current/power registers
Read calibration coefficients
```

### Milestone 6 — Measurement validation

```text
Convert to engineering units
Compare against reference instrumentation
Check timing and stability
Record error across load range
```

### Milestone 7 — Shadow control algorithms

```text
Run algorithm
Log proposed state changes
Do not actuate load
Evaluate false positives and switching frequency
```

### Milestone 8 — Manual and constrained switching

```text
Verify relay polarity
Verify contactor behavior
Use low-risk test load
Apply minimum on/off intervals
Confirm fault behavior
```

### Milestone 9 — Connectivity and OTA

```text
Add Wi-Fi provisioning
Add local telemetry
Add authenticated update path
Preserve serial recovery path
```

---

## 64.15 Validation Criteria

Each phase should have explicit exit criteria.

| Area | Minimum Criterion |
| --- | --- |
| Bootloader access | Chip identification succeeds repeatedly |
| Backup | Flash dump completes; SHA-256 recorded; backup copied independently |
| UART | CSE7761 responses pass checksum validation consistently |
| Metrology | Voltage, current, and active power are plausible and match a reference within defined tolerance |
| Measurement validity | UART timeout, corrupt packet, and out-of-range values produce explicit invalid states |
| Relay interface | Output polarity and startup state are confirmed without an uncontrolled mains load |
| Algorithm | Shadow-mode logs demonstrate stable decisions and acceptable switching frequency |
| Fault behavior | Sensor failure, firmware restart, Wi-Fi loss, and controller fault lead to the defined safe state |
| Recovery | Original image restoration procedure is documented and has been reviewed before destructive steps |

For every experiment, record:

```text
Firmware Git commit
Board identifier
Configuration version
Test mode
Load type
Reference measurement
Observed behavior
Expected behavior
Pass/fail result
Notes and anomalies
```

---

## 64.16 Data Logging and Replay

Algorithm development benefits substantially from separating data collection from actuation.

The ESP32 should be able to publish or log a time series such as:

```json
{
  "timestamp_ms": 0,
  "voltage_V": 231.4,
  "current_A": 8.72,
  "active_power_W": 2010.0,
  "apparent_power_VA": 2017.8,
  "power_factor": 0.996,
  "frequency_Hz": 50.0,
  "energy_kWh": 12.84,
  "measurement_valid": true,
  "relay_actual": false,
  "relay_requested": false,
  "controller_reason": "below_threshold"
}
```

This supports an efficient development loop:

```text
Physical SONOFF device
        │
        ▼
Capture time-series measurements
        │
        ▼
Store as CSV / JSON
        │
        ▼
Replay in host-side controller tests
        │
        ▼
Tune algorithm parameters
        │
        ▼
Deploy new firmware
        │
        ▼
Run in shadow mode
        │
        ▼
Compare predicted and observed behavior
```

This is particularly important for algorithms that depend on timing, hysteresis, moving averages, peak detection, or load cycles.

---

## 64.17 From Experimental Device to Independent Product

The experimental SONOFF device should gradually become a reference implementation rather than a permanent dependency.

The migration path is:

```text
Phase 1
Existing SONOFF hardware + original experimental firmware
        │
        ▼
Phase 2
Hardware abstraction isolates board-specific details
        │
        ▼
Phase 3
Independent ESP32 + CSE7761 evaluation platform
        │
        ▼
Phase 4
Custom sensing, power, display, and control hardware
        │
        ▼
Phase 5
POW Ring-class independent product architecture
```

The reusable parts should include:

```text
CSE7761 UART protocol implementation
Calibration handling
Measurement model
Fault handling
Control algorithms
Actuator safety state machine
Telemetry schema
Data logging
Host-side simulation tests
OTA architecture
```

The board-specific parts should remain isolated:

```text
GPIO assignments
Display wiring
LED wiring
Button wiring
Relay polarity
Power-supply behavior
Contactor interface
Board revision quirks
```

The target architecture is therefore:

```text
┌──────────────────────────────────────────────┐
│               Reusable Product Software       │
│                                              │
│  Metrology / control / telemetry / OTA       │
└─────────────────────┬────────────────────────┘
                      │
              Hardware abstraction
                      │
         ┌────────────┴────────────┐
         ▼                         ▼
Experimental SONOFF board   Independent custom board
```

This ensures that experimentation on the SONOFF platform produces durable engineering assets rather than a one-off firmware modification.

---

## 64.18 Practical Decision

The recommended immediate strategy is:

```text
1. Obtain safe, verified ESP32 serial access.
2. Identify the chip and create an immutable full-flash backup.
3. Document the board-specific pin mapping and safety assumptions.
4. Build a minimal ESP-IDF firmware with serial logging only.
5. Reconstruct CSE7761 communication and validate metrology.
6. Run control algorithms first in observe and shadow modes.
7. Introduce constrained physical switching only after actuator behavior and fault handling are verified.
8. Keep the firmware architecture portable so it can become the software basis of an independent POW Ring-class implementation.
```

The SONOFF POW Ring is therefore not the final product. It is the first hardware-in-the-loop platform for validating the metrology pipeline, control strategy, observability model, failure handling, and energy-management algorithms that will later be transferred into an independently engineered system.
