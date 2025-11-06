### **1️⃣ On the Sender (PC A)**

When PC A sends data (say, a web request), the data is **encapsulated** step by step:

| OSI Layer | Example Protocol | What’s Added |
| --- | --- | --- |
| 7–5 | HTTP, TLS, etc. | Application data |
| 4 | TCP or UDP | TCP/UDP **header** (source/dest ports) |
| 3 | IP | IP **header** (source/dest IPs) |
| 2 | Ethernet | Frame **header + trailer** (MAC addresses + FCS) |
| 1 | Physical | Bits on the wire |

At the end, it’s just a stream of **bits** sent over the cable — but conceptually it looks like this:

```css
[Ethernet Header][IP Header][TCP Header][Application Data][Ethernet Trailer]
```

---

### **2️⃣ Inside the Network (Routers)**

Each **router** performs **partial de-encapsulation**:

-   The **Ethernet frame header/trailer** are **removed** → the router reads the **IP header**.
    
-   The **IP packet** is **kept intact** (the router doesn’t touch the TCP or application data).
    
-   The router decides where to forward the packet.
    
-   It **adds a new Ethernet header/trailer** (re-encapsulation) for the next hop.
    

So routers only strip and rebuild **Layer 2 information**, not IP or TCP headers.

---

### **3️⃣ At the Final Destination (Server B)**

Now, when the packet reaches Server B:

1.  **Ethernet layer (L2):**
    
    -   The NIC (Network Interface Card) checks if the destination MAC matches its own.
        
    -   The **Ethernet header and trailer** are removed.
        
    -   The **IP packet** is passed up to the **network layer**.
        
2.  **IP layer (L3):**
    
    -   The IP header is removed (de-encapsulation).
        
    -   The **payload** (usually TCP or UDP) is passed up to **Layer 4**.
        
3.  **Transport layer (L4):**
    
    -   TCP or UDP header is removed.
        
    -   The **application data** is handed off to the correct program (e.g., a web browser for HTTP).
        
4.  **Application layer (L7):**
    
    -   The OS or app reads the actual data (e.g., HTML from a webpage).