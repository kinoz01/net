## What is ARP (Address Resolution Protocol)?

**ARP** is a communication protocol used for discovering the **Link Layer Address** (or **Media Access Control - MAC address**) associated with a known **Internet Layer Address** (typically an **IPv4 address**) on a local network segment (LAN).

In simpler terms:

* Devices on a network use **IP addresses** (Layer 3) to identify each other logically.
* But for actual data transmission on a local segment (like an Ethernet network), they need the **MAC address** (Layer 2), which is the unique physical address of the network interface card (NIC).
* **ARP is the protocol that translates the IP address into the corresponding MAC address.**

---

## How ARP Works (The ARP Process)

When one device (Host A) wants to send an IP packet to another device (Host B) on the **same local network**, here is the step-by-step process:

1.  ### **Check ARP Cache**
    * Host A first checks its local **ARP Cache** (or ARP Table). This is a temporary table maintained by the operating system that stores previously learned IP-to-MAC address mappings.
    * **If the mapping for Host B's IP address is found:** Host A uses the cached MAC address and sends the data packet directly.
    * **If the mapping is not found:** Host A must initiate an ARP request.

2.  ### **ARP Request (Broadcast)**
    * Host A crafts an **ARP Request** message. This message essentially asks, "Who has IP address $\text{X}$, and what is your MAC address?"
    * The ARP Request is encapsulated in an Ethernet frame with the following key addresses:
        * **Source MAC Address:** Host A's MAC address.
        * **Destination MAC Address:** The **broadcast MAC address** ($\text{FF:FF:FF:FF:FF:FF}$), which tells all devices on the local network to receive and process the frame.
        * **Target IP Address:** Host B's IP address ($\text{X}$).
    * The frame is broadcast onto the local network.

3.  ### **Processing the Request**
    * Every device on the local network segment receives and inspects the ARP Request.
    * Each device compares the **Target IP Address** in the request with its own IP address.
    * Devices with a non-matching IP address simply discard the request.
    * **Host B**, the device with the matching IP address, processes the request further.

4.  ### **ARP Reply (Unicast)**
    * Host B generates an **ARP Reply** message. This message contains its own MAC address, essentially saying, "I have that IP address, and my MAC address is $\text{Y}$."
    * The ARP Reply is encapsulated in an Ethernet frame with the following key addresses:
        * **Source MAC Address:** Host B's MAC address ($\text{Y}$).
        * **Destination MAC Address:** Host A's MAC address (which Host B learned from the incoming ARP Request).
    * The ARP Reply is sent directly (unicast) back to Host A.

5.  ### **Cache and Communicate**
    * Host A receives the ARP Reply and extracts Host B's MAC address ($\text{Y}$).
    * Host A then **adds the new IP-to-MAC mapping** ($\text{IP X} \rightarrow \text{MAC Y}$) to its own **ARP Cache** for future use.
    * Host A can now use the learned MAC address to send the original data packet to Host B.

---

## The ARP Cache

The ARP cache is vital for efficiency. It:

* **Stores mappings:** Keeps a record of resolved IP-to-MAC address bindings.
* **Reduces traffic:** Prevents a new ARP Request broadcast for every single packet, thus reducing network congestion.
* **Expiration:** Entries in the cache are typically **dynamic** and expire after a set time (e.g., 2-4 hours, depending on the OS) to account for devices changing their IP or MAC addresses. Entries can also be configured as **static** (manual entry) which do not expire.

---

## 🛡️ Security Concerns: ARP Spoofing

The ARP protocol itself has no built-in security, which makes it vulnerable to an attack called **ARP Spoofing** (or **ARP Cache Poisoning**):

* An attacker sends fake ARP Reply messages to other devices on the network.
* These fake replies intentionally link the attacker's MAC address to the IP address of a legitimate device (like the default gateway/router).
* Other devices update their ARP cache with the wrong information, causing them to send traffic intended for the router (or another host) to the attacker instead.
* This is a common technique used to facilitate **Man-in-the-Middle (MITM)** attacks, allowing the attacker to intercept, view, or modify all passing traffic.