## 1\. What `sysctl` actually does

**`sysctl`** is a command-line interface to read or modify **kernel parameters** at runtime.  
These parameters live inside a virtual filesystem called **`/proc/sys`** — part of the kernel’s memory, not a real disk.

So instead of editing kernel code or rebooting, you can dynamically change certain behaviors of the operating system (networking, memory management, process limits, security toggles, etc.) through `sysctl`.

---

## 2\. The basic syntax

```css
sysctl [options] [variable[=value]]
```

-   To **show** a value:
    
    ```bash
    sysctl net.ipv4.ip_forward
    ```
    
    Output:
    
    ```ini
    net.ipv4.ip_forward = 0
    ```
    
-   To **change** a value temporarily (until reboot):
    
    ```bash
    sudo sysctl -w net.ipv4.ip_forward=1
    ```
    
    Output:
    
    ```ini
    net.ipv4.ip_forward = 1
    ```
    

---

## 3\. How it works internally

Every kernel tunable parameter corresponds to a file under `/proc/sys/`.

For example:

-   `net.ipv4.ip_forward` → `/proc/sys/net/ipv4/ip_forward`
    
-   `vm.swappiness` → `/proc/sys/vm/swappiness`
    
-   `fs.file-max` → `/proc/sys/fs/file-max`
    

When you run `sysctl -w net.ipv4.ip_forward=1`, the tool literally writes `1` into `/proc/sys/net/ipv4/ip_forward`.

You can even do this manually:

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

(but `sysctl` is safer, easier, and ensures correct formatting.)

---

## 4\. Where the names come from

The variable name (`net.ipv4.ip_forward`) directly mirrors the directory tree under `/proc/sys/`.

Example:

```bash
/proc/sys/net/ipv4/ip_forward
         ↑   ↑     ↑
         │   │     └── variable
         │   └──────── subsystem (IPv4 networking)
         └──────────── category (network)
```

So when you see `kernel.`, `vm.`, `fs.`, or `net.`, it tells you which kernel subsystem the setting belongs to.

---

## 5\. What parameters can be tuned

There are **hundreds** of them, but they’re grouped roughly like this:

| Group prefix | Meaning / examples |
| --- | --- |
| `net.ipv4.*` | IPv4 networking (e.g. forwarding, ARP, TCP settings) |
| `net.ipv6.*` | IPv6 networking |
| `kernel.*` | Kernel behaviors (core dumps, panic behavior, sysrq) |
| `vm.*` | Virtual memory (swappiness, overcommit, dirty ratio) |
| `fs.*` | Filesystem and file descriptor limits |
| `dev.*` | Device-specific parameters |
| `user.*` | User namespace and limits |

To see all current settings:

```bash
sysctl -a
```

(very long list!)

---

## 6\. Temporary vs Permanent changes

### Temporary

Command-line or direct echo to `/proc/sys/`  
→ lasts until reboot.

Example:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

### Permanent

Write them into `/etc/sysctl.conf` or into a file under `/etc/sysctl.d/`.

Example:

```bash
sudo nano /etc/sysctl.conf
```

Add line:

```ini
net.ipv4.ip_forward=1
```

Then apply:

```bash
sudo sysctl -p
```

You can also use `/etc/sysctl.d/99-custom.conf` for your own configs (preferred on modern systems).

---

## 7\. Example: IP forwarding (your use case)

By default, Linux hosts do not forward packets between interfaces (they behave as simple endpoints).  
To make your router\_VM forward traffic:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

This writes `1` to `/proc/sys/net/ipv4/ip_forward`, enabling routing immediately.

To keep it after reboot:

```bash
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

## 8\. Useful options

| Option | Description |
| --- | --- |
| `-a` | List all current parameters and values |
| `-w` | Write a value (`sysctl -w key=value`) |
| `-p` | Load parameters from `/etc/sysctl.conf` (or specified file) |
| `-n` | Print only the value (no key name) |
| `-e` | Ignore unknown keys instead of erroring |
| `-q` | Quiet mode (no output on success) |
| `-r <regex>` | Filter output by regex |

Example:

```bash
sysctl -a | grep ipv4
```

---

## 9\. Common real-world examples

| Setting | Purpose |
| --- | --- |
| `net.ipv4.ip_forward=1` | Enable routing between interfaces (router behavior) |
| `net.ipv4.icmp_echo_ignore_all=1` | Ignore all pings (hide host) |
| `vm.swappiness=10` | Reduce swapping to disk |
| `fs.file-max=2097152` | Increase max open files system-wide |
| `kernel.sysrq=1` | Enable Magic SysRq key for emergency commands |
| `net.ipv4.conf.all.accept_redirects=0` | Harden against ICMP redirects |
| `net.ipv4.conf.all.rp_filter=1` | Enable reverse-path filtering (anti-spoofing) |

---

## 10\. Where sysctl fits in the Linux stack

Visual model:

```bash
┌──────────────────────────┐
│ user space (you)         │
│ ├─ sysctl command        │
│ ├─ echo > /proc/sys/...  │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ /proc/sys virtual FS     │  ← interface to kernel parameters
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Linux kernel             │  ← implements networking, VM, etc.
└──────────────────────────┘
```

So `sysctl` is simply a controlled, user-friendly way to talk to the kernel’s live configuration knobs.

---

## 11\. Quick recap

| Command | Effect |
| --- | --- |
| `sysctl -a` | Show all kernel tunables |
| `sysctl key` | Show one parameter |
| `sysctl -w key=value` | Change temporarily |
| `sysctl -p` | Reload `/etc/sysctl.conf` |
| Edit `/etc/sysctl.conf` | Make changes permanent |
| File mapping | `/proc/sys/net/ipv4/ip_forward` ↔ `net.ipv4.ip_forward` |

---

So in plain words:

> `sysctl` is how you **read or modify live kernel parameters**, which control how Linux behaves at a very low level — especially for networking, memory, and security. It acts as a gateway between your shell and the kernel’s runtime configuration stored under `/proc/sys/`.