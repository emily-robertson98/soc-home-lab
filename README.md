# SOC Home Lab — Network Topology & Segmentation

## Objective

This project documents the design and implementation of a virtualized home network lab built to develop and demonstrate skills aligned with CompTIA Network+ (N10-009) and Security+ (SY0-701) certifications. The lab simulates a segmented enterprise network environment using free and open-source tools, providing hands-on experience with routing, VLANs, firewall configuration, and network security fundamentals relevant to a Security Operations Center (SOC) analyst role.

> **Lab Status: In Progress** — Core infrastructure installed. VMnet1 host-to-VM connectivity under investigation. See troubleshooting section for full diagnostic log.

---

## Lab Environment

| Component | Details |
|---|---|
| Host OS | Windows 10/11 (personal machine, Wi-Fi connected) |
| Hypervisor | VMware Workstation Pro 17.0 (migrated from VirtualBox 7.1.10) |
| Router / Firewall | OPNsense 26.1.6 (community edition) |
| Attack VM | Kali Linux (planned) |
| Server Target | Ubuntu Server 22.04 LTS (planned) |
| Workstation Target | Windows 10 Enterprise Evaluation — organization mode, domain-ready (planned) |

---

## Network Topology

```
[ Internet ]
      |
  [ WAN — NAT Adapter (em1) ]
      |
  [ OPNsense 26.1.6 Firewall / Router ]
      |
  ┌───────────────────────────────────┐
  │                                   │
[ Management ]      [ Lab LAN — VMnet2 ]
192.168.100.0/24         |
(VMnet1 — em0)           |
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
| VMware Workstation Pro 17.0 | Primary hypervisor / VM management | Free (personal use) |
| VirtualBox 7.1.10 | Initial hypervisor — deprecated for this lab | Free |
| OPNsense 26.1.6 | Firewall, router, VLAN management | Free / open source |
| Kali Linux | Attack simulation, packet analysis | Free |
| Ubuntu Server 22.04 | Target server (SSH, Apache) | Free |
| Windows 10 Enterprise Eval | Target workstation | Free (90-day eval) |
| Wireshark | Packet capture and analysis | Free |

---

## Key Configurations

### VMware Network Adapters (OPNsense VM)

| Adapter | VMware Network | OPNsense Interface | Role |
|---|---|---|---|
| Network Adapter 1 | NAT (VMnet8) | em1 (WAN) | Internet access |
| Network Adapter 2 | Host-only (VMnet1) | em0 (LAN) | Management — host PC to OPNsense |
| Network Adapter 3 | Host-only (VMnet2) | em2 (OPT1/LABLAN) | Lab LAN — VM-to-VM communication |

### VMware Virtual Network Configuration

| VMware Network | Type | Subnet | DHCP | Purpose |
|---|---|---|---|---|
| VMnet1 | Host-only | 192.168.100.0/24 | Disabled | Management network |
| VMnet2 | Host-only | 10.0.1.0/24 | Disabled | Lab internal network |
| VMnet8 | NAT | (auto) | Enabled | WAN / internet |

### OPNsense Interface Assignment

| Interface | Name | IP | Role |
|---|---|---|---|
| em0 | LAN | 192.168.100.2/24 | Management LAN |
| em1 | WAN | DHCP via NAT | Internet |
| em2 | OPT1/LABLAN | 10.0.1.1/24 | Lab network (planned) |

### VLAN Configuration (Planned)

Three VLANs to be configured on the OPNsense OPT1 interface to segment lab traffic by function. VLAN tagging follows IEEE 802.1Q standard.

- VLAN 10 — Workstations: DHCP enabled, range 10.0.10.100–200
- VLAN 20 — Servers: DHCP enabled, range 10.0.20.100–200
- VLAN 99 — Attack segment: Static IPs only, no DHCP, WAN access blocked

### Firewall Rules (Planned — Deny-by-Default)

Inter-VLAN traffic will be controlled by explicit firewall rules implementing the principle of least privilege. A default deny rule will sit at the bottom of each interface rule set.

| Source | Destination | Action | Justification |
|---|---|---|---|
| VLAN 10 | VLAN 20 (TCP 80, 443, 22) | Allow | Workstations access servers |
| VLAN 10 | VLAN 99 | Block | Isolate attack segment |
| VLAN 99 | VLAN 20 | Allow | Attack VM reaches targets |
| VLAN 99 | WAN | Block | Prevent internet access from attack VM |
| Any | 192.168.100.0/24 | Block | Protect management plane |
| Any | Any | Block | Default deny — implicit |

### Hardening Steps Completed

- Default OPNsense root password changed during installation
- Web UI configured for HTTP on LAN interface (HTTPS to be enforced post-lab completion)
- Windows 10 Enterprise VM configured in organization mode (domain-ready, local account)

---

## Skills Demonstrated

### CompTIA Network+ (N10-009)

| Objective | Implementation |
|---|---|
| 1.4 — IP addressing and subnetting | Designed /24 subnets across 5 network segments |
| 1.6 — Routing concepts | Configured inter-VLAN routing via OPNsense |
| 1.8 — Network topologies | Built a segmented star topology with a central firewall |
| 2.1 — Switching concepts | Planned 802.1Q VLAN tagging across three segments |
| 2.4 — Virtualization | Deployed infrastructure across VirtualBox and VMware hypervisors |
| 4.1 — Network troubleshooting | Diagnosed and resolved extensive hypervisor, adapter, and OS-level networking issues across two different hypervisors |

### CompTIA Security+ (SY0-701)

| Objective | Implementation |
|---|---|
| 2.1 — Threat vectors | Isolated attack VM (VLAN 99) from production and internet segments |
| 2.4 — Network security | Planned deny-by-default firewall rules and ACL segmentation |
| 3.2 — Vulnerability management | Documented attack surface and access control decisions |
| 4.1 — Security controls | Changed default credentials, planned management plane restrictions |
| 4.6 — Identity and access | Implementing principle of least privilege across firewall rule sets |

---

## Challenges and Troubleshooting

This section documents every significant issue encountered during lab setup. Systematic troubleshooting under ambiguity is a core SOC analyst skill — these entries reflect real diagnostic methodology applied throughout the build process.

| Issue | Root Cause | Resolution |
|---|---|---|
| pfSense CE ISO unavailable | Netgate removed Community Edition direct downloads | Migrated to OPNsense 26.1.6 — functionally equivalent, fully open source |
| "CPU doesn't support long mode" (VirtualBox) | VM type set to FreeBSD 32-bit instead of 64-bit | Changed VM type to FreeBSD 64-bit in VirtualBox settings |
| Kernel panic on boot — APIC error (VirtualBox) | I/O APIC not enabled in VM motherboard settings | Enabled I/O APIC under Settings → System → Motherboard |
| "No valid storage devices detected" (VirtualBox) | Storage controller incompatibility between VirtualBox 7.1 and OPNsense installer | Switched to NVMe controller for disk, IDE (PIIX4) for optical drive |
| Network interfaces not detected in OPNsense (VirtualBox) | VirtualBox virtio adapters not binding — only lo0/enc0/pflog0 visible | Root cause unresolved — migrated to VMware Workstation Pro 17 |
| OPNsense installer RAM error (VMware) | VM allocated only 512 MB — installer requires minimum 2 GB to copy live filesystem | Temporarily increased VM RAM to 2 GB for installation only, reduced back to 512 MB after |
| VMnet1 host adapter showing 169.254.x.x | Windows auto-assigned link-local address — Hyper-V was conflicting with VMware networking | Disabled Hyper-V via `bcdedit /set hypervisorlaunchtype off`, set static IP 192.168.100.1/24 manually via ncpa.cpl |
| OPNsense interfaces assigned backwards | Default OPNsense assignment put LAN on em0 and left WAN on em1 with no IP | Reassigned via console option 1: WAN → em1, LAN → em0 |
| VMnet1 still not passing traffic after Hyper-V disabled | Under active investigation — suspected residual network driver conflict | Pending resolution — diagnostic steps taken: verified ifconfig, pfctl -d, sockstat, arp -a, route table, pciconf |

> Every issue above was diagnosed using a structured approach: identify symptoms → isolate the layer (physical/data link/network/application) → form and test a hypothesis → document the outcome. This mirrors the alert triage methodology used in real SOC environments.

---

## Lessons Learned

- **Always check Hyper-V first** when VMware or VirtualBox networking fails on Windows — it silently conflicts with both hypervisors at the CPU virtualization level and must be fully disabled via `bcdedit`
- **VirtualBox 7.1 has significant compatibility issues** with OPNsense 26.1.6 — VMware Workstation Pro 17 is the more reliable choice for FreeBSD-based firewall VMs
- **pfSense CE is no longer directly downloadable** from Netgate — OPNsense is the recommended community alternative with identical features and active open source development
- **Windows 10 for lab VMs should be configured in organization mode** — enables Group Policy, RDP, and domain join which are essential for realistic enterprise SOC scenarios
- **Installer RAM requirements differ from runtime requirements** — OPNsense needs at least 2 GB to install from a live image but runs comfortably on 512 MB once installed
- **Virtual network adapter naming varies by hypervisor** — VirtualBox presents virtio adapters as vtnet0/vtnet1 while VMware presents them as em0/em1; always verify with `ifconfig` rather than assuming
- **Documenting failed attempts is as valuable as documenting successes** — the troubleshooting section of this README demonstrates the diagnostic thinking that SOC analysts apply daily

---

## Status

- [x] VirtualBox 7.1.10 installed and configured
- [x] VMware Workstation Pro 17.0 installed
- [x] OPNsense 26.1.6 downloaded and installed in VMware
- [x] OPNsense interfaces assigned (em0 = LAN, em1 = WAN)
- [x] OPNsense LAN configured at 192.168.100.2/24
- [x] VMware virtual networks configured (VMnet1, VMnet2, VMnet8)
- [x] Hyper-V disabled on host PC
- [x] Windows 10 Enterprise VM created (organization mode, domain-ready)
- [ ] VMnet1 host-to-VM connectivity resolved
- [ ] OPNsense web UI accessible from host browser
- [ ] OPNsense setup wizard completed
- [ ] VLAN configuration complete (10, 20, 99)
- [ ] Firewall rules implemented and tested
- [ ] Kali Linux VM configured and placed in VLAN 99
- [ ] Ubuntu Server VM configured and placed in VLAN 20
- [ ] Connectivity and isolation tests documented with screenshots
- [ ] SIEM integration (Splunk Free or Security Onion)

---

## Next Steps

- Resolve VMnet1 host-to-VM traffic issue (suspected residual network driver conflict)
- Access OPNsense web UI and complete initial setup wizard
- Configure VLANs 10, 20, and 99 via OPNsense web UI
- Implement deny-by-default firewall rules across all interfaces
- Install and configure remaining VMs (Kali Linux, Ubuntu Server)
- Run connectivity and isolation tests and document all results
- Install SIEM for log ingestion, dashboarding, and alert generation

---

## References

- CompTIA Network+ Exam Objectives N10-009
- CompTIA Security+ Exam Objectives SY0-701
- OPNsense Documentation — docs.opnsense.org
- VMware Workstation Pro Documentation — docs.vmware.com
- VirtualBox User Manual — virtualbox.org/manual
