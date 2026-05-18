# Network Engineer 3 — Job Analysis & Best Approaches

**Role:** Network Engineer 3 | Boston Medical Center (BMC)  
**Department:** Tech Support | Full Time  
**Compensation:** $89,500 – $130,000  
**Required Cert:** CCNP  
**Experience:** 6–8 years enterprise network engineering

---

## 1. Core Competency Areas

| Domain | Key Focus |
|---|---|
| Data & VoIP Networks | Design, deploy, and manage LAN/WAN infrastructure |
| Routing & Switching | Cisco routers (2800–7600), switches (3750–6500), Nexus 9000 |
| Security | Palo Alto Firewalls, TACACS+, RADIUS, network segmentation |
| Wireless | Aruba wired/wireless, Airwave, ClearPass (NAC) |
| SD-WAN | Aruba Silver Peak |
| Load Balancing | F5 BIG-IP, Citrix NetScaler |
| Monitoring | SolarWinds, NetFlow, Airwave |
| Protocols | TCP/IP, OSPF, BGP4, VXLAN, Multicast, Anycast, SIP |
| ITSM | BMC ServiceNow (ticketing, change management, RCA) |
| Documentation | Microsoft Visio (network diagrams), IP address/circuit databases |

---

## 2. Best Approaches for Day-to-Day Responsibilities

### 2.1 Network Design & Architecture

- Follow a **hierarchical 3-tier model** (Core → Distribution → Access) or **spine-leaf** for datacenter (Nexus 9000 with VXLAN/EVPN).
- Apply **modular design** — segment clinical, administrative, and guest traffic using VLANs, VRFs, and micro-segmentation.
- Use **redundancy at every layer**: dual uplinks, HSRP/VRRP, port-channels (LACP), and redundant WAN circuits.
- Document all designs in Visio **before** implementation; maintain living network diagrams post-deployment.

### 2.2 Routing & Switching Best Practices

- Use **OSPF** for intra-domain routing (fast convergence, area design to limit LSA flooding).
- Use **BGP4** for WAN/internet edge and multi-homed connections; apply proper route policies (prefix-lists, route-maps).
- Leverage **VXLAN with EVPN** on Nexus 9000 for scalable datacenter overlay networking.
- Implement **Cisco Best Practice Hardening**: disable unused services, use loopback as router-ID, enable BFD for fast failure detection.
- Standardize VLANs and IP addressing schemes; document in a maintained IPAM (IP Address Management) database.

### 2.3 VoIP / SIP Infrastructure

- Ensure **QoS** (DSCP marking, priority queuing) is consistently applied end-to-end for voice and video traffic.
- Separate voice traffic into a dedicated VLAN; configure CDP/LLDP for IP phone auto-provisioning.
- Monitor jitter, latency, and packet loss with tools like SolarWinds VoIP & Network Quality Manager.
- Validate SIP trunks and codec negotiation; use Wireshark/packet captures for troubleshooting call quality issues.

### 2.4 Wireless Infrastructure (Aruba)

- Deploy Aruba APs managed via **Airwave** or **Aruba Central**; enforce RF planning (channel, power, band steering).
- Use **Aruba ClearPass** for policy-based network access control (NAC): 802.1X, MAB, guest onboarding.
- Segment SSID traffic using tunneled or bridged modes to appropriate VLANs.
- Regularly audit rogue AP detection and enforce WIPS policies in healthcare environments.

### 2.5 Security (Palo Alto, Zero Trust)

- Implement **Zero Trust segmentation** — never trust, always verify; apply least-privilege firewall policies.
- Use **Palo Alto App-ID and User-ID** for granular application-layer visibility and control.
- Zone-based security policies: separate zones for clinical systems, servers, guest, IoT/medical devices.
- Enable **Threat Prevention, URL Filtering, and WildFire** on Palo Alto for advanced threat protection.
- Regularly audit firewall rule bases; remove stale/overly permissive rules.
- Integrate with TACACS+ for centralized device authentication and accounting.

### 2.6 SD-WAN (Aruba Silver Peak)

- Centralize SD-WAN policy via **Unity Orchestrator**; apply application-aware routing policies.
- Configure **path conditioning** (FEC, packet order correction) for latency-sensitive clinical applications.
- Use **dynamic path control** to route traffic over best-performing circuit (MPLS, broadband, LTE).
- Monitor WAN health continuously; set SLA thresholds per application class.

### 2.7 Monitoring & Performance Management (SolarWinds / NetFlow)

- Deploy **SolarWinds NPM + NTA** for comprehensive node monitoring and traffic flow analysis.
- Set baseline thresholds for CPU, memory, bandwidth utilization; configure alert policies for anomalies.
- Use **NetFlow/IPFIX** to identify top talkers, bandwidth hogs, and unusual traffic patterns proactively.
- Create custom dashboards for real-time visibility into critical clinical network segments.
- Schedule regular capacity planning reviews (monthly/quarterly) using trend data from SolarWinds.

### 2.8 Change Management & ITSM (ServiceNow)

- Submit all changes via **ServiceNow Change Management**; include risk assessment, rollback plans, and test plans.
- Use the **CAB (Change Advisory Board)** process for high-risk changes (core router/switch upgrades, firewall rule changes).
- Document RCA (Root Cause Analysis) for all P1/P2 incidents; focus on permanent fixes, not just workarounds.
- Maintain a change freeze calendar aligned with hospital operational peaks (flu season, budget close, etc.).

### 2.9 Disaster Recovery & Resiliency

- Maintain and regularly test **DR runbooks** for all network device types: routers, switches, firewalls, WLCs.
- Use **configuration backup tools** (Rancid, Oxidized, or SolarWinds NCM) with versioned config storage.
- Test failover scenarios (WAN failover, core switch failover) during scheduled maintenance windows.
- Align network DR with hospital BCP (Business Continuity Plan) — ensure clinical systems (EMR, PACS) are prioritized.

### 2.10 Documentation Standards

- Maintain **Visio diagrams** for physical and logical topology (L1, L2, L3 diagrams).
- Keep an up-to-date **IP address management (IPAM)** database with circuit IDs, device hostnames, and location info.
- Document all device images/firmware versions; track EOL/EOS dates for proactive hardware refresh planning.
- Standardize naming conventions for devices, VLANs, and interfaces across the enterprise.

---

## 3. Professional Development Roadmap

| Priority | Certification / Skill | Rationale |
|---|---|---|
| Must Have | **CCNP** (Enterprise or Data Center) | Role requirement |
| High | **Palo Alto PCNSE** | Firewall is core to this role |
| High | **Aruba Certified Professional (ACP)** | Wireless & ClearPass are key platforms |
| Medium | **F5 201 / 301** (BIG-IP) | Load balancer expertise |
| Medium | **Aruba Silver Peak SD-WAN** | SD-WAN skill in use at BMC |
| Growth | **CCIE Enterprise / Data Center** | Senior/architect path |
| Growth | **AWS/Azure Networking** | Cloud network integration trending in healthcare |

---

## 4. Healthcare-Specific Considerations

- **HIPAA compliance**: All network changes must consider PHI (Protected Health Information) data flows; segmentation of clinical vs. non-clinical traffic is critical.
- **Uptime is patient safety**: Rolling changes should follow strict change windows; always have a rollback plan ready.
- **IoT / Medical devices**: Segment medical IoT devices (infusion pumps, imaging systems) in dedicated VLANs with strict access control; these devices often cannot be patched.
- **24/7 on-call**: Be prepared for off-hours support; build runbooks detailed enough for on-call response without needing full context.
- **Vendor coordination**: Manage Cisco TAC, Aruba support, Palo Alto support relationships; know support contract entitlements and escalation paths.

---

## 5. Key Success Metrics for This Role

- Network uptime/availability SLA (target: 99.99%+ for clinical systems)
- Mean Time to Resolution (MTTR) for network incidents
- Change success rate (% of changes with no unplanned outages)
- Reduction in repeat incidents (RCA effectiveness)
- Wireless coverage quality scores (signal strength, roam success rate)
- WAN circuit utilization vs. capacity thresholds
- Audit compliance (firewall rule review, config backup success rate)

---

## 6. Tooling Stack Summary

| Category | Tool / Platform |
|---|---|
| Routing/Switching | Cisco IOS/IOS-XE/NX-OS |
| Wireless | Aruba APs, Airwave, ClearPass |
| Firewall | Palo Alto (Panorama for centralized mgmt) |
| Load Balancer | F5 BIG-IP, Citrix NetScaler |
| SD-WAN | Aruba Silver Peak (Unity Orchestrator) |
| Monitoring | SolarWinds NPM/NTA/NCM, NetFlow |
| ITSM | BMC ServiceNow |
| Diagramming | Microsoft Visio |
| Packet Analysis | Wireshark |
| Config Backup | Oxidized / SolarWinds NCM |
| Scripting (optional) | Python (Netmiko, NAPALM, Ansible) |
