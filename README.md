# SOC Home Lab — Network Topology & Segmentation

## Objective

This project documents the design and implementation of a virtualized home network lab built to develop and demonstrate skills aligned with CompTIA Network+ (N10-009) and Security+ (SY0-701) certifications. The lab simulates a segmented enterprise network environment using free and open-source tools, providing hands-on experience with routing, VLANs, firewall configuration, and network security fundamentals relevant to a Security Operations Center (SOC) analyst role.

> **Lab Status: In Progress** — OPNsense installed, hardened, and configured with three VLANs and deny-by-default firewall rules. Kali Linux and Ubuntu Server installed on the lab network. Internet access for lab VMs is currently blocked by a WAN/LAN subnet conflict — see Troubleshooting and Next Steps.

---

## Lab Environment

| Component | Details |
|---|---|
| Host OS | Windows 11 (new laptop, 16 GB RAM, Wi-Fi connected) |
| Hypervisor | VMware Workstation Pro 17.0 |
| Router / Firewall | OPNsense 26.1.6 (community edition) |
| Attack VM | Kali Linux — installed, on lab network |
| Server Target | Ubuntu Server 22.04 LTS — installed, on lab network |
| Workstation Target | Windows 10 Enterprise Evaluation (not yet built on new laptop) |

> 📸 **Screenshot:**
> <img width="96" height="98" alt="11  vmware vm list" src="https://github.com/user-attachments/assets/68d8745b-0b6f-4e22-8a83-e085275abefd" />
— VMware Workstation library showing OPNsense, Kali, and Ubuntu-Server VMs

---

## Network Topology

```
[ Home Router / Internet ]
            |
   [ WAN — em1 — VMnet8 (NAT), 192.168.91.x ]
            |
            |   *** Known conflict: WAN and LAN currently share
            |       the 192.168.91.0/24 subnet — see Troubleshooting ***
            |
   [ OPNsense 26.1.6 Firewall / Router ]
            |
   [ LAN — em0 — VMnet8 (NAT), 192.168.91.10/24 ]
       (Management — web UI access, https://192.168.91.10:8443)
            |
   [ OPT1 (LABLAN) — em2 — VMnet2 (Host-only), 10.0.1.0/24 ]
       Kali Linux  → 10.0.1.20
       Ubuntu Server → 10.0.1.10
            |
   ┌────────┴──────────────────────────┐
   │                                    │
[ VLAN 10 — 10.0.10.0/24 ]      [ VLAN 20 — 10.0.20.0/24 ]
   Workstations                       Servers

         [ VLAN 99 — 10.0.99.0/24 ]
            Attack (isolated)
         (no DHCP, WAN blocked)
```

> **Note:** VLANs 10, 20, and 99 are fully configured on OPNsense with their own subnets, DHCP (10/20), and firewall rules. Kali and Ubuntu Server currently sit on the untagged LABLAN segment (10.0.1.0/24) rather than inside a specific VLAN — assigning lab VMs into VLAN10/20/99 is the next configuration task (see Next Steps).

---

## IP Addressing Scheme

| Network | Subnet | Gateway | VMware Network | Status |
|---|---|---|---|---|
| WAN | 192.168.91.0/24 (DHCP from VMware NAT) | 192.168.91.2 | VMnet8 (NAT) | Active — conflicts with LAN, see Troubleshooting |
| Management (LAN) | 192.168.91.0/24 | 192.168.91.10 | VMnet8 (NAT) | Active — web UI access |
| Lab LAN (OPT1/LABLAN) | 10.0.1.0/24 | 10.0.1.1 | VMnet2 (Host-only) | Active — Kali, Ubuntu Server |
| VLAN 10 — Workstations | 10.0.10.0/24 | 10.0.10.1 | (tagged on em2) | Configured, no VMs assigned yet |
| VLAN 20 — Servers | 10.0.20.0/24 | 10.0.20.1 | (tagged on em2) | Configured, no VMs assigned yet |
| VLAN 99 — Attack (isolated) | 10.0.99.0/24 | 10.0.99.1 | (tagged on em2) | Configured, no DHCP, WAN blocked |

> **Planned addressing (post WAN fix):** Once WAN is moved to a Bridged adapter on the home network (e.g. 192.168.1.x), Management (LAN) will be moved back to VMnet1 at 192.168.100.0/24, restoring separation between WAN and management subnets.

---

## Tools Used

| Tool | Purpose | Cost |
|---|---|---|
| VMware Workstation Pro 17.0 | Hypervisor / VM management | Free (personal use) |
| OPNsense 26.1.6 | Firewall, router, VLAN management | Free / open source |
| Kali Linux | Attack simulation, packet analysis — installed | Free |
| Ubuntu Server 22.04 | Target server (SSH, Apache) — installed | Free |
| Windows 10 Enterprise Eval | Target workstation — pending on new laptop | Free (90-day eval) |
| Wireshark | Packet capture and analysis — not yet installed | Free |

---

## Key Configurations

### VMware Network Adapters (OPNsense VM) — current

| Adapter | VMware Network | OPNsense Interface | Role |
|---|---|---|---|
| Network Adapter 1 | NAT (VMnet8) | em1 (WAN) | Internet access |
| Network Adapter 2 | NAT (VMnet8) | em0 (LAN) | Management — web UI access |
| Network Adapter 3 | Host-only (VMnet2) | em2 (OPT1/LABLAN) | Lab LAN — VM-to-VM communication |

> Adapter 2 was moved from VMnet1 to VMnet8 during this session after VMnet1 host-to-VM traffic could not be resolved. This introduced the WAN/LAN subnet conflict documented below.

### VMware Virtual Network Configuration

| VMware Network | Type | Subnet | DHCP | Purpose |
|---|---|---|---|---|
| VMnet1 | Host-only | 192.168.100.0/24 | Disabled | Originally planned management network — not currently used |
| VMnet2 | Host-only | 10.0.1.0/24 | Disabled | Lab internal network (Kali, Ubuntu Server) |
| VMnet8 | NAT | 192.168.91.0/24 | Enabled (VMware) | WAN and LAN (current — see conflict note) |

### OPNsense Interface Assignment

| Interface | Name | IP | Role |
|---|---|---|---|
| em0 | LAN | 192.168.91.10/24 | Management — web UI |
| em1 | WAN | DHCP via VMnet8 (192.168.91.x) | Internet |
| em2 | OPT1/LABLAN | 10.0.1.1/24 | Lab network — Kali, Ubuntu Server |
| vlan0.10 (on em2) | OPT2/VLAN10 | 10.0.10.1/24 | Workstation segment |
| vlan0.20 (on em2) | OPT3/VLAN20 | 10.0.20.1/24 | Server segment |
| vlan0.99 (on em2) | OPT4/VLAN99 | 10.0.99.1/24 | Attack segment — isolated |

> 📸 **Screenshot:**
> <img width="924" height="449" alt="6  Interfaces - Assignments" src="https://github.com/user-attachments/assets/8de85bf4-256c-4d0d-9092-16571c6b8816" />
 — Interfaces → Assignments showing all six interfaces

### VLAN Configuration — Complete

Three VLANs created on parent interface em2 following the IEEE 802.1Q standard, each assigned as its own OPNsense interface with a static gateway IP.

| VLAN Tag | Description | Interface | Subnet | DHCP |
|---|---|---|---|---|
| 10 | Workstations | OPT2 | 10.0.10.0/24 | Enabled, 10.0.10.100–200 |
| 20 | Servers | OPT3 | 10.0.20.0/24 | Enabled, 10.0.20.100–200 |
| 99 | Attack (isolated) | OPT4 | 10.0.99.0/24 | Disabled — static only |

> 📸 **Screenshot:**
> <img width="923" height="445" alt="5  802 1Q VLAN config" src="https://github.com/user-attachments/assets/31ea2344-3326-4061-bd83-06b98cf03a70" />
 — Interfaces → Devices showing VLAN10, VLAN20, VLAN99 created on em2
>
> 📸 **Screenshot:**
> <img width="1839" height="1777" alt="12  vlan10 interface configuration" src="https://github.com/user-attachments/assets/ce16d945-fde7-49cb-a079-759792fa8ded" />
 — VLAN10 (OPT2) interface configuration, 10.0.10.1/24
>
> 📸 **Screenshot:**
> <img width="1831" height="1778" alt="13  vlan20 interface configuration" src="https://github.com/user-attachments/assets/3d5885d9-8ed4-4d39-adfb-a6c7addacb17" />
 — VLAN20 (OPT3) interface configuration, 10.0.20.1/24
>
> 📸 **Screenshot:**
> <img width="1839" height="1777" alt="14  vlan99 interface configuration" src="https://github.com/user-attachments/assets/9ddc759d-dd4c-4d6d-9f62-04b6127feb30" />
 — VLAN99 (OPT4) interface configuration, 10.0.99.1/24

### DHCP Configuration — Kea DHCPv4

OPNsense 26.1.6 uses Kea DHCP rather than the legacy ISC DHCP server. Three subnets configured:

| Subnet | Description | Pool | Router (gateway) |
|---|---|---|---|
| 10.0.1.0/24 | LABLAN | 10.0.1.100–200 | 10.0.1.1 |
| 10.0.10.0/24 | VLAN10-Workstations | 10.0.10.100–200 | 10.0.10.1 |
| 10.0.20.0/24 | VLAN20-Servers | 10.0.20.100–200 | 10.0.20.1 |

> The "Auto collect option data" checkbox must be unchecked for each subnet before the Router (gateway) field becomes available.

> 📸 **Screenshot:**
> <img width="923" height="449" alt="7  DHCP subnets 2" src="https://github.com/user-attachments/assets/6c8a5681-f25c-421d-a1b7-9e3fdf828531" />
 — Kea DHCPv4 → Subnets showing all three subnets configured
>
> 📸 **Screenshot:**
> <img width="923" height="448" alt="15  Kea DHCPv4 settings service enabled" src="https://github.com/user-attachments/assets/25b4a087-1f86-4f10-8608-fb92ca617a93" />
 — Kea DHCPv4 → Settings showing service enabled and interfaces selected

### Firewall Rules — Implemented (Deny-by-Default)

All interfaces in OPNsense default to deny-all until explicit rules are added. Rules below implement the principle of least privilege.

**OPT1 (LABLAN):**

| Action | Source | Destination | Description |
|---|---|---|---|
| Pass | OPT1 net | any | Allow LABLAN outbound (required — new interfaces have no implicit allow) |

**VLAN10 (Workstations):**

| Action | Protocol | Source | Destination | Port | Description |
|---|---|---|---|---|---|
| Pass | TCP | VLAN10 net | VLAN20 net | 80 | Workstations to servers — HTTP |
| Pass | TCP | VLAN10 net | VLAN20 net | 443 | Workstations to servers — HTTPS |
| Pass | TCP | VLAN10 net | VLAN20 net | 22 | Workstations to servers — SSH |
| Block | any | VLAN10 net | VLAN99 net | — | Block workstations from attack VLAN |
| Block | any | any | any | — | Default deny — VLAN10 |

**VLAN20 (Servers):**

| Action | Protocol | Source | Destination | Description |
|---|---|---|---|---|
| Pass | any | VLAN20 net | any | Servers outbound access |
| Block | any | any | any | Default deny — VLAN20 |

**VLAN99 (Attack — isolated):**

| Action | Protocol | Source | Destination | Description |
|---|---|---|---|---|
| Pass | any | VLAN99 net | VLAN20 net | Attack VLAN to servers |
| Block | any | VLAN99 net | WAN net | Block attack VLAN from internet |
| Block | any | VLAN99 net | LAN net | Block attack VLAN from management |
| Block | any | any | any | Default deny — VLAN99 |

> 📸 **Screenshot:**
> <img width="923" height="445" alt="16  firewall rules opt1" src="https://github.com/user-attachments/assets/e02f9739-2977-4e42-84dc-639b7980bc40" />
 — Firewall → Rules → OPT1
>
> 📸 **Screenshot:**
> <img width="920" height="443" alt="8  VLAN10 firewall rules" src="https://github.com/user-attachments/assets/5f02380f-372d-4d1b-88df-16d4fbaf3cf6" />
 — Firewall → Rules → VLAN10
>
> 📸 **Screenshot:**
> <img width="917" height="442" alt="9  VLAN20 firewall rules" src="https://github.com/user-attachments/assets/67bd95a7-e4f8-4712-b7fa-019dacc7515f" />
 — Firewall → Rules → VLAN20
>
> 📸 **Screenshot:**
> <img width="920" height="446" alt="10  VLAN99 firewall rules" src="https://github.com/user-attachments/assets/14932dae-7477-4c0e-8ba8-9b29e4e4d781" />
 — Firewall → Rules → VLAN99

### Hardening Steps Completed

- Default OPNsense root password changed during installation
- Admin password changed via setup wizard
- Web UI moved from HTTP to HTTPS on port 8443 (`https://192.168.91.10:8443`) — self-signed certificate, browser shows "not secure" as expected for a lab environment
- Deny-by-default firewall posture confirmed on every interface (OPT1, VLAN10, VLAN20, VLAN99)
- VLAN99 explicitly blocked from WAN and from the management network

> 📸 **Screenshot:**
> <img width="923" height="445" alt="17  showing https" src="https://github.com/user-attachments/assets/88faf193-125b-42ab-8eab-c4f66ef4c243" />
 — System → Settings → Administration showing HTTPS / port 8443
>
> 📸 **Screenshot:**
> <img width="955" height="476" alt="18  opnsense dashboard" src="https://github.com/user-attachments/assets/dc7410fb-f0dc-4a8a-b7ce-c9b5552e858c" />
 — OPNsense dashboard accessed via https://192.168.91.2:8443

---

## Skills Demonstrated

### CompTIA Network+ (N10-009)

| Objective | Implementation |
|---|---|
| 1.4 — IP addressing and subnetting | Designed and implemented /24 subnets across 6 network segments |
| 1.6 — Routing concepts | Configured inter-VLAN routing via OPNsense; diagnosed a real routing conflict between WAN and LAN subnets |
| 1.8 — Network topologies | Built a segmented topology with a central firewall and three VLANs |
| 2.1 — Switching concepts | Implemented 802.1Q VLAN tagging — three VLANs created on a single parent interface |
| 2.4 — Virtualization | Deployed multi-VM lab (firewall, attack host, server) using VMware Workstation Pro |
| 4.1 — Network troubleshooting | Diagnosed DHCP gateway conflicts, duplicate-address detection, and double-NAT routing issues using ifconfig, netstat, pfctl, and tcpdump |

### CompTIA Security+ (SY0-701)

| Objective | Implementation |
|---|---|
| 2.1 — Threat vectors | Isolated attack segment (VLAN 99) with explicit blocks to WAN and management |
| 2.4 — Network security | Implemented deny-by-default firewall rules and ACL segmentation across four interfaces |
| 3.2 — Vulnerability management | Documented attack surface, open ports (80/443/22), and access control decisions |
| 4.1 — Security controls | Changed default credentials, enforced HTTPS for management access |
| 4.6 — Identity and access | Applied principle of least privilege — only required ports allowed between segments |

---

## Challenges and Troubleshooting

This section documents every significant issue encountered during lab setup. Systematic troubleshooting under ambiguity is a core SOC analyst skill — these entries reflect real diagnostic methodology applied throughout the build process.

| Issue | Root Cause | Resolution |
|---|---|---|
| pfSense CE ISO unavailable | Netgate removed Community Edition direct downloads | Migrated to OPNsense 26.1.6 — functionally equivalent, fully open source |
| 7-Zip extraction error ("Can't find the specified file", 0 byte file) | Incomplete/corrupted download, or wrong extraction tool selected | Re-downloaded and re-extracted; verified file size before extracting |
| OPNsense installer RAM error | VM allocated only 512 MB — installer requires minimum 2 GB to copy live filesystem | Temporarily increased VM RAM to 2 GB for installation only, reduced to 512 MB (later 1 GB for UI responsiveness) after |
| Browser could not reach OPNsense at 192.168.100.2 (VMnet1) | Windows routed traffic via Wi-Fi instead of VMnet1, despite correct IP and routes on VMnet1 | Switched OPNsense Adapter 2 from VMnet1 to VMnet8 (NAT) and set LAN IP to 192.168.91.2 — web UI became accessible |
| Web UI became unreachable again after VLAN/DHCP work | OPNsense LAN IP (192.168.91.2) collided with VMware's own NAT gateway address on VMnet8 | Changed LAN IP to 192.168.91.10, a non-conflicting address in the same /24 |
| OPNsense → 8.8.8.8 returns "time to live exceeded" | WAN (em1) and LAN (em0) both ended up on 192.168.91.0/24 (VMnet8) — default route pointed out em0 instead of em1 | Root cause identified; attempted fix (moving LAN to VMnet1) caused further issues — reverted to last good snapshot |
| New OPT1 interface had zero connectivity to LABLAN | OPNsense denies all traffic on new interfaces by default — no rule existed for OPT1 | Added explicit "Pass, OPT1 net → any" rule |
| Kea DHCP — no "Router" field visible when adding a subnet | "Auto collect option data" checkbox was enabled, hiding manual gateway field | Unchecked "Auto collect option data" to reveal Router (gateway) field |
| Kea DHCP service play button greyed out | Service was already running — stop/restart buttons were active instead | Confirmed via Subnets tab that configuration was active; no action needed |
| Ubuntu Server login failing repeatedly after install | Caps Lock / typo during password entry at login (password not visible while typing) | Reset password via recovery mode after confirming username was correct |
| `apt install apache2` appeared to succeed but `systemctl status apache2` showed "unit not found" | Ubuntu Server had no internet access (downstream of the WAN/LAN subnet conflict above) — package never actually downloaded | Pending — requires WAN routing fix to resolve |
| WAN (em1) lost its IP after adapter change + reboot; `dhclient em1` obtained then immediately deleted 192.168.91.128 | Duplicate Address Detection — VMware's virtual switch caused OPNsense to see its own ARP reply as a conflict | Manual `ifconfig`/`route add` workarounds caused further instability and loss of web UI access — reverted to snapshot `opnsense-configured-vlans-firewall` |

> Every issue above was diagnosed using a structured approach: identify symptoms → isolate the layer (physical/data link/network/application) → form and test a hypothesis → document the outcome. This mirrors the alert triage methodology used in real SOC environments.

> 📸 **Screenshot:** `screenshots/connectivity-tests/01-ttl-exceeded.png` — Ping showing "time to live exceeded" — WAN/LAN subnet conflict evidence
>
> 📸 **Screenshot:** `screenshots/connectivity-tests/02-tcpdump-wan.png` — tcpdump on em1 showing un-NAT'd source addresses
>
> 📸 **Screenshot:** `screenshots/connectivity-tests/03-snapshot-revert.png` — VMware snapshot list showing revert to "opnsense-configured-vlans-firewall"

---

## Lessons Learned

- **Always check Hyper-V first** when VMware networking fails on Windows — disable it via `bcdedit /set hypervisorlaunchtype off` before installing any hypervisor
- **VMware Workstation host-only networks (VMnet) can behave unpredictably with Windows routing** — even with correct IPs and routes, Windows may continue sending traffic via Wi-Fi. NAT-based VMnet8 proved more reliable for management access on this hardware
- **Never place WAN and LAN interfaces on the same subnet** — even if both are technically reachable, the routing table cannot distinguish which interface should carry default (internet) traffic, causing TTL-exceeded loops
- **VMware's NAT network (VMnet8) reserves specific addresses** (typically `.1` for the host adapter and `.2` for VMware's virtual NAT router) — assigning a guest interface to one of these addresses causes silent conflicts
- **OPNsense denies all traffic on newly created interfaces by default** — this is correct security behavior, but it means every new interface (including OPT1/LABLAN) needs at least one explicit Pass rule before VMs on that segment can communicate at all
- **Kea DHCP in OPNsense 26.1.6 hides the gateway/router field** behind "Auto collect option data" — uncheck it to manually specify a router per subnet
- **VMware Workstation host-only networks are flat L2 segments** and do not natively perform 802.1Q tagging between guests and the virtual switch. OPNsense's VLAN interfaces (10, 20, 99) are correctly configured and firewalled, but lab VMs connected to VMnet2 currently sit on the untagged LABLAN subnet rather than inside a tagged VLAN — assigning VMs to specific VLANs will require additional configuration (see Next Steps)
- **Snapshots are essential before any networking change** — reverting to `opnsense-configured-vlans-firewall` preserved a full day of VLAN and firewall configuration work when WAN troubleshooting went sideways
- **Documenting failed attempts is as valuable as documenting successes** — the troubleshooting section of this README demonstrates the diagnostic thinking that SOC analysts apply daily

---

## Screenshot Checklist

All screenshots referenced inline above are summarized here, organized by repo folder. Capture these as each item is completed.

**`screenshots/opnsense/`**
- [X] `00-vmware-vm-list.png` — VMware library: OPNsense, Kali, Ubuntu-Server
- [X] `02-interfaces.png` — Interfaces → Assignments (all 6 interfaces)
- [X] `04-kea-subnets.png` — Kea DHCPv4 → Subnets
- [X] `05-kea-settings.png` — Kea DHCPv4 → Settings
- [X] `06-https-admin-settings.png` — Administration → HTTPS / port 8443
- [X] `07-dashboard-https.png` — Dashboard via https://192.168.91.10:8443

**`screenshots/vlans/`**
- [X] `01-vlan-devices.png` — Interfaces → Devices (VLAN10/20/99 on em2)
- [X] `02-vlan10-interface.png` — VLAN10 (OPT2) config
- [X] `03-vlan20-interface.png` — VLAN20 (OPT3) config
- [X] `04-vlan99-interface.png` — VLAN99 (OPT4) config

**`screenshots/firewall-rules/`**
- [X] `01-opt1-rules.png` — Firewall rules: OPT1
- [X] `02-vlan10-rules.png` — Firewall rules: VLAN10
- [X] `03-vlan20-rules.png` — Firewall rules: VLAN20
- [X] `04-vlan99-rules.png` — Firewall rules: VLAN99

**`screenshots/connectivity-tests/`**
- [ ] `01-ttl-exceeded.png` — TTL-exceeded ping (subnet conflict evidence)
- [ ] `02-tcpdump-wan.png` — tcpdump on em1
- [ ] `03-snapshot-revert.png` — Snapshot revert in VMware

---

## Status

- [x] VMware Workstation Pro 17.0 installed on new laptop
- [x] Hyper-V disabled
- [x] OPNsense 26.1.6 installed (2 GB RAM during install, reduced after)
- [x] OPNsense interfaces assigned (em0 = LAN, em1 = WAN, em2 = OPT1)
- [x] OPNsense web UI accessible via HTTPS (port 8443)
- [x] OPT1/LABLAN enabled at 10.0.1.1/24 with Kea DHCP
- [x] VLAN10, VLAN20, VLAN99 created and assigned as interfaces
- [x] Kea DHCP configured for LABLAN, VLAN10, VLAN20
- [x] Firewall rules implemented for OPT1, VLAN10, VLAN20, VLAN99 (deny-by-default)
- [x] Kali Linux installed on LABLAN (10.0.1.20), snapshot taken
- [x] Ubuntu Server installed on LABLAN (10.0.1.10), SSH installed
- [x] Snapshot `opnsense-configured-vlans-firewall` taken and used for recovery
- [ ] WAN/LAN subnet conflict resolved (move WAN to Bridged)
- [ ] Internet access confirmed from LABLAN, VLAN10, VLAN20
- [ ] Apache installed and running on Ubuntu Server
- [ ] Kali and Ubuntu Server assigned into correct VLANs (20 and 99 respectively)
- [ ] Connectivity and isolation tests documented with screenshots
- [ ] Wireshark installed and packet captures collected
- [ ] SIEM integration (Splunk Free or Security Onion)
- [ ] Windows 10 Enterprise VM built on new laptop

---

## Next Steps

1. **Resolve WAN/LAN subnet conflict** — change OPNsense Network Adapter 1 (WAN) from NAT (VMnet8) to Bridged, giving WAN a real IP from the home router on a separate subnet from LAN (VMnet8, 192.168.91.x) and the lab network (VMnet2, 10.0.1.x)
2. Once internet access is confirmed, install and verify Apache on Ubuntu Server
3. Decide on approach for assigning Kali and Ubuntu Server into VLAN99 and VLAN20 respectively, given VMware Workstation's flat host-only networking — likely either (a) tagged sub-interfaces within each guest OS, or (b) treating LABLAN as a flat management segment and documenting VLANs as a routing/firewall demonstration on OPNsense only
4. Run and document connectivity/isolation tests (ping, traceroute, nmap) proving VLAN99 isolation and VLAN10→VLAN20 access on allowed ports only
5. Install Wireshark and capture sample inter-VLAN traffic
6. Install SIEM (Splunk Free or Security Onion) for log ingestion and alerting
7. Build Windows 10 Enterprise Evaluation VM on the new laptop (organization mode, domain-ready)

---

## References

- CompTIA Network+ Exam Objectives N10-009
- CompTIA Security+ Exam Objectives SY0-701
- OPNsense Documentation — docs.opnsense.org
- VMware Workstation Pro Documentation — docs.vmware.com
