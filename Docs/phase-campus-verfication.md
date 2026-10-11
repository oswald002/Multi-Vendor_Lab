# Phase 1: HQ Campus Verification

###All checks passed. HSRP failover completed


## 1. EtherChannel (SW1-SW2)
Goal: confirm both links are bundled into Po1 using LACP.
Command (SW1)&(SW2): 'show etherchannel summary'


SW1#sh etherchannel summ
<Redacted>
Number of channel-groups in use: 1
Number of aggregators:           1

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         LACP      Gi0/0(P)    Gi0/1(P)

SW2#sh etherchannel summ
<Redacted>
Number of channel-groups in use: 1
Number of aggregators:           1

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         LACP      Gi0/0(P)    Gi0/1(P)


Result: Po1 is 'SU' (Layer 2, in use) and both member ports show 'P'
(bundled), so traffic is load-balanced across both links.



## 2. Spanning tree root placement
Goal: confirm SW1 is root for VLANs 10 and 30, and SW2 for VLAN 20.
Commands: 'show spanning-tree vlan 10' (SW1), 'show spanning-tree vlan 20' (SW2)

SW1#sh spanning-tree vl 10

VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    24586
             Address     0c6a.d5a8.0000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    24586  (priority 24576 sys-id-ext 10)
             Address     0c6a.d5a8.0000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/2               Desg FWD 4         128.3    P2p
Gi0/3               Desg FWD 4         128.4    P2p Edge
Po1                 Desg FWD 3         128.65   P2p


SW2#sh spanning-tree vl 20

VLAN0020
  Spanning tree enabled protocol rstp
  Root ID    Priority    24596
             Address     0c11.3fc3.0000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    24596  (priority 24576 sys-id-ext 20)
             Address     0c11.3fc3.0000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/2               Desg FWD 4         128.3    P2p
Gi0/3               Desg FWD 4         128.4    P2p Edge
Po1                 Desg FWD 3         128.65   P2p




Result: each switch reports "This bridge is the root" for its intended VLANs.

## 3. HSRP state
Goal: confirm R1 is Active for VLANs 10 and 30, R2 for VLAN 20.
Command (R1 and R2):'show standby brief'

R1#sh standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Fa0/0.10    10   110 P Active  local           10.1.10.3       10.1.10.1
Fa0/0.20    20   100 P Standby 10.1.20.3       local           10.1.20.1
Fa0/0.30    30   110 P Active  local           10.1.30.3       10.1.30.1

R2#sh standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Fa0/0.10    10   100 P Standby 10.1.10.2       local           10.1.10.1
Fa0/0.20    20   110 P Active  local           10.1.20.2       10.1.20.1
Fa0/0.30    30   100 P Standby 10.1.30.2       local           10.1.30.1



Result: R1 (priority 110) is Active for groups 10 and 30 and Standby for group 20.
R2 (priority 110 on group 20) is Active for group 20 and Standby for groups 10 and 30.
Each router sees the other as its peer, so the gateway load is split across the two
routers and the Layer 3 split matches the STP root placement on the switches.

## 4. OSPF adjacency and routes
Goal: confirm R1 and R2 form a FULL OSPF adjacency over the routed link and that
loopbacks are exchanged.
Commands (R1 and R2): 'show ip ospf neighbor', show ip route ospf, 'show ip protocols'

R1#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.255.0.2        1   FULL/DR         00:00:37    172.16.0.2      FastEthernet1/0
R1#sh ip route ospf
Codes: <Redacted>

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 8 subnets, 2 masks
O        10.255.0.2/32 [110/2] via 172.16.0.2, 00:39:54, FastEthernet1/0
R1#sh ip protocols
*** IP Routing is NSF aware ***

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 10.255.0.1
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    10.1.0.0 0.0.255.255 area 0
    10.255.0.1 0.0.0.0 area 0
    172.16.0.0 0.0.0.3 area 0
  Passive Interface(s):
    FastEthernet0/0
    FastEthernet0/0.10
    FastEthernet0/0.20
    FastEthernet0/0.30
    Ethernet2/0
    Ethernet2/1
    Ethernet2/2
    Ethernet2/3
    Loopback0
  Routing Information Sources:
    Gateway         Distance      Last Update
    10.255.0.2           110      00:40:14
  Distance: (default is 110)

R2#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.255.0.1        1   FULL/BDR        00:00:33    172.16.0.1      FastEthernet1/0
R2#sh ip route ospf
Codes: <Redacted>

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 8 subnets, 2 masks
O        10.255.0.1/32 [110/2] via 172.16.0.1, 00:40:10, FastEthernet1/0
R2#sh ip protocols
*** IP Routing is NSF aware ***

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 10.255.0.2
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    10.1.0.0 0.0.255.255 area 0
    10.255.0.2 0.0.0.0 area 0
    172.16.0.0 0.0.0.3 area 0
  Passive Interface(s):
    FastEthernet0/0
    FastEthernet0/0.10
    FastEthernet0/0.20
    FastEthernet0/0.30
    Ethernet2/0
    Ethernet2/1
    Ethernet2/2
    Ethernet2/3
    Loopback0
  Routing Information Sources:
    Gateway         Distance      Last Update
    10.255.0.1           110      00:40:18
  Distance: (default is 110)




Result: Each router shows the other (router IDs 10.255.0.1 and 10.255.0.2) in the
FULL state on Fa1/0. The OSPF routing table contains the other router's loopback
(/32) via 172.16.0.x. The VLAN subnets appear as connected routes rather than OSPF
routes, because both routers have them directly attached. `show ip protocols` lists
the Fa0/0 subinterfaces as passive and Fa1/0 as active, so adjacencies can only form
on the router-to-router link.

## 5. Inter-VLAN connectivity (PC1 to PC2, and back)
Goal: confirm hosts in different VLANs can reach each other through the HSRP gateways.
Commands: 'ping 10.1.20.10' (PC1), 'ping 10.1.10.10' (PC2), 'trace 10.1.20.10' (PC1)


PC1> ping 10.1.20.10

84 bytes from 10.1.20.10 icmp_seq=1 ttl=63 time=150.427 ms
84 bytes from 10.1.20.10 icmp_seq=2 ttl=63 time=80.124 ms
84 bytes from 10.1.20.10 icmp_seq=3 ttl=63 time=38.356 ms
84 bytes from 10.1.20.10 icmp_seq=4 ttl=63 time=50.103 ms
84 bytes from 10.1.20.10 icmp_seq=5 ttl=63 time=70.680 ms


PC2> ping 10.1.10.10

84 bytes from 10.1.10.10 icmp_seq=1 ttl=63 time=66.913 ms
84 bytes from 10.1.10.10 icmp_seq=2 ttl=63 time=67.424 ms
84 bytes from 10.1.10.10 icmp_seq=3 ttl=63 time=38.086 ms
84 bytes from 10.1.10.10 icmp_seq=4 ttl=63 time=40.703 ms
84 bytes from 10.1.10.10 icmp_seq=5 ttl=63 time=40.762 ms


PC1> trace 10.1.20.10
trace to 10.1.20.10, 8 hops max, press Ctrl+C to stop
 1   10.1.10.2   28.288 ms  29.752 ms  29.991 ms
 2   *10.1.20.10   61.104 ms (ICMP type:3, code:3, Destination port unreachable)


Result: Pings succeed in both directions. The traceroute shows the first hop is
R1's VLAN 10 interface (10.1.10.2), the Active router for that VLAN, followed by
PC2. This proves traffic is routed between VLANs 10 and 20 via the HSRP gateway.
(Note: an initial failure was caused by a configuration error; see
(troubleshooting-log.md)

## 6. HSRP failover test
Goal: confirm R2 takes over as gateway when R1 fails, and R1 reclaims the role
when it recovers.
Steps:
1. On PC1, start a continuous ping to PC2: 'ping 10.1.20.10 -t'
2. On R1, shut the LAN-facing interface: 'interface FastEthernet0/0' then 'shutdown'
3. Watch the ping output and note how many replies are lost.
4. On R2, run 'show standby brief'.
5. On R1, run 'no shutdown', then 'show standby brief' on both routers.

```
PC1> ping 10.1.20.10 -t
<output shows successful pings before R1 shuts down, then the lost replies, then successful replies after R2 takes over>
84 bytes from 10.1.20.10 icmp_seq=8 ttl=63 time=44.943 ms
84 bytes from 10.1.20.10 icmp_seq=9 ttl=63 time=40.532 ms
84 bytes from 10.1.20.10 icmp_seq=10 ttl=63 time=39.512 ms
10.1.20.10 icmp_seq=11 timeout
10.1.20.10 icmp_seq=12 timeout
10.1.20.10 icmp_seq=13 timeout
10.1.20.10 icmp_seq=14 timeout
84 bytes from 10.1.20.10 icmp_seq=15 ttl=63 time=58.673 ms
84 bytes from 10.1.20.10 icmp_seq=16 ttl=63 time=68.820 ms
84 bytes from 10.1.20.10 icmp_seq=17 ttl=63 time=49.416 ms



R2#sh standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Fa0/0.10    10   100 P Active  local           unknown         10.1.10.1
Fa0/0.20    20   110 P Active  local           unknown         10.1.20.1
Fa0/0.30    30   100 P Active  local           unknown         10.1.30.1

R1#sh standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Fa0/0.10    10   110 P Active  local           10.1.10.3       10.1.10.1
Fa0/0.20    20   100 P Standby 10.1.20.3       local           10.1.20.1
Fa0/0.30    30   110 P Active  local           10.1.30.3       10.1.30.1
R1#

Result: After R1's interface was shut down, R2 became Active for groups 10 and 30
as well as 20. Zero pings were lost during the transition and
connectivity resume without any change on the PCs because they use the virtual IP
(10.1.10.1) as their gateway. After R1 recovered, preemption returned it to Active for
groups 10 and 30.