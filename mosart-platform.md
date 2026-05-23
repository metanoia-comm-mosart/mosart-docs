[MOSART Documentation Portal Home](README.md)

<details>
<summary><strong>Table of Contents</strong></summary>

- [MOSART Platform Software](#mosart-platform-software)
  - [Overview](#overview)
  - [Document Version Information](#document-version-information)
  - [Reference Platform Architecture](#reference-platform-architecture)
  - [Intended Audience and Prerequisites](#intended-audience-and-prerequisites)
  - [Topics Included](#topics-included)
  - [MOSART Cobra EVB SD/eMMC Storage Layout](#mosart-cobra-evb-sdemmc-storage-layout)
    - [Storage Image Layout](#storage-image-layout)
    - [Functional Notes](#functional-notes)
      - [Bootloader Regions](#bootloader-regions)
      - [Kernel A/B Scheme](#kernel-ab-scheme)
      - [Data Partitions](#data-partitions)
  - [Boot Flow and Platform Initialization](#boot-flow-and-platform-initialization)
  - [MTF Firmware and GPT Image Generation](#mtf-firmware-and-gpt-image-generation)
  - [Device Tree Integration](#device-tree-integration)
  - [U-Boot Environment Management](#u-boot-environment-management)
  - [Flashing and Deployment Workflows](#flashing-and-deployment-workflows)
  - [Yocto Image Integration and Build Configuration](#yocto-image-integration-and-build-configuration)
  - [DSP Firmware and PHY Integration](#dsp-firmware-and-phy-integration)
  - [MRAS Initialization and Platform Services](#mras-initialization-and-platform-services)
  - [System API and Management Frameworks](#system-api-and-management-frameworks)
  - [SFP Module Support and Management Utilities](#sfp-module-support-and-management-utilities)
  - [Other Platform Software Related Topics](#other-platform-software-related-topics)
  - [Frequently Asked Questions (FAQ)](#frequently-asked-questions-faq)
  - [Additional References](#additional-references)
</details>

# MOSART Platform Software

## Overview

The MOSART Platform Software provides a reference boot, storage,
platform initialization, and system integration framework for Cobra
SoC–based platforms.

The platform supports reproducible Yocto-based builds, modular
software integration, and deployment-oriented system architectures
suitable for wireless infrastructure and SDR applications.

MOSART is designed as an extensible open platform framework that
enables developers, researchers, ecosystem partners, and system
integrators to customize and deploy Cobra SoC platforms for
deployment-specific applications including O-RAN and AI-RAN systems.

## Document Version Information

| Item | Value |
|------|-------|
| Document | MOSART Platform Software |
| Platform | Cobra SoC |
| Reference Build | MOSART Yocto Reference Platform |
| Status | Public Preview |
| Last Updated | 2026-05-23 |

---


MOSART should be viewed as a reference BSP and integration
framework rather than a fixed end-product software stack.

MOSART leverages upstream open-source projects whenever practical
while maintaining platform-specific integration layers separately to
improve long-term maintainability and upstream alignment.

This document focuses on platform software integration and deployment
concepts rather than low-level PHY or DSP implementation details.

Unless otherwise stated, examples in this document are based on the
MOSART Yocto reference implementation running on the Cobra EVB
platform.


## Reference Platform Architecture

The following diagram illustrates the high-level MOSART platform
software architecture on Cobra SoC platforms.

```text
Applications / O-RAN Services / User Applications
                    |
Platform Services / Management Frameworks
(mpapi, rumanager, sysrepo, oran-mplane)
                    |
Linux Root Filesystem / Yocto Integration
                    |
Linux Kernel (linux-metanoia)
                    |
OpenSBI / U-Boot / MTF
                    |
Cobra SoC Hardware Platform
```

The architecture shown above represents a reference integration model.
Deployment-specific software stacks and hardware integration layers
may vary depending on customer requirements and deployment
architectures.

---


## Intended Audience and Prerequisites

This document is intended for:

- Platform developers
- System integrators
- Embedded Linux developers
- Wireless infrastructure engineers
- O-RAN and SDR platform developers
- Advanced users evaluating MOSART and Cobra SoC platforms

Readers are expected to have basic familiarity with:

- Embedded Linux systems
- Yocto Project workflows
- Bootloader concepts
- Device Tree concepts
- Storage partitioning and deployment workflows

---

## Topics Included

- [Boot Flow and Platform Initialization](#boot-flow-and-platform-initialization)
- [MTF Firmware and GPT Image Generation](#mtf-firmware-and-gpt-image-generation)
- [Storage Layout and Partitioning](#mosart-cobra-evb-sdemmc-storage-layout)
- [Device Tree Integration](#device-tree-integration)
- [U-Boot Environment Management](#u-boot-environment-management)
- [Flashing and Deployment Workflows](#flashing-and-deployment-workflows)
- [Yocto Image Integration and Build Configuration](#yocto-image-integration-and-build-configuration)
- [DSP Firmware and PHY Integration](#dsp-firmware-and-phy-integration)
- [MRAS Initialization and Platform Services](#mras-initialization-and-platform-services)
- [System API and Management Frameworks](#system-api-and-management-frameworks)
- [SFP Module Support and Management Utilities](#sfp-module-support-and-management-utilities)
- [Other Platform Software Related Topics](#other-platform-software-related-topics)

> **Note:**  
> The topics and subsections described in this document may be
> expanded, refined, or updated over time as MOSART platform
> development on Cobra SoC hardware continues to evolve.
>
> Additional implementation details, deployment workflows, platform
> integration examples, debugging procedures, and development process
> documentation may be added progressively as the MOSART ecosystem,
> software stack, and reference platforms mature.

---

## MOSART Cobra EVB SD/eMMC Storage Layout

### Storage Image Layout

The following storage layout represents a reference deployment
example for a **4GB eMMC** or **4GB SD** storage device used on the
Cobra EVB platform.

Actual deployment layouts may vary depending on product
requirements, redundancy policies, update strategies, and available
storage capacity.

> **Important:**  
> The storage layout shown below is provided as a reference example
> only. Commercial deployments may use different partition sizes,
> redundancy models, filesystems, or upgrade strategies.

> **Note:**  
> The MOSART Bootrom MTF understands **GPT format only**.  
> Regions outside standard GPT-managed partitions may be used by early
> boot firmware and next-stage bootloader components according to the
> platform boot architecture.

In the example layout below, **SPL-1** and **SPL-2** both reference a
shared **U-Boot** bootloader image for convenience. In a full U-Boot
A/B design, each SPL may have its own U-Boot image to enable complete
redundancy across both SPL and U-Boot components.

Here, **SPL** refers to **U-Boot SPL** (Secondary Program Loader).

| Region # | Type | Size | Content | Notes / Purpose |
|-----------|------|------|---------|----------------|
| **Region 1 - 3** | RAW (Boot Region) | 3 MiB | **GPT Image** | GPT table and FIP blobs for DTB, SPL, and OpenSBI images |
| **Region 4** | RAW (Boot Region) | 13 MiB | **U-Boot+ENV** | U-Boot and U-Boot environment |
| **Region 5** | RAW (Partition-3) | 32 MiB | **Kernel0** | Kernel (slot A) |
| **Region 6** | RAW (Partition-4) | 32 MiB | **Kernel1** | Kernel (slot B) |
| **Region 7** | SquashFS (Partition-5) | 128 MiB | **RootFS0** | Root filesystem (slot A) |
| **Region 8** | SquashFS (Partition-6) | 128 MiB | **RootFS1** | Root filesystem (slot B) |
| **Region 9** | EXT4 (Partition-7) | 1 GiB | **Permanent Data** | Persistent configs, calibration, logs, RU/DU DB |
| **Region 10** | EXT4 (Partition-8) | 2 GiB | **User Data** | Operator/user data |

> **Note:** For details in **Region 1–3**, see [Cobra GPT Image Layout and Generation - COMING SOON!](cobra-mtf.md#cobra-gpt-image-layout-and-generation)

---

### Functional Notes

#### Bootloader Regions
- **SPL1 / SPL2**: Redundant first-stage loaders.
- **DTB1 / DTB2**: Redundant device trees corresponding to SPL copies.
- **U-Boot + ENV**: Bootloader and environment stored in RAW space.  
  *Two copies may exist for full A/B redundancy.*

#### Kernel A/B Scheme
- **Kernel0 + RootFS0** → Primary boot slot A  
- **Kernel1 + RootFS1** → Secondary boot slot B  
This layout enables deployment-specific A/B upgrade workflows,
rollback protection strategies, and high-availability software update
mechanisms.

#### Data Partitions
- **Permanent Data (EXT4, 1 GiB)**  
  Used for calibration, persistent configs, logs, O-RU/O-DU databases.

- **User Data (EXT4, 2 GiB)**  
  Stores downloaded packages, operator files, applications, and user configurations.

---

> **Reference Platform Note:**  
> The following sections describe the MOSART reference implementation
> and documented integration model for Cobra SoC platforms.
>
> Deployment-specific customization, vendor extensions,
> customer-owned modifications, and proprietary integrations may vary
> depending on operational requirements and deployment architecture.
>
> Some interfaces, workflows, and deployment models described in this
> document may evolve as the MOSART platform continues to mature.

---

## Boot Flow and Platform Initialization

This section describes the MOSART reference boot flow on Cobra
SoC–based platforms, including Bootrom behavior, MTF loading,
OpenSBI, U-Boot initialization, Linux kernel loading, and root
filesystem startup.

Representative topics include:

- Bootrom initialization sequence
- GPT image loading flow
- MTF firmware initialization
- OpenSBI handoff
- U-Boot initialization and boot scripts
- Linux kernel startup
- Root filesystem mounting and system initialization
- Recovery and fallback boot behavior

---

## MTF Firmware and GPT Image Generation

This section describes the MOSART MTF firmware framework and GPT image
generation workflow used by the Cobra SoC platform.

Representative topics include:

- GPT image layout generation
- MTF firmware packaging
- SPL and DTB integration
- OpenSBI image integration
- Boot image redundancy models
- Image signing and deployment considerations
- Reference image generation workflows

---

## Device Tree Integration

This section describes Device Tree usage within the MOSART platform
software framework.

Representative topics include:

- Cobra SoC Device Tree hierarchy
- Board-specific DTS organization
- Peripheral enablement
- RF and platform hardware configuration
- Device Tree overlays
- Customer-specific platform extensions
- DTS integration within Yocto builds

---

## U-Boot Environment Management

This section describes the reference U-Boot environment configuration
used on MOSART Cobra SoC platforms.

Representative topics include:

- Environment variable organization
- Boot slot selection
- A/B boot handling
- Recovery boot logic
- Boot script execution
- Storage boot target selection
- Update and rollback behavior

---

## Flashing and Deployment Workflows

This section describes reference workflows for flashing and deploying
MOSART software images onto Cobra SoC platforms.

Representative topics include:

- SD card image deployment
- eMMC flashing workflows
- Recovery image usage
- GPT image flashing
- Kernel and rootfs updates
- Field upgrade workflows
- Manufacturing deployment considerations

---

## Yocto Image Integration and Build Configuration

This section describes how MOSART platform software components are
integrated into the Yocto build environment.

Representative topics include:

- Yocto image recipes
- BSP integration
- Package groups
- Root filesystem customization
- Build configuration fragments
- Deployment image generation
- SDK and release integration

---

## DSP Firmware and PHY Integration

This section describes DSP firmware integration and deployment models
within the MOSART platform framework.

Representative topics include:

- DSP firmware loading
- PHY integration architecture
- Vendor-specific PHY support
- DSP binary deployment
- Firmware packaging
- Runtime firmware initialization
- Deployment-specific PHY integration considerations

---

## MRAS Initialization and Platform Services

This section describes MRAS initialization and low-level platform
service integration within MOSART.

Representative topics include:

- Platform initialization services
- System startup sequencing
- Hardware initialization helpers
- Platform monitoring services
- Runtime management integration
- Deployment-specific service extensions

---

## System API and Management Frameworks

This section describes MOSART platform management frameworks and
system APIs.

Representative topics include:

- MPAPI framework
- RU management integration
- Sysrepo integration
- O-RAN M-Plane interfaces
- Platform management APIs
- Runtime configuration management
- Service orchestration frameworks

---

## SFP Module Support and Management Utilities

This section describes SFP transceiver integration and management
support within the MOSART framework.

Representative topics include:

- SFP EEPROM access
- Optical module monitoring
- Link status management
- Platform utility integration
- Deployment-specific optical module support
- Diagnostics and monitoring workflows

---

## Other Platform Software Related Topics

Additional platform software topics may include:

- Platform debugging workflows
- Recovery and rescue environments
- Manufacturing utilities
- Platform validation tools
- Secure boot integration
- High-availability deployment models
- CI/CD integration workflows
- Future platform software extensions

---

## Frequently Asked Questions (FAQ)

### Is the storage layout fixed for all deployments?

No. The layouts described in this document are reference deployment
examples. Commercial deployments may use different partition sizes,
redundancy models, storage devices, filesystems, or upgrade
strategies.

### Does MOSART support build systems other than Yocto?

Yes. MOSART officially supports Yocto and also has ports for other
platform environments including Buildroot. Additional deployment-
specific environments may also be supported.

### Are secure boot and production key management included?

Security hardening, secure boot policy, key provisioning, verified
boot, and production credential management are deployment-specific
responsibilities unless explicitly documented otherwise.

### Are all MOSART components open source?

Unless otherwise specified, MOSART repositories are released under
open-source licenses documented within each repository. Some
deployment-specific vendor components may remain proprietary.

---

## Additional References

Additional MOSART platform documentation may include:

- MOSART Cobra SoC Yocto Guide
- MOSART Architecture Documentation
- MOSART Boot Flow Documentation
- MOSART Device Tree Documentation
- MOSART GPT and MTF Documentation
- MOSART Platform Bring-Up Guides

Future documentation updates may expand these areas as MOSART platform
development continues to evolve.

---

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
