## 🔐 Understanding Message Authentication Code (MAC)

A **Message Authentication Code (MAC)** is a short piece of information used to authenticate a message and provide **integrity** and **authenticity** assurances on data. Unlike digital signatures, which use asymmetric cryptography, MACs are generated using a **secret key** and a cryptographic hash function.

MACs are fundamental in communication security where two parties (sender and receiver) share a secret key.

### Key Characteristics

* **Integrity:** Assures the receiver that the message has not been altered, accidentally or maliciously, since it was created.
* **Authenticity:** Assures the receiver that the message truly comes from the claimed sender (the one who possesses the secret key).
* **Symmetry:** Requires a secret key that is shared and known by both the sender (for generation) and the receiver (for verification).

---

## 🔑 How MAC Achieves Integrity

The integrity assurance provided by a MAC relies entirely on the **secrecy of the shared key** and the **cryptographic properties of the underlying hash function**.

Here is a step-by-step breakdown of the process:

### Step 1: MAC Generation (Sender Side)

1.  **Input:** The sender has the original message ($M$) and a secret key ($K$).
2.  **Calculation:** The sender inputs *both* the message and the secret key into a MAC algorithm (often called a keyed hash function, or HMAC).
    $$\text{MAC} = \text{MAC}_K(M)$$
3.  **Transmission:** The sender transmits the original message ($M$) **concatenated with** the generated MAC tag ($\text{MAC}$).

### Step 2: MAC Verification (Receiver Side)

1.  **Receipt:** The receiver receives the message ($M'$) and the attached MAC tag ($\text{MAC}'$).
2.  **Recalculation:** The receiver uses the same shared secret key ($K$) and the *same* MAC algorithm to calculate a new, expected MAC tag ($\text{MAC}_{expected}$) over the received message ($M'$). 
$$\text{MAC}_{expected} = \text{MAC}_K(M')$$
3.  **Comparison:** The receiver compares the calculated $\text{MAC}_{expected}$ with the received $\text{MAC}'$.

### The Integrity Check

* **If $\text{MAC}_{expected} = \text{MAC}'$:** The receiver concludes that the message ($M'$) has **not been modified**. Why? Because only someone with the
* secret key ($K$) could have generated the *correct* MAC tag for that specific message content. Since the calculated tag matches the received tag, the content is trusted.
* **If $\text{MAC}_{expected} \neq \text{MAC}'$:** The receiver concludes that the message has been altered during transmission (integrity breach) **or** it was sent by an unauthorized party that doesn't know the key (authenticity breach). In either case, the message is discarded. 

---

## 🧮 Types of MAC Algorithms

The term "MAC" is a generic concept, and in practice, it is implemented using specific cryptographic algorithms, the most common being:

### 1. HMAC (Hash-based Message Authentication Code)

* **Mechanism:** This is the industry standard. HMAC uses a standard cryptographic hash function (like SHA-256 or SHA-3) and combines it with the secret key in a specific, well-defined way.
* **Formula (Simplified):** $$\text{HMAC}(K, M) = \text{Hash}((K \oplus \text{opad}) \parallel \text{Hash}((K \oplus \text{ipad}) \parallel M))$$
    * **Advantage:** It leverages existing, battle-tested hash functions and is proven to be secure, provided the underlying hash function is secure.

### 2. CMAC (Cipher-based Message Authentication Code)

* **Mechanism:** This method uses a symmetric block cipher (like AES) in a special mode to generate the MAC tag.
* **Advantage:** It can be efficient in environments where block ciphers are already hardware-accelerated.

### Why the Key is Essential for Integrity

The key is what differentiates MAC from a simple unkeyed hash (like a CRC or checksum):

| Feature      | MAC (with Key $K$)                                                                                                                                  | Simple Hash (No Key)                                                                                                        |
| :----------- | :-------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------- |
| **Purpose**  | Integrity **and** Authenticity.                                                                                                                     | Integrity (Error detection) only.                                                                                           |
| **Security** | If an attacker modifies $M$, they must know $K$ to calculate a new, valid $\text{MAC}_K(M')$. Since they don't know $K$, they cannot forge the tag. | If an attacker modifies $M$, they can simply calculate a new hash for $M'$ and replace the old one. The receiver is fooled. |

---> [[MAC vs Digital Signature|How MAC is different from a digital signature?]]