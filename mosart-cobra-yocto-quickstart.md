[MOSART Documentation Portal Home](README.md)

# MOSART Cobra SoC Yocto – Quick Start Guide

> **Purpose**: Get a reference Cobra SoC Yocto image built and ready as quickly as possible.
>
> **Audience**: Developers, partners, and integrators evaluating or bringing up MOSART on Cobra SoC.

---

## 1. What This Guide Covers

This quick start guide walks you through the following by a working example build:
- Setting up the supported build environment
- Building a reference MOSART Yocto image for Cobra SoC
- Understanding where outputs are generated

It intentionally avoids advanced customization, debugging, and internal implementation details. For full documentation, refer to the **MOSART Cobra SoC Yocto Guide**.

---

## 2. Prerequisites

### 2.1 Host Requirements

- Linux host (Ubuntu 20.04 / 22.04 recommended)
  - Having at least 130 GB storage for full build tree with all caches and logs.
- Docker installed and running
- Internet access or access to local mirrors (if provided)

For example:
```bash
$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 24.04.3 LTS
Release:        24.04
Codename:       noble

$ docker --version
Docker version 28.1.1, build 4eba377

$ ssh -T git@gitlab.com
Welcome to GitLab
```

---

## 3. Get the Source

Clone the official MOSART Yocto BSP repository:

```bash
git -c protocol.file.allow=always clone \
  --recurse-submodules \
  -b mosart-main \
  git@gitlab.com:metanoia-comm/mosart/public/meta-metanoia.git
```

If you are cloning the source from GitHub, please use:

```
git clone https://github.com/metanoia-comm-mosart/meta-metanoia.git
```

After cloning, please run the recipe_update_github_uri.sh script to
update the recipe SRC_URI entries so they point to the corresponding
GitHub repositories:

```
./recipe_update_github_uri.sh
```

---

## 4. Build the Docker Environment

MOSART provides a Docker-based build environment to ensure reproducibility.

```bash
cd meta-metanoia
docker build \
  --build-arg HOST_UID=$(id -u) \
  --build-arg HOST_GID=$(id -g) \
  -t mosart-cobra-yocto-build .
```

Run the container:

```bash
$ docker run --rm -it \
  -v ${PWD}:/home/metanoia/workspace \
  -v "$HOME/.ssh:/home/metanoia/.ssh" \
  mosart-cobra-yocto-build bash
$ ssh -T git@gitlab.com
Welcome to GitLab
$

```

---

## 5. Initialize the Yocto Build Environment

Inside the Docker container:

```bash
$ cd workspace
. oe-init-build-env
~/workspace/build$ 

```

This creates (or reuses) the default `build/` directory.

---

## 6. Build the Reference Image

Build the default MOSART reference image:

```bash
$ export BB_ENV_PASSTHROUGH_ADDITIONS="USE_MOSART"
$ export USE_MOSART=1
$ time bitbake -DD dev-image  > build-evb-log-`date +%s`.txt 2>&1
real    41m23.110s
user    3m16.084s
sys     1m41.935s
```

---

## 7. Locate Build Outputs

After a successful build, the cobra-evb images are available under:

```
build/tmp/deploy/images/mt58xx-cobra-evb/
```

Typical artifacts include:
- `fitImage` (kernel + DTB)
- Root filesystem images (`.squashfs`, `.cpio.gz`, `.full.img`)
- U-Boot and SPL binaries

---

## Flash the image to SD card

Example - assume you have a SD card slot connected to /dev/sda on your Linux host. Then you can run dd as
```bash
cd meta-metanoia/build/tmp/deploy/images/mt58xx-cobra-evb
sudo dd if=dev-image-mt58xx-cobra-evb.rootfs-20260106035900.full.img of=/dev/sda bs=4M status=progress conv=fsync
sync
```

After the SD is successfully flashed, plug it into the SD slot on your Cobra EVB and check the boot source jumpers are properly set for SD. Power on you should see the boot message from the UART console.
```bash
.......
U-Boot SPL 2023.01 (Nov 05 2025 - 03:18:50 +0000)
Device ID: SHUTTLE (0x28240000)
.......
mt58xx-cobra-evb login: root
Password: 
root@mt58xx-cobra-evb:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether f2:de:7a:78:7f:d4 brd ff:ff:ff:ff:ff:ff
    inet 192.168.9.99/24 brd 192.168.9.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet 172.16.0.103/24 metric 1024 brd 172.16.0.255 scope global dynamic eth0
       valid_lft 86373sec preferred_lft 86373sec
    inet6 2000::c4/64 scope global 
       valid_lft forever preferred_lft forever
3: eth1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether 5a:6f:cf:94:23:40 brd ff:ff:ff:ff:ff:ff
4: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
root@mt58xx-cobra-evb:~#
root@mt58xx-cobra-evb:~# lsb_release -a
LSB Version:    n/a
Distributor ID: metanoia
Description:    Cobra-SDK 2026.01.06-04:09:12
Release:        2026.01.06-04:09:12
Codename:       cobra

root@mt58xx-cobra-evb:~# systemctl list-units --type=service
  UNIT                                     LOAD   ACTIVE SUB     DESCRIPTION                                    
  accutime.service                         loaded active exited  AccuTime
  board-info.service                       loaded active exited  Board-Info
  busybox-syslog.service                   loaded active running System Logging Service
  cobra-init.service                       loaded active exited  Cobra INIT
  dbus.service                             loaded active running D-Bus System Message Bus
  getty@tty1.service                       loaded active running Getty on tty1
  mpbe-ptp-eventd.service                  loaded active running Metanoia M-Plane Backend PTP Event Daemon
  mpbe.service                             loaded active exited  Metanoia M-Plane Backend
  mpbed.service                            loaded active running Metanoia M-Plane Backend Daemon
  mpbed@openamp.service                    loaded active running Metanoia M-Plane Backend Daemon of openamp
  rf-board.service                         loaded active exited  RF-Board control
  serial-getty@ttyS0.service               loaded active running Serial Getty on ttyS0
  skyworks.service                         loaded active exited  Skyworks
  sshdgenkeys.service                      loaded active exited  OpenSSH Key Generation
  sysstat.service                          loaded active exited  Resets System Activity Logs
  systemd-journal-flush.service            loaded active exited  Flush Journal to Persistent Storage
  systemd-journald.service                 loaded active running Journal Service
  systemd-logind.service                   loaded active running User Login Management
  systemd-network-generator.service        loaded active exited  Generate network units from Kernel command line
  systemd-networkd.service                 loaded active running Network Configuration
  systemd-random-seed.service              loaded active exited  Load/Save OS Random Seed
  systemd-remount-fs.service               loaded active exited  Remount Root and Kernel File Systems
  systemd-resolved.service                 loaded active running Network Name Resolution
  systemd-sysctl.service                   loaded active exited  Apply Kernel Variables
  systemd-tmpfiles-setup-dev-early.service loaded active exited  Create Static Device Nodes in /dev gracefully
  systemd-tmpfiles-setup-dev.service       loaded active exited  Create Static Device Nodes in /dev
  systemd-tmpfiles-setup.service           loaded active exited  Create Volatile Files and Directories
  systemd-udev-trigger.service             loaded active exited  Coldplug All udev Devices
  systemd-udevd.service                    loaded active running Rule-based Manager for Device Events and Files
  systemd-update-utmp.service              loaded active exited  Record System Boot/Shutdown in UTMP
  systemd-user-sessions.service            loaded active exited  Permit User Sessions
  systemd-userdbd.service                  loaded active running User Database Manager
  systemd-vconsole-setup.service           loaded active exited  Virtual Console Setup
  ts-saver.service                         loaded active exited  load timestamp
  user-runtime-dir@0.service               loaded active exited  User Runtime Directory /run/user/0
  user@0.service                           loaded active running User Manager for UID 0

Legend: LOAD   -> Reflects whether the unit definition was properly loaded.
        ACTIVE -> The high-level unit activation state, i.e. generalization of SUB.
        SUB    -> The low-level unit activation state, values depend on unit type.

36 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
root@mt58xx-cobra-evb:~# 

```

## 9. Next Steps

From here, you may:
- Add user-space packages via Yocto recipes
- Customize kernel configuration using supported mechanisms
- Start and deploy 5G O-RU applications (Not covered by the generic MOSART document, please consult Metanoia official documentation)

For advanced workflows, customization guidelines, and support boundaries, refer to the full **MOSART Cobra SoC Yocto Guide**.

---

## 10. Support Notes

- This quick start demonstrates a **reference build to bring-up the Linux platform** provided by Metanoia.
- Customizations beyond documented extension points are outside MOSART support scope.
- Always track MOSART and Cobra SDK versions for reproducibility.

---
