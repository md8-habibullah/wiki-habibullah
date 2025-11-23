---
title: Burp Suite Professional Setup & Methodology
date:
description: Configuring the industry-standard web vulnerability scanner and proxy for CTFs and Bug Bounties.
tags:
  - security
  - pentesting
  - tools
  - burp
  - web-hacking
  - LLM
---

> [!abstract] The "Man-in-the-Middle"
> Burp Suite operates on a simple premise: **Trust Logic**.
> By telling your browser to trust Burp as a Certificate Authority (CA), you can break SSL/TLS encryption. You see the raw HTTP requests before they leave your machine, and the raw responses before the browser renders them.

## 1. Installation on Fedora (`JUICE`)

While `dnf` has some security tools, it is best to download the official installer or JAR to stay current.

**Option A: The Script (Recommended)**
1.  Download the **Linux (64-bit)** installer from PortSwigger.
2.  Make it executable and run:
    ```bash
    chmod +x burpsuite_pro_linux_v2023_x_x.sh
    ./burpsuite_pro_linux_v2023_x_x.sh
    ```

**Option B: The JAR (Portable)**
If you prefer a lightweight setup:
```bash
# Launch with 4GB RAM allocation
java -jar -Xmx4g burpsuite_pro.jar
````

---

## 2. Browser Integration (The Critical Step) 🔌

Do not configure your system-wide proxy. That breaks Spotify and Discord. Use **Firefox** with **FoxyProxy**.

### Step A: FoxyProxy Setup

1. Install the **FoxyProxy Standard** extension in Firefox.
    
2. Add a new proxy:
    
    - **Type:** HTTP
        
    - **IP:** `127.0.0.1`
        
    - **Port:** `8080`
        
    - **Color:** Green (or whatever signals "Hacking Mode")
        
3. Click the FoxyProxy icon and select your new profile.
    

### Step B: Installing the CA Certificate

_If you skip this, HTTPS sites will throw "Secure Connection Failed" errors._

1. Ensure Burp is running and FoxyProxy is set to Burp.
    
2. Visit `http://burp` in Firefox.
    
3. Click **"CA Certificate"** in the top right to download `cacert.der`.
    
4. In Firefox: **Settings** -> Search "Certificates" -> **View Certificates**.
    
5. Click **Import** -> Select `cacert.der`.
    
6. **Check both boxes:**
    
    - [ ] Trust this CA to identify websites.
        
    - [ ] Trust this CA to identify email users.
        

---

## 3. The Arsenal: Core Tools 🛠️

### A. Proxy (The Gatekeeper)

- **Intercept:** Toggles traffic flow.
    
    - _On:_ Requests hang until you click "Forward". Great for modifying a request on the fly.
        
    - _Off:_ Traffic flows passively. Use this 90% of the time and check **HTTP History**.
        
- **HTTP History:** The log of everything. Filter by "Scope" to hide Google Analytics noise.
    

### B. Repeater (The Lab)

Shortcut: Ctrl + R (Send to Repeater)

This is where research happens. You take a request, tweak one parameter (e.g., change id=1 to id=1'), and hit Send to see the raw response.

- _Workflow:_ Send $\rightarrow$ Modify $\rightarrow$ Check Response Code/Length $\rightarrow$ Repeat.
    

### C. Intruder (The Machine Gun)

Shortcut: Ctrl + I

Used for brute-forcing and fuzzing.

1. **Payload Positions:** Highlight the data you want to fuzz (e.g., a password field).
    
2. **Attack Type:**
    
    - **Sniper:** Single payload set. Tries list A against slot 1, then slot 2.
        
    - **Battering Ram:** Same payload in all slots simultaneously.
        
    - **Pitchfork:** Different payloads for different slots (User/Pass lists).
        

### D. Decoder

A built-in utility for Base64, URL, HTML, and Hex encoding.

- _Tip:_ Use the "Smart Decode" button if you see garbage text.
    

---

## 4. Methodology: The Search for Anomalies

When analyzing a target, I follow this loop:

1. **Map the App:** Click every button. Fill every form. Populate the **Site Map**.
    
2. **Define Scope:** Right-click the domain in Target tab -> **"Add to Scope"**.
    
    - _Proxy Filter:_ Click the filter bar -> "Show only in-scope items".
        
3. **Hunt:**
    
    - Look for ID parameters (`user=101`) $\rightarrow$ Test IDOR.
        
    - Look for search bars $\rightarrow$ Test XSS/SQLi.
        
    - Look for file uploads $\rightarrow$ Test RCE.
        

---

## 5. Pro Tips for Fedora Users

- **Dark Mode:** User Options -> Display -> Look and Feel -> **Darcula**.
    
- **Font:** Change HTTP Message font to **JetBrains Mono** size 14 for readability.
    
- **Extensions (BApp Store):**
    
    - **Turbo Intruder:** For high-speed race condition testing.
        
    - **Logger++:** Better logging than the default history.
        
    - **JSON Web Tokens (JWT) Editor:** Essential for modern API hacking.
        

---

## Linked Notes

- [[SQL-Injection-Methodology]] - Use Repeater to test payloads.
    
- [[Web-Security-Basics]] - Fundamentals of HTTP.
    
- [[DevOps/Fedora-Workstation]] - Host OS configuration.