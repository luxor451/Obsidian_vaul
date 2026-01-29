**Tags:** #Cybersecurity #LowLevel #Cryptography #StreamCiphers #RC4 #OTP #ChaCha20 #Algorithms

---

# 1. Stream Cipher Fundamentals

## Core Concept
Unlike [[3 Block ciphers|block ciphers]] which encrypt chunks of data, **Stream Ciphers** encrypt data byte-by-byte (or bit-by-bit) by combining the plaintext with a generated sequence of bits.

* **The Mechanism:**
    1.  **Seed:** A secret key (symmetric) is used as a seed.
    2.  **PRNG:** A Pseudo-Random Number Generator uses the seed to produce a long sequence of random-looking bytes called the **Keystream**.
    3.  **Encryption:** The Plaintext is combined with the Keystream using the **XOR** ($\oplus$) operation.
    * *Formula:* $Ciphertext = Plaintext \oplus Keystream$
    * *Decryption:* $Plaintext = Ciphertext \oplus Keystream$

## Types of Stream Ciphers
* **Synchronous:** The keystream depends *only* on the key (and potentially a nonce/counter). The sender and receiver must stay exactly in sync.
* **Asynchronous:** The keystream depends on the key *and* the previous ciphertext bytes (self-synchronizing).

---

# 2. RC4 (Rivest Cipher 4)

**Status:** Deprecated / Insecure
**History:** Designed by Ron Rivest in 1987. Kept as a trade secret until leaked in 1994.

## Characteristics
* **Simplicity:** Very fast and easy to implement in software (8-16 instructions per byte).
* **Structure:** Uses a variable key length to generate a permutation of all 256 possible bytes (0-255).
* **Usage:** Was extremely popular in early SSL/TLS, WEP (Wi-Fi), and Lotus Notes.

## The Algorithm
RC4 works in two distinct phases:
1.  **KSA (Key-Scheduling Algorithm):**
    * Initializes a state vector $S$ with values $0$ to $255$.
    * Scrambles $S$ using the secret key to create a random permutation.
2.  **PRGA (Pseudo-Random Generation Algorithm):**
    * Continually swaps elements in $S$ and outputs a byte $k$.
    * This byte $k$ is XORed with the plaintext.

## Weaknesses & Fall
Despite its speed, RC4 is now considered broken.
* **Biased Output:** The initial bytes of the keystream are not truly random (they reveal information about the key).
* **Key Reuse:** If the same key is used twice (nonce reuse), it is trivially broken.
* **WEP Failure:** The implementation of RC4 in WEP (early Wi-Fi encryption) was notoriously insecure due to poor IV (Initialization Vector) management.

---

# 3. The One-Time Pad (OTP)

**Status:** Theoretically Perfect / Practically Impossible
**History:** Invented by Gilbert Vernam (1917).

## The Perfect Cipher
The OTP is the only cryptosystem with **proven perfect secrecy** (Shannon's Theorem).
* **Mathematical Proof:** The probability of a message being $P$ given ciphertext $C$ is exactly the same as the probability of $P$ without knowing $C$.
    * $\mathbb{P}[P|C] = \mathbb{P}[P]$
    * The ciphertext reveals *zero* information about the plaintext.

## The Requirements
To achieve this perfection, the OTP must meet strict criteria:
1.  **Truly Random:** The key must be truly random (not pseudo-random).
2.  **Length:** The key must be at least as long as the message.
3.  **One-Time Use:** The key must *never* be reused.

## The "Two-Time Pad" Vulnerability
If a key is reused for two different messages ($P_1$ and $P_2$), the security collapses immediately via a **Known-Plaintext Attack (KPA)**:
1.  $C_1 = P_1 \oplus K$
2.  $C_2 = P_2 \oplus K$
3.  The attacker computes $C_1 \oplus C_2$:
    * $(P_1 \oplus K) \oplus (P_2 \oplus K) = P_1 \oplus P_2$
4.  The key is eliminated, leaving the XOR sum of the two plaintexts, which is easily solvable using language redundancy (crib dragging).

---

# 4. ChaCha20

**Status:** Modern Standard (RFC 8439)
**History:** Designed by Daniel J. Bernstein (2008) as an improvement on Salsa20.

## Design Philosophy
ChaCha20 mimics the One-Time Pad but makes it practical by using a fixed-length key to generate a massive, secure keystream.
* **Focus:** High speed (especially on mobile/software), simplicity, and resistance to **timing attacks** (constant time execution).
* **Adoption:** Used in TLS 1.3, WireGuard (VPN), OpenSSH, and Signal.

## Inputs
Instead of just a key, ChaCha20 uses a combination of inputs to ensure uniqueness:
* **Key:** 256-bit (32 bytes).
* **Nonce:** 96-bit (ensures unique keystreams for different messages).
* **Counter:** 32-bit (increments for every 512-bit block).

## Internal Mechanics
1.  **The State:** A $4 \times 4$ matrix of 32-bit words (512 bits total).
    * Contains: Constants, Key, Counter, and Nonce.
2.  **The Quarter-Round:** The core operation that mixes the data. It uses only three simple operations (**ARX**):
    * **A**ddition (modulo $2^{32}$)
    * **R**otation (bitwise)
    * **X**OR
3.  **Rounds:**
    * It performs **20 rounds** (10 "double-rounds") over the state.
    * Alternates between "Column" rounds and "Diagonal" rounds to ensure every bit affects every other bit (diffusion).
4.  **Output:** The final scrambled state is added to the initial state to produce 64 bytes of keystream, which is then XORed with the plaintext.