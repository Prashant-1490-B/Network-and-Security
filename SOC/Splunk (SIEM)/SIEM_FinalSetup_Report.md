# 🛡️ Enterprise SIEM Ingestion Architecture & Endpoint Telemetry Engineering

> **Author:** B. Prashant &nbsp;|&nbsp; **Ref:** SIEM-ENG-2026-REV2 &nbsp;|&nbsp; **Classification:** Technical Lab Documentation

---

## 📋 Table of Contents

1. [Phase 2 – Ingestion Port Activation & Socket Architecture](#4-phase-2--ingestion-port-activation--socket-architecture)
2. [Phase 3 – Endpoint Telemetry Engineering & Event Log Ingestion](#5-phase-3--endpoint-telemetry-engineering--event-log-ingestion)
3. [Phase 4 – Infrastructure Hardening & Compliance Controls](#6-phase-4--infrastructure-hardening--compliance-controls)
4. [Final Assessment Summary](#7-final-assessment-summary)
5. [Industry Relevance](#8-industry-relevance)
6. [Skills Demonstrated](#9-skills-demonstrated)


---Continue of SIEM_Presetup_Report.md

## 1. Phase 2 – Ingestion Port Activation & Socket Architecture

### 1.1 Test Description

With the network layer aligned, a connection probe was executed from the Windows endpoint to validate TCP port `9997` (Splunk-to-Splunk ingestion listener).

```powershell
Test-NetConnection -ComputerName 192.168.1.150 -Port 9997
```

### 1.2 Observations

| Result | Detail |
|---|---|
| **Initial Probe** | `TcpTestSucceeded : False` |
| **Socket Audit** | `sudo ss -tulpn \| grep 9997` → empty output |
| **Root Cause** | Splunk was running but had no explicit binding directive for port `9997` |

### 1.3 Technical Findings — Remediation Sequence

**Step 1 — Force CLI Binding**

```bash
cd /opt/splunk/bin
sudo ./splunk enable listen 9997 -auth admin:YourSecurePassword123 --run-as-root
```

**Step 2 — Persist Configuration to Disk**

```bash
sudo nano /opt/splunk/etc/system/local/inputs.conf
```

```ini
[splunktcp://9997]
disabled = 0
```

**Step 3 — Restart Service**

```bash
sudo /opt/splunk/bin/splunk restart --run-as-root
```

**Step 4 — Post-Remediation Verification**

```bash
sudo ss -tulpn | grep 9997
```

```
State   Recv-Q  Send-Q  Local Address:Port  Peer Address:Port
LISTEN  0       128     0.0.0.0:9997         0.0.0.0:*   users:(("splunkd",pid=19022,fd=174))
```

### 1.4 Assessment

Repeat PowerShell probe confirmed full success:

```
ComputerName     : 192.168.1.150
RemotePort       : 9997
SourceAddress    : 192.168.1.110
TcpTestSucceeded : True ✓
```

> `splunkd` bound to wildcard `0.0.0.0:9997` — accepting connections on all interfaces. The ingestion pipeline is operational.

---

## 2. Phase 3 – Endpoint Telemetry Engineering & Event Log Ingestion

### 2.1 Test Description

With the network data pipeline established, the **Universal Forwarder** on the Windows host was configured to monitor local directories and subscribe to live Windows Event Log channels.

### 2.2 Observations

- Monitoring `C:\Windows\System32` produced **no indexed events** — the directory consists of static binary files (`.exe`, `.dll`, `.sys`) which Splunk's text-stream parser ignores.
- A dynamic test confirmed the file-monitor engine was functional:

```cmd
echo "SIEM_DEPLOYMENT_TEST_INITIATED" > C:\Windows\System32\test_log.txt
```

The forwarder detected the ASCII payload, packaged it, and transmitted it to the indexer.

- Configuration was then migrated to **live Windows Event Log subscriptions** for continuous security telemetry.

### 2.3 Technical Findings — Configuration Files

**`outputs.conf` — Transmission Target**
> `C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf`

```ini
[tcpout:default-autolb-group]
server = 192.168.1.150:9997

[tcpout-server://192.168.1.150:9997]
```

**`inputs.conf` — Event Log Subscription**
> `C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`

```ini
[default]
host = DESKTOP-WINPROD

[WinEventLog://Application]
disabled = 0
index = windows_logs
checkpointInterval = 5
renderXml = false
```

**Service Restart**

```cmd
net stop SplunkForwarder && net start SplunkForwarder
```

### 2.4 Temporal Synchronisation Troubleshooting

**Problem:** `index="windows_logs"` returned zero results on initial search.

**Root Cause:** Clock skew between the virtualised Kali clock (UTC) and the physical Windows CMOS clock (local time). Splunk indexed events using the Windows timestamp; the Kali indexer's offset caused the default `Last 24 Hours` filter to exclude them as future-dated.

**Immediate Fix:** Changed search time range to `All Time`.

**Permanent Fix:** Synchronise both systems to the same NTP source.

### 2.5 Assessment

> Live Windows Application Event Log data is now flowing continuously into `index=windows_logs` on the Kali SIEM. The telemetry pipeline is fully operational end-to-end.

---

## 3. Phase 4 – Infrastructure Hardening & Compliance Controls

### 3.1 Test Description

An intentional Nmap scan of the Kali indexer revealed multiple exposed interfaces, triggering a formal security audit and hardening cycle.

```
PORT     STATE  SERVICE
8000/tcp open   http-alt
8089/tcp open   unknown
```

### 3.2 Observations — Threat Modelling

| Port | Service | Threat Vector |
|---|---|---|
| `8000/tcp` | Splunk Web UI | Default **HTTP** (cleartext). Session cookies and credentials exposed to packet sniffing on shared segments. |
| `8089/tcp` | REST API | HTTPS with **self-signed certs**. Vulnerable to MitM certificate replacement and network-wide brute-force. |
| `9997/tcp` | S2S Ingestion | Wildcard binding (`0.0.0.0:9997`) accepts unauthenticated data from any source — log-forgery and DoS vectors. |

### 3.3 Technical Findings — Security Controls

#### Control A — Host-Based Firewall Segmentation (`ufw`)

Default-drop inbound policy with a whitelist for the trusted Windows endpoint only:

```bash
sudo ufw default deny incoming
sudo ufw default allow outbound
sudo ufw allow from 192.168.1.110 to any port 9997 proto tcp
sudo ufw enable
```

> Any connection attempt to port `9997` from an unknown IP is silently dropped at the kernel layer.

#### Control B — Persistent Queuing (Data Resiliency)

Prevents telemetry loss during SIEM maintenance or network outages:

```ini
# outputs.conf
[tcpout:default-autolb-group]
server = 192.168.1.150:9997
maxQueueSize = 500MB
persistentQueuePath = C:\Program Files\SplunkUniversalForwarder\var\run\splunk\persistent_queue
```

#### Control C — Throughput Throttling (Licence Compliance)

Splunk free tier enforces a **500 MB/day** ingestion ceiling. Post-outage catch-up bursts can breach this in minutes. The fix:

```ini
# limits.conf
[thruput]
maxKBps = 256
```

> Caps outbound throughput at **256 KB/s**, distributing catch-up data evenly and protecting the daily quota.

### 3.4 Assessment

> All three controls operate in concert: the firewall eliminates unauthorised access; the persistent queue preserves telemetry during outages; the throughput limiter prevents licence violations during recovery. The deployment is now hardened and compliant.

---

## 4. Final Assessment Summary

```
[ Windows Host Endpoint ]                         [ Kali Linux SIEM Indexer ]
+---------------------------+                     +----------------------------+
| Win32 Logging Engine      |                     | Inbound Policy: DROP ALL   |
| WinEventLog: Application  |                     | Whitelist: 192.168.1.110   |
+-----------+---------------+                     +--------------+-------------+
            |                                                    ^
            v                                                    |
+---------------------------+                                    |
| Local Persistent Queue    |              [Controlled Stream]   |
| 500 MB Buffer             | ------- Max 256 KBps TCP -------> |
+-----------+---------------+
            |
            v
+---------------------------+
| Throughput Throttle       |
| limits.conf: 256 KBps     |
+---------------------------+
```

| Layer | Control Applied | Outcome |
|---|---|---|
| **Windows Endpoint** | WinEventLog subscription + 500 MB local buffer | Continuous OS telemetry with local resilience |
| **Network Transport** | 256 KBps throttle (`limits.conf`) | Controlled catch-up; 500 MB/day licence respected |
| **Kali SIEM Indexer** | `ufw` DEFAULT DROP + whitelist `192.168.1.110:9997` | Unauthorised scan attempts silently dropped |

---

## 5. Industry Relevance

### 🔵 Security Operations (SOC Engineering)

- **Ingestion Monitoring** — Establishes enterprise-wide visibility by ingesting telemetry from disparate systems into a centralised SIEM.
- **Data Normalisation** — Hands-on parsing of system event logs, index health verification, and ingestion pipeline troubleshooting.
- **Alert Validation** — Socket-level diagnostics distinguish firewall drops from OS-level rejections, supporting SOC triage workflows.

### 🔴 Offensive Security & Penetration Testing

- **Attack Surface Evaluation** — Demonstrates how open administrative portals and cleartext services expose corporate environments to automated scanning.
- **Port State Differentiation** — Teaches analysts to distinguish open, closed, and firewall-filtered ports when assessing target networks.

### 🟡 Digital Forensics & Incident Response (DFIR)

- **Log Reliability** — Ensures forensic evidence is preserved across network boundaries during security incidents.
- **Timeline Generation** — Cross-platform clock normalisation promotes accuracy in security event timeline reconstruction.

---

## 6. Skills Demonstrated

| Skill Area | Competency |
|---|---|
| **Distributed SIEM Architecture** | Deploying and maintaining multi-node logging environments across heterogeneous operating systems |
| **Network Diagnosis & Socket Inspection** | Using `ss`, `netstat`, `Test-NetConnection` to analyse connection lifecycles and resolve socket binding conflicts |
| **Traffic Flow Engineering** | Applying rate-limiting (`limits.conf`) and persistent-queue buffering to regulate utilisation and maintain licence compliance |
| **Defensive Edge Hardening** | Designing `ufw` firewall ACLs to restrict service visibility and reduce the attack surface of critical infrastructure |
| **Log Reliability & DFIR Readiness** | Preserving forensic evidence across network boundaries and normalising clock discrepancies for accurate timeline reconstruction |


<div align="center">

**SIEM-ENG-2026-REV2** &nbsp;·&nbsp; B. Prashant &nbsp;·&nbsp; Technical Lab Documentation

</div>
