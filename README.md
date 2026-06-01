# security-labs
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

**Vulnerab
