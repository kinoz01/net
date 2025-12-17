## ⚙️ Understanding Checksums in Detail

A **checksum** is a small-sized datum derived from a block of digital data for the purpose of detecting errors that may have been introduced during its transmission or storage. It is one of the most basic and common forms of data integrity checking.

Simply put, a checksum is a **redundant value** added to the data, calculated in such a way that if even a single bit of the original data is flipped, the re-calculated checksum will almost certainly not match the attached checksum.

---

## How a Checksum is Calculated and Used

The process involves both the sender/writer and the receiver/reader performing an identical mathematical operation.

### 1. Calculation (Sender/Writer Side)

1.  The original block of data ($D$) is fed into a specific **checksum algorithm** ($A$).
2.  The algorithm performs a mathematical operation (usually simple addition, modulus, or XOR) on the data.
3.  The result is the checksum value ($C$).
4.  The sender transmits/stores the data ($D$) appended with the checksum ($C$).
    * $$\text{Checksum } (C) = A(D)$$

### 2. Verification (Receiver/Reader Side)

1.  The receiver receives the data block ($D'$) and the attached checksum ($C'$).
2.  The receiver feeds the *received data* ($D'$) into the **same algorithm** ($A$) to calculate the expected checksum ($C_{\text{expected}}$).
    * $$\text{Checksum}_{\text{expected}} = A(D')$$
3.  The receiver compares the calculated $C_{\text{expected}}$ with the received $C'$.

### 3. Integrity Check

* **If $C_{\text{expected}} = C'$:** The data is assumed to be integral (correct).
* **If $C_{\text{expected}} \neq C'$:** An error is detected; the data is corrupt and must be retransmitted or discarded.

---

## 🛠️ Common Checksum Algorithms

The term checksum is generic, and the algorithm used dictates the level of error detection capability.

### 1. Simple Sum Check (Basic Checksum)

* **Mechanism:** The algorithm simply sums all the bytes or words in the data block and truncates the result to a fixed size (e.g., 8-bit or 16-bit).
* **Weakness:** It has very poor error-detection capability. If a '1' is changed to a '0' in one byte and a '0' is changed to a '1' in another byte (a two-bit error that cancels out the change in the total sum), the checksum remains the same, and the error goes undetected.

### 2. Internet Checksum (Used in IP, UDP, ICMP)

* **Mechanism:** A more robust method used extensively in network protocols. It involves calculating the one's complement sum of all 16-bit words in the data.
* **Strength:** It is computationally inexpensive and highly effective for detecting most single-bit errors and certain patterns of burst errors in packet headers.

### 3. CRC (Cyclic Redundancy Check)

* **Mechanism:** CRCs are the most powerful and widely used form of checksum. They are based on polynomial arithmetic over a finite field (Galois field). The data block is treated as the coefficients of a large polynomial, which is divided by a fixed generating polynomial. The remainder of this division is the CRC.
* **Strengths:** CRCs are mathematically proven to be excellent at detecting common burst errors (where multiple consecutive bits are corrupted). They are used in Ethernet, Wi-Fi, hard drives, and virtually all storage and transmission mediums. Common types are CRC-32 and CRC-64. 

---

## 🧐 Checksum vs. Cryptographic Hash

It is important to distinguish checksums from cryptographic hashes (like SHA-256 or MD5).

| Feature | Checksum (e.g., CRC, Internet Checksum) | Cryptographic Hash (e.g., SHA-256) |
| :--- | :--- | :--- |
| **Purpose** | Detect **random, accidental errors** (noise, transmission failure). | Detect **malicious, intentional tampering** (forgery). |
| **Security** | **Low.** Easy for an attacker to calculate a new checksum for modified data. | **High.** Computationally infeasible for an attacker to find data that results in a target hash (collision resistance). |
| **Algorithm** | Simple, fast, non-cryptographic math operations. | Complex, slow, one-way cryptographic functions. |
| **Size** | Small (typically 8, 16, or 32 bits). | Large (typically 128, 256, or 512 bits). |

In conclusion, a checksum is a fast and simple mechanism, primarily relying on mathematical redundancy to ensure the integrity of data against **accidental errors** during storage or transmission.

---> [[checksum vs hashing|So basically a checksum is like lightweight hashing?]]