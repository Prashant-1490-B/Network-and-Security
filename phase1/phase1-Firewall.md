[Internal Network Visibility and Firewall Control.pdf](https://github.com/user-attachments/files/24597892/Internal.Network.Visibility.and.Firewall.Control.pdf)


# Phase 1.3 — Firewall Control & Network Governance

## Objective

Implement and validate host-based firewall controls on the internal target
system to enforce a **controlled and auditable attack surface**. This phase
focuses on **policy enforcement, visibility, and verification**, not on
blocking traffic blindly.

The goal is to prove:
- What traffic is allowed
- What traffic is denied
- Why that behavior occurs at the packet level

---

## Environment Overview

| Component | Description |
|---------|-------------|
| Attacker System | Kali Linux |
| Attacker IP | 10.10.10.101 |
| Target System | Linux Virtual Machine |
| Target IP | 10.10.10.102 |
| Network Type | Internal / Host-Only |
| Firewall | UFW (iptables backend) |
| Capture Interface | eth1 |
| Tools Used | ufw, nmap, curl, ssh, Wireshark |

---

## Firewall Baseline Verification

Before applying any security controls, the firewall state was verified to
ensure no pre-existing rules were influencing traffic behavior.

- **Command**

sudo ufw status

- **Observed State**

Firewall status: inactive

- **Assessment**

Confirmed a clean baseline with no inbound filtering or legacy rules applied. This establishes an accurate reference point for all subsequent changes.

Default Firewall Policy Configuration
A restrictive default policy was applied to align with enterprise host hardening standards.

- **Commands**

sudo ufw default deny incoming
sudo ufw default allow outgoing

Inbound traffic must be explicitly approved
Outbound traffic remains unrestricted for system functionality
<br>
This default-deny posture mirrors standard internal security baselines.

Controlled Service Exposure — HTTP
Inbound access was explicitly limited to a single approved service.

- **Commands**

sudo ufw allow 80/tcp
sudo ufw enable

- **Verification**

sudo ufw status verbose

- **Observed State**

Incoming: deny (default)
Outgoing: allow (default)
Allowed service: TCP 80 (HTTP)

- **Assessment**

The target system exposes only the approved HTTP service, maintaining a minimal and auditable attack surface.

Network Validation and Packet-Level Observation
Traffic behavior was validated from the attacker system while capturing packets on the internal interface.

- **ICMP Reachability Test**

- **Command**

ping 10.10.10.102

- **Observed Behavior**

ICMP Echo Requests transmitted
No Echo Replies received
No ICMP responses observed in packet capture

- **Assessment**

ICMP traffic was blocked due to the default inbound deny policy, confirming correct firewall enforcement.

- **HTTP Connectivity Test**

curl http://10.10.10.102

- **Observed Behavior**

TCP three-way handshake completed
HTTP response successfully returned
Application-layer traffic visible in capture
Assessment

Firewall rules correctly permitted the approved service while maintaining restrictions on other inbound traffic.

SSH Access Analysis and Troubleshooting
Initial SSH connection attempts failed despite firewall configuration.
Command (Attacker)
Copy code
Bash
ssh user@10.10.10.102
Observed Behavior
Connection failure
TCP RST packets observed
Root Cause
SSH service was not running on the target system
Failure was service-related, not firewall-related
SSH Service Enablement (Controlled Test)
SSH was enabled temporarily to validate firewall behavior against an additional service.
Commands (Target)
Copy code
Bash
sudo systemctl start ssh
sudo ufw allow ssh
Observed Behavior
Successful TCP handshake
SSH protocol negotiation observed
Encrypted SSH packets visible in capture
Assessment
Demonstrated clear distinction between:
Firewall filtering
Service availability
Encrypted application-layer traffic
SSH Exposure Rollback
SSH access was revoked to restore the minimal attack surface.
Command
Copy code
Bash
sudo ufw delete allow ssh
Observed Behavior
Subsequent SSH attempts resulted in TCP RST packets
No session establishment possible
Assessment
Confirmed clean rollback of exposure and effective firewall governance.
Key Findings
Default-deny inbound policy effectively restricted unsolicited traffic
Explicit allow rules functioned as intended
Packet captures clearly differentiated filtered ports from inactive services
Firewall changes were observable, measurable, and reversible
Security Outcome
This phase successfully demonstrated:
Host-based firewall governance
Controlled service exposure
Accurate packet-level validation
Professional troubleshooting methodology
The environment is now prepared for controlled vulnerable asset deployment and baseline traffic analysis in subsequent phases.
