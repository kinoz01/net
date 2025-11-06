
Before DHCP or ARP can even occur, your device must first join the **local link** — meaning, it must be able to send Ethernet/Wi-Fi frames to the router or access point.

### a) **SSID Discovery**

When your Wi-Fi interface is idle, it scans nearby frequencies (channels) for **beacon frames** that access points (APs) broadcast every 100 ms or so.

Each beacon contains:

-   The **SSID** (network name),
    
-   The **BSSID** (MAC address of the AP),
    
-   Supported speeds,
    
-   Encryption type (WPA2, WPA3, Open, etc.).
    

Your computer’s OS lists these SSIDs as “available networks.”

### b) **Authentication and Association**

Once you choose one (say, `Home_WiFi`), the Wi-Fi handshake begins:

1.  **Authentication Request/Response** (open or WPA/WPA2 handshake).
    
2.  **Association Request/Response** — the AP accepts you as a station.
    
3.  **Encryption key exchange** (for WPA2/WPA3: the 4-way handshake).
    
4.  After success, your Wi-Fi interface and the AP share an **encryption key**, allowing you to send/receive encrypted 802.11 frames.
    

Now, at this point:

-   You are **logically connected** to the AP at Layer 2.
    
-   But you **still don’t have an IP address**.
    

If you’re using **Ethernet**, this phase is simpler:

-   The NIC senses the link (link-up signal).
    
-   There’s no SSID or encryption; physical connection = immediate link layer ready.
    

Now that your NIC can send frames to the AP (or Ethernet switch), the next step is to get a network configuration launching DHCP routine.