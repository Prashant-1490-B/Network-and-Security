# ARP Spoofing Detector Documentation

## Comprehensive Technical Documentation

### Detailed Architecture
The ARP Spoofing Detector is designed to continuously monitor network traffic and detect any suspicious ARP (Address Resolution Protocol) activity. The architecture consists of several components:

1. **Packet Sniffer**: Captures data packets on the network using libraries like `pcap` or `scapy`.
2. **ARP Monitor**: Analyzes captured packets for ARP requests and replies, keeping track of MAC and IP address associations.
3. **Detection Mechanism**: Implements algorithms to detect anomalies, such as multiple IP addresses being associated with a single MAC address.
4. **Alert System**: Sends notifications when ARP spoofing is detected, through email or logging.

### Execution Flow
1. **Initialization**: The application initializes by loading necessary configurations and starting the packet sniffer.
2. **Packet Capturing**: The packet sniffer continuously captures packets and passes them to the ARP monitor.
3. **ARP Analysis**: For each packet, the ARP monitor checks if it is an ARP request or reply.
4. **Detection**: If an anomaly is detected (e.g., different MAC for the same IP), the detection mechanism triggers an alert.
5. **Logging/Alerting**: Detected anomalies are logged, and alerts are sent to the system administrator.

### Data Structures
- **ARP Table**: Stores mappings of IP addresses to MAC addresses. Updated as packets are analyzed.
  - `arp_table = { '192.168.1.1': '00:14:22:01:23:45', ... }`

- **Packet Structure**: Represents the format of captured data packets.
  - `class Packet { 
      String src_ip;
      String src_mac;
      String dest_ip;
      String dest_mac;
      ... 
    }`

### Detection Mechanisms
- **Anomaly Detection**: Uses statistical analysis and heuristics to identify abnormal patterns in ARP traffic.
- **Thresholds**: Configurable parameters that define what constitutes abnormal behavior, such as the number of changes in ARP entries within a time frame.

### Code Flow Analysis
1. The `main()` function starts the application and initializes components.
2. The packet sniffer is run in a separate thread to continuously capture packets.
3. Captured packets are sent to the ARP monitor, which updates the ARP table based on analyzed packets.
4. The detection algorithm runs periodically, checking for inconsistencies.
5. Alerts are generated, and logs are updated in case of detected ARP spoofing.

## Conclusion
This documentation covers the essential components and functionality of the ARP Spoofing Detector, providing insight into the architecture, execution flow, detection methodologies, and code flow analysis for effective understanding and further development.