
# Network and Security Lab

This repository documents the design, deployment, and validation of a
controlled internal network security lab built to mirror entry-level
enterprise security environments.

The objective of this lab is not tool usage, but **understanding how
network behavior, service exposure, and security controls manifest at
the packet and system level**.

All activities are performed in isolated virtual environments and are
fully documented for auditability and repeatability.

---

## Lab Objectives

- Design a segmented internal lab network
- Validate host-to-host communication at Layer 2–7
- Observe traffic behavior using packet analysis
- Control service exposure using host-based firewalls
- Produce professional security documentation

---

## Environment Overview

| Component | Description |
|--------|-------------|
| Host OS | Windows |
| Hypervisor | VirtualBox |
| Attacker VM | Kali Linux |
| Target VM | Linux Virtual Machine |
| Network Type | Internal / Host-Only |
| Addressing | DHCP (host-managed) |
| Traffic Scope | Layer 2 to Layer 7 |

---
