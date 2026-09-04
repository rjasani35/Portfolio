# Rohan Jasani — Hardware & Circuits Portfolio

Electrical engineering portfolio covering custom IC design, PCB design, embedded firmware, and power electronics. This repository collects full design write-ups (schematics, layouts, simulation results, and design rationale) for each project below.

**Contact:** [rohanjasani35@gmail.com](mailto:rohanjasani35@gmail.com) | [LinkedIn](https://linkedin.com/in/rohan-jasani-a51513311) | [GitHub](https://github.com/rjasani35)

---

## About Me

I'm an EE student at UC San Diego, currently pursuing an M.S. in Electrical Engineering (Sept 2026 – Dec 2027) with a concentration in Mixed-Signal and RF Integrated Circuit Design, following a B.S. in Electrical Engineering (Circuits and Computer System Design). My work spans full-custom analog/digital IC design in Cadence, multi-layer PCB design in KiCad, and embedded firmware for real-time data acquisition — most recently as a Circuit Validation & Embedded Systems Engineering Intern at Neuro Leap and as an Avionics Engineer with the UCSD Rocket Propulsion Laboratory. I will soon be taking a graduate advisor role in RPL and working at Triton Droids in which I will be developing hardware control and power distribution systems for a biped robot. 

---

## Projects

| Project                                                             | Focus                              | Tools                              | Highlight                                                                                                           |
| :------------------------------------------------------------------ | :--------------------------------- | :--------------------------------- | :------------------------------------------------------------------------------------------------------------------ |
| [8-Bit Carry-Lookahead Adder](./8%20Bit%20CLA/)                     | Full-custom digital IC design      | Cadence Virtuoso, 45nm CMOS, SPICE | 2.8 GHz fmax — a 27.3% speed-up over a synthesized baseline                                                         |
| [STM32F103 Custom MCU Board](./STM32%20PCB/)                        | Embedded systems PCB               | KiCad, STM32F103C8T6               | Native USB, USART/I2C/SWD breakout, single-supply 3.3V regulation                                                   |
| [4-Layer USB-C Variable Power Supply](./Variable%20Power%20Supply/) | Power electronics PCB              | KiCad, CH224Q                      | Negotiates 5V–28V from USB-C PD at up to 2A                                                                         |
| [Rocket DAQ PCB](./Rocket%20DAQ%20PCB/) *(write-up in progress)*    | Flight avionics / embedded systems | KiCad, ESP32, FreeRTOS             | 4-layer in-flight telemetry logger with auto USB/LiPo power switching and redundant dual-stage parachute deployment |

---

## 8-Bit Custom Carry-Lookahead Adder (CLA)

Full-custom, transistor-level 8-bit Carry-Lookahead Adder designed in a 45nm CMOS process, built to beat the delay of an automated place-and-route ripple-carry baseline.

- Hierarchical CLA architecture: two cascaded 4-bit lookahead blocks compute propagate/generate signals ahead of time, so the second block's carry-in is available almost immediately from the first.
- Every gate on the critical path was hand-designed and sized rather than pulled from a standard cell library, using low-Vth devices to maximize drive current.
- The 1-bit full adder is a single-stage static CMOS mirror adder rather than a chain of XOR/AND/OR gates, minimizing transistor count and internal node capacitance.
- **Result:** 2.8 GHz max frequency at 1.1V VDD, a 600 MHz (27.3%) improvement over the 2.2 GHz synthesized ripple-carry baseline, at the cost of higher power draw from the low-Vth devices.

[Full write-up →](/8%20Bit%20CLA/)

---

## STM32F103 Custom Microcontroller Board

Self-contained breakout board for the STM32F103C8T6 (ARM Cortex-M3), built for rapid hardware prototyping.

- Native USB Full-Speed via the MCU's internal USB peripheral (no external UART-to-USB bridge), with USART1, I2C2, and SWD broken out to 4-pin headers.
- Micro-USB input regulated to 3.3V through an AMS1117-3.3 LDO, with a dedicated LC-filtered analog supply (ferrite bead + capacitor network) isolating the ADC rail from digital switching noise.
- Hardware BOOT0 switch for entering the system bootloader, plus a debounced NRST line for reset.

[Full write-up →](./STM32%20PCB/)

---

## 4-Layer USB-C Variable Power Supply

Compact PCB that negotiates USB-C Power Delivery profiles to replace a benchtop supply for low-to-medium power testing.

- Built around a CH224Q PD sink controller that actively negotiates PD3.0/2.0 and BC1.2 profiles with the host.
- Output voltage (5V–28V) is set purely by hardware DIP switches tied to the CH224Q's configuration pins — no firmware or I2C required.
- 4-layer stackup (2 signal, 2 ground) in KiCad, with 20 mil power traces and copper pours supporting up to 2A output, and continuous ground planes for EMI control.

[Full write-up →](./Variable%20Power%20Supply/)

---

## Rocket DAQ PCB

*(This project's dedicated README is still being finalized — the summary below will be replaced with the full write-up, schematics, layout, and 3D renders once it's done.)*

### 1. Summary

Data acquisition and flight-recovery PCB designed and routed with the UCSD Rocket Propulsion Laboratory's **Hermes Hardware Team**, built to provide continuous in-flight telemetry logging alongside a redundant dual-stage parachute deployment system. The board integrates an ESP32 MCU, an onboard environmental/inertial sensor suite, and automatic USB/LiPo power management on a 4-layer stackup, and has flown and recovered successfully across multiple launches with apogees exceeding 10,000 ft.

### 2. Design Requirements & Specifications

| Parameter / Interface | Design Target / Configuration |
| :--- | :--- |
| **Microcontroller (MCU)** | ESP32-WROOM-32E |
| **PCB Layer Count** | 4-Layer |
| **Power Input** | Micro-USB (5V) or single-cell LiPo, with automatic USB/battery power-path switching |
| **Voltage Regulation** | TPS78633DCQ fixed 3.3V LDO |
| **USB Interface** | CH340C USB-to-UART bridge with automatic ESP32 reset/bootloader entry |
| **Onboard Storage** | microSD card (SPI mode, in-flight telemetry logging) |
| **Environmental Sensing** | SHT40 (temperature/humidity), BMP390 (barometric pressure/altitude) |
| **Inertial Sensing** | ADXL375 (high-g accelerometer) |
| **Sensor Bus** | I2C @ 100kHz — shared by SHT40, ADXL375, BMP390 |
| **Storage Bus** | SPI @ 1MHz — microSD card |
| **Recovery System** | Dual-stage parachute deployment (drogue + main) |
| **Flight Altimeters** | Blue Raven, RRC3+ (dual, for redundancy) |
| **Position Tracking** | GPS |
| **Design Software** | KiCad |

### 3. System Architecture & Peripheral Configuration

**3.1 Power Architecture**
The board accepts either 5V from the Micro-USB port or a single-cell LiPo through a 2-pin JST-style connector. A P-channel MOSFET (AO3401A), gated directly off the USB VBUS rail, automatically hands power over to USB when it's present and falls back to the LiPo when it isn't, while a Schottky diode (B140-E3) blocks the regulated rail from backfeeding into the USB port when running on battery. Both sources feed a TPS78633 LDO that regulates down to a clean 3.3V for the MCU, sensors, SD card, and USB bridge, with a slide switch (SW1) wired to the LDO's enable pin as a hard power on/off.

**3.2 USB Interface & Auto-Program Circuit**
Because the ESP32-WROOM-32E has no native USB peripheral, a CH340C bridges the Micro-USB port's D+/D− lines (behind a TPD2E001 ESD protection diode) to the ESP32's UART0. The CH340C's DTR and RTS outputs drive a two-transistor (2N3904) auto-reset circuit into the ESP32's EN and GPIO0 (boot-strap) pins, so the board automatically drops into the bootloader when flashing firmware from a host PC — no manual reset/boot button sequence needed.

**3.3 Sensor Suite**
Flight telemetry is captured from three sensors sharing a single I2C bus (4.7kΩ pull-ups on SDA/SCL): an SHT40 for temperature/humidity, a BMP390 for barometric pressure and altitude, and an ADXL375 high-g accelerometer for capturing launch and deployment shock loads. All three run in I2C mode, holding their SPI chip-select pins high.

**3.4 microSD Data Logging**
A microSD card is interfaced over a dedicated SPI bus (MOSI/MISO/SCK/CS) separate from the sensor I2C bus, with series resistors on the clock and MISO lines for signal integrity and a pull-up on MISO/DAT0 per the SD card's SPI-mode requirements — giving a continuous, self-contained flight data record independent of the recovery electronics.

**3.5 Dual-Stage Recovery & Redundancy**
Recovery is handled by a dual-stage parachute deployment sequence (drogue at apogee, main at a lower altitude), governed in parallel by a Blue Raven and an RRC3+ altimeter so that deployment logic has a redundant backup if either unit fails to trigger. GPS tracking is included for locating the vehicle after landing. This architecture has produced successful vehicle recovery across flights with apogees exceeding 10,000 ft.

### 4. Status

Board layout, 3D renders, and detailed test/flight data for this project are being finalized and will be added to this folder.

---

## Skills Summary

| Category | Tools / Technologies |
| :--- | :--- |
| **EDA / CAD** | KiCad, Cadence Virtuoso, LTspice, PSIM, Intel Quartus, Git |
| **Languages** | C/C++, Python, MATLAB, SystemVerilog, Assembly |
| **Embedded & RTOS** | FreeRTOS, ESP-IDF, STM32CubeIDE/CubeMX, Arduino |
| **Test & Validation** | Oscilloscopes, logic analyzers, multimeters, signal generators, power supplies, PCB soldering & rework |

---

## Repository Structure

```
Portfolio/
├── 8 Bit CLA/
├── STM32 PCB/
├── Variable Power Supply/
└── Rocket DAQ PCB/        (coming soon)
```

Each project folder contains its own detailed README with schematics, layout images, 3D renders, and simulation/test results.
