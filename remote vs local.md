### 1\. When the destination is “local”

If the **destination IP** belongs to the same **subnet** (same network prefix when compared using the subnet mask), then:

-   Your computer sends **directly** to that host, **not** through the router.
    
-   It performs an **ARP request** on the LAN to find that host’s MAC address.
    
-   The Ethernet frame goes **straight from your NIC to the other host’s NIC** (switch just forwards it at Layer 2).
    

Example:

```nginx
Your IP:      192.168.1.10/24
Destination:  192.168.1.20
```

Both belong to `192.168.1.0/24`.  
So your computer says:

> “Same subnet → send directly.”

ARP:

```nginx
Who has 192.168.1.20? Tell 192.168.1.10
```

Then it sends the frame directly to that MAC address.

---

### 2\. When the destination is “remote”

If the destination IP is **not in your subnet**, the kernel knows it must go through the router (gateway).

Example:

```nginx
Your IP:      192.168.1.10/24
Destination:  8.8.8.8
```

Compare:

```
192.168.1.10 & 255.255.255.0 = 192.168.1.0
8.8.8.8      & 255.255.255.0 = 8.8.8.0
```

→ Different → not local → send to gateway.

Then your PC sends an **ARP request for the router’s IP (gateway)**:

```nginx
Who has 192.168.1.1? Tell 192.168.1.10
```

And sends the Ethernet frame to the router’s **MAC**.  
The **IP header’s destination** is still 8.8.8.8 — but the **Ethernet frame** is addressed to the router’s MAC.  
The router then forwards the packet toward 8.8.8.8 on its WAN side.

---

### 3\. Visual summary

```csharp
[Your PC 192.168.1.10] 
        |
        |  (direct frames for same subnet)
        |
[Other PC 192.168.1.20]
        |
        |  (router only used if outside subnet)
        |
[Router 192.168.1.1] → Internet
```

---

### 4\. So in short

-   **Local traffic**: sent directly (ARP → target MAC)
    
-   **Remote traffic**: sent via router (ARP → router MAC)
    

The **router interface** only participates when your kernel decides the destination is *outside* your subnet boundary as defined by your **subnet mask**.