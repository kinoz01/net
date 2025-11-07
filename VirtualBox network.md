# 1\. What VirtualBox networking really is

VirtualBox creates **virtual Network Interface Cards (vNICs)** inside your virtual machines.  
Each of those vNICs connects to some **backend** — a piece of VirtualBox code that defines how traffic leaves the VM and where it goes (other VMs, host, Internet, etc.).

From the VM’s perspective, it just has a normal NIC like Intel PRO/1000, PCnet, or virtio — but VirtualBox captures and handles all its Ethernet frames in software.

So when you create a VM, each “Adapter” you enable (Adapter 1–4) corresponds to a **virtual Ethernet card** with its own MAC address.

---

# 2\. The VirtualBox Network Modes

VirtualBox offers **8 networking modes**. The first 5 are the most commonly used:

| Mode | Description | Internet Access | Host ↔ VM | VM ↔ VM |
| --- | --- | --- | --- | --- |
| **NAT** | VM connects to Internet through host via VirtualBox’s built-in NAT router | ✅ | ❌ (unless port forward) | ❌ |
| **NAT Network** | Like NAT, but multiple VMs share one NAT router (they can talk to each other) | ✅ | ❌ | ✅ |
| **Bridged Adapter** | VM connects directly to the same network as the host (like another physical machine) | ✅ | ✅ | ✅ |
| **Internal Network** | VMs can talk only to each other on a named internal switch | ❌ | ❌ | ✅ |
| **Host-Only Adapter** | VMs can talk to host and other VMs, but not Internet | ❌ | ✅ | ✅ |
| **Generic Driver** | Special: used for very custom link types (UDP Tunnel, VDE, etc.) | custom | custom | custom |
| **Not Attached** | Adapter exists but is unplugged | ❌ | ❌ | ❌ |
| **Disabled** | No adapter at all | ❌ | ❌ | ❌ |

---

# 3\. Let’s explain each one in full detail

---

## (1) **NAT (Network Address Translation)**

**Default mode for new VMs.**

VirtualBox acts as a **mini-router and NAT gateway** for the VM.

### The VM sees:

-   IP: `10.0.2.15`
    
-   Gateway: `10.0.2.2`
    
-   DNS: `10.0.2.3`
    

### Internally:

```nginx
VM → VirtualBox NAT Engine → Host’s physical NIC → Internet
```

VirtualBox does all the NAT translations itself — your host doesn’t need special configuration.

**Advantages:**

-   Works out of the box
    
-   Internet access guaranteed
    
-   Secure (VM invisible to LAN)
    

**Limitations:**

-   No inbound connections (can’t access the VM from host or LAN)
    
-   Each VM is isolated (unless you use NAT Network)
    

**Port forwarding example:**  
If you want host → VM access:

```bash
VBoxManage modifyvm "MyVM" --natpf1 "ssh,tcp,,2222,,22"
```

This maps host port 2222 → guest port 22.

---

## (2) **NAT Network**

Like NAT, but multiple VMs share one NAT gateway.

You can define your own NAT network (via GUI or command):

```bash
VBoxManage natnetwork add --netname labnet --network "192.168.100.0/24" --enable
```

Then attach multiple VMs to it.

Now VMs can:

-   Reach Internet (through host)
    
-   Talk to each other (same subnet)
    
-   Remain invisible to host and LAN
    

Think of this as a **private LAN behind a VirtualBox router**.

---

## (3) **Bridged Adapter**

This connects the VM **directly to your physical LAN**, as if it were another real computer plugged into your switch or Wi-Fi.

```arduino
VM → VirtualBox Bridge → Host NIC → Physical LAN → Router → Internet
```

### What happens:

-   VirtualBox uses a **host-level bridge driver** to bind VM traffic to your physical NIC.
    
-   The VM gets an IP from your LAN’s DHCP server (usually your home router).
    
-   It appears on your network like a separate device (with its own MAC address).
    

**Advantages:**

-   VM fully visible to LAN (you can SSH, ping, etc.)
    
-   Perfect for server or pentesting setups
    
-   Works with all standard networking tools
    

**Limitations:**

-   Requires physical NIC to support promiscuous mode
    
-   Sometimes blocked on Wi-Fi (some drivers restrict bridging)
    
-   Less isolation
    

---

## (4) **Internal Network**

This mode creates a **completely isolated virtual LAN**.

```csharp
VM1 ↔ VM2 ↔ (VirtualBox internal switch)
```

-   No host access
    
-   No Internet
    
-   No DHCP (unless you provide your own)
    

Used for:

-   Lab environments
    
-   Router/firewall experiments
    
-   Malware sandboxing
    

Each internal network has a **name**, e.g., “intnet”.  
Only VMs with the same name share the same switch.

**Example use:**  
Router\_VM (Internal + NAT) ↔ Client\_VM (Internal only)

---

## (5) **Host-Only Adapter**

This creates a **virtual network between your host and the VMs**, with no Internet access.

When you install VirtualBox, it creates:

-   A virtual interface on the host (`vboxnet0`)
    
-   A virtual switch that connects host and all VMs on host-only mode
    

```nginx
Host ↔ vboxnet0 ↔ VMs
```

**Typical IP range:** `192.168.56.0/24`

**Use cases:**

-   File transfer between host and VMs
    
-   Testing client-server setups locally
    
-   Combining with NAT for Internet + local host access
    

**Hybrid setup example:**

-   Adapter 1: NAT → Internet
    
-   Adapter 2: Host-Only → Access from host
    

---

## (6) **Generic Driver**

For very advanced uses — you can connect VirtualBox networks to external tools like:

-   **VDE (Virtual Distributed Ethernet)** on Linux
    
-   **UDP tunnels** for connecting labs across systems
    

Rarely used outside specialized network research or multi-host simulations.

---

## (7) **Not Attached**

The VM still “has” a NIC, but it’s **unplugged** — like pulling the Ethernet cable out.

Useful for temporarily disabling networking while preserving config.

---

## (8) **Disabled**

No virtual NIC at all — completely offline VM.

---

# 4\. How VirtualBox connects to the host OS

Every time you create a VM adapter, VirtualBox registers it with the **host’s network stack** using kernel modules (on Linux) or NDIS drivers (on Windows).

-   On **Linux hosts**: modules like `vboxnetflt`, `vboxdrv`, `vboxnetadp` create virtual interfaces and handle packet redirection.
    
-   On **Windows hosts**: a low-level NDIS filter driver hooks into your NIC.
    
-   On **macOS hosts**: similar kexts perform bridging or NAT internally.
    

So when your VM sends an Ethernet frame, VirtualBox intercepts it before it reaches the physical NIC and decides what to do with it, based on the mode.

---

# 5\. How NAT and bridging work under the hood

### NAT:

-   VirtualBox maintains a hidden router with IP `10.0.2.2`
    
-   It rewrites packet source addresses to match the host’s IP
    
-   It handles return traffic using NAT tables
    

### Bridged:

-   VirtualBox uses a network filter driver that copies Ethernet frames directly between VM and host NIC
    
-   The host NIC enters **promiscuous mode**, accepting traffic for the VM’s MAC
    
-   The VM’s traffic physically appears on your LAN
    

---

# 6\. Multiple adapters per VM

Each VM can have up to **4** (GUI) or **8** (CLI) network interfaces.  
Each can be attached to a different network mode.

Example: router lab

```yaml
Adapter 1: NAT (Internet)
Adapter 2: Internal Network (LAN)
Adapter 3: Host-Only (for management)
```

---

# 7\. VirtualBox’s internal services that make it work

| Component | Purpose |
| --- | --- |
| **vboxdrv** | Main kernel module (controls VM processes and devices) |
| **vboxnetflt** | Bridge / NAT filter module — attaches to host NIC |
| **vboxnetadp** | Host-only adapter manager (creates vboxnet0, vboxnet1, etc.) |
| **VBoxNetDHCP** | Built-in DHCP server for NAT/host-only networks |
| **VBoxManage** | Command-line tool to inspect, create, and modify virtual networks |

---

# 8\. The typical packet travel paths (per mode)

| Mode | Path |
| --- | --- |
| **NAT** | VM → VirtualBox NAT → Host NIC → Router → Internet |
| **Bridged** | VM → VirtualBox bridge → Host NIC → LAN/Router → Internet |
| **Host-Only** | VM → VirtualBox switch → Host (no Internet) |
| **Internal** | VM → VirtualBox switch → VM (no Host/Internet) |
| **NAT Network** | VM → VirtualBox NAT router → Host NIC → Internet (shared among VMs) |

---

# 9\. DHCP servers per mode

| Mode | Who gives IP |
| --- | --- |
| **NAT** | VirtualBox internal DHCP (10.0.2.0/24) |
| **NAT Network** | VirtualBox’s NAT DHCP (custom range) |
| **Bridged** | Your LAN’s DHCP (router) |
| **Host-Only** | VirtualBox DHCP (192.168.56.0/24, configurable) |
| **Internal** | None (you must run your own DHCP) |

---

# 10\. Layered diagram (simplified)

```css
┌────────────────────────────┐
          │          Internet          │
          └────────────┬───────────────┘
                       │
               ┌───────┴────────┐
               │  Host Physical │
               │  NIC (eth0)    │
               └───────┬────────┘
                       │
             (VirtualBox Driver Layer)
                       │
 ┌──────────────────────────────────────────────────────────────┐
 │                  VirtualBox Networking Core                  │
 │ ┌──────────────┬───────────────┬──────────────┬────────────┐ │
 │ │ NAT Engine   │ Bridged Engine│ Host-Only Net│ IntNet Sw  │ │
 │ └────┬─────────┴──────┬────────┴──────┬────────┴──────┬────┘ │
 │      │                │               │               │      │
 │  NAT 10.0.2.0/24   Bridged LAN    vboxnet0 (192.168.56.0)  intnet │
 └──────┼────────────────┼───────────────┼───────────────┼───────┘
        │                │               │               │
   [VM1: NAT]       [VM2: Bridged]   [VM3: Host-only]  [VM4: Internal]
```

---

# 11\. Mixing modes (common real setups)

| Goal | Adapter setup |
| --- | --- |
| Internet-only client | Adapter 1: NAT |
| LAN-visible server | Adapter 1: Bridged |
| Host-accessible lab with Internet | Adapter 1: NAT, Adapter 2: Host-Only |
| Router lab | Adapter 1: NAT, Adapter 2: Internal |
| Completely isolated network simulation | Adapter 1: Internal |

---

# 12\. Summary Table

| Mode | Internet | Host access | VM ↔ VM | DHCP | Typical Use |
| --- | --- | --- | --- | --- | --- |
| **NAT** | ✅ | ❌ | ❌ | ✅ | Simple client Internet |
| **NAT Network** | ✅ | ❌ | ✅ | ✅ | Small internal subnet with Internet |
| **Bridged** | ✅ | ✅ | ✅ | via LAN | Full LAN integration |
| **Host-Only** | ❌ | ✅ | ✅ | ✅ | Local testing with host |
| **Internal** | ❌ | ❌ | ✅ | ❌ | Isolated lab |
| **Generic** | custom | custom | custom | custom | Advanced research |

---

# 13\. Key takeaways

-   VirtualBox provides **its own complete virtual Ethernet system** inside your computer.
    
-   Each network mode defines *how those virtual cables connect* to your host and the outside world.
    
-   NAT and Bridged are for Internet access.
    
-   Host-Only and Internal are for isolation and local labs.
    
-   NAT Network gives a balance: shared Internet but still private.
    
-   All packet forwarding, ARP, DHCP, and NAT are simulated in software inside the host process — not by your real NIC.



![[3431d2e3-87fc-4551-bcaa-3a44c5581002.png]]