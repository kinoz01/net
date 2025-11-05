## 1\. What a Subnet Mask Is

A **subnet mask** is a 32-bit number (in IPv4) that separates an IP address into two parts:

| Part | Meaning |
| --- | --- |
| **Network part** | Identifies which network (LAN, subnet) the address belongs to |
| **Host part** | Identifies the specific device (host) within that network |

So it’s like saying:

> “The first N bits tell me which street (network) I’m on,  
> the remaining bits tell me which house (host) I am on that street.”

---

## 2\. Example

IP address:

```
192.168.1.10
```

Subnet mask:

```
255.255.255.0
```

Convert both to binary:

```makefile
IP:   11000000.10101000.00000001.00001010
MASK: 11111111.11111111.11111111.00000000
```

Now we apply a **bitwise AND**:

```ruby
11000000.10101000.00000001.00001010
AND
11111111.11111111.11111111.00000000
=>
11000000.10101000.00000001.00000000
```

Result:  
**Network Address = 192.168.1.0**

The **host part** (the last 8 bits) belongs to devices in that LAN:

```
192.168.1.1 → 192.168.1.254
```

---

## 3\. Why It’s Called a “Mask”

Because it *masks out* the bits that represent the network.

Think of the mask as a stencil:

-   Wherever there’s a `1` → keep the IP bit (belongs to the network).
    
-   Wherever there’s a `0` → ignore the IP bit (belongs to the host).
    

That’s why it’s called **bit masking**.

---

## 4\. CIDR Notation (Compact Form)

Instead of writing the whole mask, we can write how many bits are 1s.

Example:

```
255.255.255.0 → /24
255.255.0.0   → /16
255.0.0.0     → /8
```

That’s called **CIDR (Classless Inter-Domain Routing)** notation.

So:

```
192.168.1.10/24
```

means “192.168.1.10, with the first 24 bits as network bits.”

---

## 5\. How Subnetting Works (Dividing Networks)

Subnetting is splitting one large network into smaller ones by **extending** the mask.

Example:

Original: `192.168.1.0/24`

-   256 addresses (0–255)
    
-   Mask: `255.255.255.0`
    

We want 4 subnets:  
→ add 2 more bits to network part: `/26` (255.255.255.192)

That gives 4 subnets:

| Subnet | Range | Host usable range |
| --- | --- | --- |
| 192.168.1.0/26 | 0–63 | 1–62 |
| 192.168.1.64/26 | 64–127 | 65–126 |
| 192.168.1.128/26 | 128–191 | 129–190 |
| 192.168.1.192/26 | 192–255 | 193–254 |

---

## 6\. Network, Broadcast, and Host Addresses

Every subnet has:

| Type | Description | Example (/24) |
| --- | --- | --- |
| **Network Address** | All host bits = 0 → identifies the subnet itself | 192.168.1.0 |
| **Broadcast Address** | All host bits = 1 → used to send to all devices | 192.168.1.255 |
| **Usable Host Range** | Anything between network and broadcast | 192.168.1.1–192.168.1.254 |

---

## 7\. Binary Breakdown Visualization

```yaml
IP Address: 192.168.1.10
Binary:     11000000 10101000 00000001 00001010

Mask:       11111111 11111111 11111111 00000000 (/24)

Network:    11000000 10101000 00000001 00000000
Host:                                  00001010
```

So the computer knows:  
“All devices with first 24 bits = 11000000.10101000.00000001 belong to my subnet.”

---

## 8\. What Happens in the Kernel (Routing Logic)

When your OS wants to send an IP packet:

1.  It checks the **destination IP**.
    
2.  It goes through the **routing table** entries, each with:
    
    -   Network address
        
    -   Subnet mask
        
    -   Interface
        
    -   Gateway (optional)
        
3.  It does:  
    `if (destination_ip & mask == network) → send via this interface`
    

Example route table:

```nginx
Destination     Gateway     Genmask         Iface
192.168.1.0     0.0.0.0     255.255.255.0   eno1
default         192.168.1.1 0.0.0.0         eno1
```

When sending to `192.168.1.55`:

```lua
192.168.1.55 & 255.255.255.0 = 192.168.1.0 → match → local network
```

When sending to `8.8.8.8`:

```perl
8.8.8.8 & 255.255.255.0 = 8.8.8.0 → no match → use “default route”
```

---

## 9\. Hardware & Electrical View

At Layer 2 (Ethernet):

-   The NIC doesn’t know about subnet masks.
    
-   It only sees **MAC addresses**.
    

But when the kernel decides that the destination IP is **outside the subnet**, it sends the packet to the **gateway’s MAC address** instead of the destination host.

**Example flow:**

1.  Packet to 8.8.8.8 → not in 192.168.1.0/24.
    
2.  Kernel checks routing table → next hop = 192.168.1.1 (router).
    
3.  Kernel uses **ARP** to get router’s MAC.
    
4.  NIC sends frame to that MAC.
    

So subnet masks drive the **decision to ARP or route**.

---

## 10\. Subnet Masks in Classes (Old Method)

Before CIDR (classless addressing), IPs were divided into fixed **classes**:

| Class | Range | Default Mask | Max Hosts |
| --- | --- | --- | --- |
| A | 1.0.0.0 – 126.255.255.255 | 255.0.0.0 (/8) | ~16M |
| B | 128.0.0.0 – 191.255.255.255 | 255.255.0.0 (/16) | ~65K |
| C | 192.0.0.0 – 223.255.255.255 | 255.255.255.0 (/24) | 254 |

This system wasted a lot of addresses → replaced by **CIDR** (flexible masks).

---

## 11\. Bitwise and Mathematical Representation

In pure binary math:

```ini
Network = IP & MASK
HostID = IP & ~MASK
```

Where:

-   `&` → bitwise AND
    
-   `~` → bitwise NOT (flip bits)
    

So:

```makefile
IP:   192.168.1.10  → 11000000.10101000.00000001.00001010
Mask: 255.255.255.0 → 11111111.11111111.11111111.00000000
~Mask:              → 00000000.00000000.00000000.11111111
```

Then:

-   Network = IP & Mask → 192.168.1.0
    
-   HostID = IP & ~Mask → 0.0.0.10 → host number 10
    

---

## 12\. How Subnet Mask Helps Communication Logic

When a device wants to communicate:

-   If **destination IP** matches its subnet → ARP directly (local communication)
    
-   Else → forward to router (gateway)
    

Example:

```nginx
Host A: 192.168.1.10/24
Host B: 192.168.1.20/24
→ Same subnet → ARP directly, exchange frames.

Host C: 192.168.2.15/24
→ Different subnet → send to default gateway.
```

So subnet masks define **boundaries of direct communication**.

---

## 13\. Visualization

```less
[192.168.1.0/24]         [192.168.2.0/24]
+-------------+           +-------------+
| Host A (.10)|           | Host C (.15)|
| Host B (.20)|           +-------------+
+-------------+
         \            /
        [Router 192.168.1.1 <-> 192.168.2.1]
```

Router connects the two subnets because the subnet masks tell the hosts:

> “Your network is 192.168.1.0/24, so 192.168.2.x is external — go via gateway.”

---

## 14\. Mental Model

-   Subnet mask = **boundary marker** for your local network.
    
-   It tells the OS which IPs are local and which are external.
    
-   At runtime, it controls:
    
    -   How packets are routed.
        
    -   Whether ARP is used.
        
    -   How routers and switches isolate networks.