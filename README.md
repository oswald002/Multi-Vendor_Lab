# Multi-Vendor Enterprise Network Lab

A GNS3 lab that simulates an enterprise network spanning Cisco, MikroTik and Aruba equipment, built to demonstrate hands-on routing, switching and security skills.

## Skills demonstrated

- VLANs, rapid spanning tree and LACP EtherChannel
- First-hop redundancy with HSRP, load-balanced across VLANs
- OSPF with passive-interface hardening
- eBGP and iBGP, with local preference, AS-path prepending and prefix-list filtering
- Route summarisation with a Null0 discard route
- BGP timer tuning and measured failover
- Multi-vendor interoperability (Cisco IOS, MikroTik RouterOS, Aruba AOS-CX)
- Structured documentation and troubleshooting records
- Planned: ASA firewalling, NAT, DMZ, cross-vendor site-to-site IPsec VPN

## Topology

![Phase 1 topology](diagrams/phase1-topology.png)

![Phase 2 topology](diagrams/phase2-topology.png)

## Design summary

| Site | Devices | AS | Role |
|------|---------|----|------|
| HQ | R1, R2 (Cisco c7200), SW1, SW2 (Cisco vIOS-L2) | 65001 | Redundant campus with HSRP gateways and a dual-homed edge |
| ISP | MikroTik CHR | 65000 | Transit provider |
| Branch B | Aruba AOS-CX switch | 65003 | Layer 3 branch peering with the ISP over eBGP |

Key design points:

- HSRP and spanning-tree roots are aligned so that Layer 2 and Layer 3 paths agree. R1 and SW1 serve VLANs 10 and 30, and R2 and SW2 serve VLAN 20.
- R1 is the primary exit, using local preference 200 on routes learned from the ISP.
- R2 prepends its AS path outbound, so the ISP prefers R1 for inbound traffic.
- HQ advertises only its 10.1.0.0/16 summary, enforced with a prefix-list.

## Phases

- [x] **Phase 1:** HQ campus (VLANs, RSTP, EtherChannel, HSRP, OSPF)
- [x] **Phase 2:** multi-vendor WAN (MikroTik ISP, eBGP, Aruba branch, failover test)
- [ ] **Phase 3:** security and VPN (ASAv perimeter, FortiGate branch, IPsec VPN)
- [ ] **Phase 4 (optional):** Layer 2 hardening, secure management, config backup automation

## Results at a glance

- All eBGP and iBGP sessions reach Established.
- Traffic from every PC reaches every other PC, with traces confirming the intended paths.
- With a 30-second BGP hold time, failover from R1 to R2 took about 30 seconds, and traffic returned to R1 after the link was restored.

## Documentation

| Document | Contents |
|----------|----------|
| [IP addressing plan](docs/ip-addressing-plan.md) | VLANs, loopbacks, links, AS numbers, cabling and routing design |
| [Phase 1 verification](docs/phase1-campus-verification.md) | Evidence for the HQ campus build |
| [Phase 2 verification](docs/phase2-wan-verification.md) | Evidence for the BGP WAN and failover test |
| [Troubleshooting log](docs/troubleshooting-log.md) | Problems met, investigation and fixes |
| [Device configurations](configs/) | Sanitised configuration of every device |

## Tools

GNS3 with the GNS3 VM, VPCS, Cisco IOS (c7200), Cisco vIOS-L2, MikroTik RouterOS, ArubaOS-CX.

Vendor software images and license files are not included in this repository. Configurations have had passwords, hashes and keys removed.


## Certifications
CCNA(Sept, 2026), Network+(Feb, 2026), Security+(Mar, 2026), AWS(Mar, 2025) 