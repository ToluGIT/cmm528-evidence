# Lab 3: Suricata IDS - Written Explanation

Suricata IDS was deployed on an Ubuntu 24.04 VM to passively monitor the INSIDE network segment (192.168.3.128/25). A SPAN port on S3 mirrors all traffic from FastEthernet1/0/5 (connected to R3's INSIDE interface) to FastEthernet1/0/24 (connected to the VM's second NIC). Suricata was configured with HOME_NET set to the INSIDE subnet and monitors the mirrored interface (ens37) in af-packet mode. Two custom rules were created in local.rules: one to detect ICMP ping traffic (sid:1000001) and another to detect TCP port scans using a threshold of 5 SYN packets in 60 seconds (sid:1000002).

This addresses the security problem of undetected malicious activity within the network. Firewalls and VPNs control access but cannot identify suspicious behaviour in permitted traffic. The IDS provides visibility into network activity, generating alerts logged to both fast.log and eve.json for analysis.

The IDS integrates behind the ZPF perimeter, monitoring traffic that has already passed through the firewall. The R3 ACL was updated to allow ICMP from the INTERNET zone to the INSIDE zone to enable testing, demonstrating how the firewall and IDS work as complementary layers of defence.
