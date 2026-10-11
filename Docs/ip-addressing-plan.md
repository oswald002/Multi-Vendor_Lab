# IP Addressing Plan

## VLANs
| VLAN | Name | Subnet | HSRP VIP | R1 | R2 |
|------|------|--------|----------|----|----|
| 10 | users | 10.1.10.0/24 | 10.1.10.1 | .2 | .3 |
| 20 | servers | 10.1.20.0/24 | 10.1.20.1 | .2 | .3 |
| 30 | management | 10.1.30.0/24 | 10.1.30.1 | .2 | .3 |

## Loopbacks and links
| Device | Interface | Address |
|--------|-----------|---------|
| R1 | Loopback0 | 10.255.0.1/32 |
| R2 | Loopback0 | 10.255.0.2/32 |
| R1-R2 | Fa1/0 | 172.16.0.0/30 (R1 .1, R2 .2) |