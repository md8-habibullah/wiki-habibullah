---
title: Infrastructure - JUICE (Fedora Workstation)
date:
description: Hardware specification, filesystem architecture, and toolchain configuration for my primary mobile workstation.
tags:
  - infrastructure
  - fedora
  - linux
  - config
  - hardware
  - LLM
---

> [!abstract] Node Overview
> **Hostname:** `JUICE-SHOP`
> **Hardware:** Lenovo IdeaPad Slim 3 (15IRH8)
> **CPU:** Intel Core i5-13420H (13th Gen, 12 Threads)
> **RAM:** 8GB DDR4 + 8GB zram Swap
> **OS:** Fedora Linux 43 (Workstation Edition)

## 1. Filesystem Architecture (Btrfs)
My storage strategy leverages **Btrfs** (B-Tree Filesystem) for its snapshot capabilities and subvolume management. Unlike traditional partitioning, my root (`/`) and home (`/home`) share the same physical space on the NVMe drive, allowing dynamic space allocation.

### Partition Map (`nvme0n1`)
| Partition | Filesystem | Mount Point | Purpose |
| :--- | :--- | :--- | :--- |
| `p1` | vfat (FAT32) | `/boot/efi` | UEFI Bootloader partition. |
| `p2` | ext4 | `/boot` | Kernel images (`vmlinuz`) and initramfs. |
| `p3` | **btrfs** | `/` & `/home` | The main data volume. Uses subvolumes. |

### Memory Management (zram)
Instead of a slow disk-based Swap partition, I utilize **zram0**.
* **Size:** 7.4GB (Matching physical RAM).
* **Mechanism:** Compresses RAM content on the fly. This effectively doubles my usable memory for heavy compilations (Rust/C++) without hitting the SSD.

---

## 2. Security Posture 🛡️
I adhere to the "Secure by Default" philosophy of the Red Hat ecosystem.

* **SELinux:** `Enforcing` (Targeted Policy).
    * Every process runs in a confined domain. Even if a web service is hacked, it cannot access `/home` or `/etc` unless explicitly allowed.
* **Kernel:** `Linux 6.17.8`
    * Running on the bleeding edge allows for the latest hardware support and scheduler optimizations for the Intel Hybrid Architecture (P-cores vs E-cores).
* **Package Isolation:** Mixed ecosystem of RPM (Native), Flatpak (Desktop Apps), and Snap (Legacy CLIs).

---

## 3. The Developer Toolchain
My environment is configured for Full-Stack and Systems programming.

### Core Languages
| Language | Version | Use Case |
| :--- | :--- | :--- |
| **Python** | `3.14.0` | Scripting, CTF exploits, Data Science. |
| **Node.js** | `v22.20.0` | Frontend Development (Next.js/React). |
| **Go** | `1.25.4` | Backend Services, CLI tools. |
| **GCC** | `15.2.1` | C/C++ Compilation (Kernel modules). |

### Containerization
* **Engine:** Docker `28.5.2`
* **Privilege Model:** Rootful (Standard).
* **Context:** Used for creating reproducible builds and hosting local microservices (Postgres/Redis) for development.

---

## 4. Desktop Environment (GNOME 49)
I run a customized GNOME Shell (Wayland) focused on productivity and minimalism.

**Extensions Active:**
* **Dash to Dock:** Converts the dash into a permanent dock for quick launching.
* **System Monitor:** Real-time CPU/Net usage in the top bar.
* **Clipboard Indicator:** History manager (essential for coding).
* **Color Picker:** Frontend design utility.

---

## 5. Maintenance Routines

**System Upgrade:**
```bash
sudo dnf upgrade --refresh
````

Cleanup (Reclaim Space):

Since I use Docker and Snaps, disk usage can grow fast.

Bash

```
# 1. Clean DNF cache
sudo dnf clean all

# 2. Prune Docker (Stop hoarding images)
docker system prune -a

# 3. Vacuum System Logs
journalctl --vacuum-time=2d
```

---

## Linked Notes

- [[Linux-Basics]] - General command reference.
    
- [[Docker-Ultimate-Guide]] - Deep dive into my container setup.
    
- [[Linux-Kernel-Internals]] - Understanding the 6.17 Kernel.