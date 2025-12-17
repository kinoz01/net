
## Client – `ls ~/.ssh`

```bash
ls ~/.ssh
id_ed25519  id_ed25519.pub  known_hosts  known_hosts.old
```

- `id_ed25519`: private Ed25519 key that proves the client’s identity (never leave this machine).
- `id_ed25519.pub`: matching public key; copy this into a server’s `authorized_keys`.
- `known_hosts` / `known_hosts.old`: cache of host public keys you have already trusted; protects against host impersonation.

## Server – `ls ~/.ssh`

```bash
ls ~/.ssh
authorized_keys  known_hosts
```

- `authorized_keys`: each line is a trusted client public key; one of these lines must match the contents of the client’s `id_ed25519.pub`.
- `known_hosts` (optional): if the server itself initiates SSH sessions as this user, it will keep host fingerprints here just like a client does.


## Client – `ls /etc/ssh`

```bash
ls /etc/ssh
ssh_config  ssh_config.d
```

This workstation only has the OpenSSH *client* installed, so `/etc/ssh` contains the global `ssh_config` plus an optional `ssh_config.d/` drop-in directory.

## Server – `ls /etc/ssh`

```bash
ls /etc/ssh
moduli  ssh_config  sshd_config  sshd_config.d  ssh_host_ed25519_key  ssh_host_ed25519_key.pub  ssh_host_rsa_key  ssh_host_rsa_key.pub  ssh_host_ecdsa_key  ssh_host_ecdsa_key.pub
```

- `sshd_config` (and optionally `.d/`): daemon-side options (listen port, key exchange list, login policies).
- `ssh_host_*`: host key pairs the server uses to prove its identity during the handshake.
- `moduli`: safe prime data for classic Diffie–Hellman groups.
- `ssh_config`: client defaults (servers can also act as SSH clients when they connect elsewhere).

🔁 The overlap between client and server `/etc/ssh` is just the shared `ssh_config` template; only the server stores `sshd_config` and the host keys. Likewise, only clients keep their private login keys in `~/.ssh`, while servers retain the `authorized_keys` that mirror those client public keys.
