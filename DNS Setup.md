When your computer sends a **DHCP DISCOVER** at boot or when joining Wi-Fi, the DHCP server (usually your **router**) replies with a **DHCP OFFER** containing several *options*:

| Option name | Example value | Purpose |
| --- | --- | --- |
| `option 1` – Subnet Mask | 255.255.255.0 | Defines LAN range |
| `option 3` – Router / Gateway | 192.168.1.1 | Default gateway |
| `option 6` – DNS Servers | 81.192.17.60 | Tells clients what DNS server(s) to use |
| `option 51` – Lease Time | 86400 s | How long to keep this IP |

So **option 6** is what delivered **81.192.17.60** to you.

-   Sometimes routers instead give out their own address (192.168.1.1) and forward DNS internally.
    
-   In this case, the router simply passed along the **ISP’s real DNS** directly.

#### How Linux applied it

-   Your DHCP client (through NetworkManager or systemd-networkd) accepted the offer.
    
-   It told [[systemd-resolved]]:  
    “Use `81.192.17.60` as the upstream DNS for this interface.”
    
-   `systemd-resolved` stored that and began forwarding all lookups there.

### Commands used to find DNS setup

1.  Read what apps see as DNS  
    Command:
    
    ```bash
    cat /etc/resolv.conf
    ```
    
    What you saw:
    
    ```nginx
    nameserver 127.0.0.53
    ```
    
    Conclusion:  
    Your apps are configured to send DNS queries to a local stub at 127.0.0.53 ([[systemd-resolved]]), not directly to the router or ISP. This only tells us the local forwarder, not the real upstream DNS.
    
2.  Ask systemd-resolved who the real upstream DNS is  
    Command:
    
    ```lua
    resolvectl status
    ```
    
    Look for the “Current DNS Server” under the active link (e.g., wlan0 or enpXsY).  
    What you reported:
    
    ```nginx
    Current DNS Server: 81.192.17.60
    DNS Servers: 81.192.17.60
    ```
    
    Conclusion:  
    systemd-resolved is forwarding your queries to 81.192.17.60. That is the actual upstream DNS server being used.


***

> You can query `81.192.17.60` explicitly using:

```nginx
    dig @81.192.17.60 example.com +short
```

or you can query through [[systemd-resolved]] directly

```nginx
resolvectl query example.com
```

You should see the same IPs, but now routed internally via `127.0.0.53`, confirming that `systemd-resolved` forwards to `81.192.17.60` on your behalf.

Because you received proper replies, this tells us:

- The server 81.192.17.60 is reachable and speaking DNS (UDP port 53).

- It resolved `example.com` correctly.

- Therefore it is indeed an *operational DNS resolver*.