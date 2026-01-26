# Phase 1.3 — Firewall Control & Network Governance

## Objective

This phase implements and validates host-based firewall controls on an internal Linux target system to enforce a controlled, minimal, and auditable attack surface. The emphasis is on policy enforcement, visibility, and packet-level verification rather than indiscriminate traffic blocking.

The objective is to demonstrate:
- What traffic is explicitly allowed
- What traffic is denied
- Why those outcomes occur at the packet level

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

Before applying any security controls, the firewall state was verified to ensure no pre-existing rules were influencing traffic behavior.

- Firewall status confirmed as inactive
- No inbound or outbound filtering present

This established a clean baseline and ensured all observed behavior resulted solely from newly applied policies.

---

## Default Firewall Policy Configuration

A restrictive default policy was applied in alignment with enterprise host-hardening standards:

- Inbound traffic denied by default
- Outbound traffic allowed by default

This posture ensures that inbound access must be explicitly approved while preserving system functionality.

---

## Controlled Service Exposure — HTTP

Inbound access was explicitly limited to a single approved service:

- TCP port 80 (HTTP) allowed
- Firewall enabled after rule definition

Firewall verification confirmed:
- Default inbound policy: deny
- Default outbound policy: allow
- Only TCP port 80 permitted inbound

This resulted in a minimal and auditable attack surface.

---

## Network Validation and Packet-Level Observation

Traffic behavior was validated from the attacker system while capturing packets on the target’s internal interface.

### ICMP Reachability

- ICMP echo requests transmitted
- No echo replies received
- No ICMP responses observed in packet capture

Assessment:  
ICMP traffic was correctly blocked by the default inbound deny policy, confirming firewall enforcement.

---

### HTTP Connectivity

- TCP three-way handshake completed successfully
- HTTP response returned
- Application-layer traffic visible in packet capture

Assessment:  
The firewall correctly permitted the explicitly allowed service while maintaining restrictions on all other inbound traffic.

---

## SSH Access Analysis and Troubleshooting

Initial SSH connection attempts failed despite no firewall rule explicitly blocking SSH.

Observed behavior:
- Connection failure
- TCP RST packets visible in packet capture

Root Cause:  
The SSH service was not running on the target system. The failure was service-related, not firewall-related, demonstrating the distinction between filtering and service availability.

---

## Controlled SSH Enablement (Validation Test)

SSH was temporarily enabled to validate firewall behavior against an additional service.

Observed behavior:
- Successful TCP handshake
- SSH protocol negotiation observed
- Encrypted SSH traffic visible in packet capture

Assessment:  
This confirmed correct firewall behavior and demonstrated encrypted application-layer traffic traversal.

---

## SSH Exposure Rollback

SSH access was revoked to restore the minimal attack surface.

Observed behavior:
- Subsequent SSH attempts resulted in TCP RST packets
- No session establishment possible

Assessment:  
The rollback confirmed effective firewall governance and clean removal of exposure.

---

## Key Findings

- Default-deny inbound policy effectively restricted unsolicited traffic
- Explicit allow rules functioned as intended
- Packet captures clearly differentiated filtered ports from inactive services
- Firewall changes were observable, measurable, and reversible

---

## Security Outcome

This phase successfully demonstrated:
- Host-based firewall governance
- Controlled service exposure
- Packet-level validation of security controls
- Professional troubleshooting methodology

The environment is now prepared for controlled vulnerable asset deployment and baseline traffic analysis in subsequent phases.
