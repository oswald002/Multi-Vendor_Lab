# Phase 2: Multi-Vendor WAN Verification

**Summary:** All eBGP and iBGP sessions are Established. HQ prefers R1 for outbound
and the ISP prefers R1 for inbound. Failover to R2 completed in <N> seconds.

## 1. BGP sessions
**Goal:** every neighbor Established.
**Commands:** `show ip bgp summary` (R1, R2), `/routing bgp session print` (ISP),
`show bgp ipv4 unicast summary` (Aruba)
```
<paste outputs>
```
**Result:** All sessions up. The ISP shows prefix-count 0 for R2 because R2's route is
a standby candidate rather than the active route (see section 3).

## 2. Local preference
**Goal:** routes learned from the ISP carry local-pref 200 on R1.
**Command:** `show ip bgp` (R1, R2)
```
<paste outputs>
```
**Result:** R1's ISP routes carry LocPrf 200, and R2 receives them over iBGP with the
same value, so it prefers them over its own direct ISP paths.

## 3. Inbound path preference
**Goal:** the ISP prefers R1 for 10.1.0.0/16.
**Commands:** `/ip route print where dst-address=10.1.0.0/16`,
`/routing route print detail where dst-address=10.1.0.0/16`
```
<paste outputs>
```
**Result:** Two paths held. R1's (AS path 65001) is active; R2's (65001,65001,65001,
from the outbound prepend) is a candidate.

## 4. Learned routes
**Commands:** `show ip route bgp` (R1, Aruba)
```
<paste outputs>
```
**Result:** Each site learns the other's prefix via eBGP (distance 20).

## 5. End-to-end connectivity
**Commands:** ping and trace between PC1, PC2 and PC3
```
<paste outputs>
```
**Result:** Full connectivity. Traces show R1 used in both directions.

## 6. Failover
**Steps:** <list>
**Timers:** hold time <30 s / 180 s>
```
<paste captures>
```
**Result:** Outage of <N> s. <what changed, and recovery behaviour>