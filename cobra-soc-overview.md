[MOSART Documentation Portal Home](README.md)

<details>
<summary><strong>Table of Contents</strong></summary>

- [1. Introduction](#1-introduction)
- [2. Key Features](#2-key-features)
- [3. Major Subsystems Summary](#3-major-subsystems-summary)
- [4. Software Architecture](#4-software-architecture)
- [5. O-RU System Integration](#5-o-ru-system-integration)
- [6. Boot Flow](#6-boot-flow)
- [7. Timing & Synchronization](#7-timing--synchronization)
- [8. RFIC Integration (Yucca)](#8-rfic-integration-yucca)
- [9. Application Use Cases](#9-application-use-cases)
- [10. Summary](#10-summary)

</details>

# Cobra SoC Overview
*Metanoia MT2824 — 5G O-RU Digital Front-End System-on-Chip*

---

## 1. Introduction
The **Metanoia Cobra™ MT2824 SoC** is a next-generation, fully software-programmable 5G/B5G/6G Digital Front-End (DFE) System-on-Chip designed to enable highly flexible, cost-efficient, and standards-compliant **O-RAN Radio Units (RUs)**. Built on a scalable, software-defined architecture, Cobra supports **O-RAN split 7.2x**, disaggregated small-cell platforms, and advanced SDR research and development environments.

Cobra integrates the entire O-RU signal-processing chain—including fronthaul transport, low-PHY acceleration, digital predistortion (DPD), timing and synchronization, calibration workflows, and system management—into a compact, low-power SoC optimized for commercial deployment.

At its core, Cobra incorporates a heterogeneous compute subsystem composed of **six Cadence B20 DSP vector processors**, **five Cadence LX7 real-time controllers**, and a **dual-core RISC-V RV64 processor complex**. This architecture is purpose-built for deterministic, high-throughput radio processing. Complementing this compute fabric, Cobra integrates an **LPDDR4 memory controller** and a **dual-lane 25G network subsystem** capable of handling O-RAN S-Plane, M-Plane, and eCPRI (CU-Plane) traffic with real-time, low-latency guarantees. Through its high-speed SerDes interface, Cobra ingests packets from external SFP+ modules and routes them to the network engine and the **Metanoia Radio Acceleration Subsystem (MRAS)** for real-time baseband and radio processing.

The SoC also provides a rich set of peripheral interfaces—including **LVDS, I²C, QSPI, SPI, UART, GPIO, USB, SD/eMMC, and NOR flash**—ensuring seamless integration with RFICs, analog front-end modules, board-level control components, and storage devices.

---

## 2. Key Features
### 2.1 Processing Architecture
- Dual-core **RISC-V AX45MP** (64-bit, MMU)
- **5× Cadence LX7** RTOS cores
- **6× Cadence B20 DSPs** for low-PHY & DFE
- **2× DPD Inline Accelerators**
- Integrated **AFE** subsystem with ADC/DAC and PVT sensors

### 2.2 O-RAN Fronthaul Support
- Dual 10G/25G SerDes
- eCPRI encapsulation/decapsulation
- O-RAN eCPRI Split 7.2x Section Types 0/1/3
- Dynamic & static compression (8/12-bit)
- Timing windows compliant with O-RAN fronthaul profiles

### 2.3 Memory & Storage
- LPDDR4 memory controller
- eMMC/SD, QSPI NOR
- Internal SRAM/TCM memories

### 2.4 Peripheral Subsystems
- I2C, SPI, UART, USB2.0
- GPIO, GNSS, PTP
- LVDS RF control interface

---

## 3. Major Subsystems Summary
Major subsystems:
- **RISC-V CPU complex**
- **DSP vector processor complex**
- **LX7 RTOS subsystem**
- **Network Interface Subsystem (NIS)**: MAC/PCS, MACSec
- **DFE subsystem**: filters, compression, CFR, DPD
- **AFE IQ Bridge (IQB)**
- **Memory subsystem**
- **Boot/security subsystem**

---

## 4. Software Architecture
### 4.1 Operating System and Build Systems
The Cobra platform is built on a modern, flexible, and scalable Linux-based environment that supports multiple embedded build systems and secure boot mechanisms. The software stack includes:

#### 4.1.1 Linux Kernel
- **Linux Kernel 6.1 LTS or newer**
- Long-term–supported kernel with optimized drivers for Cobra SoC (DFE, MRAS, AFE, SerDes, peripherals)
- Supports upstream alignment for long-term maintainability

#### 4.1.2 Yocto Project (Primary Build System)
- **Yocto Project 5.0 (Kirkstone)**
- Officially supported and recommended for production firmware images
- Provides reproducible builds, package management, and long-term BSP integration

#### 4.1.3 Buildroot (Optional)
- [Metanoia Buildroot](mosart-buildroot.md) - Lightweight alternative build system for minimal root filesystem creation
- Ideal for rapid prototyping, testing, or constrained deployments

#### 4.1.4 Raspberry Pi Build Environment (Optional)
- External SBC-based environment used for early-stage development and evaluation
- Supports cross-development workflows when hardware access is limited

#### 4.1.5 Boot Chain and Secure Boot
- **OpenSBI**: RISC-V Supervisor Mode initialization and runtime services
- **U-Boot**: Secondary bootloader supporting kernel loading, flashing, and diagnostics
- [MTF (Metanoia Trusted Firmware)](cobra-mtf.md):
  - Implements secure boot
  - Validates ROTPK and authenticates early-stage boot components
  - Provides platform root of trust


### 4.2 Management and Time Synchronization Software

#### Overview
The Management and Time Synchronization Software in the Cobra O-RU architecture provides the management services needed for:
- O-RAN M-Plane configuration  
- Time synchronization (PTP/GNSS/holdover)  
- Notifications, Fault Management, and Performance reporting  
- NETCONF transport and YANG data modeling  
- Integration between Sysrepo datastore and internal RU subsystems  

It coordinates:
- NETCONF/YANG  
- mplaned  
- SysrepoHook  
- RU Manager  
- MRAS timing subsystem  

#### 4.2.1 Functional Responsibilities

##### Configuration Management
- Receives NETCONF edit-config  
- Validates via YANG  
- Updates internal configuration tree  
- Propagates changes to MRAS and system backend

##### Time Synchronization Management
Manages:
- PTP  
- GNSS  
- Holdover  
- DFET alignment  
- Timing state machine  

##### Fault & Performance Management
- Fault/Alarm collection  
- Performance KPI counters  
- M-Plane alarm notifications  

#### 4.2.2 Software Components

##### mplaned
Main M-Plane monitoring background daemon.

##### SysrepoHook
Central dispatcher for YANG updates.

##### O-RAN Module Plugins
Implements handlers, RPC callbacks, and operational providers.

#### 4.2.3 Time Synchronization Workflow

##### Configuration Flow
1. DU sends edit-config  
2. Netopeer2 → Sysrepo  
3. SysrepoHook triggers plugin  
4. Plugin validates and updates RU Manager  
5. RU Manager applies PTP/GNSS/holdover/DFET  
6. Updates returned to YANG operational  

##### Operational Monitoring Flow
- RU Manager collects PPS stability, jitter, GNSS lock  
- MRAS provides timing counters  
- SysrepoHook publishes operational data  

#### 4.2.4 NETCONF/YANG Interfaces

##### Synchronization Nodes
- sync-source  
- ptp-profile  
- clock-class  
- time-of-day  
- gnss-status  
- sync-state  

##### Alarms
- GNSS lost  
- PTP unlocked  
- PPS errors  

#### 4.2.5 Integration with RU Manager / MRAS
M-Plane config → SysrepoHook → RU Manager → MRAS Timing/DFET/Clock Synthesizer.

#### 4.2.6 Standards Compliance
Implements O-RAN WG4 Time-Sync and related synchronization requirements.


### 4.3 **MRAS** Data/Radio Processing Software
- Low PHY (FFT/iFFT, DMRS, PRACH)
- Filters, compression
- DPD/CFR

### 4.4 IPC Between Linux & MRAS
- OpenAMP messaging
- Shared memory MIB

---

## 5. O-RU System Integration
### 5.1 M-Plane
- Startup Configuration and Zero Touch Provisioning (ZTP)
- SW mgmt
- Fault/Performance mgmt
- O-RAN YANG models compliant

### 5.2 S-Plane
- IEEE 1588v2 PTP (Class B capable)
- GNSS fallback
- DFE timer alignment

### 5.3 C/U Planes
- Real-time software defined radio processing
- Header parsing
- Low-PHY compute
- Window alignment & eAxC separation

---

## 6. Boot Flow
1. [MTF (Metanoia Trusted Firmware)](cobra-mtf.md)
2. SPL (DDR init and initial system setup)
3. U-Boot
4. Linux Kernel
5. Platform drivers
6. RootFS init
7. MRAS/DSP boot
8. RU Manager starts
9. Calibration
10. O-RU ready

---

## 7. Timing & Synchronization
- DFE Timer (DFET) for deterministic timing
- GNSS/1588v2 support
- SyncE option
- 3GPP frame alignment

---

## 8. RFIC Integration (Yucca)
- LVDS-based control
- Supports 2x2 TDD/FDD
- FR1 and FR2 IF support
- Calibrations:
  - LO leakage
  - IQ imbalance
  - DC offset
  - Filter bandwidth
  - PA bias/DPD

---

## 9. Application Use Cases
### 9.1 FR1
- 2T2R to 16T16R O-RUs
- RedCap radios

### 9.2 FR2
- 2T2R / 4T4R mmWave RUs
- External mmWave FE integration

### 9.3 Small Cell / Femto
- Cost-optimized 2T2R
- Enterprise/indoor deployments

---

## 10. Summary
Cobra MT2824 is a **high-integration SDR SoC** optimized for O-RU deployments requiring:
- High compute performance
- Software-defined flexibility
- Low power & high integration
- 3GPP/O-RAN compliance

It substantially reduces time-to-market and supports a scalable architectural roadmap that enables smooth progression from 5G deployments toward B5G and 6G technologies.

