
#### 1. Executing a Single Non-Interactive Command

If you provide a command immediately after the SSH connection parameters, the SSH client executes _only_ that command on the remote server and then immediately disconnects, without opening an interactive shell.

- **Command:**
    
    
    ```bash
    ssh user@remote_host "ls -l /var/log"
    ```
    
- **Result:** The command is executed, the output is printed to your local terminal, and the connection closes. You never see the remote prompt, and you cannot type any more commands.
    

#### 2. Running a Background or Non-Interactive Script

When executing a script or a command that is designed to run in the background (daemon) or requires no user input, SSH typically doesn't bother allocating a pseudo-terminal.

- **Command:**
    
    
    ```bash
    ssh user@remote_host "myscript.sh &"
    ```
    
- **Result:** The script is started in the background on the remote machine, and the SSH session closes without presenting a prompt.
    

#### 3. Using the `-T` Option

The `-T` flag explicitly tells the SSH client **not** to allocate a pseudo-terminal. This is often used when running non-interactive commands or scripts to reduce overhead and avoid issues with programs trying to use terminal-specific features.

- **Command:**
    
    
    ```bash
    ssh -T user@remote_host "echo 'Running command without a pty'"
    ```
    
- **Result:** The command executes, but the remote environment behaves as if it's not connected to a proper terminal.
    

#### 4. Using SSH for File Transfer (SCP/SFTP)

As discussed previously, when you use tools that rely on the SSH protocol for secure data transmission, you are not opening an interactive shell.

- **Command:**
    
    
    ```bash
    scp /local/file.txt user@remote_host:/tmp/
    ```
    
- **Result:** The file transfer happens, and the utility exits. No terminal is involved.
    

#### 5. Using SSH for Port Forwarding Only

If you use the `-L`, `-R`, or `-D` flags for creating tunnels and explicitly request no command execution (which implies no need for a shell), you won't get a terminal. This is often combined with the `-N` flag (Do not execute a remote command).

- **Command:**
    
    
    ```bash
    ssh -N -L 8080:localhost:80 user@remote_host
    ```
    
- **Result:** A secure tunnel is established, but the connection just hangs (stays open) without giving you a prompt, waiting for data to pass through the tunnel.
    

### Summary

|**SSH Case**|**Terminal Provided?**|**Reason**|
|---|---|---|
|**Default Login** (`ssh user@host`)|**Yes**|SSH requests a pseudo-terminal (pty) by default.|
|**Single Command** (`ssh user@host "cmd"`)|**No**|SSH executes the command and exits, no interactive shell needed.|
|**Using `-T` flag**|**No**|Explicitly instructs SSH _not_ to allocate a pseudo-terminal.|
|**File Transfer (`scp`, `sftp`)**|**No**|These are separate utilities focused on data transfer, not shell interaction.|
|**Tunneling (`-N`)**|**No**|The `-N` flag means "no remote command," so no shell is needed.|

In short, SSH is flexible. It only provides the interactive shell when that's what you need to manage the remote system; otherwise, it operates as a pure, secure data transporter.