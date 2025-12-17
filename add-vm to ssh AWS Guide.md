### 1.
Edit `nano /etc/apt/sources.list` so every line uses the archive site, e.g.

```md
deb http://archive.debian.org/debian buster main contrib non-free
deb http://archive.debian.org/debian buster-updates main contrib non-free
deb http://archive.debian.org/debian-security buster/updates main contrib non-free
```

### 2.
Run:

```bash
apt-get update
```

then:

```bash
apt install wget
```

### 3.
Start a quick server on the host, inside the folder containing `key.pem` (or files you want to share).

```bash
python3 -m http.server 9000
```

### 4.
In the VM:

```bash
wget http://10.0.2.2:9000/key.pem
```

### 5.
`chmod 400 key.pem` and use it:

```bash
ssh -i key.pem ubuntu@56.228.11.78
```

or :

```bash
ssh -i "sshd.pem" ubuntu@ec2-56-228-11-78.eu-north-1.compute.amazonaws.com
```

### 6.
To gain root access use:
```bash
sudo su -
```
### 7.
To switch to `fr` keyboard:

```bash
apt install console-data
```

Followed by:

```bash
loadkeys fr
```

To make the French console layout stick across reboots, update the system keyboard config instead of just running loadkeys:

1. Edit `/etc/default/keyboard` (or run `sudo dpkg-reconfigure keyboard-configuration`) and set:
    
    `XKBMODEL="pc105" XKBLAYOUT="fr" XKBVARIANT="" XKBOPTIONS=""`
    
 2. Apply it to the current console: `sudo setupcon` (or simply reboot).

Alternatively, on `systemd` hosts you can run `sudo localectl set-keymap fr` which writes `/etc/vconsole.conf` for you. After that, every TTY will boot with the AZERTY layout.

### 8.
Xauthority Error while using [[SSH_X11_Forwarding]] was fixed by:

```bash
export XAUTHORITY=$HOME/.Xauthority 
```

https://unix.stackexchange.com/questions/162979/annoying-message-x11-connection-rejected-because-of-wrong-authentication-while

