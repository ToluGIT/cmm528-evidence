# Lab 1: ZPF Firewall - Written Explanation

A Zone-Based Policy Firewall (ZPF) was configured on R3 to enforce traffic control across three security zones: INSIDE (Fa0/1 - 192.168.3.128/25), CONFROOM (Fa0/0 - 192.168.33.128/25), and INTERNET (S0/0/1 - 10.2.2.0/24). Two class-maps classify permitted traffic: INSIDE_PROTOCOLS allows TCP, UDP, and ICMP, while CONFROOM_PROTOCOLS restricts the conference room to HTTP, HTTPS, and DNS only. Corresponding policy-maps apply stateful inspection to matched traffic and drop everything else. Two zone-pairs (INSIDE_TO_INTERNET and CONFROOM_TO_INTERNET) enforce these policies directionally.

This addresses the security problem of unrestricted network access between segments. Without ZPF, any host could communicate freely across all interfaces. The stateful inspection ensures only traffic initiated from trusted zones is permitted, while return traffic is automatically allowed. The CONFROOM zone demonstrates least-privilege access by limiting users to web browsing only.

R3 acts as the perimeter firewall in the topology, separating internal networks from the WAN. This layered defence integrates with subsequent security controls including VPN encryption (Lab 2) and IDS monitoring (Lab 3).
