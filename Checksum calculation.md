## 1\. The Core Idea

A **checksum** is a small number computed from a block of data that helps detect accidental corruption during transmission or storage.  
It answers: “Did this data arrive unchanged?”

In UDP, the checksum is **16 bits** and covers:

-   The UDP header
    
-   The UDP payload (your data)
    
-   A small “pseudo-header” from the IP layer (source IP, dest IP, etc.)
    

If even **one bit** flips (due to noise, faulty hardware, or driver bug), the checksum will likely differ and the packet will be dropped.

---

## 2\. Why Checksums Exist

When data travels through networks, it passes:

-   Cables (electrical or optical)
    
-   NIC buffers
    
-   CPU memory
    
-   Routers
    
-   Switches
    

Each step involves **voltage transitions, encoding, and buffering**, and any of them can introduce a **bit error**.

So each layer uses its own protection:

| Layer | Example | Error check |
| --- | --- | --- |
| L2 (Ethernet) | Frame CRC | Detects corruption on the wire |
| L3 (IP) | IPv4 header checksum | Detects corruption of header only |
| L4 (UDP/TCP) | UDP checksum | Detects corruption of header + payload |

This redundancy makes sure that if one layer misses an error, another can still catch it.

---

## 3\. The Mathematical Operation (One’s-Complement Sum)

UDP uses a **one’s-complement addition** of 16-bit words (defined in RFC 768).

### Step-by-step example

Suppose the data is:

```
0xABCD  0x1234  0xFFFF
```

We add them as 16-bit numbers using one’s-complement arithmetic:

1.  0xABCD + 0x1234 = 0xBE01
    
2.  0xBE01 + 0xFFFF = 0x1BE00  
    → overflow 1 bit beyond 16 bits → wrap around (add the carry):  
    0xBE00 + 0x1 = 0xBE01
    

Now **invert all bits (one’s complement)**:

```ini
Checksum = ~0xBE01 = 0x41FE
```

That 0x41FE is what gets stored in the UDP header’s checksum field.

---

## 4\. What Happens at the Receiver

When the receiver gets the packet, it does the **same calculation** (sum of all 16-bit words, including the checksum itself).

If there’s **no error**, the result will be all 1s:

```ini
Sum = 0xFFFF
```

If even one bit is flipped, the sum will **not** be 0xFFFF → packet is dropped.

---

## 5\. The Pseudo-Header (Extra Context)

Because UDP runs *on top* of IP, the UDP checksum covers more than just UDP data.  
It also includes a **pseudo-header** to verify that the packet wasn’t misdelivered between hosts.

Pseudo-header fields (not transmitted, used only for checksum):

| Field | Size | Description |
| --- | --- | --- |
| Source IP | 32 bits | Ensures source address is correct |
| Destination IP | 32 bits | Ensures destination address is correct |
| Zero | 8 bits | Always 0 |
| Protocol | 8 bits | 17 for UDP |
| UDP Length | 16 bits | Same as in UDP header |

So the checksum ensures:

-   Data integrity,
    
-   Correct source and destination,
    
-   Correct length,
    
-   Correct protocol.
    

---

## 6\. When UDP Checksum Is Optional

-   **IPv4:** Checksum value `0x0000` means “no checksum”.  
    (Allowed because Ethernet CRC already protects the frame.)
    
-   **IPv6:** Checksum **mandatory**.  
    IPv6 dropped the IP header checksum, so UDP must protect integrity.
    

---

## 7\. Hardware Acceleration

Modern NICs can calculate and verify checksums automatically.

When transmitting:

-   Kernel marks the packet as “needs checksum offload”.
    
-   NIC’s DMA engine reads the data, computes checksum in silicon (ALU inside NIC), and inserts it.
    

When receiving:

-   NIC verifies checksum before passing packet to CPU.
    
-   If correct → marks it “CHECKSUM OK” in metadata.
    
-   If wrong → drops packet or marks it invalid.
    

This saves CPU cycles — otherwise the kernel would have to sum potentially thousands of 16-bit words for every packet.

---

## 8\. Kernel Implementation (Linux)

Relevant kernel functions:

-   `csum_partial()` → sums bytes in software.
    
-   `udp_v4_check()` → finalizes UDP checksum.
    
-   `skb_checksum()` → handles sk\_buff (socket buffer) data.
    

When you call `sendto()` in user space:

1.  Kernel copies data into a socket buffer (`sk_buff`).
    
2.  It computes or defers checksum.
    
3.  Packet passes down the stack to NIC driver.
    
4.  Either the driver or NIC computes the checksum and inserts it.
    

---

## 9\. Electrical Level and Bit Errors

At the **physical level**, bits are transmitted as:

-   Voltage highs/lows (Ethernet),
    
-   Light pulses (fiber),
    
-   Radio signals (Wi-Fi).
    

A transient event (noise, crosstalk, cosmic ray, timing skew) can flip a bit — e.g., from 1→0.  
Checksums detect that by ensuring that the **sum of all 16-bit words plus checksum ≠ 0xFFFF**.

CRC (used by Ethernet) is stronger mathematically, but UDP’s checksum is lightweight — designed for CPU efficiency.

---

## 10\. Comparison: Checksum vs CRC

| Aspect | Checksum | CRC |
| --- | --- | --- |
| Algorithm | Addition + complement | Polynomial division |
| Complexity | Low (integer math) | Higher (bitwise polynomial) |
| Strength | Detects most 1–2 bit errors | Detects burst errors |
| Layer | IP/UDP/TCP | Ethernet, storage, protocols needing strong detection |

---

## 11\. Visualization

```pgsql
Data blocks (16-bit words):
+------+------+------+------+
| D1   | D2   | D3   | D4   |
+------+------+------+------+

One's complement sum = S
Checksum = ~S

Transmitted packet:
[D1][D2][D3][D4][Checksum]
Receiver computes sum again:
(D1 + D2 + D3 + D4 + Checksum) = 0xFFFF ? OK : DROP
```
