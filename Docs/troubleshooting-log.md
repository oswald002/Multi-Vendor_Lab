## Issue: PC1 could not ping PC2
Symptom: 'host unreachable' from PC1
Investigation: checked VLANs, trunks, HSRP state, ARP table
Root cause: I had misconfigured the IP addresses on the PCs
Fix: I correctly configured the PCs' addresses
Lesson: Always double check if the IPs are correctly configured on the PCs.



## Issue: HSRP failover test showed no change
Symptom: continuous ping showed no loss, and show standby brief on R2
  still showed R2 as Standby for groups 10 and 30.
Investigation: checked HSRP state on R2 during the test.
Root cause: R1 had not fully gone down the node was still running
Fix: I stopped the node
Lesson: always confirm the standby device actually became Active before
  trusting a failover result.