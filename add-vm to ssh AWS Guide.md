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

