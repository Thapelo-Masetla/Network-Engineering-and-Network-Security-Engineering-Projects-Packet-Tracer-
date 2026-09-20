# BrightSoft Solutions — Segmented Enterprise Network Design

**Module:** ITNSA2-44 (Network Security) — Eduvos
**Tool:** Cisco Packet Tracer
**Type:** Group project (network design, configuration, and security policy)

## Overview

This project designs and configures a segmented enterprise network in Cisco Packet Tracer for **BrightSoft Solutions**, a fictional software services company in Durban, South Africa. The build covers DHCP, DNS, VLAN segmentation with inter-VLAN routing, NAT/port forwarding for a partially internet-facing staging server, extended ACLs for lateral-movement control, and dual routing protocols (RIPv2 on Floor 1, OSPF on the Shared Router/Floor 2–3) with a backup link between domains.

The full Packet Tracer file is included in this repo: [`BrightSoft-Network-Design.pkt`](./BrightSoft-Network-Design.pkt).

## Scenario

BrightSoft Solutions is a software services company headquartered in Durban, South Africa, operating across three dedicated floors. The company has experienced rapid growth, requiring a well-planned, segmented, and secure network infrastructure to support its expanding teams and a new staging environment for client demonstrations.

The company network is allocated the IP address space **192.168.0.0/24** and is structured across three physical floors with interconnected routers, switches, servers, and end-user devices.

### Floor 1: Developers and Business Analysts

**Teams and devices:**
- Developers Team — 3 end-user devices, with a DHCP server managed locally by the Developers team.
- Business Analysts Team — 3 end-user devices, with a DHCP server managed locally by the Business Analysts team.

**Network connectivity:**
- Each team has its own router connecting the team's devices to centralised services.
- Developers' router connects to: the Central DNS Server, the Staging Environment (Floor 3), and the Internet.
- Business Analysts' router connects to: the Central DNS Server, the Production Web Server, and the Internet.
- Routing protocol: **RIP v2** (required for legacy applications).

### Floor 2: Test Engineers and UI Designers

**Teams and devices:**
- Test Engineers — 2 end-user devices.
- UI Designers — 2 end-user devices.
- Both teams share the floor but require logical separation via VLANs for security.

**Shared resources:**
- A common printer and shared file server.
- Both teams can access these resources through controlled inter-VLAN routing.

**Network connectivity:**
- A Layer 3 switch handles inter-VLAN routing with 802.1Q trunking.
- A single shared router connects Floor 2 to the staging environment (Floor 3) and the Internet.
- Routing protocol: **OSPF** (Open Shortest Path First, for scalable routing).

### Floor 3: Staging Environment

**Infrastructure — Staging Web Server:**
- Used for client demonstrations and receives updates from the Developers team.
- Accessible from the Internet (external partners) on HTTP/HTTPS.
- Accessible from the Developers team for updates and maintenance.
- Restricted from Business Analysts, Test Engineers, and UI Designers.

### Centralised Infrastructure

- **Central DNS Server — 192.168.0.13:** provides internal name resolution for all devices across all floors, and handles external DNS lookups for internet access and staging demonstrations. Accessible from all routers and all VLANs.
- **Production Web Server — 192.168.0.14:** hosts live company applications accessible by all internal departments. Connected through the Business Analysts' router.
- **Syslog Server — 192.168.0.15:** centralised logging for all routers and network events.

### Router Interconnection and Redundancy

The three routers are interconnected to allow inter-floor communication:
- Developers' router and Business Analysts' router both connect to the Central DNS Server.
- Shared router connects both Floor 2 and Floor 3 to the centralised services.
- A backup link exists between the Shared router and Business Analysts' router for failover and redundancy.
- All routers connect to the Internet through separate ISP connections.

## Questions

### Question 1 — 20 Marks

**1.1** Using Cisco Packet Tracer, design a complete network layout that reflects the three floors described in the scenario. On Floor 1, include three devices for the Developers and three devices for the Business Analysts. Floor 2 should consist of two devices for the Test Engineers and two devices for the UI Designers, while Floor 3 is dedicated to a single staging web server. The design must also incorporate the three routers outlined in the scenario. *(5 marks)*

**1.2** Configure independent DHCP servers for the Developers and the Business Analysts on Floor 1. Each DHCP server should be capable of assigning addresses exclusively to its respective VLAN or subnet, ensuring proper isolation of traffic. Once configured, verify that all devices on Floor 1 automatically obtain valid network parameters including IP addresses, subnet masks, default gateways, and DNS details. *(10 marks)*

**1.3** Install and configure a DNS server that is accessible from all three floors of the company network. This server must be capable of resolving hostnames for internal devices across the organisation while also supporting external lookups, which will allow access to internet resources as well as the staging environment used for client demonstrations. *(5 marks)*

### Question 2 — 20 Marks

**2.1** On Floor 2 of BrightSoft Solutions, create two separate VLANs to ensure logical segmentation between the Test Engineers and the UI Designers, who share the same physical space. Each VLAN should be configured with its own subnet and assigned to the appropriate switch ports, ensuring that devices from each department can only communicate within their respective VLAN by default. *(10 Marks)*

**2.2** Configure the Shared Router, or alternatively a Layer 3 switch, to provide inter-VLAN routing. This configuration should allow controlled communication between the VLANs when necessary and ensure that traffic from the Test Engineers and UI Designers can also reach common resources such as the DNS server and the staging environment. *(5 Marks)*

**2.3** Apply an extended ACL or a comparable mechanism on the Shared Router to restrict direct communication between the Test Engineers' VLAN and the UI Designers' VLAN. The ACL should block traffic unless specific exceptions are authorised, such as access to a shared printer or file server. This ensures strict isolation while still supporting minimal collaboration where required. *(5 Marks)*

### Question 3 — 30 Marks

**3.1** In Cisco Packet Tracer, configure the Floor 3 staging environment so that approved external partners can access the staging web service over HTTP/HTTPS while maintaining strict isolation from the rest of the internal network. Your configuration should demonstrate the use of NAT or port forwarding on the edge device, together with extended ACLs that restrict inbound and lateral movement, and should permit the Development team to update the staging server without exposing additional internal resources.

Provide evidence of your configuration and policy effectiveness using screenshots or command outputs (e.g., NAT translations, ACL hit counts, successful/blocked connection tests). Your submission should clearly show how you balance external demo accessibility with internal security. *(20 Marks)*

**3.2** Beyond the Packet Tracer build, discuss the theoretical design for integrating a partially externally accessible staging environment into BrightSoft's network. In your answer, justify the placement of the staging server (e.g., dedicated VLAN/DMZ segment), outline the required control plane and data plane policies (firewall rules, ACLs, routing considerations), and describe supporting services (DNS resolution approach, certificate management, logging/monitoring, and basic intrusion detection around the staging segment).

Explain how your design preserves least privilege for internal users, limits the blast radius of a compromise, and supports operational needs such as Development updates and partner access. *(10 Marks)*

### Question 4 — 30 Marks

**4.1** In Cisco Packet Tracer, enhance network resilience and monitoring by enabling RIP on one router for legacy compatibility and OSPF on the Shared Router for efficient routing. Show how you would introduce a failover or backup link to maintain connectivity if a primary link fails. *(15 Marks)*

**4.2** Explain the potential challenges BrightSoft Solutions may encounter when running both RIP and OSPF within the same network infrastructure. In your answer, consider issues such as protocol compatibility, administrative complexity, and route redistribution between the two protocols. Additionally, evaluate how these challenges could affect the introduction of a failover or backup link designed to improve network resilience. *(15 Marks)*

