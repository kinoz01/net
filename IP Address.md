## 1\. What an IP Address Really Is

An **IP address (Internet Protocol address)** is a **numerical identifier** assigned to every device connected to a network that uses the Internet Protocol for communication.  
It uniquely identifies a device’s **network interface** in order to send and receive data.

### Two Versions Exist

| Version | Example | Bit Length | Format |
| --- | --- | --- | --- |
| IPv4 | 192.168.1.10 | 32 bits | Four decimal bytes |
| IPv6 | 2001:0db8::1 | 128 bits | Hexadecimal groups |

We’ll focus first on **IPv4**, since it’s easier to visualize and the most widely used.

---

## 2\. IPv4 Structure: 32 Bits Split into Two Parts

An IPv4 address is made of **4 bytes (octets)** — 32 bits total.

Example:

```
192.168.1.10
```

→ Binary:

```
11000000.10101000.00000001.00001010
```

Every IPv4 address is logically split into:

-   **Network part**: identifies the subnet (street name)
    
-   **Host part**: identifies the device (house number)
    

The split point depends on the **subnet mask** (e.g., /24 → first 24 bits = network).

So 192.168.1.10/24 means:

-   Network = 192.168.1.0
    
-   Host = 10
    

---

## 3\. How an IP Address Is Used

When your computer communicates:

-   It sends packets with:
    
    -   **Source IP** = your address
        
    -   **Destination IP** = target address
        

Routers read only the **destination IP** to decide where to forward it.  
Each router checks which subnet it belongs to, using its routing table (via subnet masks).

---

## 4\. How IP Addresses Are Represented and Stored

Internally:

-   The OS and NIC see IPs as **binary 32-bit integers**.
    
-   Example: 192.168.1.10 → decimal 3232235786.
    
-   In C code (kernel level): stored in network byte order (big-endian).
    

---

## 5\. IP Address Assignment: Two Major Ways

### (A) **Static IP (Manual)**

You set it manually in your OS:

```makefile
Address: 192.168.1.10
Netmask: 255.255.255.0
Gateway: 192.168.1.1
DNS: 8.8.8.8
```

Your system then:

-   Binds that IP to your **network interface (NIC)**.
    
-   Adds the correct subnet and default route to its routing table.
    

Linux commands:

```bash
ip addr add 192.168.1.10/24 dev eno1
ip route add default via 192.168.1.1
```

Windows equivalent (GUI or PowerShell):

```nginx
netsh interface ip set address "Ethernet" static 192.168.1.10 255.255.255.0 192.168.1.1
```

### (B) **Dynamic IP (via DHCP)**

Most home and office networks use **DHCP (Dynamic Host Configuration Protocol)**.

When you connect your laptop or phone:

1.  It sends a **DHCPDISCOVER** broadcast (no IP yet).
    
2.  The DHCP server (often your router) replies with:
    
    -   **IP address lease**
        
    -   Subnet mask
        
    -   Default gateway
        
    -   DNS servers
        
3.  Your system **configures the IP** automatically and adds routes.
    

Linux command:

```bash
dhclient eno1
```

This is what happens silently when you plug into a network or join Wi-Fi.

---

## 6\. DHCP Detailed Lifecycle

| Step | Message | Direction | Purpose |
| --- | --- | --- | --- |
| 1 | DHCPDISCOVER | Client → Broadcast | Ask for available IP |
| 2 | DHCPOFFER | Server → Client | Suggest IP and config |
| 3 | DHCPREQUEST | Client → Server | Request chosen IP |
| 4 | DHCPACK | Server → Client | Confirm and lease IP |

This process uses **UDP ports 67 (server)** and **68 (client)**.

---

## 7\. Where the IP Lives in the OS

In Linux:

-   Each network interface has a kernel structure (e.g., `struct net_device`).
    
-   The IP address is stored in:
    
    -   `struct in_device` (IPv4)
        
    -   `struct inet_ifaddr` (list of addresses per interface)
        
-   When you run:
    
    ```sql
    ip addr show
    ```
    
    The kernel reads from these structures.
    

The address is also linked to a **routing table entry**:

```sql
192.168.1.0/24 dev eno1 scope link
default via 192.168.1.1 dev eno1
```

---

## 8\. What Physically Happens When You “Change” IP

Let’s say you change from `192.168.1.10` to `192.168.1.20`.

At a low level:

1.  The **kernel** deletes the old IP entry (`inet_del_ifa()`).
    
2.  It adds a new IP entry (`inet_insert_ifa()`).
    
3.  The **ARP cache** is cleared (MAC-to-IP table).
    
4.  The **NIC** continues to use the same MAC, but future outgoing packets use the new IP in the **source field** of IP headers.
    
5.  If DHCP was used, a **DHCPRELEASE** may be sent for the old address.
    

The new IP now defines which packets the system will accept as “mine” (destination).

---

## 9\. Link Between IP and MAC (ARP)

At the Ethernet level, devices talk with **MAC addresses**, not IPs.  
When sending to a local host, your system needs to know its **MAC**.

Example:

```css
Send packet to 192.168.1.20 → kernel checks subnet mask
→ Same network → ARP request:
"Who has 192.168.1.20? Tell 192.168.1.10"
```

→ The other host replies with its MAC (e.g., `00:11:22:33:44:55`).

That MAC is cached, and the NIC uses it for direct frame transmission.

So changing your **IP address** means your ARP cache must rebuild because your system’s “who’s who” changes.

---

## 10\. Changing IP Manually — Step by Step (Linux)

Example: computer interface `eno1`, you can find it either using Linux commands.

| Command | Purpose |
| --- | --- |
| `ip link show` | List interfaces |
| `ip addr show` | Show IPs bound to interfaces |
| `ip route` | Show which interface connects to the internet |
| `ifconfig` | Legacy command (still works) |

**Remove old address**

```bash
sudo ip addr del 192.168.1.10/24 dev eno1
```

**Add new one**

```bash
sudo ip addr add 192.168.1.20/24 dev eno1
```

**Change default gateway**

```bash
sudo ip route replace default via 192.168.1.1
```

Check:

```bash
ip addr show eno1
ip route show
```

You can also **renew via DHCP**:

```bash
sudo dhclient -r eno1  # release current lease
sudo dhclient eno1     # request new one
```

This can result in a completely different IP if the DHCP server assigns a new one.

---

## 11\. Changing IP Through Router or ISP (External/Public IP)

Your **local IP** (192.168.x.x or 10.x.x.x) is private.  
Your **public IP** (what websites see) is assigned by your **ISP** to your **router**.

Ways to change it:

1.  Reboot the router → DHCP lease may change.
    
2.  Change the router’s **MAC address** → many ISPs tie the IP lease to MAC.
    
3.  Request a new IP lease or wait until it expires.
    
4.  Use a VPN → creates a new *tunneled* virtual IP through another network.
    

---

## 12\. IPv6 Differences

IPv6 addresses are **128-bit** and can auto-configure without DHCP using **SLAAC (Stateless Address Autoconfiguration)**.

Example:

```ruby
2001:db8:abcd:0012::1/64
```

IPv6 automatically builds its IP using:

-   Router Advertisements (prefix)
    
-   Device’s MAC address (modified form)
    

You can still change it manually with:

```bash
ip -6 addr add 2001:db8::2/64 dev eno1
```

---

## 13\. Hardware and Kernel Path When Using the New IP

When a packet is sent after an IP change:

1.  **Application** writes data (socket).
    
2.  **Kernel transport layer** (TCP/UDP) wraps it with source IP (from routing table).
    
3.  **IP layer** checks if destination is local or remote (subnet mask).
    
4.  If remote → sends to **gateway MAC** ([[remote vs local]]).
    
5.  **Ethernet layer** encapsulates with MAC header.
    
6.  **NIC hardware** transmits physical signal (via DMA + PHY).
    

All these layers automatically pick up the **new IP** stored in the kernel’s `inet_ifaddr`.

---

## 14\. How to See Your IPs

**Local IP** (on LAN):

```bash
ip a
```

**Public IP** (external network):

```bash
curl ifconfig.me
```

---

## 15\. Summary

-   IP address = logical name for your NIC in a network.
    
-   Subnet mask defines what is “local.”
    
-   Gateway defines where “external” starts.
    
-   Changing IP means:
    
    -   Updating kernel routing structures.
        
    -   Possibly redoing DHCP negotiation.
        
    -   Resetting ARP and socket bindings.
        
-   At hardware level: only the bits in packet headers change — same NIC, same MAC, new logical identity.