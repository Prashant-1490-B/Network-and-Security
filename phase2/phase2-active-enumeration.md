# PHASE 2.2 — Active Enumeration (Controlled, Intentional)

## Engagement Context

- **Target Application:** Damn Vulnerable Web Application (DVWA)
- **Target Host:** 10.10.10.102
- **Application URL Base:** http://10.10.10.102/DVWA/
- **Security Level:** LOW (Explicitly configured for training and testing)
- **Testing Type:** Internal, authenticated web application testing
- **Objective:** Identify application structure, reachable endpoints, functional modules, and user-controllable input surfaces without performing exploitation.

> ⚠️ This phase intentionally avoids payloads, automated exploitation tools, or vulnerability confirmation.  
> The goal is **enumeration and understanding**, not exploitation.

---

## 1. Enumeration Philosophy

Active enumeration answers the following professional questions:

- What application functionality exists?
- What endpoints and directories are reachable?
- What parameters are user-controlled?
- Where does user input flow?
- Which parameters represent higher trust boundaries?

This phase establishes **where exploitation may occur later**, not whether it already has.

---

## 2. Directory & Endpoint Enumeration

### Tool Used
- **Gobuster** (directory enumeration mode)

### Command Executed

gobuster dir \
  -u http://10.10.10.102/DVWA/ \
  -w /usr/share/wordlists/dirb/common.txt \
  -t 30

---
Observed Results (Categorized)
2.1 Protected or Forbidden Resources (HTTP 403)

/.hta

/.htaccess

/.htpasswd

Interpretation:
These files exist but are access-restricted by the web server. This behavior is expected and not considered a security finding.

2.2 Accessible Resources (HTTP 200)

/.git/HEAD

/favicon.ico

/robots.txt

/php.ini

Interpretation:
These endpoints are reachable and represent potential configuration or metadata exposure surfaces. No content analysis or exploitation was performed during this phase.

2.3 Redirected Directories (HTTP 301 / 302)

/config/

/database/

/docs/

/external/

/tests/

Interpretation:
These directories exist within the application structure but are redirected, indicating intentional access control or internal-only use.

---

# ✅ PART 2 — Application & Parameter Enumeration (Reflected + Stored XSS)


## Application-Level Enumeration (Authenticated)

### Authentication Context

- Login performed using known test credentials provided by the application:
  - **Username:** admin
  - **Password:** password
- This was documented as authorized test credential usage.

---

## Identified Application Modules

The following functional modules were identified through authenticated navigation:

- Reflected Cross-Site Scripting (XSS)
- Stored Cross-Site Scripting (XSS)
- SQL Injection
- SQL Injection (Blind)
- Command Injection
- File Inclusion
- File Upload
- CSRF
- Weak Session IDs
- XSS (DOM)
- CSP Bypass
- JavaScript Attacks

These modules represent **input-handling functionality**, not confirmed vulnerabilities at this stage.

---

## Reflected XSS — Controlled Enumeration

### Request Characteristics
- **Method:** GET
- **Endpoint:** `/DVWA/vulnerabilities/xss_r/`
- **Parameter Identified:** `name`

### Observations
- User input is passed via a query parameter.
- Input is reflected directly in the HTTP response.
- Reflection occurs in the HTML body context.
- No output encoding was observed at LOW security level.

### Interpretation
This indicates direct trust of user-controlled input in the response generation path.

---

## Stored XSS — Controlled Enumeration

### Request Characteristics
- **Method:** POST
- **Endpoint:** `/DVWA/vulnerabilities/xss_s/`
- **Parameters Identified:**
  - `txtName`
  - `txtMessage`
  - `btnSign`

### Observations
- Submitted input is persisted server-side.
- Stored content is rendered during subsequent page loads without resubmission.
- Input transport uses URL encoding (`application/x-www-form-urlencoded`), which is decoded server-side.

---

## Parameter Handling Differences

During enumeration, inconsistent server-side handling was observed:

| Parameter   | Behavior |
|------------|----------|
| txtName    | Does not render when supplied with HTML-like input |
| txtMessage | Stored and rendered persistently without encoding |

### Interpretation
This indicates differential server-side processing and a higher trust boundary associated with the `txtMessage` parameter.

---
## Security Posture Assessment

The application was explicitly configured at **LOW security level**, which affects:

- Input validation
- Output encoding
- Server-side sanitization behavior

All observations must be interpreted within this intentionally vulnerable context.

---

## Identified Security Weakness Classes (Non-Exploitative)

The following vulnerability categories were **identified conceptually** through enumeration and analysis:

- Lack of output encoding for user-controlled data
- Persistent storage of unsanitized user input
- Inconsistent parameter validation logic
- Cleartext HTTP transport (no TLS)
- Session-based authentication without secure transport attributes

> No exploitation or payload execution was performed during this phase.

---

## Key Learning Outcomes

This phase demonstrated:

- Structured directory and endpoint discovery
- Authentication boundary identification
- Input flow mapping from client to server to response
- Differentiation between transport encoding and sanitization
- Identification of higher-risk parameters through logic analysis

---

## Phase 2.2 Conclusion

PHASE 2.2 successfully established:

- The application’s reachable attack surface
- Functional modules accepting user input
- Parameter trust boundaries
- Areas suitable for controlled exploitation in subsequent phases

No active exploitation was performed.  
The environment is now ready for **PHASE 2.3 — Controlled Exploitation**, where a single parameter and single impact will be validated with full evidence.

## Documentation of Phase 2.2

[Active Web Enumeration.pdf](https://github.com/user-attachments/files/24961175/Active.Web.Enumeration.pdf)


