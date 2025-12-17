Once you run `ssh -p 10122 root@localhost` (after the setup is complete), here is the step-by-step process of how the server verifies the client:

|**Step**|**Action by the Client (Your Machine)**|**Action by the Server (The Target)**|**Verification Purpose**|
|---|---|---|---|
|**1. Negotiation**|The client initiates the connection and securely exchanges session keys (Key Exchange Phase).|The server says, "I see you are trying to log in as `root`. Show me your proof."|Establish a secure communication channel.|
|**2. Key Offer**|The client tells the server: "I will try to authenticate using Public Key $P_A$ (e.g., the key from `id_ed25519.pub`)."|The server checks the `~/.ssh/authorized_keys` file for the `root` user to see if $P_A$ is listed.|Identify the authentication method and key to be used.|
|**3. The Challenge**|(Client waits)|The server generates a unique, random string of data, known as the **Challenge ($C$)**, and encrypts it using the client's **Public Key ($P_A$)** found in the `authorized_keys` file.|Create a test only the true owner can solve.|
|**4. Signature Creation**|The client receives the encrypted challenge. It uses its secret **Private Key ($K_A$)** (stored locally) to **decrypt** the challenge ($C$). The client then uses $K_A$ to **cryptographically sign** the original challenge along with data specific to the current SSH session. This results in the **Signature ($S$)**.|(Server waits)|Prove possession of the private key without exposing it.|
|**5. Verification**|The client sends the Signature ($S$) back to the server.|The server uses the client's **Public Key ($P_A$)** to **verify** the Signature ($S$). Since $P_A$ and $K_A$ are mathematically linked, only the correct $K_A$ could have generated a valid signature.|**Grant Access.** If the signature is valid, the server is certain the client possesses the private key and grants immediate access without a password.|

---

## 🔎 How the Server Checks the Client

The core check performed by the server is **Cryptographic Signature Verification** (Step 5 above).

The server doesn't care about a password; it cares about mathematical proof.

1. **Possession Proof:** The server encrypts a challenge with the **public key**. If the client can successfully decrypt it (using the private key) and then sign it (using the private key), the client has proven possession of the corresponding private key.
    
2. **Immutability:** Because the signature is also tied to the specific session data, an attacker cannot simply record a successful signature from a previous session and reuse it (preventing **Replay Attacks**).
    

**In short: The server checks the client by verifying a cryptographic signature created by the client's secret private key using the client's public key (which the server already trusts).**


---> [[Creating a Digital Signature]] is more accurate of what's going on during public key authentication.