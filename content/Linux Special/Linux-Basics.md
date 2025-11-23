---
title: Linux Fundamentals - The Engineer's Guide
date:
description: A deep dive into the Linux philosophy, file system hierarchy, permissions, and process management.
tags:
  - linux
  - terminal
  - bash
  - fedora
  - devops
  - LLM
---

> [!abstract] The Philosophy
> Linux is not just a Kernel; it is a way of thinking.
> 1.  **Everything is a file.** (Even your hard drive, your mouse, and your RAM).
> 2.  **Small, single-purpose programs.** (Do one thing and do it well).
> 3.  **Chainability.** (Output of one program becomes the input of another).

## 1. The File System Hierarchy (FHS) 📂
Unlike Windows (which uses `C:\`, `D:\`), Linux starts from a single root: `/`.

| Path | Purpose | The "Windows" Analogy |
| :--- | :--- | :--- |
| `/` | **Root**. The beginning of everything. | `My Computer` |
| `/bin` | **Binaries**. Essential user commands (`ls`, `cp`). | `System32` |
| `/boot` | **Bootloader**. Kernel (`vmlinuz`) and GRUB live here. | (Hidden EFI partition) |
| `/dev` | **Devices**. Hardware represented as files (`/dev/sda` is your SSD). | Device Manager |
| `/etc` | **Etcetera**. System-wide configuration files. | Registry / AppData |
| `/home` | **User Home**. Where your data lives (`/home/habibullah`). | `C:\Users` |
| `/lib` | **Libraries**. Shared code required by binaries. | `.dll` files |
| `/proc` | **Processes**. A virtual window into the Kernel's brain. | Task Manager details |
| `/root` | **Root's Home**. The VIP room for the Admin user. | Administrator Folder |
| `/var` | **Variables**. Logs, website files, database storage. | `C:\ProgramData` |

> [!tip] Hacker Tip
> You can recover deleted files from a running process by exploring `/proc/<pid>/fd`.

---

## 2. The Shell (Strap Yourself In) 🐚
The Shell (Bash/Zsh) is not just a command runner; it is a full programming environment.

### The Streams (I/O)
Every program has 3 connections to the outside world:
1.  **STDIN (0):** Standard Input (Keyboard).
2.  **STDOUT (1):** Standard Output (Screen).
3.  **STDERR (2):** Standard Error (Screen, specifically for errors).

### Redirection & Piping
The pipe `|` is the most powerful operator in Linux. It connects the STDOUT of the left command to the STDIN of the right command.

```bash
# Redirection
echo "Hello" > file.txt      # Overwrite file
echo "World" >> file.txt     # Append to file

# Piping
cat file.txt | grep "Hello"  # Read file -> Search for string
ps aux | grep firefox        # List processes -> Filter for browser

# The "Black Hole"
./script.sh > /dev/null 2>&1 # Silence all output (Send to void)
````

---

## 3. Permissions (Chmod/Chown) 🔐

Linux is multi-user by design. Every file has an Owner, a Group, and a World.

**The Syntax:** `-rwxr-xr--`

- `r` = Read (4)
    
- `w` = Write (2)
    
- `x` = Execute (1)
    

**Breakdown:**

1. **Owner:** `rwx` (7) -> Can Read, Write, Run.
    
2. **Group:** `r-x` (5) -> Can Read, Run.
    
3. **World:** `r--` (4) -> Can only Read.
    

**Commands:**

Bash

```
chmod +x script.sh       # Make executable
chmod 777 script.sh      # Give EVERYONE access (Dangerous!)
chmod 600 private.key    # Only owner can read/write (Secure)

chown user:group file    # Change ownership
```

---

## 4. Process Management ⚡

You are the god of your machine. You decide what lives and dies.

|**Command**|**Action**|
|---|---|
|`ps aux`|List all running processes.|
|`top` / `htop`|Task Manager (Real-time CPU/RAM usage).|
|`kill <pid>`|Ask a process to stop nicely (SIGTERM).|
|`kill -9 <pid>`|**Force Kill**. The Kernel assassinates the process immediately (SIGKILL).|
|`Ctrl + Z`|Pause current process (send to background).|
|`bg` / `fg`|Resume process in background or foreground.|

---

## 5. Text Manipulation (The Superpowers)

GUI editors open files. Linux tools **stream** files.

- **grep:** Search for patterns.
    
    Bash
    
    ```
    grep -r "TODO" .  # Find all "TODO" comments in current directory
    ```
    
- **head / tail:** View start or end.
    
    Bash
    
    ```
    tail -f /var/log/nginx/access.log  # Watch server hits in real-time
    ```
    
- **sed:** Stream Editor (Find & Replace).
    
    Bash
    
    ```
    sed -i 's/foo/bar/g' config.txt  # Replace 'foo' with 'bar' inside file
    ```
    

---

## 6. Networking from CLI 🌐

- **curl:** The Swiss Army Knife of HTTP.
    
    Bash
    
    ```
    curl -I google.com             # Check headers
    curl -L habibullah.dev         # Follow redirects
    ```
    
- **ss / netstat:** Who is listening?
    
    Bash
    
    ```
    ss -tuln  # Show all TCP/UDP ports listening on numbers
    ```
    
- **ssh:** Remote access.
    
    Bash
    
    ```
    ssh -i key.pem user@192.168.1.10
    ```
    

---

## 7. Package Management (DNF/RPM) 📦

Since you run **Fedora**, you use `dnf`.

Bash

```
sudo dnf update              # Update entire system
sudo dnf install git         # Install package
sudo dnf search "browser"    # Find package
sudo dnf history             # Undo mistakes (Rollback installs)
```

---

## Linked Notes

- [[Fedora-Workstation]] - My specific configuration.
    
- [[Linux-Kernel-Internals]] - How the OS works deeper down.
    
- [[Docker-Ultimate-Guide]] - Running Linux inside Linux.