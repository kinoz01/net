- **Capability check** – After the encrypted transport is up, the server replies to SSH_MSG_USERAUTH_REQUEST with SSH_MSG_USERAUTH_FAILURE listing methods it accepts (e.g., publickey,password). Your client sees publickey is allowed and proceeds.
    
- **Initial offer** – The client sends SSH_MSG_USERAUTH_REQUEST specifying method publickey and includes the public key blob (key type + public key data) so the server knows which entry in ~/.ssh/authorized_keys to check. This first message can be a “probe” without a signature to see if the server recognizes the key.
    
- **Server challenge** – If the key is acceptable, the server replies with SSH_MSG_USERAUTH_PK_OK, which acts as the challenge prompt. The challenge data implicitly includes the session ID (hash of the initial key exchange), the user, service (ssh-connection), and the exact key blob—these fields must be covered by the signature.
    
- **Client signature** – The client re-sends SSH_MSG_USERAUTH_REQUEST with the same fields plus a digital signature computed over the concatenation of the session ID and authentication request packet. The signature is produced by the private key (Ed25519, RSA, etc.).
    
- **Server verification** – The server loads the matching public key from authorized_keys, uses the relevant algorithm to verify the signature against the session ID + request data. If verification succeeds, the server sends SSH_MSG_USERAUTH_SUCCESS. If it fails, the server sends another failure message and may allow other methods (e.g., password).
    

Because only someone holding the private key can generate the correct signature for that session ID, the server accepts it as proof of identity—no password transmission needed.