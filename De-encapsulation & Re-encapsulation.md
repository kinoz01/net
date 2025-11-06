### **De-encapsulation**

-   This is the process of **removing headers and trailers** from a protocol data unit (PDU) as it moves **up the OSI layers**.
    
-   In the context of a **router**, it happens when the router **receives a frame** from one network.
    
-   The router **removes the Layer 2 (Data Link)** header and trailer to expose the **Layer 3 (IP) packet**.
    

👉 Think of it like **opening an envelope** to read the letter inside.

---

### **Re-encapsulation**

-   After processing the IP packet and deciding where to send it next, the router must **rebuild a new frame** to send it out.
    
-   It takes the IP packet and **adds a new Layer 2 header and trailer** that match the **outgoing network’s link layer** (for example, Ethernet, Serial, or PPP).
    

👉 It’s like **putting the same letter into a new envelope** to send it to the next destination.

---

## How the Router Does It (Step by Step)

1.  **Receive the frame** on an interface.
    
    -   The frame includes a **source MAC** and **destination MAC**.
        
2.  **De-encapsulation**
    
    -   The router **removes** the Layer 2 header and trailer.
        
    -   What remains is the **IP packet** (Layer 3).
        
3.  **Examine the IP packet**
    
    -   The router reads the **destination IP address**.
        
    -   It checks its **routing table** to decide where to forward the packet (which interface, and possibly a next-hop IP).
        
4.  **Find the next-hop MAC address**
    
    -   The router checks its **ARP table** for the MAC address that matches the next-hop IP.
        
    -   If not found, it sends an **ARP request** on the outgoing network to learn it.
        
    -   The ARP reply gives the **next-hop MAC address**, which the router caches.
        
5.  **Re-encapsulation**
    
    -   The router **wraps** the IP packet in a **new frame**:
        
        -   **Destination MAC:** the next-hop or destination host’s MAC.
            
        -   **Source MAC:** the router’s own outgoing interface MAC.
            
    -   It adds the **Layer 2 trailer** (e.g., Frame Check Sequence for Ethernet).
        
6.  **Forward the frame**
    
    -   The router sends the **new frame** out the outgoing interface toward the next device.
        

---

## In Simple Terms

> A router **de-encapsulates** a frame to look at the IP packet, then **re-encapsulates** it with a new frame header and trailer before forwarding it to the next hop.

-   **De-encapsulation:** remove old frame (incoming network)
    
-   **Re-encapsulation:** add new frame (outgoing network)
    
-   **ARP:** used to discover the MAC address for re-encapsulation

-> Also read the: [[The journey of an Ethernet Frame]]