# Phase 1.2 – Network Visibility & Traffic Analysis

## Objective
Validate internal network communication and service behavior by observing
ICMP, TCP, and HTTP traffic at the packet level within an isolated lab
environment.

---

## Environment Summary

| Component | Details |
|---------|--------|
| Attacker | Kali Linux (10.10.10.101) |
| Target | Linux VM (10.10.10.102) |
| Network | Internal / Host-Only |
| Capture Interface | eth1 |
| Tools | Wireshark, curl, Python HTTP server |

---

## ICMP Reachability Validation

ICMP Echo Requests were sent from the attacker to the target to confirm
basic network connectivity.

**Observations**
- ICMP Echo Request and Echo Reply packets observed
- No packet loss
- TTL value of 64 indicating Linux-based hosts
- Local Layer-2 communication confirmed via MAC addresses

**Assessment**  
The target host is reachable on the internal network, establishing a
baseline for further transport-layer testing.

---

## TCP Connectivity Analysis (Service Absent)

A TCP connection attempt was made to port 80 while no service was running.

**Observations**
- TCP SYN packets transmitted by the attacker
- TCP RST + ACK responses returned by the target

**Assessment**  
The port was closed at the operating system level. Traffic was not filtered
by a firewall, and the host actively rejected the connection.

---

## TCP Session Establishment (Service Enabled)

An HTTP service was enabled on the target system and the connection attempt
was repeated.

**Observations**
- Complete TCP three-way handshake observed (SYN → SYN-ACK → ACK)
- TCP session transitioned to ESTABLISHED state

**Assessment**  
The successful handshake confirms the service was actively listening and
stateful TCP connections were permitted.

---

## Application-Layer (HTTP) Traffic Analysis

Following session establishment, HTTP traffic was observed.

**Observations**
- HTTP GET requests and 200 OK responses
- Directory listing enabled
- No authentication or access control

**Security Implication**  
The service exposed internal directory contents, representing an
information disclosure risk due to insecure default configuration.

---

## Supporting Documentation

📄 **Detailed traffic analysis report (PDF):**  
[Internal Network Traffic Analysis Report][ICMP and TCP Network Package Analysis.pdf](https://github.com/user-attachments/files/24525387/ICMP.and.TCP.Network.Package.Analysis.pdf)

## Supporting Artifacts
<img width="1914" height="898" alt="TCP (Not Connect)" src="https://github.com/user-attachments/assets/d1c684f3-aca3-49f7-8842-6530df572ae4" />
<img width="1919" height="902" alt="ICMPs in Ping situations" src="https://github.com/user-attachments/assets/948cc9e8-4c23-44a6-a21b-911705a34c2a" />
<img width="1917" height="943" alt="Handshake TCP" src="https://github.com/user-attachments/assets/256ebf68-2a7d-41a2-a6dc-e2806779cb71" />
<img width="1919" height="931" alt="Access while TCP handshake_http GET request in victim" src="https://github.com/user-attachments/assets/1b2ef669-0ebc-4936-8335-ab1ec9769769" />

