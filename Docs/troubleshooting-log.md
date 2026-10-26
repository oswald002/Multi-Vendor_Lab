### Troubleshooting Log

Problems met while building the lab, how each was investigated, and what was learned.

### Phase 1: HQ campus

### 1. GNS3 VM refused to start (insufficient system resources)

- **Symptom:** the hypervisor reported insufficient system resources when the VM was given 8 GB of RAM on a 16 GB host.

- **Investigation:** compared the host's free memory with the lab's actual needs (ASAv 2 GB, switches about 1 GB, VM overhead).

- **Root cause:** the hypervisor could not reserve 8 GB at once, and the lab did not need that much.

- **Fix:** reduced the VM to about 4GB.

- **Lesson:** size the VM from the lab's real requirements, and check free host memory before starting it.

### 2. IOU switches would not start (no IOU license)

- **Symptom:** the IOU switch nodes failed to start, reporting that no license could be found.

- **Investigation:** IOU images require a license file on the GNS3 VM, which was not present.

- **Root cause:** IOU images are licensed separately from the GNS3 setup.

- **Fix:** replaced the IOU switches with vIOS-L2 running as QEMU VMs, which need no license file. Interface names changed from Ethernet0/x to GigabitEthernet0/x.

- **Lesson:** check an image's licensing and runtime requirements before designing around it.

### 3. HSRP failover test showed no change

- **Symptom:** the continuous ping showed no loss, and show standby brief on R2 still showed it as Standby for groups 10 and 30.

- **Investigation:** checked HSRP state on R2 during the test.

- **Root cause:** R1 had not fully gone down, so R2 never needed to take over.

- **Fix:** took R1 fully down and repeated the test.

- **Lesson:** confirm the standby device actually became Active before trusting a failover result.

### 4. VPCS ping rejected ("Cannot resolve ip")

- **Symptom:** ping ip 10.3.0.10 returned "Cannot resolve ip".

- **Root cause:** the extra word ip was read as a hostname.

- **Fix:** ping 10.3.0.10.

- **Lesson:** VPCS uses ip only for setting addresses, never for ping.

## Phase 2: multi-vendor WAN

### 5. MikroTik rejected router-id and prompted for an instance

- **Symptom:** router-id= was not accepted in the BGP template, and later every add under /routing bgp connection prompted for instance:.

- **Investigation:** tried splitting the command, using Tab completion and checking the existing objects with print.

- **Root cause:** the running RouterOS release uses the instance-based BGP layout, and the commands written for the template layout did not match it.

- **Fix:** created or set a BGP instance (default, AS 65000) and referenced it with instance=default on each connection. The router ID was left automatic.

- **Lesson:** vendor CLIs change between releases. Check the running version first.

### 6. ISP loopback mistyped as 192.51.100.1

- **Symptom:** the automatic router ID and the loopback showed 192.51.100.1 instead of 198.51.100.1.

- **Investigation:** read /routing id print and /ip address print.

- **Root cause:** one digit was mistyped when the loopback address was added. The mistake was accepted silently.

- **Fix:** removed the address, re-added 198.51.100.1/32 on lo0, and corrected the address-list entry.

- **Lesson:** always verify with print after configuring. Accepted commands can still be wrong.

### 7. Duplicate router ID objects on the ISP

- **Symptom:** /routing id print showed several entries, including two named main and an unnamed duplicate.

- **Root cause:** the same /routing id add was run more than once, one time without a name.

- **Fix:** removed the unnamed duplicate. BGP does not depend on these objects, so the rest were left alone.

- **Lesson:** repeated add commands create duplicates. Check with print before re-running them.

### 8. R2 iBGP session stuck in Idle

- **Symptom:** R1 and R2 showed Idle with zero messages for the iBGP neighbor.

- **Investigation:** confirmed OSPF reachability between the loopbacks (route present, ping succeeded), then compared R2's show running-config | section router bgp against the design.

- **Root cause:** two errors on R2: remote-as 65000 instead of 65001, and a missing update-source Loopback0.

- **Fix:** removed and re-added the neighbor with remote-as 65001, update-source Loopback0 and next-hop-self.

- **Lesson:** Idle does not always mean no route. Read the neighbor's configuration on both ends.

### 9. R2 ISP-facing address mistyped as 172.168.2.1

- **Symptom:** R2's ISP session showed Idle with "never" for uptime, and a ping to 172.16.2.2 failed.

- **Investigation:** show ip interface brief showed Fa2/0 as up/up with the wrong address.

- **Root cause:** an extra 8 was typed in the address (172.168.2.1 instead of 172.16.2.1), putting the router on the wrong subnet.

- **Fix:** corrected the address to 172.16.2.1/30. The ping succeeded and the session established.

- **Lesson:** an interface that is up/up can still be misaddressed. Check the IP column, not just the status.

### 10. Switches booted the Aruba image instead of vIOS-L2

- **Symptom:** SW1 and SW2 appeared stuck at boot, then presented a switch login: prompt with the ArubaOS-CX banner, and PC1 could not reach its gateway.

- **Investigation:** the PuTTY window title said vIOS-L2 but the boot text was ArubaOS-CX. Both extracted packages used the same disk file name, virtioa.qcow2.

- **Root cause:** most likely the second upload overwrote the first, so the vIOS-L2 template pointed at the Aruba disk.

- **Fix:** gave each image a unique file name, repointed the templates, rebuilt SW1 and SW2 from the corrected template and restored their saved configurations.

- **Lesson:** rename image files before importing them, because GNS3 identifies disks by name.

### 11. Failover test timed out for about 30 seconds

- **Symptom:** after shutting R1's ISP interface, pings from PC1 failed for about 30 seconds before traffic recovered over R2.

- **Investigation:** R2's BGP table already had its direct ISP paths. The ISP still preferred R1's path for 10.1.0.0/16 until its hold timer expired.

- **Root cause:** only R1's end of the link went down. The ISP's end stayed up, so the ISP detected the failure only by hold-timer expiry (NOTIFICATION code 4).

- **Fix:** none needed. The result was documented. BGP timers on the HQ sessions had already been reduced to keepalive 10 s, hold time 30 s.

- **Lesson:** failover time depends on failure detection, not only on route policy. BFD would give faster detection in production.

### 12. ISP showed prefix-count 0 for R2's session

- **Symptom:** the ISP's session to R2 was Established but reported 0 prefixes, while the other sessions reported 1.

- **Investigation:** confirmed R2 was advertising 10.1.0.0/16 and that the ISP held R2's path as a candidate with the prepended AS path.

- **Root cause:** not determined. The counter later settled to 1 once the sessions had fully converged.

- **Fix:** none needed.

- **Lesson:** use the route table as the primary evidence and treat summary counters as supporting information.


### 13. FortiGate VM offered only DES for IPsec
- **Symptom:** `set proposal` on the FortiGate listed only des-* options.
- **Investigation:** checked vendor documentation and forum reports.
- **Root cause:** the unlicensed FortiGate VM runs in low-encryption mode.
- **Fix:** replaced the FortiGate with a MikroTik CHR as the VPN peer, which supports AES-256 without a license.
- **Lesson:** check a platform's crypto and license limits before designing a VPN around it.

### 14. Ping to 192.51.100.1 timed out from PC4
- **Symptom:** PC4 could not reach the ISP's simulated internet address.
- **Root cause:** the address was mistyped as 192.51.100.1 instead of 198.51.100.1.
- **Fix:** pinged the correct address.
- **Lesson:** double-check the address before troubleshooting the network.


### 15. Wireshark live capture from GNS3 failed
- **Symptom:** starting a capture on the link opened Wireshark with "End of file on pipe magic during open", or opened the interface list instead of the capture.
- **Root cause:** not determined. The live pipe between GNS3 and Wireshark closed before any data arrived.
- **Fix:** captured on the ASAv's outside interface instead.
- **Lesson:** devices with a built-in packet capture give an alternative when the lab tooling fails.
