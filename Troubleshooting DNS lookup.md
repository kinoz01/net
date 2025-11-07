## 1\. Your current configuration

```ini
auto enp0s3
iface enp0s3 inet static
  address 192.168.56.10
  netmask 255.255.255.0
  gateway 192.168.56.1
  dns-nameservers 8.8.8.8
```

This sets:

-   IP: `192.168.56.10`
    
-   Gateway: `192.168.56.1` (router\_vm)
    
-   DNS: `8.8.8.8` (Google DNS)
    

Yet you said:

> DNS isn’t working.

That means:

-   `ping 8.8.8.8` **works** (you have Internet)
    
-   but `ping google.com` **fails**
    

This confirms your **DNS resolver config** is not being applied.

---

## 2\. Why this happens

When using **ifupdown** (which `/etc/network/interfaces` belongs to), the `dns-nameservers` directive only works **if the package `resolvconf` or `systemd-resolved`** is properly configured to update `/etc/resolv.conf`.

If neither is handling it, then the DNS setting in your interfaces file is ignored — your `/etc/resolv.conf` won’t be updated.

Let’s check:

```bash
cat /etc/resolv.conf
```

You’ll probably see something like:

```nginx
nameserver 127.0.0.53
```

or

```bash
# This file is managed by systemd-resolved
```

That means the **system DNS resolver** isn’t reading `dns-nameservers` from your interfaces file.

---

## 3\. Option A — Quick temporary fix (manual)

Use

```bash
sudo nano /etc/resolv.conf
```

Then replace:

```bash
127.0.0.53 -> 8.8.8.8
```


But this fix is **temporary**: many distros reset `/etc/resolv.conf` on boot.

---

## 4\. Option B — Permanent fix for ifupdown (recommended for router/client labs)

Install `resolvconf` (or `openresolv`, depending on distro):

```bash
sudo apt install resolvconf
```

Then restart networking:

```bash
sudo systemctl restart networking
```

Now your `dns-nameservers` line will automatically populate `/etc/resolv.conf` with:

```nginx
nameserver 8.8.8.8
```

Check:

```bash
cat /etc/resolv.conf
```

You should see it correctly now.

---

## 5\. Option C — Disable `systemd-resolved` if it conflicts

If you’re using Debian or Ubuntu where `/etc/resolv.conf` points to `127.0.0.53`, `systemd-resolved` might override your DNS config.

You can either configure it properly, or disable it entirely and manage DNS manually.

### To disable systemd-resolved:

```bash
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
sudo rm /etc/resolv.conf
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
```

Now your DNS is under your control.

---

## 6\. Option D — Add DNS via netplan (for newer Ubuntu)

If your distro uses `netplan` (Ubuntu 18.04+), `/etc/network/interfaces` may be ignored entirely.  
You can check:

```bash
ls /etc/netplan/
```

If you see a `.yaml` file (like `01-netcfg.yaml`), your system is using **netplan** instead of `ifupdown`.

In that case, edit that YAML file instead:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.56.10/24]
      gateway4: 192.168.56.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```

Then apply:

```bash
sudo netplan apply
```
