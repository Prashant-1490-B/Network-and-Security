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

**Command**
```bash
sudo ufw status
