# soc-home-lab
# SOC Home Lab — Network Topology & Segmentation

## Objective

This project documents the design and implementation of a virtualized home network lab built to develop and demonstrate skills aligned with CompTIA Network+ (N10-009) and Security+ (SY0-701) certifications. The lab simulates a segmented enterprise network environment using free and open-source tools, providing hands-on experience with routing, VLANs, firewall configuration, and network security fundamentals relevant to a Security Operations Center (SOC) analyst role.

---

## Lab Environment

| Component | Details |
|---|---|
| Host OS | Windows 10/11 |
| Hypervisor | Oracle VirtualBox 7.1.10 |
| Router / Firewall | OPNsense (pfSense CE equivalent) |
| Attack VM | Kali Linux |
| Server Target | Ubuntu Server 22.04 LTS |
| Workstation Target | Windows 10 Enterprise (Evaluation) |

---

## Network Topology

```
[ Internet ]
      |
  [ WAN — NAT Adapter ]
      |
  [ OPNsense Firewall / Router ]
      |
  ┌───────────────────────────────────┐
  │                                   │
[ Management ]     [ Lab LAN — labnet1 Internal Network ]
192.168.100.0/24        |
(Host-only adapter)     |
                   ┌────┴─────────────┐
                   │                  │
             [ VLAN 10 ]        [ VLAN 20 ]
           Workstations          Servers
          10.0.10.0/24        10.0.20.0/24

                        [ VLAN 99 ]
                     Attack (Isolated)
                       10.0.99.0/24
                    (No internet access)
```

---

## IP Addressing Scheme

| Network | Subnet | Gateway | Purpose |
|---|---|---|---|
| Management | 192.168.100.0/24 | 192.168.100.2 | Host PC to firewall access |
| Lab LAN | 10.0.1.0/24 | 10.0.1.1 | General VM network |
| VLAN 10 | 10.0.10.0/24 | 10.0.10.1 | Workstation segment |
| VLAN 20 | 10.0.20.0/24 | 10.0.20.1 | Server segment |
| VLAN 99 | 10.0.99.0/24 | 10.0.99.1 | Attack VM — isolated, no WAN |

---

## Tools Used

| Tool | Purpose | Cost |
|---|---|---|
| VirtualBox 7.1.10 | Hypervisor / VM management | Free |
| OPNsense | Firewall, router, VLAN management | Free / open source |
| Kali Linux | Attack simulation, packet analysis | Free |
| Ubuntu Server 22.04 | Target server (SSH, Apache) | Free |
| Windows 10 Enterprise Eval | Target workstation | Free (90-day eval) |
| Wireshark | Packet capture and analysis | Free |

---

## Key Configurations

### VirtualBox Network Adapters (OPNsense VM)

| Adapter | Type | Role |
|---|---|---|
| Adapter 1 | NAT | WAN — internet access |
| Adapter 2 | Host-only (192.168.100.1) | LAN — management access from host PC |
| Adapter 3 | Internal Network (labnet1) | Lab LAN — VM-to-VM communication |

### VLAN Configuration

Three VLANs configured on the OPNsense OPT1 interface to segment lab traffic by function. VLAN tagging follows IEEE 802.1Q standard.

- VLAN 10 — Workstations: DHCP enabled, range 10.0.10.100–200
- VLAN 20 — Servers: DHCP enabled, range 10.0.20.100–200
- VLAN 99 — Attack segment: Static IPs only, no DHCP, WAN access blocked

### Firewall Rules (Deny-by-Default)

Inter-VLAN traffic is controlled by explicit firewall rules implementing the principle of least privilege. A default deny rule sits at the bottom of each interface rule set.

| Source | Destination | Action | Justification |
|---|---|---|---|
| VLAN 10 | VLAN 20 (TCP 80, 443, 22) | Allow | Workstations access servers |
| VLAN 10 | VLAN 99 | Block | Isolate attack segment |
| VLAN 99 | VLAN 20 | Allow | Attack VM reaches targets |
| VLAN 99 | WAN | Block | Prevent internet access from attack VM |
| Any | 192.168.100.0/24 | Block | Protect management plane |
| Any | Any | Block | Default deny — implicit |

### Hardening Steps Applied

- Default admin credentials changed immediately after setup
- Web UI moved from HTTP to HTTPS on non-default port 8443
- Login attempt lockout configured (3 attempts, 5-minute lockout)
- Default deny firewall rule applied to all interfaces
- VirtualBox snapshots taken at clean baseline state

---

## Skills Demonstrated

### CompTIA Network+ (N10-009)

| Objective | Implementation |
|---|---|
| 1.4 — IP addressing and subnetting | Designed /24 subnets across 5 network segments |
| 1.6 — Routing concepts | Configured inter-VLAN routing via OPNsense |
| 1.8 — Network topologies | Built a segmented star topology with a central firewall |
| 2.1 — Switching concepts | Implemented 802.1Q VLAN tagging |
| 2.4 — Virtualization | Deployed all infrastructure using VirtualBox VMs |
| 4.1 — Network troubleshooting | Diagnosed and resolved multiple VirtualBox adapter and boot issues |

### CompTIA Security+ (SY0-701)

| Objective | Implementation |
|---|---|
| 2.1 — Threat vectors | Isolated attack VM (VLAN 99) from production and internet segments |
| 2.4 — Network security | Applied deny-by-default firewall rules, ACL segmentation |
| 3.2 — Vulnerability management | Documented attack surface and access control decisions |
| 4.1 — Security controls | Changed default credentials, restricted management plane access |
| 4.6 — Identity and access | Implemented principle of least privilege across firewall rule sets |

---

## Challenges and Troubleshooting

| Issue | Root Cause | Resolution |
|---|---|---|
| "CPU doesn't support long mode" error | VM configured as 32-bit instead of 64-bit | Changed VM type to FreeBSD 64-bit in VirtualBox settings |
| Kernel panic on boot (APIC error) | I/O APIC not enabled in VM settings | Enabled I/O APIC under Settings → System → Motherboard |
| "No valid storage devices detected" | Storage controller incompatibility with installer | Switched from IDE/AHCI to NVMe controller for disk, IDE for optical drive |
| pfSense CE ISO unavailable | Netgate removed CE direct downloads | Migrated to OPNsense — functionally equivalent, fully open source |

> Documenting these troubleshooting steps demonstrates a core SOC analyst skill: systematic diagnosis, root cause identification, and resolution under ambiguity.

---

## Lessons Learned

- VirtualBox 7.1 moved several key menus compared to earlier versions — Host Network Manager is now under File → Tools, and snapshots are accessed via the VM hamburger menu in the sidebar
- OPNsense is a production-viable alternative to pfSense CE with a cleaner web UI and identical feature set for lab purposes
- Deny-by-default firewall rules require careful ordering — rules are evaluated top to bottom and the first match wins
- Taking VM snapshots before any major configuration change is critical — it saved significant setup time when recovering from boot issues

---

## Status

- [x] VirtualBox installed and configured
- [x] VM network adapters planned and configured
- [x] Internal network (labnet1) created
- [ ] OPNsense installation complete
- [ ] VLAN configuration complete
- [ ] Firewall rules implemented and tested
- [ ] Kali Linux VM configured
- [ ] Ubuntu Server VM configured
- [ ] Windows 10 VM configured
- [ ] Connectivity tests documented

---

## Next Steps

- Complete OPNsense installation and VLAN configuration
- Install and configure SIEM (Splunk Free or Security Onion) for log ingestion
- Run initial vulnerability scan using Nessus Essentials against Ubuntu Server target
- Capture and analyze lab traffic using Wireshark
- Document attack simulation from VLAN 99 and corresponding SIEM alerts

---

## References

- CompTIA Network+ Exam Objectives N10-009
- CompTIA Security+ Exam Objectives SY0-701
- OPNsense Documentation — docs.opnsense.org
- VirtualBox User Manual — virtualbox.org/manual
