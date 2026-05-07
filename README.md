<div align="center">

# 🌐 Final Network Project

### Multi-Site Enterprise Network — Cisco Packet Tracer

[![GitHub](https://img.shields.io/badge/GitHub-engYSF-1A3A5C?style=for-the-badge&logo=github)](https://github.com/engYSF)
[![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco)](https://www.netacad.com/)
[![License](https://img.shields.io/badge/License-MIT-2E86C1?style=for-the-badge)](LICENSE)
[![EIGRP](https://img.shields.io/badge/Routing-EIGRP%20%2B%20RIP-success?style=for-the-badge)]()
[![NAT](https://img.shields.io/badge/Internet-PAT%20%2F%20NAT-orange?style=for-the-badge)]()

> **Designed & built by [Yusuf](https://github.com/engYSF)**  
> Faculty of Computers & Information Technology — University of Tabuk

</div>

-----

## 📋 Table of Contents

- [Overview](#-overview)
- [Topology](#-topology)
- [Network Design](#-network-design)
- [VLAN Segmentation](#-vlan-segmentation)
- [IP Addressing](#-ip-addressing)
- [Router Configurations](#-router-configurations)
- [Services](#-services)
- [Testing & Verification](#-testing--verification)
- [Challenges & Resolutions](#-challenges--resolutions)
- [Project Structure](#-project-structure)
- [Copyright](#-copyright)

-----

## 🔭 Overview

A fully functional **two-site enterprise network** simulated in Cisco Packet Tracer, featuring:

|Feature              |Detail                                                  |
|---------------------|--------------------------------------------------------|
|**Sites**            |Main Office + Branch                                    |
|**Routing Protocols**|EIGRP AS 10 (HQ) · RIPv2 (Branch) · Route Redistribution|
|**Internet Access**  |PAT (Overload NAT) via Edge router                      |
|**VLANs**            |6 VLANs — HR, Finance, IT, Wireless, Voice, Servers     |
|**Services**         |DHCP · DNS · HTTP — centralized at `192.168.100.10`     |
|**Wireless**         |Access Point on VLAN 40                                 |
|**Voice**            |IP Phone on VLAN 110                                    |

-----

## 🗺 Topology

![Network Topology](topology.jpeg)

```
          ┌─────────────┐
          │  ISP/R-NET  │  loopback 8.8.8.8/32
          │ 203.0.113.2 │
          └──────┬──────┘
                 │ Serial (203.0.113.0/30)
          ┌──────┴──────┐
          │   R-EDGE    │  Route Redistribution + NAT/PAT
          │ 203.0.113.1 │
          └──┬──────┬───┘
    GigE     │      │ Serial (172.16.1.0/30)
  10.1.1.0/30│      │
      ┌───────┴──┐ ┌┴──────────┐
      │  R-MAIN  │ │ R-BRANCH  │
      │  EIGRP   │ │   RIPv2   │
      └────┬─────┘ └─────┬─────┘
           │              │
      ┌────┴──────┐  ┌────┴──────┐
      │ Switch0/1 │  │  Switch   │
      └───────────┘  └───────────┘
      VL10/20/30/40  VL50
      VL100/110
```

-----

## 🏗 Network Design

### Sites

**🏢 Main Office**

- Router-on-a-stick (`R-MAIN`) — inter-VLAN routing via sub-interfaces on `Gi1/0`
- Two multilayer switches (`Switch0`, `Switch1`) — trunking + access ports
- Wireless Access Point — VLAN 40 (`192.168.40.0/24`)
- IP Phone — VLAN 110 (`192.168.110.0/24`)
- Server farm — VLAN 100 (`192.168.100.0/24`) hosting DHCP/DNS/HTTP

**🏭 Branch**

- `R-BRANCH` running RIPv2
- Single switch with PC4 and PC5 on VLAN 50
- Connected to Edge via Serial link (DCE — `clock rate 64000`)

**🌐 Edge Router**

- Interconnects Main Office (EIGRP) ↔ Branch (RIP) via route redistribution
- Provides Internet access via **PAT** — `access-list 1` / `ip nat inside source list 1`
- Default route: `0.0.0.0/0` → Serial3/0 (ISP)

-----

## 🔀 VLAN Segmentation

|VLAN|Name            |Subnet            |Gateway        |
|----|----------------|------------------|---------------|
|10  |HR              |`192.168.10.0/24` |`192.168.10.1` |
|20  |Finance         |`192.168.20.0/24` |`192.168.20.1` |
|30  |IT              |`192.168.30.0/24` |`192.168.30.1` |
|40  |Wireless        |`192.168.40.0/24` |`192.168.40.1` |
|50  |Branch Data     |`192.168.50.0/24` |`192.168.50.1` |
|100 |Servers         |`192.168.100.0/24`|`192.168.100.1`|
|110 |IP Phone (Voice)|`192.168.110.0/24`|`192.168.110.1`|

-----

## 📡 IP Addressing

### WAN Links

|Link             |Type           |Subnet          |
|-----------------|---------------|----------------|
|R-MAIN ↔ R-EDGE  |GigabitEthernet|`10.1.1.0/30`   |
|R-EDGE ↔ R-BRANCH|Serial         |`172.16.1.0/30` |
|R-EDGE ↔ ISP     |Serial         |`203.0.113.0/30`|
|ISP Loopback     |Loopback0      |`8.8.8.8/32`    |

-----

## ⚙️ Router Configurations

<details>
<summary><b>🔵 R-MAIN — Router-on-a-Stick + EIGRP</b></summary>

```cisco
en
conf t
host R0-MAIN

int gi1/0.10
 encap dot1q 10
 ip add 192.168.10.1 255.255.255.0
 ip helper-address 192.168.100.10

int gi1/0.20
 encap dot1q 20
 ip add 192.168.20.1 255.255.255.0
 ip helper-address 192.168.100.10

int gi1/0.30
 encap dot1q 30
 ip add 192.168.30.1 255.255.255.0
 ip helper-address 192.168.100.10

int gi1/0.40
 encap dot1q 40
 ip add 192.168.40.1 255.255.255.0
 ip helper-address 192.168.100.10

int gi1/0.100
 encap dot1q 100
 ip add 192.168.100.1 255.255.255.0

int gi1/0.110
 encap dot1q 110
 ip add 192.168.110.1 255.255.255.0
 ip helper-address 192.168.100.10

int gi0/0
 desc To EDGE
 ip add 10.1.1.1 255.255.255.252

router eigrp 10
 net 10.1.1.0 0.0.0.3
 net 192.168.10.0 0.0.0.255
 net 192.168.20.0 0.0.0.255
 net 192.168.30.0 0.0.0.255
 net 192.168.40.0 0.0.0.255
 net 192.168.100.0 0.0.0.255
 net 192.168.110.0 0.0.0.255
 no auto-summary
end
wr mem
```

</details>

<details>
<summary><b>🔴 R-EDGE — EIGRP ↔ RIP Redistribution + PAT</b></summary>

```cisco
en
conf t
host R1-EDGE

int gi0/0
 desc To R-MAIN
 ip add 10.1.1.2 255.255.255.252
 ip nat inside

int s2/0
 desc To R-BRANCH
 ip add 172.16.1.1 255.255.255.252
 ip nat inside

int s3/0
 desc To ISP
 ip add 203.0.113.1 255.255.255.252
 ip nat outside

router eigrp 10
 net 10.1.1.0 0.0.0.3
 redistribute rip metric 10000 100 255 1 1500
 no auto-summary

router rip
 ver 2
 no auto-summary
 net 172.16.1.0
 redistribute eigrp 10 metric 2

ip route 0.0.0.0 0.0.0.0 s3/0
access-list 1 permit 192.168.0.0 0.0.255.255
ip nat inside source list 1 int s3/0 overload
end
wr mem
```

</details>

<details>
<summary><b>🟡 R-BRANCH — VLAN 50 + RIPv2</b></summary>

```cisco
en
conf t
host R2-BRANCH

int gi9/0.50
 encap dot1q 50
 ip add 192.168.50.1 255.255.255.0
 ip helper-address 192.168.100.10

int s2/0
 desc To EDGE
 ip add 172.16.1.2 255.255.255.252
 clock rate 64000

router rip
 ver 2
 no auto-summary
 net 172.16.1.0
 net 192.168.50.0
end
wr mem
```

</details>

<details>
<summary><b>🟣 R-INTERNET (ISP) — Return Route</b></summary>

```cisco
en
conf t
host R-INTERNET

int s0/0
 ip add 203.0.113.2 255.255.255.252
 no shut

int lo0
 ip add 8.8.8.8 255.255.255.255

ip route 192.168.0.0 255.255.0.0 203.0.113.1
end
wr mem
```

</details>

-----

## 🛠 Services

### DHCP Pools (hosted on `192.168.100.10`)

|Pool          |Gateway        |DNS             |Start IP        |
|--------------|---------------|----------------|----------------|
|BRANCH VL50   |`192.168.50.1` |`192.168.100.10`|`192.168.50.50` |
|IP PHONE VL110|`192.168.110.1`|`192.168.100.10`|`192.168.110.50`|
|Wireless VL40 |`192.168.40.1` |`192.168.100.10`|`192.168.40.50` |
|IT VL30       |`192.168.30.1` |`192.168.100.10`|`192.168.30.50` |
|Finance VL20  |`192.168.20.1` |`192.168.100.10`|`192.168.20.50` |
|HR VL10       |`192.168.10.1` |`192.168.100.10`|`192.168.10.50` |

### DNS

- Domain `ysf` resolves to `192.168.100.10`
- Clients use `192.168.100.10` as DNS server across all VLANs

### Web Server (HTTP)

- Accessible via `http://192.168.100.10` or `http://ysf`
- Hosts the project page: *“Final Project — Yusuf”*

-----

## ✅ Testing & Verification

### Routing Tables — Expected Results

|Router      |Key Routes Verified                                      |
|------------|---------------------------------------------------------|
|**R-MAIN**  |Connected VLANs + `D*EX 0.0.0.0/0` via `10.1.1.2`        |
|**R-EDGE**  |EIGRP HQ routes + `R 192.168.50.0` + `S* 0.0.0.0/0`      |
|**R-BRANCH**|All HQ VLANs via RIP + `R* 0.0.0.0/0` via `172.16.1.1`   |
|**ISP**     |`S 192.168.0.0/16` via `203.0.113.1` + loopback `8.8.8.8`|

### Reachability Tests

```
✅ Web by IP   — PC4 → http://192.168.100.10   → Page loads
✅ Web by DNS  — PC0 → http://ysf              → DNS resolves, page loads
✅ Traceroute  — PC4 (Branch) → 192.168.40.51  → 4 hops
✅ Internet    — All VLANs → 8.8.8.8 via PAT   → Ping success
```

### Traceroute Output (Branch → VLAN 40)

```
C:\>tracert 192.168.40.51

Tracing route to 192.168.40.51 over a maximum of 30 hops:

  1    0 ms    0 ms    0 ms   192.168.50.1   ← R-BRANCH
  2   24 ms   13 ms    0 ms   172.16.1.1     ← R-EDGE
  3    0 ms    0 ms    0 ms   10.1.1.1       ← R-MAIN
  4    *       67 ms  53 ms   192.168.40.51  ← Target

Trace complete.
```

### NAT Statistics

```
Total translations: 9 (0 static, 9 dynamic, 9 extended)
Outside Interfaces: Serial3/0
Inside  Interfaces: GigabitEthernet0/0, Serial2/0
Hits: 54   Misses: 73
```

-----

## 🚧 Challenges & Resolutions

|#|Problem                 |Root Cause                |Fix                                                      |
|-|------------------------|--------------------------|---------------------------------------------------------|
|1|Internet replies dropped|No return route on ISP    |`ip route 192.168.0.0 255.255.0.0 203.0.113.1` on ISP    |
|2|Branch couldn’t reach HQ|One-way redistribution    |Added bidirectional EIGRP ↔ RIP with explicit metrics    |
|3|DHCP not working on VLAN|Missing helper-address    |Added `ip helper-address 192.168.100.10` on sub-interface|
|4|NAT not translating     |Wrong inside/outside roles|Corrected interface NAT roles + verified ACL 1           |
|5|STP instability         |Broad trunk allowed VLANs |Restricted trunk VLANs + enabled PortFast/BPDU Guard     |

-----

## 📁 Project Structure

```
📦 Final-Network-Project/
├── 📄 README.md                        ← You are here
├── 📄 Final_Network_Report_engYSF.pdf  ← Full LaTeX report
├── 📄 network_report.tex               ← LaTeX source
├── 🖼  topology.jpeg                   ← Packet Tracer topology screenshot
└── 📦 network_project.pkt              ← Cisco Packet Tracer file
```

-----

## 📜 Copyright

```
© Yusuf  |  github.com/engYSF
All rights reserved.

This project was submitted as a final assignment for the
Faculty of Computers & Information Technology, University of Tabuk.
Unauthorized reproduction or redistribution is prohibited.
```

<div align="center">

-----

Made with ❤️ by **[Yusuf](https://github.com/engYSF)**

[![GitHub followers](https://img.shields.io/github/followers/engYSF?style=social)](https://github.com/engYSF)

</div>
