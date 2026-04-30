# Lab 7: Layer 2 Security Controls - Written Explanation

Layer 2 security controls were implemented across two Cisco 3750 switches (S3 and S2) to mitigate common switching attacks. On S3, DHCP snooping was enabled on VLANs 1 and 20, with Fa1/0/5 configured as the trusted port connecting to R3's DHCP server and Fa1/0/18 rate-limited to 10 packets/sec. Port security was configured on S3's Fa1/0/5 with a static MAC binding (001e.1336.4c41), and a violation test using a spoofed MAC (aaaa.bbbb.cccc) successfully triggered a shutdown violation. Trunk ports on both switches use 802.1Q with native VLAN 99 and DTP disabled (nonegotiate) to prevent VLAN hopping.

On S2, port security allows a maximum of 3 MAC addresses on Fa1/0/18 with 120-second aging. STP protections include loop guard globally, root guard on Gi1/0/1, BPDU guard and PortFast on access ports, and S3 configured as root bridge (priority 0). R3 was extended with DHCP pools and sub-interfaces for inter-VLAN routing (VLAN 20: 192.168.20.0/25, VLAN 99: management).

These controls address Layer 2 threats including MAC flooding, DHCP starvation, VLAN hopping, and STP manipulation, completing the defence-in-depth topology alongside the firewall, VPN, IDS, and honeypot.
