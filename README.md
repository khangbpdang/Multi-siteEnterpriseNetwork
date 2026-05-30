# Multi-Site Enterprise Network Design
Project Overview
This project simulates a multi-site enterprise network built in Cisco Packet Tracer. It features a hierarchical 3-tier architecture (core, distribution and access) and utilizes many standard network design principles including VLAN segmentation, dynamic routing, firewall ACL and NAT/PAT configuration.

### Cisco Packet Tracer | Hierarchical Network Architecture | OSPF | VLANs | ASA Firewall

---

## Skills Demonstrated

- Hierarchical three-tier network design (Core / Distribution / Access)
- VLAN design and 802.1Q trunking
- Inter-VLAN routing using Layer 3 distribution switches (Cisco 3560)
- OSPF dynamic routing with router IDs and passive interfaces
- NAT/PAT for internet access
- Cisco ASA 5506-X firewall configuration with security zones
- ACL policy for guest and IoT traffic isolation
- DMZ architecture for public-facing services
- WAN site-to-site connectivity over serial links
- Network troubleshooting methodology
- Full IP addressing scheme design with zero subnet overlap
- GitHub documentation and configuration management

---

## Network Topology

```
Internet
    │
  ISP Router (203.0.113.1)
    │
  ASA 5506-X Firewall
  ├── Outside  G1/1  203.0.113.2/30
  ├── Inside   G1/2  10.0.0.9/30
  └── DMZ      G1/3  172.16.1.1/24  →  Web Server (172.16.1.10)
    │
  Core Router (Cisco 2911)
  ├── G0/0  10.0.0.1/30  →  Dist-A
  ├── G0/1  10.0.0.5/30  →  Dist-B
  └── S0/0/0  172.16.0.1/30  →  Branch Router
    │
  ┌─────────────────────────┐
  │                         │
Dist-A (3560)           Dist-B (3560)
G0/1 10.0.0.2/30        G0/1 10.0.0.6/30
VLAN SVIs .10.1–.40.1   VLAN SVIs .11.1–.41.1
  │                         │
SW1 (2960)   SW2 (2960)   SW3 (2960)   SW4 (2960)
VLAN 10,20   VLAN 30,40   VLAN 10,20   VLAN 30,40
```

---

## Devices Used

| Device | Model | Quantity | Role |
|---|---|---|---|
| Firewall | Cisco ASA 5506-X | 1 | Perimeter security, NAT, DMZ |
| Core Router | Cisco 2911 | 1 | Inter-site routing, WAN, OSPF |
| Distribution Switch | Cisco 3560 (Layer 3) | 2 | Inter-VLAN routing, OSPF |
| Access Switch | Cisco 2960 (Layer 2) | 4 | End device connectivity |
| End devices | PCs, phones, servers | - | Simulated users and services |

---

## IP Addressing Scheme

### Infrastructure Links

| Link | Subnet | Device A | IP A | Device B | IP B |
|---|---|---|---|---|---|
| ISP → Firewall | 203.0.113.0/30 | ISP | 203.0.113.1 | FW G1/1 | 203.0.113.2 |
| Firewall → Core | 10.0.0.8/30 | FW G1/2 | 10.0.0.9 | Core G0/2 | 10.0.0.10 |
| Core → Dist-A | 10.0.0.0/30 | Core G0/0 | 10.0.0.1 | Dist-A G0/1 | 10.0.0.2 |
| Core → Dist-B | 10.0.0.4/30 | Core G0/1 | 10.0.0.5 | Dist-B G0/1 | 10.0.0.6 |

### VLAN Subnets — HQ

| VLAN | Name | Dist-A Subnet | Dist-A Gateway | Dist-B Subnet | Dist-B Gateway |
|---|---|---|---|---|---|
| 10 | Staff | 10.0.10.0/24 | 10.0.10.1 | 10.0.11.0/24 | 10.0.11.1 |
| 20 | VoIP | 10.0.20.0/24 | 10.0.20.1 | 10.0.21.0/24 | 10.0.21.1 |
| 30 | Servers | 10.0.30.0/24 | 10.0.30.1 | 10.0.31.0/24 | 10.0.31.1 |
| 40 | Wireless | 10.0.40.0/24 | 10.0.40.1 | 10.0.41.0/24 | 10.0.41.1 |

---

## VLAN Register

| VLAN | Name | Purpose |
|---|---|---|---|---|
| 10 | Staff | Employee workstations |
| 20 | VoIP | IP phones |
| 30 | Servers | File, DNS, DHCP servers 
| 40 | Wireless AP mgmt | Access point management |

---

## Routing Design

**Protocol:** OSPF (Open Shortest Path First)  
**Area:** All devices in Area 0 (backbone)

| Device | Router ID | Networks Advertised |
|---|---|---|
| Core Router | 1.1.1.1 | 10.0.0.0/30, 10.0.0.4/30, 172.16.0.0/30 |
| Dist-A | 2.2.2.2 | 10.0.0.0/30, 10.0.10–60.0/24 |
| Dist-B | 3.3.3.3 | 10.0.0.4/30, 10.0.11–61.0/24 |

OSPF passive interfaces configured on all VLAN SVIs to prevent hello packets reaching end devices.

Core Router uses `default-information originate` to distribute the default route to all OSPF neighbors.

---

## Security Policy

### Firewall Zones (ASA 5506-X)

| Zone | Interface | Security Level | Policy |
|---|---|---|---|
| Outside | G1/1 | 0 | Block all inbound except ICMP replies|
| Inside | G1/2 | 100 | Allow outbound, block Guest/IoT from internal (to implement Guest/IoT VLAN at a later time) |

### ACL Policy Summary

| Traffic | Enforcement Point | Result |
|---|---|---|
| Internet → internal | Firewall OUTSIDE-IN | Blocked |
| Staff/VoIP/Server/Wireless → anywhere | Firewall INSIDE-POLICY | Allowed |

---

## Repository Structure

```
enterprise-network-project/
├── README.md
├── topology.png
├── project.pkt
├── configs/
│   ├── firewall-asa5506.txt
│   ├── core-router.txt
│   ├── dist-a.txt
│   ├── dist-b.txt
│   ├── sw1.txt
│   ├── sw2.txt
│   ├── sw3.txt
│   └── sw4.txt
```

---

## Test Results

### Inter-VLAN Routing (HQ)

| Source | Destination | Result |
|---|---|---|
| PC SW1 (10.0.10.2) | Server SW2 (10.0.30.2) | ✅ Success |
| PC SW1 (10.0.10.2) | Server SW4 (10.0.31.2) | ✅ Success |
| PC SW3 (10.0.11.2) | Server SW4 (10.0.31.2) | ✅ Success |

### Firewall

| Test | Result |
|---|---|
| Internal PC → internet | ✅ NAT translation confirmed |
| Internet → internal host | ❌ Blocked by implicit deny |

### OSPF Verification
```
Core-Router# show ip ospf neighbor

Neighbor ID   State   Address    Interface
2.2.2.2       FULL    10.0.0.2   G0/0
3.3.3.3       FULL    10.0.0.6   G0/1
```
---

## Key Troubleshooting Lessons

**Duplicate SVI IPs across distribution switches**  
Configuring the same VLAN subnet on both Dist-A and Dist-B caused OSPF to advertise duplicate routes. The Core Router load balanced between them, producing an alternating reply/timeout pattern. Fixed by assigning unique subnets per distribution switch (x.168.X0.0 for Dist-A, x.168.X1.0 for Dist-B).


**OSPF passive interface on VLAN SVIs**  
Missing passive-interface on VLAN SVIs caused OSPF hello packets to flood toward end devices. Added `passive-interface vlan X` on all SVI interfaces across all distribution switches.

**NAT one-way traffic**  
Traffic left the network via NAT but replies were dropped because the ASA had no static routes for internal subnets. Added three static routes on the ASA pointing all internal ranges toward the Core Router next hop.

---

## Tools and Technologies

![Cisco Packet Tracer](https://img.shields.io/badge/Cisco-Packet%20Tracer-blue)
![OSPF](https://img.shields.io/badge/Routing-OSPF-green)
![ASA](https://img.shields.io/badge/Firewall-ASA%205506--X-red)
![VLANs](https://img.shields.io/badge/Switching-VLANs%20%26%20Trunking-orange)

| Technology | Implementation |
|---|---|
| Simulation tool | Cisco Packet Tracer |
| Routing protocol | OSPF Area 0 |
| Switching | 802.1Q VLANs, STP PortFast |
| Firewall | Cisco ASA 5506-X stateful inspection |
| NAT | PAT (overload) via ASA object NAT |
| WAN | Serial DCE/DTE link, static routing |
| Wireless | Multi-SSID per AP, VLAN-mapped |
| Management | SSH on all devices |

---

## Related Certifications

This project covers practical skills tested in:

- **CompTIA Network+** — VLANs, routing, subnetting, WAN
- **Cisco CCNA (200-301)** — OSPF, ACLs, NAT, switching, security
- **CompTIA Security+** — Firewall zones, DMZ, network segmentation

---

## Author

Khang Dang  
Aspiring Network Administrator  
www.linkedin.com/in/khang-dang-216601270 | khangdpdang@gmail.com

---
