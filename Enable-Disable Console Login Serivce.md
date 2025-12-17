### 1. Disable the Console Login Service (Recommended)

Most modern Linux distributions manage the console login prompt (where you see `login:`) as a **systemd service** called a **getty** (from "get tty"). By default, this is usually active on `tty1`.

#### Using `systemctl`

You can disable the `getty@tty1.service` (or similar) to prevent the login prompt from appearing on the physical console.

1. **Stop the service immediately:**
    
    
    ```bash
    sudo systemctl stop getty@tty1.service
    ```
    
2. **Disable the service permanently (to survive reboots):**
    
    
    ```bash
    sudo systemctl disable getty@tty1.service
    ```
    
    This removes the symbolic link that starts the service at boot time.
    
3. **Verify the status:**
    
    
    ```bash
    sudo systemctl status getty@tty1.service
    ```
    
    The output should show **"Loaded: disabled"** and **"Active: inactive (dead)"**.
    

### 2. Mask the Console Login Service (More Restrictive)

If you want to ensure the service can **never** be started, even by another service, you can "mask" it. Masking creates a symlink from the service file to `/dev/null`, making it impossible to activate.

1. **Mask the service:**
    
    
    ```bash
    sudo systemctl mask getty@tty1.service
    ```
    
2. **Disable and Stop (just in case):**
    
    
    ```bash
    sudo systemctl disable getty@tty1.service
    sudo systemctl stop getty@tty1.service
    ```
    
3. **To undo the masking:**
    
    
    ```bash
    sudo systemctl unmask getty@tty1.service
    ```
    

### 3. Modify `inittab` (For Older/Minimal Systems - SysVinit or OpenRC)

If your minimal distro uses an older init system like **SysVinit** (e.g., older Debian/Ubuntu, some minimal builds) or **OpenRC** (e.g., Alpine Linux, Gentoo), you would modify the `/etc/inittab` file.

1. **Open the configuration file:**
    
    
    ```bash
    sudo nano /etc/inittab
    ```
    
2. **Comment out the TTY lines:** Find the lines responsible for starting the `getty` processes, which usually look something like this:
    
    ```
    1:2345:respawn:/sbin/agetty 38400 tty1
    2:2345:respawn:/sbin/agetty 38400 tty2
    ```
    
    Add a hash sign (`#`) to the beginning of the line(s) you want to disable (e.g., just `tty1`):
    
    ```
    # 1:2345:respawn:/sbin/agetty 38400 tty1
    2:2345:respawn:/sbin/agetty 38400 tty2
    ```
    
3. **Apply the changes:** You may need to reboot or run a command like `sudo telinit q` to reload the `inittab` file, depending on the system. **A reboot is the safest way to test this.**
    

### ⚠️ Important Security Considerations

- **SSH Access:** Ensure the **SSH server service (`sshd`) is running and enabled** before disabling the local console. You don't want to lock yourself out!
    
    - **Enable/Start SSH:**
        
        
        ```bash
        sudo systemctl enable sshd
        sudo systemctl start sshd
        ```
        
- **Physical Console Access:** Keep in mind that disabling the console means you will **need network access (SSH)** to log in. If the network goes down or SSH fails, you will need a way to re-enable the console, potentially by booting into a recovery environment or using a live CD/USB to edit the `/etc/systemd/system` or `/etc/inittab` files.
    
- **Security, Not Obscurity:** This prevents casual physical access, but it's not a complete security solution. Anyone with physical access can still bypass this by:
    
    - Rebooting and entering the bootloader (e.g., GRUB) to gain a root shell.
        
    - Booting from a live CD/USB.

## Re-enabling
### 1. Re-enable Console Login via SSH (Systemd)

From your active SSH session, run the following commands as a user with `sudo` privileges (or as `root`):

1. Unmask the Service (if you masked it):
    
    If you used the highly restrictive mask command previously, you must first undo it.

    
    ```bash
    sudo systemctl unmask getty@tty1.service
    ```
    
    _(If you never ran the `mask` command, this step will do no harm, but it's safe to run just in case.)_
    
2. Enable the Service:
    
    This re-creates the symbolic link to ensure the getty service starts automatically every time the system boots up.
    
    
    ```bash
    sudo systemctl enable getty@tty1.service
    ```
    
3. Start the Service Immediately:
    
    This forces the service to start right now without requiring a reboot, which is crucial since you are logged in remotely.
    
    
    ```bash
    sudo systemctl start getty@tty1.service
    ```
    
4. Verify the Status:
    
    Check that the service is running and enabled:
    
    
    ```
    sudo systemctl status getty@tty1.service
    ```
    
    You should see `Active: active (running)`.
    

### ✅ What to Expect Now

Immediately after running `sudo systemctl start getty@tty1.service`, if you go back to the physical screen (or switch to `tty1` using $\text{Ctrl}+\text{Alt}+\text{F1}$ if you are on a different tty), you should see the familiar **`login:`** prompt appear, and you should be able to type your username and password.

---

### ⚠️ If SSH is NOT Working (Emergency Procedure)

If you are physically in front of the machine and **cannot** log in via SSH (e.g., the network is down, the SSH service failed to start, or you locked it down too aggressively), then you are in the difficult state described in the previous answer.

In this case, you **must** use the physical console to fix it. This is done by interrupting the normal boot process:

1. **Reboot** the machine.
    
2. During boot, access the **GRUB menu** (usually by pressing $\text{Shift}$ or $\text{Esc}$).
    
3. **Edit** the boot entry (press `e`).
    
4. Find the line starting with `linux` or `linuxefi` and append `init=/bin/bash` to the end.
    
5. Boot with this parameter ($\text{Ctrl}+\text{x}$ or $\text{F10}$) to get a root shell.
    
6. Remount the filesystem as read-write: `mount -o remount,rw /`
    
7. Run the same `systemctl` commands from the SSH section above.
    
8. Reboot: `reboot -f`