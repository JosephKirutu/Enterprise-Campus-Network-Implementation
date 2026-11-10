# Enterprise Campus Network Design & Implementation
### Comprehensive CCNA Infrastructure & Security Capstone

A complete, dual-site enterprise campus network deployed and validated in Cisco Packet Tracer. The infrastructure follows Cisco’s hierarchical three-tier model (Core, Distribution, Access) and incorporates end-to-end routing redundancy, Layer-2 defense-in-depth, centralized wireless administration, and automated services.

---

## 📸 Network Architecture & Verification

### Topology Overview
![Network Topology](Topology.png)

### Automated Assessment Audit: 1,747 / 1,753 (99.66%)
![Verification Audit](Score.png)

> **Engineering Note on Audit Score:**  
> The lab scored 1,747 out of 1,753 test cases (99.66%). The remaining 6 unscored items relate to an automated grading rule requiring a 4096-bit RSA key on the Catalyst 3560/3650 distribution switches. In Cisco IOS on this specific hardware platform, the maximum supported modulus is strictly 2048 bits (`<360-2048>`). All functional routing, switching, redundancy, wireless, and security objectives were fully achieved.

---

## 🏛️ Network Design Highlights

### 1. Core & Distribution Layer (High Availability & Routing)
- **Hierarchical Design:** Decoupled Core and Distribution layers with dual-homed links for fault tolerance.
- **Routing Engine:** Multi-area **OSPFv2** linking remote campus buildings, core switches, and edge routing.
- **First-Hop Redundancy:** **HSRPv2** deployed across distribution switches to ensure seamless default gateway failover with active preemption.
- **Link Aggregation:** Routed (Layer-3) EtherChannels between core switches using Cisco PAgP for high-throughput inter-switch transport.
- **Dual-Stack WAN:** Dual-ISP connectivity with IPv4 DHCP negotiation and IPv6 floating static route failover.

### 2. Access Layer & Defense-in-Depth
- **VLAN Segmentation:** Clean segmentation across Data, Voice, Wireless, and Management subnets.
- **Spanning Tree Architecture:** Rapid PVST+ configured with deterministic root bridge placement, `portfast`, and `bpduguard` enabled on edge interfaces.
- **Layer 2 Security:**
  - Dynamic ARP Inspection (**DAI**) with combined source MAC, destination MAC, and IP validation.
  - **DHCP Snooping** enabled with trusted uplinks to prevent rogue DHCP server attacks.
  - Hardened trunking by explicitly disabling Dynamic Trunking Protocol (`switchport nonegotiate`).

### 3. Wireless LAN Architecture
- **Centralized Management:** Cisco 3504 Wireless LAN Controller (WLC) managing lightweight access points (LWAPs) over CAPWAP tunnels.
- **Interface Mapping:** Segregated dynamic wireless interfaces on dedicated VLANs with WPA2-PSK (AES) client authentication.

### 4. Administrative Security & Operations
- **Next-Gen Password Security:** Hardened user accounts and enable secrets using Cisco Type 9 (`scrypt`) hash algorithms.
- **Secure Management:** SSHv2-only remote administration with customized crypto keys and inactivity timeouts.
- **Core Services:** Redundant DHCP relay configurations (`ip helper-address`), local DNS record hosting, and stateful NAT operations.

---

## 🛠️ Key Troubleshooting Case Studies

During deployment and testing, several real-world edge cases were diagnosed and resolved:

* **IOS XE Cryptographic Syntax:** On Catalyst 3650 series switches, standard `enable secret` commands default to Type 5 hashes. Enabling Type 9 protection required using the `algorithm-type scrypt` keyword during configuration.
* **WLC Dynamic Interface Addressing:** Resolved a subnet conflict where the initial wireless dynamic interface shared an IP address with the distribution switch’s SVI, correcting the host assignment to `.4` to restore non-overlapping gateway operations.
* **Hardware Modulus Boundaries:** Identified and documented an architectural difference between Catalyst 2960 access switches (which permit 4096-bit RSA keys) and Catalyst 3560/3650 switches (which enforce a hard hardware ceiling of 2048 bits).

---

## 📁 Repository Contents

| File | Description |
| :--- | :--- |
| **`CCNA_Mega_Lab_Completed.pka`** | Complete, working Packet Tracer deployment file |
| **`R1.cfg`** | Core edge router configuration (OSPF, NAT, IPv6, DHCP) |
| **`CSW1.cfg`** | Core Layer-3 switch configuration (L3 EtherChannels, OSPF) |
| **`DSW1.cfg`** | Distribution switch configuration (HSRP, SVIs, Scrypt hashing) |
| **`ASW1.cfg`** | Access switch configuration (VLANs, STP, DAI, PortFast) |

---
<sub>*Framework adapted from the CCNA training scenario by Jeremy's IT Lab; design, deployment, configuration, and troubleshooting completed independently.*</sub>
