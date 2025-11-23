---
title: DNS Architecture & Management
date:
description: A comprehensive guide to the Domain Name System, recursion, caching, and overriding resolvers on Linux/Windows.
tags:
  - networking
  - dns
  - linux
  - windows
  - security
  - LLM
---

> [!abstract] The Phonebook of the Internet
> DNS is a hierarchical, distributed database. When you type `google.com`, you are asking a specific server to translate that human-readable name into a machine-readable IP address (`142.250.190.46`).
> **Control the DNS, and you control the traffic.**

## 1. How DNS Works (The Query Lifecycle)



[Image of DNS Query Resolution Steps]


When you request `wiki.habibullah.dev`, the following happens:

1.  **Stub Resolver (Your PC):** Checks local cache (`/etc/hosts` or OS cache). If missing, asks the **Recursive Resolver**.
2.  **Recursive Resolver (ISP/Cloudflare):** The workhorse. It asks the **Root Server (.)**.
3.  **Root Server:** "I don't know, but `.dev` is managed by Google Registry. Go ask them."
4.  **TLD Server (.dev):** "I don't know, but `habibullah.dev` is managed by Cloudflare. Go ask them."
5.  **Authoritative Server (Cloudflare):** "Yes, I know that specific subdomain. The IP is `104.21.65.200`."
6.  **Answer:** The IP is sent back to your PC.

---

## 2. DNS Record Types 📝

| Type | Name | Purpose |
| :--- | :--- | :--- |
| **A** | Address | Maps Hostname $\to$ IPv4 Address. |
| **AAAA** | Quad A | Maps Hostname $\to$ IPv6 Address. |
| **CNAME** | Canonical Name | Maps Hostname $\to$ Another Hostname (Alias). |
| **MX** | Mail Exchange | Tells email servers where to send email. |
| **TXT** | Text | Arbitrary data. Used for verification (Google, SPF, DKIM). |
| **NS** | Name Server | Delegates authority to another DNS server. |

---

## 3. Overwriting Global DNS (Linux/Fedora) 🐧

By default, Linux gets its DNS from your Router (DHCP). This is often slow and insecure. We want to force it to use **Cloudflare (1.1.1.1)** or **Google (8.8.8.8)**.

### Method A: The Temporary Fix (`/etc/resolv.conf`)
*Note: This file is usually overwritten by NetworkManager on reboot.*

```bash
sudo nano /etc/resolv.conf
# Add these lines
nameserver 1.1.1.1
nameserver 1.0.0.1
````

### Method B: The Permanent Fix (Systemd-Resolved)

Modern Fedora uses `systemd-resolved`.

1. **Edit the config:**
    
    Bash
    
    ```
    sudo nano /etc/systemd/resolved.conf
    ```
    
2. **Uncomment and set DNS:**
    
    Ini, TOML
    
    ```
    [Resolve]
    DNS=1.1.1.1 1.0.0.1
    # FallbackDNS=8.8.8.8
    # Domains=~.
    ```
    
3. **Restart the service:**
    
    Bash
    
    ```
    sudo systemctl restart systemd-resolved
    ```
    
4. **Verify:**
    
    Bash
    
    ```
    resolvectl status
    ```
    

### Method C: NetworkManager (GUI/CLI)

This sets it per-connection (e.g., specifically for your WiFi).

Bash

```
# List connections
nmcli connection show

# Modify (Replace 'WiFi-Name' with your actual connection name)
nmcli connection modify "WiFi-Name" ipv4.dns "1.1.1.1 1.0.0.1"
nmcli connection modify "WiFi-Name" ipv4.ignore-auto-dns yes

# Apply
nmcli connection up "WiFi-Name"
```

---

## 4. Overwriting Global DNS (Windows) 🪟

Windows hides this deep in menus, but PowerShell is faster.

### Method A: The GUI Way

1. **Run:** `ncpa.cpl` (Opens Network Connections).
    
2. Right-click your Adapter (WiFi/Ethernet) $\to$ **Properties**.
    
3. Select **Internet Protocol Version 4 (TCP/IPv4)** $\to$ **Properties**.
    
4. Select "Use the following DNS server addresses":
    
    - Preferred: `1.1.1.1`
        
    - Alternate: `1.0.0.1`
        

### Method B: The PowerShell Way (Admin)

This is instant and scriptable.

PowerShell

```
# 1. Get Interface Index
Get-NetAdapter | Select-Object Name, InterfaceIndex

# 2. Set DNS (Replace index '12' with your InterfaceIndex)
Set-DnsClientServerAddress -InterfaceIndex 12 -ServerAddresses ("1.1.1.1","1.0.0.1")

# 3. Clear Cache
Clear-DnsClientCache
```

---

## 5. Advanced: Encrypted DNS (DoH / DoT) 🛡️

Standard DNS is **cleartext**. Your ISP (and hackers on public WiFi) can see every site you visit.

### DNS over HTTPS (DoH)

Encapsulates DNS queries inside regular HTTPS traffic (Port 443). To the ISP, it looks like regular web browsing.

**Enabling in Firefox:**

- Settings $\to$ Privacy & Security $\to$ DNS over HTTPS $\to$ Max Protection (Cloudflare).
    

**Enabling in Windows 11:**

- Settings $\to$ Network & Internet $\to$ Wi-Fi $\to$ Hardware Properties $\to$ DNS Server Assignment $\to$ Edit $\to$ Encrypted DNS (On).
    

---

## 6. Debugging DNS Issues 🕵️‍♂️

If a site isn't loading, is it the server or the DNS?

**1. `dig` (The Engineer's Tool)**

Bash

```
dig google.com
dig @1.1.1.1 google.com  # Ask 1.1.1.1 specifically
```

**2. `nslookup` (Legacy/Windows)**

Bash

```
nslookup wiki.habibullah.dev
```

3. The Hosts File (The "Hard Override")

You can manually force a domain to an IP, bypassing DNS entirely.

- **Linux:** `/etc/hosts`
    
- **Windows:** `C:\Windows\System32\drivers\etc\hosts`
    

Plaintext

```
# Format: IP Domain
127.0.0.1  localhost
192.168.1.50  dev-server.local
```

---

## Linked Notes

- [[Web-Security-Basics]] - Why HTTPs is vital (DoH).
    
- [[Linux-Basics]] - Managing `/etc` files.
    
- [[DevOps/Fedora-Workstation]] - My network stack.