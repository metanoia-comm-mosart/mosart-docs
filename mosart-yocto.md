[MOSART Documentation Portal Home](README.md)

# MOSART Cobra SoC Yocto Guide

> **Audience**: Partners, system integrators, and advanced users
>    building Linux images on Metanoia Cobra SoC using MOSART.
>
> **Scope**: This document describes the *supported* Yocto build
    workflow, configuration model, and extension points for Cobra SoC
    within the MOSART framework. It intentionally omits internal
    infrastructure details and non-supported customization paths.

This document focuses on the Linux system layer built using the
**Yocto Project**, an open-source and community-driven framework for
creating reproducible and customizable embedded Linux distributions
tailored to specific hardware platforms and system requirements.

The **MOSART** open platform is intentionally designed to remain
portable and extensible, enabling future support for alternative build
systems such as Buildroot, OpenWrt, and other open-source or
proprietary environments. Developers, partners, and contributors are
encouraged to explore the MOSART documentation portal and participate
in the MOSART ecosystem through collaboration and contributions.

MOSART provides a reference integration framework and reference
deployment architecture. Production deployment architectures may vary
depending on customer system requirements, carrier specifications, and
hardware integration choices.

MOSART leverages upstream open-source projects whenever practical and
maintains platform-specific integration layers separately to improve
long-term maintainability and upstream alignment.

Security hardening, secure boot policy, key provisioning, and
production credential management are deployment-specific
responsibilities unless explicitly documented otherwise.

The MOSART Yocto environment should be considered a reference BSP and
integration framework rather than a fixed end-product software stack.

> **Note 1:** This document describes a specific reference Yocto build
> for a 5G O-RU based on the MOSART open platform. Throughout the
> remainder of this document, this reference platform may be referred
> to as **MOSART-XG**, or simply **MOSART**, depending on context.

> **Note 2:** If you are familiar with Yocto, you may jump directly to
> [MOSART Cobra Yocto Build Quick Start](mosart-cobra-yocto-quickstart.md).

> **Note 3:** This document focuses specifically on MOSART running on
> Cobra SoC–based hardware platforms and the associated reference
> Yocto integration environment.
>
> Throughout this document, the term **MOSART** may refer
> specifically to the MOSART software platform as integrated and
> validated on Cobra SoC hardware unless otherwise explicitly stated.

## Table of Contents

- [MOSART Cobra SoC Yocto Guide](#mosart-cobra-soc-yocto-guide)
- [1. MOSART Yocto Context](#1-mosart-yocto-context)
- [2. Supported Yocto Model](#2-supported-yocto-model)
- [3. Layer and Responsibility Boundaries](#3-layer-and-responsibility-boundaries)
- [4. Hardware Targets](#4-hardware-targets)
- [5. Build Environment](#5-build-environment)
  - [5.1 Host Requirements](#51-host-requirements)
  - [5.2 Repository Checkout](#52-repository-checkout)
- [6. Image Build Workflow](#6-image-build-workflow)
  - [6.1 Initialize Build Environment](#61-initialize-build-environment)
  - [6.2 Build Reference Image](#62-build-reference-image)
  - [6.3 Build Output Structure](#63-build-output-structure)
- [7. Kernel and Bootloader Updates](#7-kernel-and-bootloader-updates)
  - [7.1 Kernel](#71-kernel)
  - [7.2 U-Boot](#72-u-boot)
- [8. Root Filesystem Customization](#8-root-filesystem-customization)
  - [8.1 Adding Packages](#81-adding-packages)
  - [8.2 Initramfs Images](#82-initramfs-images)
- [9. Supported Development Workflow (devtool)](#9-supported-development-workflow-devtool)
- [10. Upgrade Strategy](#10-upgrade-strategy)
- [11. Version Identification](#11-version-identification)
- [12. Debug and Bring-Up](#12-debug-and-bring-up)
- [13. What Is Not Covered](#13-what-is-not-covered)
- [14. Summary](#14-summary)

---

## 1. MOSART Yocto Context

The **MOSART** Cobra Yocto environment is a reference build and integration
framework for Metanoia’s open platform built around the Cobra SoC. It
defines:

- A **reference system architecture** for 7.2x O-RU deployments

- A **supported Yocto-based software stack**, including bootloader,
  Linux kernel, root filesystem, platform services, and development
  tooling

- Clear **responsibility boundaries** between MOSART core components,
  vendor-maintained modules, and customer-specific integrations

- A reproducible and maintainable Linux system software environment
  intended for platform bring-up, development, validation, and
  deployment

- A modular integration model that allows deployment-specific
  customization of proprietary components such as PHY layers,
  synchronization hardware, RF front-end devices, power amplifiers
  (PAs), and other vendor-specific subsystems outside the MOSART core
  platform scope

Within MOSART:

- Third-party PHY and DSP software layers may remain vendor-maintained
  and externally integrated depending on deployment requirements.

- The Yocto-based MOSART environment is intended to deliver a
  reproducible, maintainable, and supportable Linux system software
  stack for the Cobra SoC platform.

- Proprietary customer-specific integrations and customizations —
  including vendor-specific PHY implementations, timing and
  synchronization hardware, RF front-end components, power amplifiers
  (PAs), and other specialized hardware or software modules — remain
  deployment-dependent choices outside the scope of the MOSART core
  platform.

---

## 2. Supported Yocto Model

The Cobra SoC platform uses a **Poky-based Yocto Project**
distribution with dedicated BSP and integration layers.

Core layers include:

- **meta-metanoia** – Cobra SoC BSP, platform integration, and MOSART
  reference system layer

The Yocto environment is designed to:

- Enable reproducible and repeatable Linux image builds

- Support controlled customization through Yocto recipes, layers,
  configuration fragments, and overlays

- Preserve upgrade compatibility across multiple software releases and
  deployment generations

- Provide a maintainable and supportable Linux system software stack
  for development, validation, and deployment workflows

- Support modular integration of deployment-specific components and
  proprietary vendor extensions outside the MOSART core platform scope

---

## 3. Layer and Responsibility Boundaries

Within the MOSART framework, the core Linux platform, BSP, and system
integration layers are maintained as the reference open platform,
while deployment-specific proprietary integrations remain under vendor
or customer responsibility.

### Documented Extension Points

Documented extension points refer to supported customization
mechanisms explicitly defined by MOSART, including but not limited to:

- Yocto recipes and `.bbappend` files in customer-owned layers
- Device Tree overlays and documented DTS hooks
- Kernel configuration fragments and approved patch mechanisms
- User-space applications, services, and startup scripts
- Image-level package selection and root filesystem composition

These mechanisms are intended to provide controlled extensibility
while preserving maintainability, upgrade compatibility, and support
coverage across MOSART software releases.

### What Is *Not* a Documented Extension Point

Equally important, the following actions are **not** considered
documented extension points within the MOSART framework:

- Direct modification of platform-owned BSP recipes
- Forking or rewriting core logic within `meta-metanoia`
- Undocumented changes to the Linux kernel, bootloader, or firmware
- Modifying the boot flow or partition layout outside published and
  supported interfaces

While such changes may function technically, they fall outside the
defined MOSART support scope and are not guaranteed to remain
compatible across future software releases.

Deployment-specific proprietary integrations — including vendor PHY
implementations, synchronization hardware, RF front-end devices, power
amplifiers (PAs), and other specialized hardware or software modules
— remain customer or vendor responsibilities outside the MOSART core
platform scope.

### MOSART Architecture

The MOSART architecture is designed as a modular, extensible software
platform centered around the Cobra SoC ecosystem.
The current
reference architecture consists of the following major repositories
and software components.

The repository structure shown below represents the current reference
MOSART integration architecture and may evolve across future software
releases.

```text
mosart
├── ddrfw            # DDR training and initialization firmware
├── dspfw            # DSP firmware and radio processing binaries
├── gnss-module      # GNSS synchronization and timing integration
├── linux            # Linux kernel with Cobra SoC BSP support
├── meta-metanoia    # Core Yocto BSP and platform integration layer
├── metanoia-base    # Common platform utilities and base components
├── meta-rf          # RF-related Yocto integration layer
├── meta-sysrepo     # Sysrepo and management framework integration
├── mosart-docs      # MOSART and Cobra SoC public and reference documentation
├── mpapi            # Metanoia platform API framework
├── mras-init        # MRAS initialization and startup utilities
├── opensbi          # OpenSBI firmware and platform support
├── oran-mplane      # O-RAN M-Plane management components
├── ruboard          # Board support utilities and platform tools
├── rumanager        # Radio Unit management framework
├── sfp-module       # SFP transceiver management components
└── u-boot           # U-Boot bootloader with Cobra platform support
```

Unless otherwise specified, MOSART repositories are released under
open-source licenses documented within each repository.

The MOSART repository structure is intentionally modular to enable:

- Independent component maintenance and versioning
- Flexible deployment-specific integration
- Controlled customization through documented extension mechanisms
- Reproducible software builds and release management
- Long-term maintainability across multiple software generations
- Future extensibility for additional hardware platforms, software
  modules, and deployment environments

Additional proprietary, customer-specific, or vendor-maintained
components may be integrated into deployment environments depending on
system architecture and operational requirements.


## 4. Hardware Targets

The MOSART Cobra Yocto platform supports multiple hardware targets and
deployment profiles through machine configurations, Device Tree
separation, and modular BSP integration.

Current reference hardware targets include:

- **Cobra EVB (Evaluation Board)**  
  Primary reference platform for software development, platform
  bring-up, validation, debugging, and integration workflows.

- **O-RAN WG7 Compliant O-RU Whitebox Platforms**  
  Third-party O-RU whitebox hardware platforms based on the Cobra SoC
  and aligned with O-RAN WG7 hardware specifications.

The MOSART framework is designed to support long-term platform
maintainability and deployment flexibility through explicit machine,
board, and Device Tree separation.

Deployment-specific hardware integrations — including RF front-end
designs, synchronization hardware, timing modules, power amplifiers
(PAs), and carrier-specific configurations — may vary by platform and
remain outside the MOSART core platform scope.

---

## 5. Build Environment

### 5.1 Host Requirements

The MOSART Yocto build environment is intended to run on a Linux host
system using Docker-based reproducible build containers.

Minimum host requirements include:

- Validated build hosts currently include Ubuntu 24.04 LTS and
  compatible Linux distributions.
- Docker
- **Note**: Yocto build environments may require substantial disk space
  depending on enabled features, build artifacts, and cache retention
  policies.

The official build environment is distributed as a pre-configured
Docker image to ensure reproducibility, dependency consistency, and
long-term build compatibility across different development hosts.

Since BitBake internally uses Linux network namespaces (`netns`),
running BitBake as a non-root user requires enabling unprivileged user
namespace support on the host system.

Configure the following sysctl parameters (for example via
`/etc/sysctl.conf`):

```
kernel.apparmor_restrict_unprivileged_userns=0
kernel.unprivileged_userns_clone=1
```

### 5.2 Repository Checkout

The MOSART reference Yocto environment is distributed through Git
repositories hosted on GitLab.

Using the Git SSH protocol, clone the repository and all submodules as
follows:

```bash
git -c protocol.file.allow=always clone \
  --recurse-submodules \
  -b use-mosart \
  git@gitlab.com:metanoia-comm/mosart/public/meta-metanoia.git
```

If you are behind a firewall that only allows HTTP/HTTPS traffic, you
can rewrite Git SSH URLs to HTTPS by updating your `.gitconfig` and,
if required, configuring a GitLab HTTPS access token:

```
[credential]
       helper = store

[url "https://gitlab.com/"]
       insteadOf = git@gitlab.com:

[url "https://git.yoctoproject.org/"]
       insteadOf = git://git.yoctoproject.org/

[url "https://github.com/"]
       insteadOf = git@github.com:
```

---

## 6. Image Build Workflow

The reproducible build model is intended to support both local
development workflows and automated CI/CD integration pipelines.

### 6.1 Initialize Build Environment

After cloning the repository and starting the official MOSART Yocto
Docker build environment, initialize the Yocto build environment as follows:

```bash
export BB_ENV_PASSTHROUGH_ADDITIONS="USE_MOSART"
export USE_MOSART=1
. ./oe-init-build-env
```

### 6.2 Build Reference Image

```bash
bitbake dev-image
```

### 6.3 Build Output Structure

After compilation, the build directory contains the following structure:

```
build/
├── conf/                    # Build configuration (local.conf, bblayers.conf)
├── downloads/               # Downloaded source tarballs (DL_DIR)
├── sstate-cache/            # Shared state cache for faster rebuilds
├── cache/                   # BitBake parse cache
├── tmp/                     # Main build output directory
│   ├── deploy/
│   │   ├── images/          # Final deployable binary images
│   │   │   └── <MACHINE>/   # Machine-specific images (e.g., mt58xx-cobra-evb)
│   │   ├── rpm/             # Built packages (RPM format)
│   │   ├── licenses/        # License information
│   │   └── spdx/            # Software Bill of Materials
│   ├── work/                # Per-recipe build directories
│   ├── work-shared/         # Shared sources (kernel, gcc)
│   ├── sysroots-components/ # Staged components for cross-compilation
│   ├── stamps/              # Build completion markers
│   ├── log/                 # Build logs
│   └── buildstats/          # Build timing statistics
```

The `downloads/` and `sstate-cache/` directory may be shared across
multiple build environments to accelerate rebuild times and improve CI
efficiency.

The final deployable binary images are generated under:

```
build/tmp/deploy/images/<MACHINE>/
```

For the Cobra EVB reference platform, the image output directory is:
```
build/tmp/deploy/images/mt58xx-cobra-evb/
```

#### Key Image Files

The final deployment directory contains image artifacts used for
development, flashing, deployment, upgrade, recovery, and debugging
purposes.

| File | Description |
|------|-------------|
| `dev-image-*.full.img` | Complete flash image for full device programming and SD card generation |
| `dev-image-*.squashfs-xz` | Compressed root filesystem image using SquashFS with XZ compression |
| `fitImage*.bin` | Linux kernel and Device Tree packaged in FIT image format |
| `u-boot*.itb` | U-Boot bootloader packaged as a FIT image |
| `mtf-fip.bin` | Metanoia firmware image package used during early boot initialization |
| `cobra_lpddr4_3200*.bin` | DDR initialization and training firmware blobs |
| `*.dtb` | Device Tree Blob (DTB) describing board-specific hardware configuration |
| `swupgrade-*.zip` | Software upgrade package used for field or OTA upgrade workflows |
| `*.sysimg` | System deployment image used for manufacturing or deployment flows |
| `*.manifest` | Package manifest listing all software packages included in the image |

Depending on the selected build configuration and deployment profile,
additional debug symbols, SDK artifacts, recovery images, test images,
or intermediate build artifacts may also be generated.

---

## 7. Kernel and Bootloader Updates

### 7.1 Kernel

The supported Linux kernel within the MOSART framework is provided as
the `linux-metanoia` recipe, which contains the Cobra SoC BSP,
platform enablement patches, Device Tree support, and integration
configuration required for the reference platform.

Common kernel development operations include:

```bash
bitbake -c compile linux-metanoia
bitbake -c deploy virtual/kernel
```

Fast iteration is possible via Yocto task scripts (`run.do_compile`).

### 7.2 U-Boot

The MOSART reference platform uses U-Boot as the primary second-stage
bootloader for Cobra SoC platforms.

```bash
bitbake -c compile u-boot
bitbake -c deploy virtual/bootloader
```

---

## 8. Root Filesystem Customization

### 8.1 Adding Packages

User-space functionality within the MOSART Yocto environment is
typically added through:

- Image recipes
- Package groups
- Customer-owned Yocto layers
- `.bbappend` customization mechanisms

This approach preserves maintainability, upgrade compatibility, and
reproducibility across future MOSART SDK releases.

Recommended customization methods include:

- Adding packages through image recipe dependencies
- Creating deployment-specific package groups
- Extending images through customer-managed layers
- Using documented Yocto configuration and overlay mechanisms

Direct modification of platform-owned core recipes is discouraged
unless explicitly documented and supported.

### 8.2 Initramfs Images

Initramfs-based images are supported for:

- Early platform bring-up
- Recovery and rescue workflows
- Manufacturing and deployment testing
- Debug and development use cases

Example build command:

```bash
bitbake dev-image-initramfs
```

---

## 9. Supported Development Workflow (`devtool`)

Yocto `devtool` is the preferred and recommended mechanism for
modifying supported MOSART software components during development.

Example workflow:

```bash
devtool modify linux-metanoia
devtool build linux-metanoia
devtool update-recipe --append ../meta-metanoia linux-metanoia
```

This approach helps ensure that source modifications remain traceable,
reviewable, reproducible, and maintainable across future MOSART
software releases by capturing changes as explicit patches and recipe
extensions.

---

## 10. Upgrade Strategy

MOSART supports multiple upgrade approaches to accommodate different
deployment, operational, and lifecycle management requirements.

Supported upgrade approaches may include:

- **Full image upgrades**  
  Complete device reprogramming workflows such as SD card or eMMC
  image replacement.

- **Partial software updates**  
  Selective component updates such as kernel-only, bootloader-only, or
  root filesystem–only upgrades.

- **Customized deployment-specific upgrade mechanisms**  
  Customers may implement their own upgrade strategies and lifecycle
  management flows using the MOSART open platform according to their
  specific operational, system architecture, or deployment
  requirements.

    - Customers may implement A/B redundancy, rollback protection, or
      OTA lifecycle management frameworks as part of
      deployment-specific upgrade architectures.

On the Cobra SoC reference platforms, the default partition layout,
boot flow, and software upgrade behavior are defined through
Metanoia-provided Yocto image classes and BSP integration layers.

These reference mechanisms are intended to provide a reproducible and
maintainable baseline implementation for development, validation, and
demonstration purposes.

Customers are free to modify, extend, or replace the reference upgrade
strategy according to their own system requirements, manufacturing
flows, deployment architecture, redundancy models, or operational
policies.

Such deployment-specific upgrade solutions may be implemented
independently of Metanoia provided they align with the customer’s
overall system design and operational objectives.

> **Support Disclaimer**
>
> The reference partition layout, boot flow, and software upgrade
> mechanisms provided by Metanoia are intended primarily as reference
> implementations for development and validation.
>
> Upgrade architectures or partition layouts that diverge from the
> documented MOSART reference implementation may fall outside the
> standard MOSART support scope.
>
> Validation, maintenance, integration, and long-term compatibility of
> such customized upgrade mechanisms remain the responsibility of the
> customer or deployment integrator.


## 11. Version Identification

MOSART builds support explicit software version identification and
traceability through build metadata propagation.

Example environment configuration:

```bash
export BB_ENV_PASSTHROUGH_ADDITIONS="METANOIA_TAG METANOIA_HASH METANOIA_BRANCH METANOIA_SDKVERSION"
```
These variables allow the build system to embed version and source
control metadata into generated system artifacts and runtime
identification files.

Typical metadata may include:

- Software release tag
- Git commit hash
- Source branch name
- SDK or platform release version
- Build provenance information

This metadata is commonly propagated into:

- System identification files
- Build manifests
- Runtime version reporting utilities
- Support and diagnostic logs
- Deployment and upgrade metadata

Consistent version identification is important for:

- Build reproducibility
- Software release management
- Deployment tracking
- Upgrade compatibility validation
- Support and issue traceability

Deployment-specific build systems or customer-managed release flows
may extend or customize these metadata mechanisms according to their
own operational requirements.

---

## 12. Debug and Bring-Up

MOSART supports multiple debugging and platform bring-up workflows for
software development, system integration, and low-level platform
validation.

Supported debugging targets may include:

- Boot firmware and early boot stages
- OpenSBI
- U-Boot
- Linux kernel
- Device Tree and BSP bring-up
- User-space applications and services

Standard GDB-based debugging workflows are typically used together
with platform-specific debug probes, UART consoles, JTAG interfaces,
or remote debug servers depending on the deployment environment and
hardware configuration.

Common bring-up activities may include:

- Boot flow validation
- Early console debugging
- DDR initialization verification
- Kernel boot analysis
- Device Tree validation
- Driver and peripheral bring-up
- Platform service debugging
- DSP and subsystem integration testing

Toolchain paths, debug probe configuration, JTAG setup, and remote
debug server environments are deployment-specific and therefore
outside the scope of this public reference guide.

Deployment-specific debugging environments, proprietary tooling, or
customer-specific workflows may vary depending on the target platform,
hardware integration, and operational requirements.

---

## 13. What Is *Not* Covered

This public reference guide intentionally does **not** cover certain
deployment-specific, proprietary, or vendor-internal implementation
details.

Areas outside the scope of this document include:

- PHY, Low-PHY, and Hi-PHY internal implementation details
- DSP firmware architecture and development workflows
- Custom modem, waveform, or Layer-1 algorithm development
- Proprietary vendor-specific radio processing implementations
- Deployment-specific RF tuning and calibration procedures
- Carrier-specific optimization or certification workflows
- Undocumented Yocto layer modifications or unsupported BSP changes
- Proprietary customer integration frameworks and deployment tooling

These areas may remain vendor-internal, proprietary, or fully
customer-owned depending on the deployment model and system
architecture.

Such components and workflows are generally outside the standard
MOSART reference platform support scope unless explicitly documented
or separately agreed upon through commercial support or integration
engagements.

## 14. Summary

The MOSART Cobra Yocto platform provides a reproducible and
maintainable Linux system software environment for Cobra SoC–based
wireless infrastructure platforms.

Key characteristics of the MOSART reference platform include:

- A stable and supported Linux system software stack
- A modular Yocto-based BSP and integration framework
- Clearly defined extension and responsibility boundaries
- Reproducible builds aligned with Cobra SDK software releases
- Controlled customization through documented extension mechanisms
- Long-term maintainability and upgrade compatibility across software
  generations

By following the documented workflows, supported customization
mechanisms, and integration boundaries, developers, partners, and
system integrators can build, customize, validate, and maintain
Cobra-based systems with predictable deployment, upgrade, and support
characteristics.

The MOSART framework is intended to provide an open and extensible
foundation for future wireless infrastructure development while
allowing deployment-specific flexibility for proprietary integrations,
customer customization, and operational requirements.

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
