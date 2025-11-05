## 1\. Big Picture: What an Ethernet Frame Is

An **Ethernet frame** is the **lowest-level data container** used to transmit information across a **local network (LAN)**.  
It exists at **Layer 2** of the **OSI model** — the **Data Link Layer**.

It’s the actual **structure of bits** that gets sent over the cable or Wi-Fi radio.

You can think of it like an *envelope* that carries higher-level data (like an IP packet) between two physical devices (identified by **MAC addresses**).
It's is the foundation of all network communication — everything from ARP, IP, TCP, UDP rides _inside_ Ethernet frames.

---

## 2\. Where It Fits in the Networking Stack

| OSI Layer | Function | Unit of Data |
| --- | --- | --- |
| 7 | Application | Message |
| 4 | Transport (TCP/UDP) | Segment / Datagram |
| 3 | Network (IP) | Packet |
| 2 | Data Link (Ethernet) | **Frame** |
| 1 | Physical (Cable, Fiber, Radio) | Bits (voltages/light/waves) |

So the **Ethernet frame** is what actually travels on the wire — it wraps around your IP packet.

---

## 3\. The Structure of an Ethernet Frame (IEEE 802.3)

A standard Ethernet frame has **14 bytes of header**, followed by data and a checksum (FCS).

```
| Preamble | SFD | Destination MAC | Source MAC | EtherType | Payload | FCS |
```

Let’s break each part.

---

### (1) Preamble — 7 bytes (56 bits)

-   Pattern: `10101010` repeated (alternating 1s and 0s).
    
-   Purpose: **Synchronize clocks** between sender and receiver PHY chips.
    
-   It helps the receiver’s hardware PLL (phase-locked loop) lock onto the bit timing.
    

### (2) Start Frame Delimiter (SFD) — 1 byte

-   Pattern: `10101011`
    
-   Signals “the frame starts right after this byte”.
    

### (3) Destination MAC Address — 6 bytes

-   Identifies the **recipient NIC** on the LAN.
    
-   Example: `00:11:22:33:44:55`
    
-   Can be:
    
    -   **Unicast** → one device
        
    -   **Broadcast** → all devices (`ff:ff:ff:ff:ff:ff`)
        
    -   **Multicast** → a group (starts with `01:00:5e...`)
        

### (4) Source MAC Address — 6 bytes

-   The **sender’s NIC hardware address**.
    

### (5) EtherType / Length — 2 bytes

-   If ≥ 0x0600 → EtherType = which protocol is inside.
    
    -   0x0800 → IPv4
        
    -   0x86DD → IPv6
        
    -   0x0806 → ARP
        
-   If ≤ 1500 → interpreted as “length” (used in IEEE 802.3 frames).
    

### (6) Payload — 46 to 1500 bytes

-   Contains the actual data (e.g., an IP packet).
    
-   Minimum 46 bytes (if smaller, padding is added).
    
-   Maximum 1500 bytes (MTU).
    

Example:  
For an IPv4 packet, the payload starts with the IP header.

### (7) Frame Check Sequence (FCS) — 4 bytes

-   A **CRC-32** checksum computed by the NIC.
    
-   Detects transmission errors caused by noise, voltage spikes, etc.
    
-   Receiver recomputes CRC — if it doesn’t match, the frame is **dropped silently** by hardware.
    

---

## 4\. Size Constraints

| Type | Bytes |
| --- | --- |
| Minimum Frame | 64 bytes (14 header + 46 payload + 4 FCS) |
| Maximum Frame | 1518 bytes (without VLAN tag) |
| Jumbo Frame | up to 9000 bytes (used in data centers) |

If a frame is shorter → NIC adds **padding bytes** to reach 64 bytes.

---

## 5\. Electrical / Physical View

At **Layer 1 (Physical)**:

-   Each bit in the frame is represented by **electrical voltage transitions** on twisted-pair copper or **light pulses** in fiber.
    
-   Encoding methods:
    
    -   10BASE-T → Manchester encoding (transition = bit)
        
    -   100BASE-TX → MLT-3 with 4B/5B encoding
        
    -   1000BASE-T → 4-level PAM-5 signaling
        
-   Each NIC has a **PHY chip** that modulates these signals.
    

Example flow:

1.  CPU writes frame bytes to NIC buffer via DMA.
    
2.  NIC serializes bits, adds preamble + FCS, sends them over the wire.
    
3.  Switch receives the signals, reconstructs bytes, checks FCS, forwards based on **MAC address table**.
    

---

## 6\. MAC Addresses and Switching

Every Ethernet NIC has a **unique 48-bit MAC address** burned into firmware.

Switches use these to decide **where to forward** frames:

| MAC Table Entry | Interface |
| --- | --- |
| 00:11:22:33:44:55 | Port 1 |
| 00:11:22:AA:BB:CC | Port 2 |

When a switch receives a frame:

-   Reads **destination MAC**.
    
-   If known → forwards to that specific port.
    
-   If unknown → floods to all ports (like broadcast).
    
-   If broadcast → floods to all ports by design.
    

This is how **local communication** (within the same subnet) stays fast and direct.

---

## 7\. How Frames Carry IP Packets

Let’s visualize layering:

```css
[Ethernet Frame]
 ├─ Ethernet Header (14 bytes)
 │   ├─ Dest MAC
 │   ├─ Src MAC
 │   ├─ EtherType: 0x0800 (IPv4)
 ├─ Payload (IP Packet)
 │   ├─ IP Header
 │   │   ├─ Src IP: 192.168.1.10
 │   │   ├─ Dst IP: 192.168.1.1
 │   ├─ Data (UDP/TCP segment)
 ├─ FCS
```

So when you `ping 192.168.1.1`, your OS sends:

1.  ARP (EtherType 0x0806) to get the router’s MAC.
    
2.  Then ICMP (inside IPv4, EtherType 0x0800) frame to that MAC.
    

---

## 8\. VLAN Tagging (802.1Q)

A VLAN adds **4 bytes** between the Source MAC and EtherType.

```css
[Dest MAC][Src MAC][TPID][TCI][EtherType][Payload][FCS]
```

-   TPID (Tag Protocol ID): `0x8100`
    
-   TCI (Tag Control Info): 12-bit VLAN ID + 3-bit priority + 1-bit DEI
    
-   VLAN lets switches **isolate traffic** into virtual LANs.
    

Example frame size with VLAN: 1522 bytes.

---

## 9\. Inside the Kernel (Linux)

In Linux:

-   Each frame received by NIC is placed in an `sk_buff` structure.
    
-   Drivers and the kernel’s network stack dissect:
    
    -   Ethernet header → identifies upper-layer protocol.
        
    -   IP header → source/destination IP.
        
    -   Transport layer → TCP/UDP port, etc.
        

**Key kernel functions:**

-   `eth_type_trans()` — determines protocol from EtherType.
    
-   `netif_rx()` — passes frame up to networking stack.
    
-   `dev_queue_xmit()` — sends frame out on an interface.
    

When you run:

```bash
tcpdump -i eno1 -e -XX
```

You’re literally seeing raw Ethernet frames captured before/after NIC processing.

---

## 10\. Hardware Involvement

NIC (Network Interface Controller):

-   Contains **MAC (Media Access Control)** logic.
    
-   Manages **TX (transmit) and RX (receive)** descriptor rings in RAM.
    
-   Handles:
    
    -   CRC generation & checking
        
    -   Padding & preamble insertion
        
    -   Interrupts for received frames
        
    -   DMA transfer to/from main memory
        

The CPU rarely touches individual bits — NICs offload most low-level work.

---

## 11\. Error Handling and Collisions

### In half-duplex Ethernet (old coaxial):

-   Used **CSMA/CD** (Carrier Sense Multiple Access with Collision Detection).
    
-   If two stations transmit simultaneously → collision → both stop and retry with backoff.
    
-   Modern full-duplex switched Ethernet no longer has collisions.
    

### Error detection:

-   Done by **FCS (CRC32)** — if mismatch → frame dropped.
    
-   No retransmission at this layer; upper layers (TCP) handle it.
    

---

## 12\. Example Hex Dump

Example Ethernet frame (captured):

```nginx
ff ff ff ff ff ff 00 1a a0 b1 c2 d3 08 06 00 01
08 00 06 04 00 01 00 1a a0 b1 c2 d3 c0 a8 01 64
00 00 00 00 00 00 c0 a8 01 01
```

Breakdown:

-   `ff ff ff ff ff ff` → broadcast destination
    
-   `00 1a a0 b1 c2 d3` → source MAC
    
-   `08 06` → EtherType = 0x0806 (ARP)
    
-   Rest → ARP packet payload (who-has query)
    

---

## 13\. Real Frame Flow in a Simple LAN

```lua
+---------+     +-----------+     +------------+
| Laptop  |<--->| Switch    |<--->| Router      |
|  MAC A  |     | MAC Table |     | MAC R       |
+---------+     +-----------+     +------------+

Laptop sends IP packet to 8.8.8.8:
→ Encapsulated in Ethernet frame:
   Dest MAC = Router MAC R
   Src MAC = Laptop MAC A
   EtherType = 0x0800 (IPv4)
→ Switch forwards based on MAC table.
→ Router decapsulates, forwards to WAN.
```

---

## 14\. Mental Model Summary

-   **Frame = physical packet** that travels on a wire.
    
-   Contains:
    
    -   MAC addresses (physical)
        
    -   EtherType (protocol)
        
    -   Payload (IP/TCP/UDP data)
        
    -   FCS (error detection)
        
-   Handled by:
    
    -   **NIC hardware** for transmission and CRC
        
    -   **Switches** for delivery inside LAN
        
    -   **Kernel drivers** for encapsulation and parsing
        

Everything you do — ping, HTTP, SSH — ends up being wrapped in one or more Ethernet frames before it leaves your machine.