# BrightSoft Solutions — Segmented Enterprise Network Design

**Module:** ITNSA2-44 (Network Security) — Eduvos
**Tool:** Cisco Packet Tracer
**Type:** Group project (network design, configuration, and security policy)

## Overview

This project designs and configures a segmented enterprise network in Cisco Packet Tracer for **BrightSoft Solutions**, a fictional software services company in Durban with three floors: Developers/Business Analysts (Floor 1), Test Engineers/UI Designers (Floor 2), and a client-facing staging environment (Floor 3). The build covers DHCP, DNS, VLAN segmentation with inter-VLAN routing, NAT/port forwarding for a partially internet-facing staging server, extended ACLs for lateral-movement control, and dual routing protocols (RIPv2 on Floor 1, OSPF on the Shared Router/Floor 2–3) with a backup link between domains.

The full Packet Tracer file is included in this repo: [`BrightSoft-Network-Design.pkt`](./BrightSoft-Network-Design.pkt).

## Table of Contents

- [Scenario](#scenario)
- [Question 1 — Topology, DHCP, DNS](#question-1--topology-dhcp-dns)
  - [1.1 Network Topology](#11-network-topology)
  - [1.2 DHCP Server Configuration](#12-dhcp-server-configuration)
  - [1.3 DNS Server Configuration](#13-dns-server-configuration)
- [Question 2 — VLANs, Inter-VLAN Routing, ACLs](#question-2--vlans-inter-vlan-routing-acls)
  - [2.1 VLAN Segmentation](#21-vlan-segmentation)
  - [2.2 Inter-VLAN Routing](#22-inter-vlan-routing)
  - [2.3 Access Control Lists](#23-access-control-lists)
- [Question 3 — Staging Environment: NAT, ACLs, DMZ Design](#question-3--staging-environment-nat-acls-dmz-design)
  - [3.1 Packet Tracer Configuration and Evidence](#31-packet-tracer-configuration-and-evidence)
  - [3.2 Theoretical Design Discussion](#32-theoretical-design-discussion)
- [Question 4 — RIP/OSPF Resilience and Redistribution](#question-4--ripospf-resilience-and-redistribution)
  - [4.1 Routing Configuration and Backup Link](#41-routing-configuration-and-backup-link)
  - [4.2 RIP/OSPF Mixed-Environment Challenges](#42-ripospf-mixed-environment-challenges)
- [Bibliography](#bibliography)

---

## Scenario

BrightSoft Solutions is allocated the address space `192.168.0.0/24` (mapped in the build to `172.168.0.0/24` for the LAN side), structured across three floors:

**Floor 1 — Developers & Business Analysts**
- Developers (3 devices) and Business Analysts (3 devices), each with its own locally-managed DHCP server.
- Developers' router connects to: the central DNS server, the staging environment, and the internet.
- Business Analysts' router connects to: the central DNS server, the production web server, and the internet.
- Routing protocol: **RIPv2** (legacy compatibility requirement).

**Floor 2 — Test Engineers & UI Designers**
- Two teams sharing a floor, logically separated with **802.1Q VLANs** and inter-VLAN routing via a Layer 3 (multilayer) switch.
- Shared resources (printer, file server) reachable through controlled inter-VLAN rules.
- Connects to Floor 3 and the internet through a single **Shared Router** (cost-saving measure).
- Routing protocol: **OSPF**.

**Floor 3 — Staging Environment**
- A staging web server used for external partner demos and updated by the Developers team.
- Accessible from the internet on HTTP/HTTPS only, and restricted from Business Analysts, Test Engineers, and UI Designers.

**Centralised infrastructure**
- Central DNS Server `192.168.0.13` — internal name resolution for all floors plus external lookups.
- Production Web Server `192.168.0.14` — reachable by all internal departments via the Business Analysts' router.
- Syslog Server `192.168.0.15` — centralised logging for all routers.
- A backup link between the Shared Router and the Business Analysts' Router provides failover/redundancy.

---

## Question 1 — Topology, DHCP, DNS

### 1.1 Network Topology

Full logical topology showing all three floors, the three routers (Developers, Business Analysts, Shared), the central DNS server, and the external partner network, followed by `show ip int brief` on the three routers to confirm interface addressing.

![Network topology](images/p04_0.png)
![show ip int brief output on the three routers](images/p04_1.png)
![Router30 and Router0 interface verification](images/p05_0.png)
![Router30 and Router0 interface verification continued](images/p05_1.png)

`show running-config | section interface` confirms the addressing scheme on all routers:

![show running-config section interface on all routers](images/p06_0.png)

`show vlan brief` — VLANs live on the Floor 2/3 multilayer switches, not on the Business Analysts' or Developers' routers, which only route:

![show vlan brief on the switches](images/p07_0.png)

`ipconfig` verification per team, confirming each device pulled a valid address, mask, gateway, and DNS server from DHCP:

**Developers**
![Developers PC ipconfig](images/p08_0.png)

**Business Analysts**
![Business Analysts PC ipconfig](images/p09_0.png)

**UI Designers**
![UI Designers PC ipconfig](images/p10_0.png)

**Test Engineers**
![Test Engineers PC ipconfig](images/p11_0.png)

**Staging server**
![Staging server ipconfig](images/p12_0.png)

### 1.2 DHCP Server Configuration

Independent DHCP pools for Developers and Business Analysts, each scoped to its own subnet so address assignment stays isolated per team.

![Developers and Business Analyst DHCP pool configuration](images/p13_0.png)
![Developers and Business Analyst DHCP pool configuration continued](images/p13_1.png)

Verification that DHCP is actually handing out addresses (`IP Configuration` panel set to DHCP, plus a full `ipconfig /all` showing the DHCP server, lease, and client ID):

![DHCP verification on client PCs](images/p14_0.png)
![ipconfig /all on Floor 1 PCs](images/p14_1.png)

### 1.3 DNS Server Configuration

A single central DNS server holds A records for every team and the staging host, resolving both internally and for the staging demo domain. `nslookup` from PCs across the topology confirms resolution end-to-end.

![DNS server A records and nslookup from Floor 1 PCs](images/p15_0.png)
![nslookup continued](images/p15_1.png)
![nslookup from the staging server and PC ping tests](images/p16_0.png)
![nslookup and ping continued](images/p16_1.png)

Reachability of the DNS server was verified from every floor:

**From UI Designers**
![Ping DNS server from UI Designers](images/p17_0.png)

**From Test Engineers**
![Ping DNS server from Test Engineers](images/p18_0.png)

**From the staging environment**
![Ping DNS server from staging](images/p19_0.png)

**From Floor 1**
![Ping DNS server from Floor 1](images/p20_0.png)

`show hosts` on the Developers' router confirms the resolved DNS cache for the staging host and each team's device:

![show hosts output](images/p21_0.png)

---

## Question 2 — VLANs, Inter-VLAN Routing, ACLs

### 2.1 VLAN Segmentation

Test Engineers and UI Designers share Floor 2 physically but sit in separate VLANs on separate subnets, confirmed with `show vlan brief` on both access switches and the multilayer switch:

![show vlan brief across Floor 2/3 switches](images/p22_0.png)

`show interface switchport` on the Floor 2 access switch confirms each access port's VLAN assignment:

![show interface switchport part 1](images/p23_0.png)
![show interface switchport part 2](images/p23_1.png)

`show interface trunk` confirms the 802.1Q trunk carrying VLANs 20, 30, and 50 up to the multilayer switch:

![show interface trunk](images/p24_0.png)

### 2.2 Inter-VLAN Routing

The multilayer switch handles inter-VLAN routing (Layer 3 switching rather than router-on-a-stick). `ipconfig` on Test Engineer and UI Designer PCs plus `show ip route` confirm connectivity between VLANs and to shared resources like DNS and staging:

![ipconfig on Test Engineer/UI Designer PCs and show ip route](images/p25_0.png)
![ipconfig continued](images/p25_1.png)
![show ip int brief](images/p26_0.png)
![show running-config section interface Vlan](images/p27_0.png)

### 2.3 Access Control Lists

Extended ACLs (`VLAN_ISOLATION`) restrict direct Test Engineer ↔ UI Designer traffic while carving out explicit exceptions for shared resources (printer, file server):

![show access-list and show running-config section access-list](images/p28_0.png)
![ACL configuration continued](images/p28_1.png)

Hit counts confirm the isolation rules are actually being matched by real traffic:

![ACL hit counts](images/p29_0.png)

---

## Question 3 — Staging Environment: NAT, ACLs, DMZ Design

### 3.1 Packet Tracer Configuration and Evidence

Static NAT/port forwarding exposes the staging server's HTTP/HTTPS ports to external partners without exposing any other internal service:

![show ip nat translation on the router](images/p30_0.png)
![show ip nat translation on the multilayer switch (no router-on-a-stick)](images/p31_0.png)
![show ip nat statistics](images/p32_0.png)
![show running-config section ip nat](images/p33_0.png)

Extended ACLs on the router and both multilayer switches enforce the policy: allow external partner HTTP/HTTPS to staging, allow Developers to manage staging, and explicitly block staging from initiating connections back into Floor 1 or Floor 2:

![Access lists on router and multilayer switches for staging isolation](images/p34_0.png)
![ACL hit counts confirming policy is enforced](images/p35_0.png)
![show logging confirming syslog capture of ACL/routing events](images/p36_0.png)

### 3.2 Theoretical Design Discussion

The staging server sits on its own **VLAN on the Multilayer switch**, acting as a controlled buffer/DMZ segment rather than living directly on the internal LAN. This keeps external partner traffic physically and logically separated from the internal corporate network — if the staging server is compromised, the attacker cannot easily pivot ("east-west" movement) into the Developer/Business Analyst network on Floor 1 or the Test Engineer/UI Designer network on Floor 2.

**Control-plane policy:** OSPF was chosen for the Floor 2/3 domain because it supports routing protocol authentication, which helps prevent rogue route injection into the staging segment.

**Data-plane policy:** the two multilayer switches enforce least privilege —
- All subnets other than Developers and approved external partners are denied access to the staging server.
- External partners may only reach staging over HTTP/HTTPS (TCP 80/443) to its public NAT address.
- Developers get an explicit permit to manage the server.
- An explicit deny rule prevents the staging server from initiating connections back into Floor 1 or Floor 2.
- Further ACLs prevent external partners from reaching any internal subnet other than staging.

**Supporting services:** a **split-horizon DNS** approach is used — internal queries for the staging hostname resolve to the server's private LAN address, while external partner queries resolve to the public NAT address, hiding the internal addressing scheme from outside partners. Logging is centralised via syslog on all routers and switches (RIP and OSPF events included), and ACL hit counters double as basic intrusion/anomaly detection for the staging VLAN.

This design preserves least privilege for internal users, limits the blast radius of a staging compromise to that one VLAN, and still supports the two operational needs the scenario requires: Development updates to staging, and partner-facing demo access.

---

## Question 4 — RIP/OSPF Resilience and Redistribution

### 4.1 Routing Configuration and Backup Link

`show ip protocols` on the Business Analysts' router and the Floor 2/3 multilayer switch confirms RIPv2 on Floor 1 and OSPF on Floor 2/3 respectively:

![show ip protocols on Floor 1 router and Floor 2/3 switch](images/p38_0.png)
![show ip rip database](images/p38_1.png)
![show ip route rip](images/p39_0.png)

`show ip ospf neighbor` confirms full adjacency between the Shared Router and both multilayer switches:

![show ip ospf neighbor](images/p40_0.png)
![show ip ospf interface — multilayer switch acting as a bridge](images/p41_0.png)
![show ip ospf interface — Floor 2/3 multilayer switch, part 1](images/p42_0.png)
![show ip ospf interface — Floor 2/3 multilayer switch, part 2](images/p42_1.png)
![show ip ospf interface on the Shared Router](images/p43_0.png)
![show ip ospf database for all three Layer 3 devices](images/p43_1.png)
![show ip route ospf for all three Layer 3 devices](images/p44_0.png)

**Backup link:** a direct connection between the two multilayer switches (Fa0/5 ↔ Fa0/5) provides failover if the Shared Router's primary path fails, with route redistribution carrying reachability across both routing domains:

![Backup link topology and show ip route on the Shared Router / Floor 2-3 switch](images/p45_0.png)
![show ip route continued](images/p45_1.png)
![show ip route on the multilayer switch acting as the bridge](images/p46_0.png)

### 4.2 RIP/OSPF Mixed-Environment Challenges

Running RIPv2 on Floor 1 alongside OSPF on Floor 2/3 creates a fundamental protocol mismatch that route redistribution only partially papers over:

- **Different philosophies.** RIPv2 is distance-vector ("routing by rumor"), picking paths purely on hop count with slow, periodic full-table updates (up to a 15-hop limit and poor convergence). OSPF is link-state, building a full topology map per area and converging in seconds via triggered LSAs. Neither protocol natively understands the other, so an **ASBR** (Autonomous System Boundary Router) has to run both and perform route redistribution between them.
- **Incompatible metric translation.** OSPF costs (bandwidth-based) don't mean anything to RIP, so redistributed routes get an arbitrary static "seed metric" (e.g. 5 hops) — a 10 Gbps and a 100 Mbps path look identical to RIP once redistributed, so Floor 1 can't make an intelligent path choice. In the other direction, RIP routes injected into OSPF default to External Type 2 (fixed cost regardless of distance from the ASBR), risking suboptimal routing unless manually changed to Type 1.
- **Risk of routing loops.** Because the topology needs *mutual* redistribution (RIP→OSPF and OSPF→RIP), a route can leak out one boundary and be re-injected at the other, causing route poisoning in RIP or LSA flooding in OSPF. Route-tagging, route-maps, and prefix-lists are needed to filter what gets re-advertised, adding configuration complexity and risk.
- **Administrative distance conflicts.** OSPF's default AD (110) beats RIP's (120), so a router will always prefer an OSPF path over a RIP path even when the RIP path is intended as primary — this matters directly for how the backup link behaves.
- **Inconsistent convergence.** OSPF converges in seconds; RIP can take minutes (hold-down/flush timers up to 180–240s). A Floor 1 link failure can leave the OSPF side of the network black-holing traffic toward the ASBR for that whole window.
- **Asymmetric routing.** A packet can leave via the RIP domain and return via a different OSPF path, breaking `traceroute` and, more seriously, breaking stateful firewalls that don't see a matching request for an inbound response and drop the connection as suspicious.
- **The backup link makes this worse.** The direct link between the Shared Router and the Business Analysts' router creates a second, uncontrolled redistribution point — effectively a second ASBR. With two places a route can leak across the boundary, routing loops become close to guaranteed, and failover behaviour becomes non-deterministic: after a primary-link failure, both protocols try to reconverge on different timescales, traffic can flap between paths, and the network may settle on a slow backup route and never automatically return to the original path even after it's restored.

---

## Bibliography

- Adhikari, M. (2025). *Creating Network Diagram or Topology with Cisco Packet Tracer.* Medium. https://medium.com/@minwork/creating-network-diagram-or-topology-with-cisco-packet-tracer-093074d89c20
- Anu Preethi (2022). *Port forwarding using Cisco Packet Tracer.* Studocu. https://www.studocu.com/in/document/vellore-institute-of-technology/information-security-analysis-and-audit/port-forwarding-using-cisco-packet-tracer/24794930
- Breezy Codes (2023). *DHCP Server Configuration Tutorial With Multiple Switches using CISCO Packet Tracer.* YouTube. https://www.youtube.com/watch?v=orLhQDjYTvc
- Cisco (n.d.). *Network Resilience.* https://www.cisco.com/c/en/us/about/trust-center/network-resilience.html
- Computer Networking Tips (2018). *DNS server configuration in Packet Tracer.* https://computernetworking747640215.wordpress.com/2018/07/05/dns-server-configuration-in-packet-tracer
- GeeksforGeeks (2018). *Virtual LAN (VLAN).* https://www.geeksforgeeks.org/computer-networks/virtual-lan-vlan
- GeeksforGeeks (2020). *Difference between RIP and OSPF.* https://www.geeksforgeeks.org/computer-networks/difference-between-rip-and-ospf
- GeeksforGeeks (2022). *Layer 3 Switches in Cisco.* https://www.geeksforgeeks.org/computer-networks/layer-3-switches-in-cisco
- GeeksforGeeks (2018). *Extended Access List.* https://www.geeksforgeeks.org/computer-networks/extended-access-list
- Kaur, N. (2025). *Introduction to HTTP Web Server in Cisco.* LinkedIn. https://www.linkedin.com/posts/dr-navjyot-kaur-85989616_introduction-to-http-web-server-in-cisco-activity-7295146080751521793-Ywl3
- Nick (2021). *Project: DMZ and Network Hardening Tutorial with Packet Tracer.* Cybr. https://cybr.com/network-security-archives/project-dmz-and-network-hardening-tutorial-with-packet-tracer
- Tutorial TKJ (2020). *6.2.4.4 Packet Tracer - Router and Switch Resilience.* YouTube. https://www.youtube.com/watch?v=C-HlVgzIFOI

---

*This was a group project for the ITNSA2-44 Network Security module at Eduvos. This write-up covers the technical design and configuration; group membership and administrative details are omitted here.*
