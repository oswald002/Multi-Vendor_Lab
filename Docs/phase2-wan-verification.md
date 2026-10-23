# Phase 2: Multi-Vendor WAN Verification

**Summary:** All eBGP and iBGP sessions are Established. HQ prefers R1 for outbound
and the ISP prefers R1 for inbound. Failover to R2 completed in <N> seconds.

## 1. BGP sessions
**Goal:** every neighbor Established.
**Commands:** `show ip bgp summary` (R1, R2), `/routing bgp session print` (ISP),
`show bgp ipv4 unicast summary` (Aruba)
```

R1#sh ip bgp summ
BGP router identifier 10.255.0.1, local AS number 65001
BGP table version is 4, main routing table version 4
3 network entries using 432 bytes of memory
4 path entries using 320 bytes of memory
6/3 BGP path/bestpath attribute entries using 816 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1616 total bytes of memory
BGP activity 3/0 prefixes, 4/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.255.0.2      4        65001      17      18        4    0    0 00:12:14        1
172.16.1.2      4        65000      68      71        4    0    0 00:10:31        2


R2#sh ip bgp summ
BGP router identifier 10.255.0.2, local AS number 65001
BGP table version is 4, main routing table version 4
3 network entries using 432 bytes of memory
6 path entries using 480 bytes of memory
6/3 BGP path/bestpath attribute entries using 816 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1776 total bytes of memory
BGP activity 3/0 prefixes, 6/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.255.0.1      4        65001      19      17        4    0    0 00:12:53        3
172.16.2.2      4        65000      72      74        4    0    0 00:11:06        2



[admin@ISP] > routing bgp session print
Flags: E - ESTABLISHED
 0 E name="to-R1-1" instance=default
     remote.address=172.16.1.1 .as=65001 .id=10.255.0.1
     .capabilities=mp,rr,as4,err .afi=ip .hold-time=30s .messages=79
     .bytes=1539 .eor=ip
     local.address=172.16.1.2 .as=65000 .id=198.51.100.1
     .cluster-id=198.51.100.1 .capabilities=mp,rr,enhe,gr,as4 .afi=ip
     .messages=75 .bytes=1488 .eor=""
     output.procid=20 .network=ISP-NETS
     input.procid=20 ebgp
     hold-time=30s keepalive-time=10s uptime=12m8s650ms
     last-started=2026-10-23 09:05:34 prefix-count=1

 1 E name="to-BranchB-1" instance=default
     remote.address=172.16.3.1 .as=65003 .id=10.255.0.3
     .capabilities=mp,rr,gr,as4,err .afi=ip .messages=15 .bytes=317
     .gr-time=120 .gr-afi=ip .eor=ip
     local.address=172.16.3.2 .as=65000 .id=198.51.100.1
     .cluster-id=198.51.100.1 .capabilities=mp,rr,enhe,gr,as4 .afi=ip
     .messages=15 .bytes=347 .eor=""
     output.procid=21 .network=ISP-NETS
     input.procid=21 ebgp
     hold-time=3m keepalive-time=1m uptime=12m4s940ms
     last-started=2026-10-23 09:05:38 prefix-count=1

 2 E name="to-R2-1" instance=default
     remote.address=172.16.2.1 .as=65001 .id=10.255.0.2
     .capabilities=mp,rr,as4,err .afi=ip .hold-time=30s .messages=78
     .bytes=1528 .eor=ip
     local.address=172.16.2.2 .as=65000 .id=198.51.100.1
     .cluster-id=198.51.100.1 .capabilities=mp,rr,enhe,gr,as4 .afi=ip
     .messages=76 .bytes=1539 .eor=""
     output.procid=22 .network=ISP-NETS
     input.procid=22 ebgp
     hold-time=30s keepalive-time=10s uptime=12m6s900ms
     last-started=2026-10-23 09:05:38 prefix-count=1




BranchB-CX# sh bgp ipv4 unicast summary
VRF : default
BGP Summary
-----------
 Local AS               : 65003        BGP Router Identifier  : 10.255.0.3
 Peers                  : 1            Log Neighbor Changes   : No
 Cfg. Hold Time         : 180          Cfg. Keep Alive        : 60
 Confederation Id       : 0

 Neighbor        Remote-AS MsgRcvd MsgSent   Up/Down Time State        AdminStatus
 172.16.3.2      65000       21      22      00h:16m:34s  Established   Up      


```
**Result:** All three ISP sessions are Established, each receiving one prefix
(10.1.0.0/16 from R1 and R2, 10.3.0.0/24 from the Aruba). The HQ sessions use a
30-second hold time and 10-second keepalive; the branch session uses the defaults.

## 2. Local preference
**Goal:** routes learned from the ISP carry local-pref 200 on R1.
**Command:** `show ip bgp` (R1, R2)
```

R1#sh ip bgp
BGP table version is 4, local router ID is 10.255.0.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i 10.1.0.0/16      10.255.0.2               0    100      0 i
 *>                   0.0.0.0                  0         32768 i
 *>  10.3.0.0/24      172.16.1.2                    200      0 65000 65003 i
 *>  198.51.100.1/32  172.16.1.2                    200      0 65000 i


R2#sh ip bgp
BGP table version is 4, local router ID is 10.255.0.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i 10.1.0.0/16      10.255.0.1               0    100      0 i
 *>                   0.0.0.0                  0         32768 i
 *   10.3.0.0/24      172.16.2.2                             0 65000 65003 i
 *>i                  10.255.0.1               0    200      0 65000 65003 i
 *   198.51.100.1/32  172.16.2.2                             0 65000 i
 *>i                  10.255.0.1               0    200      0 65000 i


```
**Result:** R1's ISP routes carry LocPrf 200, and R2 receives them over iBGP with the
same value, so it prefers them over its own direct ISP paths.

## 3. Inbound path preference
**Goal:** the ISP prefers R1 for 10.1.0.0/16.
**Commands:** `/ip route print where dst-address=10.1.0.0/16`,
`/routing route print detail where dst-address=10.1.0.0/16`
```
[admin@ISP] > ip route print where dst-address=10.1.0.0/16
Flags: D - DYNAMIC; A - ACTIVE; b - BGP
Columns: DST-ADDRESS, GATEWAY, ROUTING-TABLE, DISTANCE
    DST-ADDRESS  GATEWAY     ROUTING-TABLE  DISTANCE
DAb 10.1.0.0/16  172.16.1.1  main                 20
D b 10.1.0.0/16  172.16.2.1  main                 20
[admin@ISP] > routing route print detail where dst-address=10.1.0.0/16
Flags: X - DISABLED, F - FILTERED, U - UNREACHABLE, A - ACTIVE;
c - CONNECT, s - STATIC, r - RIP, b - BGP, n - BGP-NET, o - OSPF, i - ISIS, d >
H - HW-OFFLOADED; + - ECMP, B - BLACKHOLE
 Ab   afi=ip contribution=active dst-address=10.1.0.0/16 routing-table=main
       gateway=172.16.1.1 immediate-gw=172.16.1.1%ether1 distance=20 scope=40
       target-scope=10 belongs-to="bgp-IP-172.16.1.1"
       bgp.session=to-R1-1 .as-path="65001" .med=0 .origin=igp
       debug.fwp-ptr=0x202823C0

  b   afi=ip contribution=candidate dst-address=10.1.0.0/16
       routing-table=main gateway=172.16.2.1 immediate-gw=172.16.2.1%ether2
       distance=20 scope=40 target-scope=10 belongs-to="bgp-IP-172.16.2.1"
       bgp.session=to-R2-1 .as-path="65001,65001,65001" .med=0 .origin=igp
       debug.fwp-ptr=0x20282480

```
**Result:** Two paths held. R1's (AS path 65001) is active; R2's (65001,65001,65001,
from the outbound prepend) is a candidate.

## 4. Learned routes
**Commands:** `show ip route bgp` (R1, Aruba)
```
R1#sh ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 10 subnets, 3 masks
B        10.3.0.0/24 [20/0] via 172.16.1.2, 00:26:39
      198.51.100.0/32 is subnetted, 1 subnets
B        198.51.100.1 [20/0] via 172.16.1.2, 00:26:43


BranchB-CX# sh ip route

Displaying ipv4 routes selected for forwarding

Origin Codes: C - connected, S - static, L - local
              R - RIP, B - BGP, O - OSPF
Type Codes:   E - External BGP, I - Internal BGP, V - VPN, EV - EVPN
              IA - OSPF internal area, E1 - OSPF external type 1
              E2 - OSPF external type 2

VRF: default

Prefix              Nexthop          Interface     VRF(egress)       Origin/   D                                                                                                             istance/    Age
                                                                     Type      M                                                                                                             etric
--------------------------------------------------------------------------------                                                                                                             -------------------------
10.1.0.0/16         172.16.3.2       1/1/1         -                 B/E       [                                                                                                             20/0]       00h:28m:09s
10.3.0.0/24         -                vlan10        -                 C         [                                                                                                             0/0]        -
10.3.0.1/32         -                vlan10        -                 L         [                                                                                                             0/0]        -
10.255.0.3/32       -                loopback0     -                 L         [                                                                                                             0/0]        -
172.16.3.0/30       -                1/1/1         -                 C         [                                                                                                             0/0]        -
172.16.3.1/32       -                1/1/1         -                 L         [                                                                                                             0/0]        -
198.51.100.1/32     172.16.3.2       1/1/1         -                 B/E       [                                                                                                             20/0]       00h:28m:09s

Total Route Count : 7




```
**Result:** Each site learns the other's prefix via eBGP (distance 20).

## 5. End-to-end connectivity
**Goal:** confirm hosts at HQ and Branch B can reach each other through the ISP,
and record the path used in each direction.
**Commands:** `ping` and `trace` between PC1, PC2 and PC3.

**HQ to branch (PC1 to PC3):**
```
PC1> ping 10.3.0.10

84 bytes from 10.3.0.10 icmp_seq=1 ttl=61 time=68.664 ms
84 bytes from 10.3.0.10 icmp_seq=2 ttl=61 time=59.219 ms
84 bytes from 10.3.0.10 icmp_seq=3 ttl=61 time=38.403 ms
84 bytes from 10.3.0.10 icmp_seq=4 ttl=61 time=38.385 ms
84 bytes from 10.3.0.10 icmp_seq=5 ttl=61 time=51.588 ms

PC1> trace 10.3.0.10
trace to 10.3.0.10, 8 hops max, press Ctrl+C to stop
 1   10.1.10.2   9.380 ms  30.337 ms  30.420 ms
 2   172.16.1.2   70.082 ms  60.520 ms  60.021 ms
 3   172.16.3.1   60.538 ms  59.721 ms  60.693 ms
 4   *10.3.0.10   60.624 ms (ICMP type:3, code:3, Destination port unreachable)
```
Path: R1, ISP, Aruba CX, PC3.

**Branch to HQ (PC3 to PC1):**
```
PC3> ping 10.1.10.10

84 bytes from 10.1.10.10 icmp_seq=1 ttl=61 time=46.977 ms
84 bytes from 10.1.10.10 icmp_seq=2 ttl=61 time=37.888 ms
84 bytes from 10.1.10.10 icmp_seq=3 ttl=61 time=49.203 ms
84 bytes from 10.1.10.10 icmp_seq=4 ttl=61 time=70.624 ms
84 bytes from 10.1.10.10 icmp_seq=5 ttl=61 time=68.015 ms

PC3> trace 10.1.10.10
trace to 10.1.10.10, 8 hops max, press Ctrl+C to stop
 1   10.3.0.1   3.185 ms  3.242 ms  1.738 ms
 2   172.16.3.2   7.677 ms  6.000 ms  3.967 ms
 3   172.16.1.1   38.083 ms  31.752 ms  28.044 ms
 4   *10.1.10.10   61.336 ms (ICMP type:3, code:3, Destination port unreachable)
```
Path: Aruba CX, ISP, R1, PC1.

**VLAN 20 to branch (PC2 to PC3, trace only):**
```
PC2> trace 10.3.0.10
trace to 10.3.0.10, 8 hops max, press Ctrl+C to stop
 1   10.1.20.3   22.263 ms  29.871 ms  30.085 ms
 2   172.16.0.1   51.455 ms  28.493 ms  29.358 ms
 3   172.16.1.2   60.128 ms  61.479 ms  60.004 ms
 4   172.16.3.1   60.088 ms  60.393 ms  61.447 ms
 5   *10.3.0.10   69.744 ms (ICMP type:3, code:3, Destination port unreachable)
```
Path: R2, R1, ISP, Aruba CX, PC3.

PC1 to PC2, PC2 to PC1 and PC3 to PC2 pings also succeeded.

**Result:** All PCs reach each other. Outbound traffic from HQ exits via R1 (VLAN 20
traffic first crosses the R1-R2 link because of local-preference 200), and return
traffic from the branch enters HQ via R1, because R2's advertisement to the ISP is
prepended. Traffic between VLAN 20 and the branch is therefore asymmetric, which
follows from the HSRP load-balancing design combined with a single preferred exit.

## 6. Failover
**Steps:** <list>
**Timers:** hold time <30 s / 180 s>
```

PC1> ping 10.3.0.10 -t
<Redacted>
84 bytes from 10.3.0.10 icmp_seq=13 ttl=61 time=50.250 ms
84 bytes from 10.3.0.10 icmp_seq=14 ttl=61 time=70.477 ms
84 bytes from 10.3.0.10 icmp_seq=15 ttl=61 time=39.982 ms
10.3.0.10 icmp_seq=16 timeout
10.3.0.10 icmp_seq=17 timeout
10.3.0.10 icmp_seq=18 timeout
10.3.0.10 icmp_seq=19 timeout
10.3.0.10 icmp_seq=20 timeout
10.3.0.10 icmp_seq=21 timeout
10.3.0.10 icmp_seq=22 timeout
10.3.0.10 icmp_seq=23 timeout
10.3.0.10 icmp_seq=24 timeout
10.3.0.10 icmp_seq=25 timeout
10.3.0.10 icmp_seq=26 timeout
10.3.0.10 icmp_seq=27 timeout
10.3.0.10 icmp_seq=28 timeout
10.3.0.10 icmp_seq=29 timeout
10.3.0.10 icmp_seq=30 timeout
10.3.0.10 icmp_seq=31 timeout
10.3.0.10 icmp_seq=32 timeout
10.3.0.10 icmp_seq=33 timeout
10.3.0.10 icmp_seq=34 timeout
10.3.0.10 icmp_seq=35 timeout
10.3.0.10 icmp_seq=36 timeout
84 bytes from 10.3.0.10 icmp_seq=37 ttl=61 time=55.712 ms
84 bytes from 10.3.0.10 icmp_seq=38 ttl=61 time=51.427 ms
84 bytes from 10.3.0.10 icmp_seq=39 ttl=61 time=68.634 ms
84 bytes from 10.3.0.10 icmp_seq=40 ttl=61 time=71.032 ms
84 bytes from 10.3.0.10 icmp_seq=41 ttl=61 time=41.080 ms
84 bytes from 10.3.0.10 icmp_seq=42 ttl=61 time=41.046 ms
84 bytes from 10.3.0.10 icmp_seq=43 ttl=61 time=39.400 ms
84 bytes from 10.3.0.10 icmp_seq=44 ttl=61 time=51.073 ms
84 bytes from 10.3.0.10 icmp_seq=45 ttl=61 time=52.646 ms
<Redacted>







R1#sh ip bgp summ
BGP router identifier 10.255.0.1, local AS number 65001
BGP table version is 4, main routing table version 4
3 network entries using 432 bytes of memory
4 path entries using 320 bytes of memory
5/3 BGP path/bestpath attribute entries using 680 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1480 total bytes of memory
BGP activity 3/0 prefixes, 6/2 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.255.0.2      4        65001       8       5        4    0    0 00:01:26        3
172.16.1.2      4        65000       0       0        1    0    0 00:01:30 Idle
R1#
R1#
R1#sh ip route 10.3.0.0
Routing entry for 10.3.0.0/24
  Known via "bgp 65001", distance 200, metric 0
  Tag 65000, type internal
  Last update from 10.255.0.2 00:00:52 ago
  Routing Descriptor Blocks:
  * 10.255.0.2, from 10.255.0.2, 00:00:52 ago
      Route metric is 0, traffic share count is 1
      AS Hops 2
      Route tag 65000
      MPLS label: none





[admin@ISP] > routing bgp session print without-paging
Flags: E - ESTABLISHED
 0 E name="to-R1-1" instance=default
     remote.address=172.16.1.1 .as=65001 .id=10.255.0.1
     .capabilities=mp,rr,as4,err .afi=ip .hold-time=30s .messages=0 .bytes=0
     .eor=""
     local.address=172.16.1.2 .as=65000 .id=198.51.100.1
     .cluster-id=198.51.100.1 .capabilities=mp,rr,enhe,gr,as4 .afi=ip
     .messages=35 .bytes=768 .eor=""
     output.procid=20 .network=ISP-NETS
     .last-notification=FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF0015030400
     input.procid=20 ebgp
     hold-time=30s keepalive-time=10s uptime=5m10s670ms
     last-started=2026-10-23 09:54:13 last-stopped=2026-10-23 09:52:29
     prefix-count=0

 1 E name="to-BranchB-1" instance=default
     remote.address=172.16.3.1 .as=65003 .id=10.255.0.3
     .capabilities=mp,rr,gr,as4,err .afi=ip .messages=63 .bytes=1229
     .gr-time=120 .gr-afi=ip .eor=ip
     local.address=172.16.3.2 .as=65000 .id=198.51.100.1
     .cluster-id=198.51.100.1 .capabilities=mp,rr,enhe,gr,as4 .afi=ip
     .messages=59 .bytes=1295 .eor=""
     output.procid=21 .network=ISP-NETS
     input.procid=21 ebgp
     hold-time=3m keepalive-time=1m uptime=53m45s940ms
     last-started=2026-10-23 09:05:38 prefix-count=1

 2 E name="to-R2-1" instance=default
     remote.address=172.16.2.1 .as=65001 .id=10.255.0.2
     .capabilities=mp,rr,as4,err .afi=ip .hold-time=30s .messages=340
     .bytes=6506 .eor=ip
     local.address=172.16.2.2 .as=65000 .id=198.51.100.1
     .cluster-id=198.51.100.1 .capabilities=mp,rr,enhe,gr,as4 .afi=ip
     .messages=329 .bytes=6392 .eor=""
     output.procid=22 .network=ISP-NETS
     input.procid=22 ebgp
     hold-time=30s keepalive-time=10s uptime=53m45s900ms
     last-started=2026-10-23 09:05:38 prefix-count=1


dmin@ISP] > ip route print where dst-address=10.1.0.0/16
Flags: D - DYNAMIC; A - ACTIVE; b - BGP
Columns: DST-ADDRESS, GATEWAY, ROUTING-TABLE, DISTANCE
    DST-ADDRESS  GATEWAY     ROUTING-TABLE  DISTANCE
DAb 10.1.0.0/16  172.16.2.1  main                 20









R2#sh ip bgp
BGP table version is 10, local router ID is 10.255.0.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i 10.1.0.0/16      10.255.0.1               0    100      0 i
 *>                   0.0.0.0                  0         32768 i
 *>  10.3.0.0/24      172.16.2.2                             0 65000 65003 i
 *>  198.51.100.1/32  172.16.2.2                             0 65000 i




PC1> trace 10.3.0.10
trace to 10.3.0.10, 8 hops max, press Ctrl+C to stop
 1   10.1.10.2   39.036 ms  29.996 ms  30.062 ms
 2   172.16.0.2   56.200 ms  60.140 ms  60.239 ms
 3   172.16.2.2   90.549 ms  90.691 ms  59.600 ms
 4   172.16.3.1   61.088 ms  59.523 ms  71.453 ms
 5   *10.3.0.10   90.209 ms (ICMP type:3, code:3, Destination port unreachable)





PC2> trace 10.3.0.10
trace to 10.3.0.10, 8 hops max, press Ctrl+C to stop
 1   10.1.20.3   30.033 ms  30.020 ms  29.007 ms
 2   172.16.2.2   59.967 ms  60.320 ms  70.554 ms
 3   172.16.3.1   59.171 ms  60.133 ms  60.569 ms
 4   *10.3.0.10   60.190 ms (ICMP type:3, code:3, Destination port unreachable)

```


**Result:** Traffic was interrupted for about 30 seconds (measured with a stopwatch;
21 pings timed out), after which it resumed over R2's path. The outage matches the
hold time: the virtual link stayed up on the ISP's side, so the ISP detected the
failure only when the hold timer expired (NOTIFICATION code 4, Hold Timer Expired).
- R1's route to 10.3.0.0/24 changed to an iBGP route via R2 (10.255.0.2).
- R2 selected its direct ISP paths as best.
- The ISP kept only R2's path for 10.1.0.0/16.
- Traces: PC1 crossed R1, R2, ISP, Aruba; PC2 exited via R2 directly.


```

R1#sh ip bgp summ
BGP router identifier 10.255.0.1, local AS number 65001
BGP table version is 6, main routing table version 6
3 network entries using 432 bytes of memory
4 path entries using 320 bytes of memory
6/3 BGP path/bestpath attribute entries using 816 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1616 total bytes of memory
BGP activity 3/0 prefixes, 8/4 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.255.0.2      4        65001      25      21        6    0    0 00:14:15        1
172.16.1.2      4        65000      30      29        6    0    0 00:03:54        2
R1#


dmin@ISP] > ip route print where dst-address=10.1.0.0/16
Flags: D - DYNAMIC; A - ACTIVE; b - BGP
Columns: DST-ADDRESS, GATEWAY, ROUTING-TABLE, DISTANCE
    DST-ADDRESS  GATEWAY     ROUTING-TABLE  DISTANCE
D b 10.1.0.0/16  172.16.2.1  main                 20
DAb 10.1.0.0/16  172.16.1.1  main                 20


PC1> trace 10.3.0.10
trace to 10.3.0.10, 8 hops max, press Ctrl+C to stop
 1   10.1.10.2   29.712 ms  29.113 ms  30.639 ms
 2   172.16.1.2   60.284 ms  59.954 ms  59.976 ms
 3   172.16.3.1   60.117 ms  60.832 ms  60.468 ms
 4   *10.3.0.10   60.187 ms (ICMP type:3, code:3, Destination port unreachable)




```
**Recovery:** After `no shutdown` on R1's Fa2/0, the R1-ISP session re-established
(2 prefixes received), the ISP again selected R1's path for 10.1.0.0/16 as active
(R2's path returned to standby), and the PC1 trace again crossed R1 first. The iBGP
session with R2 remained stable. Failback time was not measured.


**Summary:** All eBGP and iBGP sessions are Established. HQ prefers R1 for outbound and
the ISP prefers R1 for inbound. With a 30-second hold time, failover to R2 took about
30 seconds, and traffic returned to R1 after the link was restored.