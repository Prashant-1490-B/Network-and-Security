# 🛡️ Enterprise SIEM Ingestion Architecture & Endpoint Telemetry Engineering

> **Author:** B. Prashant &nbsp;|&nbsp; **Ref:** SIEM-ENG-2026-REV2 &nbsp;|&nbsp; **Classification:** Technical Lab Documentation

---

## 📋 Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Environment Overview](#2-environment-overview)
3. [Phase 1 – Virtual Network Topology & Bridged Adapter Optimisation](#3-phase-1--virtual-network-topology--bridged-adapter-optimisation)
---

## 1. Executive Summary

This report documents the full engineering lifecycle of a distributed **Security Information and Event Management (SIEM)** architecture — from initial network topology design through transport-layer activation, endpoint telemetry configuration, and defensive hardening.

The primary objective was to establish a **secure, stateful logging pipeline** between a production Windows Enterprise endpoint and a virtualised Kali Linux Splunk Indexer instance.

**Key outcomes achieved:**

- ✅ Resolved virtual network isolation via Bridged Adapter migration
- ✅ Activated and verified Splunk TCP ingestion listener on port `9997`
- ✅ Configured live Windows Event Log subscriptions to the SIEM index
- ✅ Hardened the deployment with firewall ACLs, persistent queuing, and throughput throttling

---

## 2. Environment Overview

| Infrastructure Node | Platform | Network Config | Operational Scope |
|---|---|---|---|
| **Central SIEM Indexer** | Kali Linux (Rolling Release) | Bridged Physical LAN — Dynamic/Static | Logging engine. Listens on TCP `8000` (Web UI), `8089` (REST API), `9997` (S2S Ingestion) |
| **Telemetry Producer** | Windows Enterprise Host | Physical NIC — Shared Broadcast Domain | Runs `SplunkUniversalForwarder` (`splunkd`) to capture and forward OS events |
| **Virtualisation Layer** | Type-2 Hypervisor | Bridged Network Adapter | Maps virtual guest interfaces onto physical Layer-2, removing routing isolation |
| **Capture Boundaries** | Win32 Audit Subsystem | Persistent Files & Live Event Channels | Folder auditing, file delta monitoring, high-velocity application log ingestion |

---

## 3. Phase 1 – Virtual Network Topology & Bridged Adapter Optimisation

### 3.1 Test Description

In its baseline state the Kali Linux VM was isolated inside a **software-defined NAT engine**. NAT drops arbitrary inbound connections from external physical hosts, making a stateful logging channel impossible. The virtual interface layer had to be re-engineered to a Bridged Adapter profile.

### 3.2 Observations

- ICMP echo requests from the Windows host (`192.168.1.110`) to the Kali NAT address (`10.0.2.15`) returned unroutable failures — no interface mappings existed in the host forwarding table.
- The VM was halted and the virtual NIC migrated from **NAT mode → Bridged Adapter**, binding it directly to the host's active physical NIC.
- On reboot, the network stack executed a DHCP discover sequence and acquired a routable IP on the physical LAN subnet.

```bash
sudo dhclient -v eth0
# Acquired: 192.168.1.150/24
```

### 3.3 Technical Findings

- Bridged mode exposes the guest OS directly to the LAN routing layer
- Windows host can now establish raw TCP sessions to any Kali listening port without virtual-layer encapsulation overhead
- **Flat network transparency achieved** — prerequisite for all subsequent phases

### 3.4 Assessment

> Transitioning to Bridged Adapter resolved all routing isolation limitations and established full bidirectional reachability between `192.168.1.110` (Windows) and `192.168.1.150` (Kali SIEM). This was the foundational requirement for the entire pipeline.



