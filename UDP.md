### 1\. Overview: What UDP Is

UDP is a **Transport Layer protocol (Layer 4 in the OSI model)** defined in \[RFC 768\].  
It provides a **connectionless**, **unreliable**, **best-effort** data delivery service between applications using **IP networks**.

-   **Connectionless** → There’s no “handshake” before sending data (unlike TCP’s SYN/ACK).
    
-   **Unreliable** → No retransmission, no acknowledgment, no guarantee the data arrives.
    
-   **Best-effort** → The network stack tries to deliver, but errors or drops aren’t corrected.
    
-   **Low overhead** → Only 8 bytes of header per packet.
    
-   **Used when**: speed and simplicity matter more than reliability (DNS, VoIP, video, games).
    

---

### 2\. Relation to Other Layers

Let’s place UDP in the **networking stack**:

| Layer | Protocol | Function |
| --- | --- | --- |
| 7 | Application | DNS, DHCP, QUIC, VoIP |
| 4 | UDP | Multiplexes app data into datagrams |
| 3 | IP | Routes packets across networks |
| 2 | Ethernet | Delivers frames inside local network |
| 1 | Physical | Sends raw bits via electrical/optical/radio signals |

When an application uses UDP, it passes data to the **kernel** (transport layer), which adds:

-   UDP header → for ports and [[checksum]]
    
-   IP header → for routing
    
-   Ethernet header → for MAC addressing
    

Finally, the NIC converts it into **electrical or optical signals** and transmits it.

---

### 3\. UDP Datagram Structure (Header Format)

A UDP datagram has two parts: **Header (8 bytes)** + **Payload (variable length)**.

| Field            | Size    | Description                                 |
| ---------------- | ------- | ------------------------------------------- |
| Source Port      | 16 bits | Identifies sending process                  |
| Destination Port | 16 bits | Identifies receiving process                |
| Length           | 16 bits | Total length (header + data)                |
| [[Checksum]]     | 16 bits | Optional integrity check over header + data |

**Example:**

```diff
+-------------------+-------------------+
| Source Port       | Destination Port  |
+-------------------+-------------------+
| Length            | Checksum          |
+-------------------+-------------------+
|       DATA (payload) ...              |
+---------------------------------------+
```

#### [[Checksum calculation]]

-   Computed over the UDP header, data, and a **pseudo-header** (from IP layer: source IP, dest IP, protocol number, UDP length).
    
-   Ensures data integrity; 0 means no checksum (allowed in IPv4, not in IPv6).
    

---

### 4\. UDP Operation: Step-by-Step Path

Let’s trace what happens **when you send a UDP packet** (e.g., a DNS query).

#### 1\. Application Layer

Your program (e.g., `dig @8.8.8.8 example.com`) calls:

```c
sendto(socket_fd, buffer, len, 0, &dest_addr, sizeof(dest_addr));
```

This tells the kernel:

-   Destination IP = 8.8.8.8
    
-   Destination port = 53 (DNS)
    
-   Protocol = UDP
    

#### 2\. Transport Layer (Kernel UDP)

-   Kernel creates a **UDP header** with your source port (e.g., 54321) and destination port (53).
    
-   It computes the **checksum**.
    
-   It passes this to IP layer.
    

#### 3\. Network Layer (IP)

-   Adds an **IP header**: source IP, destination IP, protocol=17 (UDP).
    
-   Routes packet via the routing table (uses ARP for MAC resolution).
    

#### 4\. Data Link Layer (Ethernet)

-   Adds **Ethernet header** with destination and source **MAC addresses**.
    
-   Sends frame to **NIC (Network Interface Controller)**.
    

#### 5\. Physical Layer

-   NIC converts the frame into **electrical (Ethernet copper)** or **optical (fiber)** signals.
    
-   Signals propagate along the wire → switch → router → Internet.
    

#### 6\. Receiver Side

-   NIC at the other end receives the signal.
    
-   Ethernet layer strips Ethernet header → passes IP packet to IP layer.
    
-   IP layer verifies checksum and passes UDP datagram to UDP layer.
    
-   UDP layer checks port number:
    
    -   If port 53 has a socket listening (DNS daemon), deliver payload.
        
    -   If not, send back **ICMP “Port Unreachable”**.
        

---

### 5\. Kernel-Level Details

In Linux:

-   UDP sockets are represented by the `struct sock` in kernel memory.
    
-   UDP packets are processed by functions like:
    
    -   `udp_sendmsg()` → when user space sends.
        
    -   `udp_rcv()` → when packet is received from IP layer.
        
-   Data arrives in **socket buffers (sk\_buff)** — kernel memory structures that hold packet data and metadata.
    

When `recvfrom()` is called in user space:

-   The kernel checks the receive queue of that socket.
    
-   Copies payload (not header) to user space buffer.
    

---

### 6\. Hardware Involvement

At the **electronic level**:

-   The NIC has a **DMA engine** (Direct Memory Access) that copies frames directly from system memory to the hardware transmit buffer.
    
-   When a UDP datagram is sent:
    
    -   CPU writes packet data into RAM.
        
    -   NIC reads it via DMA.
        
    -   PHY (Physical Layer chip) converts bits to electrical or optical signals using encoding schemes (e.g., 8b/10b or PAM4).
        

On receive:

-   The PHY decodes electrical signals back to bits.
    
-   MAC layer in NIC reassembles frames.
    
-   NIC raises an **interrupt** or uses **polling (NAPI)** to tell the kernel data arrived.
    
-   Kernel places data in the UDP socket receive queue.
    

---

### 7\. Port Numbers and Multiplexing

-   Each UDP application uses a **port**.
    
    -   e.g., DNS → 53, DHCP → 67/68, NTP → 123, QUIC → dynamic.
        
-   The **tuple (src IP, src port, dst IP, dst port)** uniquely identifies a UDP flow.
    
-   Even though UDP has no connection state, the kernel maintains a small mapping table for routing replies.
    

---

### 8\. Performance and Limitations

**Advantages:**

-   Very low overhead.
    
-   Great for real-time apps (no retransmission delay).
    
-   Works well for broadcasting and multicasting.
    

**Limitations:**

-   No congestion control.
    
-   No retransmission or ordering.
    
-   Application must handle packet loss, duplication, or reordering itself (e.g., RTP, QUIC, custom logic).
    

---

### 9\. Real-World Use Cases

| Application | Why UDP? |
| --- | --- |
| DNS | Small, fast queries |
| DHCP | Works before IP is configured |
| VoIP (SIP/RTP) | Real-time, tolerate small loss |
| Video streaming | Better latency, not all frames must arrive |
| Gaming | Fast updates matter more than reliable delivery |
| QUIC/HTTP3 | Builds reliability & TLS over UDP for low latency |

---

### 10\. Example Packet (Hex View)

Example (UDP over IPv4):

```yaml
4500 0021 1c46 4000 4011 b861 c0a8 0102 c0a8 0101
c350 0035 000d 27c1 48656c6c6f
```

Breakdown:

-   `4500` → IPv4 header version & length
    
-   `c350` → source port 50000
    
-   `0035` → destination port 53
    
-   `000d` → length 13
    
-   `27c1` → checksum
    
-   `48656c6c6f` → “Hello”