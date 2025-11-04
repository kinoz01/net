### Command

```bash
sudo dhclient -v <interface>
```

### 🧱 Full output

```nginx
Internet Systems Consortium DHCP Client 4.4.1
Copyright 2004-2018 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/wlx90de80d2526b/90:de:80:d2:52:6b
Sending on   LPF/wlx90de80d2526b/90:de:80:d2:52:6b
Sending on   Socket/fallback
DHCPREQUEST for 192.168.1.39 on wlx90de80d2526b to 255.255.255.255 port 67 (xid=0x35e86fea)
DHCPACK of 192.168.1.39 from 192.168.1.1 (xid=0xea6fe835)
Error: ipv4: Address already assigned.
bound to 192.168.1.39 -- renewal in 20791 seconds.
```

---
## 🔍 Line-by-line explanation

### **1️⃣ DHCP client banner**

```arduino
Internet Systems Consortium DHCP Client 4.4.1
```

This is just the version and copyright notice of the **`dhclient`** program (from ISC).

---

### **2️⃣ Listening/Sending on LPF**

```ruby
Listening on LPF/wlx90de80d2526b/90:de:80:d2:52:6b
Sending on   LPF/wlx90de80d2526b/90:de:80:d2:52:6b
```

-   `LPF` = **Link-Layer Packet Filter**, a Linux mechanism that lets `dhclient` directly send and receive Ethernet frames.
    
-   `wlx90de80d2526b` = your **wireless interface name**.
    
-   `90:de:80:d2:52:6b` = your **network card’s MAC address**.
    

So, it’s saying:

> “I’m using interface `wlx90de80d2526b` with MAC address `90:de:80:d2:52:6b` to send DHCP requests.”

---

### **3️⃣ Fallback socket**

```nginx
Sending on   Socket/fallback
```

This means the DHCP client also opens a **UDP socket** in case it can’t use the link-layer interface directly — a backup method. (You can ignore this; it’s standard.)

---

### **4️⃣ DHCPREQUEST**

```nginx
DHCPREQUEST for 192.168.1.39 on wlx90de80d2526b to 255.255.255.255 port 67 (xid=0x35e86fea)
```

This line tells us:

-   Your client is **requesting** the IP address `192.168.1.39`.
    
-   It’s sending the request as a [[broadcast]] to `255.255.255.255` (all DHCP servers on the network).
    
-   `port 67` is the DHCP server port ([[DHCP Ports]]).
    
-   `(xid=0x35e86fea)` is a **transaction ID** to match replies.
    

> Because it requested a specific address (`192.168.1.39`), this means it already had a previous lease and is trying to renew it rather than start from scratch.

---

### **5️⃣ DHCPACK**

```nginx
DHCPACK of 192.168.1.39 from 192.168.1.1 (xid=0xea6fe835)
```

-   The DHCP server (your **router**, `192.168.1.1`) **acknowledged** the request.
    
-   It confirms:
    
    > “Yes, you can continue using 192.168.1.39.”
    

So now the lease is renewed.

---

### **6️⃣ Error: ipv4: Address already assigned.**

```javascript
Error: ipv4: Address already assigned.
```

This line is **not actually a fatal error** — it’s informational.

It means:

> The IP address `192.168.1.39` was already configured on that interface, so the client didn’t need to assign it again.

In short:  
✅ You already had the same IP.  
🚫 The DHCP client just skipped reassigning it to avoid a duplicate configuration.

---

### **7️⃣ Lease bound & renewal time**

```nginx
bound to 192.168.1.39 -- renewal in 20791 seconds.
```

This means:

-   Your DHCP lease is now **active (“bound”)**.
    
-   You’re using IP `192.168.1.39`.
    
-   The client will automatically try to **renew** the lease after `20791` seconds (≈ **5 hours, 46 minutes**).
    

If the renewal succeeds, you keep your IP.  
If not, it will retry or request a new IP.