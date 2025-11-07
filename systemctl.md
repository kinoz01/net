# 1\. The full command:

```nginx
sudo systemctl restart networking
```

We can split it into 3 major pieces:

| Part | Meaning |
| --- | --- |
| `sudo` | Run the following command as the superuser (root), because changing network interfaces requires admin rights. |
| `systemctl` | The control tool for **systemd**, which manages all system services (daemons). |
| `restart networking` | Tells `systemd` to restart the **networking service** defined in its service file (`networking.service`). |

---

# 2\. What `systemctl` is

`systemctl` is the **front-end command** to control `systemd` — the init system used by most modern Linux distros (Debian, Ubuntu, Kali, etc.).  
It replaced the old `service` command from SysVinit.

`systemctl` can:

-   start, stop, restart, or reload system [[services]]
    
-   enable or disable them at boot
    
-   check their status
    
-   manage system targets (like runlevels)
    

**Examples:**

```bash
systemctl start apache2      # Start the Apache service
systemctl stop ssh           # Stop the SSH service
systemctl status networking  # Show network service status
systemctl enable ssh         # Auto-start SSH at boot
```

---

# 3\. What is `networking.service`

This is a **systemd service unit** that controls the legacy Debian/Ubuntu network manager called **ifupdown**, which reads configuration from `/etc/network/interfaces`.

You can see its definition with:

```bash
systemctl cat networking
```

It looks roughly like this:

```ini
[Unit]
Description=Raise network interfaces
After=local-fs.target

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/sbin/ifup -a
ExecStop=/sbin/ifdown -a

[Install]
WantedBy=multi-user.target
```

---

# 4\. What happens internally when you run `systemctl restart networking`

Let’s go step by step:

### Step 1 — systemd receives the request

`systemctl` tells systemd: “restart the unit named `networking.service`”.

### Step 2 — systemd executes the “stop” phase

-   It runs `/sbin/ifdown -a`, which means:
    
    > “Take down all interfaces defined in `/etc/network/interfaces` that are marked `auto` or `allow-hotplug`.”
    

This deactivates each interface by:

-   Releasing IP addresses (DHCP release)
    
-   Bringing the link down
    
-   Flushing routes
    
-   Removing the interface configuration
    

You can see this manually if you run:

```bash
sudo ifdown enp0s3
```

### Step 3 — systemd executes the “start” phase

-   It runs `/sbin/ifup -a`, which means:
    
    > “Bring up all interfaces defined in `/etc/network/interfaces` that are marked `auto`.”
    

This reads `/etc/network/interfaces`, then:

-   Applies static IPs (if defined)
    
-   Requests IPs from DHCP servers (if `inet dhcp`)
    
-   Configures routes and DNS settings
    
-   Activates each interface (via `ip link set up` internally)
    

---

# 5\. So effectively, this command:

```nginx
sudo systemctl restart networking
```

is equivalent to:

```css
sudo ifdown -a && sudo ifup -a
```

It’s just managed through **systemd** instead of you calling `ifupdown` directly.

---

# 6\. When do you use it?

You run it whenever you:

-   Edit `/etc/network/interfaces` (to apply changes without rebooting)
    
-   Change IP, gateway, or DNS settings manually
    
-   Need to reinitialize a failed or disconnected interface
    

It’s **not** needed if you use NetworkManager or `netplan` (those have their own restart commands), but it’s still standard for server environments and router setups using `ifupdown`.

---

# 7\. What happens in logs

When you run the command, you can check what systemd did:

```bash
sudo journalctl -u networking
```

This shows logs from the `networking.service` — typically you’ll see lines like:

```less
networking[520]: Configuring network interfaces...
networking[520]: Restarting networking (via systemctl): networking.service.
```

---

# 8\. Summary Table

| Component | Role |
| --- | --- |
| `sudo` | Run as root (network configuration requires elevated permissions) |
| `systemctl` | Controls system services managed by systemd |
| `restart` | Stop then start a service cleanly |
| `networking` | The service that runs ifup/ifdown based on `/etc/network/interfaces` |
| Result | All “auto” interfaces go down, then come up with new IP/routing config |

---

# 9\. Related useful commands

| Command | What it does |
| --- | --- |
| `systemctl status networking` | Show whether the service is active and last logs |
| `systemctl stop networking` | Take down all interfaces (dangerous if remote!) |
| `systemctl start networking` | Bring interfaces back up |
| `ifquery --list` | Show which interfaces are managed by `/etc/network/interfaces` |
| `ifup <iface>` / `ifdown <iface>` | Bring up/down only one interface manually |

---

In short:

> `sudo systemctl restart networking` is a high-level, systemd-managed way to re-run `/sbin/ifdown -a` and `/sbin/ifup -a`, causing Linux to re-read `/etc/network/interfaces`, bring all network links down, then bring them up again with the new configuration.