let’s explain **what `systemd-resolved` actually is**, what it does, and why it exists on your Linux system.

---

## 1\. The problem it solves

Before `systemd-resolved`, different programs and network managers each tried to manage DNS on their own:

-   NetworkManager wrote to `/etc/resolv.conf` directly.
    
-   VPN clients overwrote it.
    
-   Some services cached DNS locally, others didn’t.
    
-   No unified way to handle multiple interfaces (e.g., Wi-Fi + VPN).
    

Result: conflicts, race conditions, and broken DNS when switching networks.

So `systemd-resolved` was created to **centralize and standardize** DNS handling inside Linux systems that use `systemd`.

---

## 2\. What it actually is

`systemd-resolved` is a **daemon (background service)** that manages name resolution for your system.

It:

-   Listens locally on `127.0.0.53:53`
    
-   Receives DNS queries from applications
    
-   Forwards them to the correct upstream DNS server(s)
    
-   Caches answers to speed up repeated lookups
    
-   Integrates with DHCP, VPNs, and other interfaces
    

In other words, it’s your system’s **DNS broker**.

---

## 3\. How it works in practice

**Step-by-step:**

1.  When your computer connects to a network, DHCP gives it:
    
    -   an IP
        
    -   a gateway
        
    -   one or more DNS servers (e.g., `81.192.17.60`)
        
2.  NetworkManager or `systemd-networkd` tells `systemd-resolved` about that DNS server.
    
3.  `systemd-resolved` starts forwarding all DNS queries to that server.
    
4.  Applications don’t contact the server directly — they send queries to:
    
    ```
    127.0.0.53
    ```
    
5.  `systemd-resolved` takes care of:
    
    -   Choosing the right DNS per interface
        
    -   Caching results
        
    -   Supporting DNSSEC validation
        
    -   Handling fallback and split-DNS (VPN vs normal network)
        

---

## 4\. Files it manages

| File | Purpose |
| --- | --- |
| `/etc/resolv.conf` | Symlink to `/run/systemd/resolve/stub-resolv.conf` (points apps to 127.0.0.53) |
| `/run/systemd/resolve/resolv.conf` | Shows the *real* upstream DNS servers |
| `/etc/systemd/resolved.conf` | Configuration file (to set fallback DNS, DNSSEC, etc.) |

You can view current status with:

```bash
resolvectl status
```

and restart the service with:

```bash
sudo systemctl restart systemd-resolved
```

---

## 5\. Why you see `127.0.0.53` in `/etc/resolv.conf`

That’s the local “stub listener” address of `systemd-resolved`.  
It’s not an external DNS — it’s the **local endpoint** where your system’s resolver listens.

Internally, it forwards to whatever DHCP or static DNS servers are configured (like your ISP’s `81.192.17.60`).