---
title: Web Security Fundamentals
date:
description: The core principles of HTTP, Browser Security (SOP/CORS), OWASP Top 10, and Attack Vectors.
tags:
  - security
  - http
  - owasp
  - networking
  - theory
  - LLM
---

> [!abstract] The Security Mindset
> Security is not a feature you add at the end; it is a state of being.
> In web development, we assume **User Input is Evil**. The browser is a hostile environment controlled by the user, and the network is a public wire monitored by adversaries.

## 1. The Protocol: HTTP & HTTPS 🌐
The web is built on the HyperText Transfer Protocol. Understanding this is non-negotiable.



### Anatomy of a Request
```http
POST /login HTTP/1.1
Host: bank.com
User-Agent: Mozilla/5.0...
Cookie: session_id=xyz123
Content-Type: application/x-www-form-urlencoded

username=admin&password=password123
````

### The CIA Triad

Every security control maps to one of these three:

1. **Confidentiality:** Only authorized people can see the data (Encryption).
    
2. **Integrity:** The data has not been tampered with (Hashing/Signatures).
    
3. **Availability:** The system is up and running (DDoS Protection).
    

### HTTPS (TLS/SSL)

HTTP is cleartext. Anyone on the WiFi can read your password using Wireshark.

TLS (Transport Layer Security) solves this by encrypting the pipe.

- **Handshake:** The client and server agree on a "Cipher Suite" and exchange keys.
    
- **Certificates:** The server proves its identity using a certificate signed by a trusted CA (Certificate Authority).
    

---

## 2. The Browser Security Model 🛡️

As a Frontend Developer, this is your battleground.

### Same Origin Policy (SOP)

The most important rule in the browser.

- **Definition:** A script loaded from `Origin A` cannot access data from `Origin B`.
    
- **Origin:** Defined by `Protocol + Domain + Port`.
    
    - `https://google.com` $\neq$ `http://google.com` (Different Protocol)
        
    - `https://google.com` $\neq$ `https://api.google.com` (Different Domain)
        

### CORS (Cross-Origin Resource Sharing)

SOP breaks the modern web (where your React frontend talks to your Node backend on a different port).

CORS is the "Exception" to SOP.

- The **Browser** asks the server: _"Hey, `localhost:3000` wants to read your data. Is that cool?"_ (Preflight Request).
    
- The **Server** replies: `Access-Control-Allow-Origin: localhost:3000`.
    

> [!danger] Misconfiguration
> 
> Setting Access-Control-Allow-Origin: * allows ANY website to read your users' private data if they are logged in. Never use * with Access-Control-Allow-Credentials: true.

### CSP (Content Security Policy)

A header that tells the browser which sources are trusted for scripts, images, and styles.

- _Header:_ `Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.com`
    
- _Benefit:_ It kills XSS. Even if an attacker injects `<script>alert(1)</script>`, the browser refuses to execute it because it violates the policy.
    

---

## 3. Client-Side Attacks

### Cross-Site Scripting (XSS)

The attacker injects malicious JavaScript into a page viewed by other users.

|**Type**|**Mechanism**|**Persistence**|
|---|---|---|
|**Reflected**|Payload is in the URL (`?q=<script>...`). User clicks a bad link.|No (Single Request)|
|**Stored**|Payload is saved in the DB (Comment section). Every visitor gets hacked.|Yes (Persistent)|
|**DOM-based**|Payload executes purely in Client-side JS without hitting the server.|No|

- **Impact:** Cookie theft (`document.cookie`), Keylogging, Phishing.
    
- **Defense:** Context-aware Output Encoding (React does this by default).
    

### Cross-Site Request Forgery (CSRF)

The attacker forces an authenticated user to perform an action they didn't intend to.

- **Scenario:** You are logged into `bank.com`. You visit `evil.com`.
    
- **Attack:** `evil.com` has a hidden form that auto-submits a POST request to `bank.com/transfer`. Since your browser sends your Cookies automatically, the bank thinks _you_ made the request.
    
- **Defense:** Anti-CSRF Tokens (Hidden random values in forms) or SameSite Cookie attributes (`SameSite=Strict`).
    

---

## 4. Server-Side Attacks

### IDOR (Insecure Direct Object Reference)

The application exposes an internal object key (ID) and fails to check authorization.

- **Request:** `GET /invoices?id=100` (Your invoice)
    
- **Attack:** Change to `GET /invoices?id=101` (Someone else's invoice).
    
- **Defense:** Check ownership (`if invoice.user_id == current_user.id`) for _every_ request.
    

### SSRF (Server-Side Request Forgery)

The attacker tricks the server into making a request to an internal resource.

- **Scenario:** An image fetcher: `POST /upload?url=http://example.com/image.png`
    
- **Attack:** `POST /upload?url=http://169.254.169.254/latest/meta-data/` (AWS Metadata service).
    
- **Impact:** Stealing cloud credentials (AWS Keys) or scanning internal ports.
    

---

## 5. Authentication vs. Authorization

- **Authentication (AuthN):** "Who are you?" (Login, MFA).
    
- **Authorization (AuthZ):** "What are you allowed to do?" (Permissions, Roles).
    

### JWT (JSON Web Tokens)

The standard for modern APIs.

- **Structure:** `Header . Payload . Signature`
    
- **The Signature:** Created using a Secret Key on the server. Prevents tampering.
    
- **The "None" Algorithm:** A classic vulnerability where attackers strip the signature and set `alg: none`.
    

---

## 6. The Hardening Checklist 📝

1. **HTTPS Everywhere:** Use HSTS (`Strict-Transport-Security`).
    
2. **Secure Cookies:** Set `Secure` (HTTPS only), `HttpOnly` (No JS access), and `SameSite`.
    
3. **Headers:**
    
    - `X-Frame-Options: DENY` (Prevents Clickjacking).
        
    - `X-Content-Type-Options: nosniff` (Prevents MIME sniffing).
        
4. **Dependencies:** Run `npm audit` or use **Snyk** to check for CVEs in your `node_modules`.
    

---

## Linked Notes

- [[SQL-Injection-Methodology]] - A deep dive into injection.
    
- [[Burp-Suite-Setup]] - The tool to test these flaws.
    
- [[JavaScript-Ultimate-Guide]] - Understanding the language of XSS.