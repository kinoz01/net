You need to perform a one-time setup that involves generating a cryptographic key pair on your client machine and copying the public half to the server. This process is secure because it relies on the client proving possession of the private key (which never leaves your machine) instead of sending a guessable password.

Here is the step-by-step process to connect without a password:

## 🔑 SSH Passwordless Login Setup

### Step 1: Generate an SSH Key Pair (On your **Client** Machine)

Run the following command in your terminal. We use the `-t ed25519` option for a modern, secure key type, or you can use `rsa` if your system is older.


```bash
ssh-keygen -t ed25519 -C "your_username@your_machine"
```

1. **Enter file in which to save the key:** Press **Enter** to accept the default location (`~/.ssh/id_ed25519`).
    
2. **Enter passphrase (empty for no passphrase):**
    
    - **To go completely passwordless,** press **Enter** twice to leave the passphrase empty.
        
    - **RECOMMENDED:** To add an extra layer of security, enter a strong **passphrase**. If you do this, you will be prompted for this passphrase once per session, which is better than entering a long account password every time.
        

This command creates two files in your `~/.ssh/` directory:

- `id_ed25519`: Your **Private Key** (Keep this file absolutely secret).
    
- `id_ed25519.pub`: Your **Public Key** (This is safe to share).
    

### Step 2: Copy the Public Key to the Server (The Target)

This step takes the contents of your public key (`id_ed25519.pub`) and appends it to the `authorized_keys` file on the remote server (`localhost` in your case, for the `root` user).

The simplest way is to use the `ssh-copy-id` utility:


```bash
ssh-copy-id -p 10122 root@localhost
```

- **Note:** The first time you run this, you **will be prompted for the `root@localhost` password**. This is the only time you need to enter the password during the setup process.
    
- The utility handles creating the correct directories and setting the proper permissions on the server for you.
    

#### Alternative (If `ssh-copy-id` is not available):

You can manually pipe the key using the following command (you will still be prompted for the password once):


```bash
cat ~/.ssh/id_ed25519.pub | ssh -p 10122 root@localhost "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

### Step 3: Test the Passwordless Connection

After Step 2 is complete, try connecting again using your original command:


```bash
ssh -p 10122 root@localhost
```

If the setup was successful, you should be logged in immediately **without a password prompt** (or without a passphrase prompt, if you chose to set one).

### ⚠️ Critical Permissions and Troubleshooting

If the connection still asks for a password, the issue is almost always file permissions on the server. SSH is extremely strict about permissions.

On the **remote server** (`localhost`):

1. The `.ssh` directory must have permissions: **`700` (rwx------)**
    
2. The `authorized_keys` file must have permissions: **`600` (rw-------)**
    

You can fix these permissions by running the following commands (after you are successfully connected, even if it requires a password):


```bash
# On the remote server:
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Once you have verified that key-based authentication is working, you can optionally disable password authentication entirely on the server for a much higher level of security.

For a visual demonstration of how to set up passwordless SSH login, this video is helpful: [How SSH password-less key based authentication work in 4 minutes (with example)](https://www.youtube.com/watch?v=RfolgB-rVe8).