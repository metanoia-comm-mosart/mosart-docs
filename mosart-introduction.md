# MOSART Introduction

<details>
<summary><strong>Table of Contents</strong></summary>

- [Key Features](#key-features)
- [Objectives](#objectives)
- [Architecture Overview](#architecture-overview)
- [Open-Source Vision](#open-source-vision)
- [Roadmap](#roadmap)
  - [Areas Under Evaluation](#areas-under-evaluation)
    - [Linux-to-DSP Management Framework](#linux-to-dsp-management-framework)
    - [Alternative Build System Support](#alternative-build-system-support)
    - [RTOS Support on DSP Processors](#rtos-support-on-dsp-processors)
    - [DFE Common Libraries](#dfe-common-libraries)
    - [Open DSP Demonstration Platform](#open-dsp-demonstration-platform)
  - [Longer-Term Direction](#longer-term-direction)
    - [Open CPU and DSP Platform](#open-cpu-and-dsp-platform)
  - [Additional Information](#additional-information)
- [Legal Notices](#legal-notices)
  - [Open-Source Licensing](#open-source-licensing)
  - [Trademark Notice](#trademark-notice)
  - [Reference Platform Disclaimer](#reference-platform-disclaimer)

</details>

MOSART™ (Metanoia Open-Source Advanced Radio Technology) is an
open-source software-defined radio (SDR) platform and development SDK
designed for modern wireless infrastructure targeting current and
future AI-RAN systems and beyond.

MOSART is initially demonstrated on Metanoia’s Cobra SoC platform,
which serves as the primary reference implementation platform for the
project. The platform provides a cost-efficient, high-performance, and
production-quality environment for developing, evaluating, and
deploying current and next-generation wireless communication
solutions, especially for 3GPP and O-RAN compliant radio systems.

MOSART is designed to support future AI-assisted radio optimization,
edge inference workloads, intelligent RAN orchestration frameworks,
and evolving AI-RAN architectures for B5G and 6G wireless systems.

The MOSART SDK integrates the essential software components required
for embedded Linux systems, real-time radio processing, platform
management, hardware abstraction, and O-RAN interfaces into a unified
and reproducible development environment. It is intended to support
developers, researchers, customers, and ecosystem partners working on
B5G/6G wireless technologies, Open RAN infrastructure, SDR
experimentation, and embedded networking systems.

MOSART encourages open collaboration, interoperability, and
upstream-friendly development across the wireless ecosystem.

## Key Features

- Open-source SDR platform
- O-RAN compliant software stack
- Yocto-based SDK and development environment
- Portable architecture adaptable to other build systems such as Buildroot
- Modular repository and software architecture
- Real-time DSP and radio processing integration
- Linux-based embedded wireless platform
- AI-RAN ready platform foundation
- Cobra SoC reference implementation platform
- Production-quality wireless infrastructure integration

## Objectives

The primary objectives of MOSART are:

- Provide an open and accessible SDR platform for wireless innovation
- Enable rapid prototyping and evaluation of AI-RAN and O-RAN systems
- Simplify integration between hardware, firmware, Linux, and radio software stacks
- Offer a reproducible Yocto-based SDK and development environment
  while remaining portable to other build systems such as Buildroot
- Support collaboration between industry, academia, and ecosystem partners
- Demonstrate production-quality embedded wireless system integration

## Architecture Overview

MOSART is built on a modular architecture demonstrated on the O-RAN
WG7 Whitebox compliant Cobra SoC evaluation platform. The platform
combines general-purpose processing, DSP acceleration, RF control,
synchronization, timing, and high-speed networking into a unified SDR
system architecture.

Major software and platform components include:

- Linux kernel and device drivers
- OpenSBI and U-Boot boot stack
- Yocto-based embedded Linux distribution
- DSP and radio firmware integration
- Hardware abstraction and board support packages
- O-RAN software interfaces and management components
- RF configuration and synchronization support
- Ethernet fronthaul and networking integration
- Platform APIs and management utilities

The modular repository structure allows components to be independently
maintained, extended, customized, and upstreamed where appropriate.

## Open-Source Vision

MOSART aims to bridge the gap between research-oriented SDR platforms
and carrier-grade wireless infrastructure solutions. By combining
open-source software methodologies with commercial-quality platform
integration, MOSART provides a foundation for long-term ecosystem
collaboration, innovation, and future wireless experimentation.

Public documentation, source repositories, development guides, and
reference demonstrations will continue to expand as the project
evolves.

## Roadmap

MOSART is intended to evolve progressively from an open O-RAN O-RU
reference implementation into a broader software-defined radio (SDR)
platform supporting wireless infrastructure innovation, AI-RAN
research, and advanced radio system development.

The following areas are currently being evaluated and explored as part
of the ongoing evolution of the MOSART ecosystem.

### Areas Under Evaluation

#### Linux-to-DSP Management Framework

- Further enhance OpenAMP-based communication and management
  capabilities between Linux and DSP subsystems.
- Explore APIs for provisioning, monitoring, control, and firmware
  lifecycle management of DSP applications.
- Simplify integration between application software and real-time
  processing components.

#### Alternative Build System Support

- Investigate support for build systems beyond Yocto, including
  Buildroot and other embedded Linux environments.
- Improve portability and flexibility across different development
  workflows.

#### RTOS Support on DSP Processors

- Evaluate support for real-time operating systems such as FreeRTOS
  and Zephyr on Cobra DSP subsystems.
- Facilitate development of real-time applications and platform
  customization.

#### DFE Common Libraries

- Develop reusable Digital Front-End (DFE) software components.
- Provide common building blocks that may simplify development of
  radio signal processing applications.

#### Open DSP Demonstration Platform

- Provide DSP-focused demonstration examples and reference
  applications.
- Illustrate software development workflows on Cobra SoC DSP
  processors.
- Support experimentation, education, and evaluation of customized
  radio processing techniques.

### Longer-Term Direction

#### Open CPU and DSP Platform

- Explore broader openness and extensibility across CPU and DSP
  software environments while continuing to support customized and
  proprietary deployments where appropriate.
- Enable greater flexibility for Linux, DSP, AI, and radio processing
  applications.
- Support research, experimentation, and ecosystem collaboration
  related to SDR, AI-RAN, and future wireless technologies.

### Additional Information

MOSART will continue to evolve through open-source development and
ecosystem collaboration. Some Cobra SoC platform capabilities,
reference implementations, and deployment-specific features may be
documented or released separately from the core MOSART project.

For information regarding Cobra SoC-specific capabilities,
customizations, evaluation programs, commercial support, or other
advanced development opportunities, please contact Metanoia
Communications.

Roadmap items are provided for informational purposes only and are
subject to change as project priorities, community contributions,
technology developments, and ecosystem requirements evolve.


## Legal Notices

### Open-Source Licensing

Unless otherwise specified, MOSART repositories and software
components are released under the open-source licenses documented
within their respective repositories.

Third-party software components may remain subject to their original
upstream licenses and terms.

### Trademark Notice

MOSART™ (Metanoia Open-Source Advanced Radio Technology), Cobra™, and
related names, logos, and branding are trademarks or registered
trademarks of Metanoia Communications Inc. or their respective owners.

Use of the MOSART name, logo, or branding does not grant endorsement,
certification, or official partnership status unless explicitly
authorized by Metanoia.

### Reference Platform Disclaimer

The MOSART platform software framework and reference deployment
examples described in this document are provided for development,
evaluation, integration, and educational purposes.

Commercial deployment architectures, regulatory compliance,
performance validation, security hardening, and production lifecycle
management remain deployment-specific responsibilities unless
explicitly documented otherwise.
