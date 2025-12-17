### SSH X11 Forwarding Setup (Linux Client & Server)

1. **Install X11 helpers on the server**
   ```bash
   sudo apt install xauth x11-apps   # use your distro’s package manager
   ```
   These packages let the remote host hand off GUI apps to your desktop.

2. **Ensure your desktop client runs an X server**
   Any standard Linux desktop already has X11 running, so nothing extra is needed on the client.

3. **Enable forwarding on the client**
   - One-off: `ssh -X user@host`
   - Permanent: add to `~/.ssh/config`
     ```
     Host your-server
         ForwardX11 yes
         ForwardX11Trusted yes   # optional, equivalent to -Y
     ```

4. **Allow X11 forwarding on the server**
   Edit `/etc/ssh/sshd_config` and confirm:
   ```
   X11Forwarding yes
   X11DisplayOffset 10
   ```
   Then reload SSH:
   ```bash
   sudo systemctl restart sshd
   ```

5. **Test the setup**
   ```bash
   ssh -X user@host
   xclock    # or xeyes/xterm
   ```
   A window should pop up on your local screen, proving the GUI app is running remotely but displayed locally.
