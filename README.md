# Enterprise Network Design & Implementation - Three-Tier Architecture

A full enterprise network simulation built in **Cisco Packet Tracer**, connecting a **Headquarters** site and a **Branch Office** over a simulated ISP/Internet cloud, using a hierarchical three-tier design (Core / Distribution / Access).

---

## Overview

The project models a scalable, redundant, and secure enterprise network. Both sites run segmented VLANs, redundant Layer 2/Layer 3 paths, dynamic routing, centralised network services, and a site-to-site encrypted tunnel across the public Internet.

**Design goals**

- Model a three-tier (Core, Distribution, Access) network architecture
- Configure routing protocols for efficient inter-site and Internet communication
- Implement Layer 2 segmentation and redundancy
- Establish secure branch-to-HQ communication over IPSec VPN
- Integrate monitoring, security, and redundancy services
- Provide a segregated wireless network for guest users
- Document risk management, compliance, and legal considerations

---

## Architecture

| Layer | Role | Devices |
|---|---|---|
| **Access** | End-device connectivity, port-level security | Layer 2 switches (2960) |
| **Distribution** | Inter-VLAN routing, policy enforcement, redundancy | Multilayer switches (3650) |
| **Core** | High-speed backbone, load balancing | Multilayer switches / routers (2911) |

Both sites are dual-homed at the distribution and core layers so that no single link or device failure isolates a department.

---

## IP Addressing Scheme

### Headquarters - VLANs

| Department | VLAN | Network | Prefix | Hosts |
|---|---|---|---|---|
| Sales | 10 | 192.168.0.0 | /24 | 250 |
| HR | 20 | 192.168.1.0 | /25 | 120 |
| Finance | 30 | 192.168.1.128 | /25 | 120 |
| Admin | 40 | 192.168.2.0 | /28 | 60 |
| Guest | 50 | 192.168.2.192 | /27 | 25 |
| WLAN | 60 | 10.10.0.0 | /16 | 25 |
| Management | 70 | 192.168.10.0 | /24 | 250 |

### Branch Office - VLANs

| Department | VLAN | Network | Prefix | Hosts |
|---|---|---|---|---|
| IT | 10 | 192.168.12.0 | /24 | 250 |
| Finance | 20 | 192.168.13.0 | /25 | 120 |
| Sales | 30 | 192.168.13.128 | /25 | 120 |
| Guest | 40 | 192.168.14.0 | /28 | 60 |
| WLAN | 60 | 30.30.0.0 | /16 | 3000 |
| Management | 70 | 192.168.20.0 | /24 | 250 |

### Server Farms

| Site | Subnet | Services |
|---|---|---|
| Headquarters (DMZ) | 10.20.20.0/27 | DHCP `.5`, DNS `.6`, E-mail `.7`, NTP `.8` |
| Branch | 20.20.20.0/27 | DHCP `.2`, DNS `.3`, SYSLOG `.4`, Backup DHCP `.5`, AAA |

### Infrastructure Links

- **Core ↔ Distribution ↔ Edge:** `10.0.1.0/24` carved into `/30` point-to-point links
- **Internet / ISP backbone:** `203.0.113.0/24` carved into `/30` links

---

## Layer 2 Configuration

| Feature | Implementation |
|---|---|
| **VLANs** | 7 VLANs per site - 5 departmental, 1 wireless, 1 management |
| **RPVST+** | Per-VLAN rapid spanning tree for fast convergence and load balancing across multilayer switches |
| **EtherChannel** | Static Layer 2 bundles between switches for bandwidth aggregation and link redundancy |
| **VTP** | One distribution multilayer switch in server mode (domain `aayush`); all other switches as clients |
| **PortFast + BPDU Guard** | Enabled on all access ports facing end devices |
| **Trunk / Access ports** | 802.1Q trunks between access and distribution; access ports for hosts and APs |
| **HSRP** | Gateway redundancy - active/standby roles aligned with STP root priority per VLAN |

---

## Layer 3 Configuration

| Protocol / Feature | Where it is used |
|---|---|
| **OSPF (area 0)** | Between core routers, distribution multilayer switches, and edge routers at both sites |
| **EIGRP (AS 100)** | Inside the simulated ISP / Internet cloud |
| **BGP (AS 65000 ↔ 65001)** | Between edge routers and ISP routers; internal networks and EIGRP routes redistributed |
| **RIP** | Demonstrated on a small router segment |
| **Default static route** | On edge routers toward the ISP as a fallback path |
| **Inter-VLAN routing** | SVIs for all 7 VLANs on the distribution multilayer switches |
| **Layer 3 EtherChannel** | Two routed bundles at HQ with source/destination-IP load balancing |
| **PAT** | Overload NAT on the edge router for outbound Internet access |
| **SSH** | Encrypted device management, restricted to management VLAN by ACL |

---

## Network Services

- **DNS** - internal zone `aayush.com` resolving to the HQ web server
- **DHCP** - per-VLAN pools at both sites, with `ip helper-address` relay since the servers live in a separate subnet
- **SNMP** - read-only community configured for monitoring
- **SYSLOG** - centralised logging server in the branch server farm
- **AAA / RADIUS** - centralised authentication, authorization, and accounting for device login
- **NTP** - authenticated time source (UTC+05:45) with key and password

---

## Security Measures

| Control | Details |
|---|---|
| **ACLs** | Standard ACLs restrict SSH to the management VLAN; extended ACLs permit HTTP/HTTPS/FTP/DNS/ICMP/SSH per subnet |
| **Port Security** | Sticky MAC learning with `violation restrict` on access ports |
| **DHCP Snooping** | Enabled on VLANs 10–70; rate limits of 500 pps (WLAN) and 15 pps (other VLANs); uplinks marked trusted |
| **Firewall** | ACL-based firewalling on the edge routers in place of a dedicated appliance |
| **IPSec VPN** | Site-to-site tunnel between HQ and branch edge routers, ISAKMP key exchange, PFS enabled |
| **Unused ports** | Administratively shut down; CDP/LLDP/DTP disabled to limit reconnaissance and VLAN hopping |
| **Password protection** | `service password-encryption`, local usernames, enable secret on all devices |

> **Note:** all credentials in this repository are lab-only values used inside Packet Tracer. Do not reuse them in any real environment, and do not commit real keys or passphrases.

---

## Wireless Network

- Access Points deployed at both sites on a dedicated **WLAN VLAN (60)**, isolating guest/wireless traffic from departmental VLANs
- **SSID:** `WLAN`, secured with **WPA2-PSK (AES)**
- APs hold static IPs; clients receive addresses from a dedicated DHCP pool
- A **Wireless LAN Controller (WLC)** centrally provisions the APs and balances client load

---

## Repository Contents

```
.
├── README.md
├── docs/
│   └── ST5064CEM_Networking_Coursework.pdf   # full report
├── packet-tracer/
│   ├── headquarters.pkt
│   ├── branch.pkt
│   └── internet.pkt
├── configs/                                  # exported running-configs per device
└── diagrams/                                 # physical & logical topology diagrams
```

*(Adjust the paths above to match what you actually upload.)*

---

## Getting Started

1. Install **Cisco Packet Tracer 8.x** or later.
2. Open the `.pkt` file from `packet-tracer/`.
3. Wait for STP, OSPF, and BGP adjacencies to converge (switch links turn green).
4. Verify with:

```
show ip interface brief
show vlan brief
show spanning-tree vlan 10
show etherchannel summary
show standby brief
show ip ospf neighbor
show ip bgp summary
show ip nat translations
show crypto map
show ip dhcp snooping
```

5. Test end-to-end reachability by pinging between HQ and branch hosts - traffic should traverse the IPSec tunnel.

---

## Report Sections

The accompanying report also covers:

- **Comparison of routing protocols** - OSPF vs EIGRP on open standard vs proprietary, convergence, scalability, and policy control
- **Security challenges and network impact** - DoS/DDoS, man-in-the-middle, missing segmentation, supply chain attacks, APTs, weak authentication
- **Risk management, compliance & legal issues** - GDPR, HIPAA, PCI DSS, ISO 27001, plus social and legal considerations
- **Emerging trends** - SDN, NFV, cloud computing, and 5G, and how each reshapes the traditional three-tier model

---
