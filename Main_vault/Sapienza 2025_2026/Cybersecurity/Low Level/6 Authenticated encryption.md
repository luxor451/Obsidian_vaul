**Tags:** #Cybersecurity #LowLevel #Cryptography #AuthenticatedEncryption #AEAD #AESGCM #Poly1305 #TLS

---

# 1. Introduction to Authenticated Encryption

## Definition
**Authenticated Encryption (AE)** is a cryptographic primitive that simultaneously provides:
1.  **Confidentiality:** Encrypting the data.
2.  **Integrity & Authenticity:** Ensuring the data has not been modified and comes from a legitimate sender.

## Importance
* **Crucial Standard:** Recommended in modern protocols like [[15 TLS|TLS 1.3]], IPsec, and messaging apps.
* **Unified Primitive:** Often offered as a single API call to prevent implementation errors (like chosen-ciphertext attacks).
* **Workflow:**
    * **Input:** Message + Secret Key(s).
    * **Output:** Ciphertext + Authentication Tag.
    * **Decryption:** The system *must* verify the tag first. If valid, the plaintext is revealed; otherwise, it returns an error.

---

# 2. Approaches to AE Construction

There are three main ways to combine Encryption ($E$) and Message Authentication Codes ($MAC$):

## 1. Encrypt-then-MAC (EtM)
* **Process:**
    1.  Encrypt the plaintext: $C = E_k(P)$.
    2.  Compute MAC over the ciphertext: $T = MAC_k(C)$.
    3.  Send $(C, T)$.
* **Security:** Widely accepted as the **most secure and robust** approach. Proven secure under strong assumptions.
* **Decryption:** Verify $T$ on $C$ first. If valid, decrypt $C$.

## 2. Encrypt-and-MAC (E&M)
* **Process:**
    1.  Encrypt plaintext: $C = E_k(P)$.
    2.  Compute MAC over plaintext: $T = MAC_k(P)$.
    3.  Send $(C, T)$.
* **Security:** Generally considered **insecure**. The MAC might leak information about the plaintext.

## 3. MAC-then-Encrypt (MtE)
* **Process:**
    1.  Compute MAC over plaintext: $T = MAC_k(P)$.
    2.  Encrypt both: $C = E_k(P || T)$.
    3.  Send $C$.
* **Security:** Proven secure only in limited settings. Vulnerable to **Padding Oracle Attacks** (e.g., the "Lucky 13" attack against TLS 1.2).

---

# 3. Authenticated Encryption with Associated Data (AEAD)

## Concept
AEAD allows validating the integrity of data that is **not** encrypted (Associated Data), usually headers or routing information, alongside the encrypted payload.

## Structure
The input consists of four elements:
1.  **Secret Key ($K$):** Shared secret.
2.  **Nonce ($N$):** A unique number used once (IV).
3.  **Associated Data ($A$):** Data to authenticate but not encrypt (e.g., IP headers).
4.  **Plaintext ($P$):** Data to encrypt and authenticate.

$$
Enc(K, N, A, P) = (C, T)
Dec(K, N, A, C, T) = P \text{ or FAIL}
$$


## Usage
Used heavily in networking where headers ($A$) must remain visible for routing but must be protected against tampering.

---

# 4. AES-GCM (Galois/Counter Mode)

**AES-GCM** is the most widely used AEAD algorithm today.

## Components
1.  **CTR Mode (Encryption):** Uses standard AES Counter mode for high-speed encryption.
2.  **GMAC (Authentication):** A Galois Message Authentication Code that uses a specialized hash function called **GHASH**.

## GHASH Function
* Operates in the Finite Field $GF(2^{128})$.
* It is a polynomial evaluation based on the ciphertext blocks and associated data.
* **Mathematical Property:** It is a "Universal Hashing" function, not a standard cryptographic hash.
* **Formula:**
    Usually calculates $X_{m+n+1}$ where inputs are treated as coefficients of a polynomial.

## Characteristics
* **Parallelizable:** Can be pipelined for high throughput (hardware friendly).
* **Efficiency:** No padding required.
* **Nonce ($IV$):**
    * Typically **96 bits**.
    * **Critical:** Reusing a nonce with the same key destroys security (XOR stream reuse).

---

# 5. AES-CCM (Counter with CBC-MAC)

**AES-CCM** is an alternative AE mode used in specific protocols like WPA2 (Wi-Fi) and ZigBee.

## Construction
* **Combination:** Combines **AES-CTR** (for confidentiality) and **AES-CBC-MAC** (for integrity).
* **Approach:** It effectively uses a **MAC-then-Encrypt** strategy.

## Pros & Cons
* **Pros:** Uses only one underlying primitive (AES encryption block) for both encryption and MAC.
* **Cons:** Not parallelizable (due to CBC-MAC) and requires two passes over the data (slower than GCM).

---

# 6. ChaCha20-Poly1305

A modern AEAD scheme that does **not** use AES. It combines a stream cipher with a fast MAC.

## Components
1.  **ChaCha20:** A fast, secure stream cipher (variant of Salsa20).
2.  **Poly1305:** A high-speed message authentication code designed by Daniel J. Bernstein.

## Poly1305 Details
* **Type:** One-time authenticator (Carter-Wegman MAC).
* **Key:** Uses a 256-bit key pair $(r, s)$.
* **Math:** Evaluates a polynomial over the prime field $2^{130}-5$.
$$
Tag = (c_1 r^q + c_2 r^{q-1} + ... + c_q r) \pmod{2^{130}-5} + s \pmod{2^{128}}
$$


## Security & Performance
* **Security:** Provably secure if ChaCha20 is a secure PRF. Tag forgery probability is approx $8 \times 2^{-106}$.
* **Performance:** Faster than AES on CPUs without dedicated AES hardware (e.g., mobile devices, IoT). Optimizes well for 64-bit processors.

## Applications
* **TLS 1.3:** Mandatory cipher suite option.
* **OpenSSH:** Default in many configurations.
* **WireGuard:** The default VPN protocol encryption.
* **Android:** Used in BoringSSL for mobile performance.

---

# 7. Summary Comparison

| Algorithm | Encryption | Authentication | Type | Parallelizable? | Use Case |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **AES-GCM** | AES-CTR | GHASH ($GF(2^{128})$) | Encrypt-then-MAC (variant) | **Yes** | Web (TLS), VPNs |
| **AES-CCM** | AES-CTR | CBC-MAC | MAC-then-Encrypt | **No** | WPA2, IoT (ZigBee) |
| **ChaCha20-Poly1305** | Stream Cipher | Poly1305 ($2^{130}-5$) | Encrypt-then-MAC | **Yes** | Mobile, SSH, WireGuard |

**Recommendation:**
* Use **AES-GCM** if hardware acceleration (AES-NI) is available.
* Use **ChaCha20-Poly1305** for software-only implementations or mobile devices.