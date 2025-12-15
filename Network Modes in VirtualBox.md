The "Attached to" setting defines which network your Virtual Machine's (VM) network adapter is connected to. Each mode has a distinct purpose, affecting isolation, connectivity, and security.



---

## 1. NAT (Network Address Translation)

This is the **most common and default** setting, offering a safe and isolated connection to the internet.

| Feature | Description |
| :--- | :--- |
| **What it does** | The VM can access the internet. The internet **cannot** access the VM by default. |
| **Characteristics** | The VM gets a private IP address (often in the $10.0.2.x$ range) that is hidden from the host network. |
| **Connectivity** | **Outbound** connections (VM to internet) are allowed. **Inbound** connections (Internet to VM) are blocked, unless specific **port forwarding** is configured. |
| **Use When** | You simply want internet access for the VM, or when you need network isolation. It is the default safe option. |

---

## 2. Bridged Adapter

The VM becomes a **real, independent machine** on your physical Local Area Network (LAN).

| Feature | Description |
| :--- | :--- |
| **What it does** | The VM is treated as if it were plugged directly into your router/switch. |
| **Characteristics** | The VM gets its own IP address directly from your physical router, just like the host machine. |
| **Connectivity** | **No NAT** is involved. Other devices on your LAN (and the internet, if configured) can reach the VM directly. |
| **Use When** | The VM should act like a server (e.g., a web server, file server) that needs to be directly accessible from other computers on your network. |
| **Security Note** | The VM is **fully exposed** to the network, just like any other computer. |

---

## 3. Host-only Adapter

Creates a private network **only** between the Host machine and the VM(s).

| Feature | Description |
| :--- | :--- |
| **What it does** | Sets up a network connection between the host operating system and the VM. |
| **Characteristics** | **No internet access** (unless combined with another adapter/NAT). The VM and host can communicate freely. |
| **Connectivity** | Other devices on your main LAN **cannot** see or access the VM. |
| **Use When** | For lab environments, safe testing, or management access. It is very common to use this as a second adapter alongside a NAT adapter for internet. |

---

## 4. Internal Network

Creates a completely isolated, private network shared **only** between select VMs.

| Feature | Description |
| :--- | :--- |
| **What it does** | Functions like a simple, isolated switch inside the hypervisor. |
| **Characteristics** | **No host access** and **no internet access**. |
| **Connectivity** | VMs configured with the same Internal Network name can communicate with each other. |
| **Use When** | Simulating enterprise network setups, creating firewall/router labs, or malware analysis labs where total isolation is critical. |

---

## 5. NAT Network (Different from simple NAT)

A variation of NAT that allows multiple VMs to share a single NAT connection **and** talk to each other.

| Feature | Description |
| :--- | :--- |
| **What it does** | Like NAT, but the NAT connection is shared and managed across multiple VMs. |
| **Characteristics** | VMs can talk to each other within the network. They are still NATed to the internet. |
| **Management** | Managed via the virtualization software's global settings (e.g., VirtualBox Global Tools). |
| **Use When** | Multi-VM setups that need both internet access and inter-VM communication (often a cleaner solution than plain NAT). |

---

## 6. Not Attached

The network interface card (NIC) exists in the VM configuration but is virtually "unplugged."

| Feature | Description |
| :--- | :--- |
| **What it does** | The NIC is present but inactive; there is no network connectivity. |
| **Use When** | Temporarily disabling network access for testing, taking snapshots, or during specific system configurations. |

---

## ⚡ Quick Comparison (Mental Shortcut)

| Mode | Internet Access | Host $\leftrightarrow$ VM | LAN $\leftrightarrow$ VM | VM $\leftrightarrow$ VM |
| :--- | :--- | :--- | :--- | :--- |
| **NAT** | Yes | Via Port Forward | No | No |
| **Bridged** | Yes | Yes | Yes | Yes |
| **Host-only** | No | Yes | No | Yes |
| **Internal** | No | No | No | Yes |
| **NAT Network** | Yes | Yes | No | Yes |
