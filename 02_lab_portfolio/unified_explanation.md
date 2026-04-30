# Unified Network Topology — All 5 Labs Integrated

The diagram `unified_network_diagram.png` presents the **composite defence-in-depth architecture** produced by layering all five lab controls onto the same physical topology. Each control operates at a different OSI layer and addresses a distinct threat category, so together they form overlapping protective measures rather than redundant ones.

## Layered Controls

- **Lab 1 — Zone-Based Policy Firewall (R3)**: enforces traffic policy between the INSIDE, CONFROOM, and INTERNET zones at Layer 3/4 using stateful inspection.
- **Lab 2 — IPSec Site-to-Site VPN (R1 ↔ R3)**: provides confidentiality and integrity for inter-site traffic across the untrusted core using IKEv1 with AES-256 and SHA-256 under DH group 14.
- **Lab 3 — Suricata IDS (Ubuntu VM)**: passively monitors the SPAN port mirror from S3 for malicious patterns — detection without throughput impact.
- **Lab 6 — Cowrie Honeypot (192.168.33.150)**: deliberately exposes fake SSH/Telnet services on the CONFROOM segment to capture attacker behaviour and generate high-fidelity alerts.
- **Lab 7 — Layer 2 Hardening (S2/S3)**: prevents CAM-flooding, ARP-spoofing, DHCP starvation, STP manipulation, and VLAN hopping at the access edge.

## Framework Alignment

The combined architecture reflects the **NIST Cybersecurity Framework's Protect and Detect functions** and embodies the principle that no single control is sufficient — defence must be layered.
