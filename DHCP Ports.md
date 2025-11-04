#### Port Numbers

| Protocol | Purpose | UDP Port (Server) | UDP Port (Client) | Notes |
| --- | --- | --- | --- | --- |
| **DHCPv4** | IPv4 address assignment | **67** | **68** | Based on the older BOOTP protocol |
| **DHCPv6** | IPv6 address assignment | **547** | **546** | Updated for IPv6 |

-   DHCP uses **UDP**, not TCP.
    
-   The **server listens** on its server port (67 or 547).
    
-   The **client sends** requests from its client port (68 or 546).
    

#### How to Verify on Your System

1.  **Check the `/etc/services` file (Linux/macOS)**  
    This file lists known ports.
    
    -   For IPv4 (DHCPv4):
        
        ```bash
        grep bootp /etc/services
        ```
        
        Output example:
        
        ```bash
        bootps    67/udp   # Bootstrap Protocol Server (DHCP)
        bootpc    68/udp   # Bootstrap Protocol Client (DHCP)
        ```
        
    -   For IPv6 (DHCPv6):
        
        ```bash
        grep dhcp /etc/services
        ```
        
        Output example:
        
        ```bash
        dhcpv6-client   546/udp
        dhcpv6-server   547/udp