# Security Labs — Akshay Nath V

Documented findings from hands-on web application security labs
completed on PortSwigger Web Security Academy.

Each writeup follows penetration testing report structure:
vulnerability description, reproduction steps, CVSS score,
and remediation recommendations.

---

## Progress

| Level | Completed |
|---|---|
| Apprentice | 21 / 61 |
| Practitioner | 11 / 174 |
| Total | 32 labs |

Categories covered: SQL Injection, XSS, SSRF, OS Command Injection,
Path Traversal, Access Control, Authentication, Information Disclosure

---

## SQL Injection

### Lab 01 — SQL Injection: Database Version Detection (Oracle)
**Level:** Practitioner
**CVSS Score:** 7.5 (High)

**Vulnerability:**
The application passes user input directly into a SQL query
without sanitization, allowing an attacker to manipulate
the query to extract database metadata.

**Steps to Reproduce:**
1. Intercept the category filter request using Burp Suite
2. Inject a UNION-based payload to query the Oracle version:
   `' UNION SELECT banner,NULL FROM v$version--`
3. Observe database version returned in application response

**Impact:**
Attacker can enumerate database type, version, and structure,
enabling targeted exploitation of known vulnerabilities.

**Remediation:**
- Use parameterized queries / prepared statements
- Restrict database user privileges
- Suppress verbose error messages in production

---

### Lab 02 — SQL Injection: Database Version Detection (MySQL/Microsoft)
**Level:** Practitioner
**CVSS Score:** 7.5 (High)

**Vulnerability:**
Same class as Lab 01 with syntax differences specific to
MySQL and Microsoft SQL Server environments.

**Steps to Reproduce:**
1. Intercept category filter request in Burp Suite
2. Inject UNION payload adapted for MySQL/MSSQL:
   `' UNION SELECT @@version,NULL#`
3. Confirm database version in response body

**Remediation:**
- Parameterized queries
- Disable detailed error responses
- Input validation and allowlisting

---

### Lab 03 — Blind SQL Injection with Conditional Responses
**Level:** Practitioner
**CVSS Score:** 8.8 (High)

**Vulnerability:**
Application does not return query results directly but
changes behavior based on query truth value, enabling
data extraction through boolean-based blind injection.

**Steps to Reproduce:**
1. Identify tracking cookie parameter vulnerable to injection
2. Inject conditional payload:
   `' AND '1'='1` (true — normal response)
   `' AND '1'='2` (false — modified response)
3. Enumerate data character by character using conditional logic

**Impact:**
Full database content extractable despite no visible output,
including credentials and sensitive application data.

**Remediation:**
- Parameterized queries
- Anomaly detection on repeated similar requests
- Rate limiting on authentication endpoints

---

## Cross-Site Scripting (XSS)

### Lab 04 — Reflected XSS: HTML Context, No Encoding
**Level:** Apprentice
**CVSS Score:** 6.1 (Medium)

**Vulnerability:**
User input reflected directly into HTML response without
encoding, allowing script injection via crafted URL.

**Steps to Reproduce:**
1. Locate search parameter reflected in page response
2. Inject: `<script>alert(1)</script>`
3. Confirm JavaScript executes in browser

**Remediation:**
- Output encode all user-supplied data
- Implement Content Security Policy (CSP)

---

### Lab 05 — Stored XSS: HTML Context, No Encoding
**Level:** Apprentice
**CVSS Score:** 8.2 (High)

**Vulnerability:**
User input stored in database and rendered without encoding,
allowing persistent script execution for all users who
view the affected page.

**Steps to Reproduce:**
1. Submit `<script>alert(1)</script>` in comment field
2. Confirm payload stored and executes on page load
3. Verify persistence across sessions

**Impact:**
Stored XSS can be used for session hijacking, credential
theft, and malware delivery at scale.

**Remediation:**
- Sanitize input on storage
- Encode output on render
- Implement strict CSP headers

---

### Lab 06 — DOM XSS in document.write Sink
**Level:** Apprentice
**CVSS Score:** 6.1 (Medium)

**Vulnerability:**
Application passes location.search source into
document.write sink without sanitization, enabling
DOM-based XSS via crafted URL parameter.

**Steps to Reproduce:**
1. Identify search term written into DOM via document.write
2. Inject: `"><img src=1 onerror=alert(1)>`
3. Confirm execution without server-side reflection

**Remediation:**
- Avoid dangerous sinks (document.write, innerHTML)
- Use safe DOM APIs (textContent, createElement)
- Implement CSP

---

### Labs 07-12 — Additional XSS Variants

| Lab | Type | Context | Level |
|---|---|---|---|
| innerHTML sink via location.search | DOM | innerHTML | Apprentice |
| jQuery href attribute sink | DOM | jQuery anchor | Apprentice |
| jQuery selector hashchange | DOM | jQuery selector | Apprentice |
| Reflected XSS, angle brackets encoded | Reflected | HTML attribute | Apprentice |
| Stored XSS in href attribute | Stored | href attribute | Apprentice |
| Reflected XSS in JavaScript string | Reflected | JS string | Apprentice |

---

## Server-Side Request Forgery (SSRF)

### Lab 13 — Basic SSRF Against Local Server
**Level:** Apprentice
**CVSS Score:** 9.8 (Critical)

**Vulnerability:**
Application fetches URLs supplied by user without
validation, enabling requests to internal services
not exposed externally.

**Steps to Reproduce:**
1. Intercept stock check request in Burp Suite
2. Modify stockApi parameter to:
   `http://localhost/admin`
3. Observe internal admin panel returned in response

**Impact:**
Full access to internal administrative interfaces,
bypassing all external access controls.

**Remediation:**
- Validate and allowlist permitted URLs and IP ranges
- Block requests to localhost and internal IP ranges
- Use egress firewall rules

---

### Lab 14 — SSRF with Blacklist-Based Input Filter Bypass
**Level:** Practitioner
**CVSS Score:** 9.1 (Critical)

**Vulnerability:**
Application attempts to block internal addresses via
blacklist but filter is bypassable using alternative
representations of localhost.

**Steps to Reproduce:**
1. Confirm basic `http://localhost/admin` is blocked
2. Bypass using: `http://127.1/admin` or `http://2130706433/admin`
3. Confirm admin panel accessible despite blacklist

**Impact:**
Demonstrates that blacklist-based SSRF protection is
insufficient — only allowlists provide reliable defense.

**Remediation:**
- Replace blacklist with strict allowlist
- Resolve URLs server-side before validation
- Segment internal network access

---

## OS Command Injection

### Lab 15 — OS Command Injection: Simple Case
**Level:** Apprentice
**CVSS Score:** 9.8 (Critical)

**Vulnerability:**
User input passed directly to OS shell command without
sanitization, enabling arbitrary command execution.

**Steps to Reproduce:**
1. Intercept stock check request
2. Inject into productId parameter: `1|whoami`
3. Observe OS username returned in response

**Remediation:**
- Never pass user input to shell commands
- Use language-native APIs instead of shell calls
- Apply strict input validation

---

### Lab 16 — Blind OS Command Injection with Time Delays
**Level:** Practitioner
**CVSS Score:** 9.8 (Critical)

**Vulnerability:**
Command injection exists but produces no visible output.
Exploitable by measuring response time differences to
confirm execution and exfiltrate data.

**Steps to Reproduce:**
1. Inject time-delay payload into email parameter:
   `email=x||ping+-c+10+127.0.0.1||`
2. Measure response time — 10 second delay confirms injection
3. Use delay-based technique to enumerate system data

**Remediation:**
- Input sanitization at all entry points
- Application-level anomaly detection on slow responses

---

### Lab 17 — Blind OS Command Injection with Output Redirection
**Level:** Practitioner
**CVSS Score:** 9.8 (Critical)

**Vulnerability:**
Blind injection with no direct output — exploited by
redirecting command output to a web-accessible file
for retrieval.

**Steps to Reproduce:**
1. Inject: `email=||whoami>/var/www/images/output.txt||`
2. Fetch `/image?filename=output.txt` to retrieve output
3. Confirm arbitrary command output retrievable

**Remediation:**
- Restrict write permissions on web-accessible directories
- Input sanitization
- Monitor for unusual file creation in web roots

---

## Path Traversal

### Labs 18-23 — Path Traversal Variants

| Lab | Bypass Technique | Level | CVSS |
|---|---|---|---|
| Simple case | `../../../etc/passwd` | Apprentice | 7.5 |
| Absolute path bypass | `/etc/passwd` directly | Practitioner | 7.5 |
| Stripped sequences (non-recursive) | `....//....//etc/passwd` | Practitioner | 7.5 |
| Superfluous URL decode | `..%252f..%252fetc/passwd` | Practitioner | 7.5 |
| Start of path validation | `/var/www/images/../../../etc/passwd` | Practitioner | 7.5 |
| Null byte bypass | `../../../etc/passwd%00.png` | Practitioner | 7.5 |

**Key Learning:** Six different bypass techniques defeated six
different developer attempts to patch path traversal.
Demonstrates why input allowlisting beats blacklisting.

---

## Access Control

### Labs 24-27 — Access Control Vulnerabilities

| Lab | Vulnerability | Level |
|---|---|---|
| Unprotected admin functionality | No auth on admin URL | Apprentice |
| Admin with unpredictable URL | Security through obscurity | Apprentice |
| Role controlled by request parameter | Client-side role parameter | Apprentice |
| Role modified in user profile | Mass assignment vulnerability | Apprentice |

---

## Authentication

### Lab 28 — Username Enumeration via Different Responses
**Level:** Apprentice
**CVSS Score:** 5.3 (Medium)

**Steps to Reproduce:**
1. Submit login attempts with wordlist of usernames
2. Observe response differences for valid vs invalid usernames
3. Enumerate valid username, then brute force password

---

### Lab 29 — 2FA Simple Bypass
**Level:** Apprentice
**CVSS Score:** 8.8 (High)

**Vulnerability:**
2FA code requested after password but application does
not enforce completion — direct URL navigation bypasses
second factor entirely.

**Remediation:**
- Enforce 2FA state server-side before granting access
- Do not rely on client-side flow control for auth steps

---

## Information Disclosure

### Lab 30 — Information Disclosure in Error Messages
**Level:** Apprentice

**Vulnerability:**
Verbose error messages expose framework version and
internal path information useful for targeted attacks.

---

### Lab 31 — Source Code Disclosure via Backup Files
**Level:** Apprentice

**Vulnerability:**
Backup files left in web root expose application source
code including hardcoded credentials and internal logic.

---

*Last updated: June 2026*
*Platform: PortSwigger Web Security Academy*
*Author: Akshay Nath V — github.com/telnetghost*
