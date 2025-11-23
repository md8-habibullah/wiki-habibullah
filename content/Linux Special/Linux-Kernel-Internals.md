---
title: Linux Kernel Internals & Architecture
date:
description: A comprehensive guide to Ring 0, System Calls, Memory Management, and compiling custom kernels on Fedora.
tags:
  - linux
  - low-level
  - c
  - kernel
  - security
  - LLM
---

> [!abstract] The Core
> The Kernel is the **sovereign** of the operating system. It is the only program that has full access to the hardware (Ring 0). Everything else—your shell, your browser, Docker—lives in "User Space" (Ring 3) and must politely ask the Kernel to do things via **System Calls**.

## 1. Architecture Overview
Linux is a **Monolithic**, yet **Modular** kernel.
* **Monolithic:** The file system, drivers, network stack, and scheduler all run in the same high-privileged memory space.
* **Modular:** You can insert code (Kernel Modules) into the running kernel on the fly without rebooting.



### The User Space Barrier (Ring 3 vs Ring 0)
When you run `mkdir folder`, your CPU switches modes.
1.  **User Space:** `mkdir` command runs.
2.  **Context Switch:** Program calls `mkdir()` syscall.
3.  **Kernel Space:** CPU jumps to Ring 0. Kernel checks permissions, writes to disk inode.
4.  **Return:** CPU jumps back to Ring 3. `mkdir` exits.

---

## 2. Key Subsystems

### A. The Scheduler (CFS)
The **Completely Fair Scheduler (CFS)** decides which process gets the CPU.
* It uses a **Red-Black Tree** to track processes.
* Processes waiting the longest (lowest `vruntime`) are picked next.
* **Nice Value:** You can manipulate this priority. `nice -n -20` tells the kernel "I am important."

### B. Memory Management (The Great Lie)
The Kernel lies to every process.
* **Virtual Memory:** Every application thinks it has access to a massive, contiguous block of RAM.
* **Paging:** The Kernel maps these "Virtual" addresses to scattered "Physical" RAM pages (4KB chunks).
* **The OOM Killer:** If RAM is full, the kernel scans for a victim process (usually Chrome or Java) and executes it to save the system.

### C. The Virtual File System (VFS)
"Everything is a file."
The VFS creates a uniform abstraction. It doesn't matter if data is on an SSD (`ext4`), a USB stick (`fat32`), or a network share (`nfs`). The Kernel treats them all as files.
* `/proc`: This is a "pseudo-filesystem". It doesn't exist on disk. It is a window directly into the Kernel's RAM.

---

## 3. Kernel Modules (Drivers)
On Fedora, your kernel is `vmlinuz`. It is small. Most drivers (WiFi, GPU) are loaded dynamically as `.ko` (Kernel Object) files.

**Management Commands:**
```bash
lsmod             # List loaded modules
modinfo kvm       # Get info about a specific module
sudo modprobe -v kvm_intel  # Load a module safely
sudo rmmod kvm_intel        # Unload a module
````

> [!danger] Security Risk
> 
> Rootkits often hide as Kernel Modules. If an attacker can insmod a malicious .ko file, they own the machine completely. They can hide processes from top and files from ls.

---

## 4. The Boot Process (From Power to Login)

Understanding this is crucial for troubleshooting "Unbootable" systems.

1. **BIOS/UEFI:** Hardware check (POST). Loads the Bootloader.
    
2. **Bootloader (GRUB2):** The menu where you select Fedora. It loads the Kernel Image (`vmlinuz`) into RAM.
    
3. **Initramfs:** A tiny, temporary file system loaded into RAM. It contains just enough drivers to mount your _actual_ hard drive (e.g., decryption tools for LUKS).
    
4. **Kernel Init:** The kernel mounts the root filesystem (`/`) as read-only, then read-write.
    
5. **User Space Init:** The Kernel starts PID 1 (`systemd`).
    
6. **Systemd:** Starts services (NetworkManager, GDM, Docker) in parallel.
    

---

## 5. Compiling a Custom Kernel

The ultimate rite of passage. Why do this? To strip out bloat, enable experimental features, or just to learn.

**Steps on Fedora:**

Bash

```
# 1. Install Dev Tools
sudo dnf5 group install "development-tools"
sudo dnf install ncurses-devel bison flex elfutils-libelf-devel openssl-devel

# 2. Download Source (The Mainline)
wget [https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.8.9.tar.xz](https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.8.9.tar.xz)
tar -xvf linux-6.8.9.tar.xz
cd linux-6.8.9

# 3. Configure (The Hard Part)
# Copy your existing Fedora config as a base
cp /boot/config-$(uname -r) .config
make menuconfig
# > A blue menu appears. This is where you enable/disable drivers.

# 4. Compile (Takes time! Use all cores)
make -j $(nproc)

# 5. Install Modules & Kernel
sudo make modules_install
sudo make install

# 6. Update Bootloader
# Fedora usually does this automatically on 'make install', 
# but verify it appears in /boot/grub2/grub.cfg
```

---

## 6. Kernel Hacking & Security

### eBPF (Extended Berkeley Packet Filter)

The hottest topic in DevOps right now. eBPF allows you to run sandboxed programs _inside_ the kernel without changing kernel source code or loading modules.

- **Use cases:** High-performance networking (Cilium), Observability (Pixie), Security (Falco).
    

### SELinux (Security Enhanced Linux)

Developed by the NSA, standard on Fedora. It enforces Mandatory Access Control (MAC).

Even if you are Root, SELinux can block you from doing things the policy forbids (like Apache writing to /home).

### Analyzing a Crash (Kernel Panic)

When the kernel crashes, it often dumps logs to `dmesg`.

- **Oops:** The kernel killed a process but the system is still alive.
    
- **Panic:** The kernel detected a fatal error and halted the system to prevent data corruption.
    

---

## 7. The Engineer's Toolkit

|**Command**|**Purpose**|
|---|---|
|`uname -r`|Print kernel version (e.g., `6.17.8-300.fc43`)|
|`dmesg -w`|Watch the kernel ring buffer (hardware events)|
|`sysctl -a`|List all runtime kernel parameters|
|`strace -p <pid>`|Trace system calls of a running process|
|`uptime`|Shows "Load Average" (CPU queue length)|
|`/proc/cpuinfo`|Detailed processor data|

---

## Linked Notes

- [[Docker-Ultimate-Guide]] - How Docker uses Namespaces & Cgroups.
    
- [[Fedora-Workstation]] - My local environment.
    
- [[CPP-Programming]] - The language of the kernel.