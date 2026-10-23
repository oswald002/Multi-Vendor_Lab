\# IP Addressing Plan



Covers Phase 1 (HQ campus) and Phase 2 (multi-vendor WAN). Phase 3 entries are planned and not yet built.



\## Autonomous systems



| AS | Site | Devices | Notes |

|----|------|---------|-------|

| 65000 | ISP | MikroTik CHR | Transit provider |

| 65001 | HQ | R1, R2 | eBGP to ISP, iBGP between R1 and R2 |

| 65002 | Branch A (planned) | FortiGate | Phase 3 |

| 65003 | Branch B | Aruba AOS-CX | eBGP to ISP |



All AS numbers are from the private range (RFC 6996).



\## HQ VLANs



| VLAN | Name | Subnet | HSRP VIP | R1 | R2 | HSRP active |

|------|------|--------|----------|----|----|-------------|

| 10 | USERS | 10.1.10.0/24 | 10.1.10.1 | .2 | .3 | R1 |

| 20 | SERVERS | 10.1.20.0/24 | 10.1.20.1 | .2 | .3 | R2 |

| 30 | MGMT | 10.1.30.0/24 | 10.1.30.1 | .2 | .3 | R1 |

| 99 | NATIVE\_UNUSED | none | none | none | none | n/a |



SW1 is the STP root for VLANs 10 and 30 and SW2 for VLAN 20, matching the HSRP active routers.



\## Loopbacks



| Device | Interface | Address | Purpose |

|--------|-----------|---------|---------|

| R1 | Loopback0 | 10.255.0.1/32 | OSPF and BGP router ID, iBGP source |

| R2 | Loopback0 | 10.255.0.2/32 | OSPF and BGP router ID, iBGP source |

| Aruba CX | loopback 0 | 10.255.0.3/32 | BGP router ID |

| ISP | lo0 (bridge) | 198.51.100.1/32 | Simulated internet address |



198.51.100.0/24 is a documentation range (RFC 5737), used here to stand in for an internet destination.



\## Point-to-point links



| Link | Subnet | Side A | Side B |

|------|--------|--------|--------|

| R1 to R2 (OSPF) | 172.16.0.0/30 | R1 Fa1/0 = .1 | R2 Fa1/0 = .2 |

| R1 to ISP | 172.16.1.0/30 | R1 Fa2/0 = .1 | ISP ether1 = .2 |

| R2 to ISP | 172.16.2.0/30 | R2 Fa2/0 = .1 | ISP ether2 = .2 |

| Aruba CX to ISP | 172.16.3.0/30 | Aruba 1/1/1 = .1 | ISP ether3 = .2 |



\## Branch sites



| Site | Subnet | Gateway | Hosts |

|------|--------|---------|-------|

| Branch B | 10.3.0.0/24 | 10.3.0.1 (Aruba VLAN 10) | PC3 = 10.3.0.10 |

| Branch A (planned) | 10.2.0.0/24 | to be assigned | to be assigned |



\## End hosts



| Host | Address | Gateway | VLAN / site |

|------|---------|---------|-------------|

| PC1 | 10.1.10.10/24 | 10.1.10.1 | VLAN 10 |

| PC2 | 10.1.20.10/24 | 10.1.20.1 | VLAN 20 |

| PC3 | 10.3.0.10/24 | 10.3.0.1 | Branch B |



\## Physical cabling



\### HQ campus



| Link | Port A | Port B |

|------|--------|--------|

| Port-channel 1 (LACP) | SW1 Gi0/0 and Gi0/1 | SW2 Gi0/0 and Gi0/1 |

| R1 trunk | R1 Fa0/0 | SW1 Gi0/2 |

| R2 trunk | R2 Fa0/0 | SW2 Gi0/2 |

| PC1 | PC1 | SW1 Gi0/3 (VLAN 10) |

| PC2 | PC2 | SW2 Gi0/3 (VLAN 20) |



\### Branch B



| GNS3 port (Aruba) | Aruba interface | Connected to |

|-------------------|-----------------|--------------|

| Ethernet0 | management | not connected |

| Ethernet1 | 1/1/1 | ISP ether3 |

| Ethernet2 | 1/1/2 | PC3 |



\## Routing design



| Function | Where | Detail |

|----------|-------|--------|

| OSPF area 0 | R1, R2 | Internal HQ routing and loopback reachability for iBGP. User-facing subinterfaces are passive. |

| eBGP | R1, R2, Aruba to ISP | ISP is AS 65000 |

| iBGP | R1 to R2 | Between loopbacks, `next-hop-self` on both |

| Summary origination | R1, R2 | 10.1.0.0/16 advertised, backed by a Null0 route |

| Outbound filter | R1, R2 | Prefix-list HQ-OUT permits only 10.1.0.0/16 |

| Primary exit | R1 | Local-preference 200 on routes from the ISP |

| Inbound preference | R2 | AS-path prepend (65001 twice) so the ISP prefers R1 |

| BGP timers (HQ to ISP) | R1, R2, ISP | Keepalive 10 s, hold time 30 s |

| BGP timers (Branch B to ISP) | Aruba, ISP | Defaults (keepalive 60 s, hold time 180 s) |

