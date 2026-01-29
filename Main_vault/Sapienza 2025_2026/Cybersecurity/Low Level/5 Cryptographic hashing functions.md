**Tags:** #Cybersecurity #LowLevel #Cryptography #Hashing #HMAC #Integrity #Blockchain #KDF

---

# 1. Cryptographic Hashing Functions

## Definition
A **cryptographic hash function** takes an input of any length and maps it to a fixed-size output (digest or fingerprint).
* **Determinism:** The same input always produces the same output.
* **Efficiency:** It is computationally fast to compute the hash.
* **Avalanche Effect:** A small change in input results in a massive, unpredictable change in the output.

## Security Requirements
To be secure, a hash function must satisfy three properties:

1.  **Preimage Resistance (One-Wayness):**
    Given a digest $y$, it is computationally infeasible to find an input $x$ such that $h(x) = y$.
    * *Context:* "Infeasible" means the computation would take years or centuries.

2.  **Second Preimage Resistance (Weak Collision Resistance):**
    Given a specific input $x$, it is infeasible to find a different input $x' \neq x$ such that $h(x) = h(x')$.
    * *Term:* "Weak" because one input is fixed.

3.  **Collision Resistance (Strong Collision Resistance):**
    It is infeasible to find *any* pair of inputs $x$ and $x'$ ($x \neq x'$) such that $h(x) = h(x')$.
    * *Term:* "Strong" because the attacker can choose any pair.
    * *Proof:* Strong collision resistance implies weak collision resistance.

$$
\begin{aligned}
&\text{Proof Sketch: Show } \neg \text{Weak} \iff \neg \text{Strong} \\
&\text{Given } h \text{, suppose there is an algorithm } A_h \text{ such that } A_h(x) = x' \text{ where } h(x) = h(x'). \\
&\text{We can construct algorithm } B_h: \\
&1. \text{Arbitrarily choose } x. \\
&2. \text{Return } (x, A_h(x)). \\
&\text{This finds a collision in the strong sense.}
\end{aligned}
$$

## The Birthday Bound
Cryptographic hashes are subject to the **Birthday Paradox**.
* In a set of $N$ possible outputs, a collision is likely found after approx $\sqrt{N}$ operations.
* **Security Level:** An $n$-bit hash has a collision resistance of roughly $2^{n/2}$.
* **Threshold:** Fingerprints of 160 bits or less are considered "short" and potentially feasible to break today.

---

# 2. Hash Algorithms and Families

## Historical Designs
* **MD5:** Broken and deprecated. Insecure for security purposes.
* **SHA-1:**
    * 160-bit output.
    * **Broken:** Collision attacks are practical and affordable.
    * **Deprecated:** Google sunset support in Chrome in 2014; NIST deprecated it in 2010.
* **SHA-2:**
    * Family: SHA-224, SHA-256, SHA-384, SHA-512.
    * **Status:** Secure, widely used, industry standard (FIPS 180-4).
    * **Design:** Based on Merkle-Damgård.
* **SHA-3 (Keccak):**
    * Family: SHA3-224, SHA3-256, SHAKE128 (XOFs).
    * **Status:** Very secure, post-quantum suitable.
    * **Design:** Based on Sponge construction.

---

# 3. Constructions

## Merkle-Damgård Construction
Used in SHA-1 and SHA-2.
1.  **Block Processing:** Processes message in fixed-size blocks (e.g., 512 bits).
2.  **Padding:** Adds padding (including message length) to fit block size.
3.  **Compression Function:** Uses a function $H$ (often Davies-Meyer using a block cipher) to combine the current block with the previous state.

$$
H_i = E_{M_i}(H_{i-1}) \oplus H_{i-1}
$$
![[Pasted image 20260127192151.png]]
4.  **Vulnerability:** **Length Extension Attack**.
    * Since the output is the internal state, an attacker can append data to a hashed message and compute the new hash without knowing the original message content (if it acts as a key).

## Sponge Construction
Used in SHA-3 (Keccak).
1.  **State:** Large internal state (1600 bits) divided into **Rate** ($r$) and **Capacity** ($c$).
2.  **Absorbing:** Input data is XORed into the *Rate* portion, followed by a permutation function $f$.
3.  **Squeezing:** Output is extracted from the *Rate* portion.
4.  **Advantages:** Immune to length extension attacks; capacity ($c$) remains hidden.

---

# 4. Applications

## Key Derivation Functions (KDF)
Algorithms to generate secure keys from low-entropy inputs (passwords).
* **Purpose:** Strengthen weak inputs, provide key separation, and slow down brute-force attacks.
* **Mechanism:** Uses Salting (random value) and Iteration.
* **Definition:**

$$
Key = KDF(Password, Salt, Parameters)
$$

## Blockchain
A distributed ledger where blocks are cryptographically linked.
* **Linkage:** Each block contains the **hash of the previous block**.
* **Immutability:** Modifying block $N$ changes its hash, breaking the link to $N+1$.
* **Proof of Work (PoW):**
    * Miners must solve a computational puzzle.
    * **Goal:** Find a `nonce` such that:
    
    $$
    Hash(BlockData || nonce) < Target
    $$
    
    * This requires brute force and energy.

## Data Integrity & Optimization
* **Integrity:** Verify software downloads or file backups.
* **Deduplication:**
    1. Compute hash of files.
    2. If $Hash(A) \neq Hash(B)$, files are definitely different.
    3. If $Hash(A) == Hash(B)$, files are likely identical (store only one).
    * *Use Cases:* Cloud storage, Databases, CDNs.

## Privacy
* **Anonymization:** Hash identifiers (e.g., emails) before sharing data.
* **Zero-Knowledge Proofs:** Commit to values without revealing them.

---

# 5. Message Authentication (Keyed Hashing)

**Goal:** Provide both **Integrity** (data unchanged) and **Authenticity** (origin verification).
**Mechanism:** Use a shared secret key $k$.

## Naïve Constructions (Insecure)

## 1. Key Prefix Method
Concatenate key *before* message.

$$
MAC = H(k || M)
$$

* **Vulnerability:** **Length Extension Attack**.
* An attacker knowing $H(k || M)$ can compute $H(k || M || padding || M')$ without knowing $k$.

## 2. Key Suffix Method
Concatenate key *after* message.

$$
MAC = H(M || k)
$$

* **Vulnerability:** **Collision Attack**.
* If an attacker finds $M$ and $M'$ such that $H(M) = H(M')$, then $H(M || k) = H(M' || k)$ due to the iterative nature of Merkle-Damgård.

## 3. Key Sandwich Method
Concatenate key both *before* and *after*.

$$
MAC = H(k || M || k)
$$

* **Status:** Better, but requires strong assumptions about the hash function.

---

# 6. HMAC (Hash-Based Message Authentication Code)

HMAC is the industry standard (RFC 2104, FIPS 198) designed to secure message authentication using hash functions.

## The Construction
HMAC uses two fixed padding constants:
* **ipad (inner pad):** Fixed constant ($0x36$ repeated).
* **opad (outer pad):** Fixed constant ($0x5C$ repeated).

**Formula:**

$$
HMAC(K, M) = H\Big( (K \oplus opad) \;||\; H\big( (K \oplus ipad) \;||\; M \big) \Big)
$$

## Mechanism
1.  **Inner Hash:** $Inner = H((K \oplus ipad) || M)$
    * Prevents length extension attacks because the internal state is "sealed" inside the outer hash.
2.  **Outer Hash:** $Tag = H((K \oplus opad) || Inner)$
    * Prevents collision attacks associated with suffix constructions.

## Security Properties
* **Proof:** HMAC is secure if the underlying compression function is a Pseudo-Random Function (PRF).
* **Birthday Bound:** Applies to the output tag size.
    * *Note:* An attacker cannot verify collisions off-line without the key $k$. To the attacker, the HMAC output appears random.
* **Oracle Attacks:** Resists chosen-message attacks.

## Best Practices
* **Algorithm Choice:** Use HMAC-SHA256, HMAC-SHA512, or HMAC-SHA3.
* **Key Management:**
    * Use Cryptographically Secure Pseudo-Random Number Generators (CSPRNG) for keys.
    * Rotate keys regularly.
    * Separate keys for different purposes (e.g., don't use the encryption key for HMAC).
* **Implementation:** Use constant-time comparison to prevent timing side-channel attacks.