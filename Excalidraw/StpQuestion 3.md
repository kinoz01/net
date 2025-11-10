## 1\. Topology Overview

| Device                       | Interface     | IP                | MAC                            |
| ---------------------------- | ------------- | ----------------- | ------------------------------ |
| **PC**                       | LAN           | 192.168.1.10      | AA:AA:AA:AA:AA:AA              |
| **Home Router (LAN)**        | 192.168.1.1   | BB:BB:BB:BB:BB:BB | PC                             |
| **Home Router (WAN)**        | 203.0.113.5   | CC:CC:CC:CC:CC:CC | ISP Router (LAN)               |
| **ISP Router (LAN)**         | 203.0.113.1   | DD:DD:DD:DD:DD:DD | Home Router (WAN)              |
| **ISP Router (WAN)**         | 198.51.100.1  | EE:EE:EE:EE:EE:EE | Google Edge Router (LAN)       |
| **Google Edge Router (LAN)** | 198.51.100.2  | FF:FF:FF:FF:FF:FF | ISP Router (WAN)               |
| **Google Edge Router (WAN)** | 142.250.190.1 | HH:HH:HH:HH:HH:HH | Google internal server network |
| **Google Server**            | 142.250.190.4 | GG:GG:GG:GG:GG:GG | Google Edge Router (WAN)       |

---

## 2\. OUTBOUND — From PC to Google

### Step 1 – DNS Resolution

Your PC resolves `www.google.com → 142.250.190.4`.
(This DNS lookup normally takes the same process we will see here)

---

### Step 2 – PC sends the request

-   IP packet:
    
    ```nginx
    Src IP: 192.168.1.10
    Dst IP: 142.250.190.4
    ```
    
-   Routing table → default gateway = 192.168.1.1
    
-   **ARP on LAN:**  
    “Who has 192.168.1.1?” → “192.168.1.1 is at BB:BB:BB:BB:BB:BB.”
    

**Frame:**

```yaml
Ethernet:
  Src MAC: AA (PC)
  Dst MAC: BB (Home Router LAN)
IP:
  Src IP: 192.168.1.10
  Dst IP: 142.250.190.4
```

---

### Step 3 – Home router performs NAT

-   Removes Ethernet header.
    
-   **NAT:**  
    192.168.1.10:50500 → 203.0.113.5:50500
    
-   Routing: next hop = 203.0.113.1 (ISP router LAN)
    

**ARP on WAN:**  
“Who has 203.0.113.1?” → “203.0.113.1 is DD:DD:DD:DD:DD:DD.”

**Frame:**

```yaml
Ethernet:
  Src MAC: CC (Home Router WAN)
  Dst MAC: DD (ISP Router LAN)
IP:
  Src IP: 203.0.113.5
  Dst IP: 142.250.190.4
```

---

### Step 4 – ISP router forwards

-   Removes Ethernet header (DD…).
    
-   Looks up route for 142.250.190.4 → next hop 198.51.100.2 (Google Edge LAN).
    
-   **ARP (ISP WAN):**  
    “Who has 198.51.100.2?” → “198.51.100.2 is FF:FF:FF:FF:FF:FF.”
    

**Frame:**

```yaml
Ethernet:
  Src MAC: EE (ISP WAN)
  Dst MAC: FF (Google Edge LAN)
IP:
  Src IP: 203.0.113.5
  Dst IP: 142.250.190.4
```

---

### Step 5 – Google edge router forwards internally

-   Receives on LAN (FF…).
    
-   Route to internal server 142.250.190.4 → via WAN interface 142.250.190.1.
    
-   **ARP (Google Edge WAN):**  
    “Who has 142.250.190.4?” → “142.250.190.4 is GG:GG:GG:GG:GG:GG.”
    

**Frame:**

```yaml
Ethernet:
  Src MAC: HH (Google Edge WAN)
  Dst MAC: GG (Server)
IP:
  Src IP: 203.0.113.5
  Dst IP: 142.250.190.4
```

**Server receives packet.**

---

## 3\. INBOUND — Google → PC (response)

### Step 6 – Google server sends reply

```yaml
IP:
  Src IP: 142.250.190.4
  Dst IP: 203.0.113.5
Ethernet:
  Src MAC: GG (Server)
  Dst MAC: HH (Google Edge WAN)
```

---

### Step 7 – Google edge router forwards to ISP

-   **ARP (LAN side):** already knows ISP 198.51.100.1 → EE.
    
-   **Frame:**
    
    ```yaml
    Src MAC: FF (Google Edge LAN)
    Dst MAC: EE (ISP Router WAN)
    IP: Src 142.250.190.4, Dst 203.0.113.5
    ```
    

---

### Step 8 – ISP router forwards to Home router

-   **ARP (LAN side):** 203.0.113.5 → CC.
    
-   **Frame:**
    
    ```yaml
    Src MAC: DD (ISP LAN)
    Dst MAC: CC (Home Router WAN)
    IP: Src 142.250.190.4, Dst 203.0.113.5
    ```
    

---

### Step 9 – Home router performs reverse NAT

-   Arrives on WAN interface CC.
    
-   NAT table says:  
    203.0.113.5:50500 → 192.168.1.10:50500
    
-   **Rewrite:**
    
    ```makefile
    IP: Src 142.250.190.4 → Dst 192.168.1.10
    ```
    

---

### Step 10 – Router sends to PC (LAN)

-   **ARP:** “Who has 192.168.1.10?” → “192.168.1.10 is AA.”
    
-   **Frame:**
    
    ```yaml
    Src MAC: BB (Router LAN)
    Dst MAC: AA (PC)
    IP: Src 142.250.190.4, Dst 192.168.1.10
    ```
    

**PC receives response → application gets data.**

---

## 4\. Event Summary (Simplified Timeline)

| Stage | ARP | NAT | MAC Source → Dest | IP Source → Dest |
| --- | --- | --- | --- | --- |
| PC → Router | PC↔Router LAN | – | AA→BB | 192.168.1.10→142.250.190.4 |
| Router → ISP | Router↔ISP LAN | Outbound | CC→DD | 203.0.113.5→142.250.190.4 |
| ISP → Google Edge | ISP↔Google LAN | – | EE→FF | 203.0.113.5→142.250.190.4 |
| Google Edge → Server | Google Edge↔Server | – | HH→GG | 203.0.113.5→142.250.190.4 |
| Server → Google Edge | Server↔Edge | – | GG→HH | 142.250.190.4→203.0.113.5 |
| Google Edge → ISP | Edge↔ISP | – | FF→EE | 142.250.190.4→203.0.113.5 |
| ISP → Home Router | ISP↔Router | – | DD→CC | 142.250.190.4→203.0.113.5 |
| Router → PC | Router↔PC | Reverse | BB→AA | 142.250.190.4→192.168.1.10 |

---

## 5\. Key Concepts Recap

1.  **Each router has two sides (LAN & WAN)**, each with its own **IP and MAC**.  
    That’s why every hop uses different MAC pairs.
    
2.  **ARP** happens independently on every local link (LAN or WAN) to find the MAC of the next hop.
    
3.  **NAT** occurs only on your home router:
    
    -   Outbound: private → public (192.168.x → 203.x)
        
    -   Inbound: public → private
        
4.  **ISP and Google routers** only forward; they change **MAC headers**, not IP addresses.
    
5.  **IP headers remain stable** across routers after NAT — only Ethernet (MAC) headers are rewritten per hop.
    
6.  **The IP you see on ifconfig.me** = the router’s WAN IP (203.0.113.5) — the public identity of your entire LAN.