# IP Addressing Plan

Covers Phase 1 (HQ campus), Phase 2 (multi-vendor WAN) and Phase 3 (perimeter security and VPN).

## Autonomous systems

| AS | Site | Devices | Notes |
|----|------|---------|-------|
| 65000 | ISP | MikroTik CHR | Transit provider |
| 65001 | HQ campus | R1, R2 | eBGP to ISP, iBGP between R1 and R2 |
| 65003 | Branch B | Aruba AOS-CX | eBGP to ISP |
| none | HQ firewall | Cisco ASAv | Static default route to the ISP |
| none | Branch A | MikroTik CHR | Static default route to the ISP |

All AS numbers are from the private range (RFC 6996). AS 65002 was reserved for Branch A in the original plan but is unused, because Branch A uses static routing.

## HQ campus VLANs

| VLAN | Name | Subnet | HSRP VIP | R1 | R2 | HSRP active |
|------|------|--------|----------|----|----|-------------|
| 10 | USERS | 10.1.10.0/24 | 10.1.10.1 | .2 | .3 | R1 |
| 20 | SERVERS | 10.1.20.0/24 | 10.1.20.1 | .2 | .3 | R2 |
| 30 | MGMT | 10.1.30.0/24 | 10.1.30.1 | .2 | .3 | R1 |
| 99 | NATIVE_UNUSED | none | none | none | none | n/a |

SW1 is the STP root for VLANs 10 and 30 and SW2 for VLAN 20, matching the HSRP active routers.

## Loopbacks

| Device | Interface | Address | Purpose |
|--------|-----------|---------|---------|
| R1 | Loopback0 | 10.255.0.1/32 | OSPF and BGP router ID, iBGP source |
| R2 | Loopback0 | 10.255.0.2/32 | OSPF and BGP router ID, iBGP source |
| Aruba CX | loopback 0 | 10.255.0.3/32 | BGP router ID |
| ISP | lo0 (bridge) | 198.51.100.1/32 | Simulated internet address |

198.51.100.0/24 is a documentation range (RFC 5737), used here to stand in for an internet destination.

## Point-to-point links

| Link | Subnet | Side A | Side B |
|------|--------|--------|--------|
| R1 to R2 (OSPF) | 172.16.0.0/30 | R1 Fa1/0 = .1 | R2 Fa1/0 = .2 |
| R1 to ISP | 172.16.1.0/30 | R1 Fa2/0 = .1 | ISP ether1 = .2 |
| R2 to ISP | 172.16.2.0/30 | R2 Fa2/0 = .1 | ISP ether2 = .2 |
| Aruba CX to ISP | 172.16.3.0/30 | Aruba 1/1/1 = .1 | ISP ether3 = .2 |
| HQ firewall to ISP | 172.16.4.0/30 | ASAv Gi0/0 (outside) = .1 | ISP ether4 = .2 |
| Branch A to ISP | 172.16.5.0/30 | BranchA-RTR ether1 = .1 | ISP ether5 = .2 |

## ISP interface summary (MikroTik CHR)

| Interface | Address | Connected to |
|-----------|---------|--------------|
| ether1 | 172.16.1.2/30 | R1 Fa2/0 |
| ether2 | 172.16.2.2/30 | R2 Fa2/0 |
| ether3 | 172.16.3.2/30 | Aruba CX 1/1/1 |
| ether4 | 172.16.4.2/30 | ASAv Gi0/0 |
| ether5 | 172.16.5.2/30 | BranchA-RTR ether1 |
| lo0 | 198.51.100.1/32 | simulated internet |

## HQ firewall (Cisco ASAv, HQ-FW)

| Interface | Name | Security level | Address | Connected to |
|-----------|------|----------------|---------|--------------|
| Gi0/0 | outside | 0 | 172.16.4.1/30 | ISP ether4 |
| Gi0/1 | inside | 100 | 10.4.1.1/24 | PC4 |
| Gi0/2 | dmz | 50 | 10.4.2.1/24 | PC6 |

Default route: `0.0.0.0/0` via 172.16.4.2 (the ISP).

| Segment | Subnet | Gateway | Hosts |
|---------|--------|---------|-------|
| HQ inside (servers) | 10.4.1.0/24 | 10.4.1.1 | PC4 = 10.4.1.10 |
| HQ DMZ | 10.4.2.0/24 | 10.4.2.1 | PC6 = 10.4.2.10 |

These networks are separate from the campus VLANs (10.1.0.0/16) and are not advertised into BGP.

## Branch sites

| Site | Subnet | Gateway | Device | Hosts |
|------|--------|---------|--------|-------|
| Branch A | 10.2.0.0/24 | 10.2.0.1 | BranchA-RTR ether2 | PC5 = 10.2.0.10 |
| Branch B | 10.3.0.0/24 | 10.3.0.1 | Aruba VLAN 10 | PC3 = 10.3.0.10 |

### Branch A router (MikroTik CHR, BranchA-RTR)

| Interface | Address | Connected to |
|-----------|---------|--------------|
| ether1 | 172.16.5.1/30 | ISP ether5 |
| ether2 | 10.2.0.1/24 | PC5 |

Default route: `0.0.0.0/0` via 172.16.5.2 (the ISP). The router masquerades LAN traffic to the internet, except traffic for 10.4.1.0/24, which is exempt.

## End hosts

| Host | Address | Gateway | Location |
|------|---------|---------|----------|
| PC1 | 10.1.10.10/24 | 10.1.10.1 | HQ campus, VLAN 10 |
| PC2 | 10.1.20.10/24 | 10.1.20.1 | HQ campus, VLAN 20 |
| PC3 | 10.3.0.10/24 | 10.3.0.1 | Branch B |
| PC4 | 10.4.1.10/24 | 10.4.1.1 | HQ firewall, inside |
| PC5 | 10.2.0.10/24 | 10.2.0.1 | Branch A |
| PC6 | 10.4.2.10/24 | 10.4.2.1 | HQ firewall, DMZ |

## Physical cabling

### HQ campus

| Link | Port A | Port B |
|------|--------|--------|
| Port-channel 1 (LACP) | SW1 Gi0/0 and Gi0/1 | SW2 Gi0/0 and Gi0/1 |
| R1 trunk | R1 Fa0/0 | SW1 Gi0/2 |
| R2 trunk | R2 Fa0/0 | SW2 Gi0/2 |
| PC1 | PC1 | SW1 Gi0/3 (VLAN 10) |
| PC2 | PC2 | SW2 Gi0/3 (VLAN 20) |

### Branch B

| GNS3 port (Aruba) | Aruba interface | Connected to |
|-------------------|-----------------|--------------|
| Ethernet0 | management | not connected |
| Ethernet1 | 1/1/1 | ISP ether3 |
| Ethernet2 | 1/1/2 | PC3 |

### HQ firewall and Branch A

| From | To |
|------|----|
| ISP ether4 | ASAv Gi0/0 (outside) |
| ASAv Gi0/1 (inside) | PC4 |
| ASAv Gi0/2 (dmz) | PC6 |
| ISP ether5 | BranchA-RTR ether1 |
| BranchA-RTR ether2 | PC5 |

## Routing design

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
| Static default routes | ASAv, BranchA-RTR | Via the ISP; neither site runs a routing protocol |

## NAT design (ASAv)

| Rule | Source | Destination | Translation |
|------|--------|-------------|-------------|
| 1 (twice NAT, identity) | 10.4.1.0/24 (inside) | 10.2.0.0/24 (outside) | None. Exempts VPN traffic from NAT. |
| 2 (object NAT, dynamic) | 10.4.1.0/24 (inside) | any | PAT to the outside interface address (172.16.4.1) |
| 3 (object NAT, dynamic) | 10.4.2.0/24 (dmz) | any | PAT to the outside interface address (172.16.4.1) |

The DMZ policy (ACL `DMZ-IN`, applied inbound on the dmz interface) denies DMZ traffic to the inside network with logging, and permits ICMP from the DMZ to anywhere else.

## Site-to-site VPN

| Parameter | Value |
|-----------|-------|
| Peers | ASAv outside 172.16.4.1 and BranchA-RTR ether1 172.16.5.1 |
| Protected traffic | 10.4.1.0/24 to 10.2.0.0/24 |
| IKE version | IKEv2 |
| IKE policy | AES-256, SHA-256, DH group 14, lifetime 86400 s |
| IPsec (ESP) | AES-256, SHA-256, no PFS |
| Authentication | Pre-shared key (not recorded in this repository) |
| ASAv objects | crypto map `OUTSIDE-MAP`, ACL `VPN-TRAFFIC`, proposal `AES256-SHA256` |
| MikroTik objects | profile `asav-profile`, proposal `asav-proposal`, peer `HQ-FW` |

On the MikroTik, the Branch A LAN is exempt from masquerade for traffic to 10.4.1.0/24, which mirrors the ASAv's NAT exemption.