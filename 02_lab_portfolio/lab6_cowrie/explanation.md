# Lab 6: Cowrie Honeypot - Written Explanation

Cowrie, a medium-interaction SSH/Telnet honeypot, was deployed on the Ubuntu 24.04 VM within the CONFROOM subnet (192.168.33.150). Cowrie listens on port 2222 and emulates a Linux system (hostname "svr04"), logging all authentication attempts and commands executed by attackers. The R3 firewall was updated with a new zone-pair (INTERNET_TO_CONFROOM) and ACL permitting ICMP and SSH from the R1 site to the honeypot, simulating external attacker access. The CONFROOM class-map was expanded to include ICMP and TCP protocols.

Testing confirmed Cowrie captured a full SSH session: the attacker logged in as root, executed reconnaissance commands (pwd, ls, cat /etc/passwd, whoami), and the session was recorded in cowrie.json with event types including login.success, command.input, and session.closed. The cowrie_detect.py tool verified the honeypot scored 90.91% detection accuracy.

This addresses the security problem of lacking visibility into attacker behaviour. While firewalls and IDS detect and block threats, honeypots actively deceive attackers, capturing TTPs for threat intelligence. The honeypot integrates behind the ZPF perimeter alongside the existing VPN and IDS controls.
