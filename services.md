## 1\. General definition

A **service** is a **background program** that runs continuously (or on demand) to perform a specific system function, **without direct user interaction**.

Examples:

-   A **web server** (`nginx`, `apache2`) serves web pages.
    
-   A **network service** (`networking`, `NetworkManager`) manages IP and routing.
    
-   A **database service** (`mysql`, `postgresql`) handles data queries.
    
-   A **logging service** (`rsyslog`, `journald`) records system messages.
    
-   A **DHCP service** (`dnsmasq`, `isc-dhcp-server`) gives out IP addresses.
    

These are sometimes called:

-   *daemons* (traditional Unix term)
    
-   *background processes*
    
-   *system services*
    

---

## 2\. Difference between a normal program and a service

| Feature | Normal Program | Service |
| --- | --- | --- |
| Starts | When you run it manually (e.g., `ls`, `vim`) | Automatically at boot (or manually via `systemctl start`) |
| Stops | When you close it | Keeps running until stopped by system or admin |
| Visible in terminal? | Yes (interactive) | No — runs in background, no terminal attached |
| Managed by | You (user shell) | The system (systemd or init) |
| Example | `nano`, `top`, `python script.py` | `sshd`, `cron`, `networking`, `dhcpd` |

---

## 3\. Why services exist

They provide **continuous functionality** the system or users need:

-   Handle network connections
    
-   Run scheduled jobs
    
-   Listen for incoming data
    
-   Serve requests to other computers
    
-   Monitor the system
    

So instead of you running `nginx` every minute to serve a page, the OS keeps it running permanently as a *service*.

---

## 4\. In Linux: “Service” = “Systemd Unit”

Modern Linux uses **systemd** (the init system) to manage all services.  
Each service has a *unit file* that tells systemd how to start, stop, and monitor it.

Example service file:

```swift
/lib/systemd/system/ssh.service
```

Contents (simplified):

```ini
[Unit]
Description=OpenSSH Server

[Service]
ExecStart=/usr/sbin/sshd -D
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

This defines:

-   What program runs (`ExecStart`)
    
-   What to do if it crashes
    
-   When it should start (`multi-user.target` = normal boot mode)
    

---

## 5\. How to interact with services (via `systemctl`)

Systemd gives you commands to manage these services.

| Command | What it does |
| --- | --- |
| `systemctl start ssh` | Start the SSH service now |
| `systemctl stop ssh` | Stop it |
| `systemctl restart ssh` | Restart it |
| `systemctl status ssh` | Show if it’s running and logs |
| `systemctl enable ssh` | Start automatically at boot |
| `systemctl disable ssh` | Don’t start automatically |
| `systemctl list-units --type=service` | List all active services |

---

## 6\. Example: the networking service

When you run:

```bash
sudo systemctl restart networking
```

you are restarting the *service* named `networking.service`.  
That service’s job is to bring network interfaces up/down according to `/etc/network/interfaces`.

Another example:

```bash
sudo systemctl status ssh
```

You’ll see:

```yaml
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled)
     Active: active (running) since Fri 2025-11-07 21:00:32 CET; 5h ago
```

That tells you it’s a **service**, running continuously in the background.

---

## 7\. Relation between “service” and “daemon”

A **daemon** is the actual process.  
A **service** is the *system-managed definition* (wrapper + behavior rules) for that daemon.

Example:

-   Process: `/usr/sbin/sshd`
    
-   Service: `ssh.service` (tells systemd how to manage `/usr/sbin/sshd`)
    

---

## 8\. Summary

| Concept | Description |
| --- | --- |
| **Service** | A long-running background task managed by systemd |
| **Daemon** | The actual executable/program providing the service |
| **systemd** | The system manager that starts/stops services |
| **Unit file** | Defines how a service behaves |
| **systemctl** | Command-line tool to control systemd and services |

---

### Analogy

Think of your computer as a **factory**:

-   The **services** are the workers who keep things running (web server, DHCP server, network manager, etc.).
    
-   **systemd** is the factory manager — it starts them in order, keeps them alive, restarts them if they crash.
    
-   **systemctl** is your control panel to tell the manager what to do.
    
