## 🌐 Dynamic Host Configuration Protocol (DHCP) Explained

**DHCP** stands for **Dynamic Host Configuration Protocol**, a crucial network management protocol that simplifies and automates the process of assigning **IP addresses** and other necessary network configuration parameters to devices (hosts) on a network.

Without DHCP, a network administrator would have to manually configure an IP address, subnet mask, default gateway, and DNS server for every single device, a task that is time-consuming, prone to errors (like IP conflicts), and unmanageable on a large or dynamic network. DHCP eliminates this manual effort by maintaining a pool of addresses and dynamically leasing them to clients.

---

### Key Components of DHCP

The operation of DHCP involves three primary components:

* **DHCP Server:** This device (often a router or a dedicated server) holds the pool of IP addresses and configuration settings (called a **scope**). It's responsible for managing and allocating these settings to clients.
* **DHCP Client:** This is any device (computer, smartphone, printer, IoT device) that needs network configuration to communicate on the IP network. The client initiates the process to obtain its settings.
* **DHCP Relay Agent (Optional):** Used in networks with multiple subnets, this agent forwards DHCP messages between clients and servers that are not on the same local subnet, eliminating the need for a DHCP server on every subnet.

---

### How DHCP Works: The DORA Process

The process a DHCP client uses to obtain an IP address is a four-step sequence commonly referred to as **DORA**:

1.  **D**iscover:
    * The client, upon booting up or connecting to the network, sends a **DHCPDISCOVER** message as a broadcast (since it doesn't know the server's address) to locate any available DHCP servers.
    * *Source IP:* $0.0.0.0$ (no address yet).
    * *Destination IP:* $255.255.255.255$ (broadcast).

2.  **O**ffer:
    * Any DHCP server that receives the broadcast and has an available address responds with a **DHCPOFFER** message.
    * This message includes an *offered* IP address, a **subnet mask**, a **default gateway**, **DNS server** information, and a **lease duration** (the time the address is valid).
    * *Source IP:* Server's IP address.
    * *Destination IP:* $255.255.255.255$ (broadcast) or the client's MAC address.

3.  **R**equest:
    * The client selects one of the offers (if multiple were received) and sends a **DHCPREQUEST** message, broadcasting its acceptance to the chosen server (and implicitly declining any other offers).
    * This message formalizes the client's request for the specific offered IP address and configuration.

4.  **A**cknowledge:
    * The selected DHCP server finalizes the process by sending a **DHCPACK** (Acknowledgement) message.
    * This message confirms the configuration parameters and the **lease time**. The server then marks the IP address as officially leased in its database. The client can now configure its TCP/IP settings and fully participate in the network.

### ⏳ DHCP Lease and Renewal

The IP address assigned is not permanent; it's a **lease** with an expiration time.

* When the lease is about **50% expired**, the client automatically attempts to renew it by sending a unicast **DHCPREQUEST** directly to the leasing server.
* If the original server doesn't respond, the client waits until the lease is about **87.5% expired** and attempts to renew again.
* If the lease reaches **100% expiration** without renewal, the client releases the IP address and must start the **DORA** process over again with a new **DHCPDISCOVER** broadcast. This mechanism ensures IP addresses are reclaimed and reused when devices leave the network.

The following videos further details:
* Protocol: [Dynamic Host Configuration Protocol | How Does DHCP Work](https://www.youtube.com/watch?v=Wm7RtmUpZ94).
* Manual setup: https://www.youtube.com/watch?v=e6-TaH5bkjo
* Messages details: https://www.youtube.com/watch?v=4pkDL1pgCgQ
* 
