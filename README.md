# 🛍️ Boutique Retail Store & POS Infrastructure Design

A production-ready retail branch network architecture designed and verified in **Cisco Packet Tracer**. This implementation emphasizes secure payment handling aligned with **PCI-DSS** principles, inter-VLAN routing using 802.1Q sub-interfaces, perimeter NAT/PAT translation, and strict Layer 3/4 traffic isolation policies using Extended Access Control Lists (ACLs).

---

## 📋 Table of Contents
- [Executive Overview](#-executive-overview)
- [Business Requirements & Security Policies](#-business-requirements--security-policies)
- [Network Architecture & Topology](#-network-architecture--topology)
- [Addressing & VLAN Schema](#-addressing--vlan-schema)
- [Key Technical Implementations](#-key-technical-implementations)
- [Verification & Security Audit](#-verification--security-audit)
- [Repository Structure](#-repository-structure)


---

## 🏢 Executive Overview

In retail environments, cardholder data environments (CDE) and physical surveillance infrastructure require strict isolation from standard management endpoints and public WAN networks.

This project delivers a multi-tier switched and routed design for a boutique retail storefront:
* Isolates the **Sales & POS** counter to maintain transactional integrity.
* Restricts **CCTV surveillance streams** strictly to internal monitoring, eliminating exposure to public internet leaks.
* Provides **Store Management** full operational visibility across inventory, sales, and security systems.
* Implements dynamic **Port Address Translation (PAT)** to allow authorized internal hosts to communicate with the bank payment processing host over a simulated ISP uplink.

---

## 🎯 Business Requirements & Security Policies

| Requirement | Policy Rule | Enforcement Mechanism |
| :--- | :--- | :--- |
| **Payment & Billing** | Cashier POS must reach the external Bank Gateway (`198.51.100.2`) and internal Inventory Server (`192.168.120.x`). | Router-on-a-Stick Routing + Outbound PAT |
| **Surveillance Lockdown** | CCTV cameras must **never** send or receive traffic across the WAN (zero outbound leak). | Extended Ingress ACL (`CCTV_TRAFFIC_POLICY`) |
| **Surveillance Access** | Only Store Management hosts can pull video streams from CCTV endpoints. | Extended ACL Filter Rule |
| **Host Provisioning** | Automated IP assignment for workstations, POS systems, and IP cameras with localized gateways. | Cisco IOS DHCP Pools per VLAN |

---

## 🗺️ Network Architecture & Topology

![Boutique Retail Store POS Topology](https://raw.githubusercontent.com/fiaz443/-Boutique-Retail-Store-POS-Infrastructure-Design/main/file_000000006fe0820a9c7e021b8629c35e.png)

****🌐 Addressing & VLAN Schema****


| Segment Name | VLAN ID | Subnet / CIDR | Default Gateway | Switch Ports | Internet / WAN |
| :--- | :---: | :--- | :--- | :--- | :--- |
| **Sales & Billing** | 110 | `192.168.110.0/24` | `192.168.110.1` | `Fa0/1` | Permitted (PAT) |
| **Store Management** | 120 | `192.168.120.0/24` | `192.168.120.1` | `Fa0/2 - Fa0/3` | Permitted (PAT) |
| **CCTV Surveillance** | 130 | `192.168.130.0/24` | `192.168.130.1` | `Fa0/4 - Fa0/5` | Blocked (Isolated) |
| **ISP / WAN Link** | — | `198.51.100.0/30` | `198.51.100.1` | `Gi0/1` | Public Uplink |


****⚙️ Key Technical Implementations****

**1. VLAN Segmentation & Layer 2 Access Controls (Store-SW1)**

Segregated physical access ports into distinct broadcast domains (VLAN 110, VLAN 120, VLAN 130).

Configured GigabitEthernet0/1 as an 802.1Q trunk port carrying encapsulated multi-VLAN frames to the router.

**2. Router-on-a-Stick Inter-VLAN Routing (Store-Gateway)**

Provisioned logical sub-interfaces on GigabitEthernet0/0:

Gi0/0.110 (Encapsulation dot1Q 110)

Gi0/0.120 (Encapsulation dot1Q 120)

Gi0/0.130 (Encapsulation dot1Q 130)

Configured dedicated DHCP server scopes for each VLAN with gateway and DNS parameters.

**3. Extended Access Control Lists (Surveillance Isolation)**

Applied ingress extended ACL on sub-interface Gi0/0.130:

Permits camera traffic destined exclusively to the Store Management subnet (192.168.120.0/24).

Denies all remaining outbound traffic from CCTV cameras to any destination (deny ip 192.168.130.0 0.0.0.255 any), preventing external data leakage.

**4. Edge NAT/PAT (Port Address Translation)**

Applied ip nat inside on sub-interfaces 0/0.110 and 0/0.120.

Applied ip nat outside on the WAN interface GigabitEthernet0/1.

Bound Standard ACL INTERNET_PERMITTED to WAN NAT Overload, intentionally omitting VLAN 130 so camera traffic is untranslatable even if routing attempts occur.

### 🧪 Verification & Security Audit

| Test Action | Source Device | Target Destination | Expected Behavior | Verification Status |
| :--- | :---: | :---: | :--- | :---: |
| **Card Processing** | Cashier-POS | `198.51.100.2` (Bank Gateway) | Ping & TCP authorization succeed via PAT | ✅ PASSED |
| **Stock Sync** | Cashier-POS | `192.168.120.x` (Inventory) | Seamless Inter-VLAN communication | ✅ PASSED |
| **Camera Monitoring** | Manager-PC | `192.168.130.x` (CCTV) | Stream viewed successfully | ✅ PASSED |
| **Surveillance WAN Leak** | CCTV-Front | `198.51.100.2` (WAN) | Destination Unreachable (Dropped by ACL) | 🛡️ BLOCKED |
| **POS Network Tampering** | CCTV-Billing | `192.168.110.x` (POS) | Access denied by extended traffic policy | 🛡️ BLOCKED |

****Diagnostic CLI Commands****

! Check dynamic NAT address translations

Store-Gateway# show ip nat translations

! Inspect hit counters for the CCTV security ACL

Store-Gateway# show access-lists CCTV_TRAFFIC_POLICY

! Confirm 802.1Q sub-interface operation and status

Store-Gateway# show ip interface brief

****📂 Repository Structure****
.
├── configs/
│   ├── Store-SW1.cfg              # Full Cisco IOS configuration for Access Switch
│   └── Store-Gateway.cfg          # Full Cisco IOS configuration for Edge Router
├── docs/
│   └── topology-diagram.png       # Network topology diagram screenshot
├── packet-tracer/
│   └── Boutique_Retail_Store.pkt  # Cisco Packet Tracer lab file
└── README.md                      # Comprehensive project documentation
