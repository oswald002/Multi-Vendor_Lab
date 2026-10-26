# Multi-Vendor Enterprise Network Lab

A GNS3 lab that simulates an enterprise network spanning Cisco, MikroTik and Aruba equipment, built to demonstrate hands-on routing, switching and security skills.

## Skills demonstrated

- VLANs, rapid spanning tree and LACP EtherChannel
- First-hop redundancy with HSRP, load-balanced across VLANs
- OSPF with passive-interface hardening
- eBGP and iBGP, with local preference, AS-path prepending and prefix-list filtering
- Route summarisation with a Null0 discard route
- BGP timer tuning and measured failover
- Stateful firewalling with security levels (inside, DMZ, outside) on a Cisco ASAv
- NAT with a VPN exemption, and ACLs with logging
- Site-to-site IPsec VPN using IKEv2 with AES-256 and SHA-256, between Cisco ASA and MikroTik
- Packet-capture proof that traffic is encrypted on the WAN link
- Multi-vendor interoperability (Cisco IOS and ASA, MikroTik RouterOS, Aruba AOS-CX)
- Structured documentation and troubleshooting records

## Topology

### Phase 1: HQ campus

![Phase 1 topology](diagrams/phase1-topology.png)

### Phase 2: multi-vendor WAN

![Phase 2 topology](diagrams/phase2-topology.png)

### Phase 3: perimeter security and VPN

![Phase 3 topology](diagrams/phase3-topology.png)

## Design summary

| Site | Devices | AS | Role |
|------|---------|----|------|
| HQ campus | R1, R2 (Cisco c7200), SW1, SW2 (Cisco vIOS-L2) | 65001 | Redundant campus with HSRP gateways and a dual-homed edge |
| ISP | MikroTik CHR | 65000 | Transit provider |
| Branch B | Aruba AOS-CX switch | 65003 | Layer 3 branch peering with the ISP over eBGP |
| HQ firewall | Cisco ASAv | none (static default route) | Perimeter firewall for the HQ server and DMZ segments, and VPN endpoint |
| Branch A | MikroTik CHR | none (static default route) | Branch router and VPN endpoint |

Key design points:

- HSRP and spanning-tree roots are aligned so that Layer 2 and Layer 3 paths agree. R1 and SW1 serve VLANs 10 and 30, and R2 and SW2 serve VLAN 20.
- R1 is the primary exit, using local preference 200 on routes learned from the ISP.
- R2 prepends its AS path outbound, so the ISP prefers R1 for inbound traffic.
- HQ advertises only its 10.1.0.0/16 summary, enforced with a prefix-list.
- The ASAv separates an inside segment, a DMZ and the outside. The DMZ can reach the internet but is blocked from the inside network.
- An IKEv2 tunnel between the ASAv and Branch A protects traffic between 10.4.1.0/24 and 10.2.0.0/24, and that traffic is exempt from NAT.

## Phases

- [x] **Phase 1:** HQ campus (VLANs, RSTP, EtherChannel, HSRP, OSPF)
- [x] **Phase 2:** multi-vendor WAN (MikroTik ISP, eBGP, Aruba branch, failover test)
- [x] **Phase 3:** security and VPN (ASAv perimeter, NAT, DMZ policy, MikroTik branch, IKEv2 IPsec VPN)

## Results at a glance

- All eBGP and iBGP sessions reach Established.
- Traffic from every PC reaches every other PC, with traces confirming the intended paths.
- With a 30-second BGP hold time, failover from R1 to R2 took about 30 seconds, and traffic returned to R1 after the link was restored.
- The IPsec tunnel carries traffic in both directions, and a capture on the WAN link shows ESP packets with no readable ICMP.
- The DMZ policy blocks DMZ-to-inside traffic while allowing internet access, confirmed by ACL hit counts.

![ESP capture](diagrams/phase3-esp-capture.png)

## Documentation

| Document | Contents |
|----------|----------|
| [IP addressing plan](docs/ip-addressing-plan.md) | VLANs, loopbacks, links, AS numbers, cabling and routing design |
| [Phase 1 verification](docs/phase1-campus-verification.md) | Evidence for the HQ campus build |
| [Phase 2 verification](docs/phase2-wan-verification.md) | Evidence for the BGP WAN and failover test |
| [Phase 3 verification](docs/phase3-security-vpn-verification.md) | Evidence for the firewall, NAT, DMZ policy and VPN |
| [Troubleshooting log](docs/troubleshooting-log.md) | Problems met, investigation and fixes |
| [Device configurations](configs/) | Sanitised configuration of every device |

## Notes and limitations

- A FortiGate VM was tried as the Branch A router, but the unlicensed VM offers only DES for IPsec, so a MikroTik CHR was used instead. This is recorded in the troubleshooting log.
- The ASAv runs without a Smart License, which limits its throughput, so the tests demonstrate function rather than performance.
- The ASAv firewall sits beside the HQ campus, not in its traffic path. This kept the working Phase 1 and Phase 2 design intact.

## Tools

GNS3 with the GNS3 VM, VPCS, Cisco IOS (c7200), Cisco vIOS-L2, Cisco ASAv, MikroTik RouterOS, ArubaOS-CX.

Vendor software images and license files are not included in this repository. Configurations have had passwords, hashes and keys removed, including the VPN pre-shared key.


## Certifications
CCNA(Sept, 2026), Network+(Feb, 2026), Security+(Mar, 2026), AWS(Mar, 2025) 