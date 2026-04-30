# Lab 2: IPSec VPN - Written Explanation

A site-to-site IPSec VPN was configured between R1 (192.168.1.128/25) and R3 (192.168.3.128/25) to encrypt traffic traversing the untrusted WAN via R2. IKE Phase 1 uses an ISAKMP policy with AES-256 encryption, pre-shared key authentication, and DH group 14. IKE Phase 2 uses a transform set (esp-aes 256, esp-sha-hmac) with Perfect Forward Secrecy (PFS group 14). Crypto maps on both routers bind the peer address, transform set, and ACL 101 defining interesting traffic. OSPF routing was added across all routers to enable dynamic route advertisements.

This addresses the security problem of data confidentiality over untrusted networks. Without the VPN, traffic between the two sites crosses R2 (simulating the internet) in plaintext, exposing it to eavesdropping and man-in-the-middle attacks. The IPSec tunnel ensures all inter-site traffic is encrypted and authenticated, with the crypto map applied to the WAN-facing serial interfaces.

The VPN integrates with the existing ZPF on R3. An additional zone-pair (INTERNET_TO_INSIDE) was created to permit ICMP from the internet to the INSIDE zone, enabling IDS testing in Lab 3 while maintaining firewall protection.
