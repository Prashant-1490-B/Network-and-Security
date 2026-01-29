# Phase 2.1 – Reconnaissance & Asset Understanding

## Objective

The objective of **Phase 2.1** was to perform **controlled reconnaissance and initial asset understanding** of the target web application.  
This phase focuses on **environment discovery, application behavior analysis, authentication flow observation, and session handling review**, without attempting any exploitation or vulnerability confirmation.

---

## Scope Validation

All reconnaissance activities were performed **strictly within the authorized engagement scope**.

- Target application: **Damn Vulnerable Web Application (DVWA)**
- Exposure: **HTTP service over TCP port 80**
- Network interaction: **Limited only to explicitly authorized URLs**
- No interaction was performed with:
  - External services
  - Non-target hosts
  - Unauthorized endpoints
  - Background system services

> **Scope adherence was maintained at all times. No actions exceeded the defined boundaries of the engagement.**

---

## Target Asset Overview

### Application Type
- **Web Application**
- **Server-side language:** PHP
- **Authentication model:** Form-based authentication

### Web Server & Platform
- **Web Server:** Apache HTTP Server
- **Version:** Apache/2.4.66
- **Operating System:** Debian-based Linux distribution

### Transport Layer
- **Protocol:** HTTP (unencrypted)
- **TLS/SSL:** Not implemented

---

## Application Exposure & Entry Points

### Identified Entry Points

| Component | Description |
|---------|------------|
| `/DVWA/login.php` | Primary authentication endpoint |
| `/DVWA/` | Main application root |
| HTTP POST Parameters | Used for credential submission |

The application exposes a **traditional login form**, which submits credentials via HTTP POST requests.

---

## Authentication Flow Analysis

### Login Mechanism

- Authentication requests are handled by:
---
## /DVWA/login.php

- User credentials are submitted using **HTTP POST**
- On successful authentication:
- Server responds with **HTTP 302 Redirect**
- User is redirected to an authenticated application page

### Observations

- No multi-factor authentication (MFA) mechanisms observed
- No CAPTCHA or rate-limiting controls observed during login
- Login responses are deterministic and predictable

---

## Session Management Observations

### Session Handling

- Session management is implemented using **cookie-based sessions**
- Session identifier observed:

---
  
## PHPSESSID


### Key Findings

- Session cookies are transmitted **in plaintext** due to lack of TLS
- Session identifiers are visible in network traffic
- No additional session hardening flags observed during reconnaissance:
- `Secure` flag not enforced
- `HttpOnly` flag behavior not validated in this phase
- `SameSite` attribute not confirmed

> Session behavior has been documented for controlled testing in later phases.

---

## Transport Security Observations

### HTTP Traffic Characteristics

- All application traffic is transmitted over **unencrypted HTTP**
- Credentials and session identifiers are **observable in plaintext**
- Susceptible to:
- Network sniffing
- Credential interception
- Session hijacking (theoretical at this phase)

> **No exploitation or active interception was performed in this phase.**

---

## Input & Parameter Awareness

During passive observation, the following were noted:

- Login parameters are predictable and consistent
- Input fields accept raw user input
- No client-side obfuscation or encryption mechanisms observed

These observations will assist in:
- Input validation testing
- Injection testing
- Authentication bypass testing (future phases)

---

## Reconnaissance Outcome

- No vulnerabilities are **confirmed or exploited** in this phase
- The following areas have been **mapped and documented**:
- Application stack
- Authentication workflow
- Session handling behavior
- Transport security posture
- Visible attack surface

This phase serves as a **foundation** for structured and controlled testing in subsequent phases.

---
