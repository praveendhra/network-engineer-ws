# Work Sample: Enterprise Healthcare Network Modernization Proposal
**Prepared by:** [Your Name]  
**Role Applied:** Network Engineer 3 — Boston Medical Center  
**Date:** May 2026  
**Document Type:** Technical Work Sample / Portfolio Deliverable

---

## Executive Summary

This document demonstrates my approach to a real-world scenario directly relevant to the Network Engineer 3 role at BMC: a campus network infrastructure assessment and modernization proposal for a multi-site hospital environment. It covers network design, security segmentation, SD-WAN, wireless upgrade, and operational practices — aligned to the specific technologies and responsibilities outlined in the job description.

---

## Scenario

**Organization:** Mid-to-large hospital system (3 campuses, ~4,500 users, 1,200 beds)  
**Problem Statement:** Aging Cisco 3750/6500 core infrastructure, flat network architecture with no clinical/IoT segmentation, unreliable Silver Peak SD-WAN configuration, and no consistent NAC policy. Frequent complaint of VoIP call drops and slow EHR response times. DR network failover has never been successfully tested.

**My role:** Lead Network Engineer responsible for assessment, design, and implementation planning.

---

## 1. Current State Assessment

### 1.1 Network Topology (Current)

```
[Internet/WAN]
      |
  [ASA Firewall] ← hardware EOL, no NGFW capabilities
      |
  [Cisco 6500 Core] ← no redundancy, single supervisor
      |
  [Cisco 3750 Distribution/Access]
      |
  [End Devices: Workstations, VoIP Phones, Medical IoT — all on flat VLAN 1]
```

### 1.2 Key Issues Identified

| Issue | Impact | Risk Level |
|---|---|---|
| Flat network (VLAN 1 everywhere) | No clinical/IoT segmentation; lateral movement risk | CRITICAL |
| Cisco 6500 single supervisor | Core failure = full campus outage | CRITICAL |
| Legacy ASA firewall (no App-ID) | No application visibility or Layer 7 control | HIGH |
| Silver Peak SD-WAN misconfigured | No QoS policies; voice/video traffic not prioritized | HIGH |
| No 802.1X / NAC enforcement | Any device can connect to any port | HIGH |
| Aruba APs on old firmware, no WIPS | Rogue AP risk; compliance gap | MEDIUM |
| No automated config backup | DR runbooks untested; configs undocumented | MEDIUM |
| SolarWinds polling interval too long | 10-min polling misses short-duration outages | MEDIUM |

---

## 2. Proposed Architecture

### 2.1 Target Topology

```
                        [Internet]
                            |
                    [Palo Alto PA-5250 HA Pair]
                    [Panorama Central Management]
                            |
               [Cisco Nexus 9508 — Core (VXLAN/EVPN)]
               [Dual Supervisors | VSS/vPC | BGP/OSPF]
                /                           \
    [Nexus 9300 — Dist A]           [Nexus 9300 — Dist B]
         |                                   |
  [Access Switches]                   [Access Switches]
  [Cisco 9200/9300]                   [Cisco 9200/9300]
         |
  [Aruba APs (Wi-Fi 6)]
  [Aruba ClearPass — NAC]
         |
  [Silver Peak SD-WAN — Unity Orchestrator]
         |
  [WAN: MPLS Primary | Broadband Failover | LTE Emergency]
```

### 2.2 Network Segmentation (VRF + VLAN Design)

| Zone | VLAN Range | VRF | Access Policy |
|---|---|---|---|
| Clinical Workstations | 100–149 | VRF-CLINICAL | 802.1X; ClearPass; EHR/PACS access only |
| VoIP | 200–209 | VRF-VOICE | Trusted; DSCP EF; CDP auto-VLAN |
| Medical IoT | 300–349 | VRF-IOT | MAC-Auth Bypass; no internet; tightly ACL'd |
| Server / Datacenter | 400–449 | VRF-DC | Firewall-enforced; no direct user access |
| Guest / Visitor Wi-Fi | 500 | VRF-GUEST | Internet-only; isolated; ClearPass sponsored access |
| Management | 999 | VRF-MGMT | Jump host access only; OOB preferred |

> **Healthcare Rationale:** Medical IoT devices (infusion pumps, imaging systems) often run legacy OS and cannot be patched. Isolation in a dedicated VRF with strict ACLs containing their blast radius is critical for HIPAA Security Rule compliance (§164.312 — Access Control).

---

## 3. Security Design — Palo Alto Zero Trust Implementation

### 3.1 Zone Architecture

```
Zones:
  UNTRUST  → internet-facing
  DMZ      → public-facing services (patient portal, external APIs)
  CLINICAL → EHR, PACS, clinical workstations
  IOT      → medical devices
  VOICE    → VoIP infrastructure
  DC       → datacenter servers
  MGMT     → network management plane
  GUEST    → visitor internet
```

### 3.2 Firewall Policy Philosophy

- **Default Deny All** between zones; explicit permit only.
- Use **App-ID** instead of port-based rules: e.g., allow `epic-ehr` application, not just TCP/443.
- **User-ID** integration with Active Directory: policies apply to user groups, not just IPs.
- **Threat Prevention** profile on all inter-zone rules; **WildFire** for unknown file inspection.
- Clinical-to-DC rule example:

```
Rule: Allow-EHR-Access
  Source Zone:      CLINICAL
  Destination Zone: DC
  Application:      epic-ehr, ssl
  Service:          application-default
  Source User:      domain\clinical-staff
  Action:           Allow
  Profile:          Threat-Prevention-Strict
  Log:              Yes (forward to Panorama)
```

### 3.3 Palo Alto High Availability Config (Key Settings)

```
set deviceconfig high-availability enabled yes
set deviceconfig high-availability group 1 mode active-passive
set deviceconfig high-availability group 1 peer-ip 10.0.99.2
set deviceconfig high-availability group 1 election-option heartbeat-backup enabled yes
set deviceconfig high-availability group 1 state-synchronization enabled yes
```

---

## 4. SD-WAN Configuration — Aruba Silver Peak

### 4.1 Problem Diagnosed

Silver Peak was operational but using default routing with no application-aware policies. VoIP (SIP/RTP) was being sent over the higher-latency broadband link when MPLS was available, causing jitter exceeding 30ms and call drops.

### 4.2 Solution: Application-Aware QoS Routing

**Business Intent Overlays configured in Unity Orchestrator:**

| Overlay | Applications | Primary Path | Failover | SLA Threshold |
|---|---|---|---|---|
| Realtime-Voice | SIP, RTP, H.323 | MPLS | LTE | Latency < 20ms, Jitter < 5ms, Loss < 0.1% |
| Clinical-Data | Epic EHR, PACS, HL7 | MPLS | Broadband | Latency < 50ms, Loss < 0.5% |
| General-Business | HTTP/S, Email, DNS | Broadband | MPLS | Best-effort |
| Guest | Internet browsing | Broadband | — | Best-effort; throttled 10Mbps |

**Packet Order Correction + FEC** enabled on Realtime-Voice overlay to compensate for broadband path impairments.

### 4.3 Result (Post-Change Validation)

- VoIP MOS score improved from 2.8 → 4.2 (SolarWinds VoIP Quality Manager)
- EHR screen load times reduced by ~40% during peak hours
- WAN failover tested: MPLS → broadband cutover in < 2 seconds (verified via continuous ping + SolarWinds alert timestamps)

---

## 5. Wireless Infrastructure Upgrade — Aruba Wi-Fi 6

### 5.1 RF Design Approach

- Conducted **Ekahau site survey** for each floor/wing before AP placement.
- Deployed **Aruba AP-635** (Wi-Fi 6, tri-radio) in high-density clinical areas; **AP-515** in corridors.
- Configured **band steering** to push capable clients to 5 GHz; **airtime fairness** enabled.
- Set transmit power to **auto** with min/max guard rails (7 dBm min, 18 dBm max) to prevent co-channel interference.
- SSID design:

| SSID | Band | Security | VLAN | NAC Policy |
|---|---|---|---|---|
| BMC-Clinical | 5 GHz preferred | WPA3-Enterprise / 802.1X | 100 | ClearPass: domain device + user cert |
| BMC-Voice | 5 GHz | WPA2-Enterprise | 200 | ClearPass: MAC-Auth for handsets |
| BMC-IoT | 2.4 GHz / 5 GHz | WPA2-PSK (per-device) | 300 | ClearPass: MAC-Auth Bypass |
| BMC-Guest | 5 GHz | Captive Portal | 500 | ClearPass: sponsored / self-register |

### 5.2 ClearPass NAC Policy (802.1X Flow)

```
Authentication:
  1. EAP-TLS (device certificate) → AD computer object check
  2. PEAP-MSCHAPv2 fallback (user credentials) → AD group membership check

Authorization Rules:
  IF [AD-Group = "Clinical-Staff"] AND [Device-Cert = Valid]
    → VLAN 100, dACL: permit-clinical-apps
  IF [AD-Group = "Contractor"] AND [Device-Cert = None]
    → VLAN 500 (Guest), redirect to IT approval portal
  IF [MAC = known-IoT-device-list]
    → VLAN 300, dACL: permit-dst-only 10.40.0.0/16 (healthcare app servers)
  Default:
    → DENY / quarantine VLAN
```

---

## 6. Monitoring & Observability — SolarWinds

### 6.1 Monitoring Architecture

- **SolarWinds NPM**: All network nodes (SNMP v3 only; v1/v2c disabled for security).
- **SolarWinds NTA (NetFlow)**: NetFlow v9 exported from all distribution-layer switches; top-talker analysis per VLAN.
- **SolarWinds NCM**: Automated nightly config backups; compliance policy checking (no `enable password`, no telnet, SSH v2 enforced).
- **Airwave / Aruba Central**: Wireless health dashboards; client roaming analysis; rogue AP alerting.

### 6.2 Alert Thresholds (Production, Not Default)

| Metric | Warning | Critical | Action |
|---|---|---|---|
| Core switch CPU | 60% | 80% | Page on-call + auto-create ServiceNow incident |
| WAN circuit utilization | 70% | 90% | Capacity review ticket created automatically |
| VoIP VLAN packet loss | 0.1% | 0.5% | Immediate escalation to network on-call |
| AP client count per radio | 25 | 40 | RF team notified for load balancing |
| Firewall session table | 75% | 90% | Palo Alto TAC + change request for scale-out |
| Config drift detected | — | Any | Auto-restore from NCM + ServiceNow change alert |

### 6.3 Custom SolarWinds Dashboard: Clinical Network Health

Created a role-specific dashboard for the Help Desk showing:
- Real-time status of EHR/PACS server farm connectivity
- Top 10 bandwidth users per clinical VLAN
- Active wireless clients per floor/wing
- WAN link health (latency/jitter/loss per Silver Peak overlay)

This reduced L1 escalations to the network team by ~30% — Help Desk could self-triage before escalating.

---

## 7. Change Management Sample — ServiceNow CHG Record

**Change Title:** Core Network Upgrade — Replace Cisco 6500 with Nexus 9508 (Campus A)  
**Change Type:** Normal (Standard approval required)  
**Risk Level:** High  
**Change Window:** Saturday 01:00–05:00 (4-hour window)

### Pre-Change Checklist

- [ ] Nexus 9508 staged, tested in lab with production config
- [ ] All interface configs migrated and peer-reviewed by second engineer
- [ ] SolarWinds NCM config backup of 6500 captured (timestamped)
- [ ] Rollback tested in lab: old 6500 reconnected in < 15 minutes
- [ ] On-call clinician IT liaison notified; Epic team on standby
- [ ] Maintenance mode set in SolarWinds (suppress alerts during window)
- [ ] Change approved by CAB; PAUL MITCHELL sign-off confirmed

### Implementation Steps (Condensed)

```
Step 1: (00:00) Confirm no active clinical procedures on affected segment (with charge nurse)
Step 2: (00:05) Enable maintenance mode in SolarWinds NPM
Step 3: (00:10) Gracefully migrate routing: redistribute routes to backup path
Step 4: (00:20) Disconnect 6500; patch fiber to Nexus 9508
Step 5: (00:30) Bring up Nexus 9508; verify BGP/OSPF adjacencies
Step 6: (00:45) Validate: ping all gateway IPs, verify EHR connectivity from test workstation
Step 7: (01:00) Monitor SolarWinds for 30 min; verify no alerts
Step 8: (01:30) Disable maintenance mode; notify stakeholders — change successful
```

### Rollback Plan

If EHR/critical system connectivity not restored within 30 minutes of cutover:
1. Reconnect Cisco 6500 to fiber uplinks
2. Re-enable 6500 routing (configs preserved, not wiped)
3. Restore OSPF adjacencies (< 5 min convergence expected)
4. Notify on-call engineer; open P1 incident; CAB post-mortem scheduled within 48 hours

---

## 8. Root Cause Analysis Sample — VoIP Outage

**Incident:** P2 — Intermittent VoIP call drops, Campus B, affecting ~200 users  
**Duration:** 4 hours  
**Detection:** SolarWinds NTA alert — VoIP VLAN packet loss spiked to 3.2%

### Timeline

| Time | Event |
|---|---|
| 09:15 | SolarWinds alert: VLAN 200 packet loss > 0.5% threshold |
| 09:17 | On-call network engineer paged via ServiceNow |
| 09:25 | Identified: distribution switch SW-B-DIST-02 CPU at 98% |
| 09:35 | Root cause isolated: misconfigured spanning tree (BPDU storm from unmanaged switch plugged into port by facilities team) |
| 09:50 | Port shut down; BPDU Guard triggered; STP topology stabilized |
| 10:00 | VoIP packet loss returned to < 0.05%; calls restored |
| 13:15 | Post-incident: BPDU Guard enabled globally on all access ports; incident closed |

### Root Cause

An unmanaged 8-port consumer switch was connected to an access port by facilities, creating a loop. BPDU Guard was not enabled on that port, allowing the loop to propagate and flood VLAN 200.

### Corrective Actions

1. **Immediate:** Enabled BPDU Guard on all access ports campus-wide via automated NCM script.
2. **Short-term:** ClearPass policy updated — non-802.1X ports auto-shut after 30 seconds.
3. **Long-term:** Physical security audit of all IDF closets; keycard access required.
4. **Process:** Added "unmanaged switch connected" scenario to NOC Level 1 troubleshooting runbook.

---

## 9. Disaster Recovery — Network DR Runbook Excerpt

### Scenario: Campus A Core Switch Failure

**Recovery Time Objective (RTO):** 30 minutes  
**Recovery Point Objective (RPO):** Last nightly config backup (SolarWinds NCM)

### Runbook

```
1. DETECT
   - SolarWinds NPM critical alert: Core-A-NEXUS-01 unreachable
   - Confirm: physical check or out-of-band console (OOB MGMT network)

2. ISOLATE
   - Confirm whether failure is hardware (supervisor) or software (process crash)
   - Check: "show system resources" and "show module" via OOB console

3. FAILOVER (if hardware failure confirmed)
   - Redundant supervisor: auto-failover (< 30 sec if NSF/SSO configured)
   - If chassis failure: activate pre-staged spare Nexus 9508 in DR rack
     a. Load config from NCM backup (SCP from jump host)
     b. Reconnect fiber uplinks in IDF patch panel
     c. Verify OSPF/BGP adjacencies: "show ip ospf neighbor" / "show bgp summary"
     d. Confirm EHR reachability from clinical test workstation

4. VALIDATE
   - Ping all 20 critical server IPs (automated script: /scripts/validate-core.sh)
   - Confirm SolarWinds shows all nodes green
   - Call charge nurse on each floor to confirm clinical system access

5. COMMUNICATE
   - Update ServiceNow P1 incident every 15 minutes
   - Notify PAUL MITCHELL (Hiring Manager / Network Lead) and IT leadership
   - Post-incident RCA within 48 hours
```

---

## 10. Automation & Scripting (Value-Add)

While not explicitly required, I bring automation skills that reduce toil and human error:

### Config Compliance Check (Python + Netmiko)

```python
from netmiko import ConnectHandler
import re

# Ensure all devices comply: SSH v2 only, no telnet, BPDU Guard on access ports
compliance_checks = [
    ("transport input ssh", "Telnet disabled"),
    ("ip ssh version 2", "SSH v2 enforced"),
    ("spanning-tree portfast bpduguard default", "BPDU Guard global"),
]

devices = [...]  # loaded from IPAM/SolarWinds asset list

for device in devices:
    conn = ConnectHandler(**device)
    config = conn.send_command("show running-config")
    for check, description in compliance_checks:
        status = "PASS" if re.search(check, config) else "FAIL"
        print(f"{device['host']} | {description}: {status}")
    conn.disconnect()
```

This script runs nightly via cron, outputs to a CSV, and creates a ServiceNow task for any FAIL result — zero manual auditing required.

---

## Summary: Why This Approach Stands Out

| Differentiator | Detail |
|---|---|
| Healthcare context awareness | HIPAA segmentation, IoT isolation, uptime-as-patient-safety mindset |
| End-to-end scope | Design → implementation → monitoring → DR → automation |
| Real troubleshooting methodology | BPDU storm RCA with permanent corrective actions, not just fixes |
| Formal change discipline | CAB, rollback plans, stakeholder communication baked in |
| Proactive monitoring | Custom dashboards reduced L1 escalations; alerts tuned to clinical SLAs |
| Quantified outcomes | VoIP MOS 2.8 → 4.2; EHR load time -40%; escalations -30% |
| Automation thinking | Compliance scripts reduce audit toil and enforce standards at scale |

---

*This work sample reflects the depth of technical and operational experience I would bring to the Network Engineer 3 role at Boston Medical Center. All scenario details are representative composites based on real-world enterprise healthcare network engineering experience.*
