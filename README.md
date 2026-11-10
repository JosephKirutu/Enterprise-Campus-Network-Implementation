# Enterprise Multi-Site Campus Network Implementation
### End-to-End Design, Routing Architecture & Security Hardening

A fully redundant, hierarchical campus network designed and deployed in Cisco Packet Tracer. The topology models a multi-site enterprise infrastructure featuring a 3-tier core/distribution/access architecture, resilient routing failover, comprehensive Layer-2 edge defense, centralized wireless LAN administration, and secure administrative services.

---

## 🗺️ Network Architecture & Audit

### Campus Topology Overview
![Network Topology](Topology.png)

### Automated Test Audit: 1,747 / 1,753 Passed (99.66%)
![Verification Audit](Score.png)

> **Engineering Note on Audit Metric:**  
> The deployment passed 1,747 out of 1,753 test parameters (99.66%). The remaining 6 unscored items relate to an automated grading rule requiring a 4096-bit RSA modulus on the Catalyst 3560/3650 distribution switches. On this hardware family in Cisco IOS, key generation is strictly hardware-capped at 2048 bits (`<360-2048>`). All functional routing, switching, security, and service criteria were validated 100% operational.

---

## 🏗️ Technical Implementation Details

### 1. Core & Distribution Infrastructure (High Availability)
* **Layer-3 EtherChannel Core:** Configured routed Port-Channels between Core switches using Cisco PAgP (`channel-group 1 mode desirable`) on `/30` point-to-point transit subnets (`10.0.0.40/30`), eliminating STP overhead in the core.
* **First-Hop Redundancy (HSRPv2):** Deployed HSRP version 2 across distribution switches with deterministic load distribution:
  * **VLAN 10 (Office A Data):** Active on DSW-A1 (`Priority 105`, `preempt`, VIP `10.1.0.1`, Group 2).
  * **VLAN 99 (Management):** Active on DSW-A1 (`Priority 105`, `preempt`, VIP `10.0.0.1`, Group 1).
  * **VLAN 20 (Voice) & VLAN 40 (Wi-Fi):** Standby on DSW-A1 (`Priority 100`, VIPs `10.2.0.1` and `10.6.0.1`).
  * Custom virtual MAC addresses (`000c.cfe7.27xx`) assigned across all SVIs for granular traffic identification.
* **Rapid PVST+ Tuning:** DSW-A1 designated as Primary Root Bridge (`priority 0`) for VLANs 10 & 99, and Secondary Root (`priority 4096`) for VLANs 20 & 40.

### 2. Interior Routing (Multi-Area OSPFv2 & Dual-Stack Edge)
* **OSPFv2 Routing Architecture:**
  * Area 0 backbone spanning all core, distribution, and edge routing interfaces.
  * Point-to-point network types (`ip ospf network point-to-point`) configured across all routed transit interfaces (`/30` subnets) to accelerate adjacency convergence and suppress DR/BDR elections.
  * Passive-interface tuning deployed across all client-facing SVIs (VLANs 10, 20, 40) and Loopbacks to prevent unnecessary OSPF HELLO multicasts into user subnets.
  * Default route propagation using `default-information originate` on edge router R1.
* **Dual-Homed WAN Resiliency (IPv4 & IPv6):**
  * Primary Internet link via `GigabitEthernet0/0/0` (DHCP-acquired IPv4, static IPv6 `2001:DB8:A::2/64`).
  * Floating static route backup configured across secondary ISP on `GigabitEthernet0/1/0` with Administrative Distance 2 (`ip route 0.0.0.0 0.0.0.0 203.0.113.5 2` and `ipv6 route ::/0 GigabitEthernet0/1/0 2001:DB8:B::1 2`).

### 3. Layer-2 Edge Defense & Access Hardening
* **Dynamic ARP Inspection (DAI):** Enforced across VLANs 10, 20, 40, and 99 with full header sanity validation: `ip arp inspection validate src-mac dst-mac ip`. Uplink trunks to distribution switches configured as trusted (`ip arp inspection trust`).
* **DHCP Snooping:** Configured globally and per-VLAN with Option 82 insertion suppressed (`no ip dhcp snooping information option`) to maintain compatibility with Cisco switches. Access edge ports rate-limited to 15 pps.
* **Port Security & STP Guarding:**
  * End-user access interfaces configured with sticky MAC learning (`switchport port-security mac-address sticky`) and `restrict` violation modes.
  * STP edge ports hardened with `spanning-tree portfast` and `spanning-tree bpduguard enable` to prevent rogue bridge injection.
* **VLAN Trunk Hardening:** Native VLAN explicitly changed to an unused ID (`VLAN 1000`) across all inter-switch trunks with DTP statically disabled (`switchport nonegotiate`). Unused ports (FastEthernet0/3–24) administratively shutdown.

### 4. Traffic Engineering & Administrative Security
* **Inter-Branch Traffic Filtering:** Deployed extended ACL `OfficeA_to_OfficeB` on DSW-A1’s data SVI to permit administrative diagnostics (ICMP echo) between Office A (`10.1.0.0/24`) and Office B (`10.3.0.0/24`) while explicitly denying general inter-office IP traffic.
* **Management Plane Hardening:**
  * Remote VTY access locked to SSHv2 only and restricted via Access Class 1 strictly to the administrative management network (`10.1.0.0/24`).
  * Inactivity exec-timeouts set to 30 minutes with synchronous logging across console and VTY lines.
  * Passwords hashed using modern Cisco Type 9 (`scrypt`) encryption on Catalyst 3650 switches.

### 5. Infrastructure Network Services
* **Centralized DHCP Architecture:** Router R1 hosts 7 segmented pools (`A-Mgmt`, `A-PC`, `A-Phone`, `B-Mgmt`, `B-PC`, `B-Phone`, `Wi-Fi`) with excluded gateway ranges. Distribution SVIs operate as DHCP Relays via `ip helper-address 10.0.0.76`.
* **Centralized Wireless (WLC & CAPWAP):** DHCP Option 43 configured pointing to Cisco 3504 WLC (`10.0.0.7`). LWAP access points associate over CAPWAP with dynamic VLAN 40 mapping and WPA2-AES encryption.
* **Enterprise NAT:** Dynamic NAT overload (PAT) configured via IP pool `POOL1` (`203.0.113.200/29`), paired with a one-to-one static NAT translation exposing internal DMZ server `10.5.0.4` to public address `203.0.113.113`.
* **Telemetry & Time Synchronization:**
  * Centralized NTP authentication configured using MD5 Key 1; R1 serves as authoritative clock (`ntp master 5`) while all switches synchronize as authenticated clients.
  * Centralized logging directed to remote Syslog host `10.5.0.4` at severity level `debugging`.

---

## 🔬 Engineering & Troubleshooting Case Studies

1. **Cisco IOS XE Scrypt Type 9 Implementation:**  
   Standard `enable secret` commands default to legacy Type 5 MD5 hashes. On the Catalyst 3650 distribution switches, security standards required Type 9 hashing. Configuring this required invoking the `algorithm-type scrypt` keyword during credential initialization (`enable algorithm-type scrypt secret <password>`).

2. **WLC Dynamic Subnet Collision:**  
   During wireless integration, the dynamic interface for the Wi-Fi client network required alignment with the existing HSRP gateway infrastructure. Corrected an overlapping host IP assignment to place the controller interface on host `.4` (`10.6.0.4/24`), maintaining DSW-A1 SVI on `.2` and the virtual gateway on `.1`.

3. **Platform Asymmetric Key Generation Limits:**  
   Documented a platform discrepancy between access and distribution layers. While the 2960 access series supports 4096-bit RSA keys, Catalyst 3560/3650 images enforce a strict upper limit of 2048 bits for general-purpose keys (`<360-2048>`), requiring exact modulus sizing matching device capabilities.

---

## 📁 Repository Contents

| File | Device / Role | Key Technologies Configured |
| :--- | :--- | :--- |
| **`CCNA_Mega_Lab_Completed.pka`** | Master Topology | Full Packet Tracer simulation environment |
| **`R1.cfg`** | Core Edge Router | DHCP Server Pools, NAT Overload & Static, OSPFv2, Floating Static Routes, NTP Master |
| **`CSW1.cfg`** | Core Switch 1 | Routed Port-Channel (PAgP), OSPFv2 Transit Links, Type 9 Scrypt Authentication |
| **`DSW1.cfg`** | Distribution Switch A1 | HSRPv2 Gateway Redundancy, Rapid-PVST+ Root, Inter-VLAN ACLs, DHCP Relay |
| **`ASW1.cfg`** | Access Switch A1 | Dynamic ARP Inspection, DHCP Snooping, Port Security, PortFast/BPDU Guard, VTP |

---
<sub>*Architecture and scenario based on enterprise campus training materials; design, CLI configurations, and troubleshooting completed independently.*</sub>
