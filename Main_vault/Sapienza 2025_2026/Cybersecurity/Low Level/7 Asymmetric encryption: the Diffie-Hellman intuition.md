# Asymmetric Cryptography: Diffie-Hellman, RSA, and PKCS Standards

**Tags:** #Cybersecurity #LowLevel #Cryptography #AsymmetricEncryption #RSA #DiffieHellman #PKCS #OAEP

---

## 1. Introduction to Asymmetric Encryption

### Concept
Unlike symmetric encryption where $K_1 = K_2$ (shared secret), asymmetric encryption uses a pair of keys:
* **Public Key ($K_1$):** Used for encryption. Publicly available.
* **Private Key ($K_2$):** Used for decryption. Kept secret by the owner.

**The Model:**
$$
\text{Alice} \xrightarrow{\text{Plaintext}} \text{Encrypt}(K_{pub}) \xrightarrow{\text{Ciphertext}} \text{Decrypt}(K_{priv}) \xrightarrow{\text{Plaintext}} \text{Bob}
$$

### Purposes
1.  **Confidentiality:** Alice encrypts with Bob's public key; only Bob can decrypt.
2.  **Key Exchange:** Securely establishing a shared symmetric key over an insecure channel.
3.  **Authentication/Integrity:** Bob encrypts (signs) with his private key; anyone can verify with his public key (basis for Digital Signatures).

---

## 2. Diffie-Hellman (DH) Key Exchange

Proposed by Diffie and Hellman in 1976 ("New Directions in Cryptography"). It allows two parties to establish a shared secret over an insecure channel.

### The Protocol
**Public Parameters:**
* $p$: A large prime number (ideally a "safe prime" $p = 2q + 1$).
* $g$: A generator of the multiplicative group $\mathbb{Z}_p^*$.

**Procedure:**
1.  **Alice:** Chooses private random $a \in [1, p-1]$. Computes public $A = g^a \mod p$. Sends $A$ to Bob.
2.  **Bob:** Chooses private random $b \in [1, p-1]$. Computes public $B = g^b \mod p$. Sends $B$ to Alice.
3.  **Shared Secret:**
    * Alice computes: $K = B^a \mod p = (g^b)^a \mod p = g^{ab} \mod p$.
    * Bob computes: $K = A^b \mod p = (g^a)^b \mod p = g^{ab} \mod p$.

### Security & Vulnerabilities
* **Discrete Logarithm Problem (DLP):** Security relies on the fact that given $g, p, A$, it is computationally hard to find $a$.
* **Man-in-the-Middle (MITM):**
    * DH has **no authentication**.
    * Attacker $T$ intercepts $A$ and sends $g^t$ to Bob.
    * $T$ intercepts $B$ and sends $g^t$ to Alice.
    * Alice agrees on key with $T$; Bob agrees on key with $T$. $T$ can decrypt/re-encrypt everything.
    * *Mitigation:* Use Authenticated DH (e.g., via Certificates).

### Forward Secrecy
* **Static DH:** Uses long-term keys. If the private key is compromised, past sessions are compromised.
* **Ephemeral DH (DHE/ECDHE):** Generates new temporary keys for every session.
    * **Perfect Forward Secrecy (PFS):** Compromise of a long-term key does not compromise past session keys.

---

## 3. Mathematical Foundations: One-Way Functions (OWF)

A function $f: \{0,1\}^* \rightarrow \{0,1\}^*$ is a **One-Way Function** if:
1.  **Easy to compute:** Polynomial time algorithm to compute $f(x)$.
2.  **Hard to invert:** For any probabilistic polynomial-time algorithm, the probability of finding $x$ given $f(x)$ is negligible.

### Examples of Hard Problems
1.  **Integer Factorization:**
    * Easy: $z = x \cdot y$ (where $x, y$ are large primes).
    * Hard: Given $z$, find $x$ and $y$.
    * *Basis for:* RSA.
2.  **Discrete Logarithm (DL):**
    * Easy: $y = g^x \mod p$.
    * Hard: Given $y, g, p$, find $x$.
    * *Basis for:* Diffie-Hellman, ElGamal.

### Modular Exponentiation
Computing $g^x \mod p$ efficiently is done using **Square-and-Multiply** (or Fast Mod Exponentiation).
* Complexity: $O(\log x)$ multiplications.

~~~c
long fastModExp(unsigned int base, unsigned int exp, long mod) {
    long result = 1;
    long b = base % mod;
    while (exp > 0) {
        if (exp & 1) result = (result * b) % mod;
        b = (b * b) % mod;
        exp >>= 1;
    }
    return result;
}
~~~

---

## 4. RSA (Rivest-Shamir-Adleman)

Invented in 1978. It implements a Public-Key Cryptosystem used for encryption and signatures.

### Key Generation
1.  Choose two large distinct primes $p$ and $q$.
2.  Compute $N = p \cdot q$. ($N$ is the modulus, length is key size, e.g., 2048 bits).
3.  Compute Euler's Totient: $\phi(N) = (p-1)(q-1)$.
4.  Choose public exponent $e$ such that $1 < e < \phi(N)$ and $\gcd(e, \phi(N)) = 1$. (Common choice: $e = 65537$).
5.  Compute private exponent $d$ such that $d \cdot e \equiv 1 \pmod{\phi(N)}$.
    * Use **Extended Euclidean Algorithm (EEA)** to find $d$.

**Keys:**
* **Public Key:** $(N, e)$
* **Private Key:** $(N, d)$

### Encryption & Decryption
* **Encryption:** $C = M^e \mod N$
* **Decryption:** $M = C^d \mod N$
* *Note:* $M < N$.

### Correctness Proof
Based on Euler's Theorem ($x^{\phi(N)} \equiv 1 \pmod N$).
$$
C^d \equiv (M^e)^d \equiv M^{ed} \equiv M^{1 + k\phi(N)} \equiv M \cdot (M^{\phi(N)})^k \equiv M \cdot 1^k \equiv M \pmod N
$$

---

## 5. Attacks on "Textbook" RSA

Textbook RSA (raw $M^e \mod N$) is insecure.

### 1. Factoring Attack
If an attacker factors $N$ into $p$ and $q$, they can compute $\phi(N)$ and then $d$.
* **Countermeasures:** Use large primes (2048+ bits), ensure $p, q$ are not too close, ensure $p-1, q-1$ have large prime factors.

### 2. Small Message & Small Exponent Attack
If $e=3$ and $M$ is small such that $M^e < N$, then $C = M^3$.
* **Attack:** Simply compute the real cube root $\sqrt[3]{C}$ to recover $M$.
* **Solution:** Padding (add random bits).

### 3. Common Modulus Attack
If the same $N$ is used with different public exponents ($e_1, e_2$) to encrypt the same message $M$.
* **Attack:** Use Extended Euclidean Algorithm to recover $M$.
* **Solution:** Never reuse $N$.

### 4. Malleability (Homomorphic Property)
RSA is multiplicative:
$$
RSA(M_1) \cdot RSA(M_2) \equiv (M_1^e) \cdot (M_2^e) \equiv (M_1 \cdot M_2)^e \equiv RSA(M_1 \cdot M_2) \pmod N
$$
* **Attack:** An attacker can modify ciphertext $C$ into $C' = C \cdot 2^e \mod N$. When decrypted, it yields $2M$. This is a specific **Chosen Ciphertext Attack (CCA)**.

### 5. Implementation Attacks
* **Timing Attacks:** Measuring time taken to compute $C^d$ to infer bits of $d$.
* **Fault Injection:** Introducing errors during computation to leak private key info.

---

## 6. PKCS Standards (Public-Key Cryptography Standards)

Developed by RSA Laboratories to ensure interoperability.

| Standard | Description | Status |
| :--- | :--- | :--- |
| **PKCS #1** | RSA Cryptography (Encryption, Padding, Signatures) | Standard (RFC 8017) |
| **PKCS #3** | Diffie-Hellman Key Agreement | Obsolete (Replaced by RFCs) |
| **PKCS #5** | Password-Based Cryptography (PBKDF2) | Widely used |
| **PKCS #7** | Cryptographic Message Syntax (CMS) | Superseded by CMS |
| **PKCS #8** | Private Key Information Syntax (.pem, .key) | Widely used |
| **PKCS #10** | Certificate Signing Request (CSR) | Ubiquitous (TLS) |
| **PKCS #11** | Cryptoki (API for HSMs/Smart Cards) | Active Standard |
| **PKCS #12** | Personal Info Exchange (.p12, .pfx) | Widely used |
| **PKCS #15** | Token Information Format | Smart card storage |

---

## 7. RSA Padding Schemes

Padding is mandatory to fix textbook RSA vulnerabilities (determinism, malleability, small message attacks).

### PKCS#1 v1.5 Padding
* **Structure:** `0x00 || 0x02 || PS || 0x00 || M`
    * `0x02`: Indicates encryption.
    * `PS`: Random non-zero padding string (at least 8 bytes).
    * `M`: Message.
* **Pros:** Simple, legacy compatibility.
* **Cons:** Vulnerable to **Bleichenbacher's Attack** (Adaptive Chosen Ciphertext Attack) due to error handling of the padding format.

### RSA-OAEP (Optimal Asymmetric Encryption Padding) - PKCS#1 v2.2
A modern, secure padding scheme designed to provide **IND-CCA** security (Indistinguishability under Chosen Ciphertext Attack).

**Structure:**
Uses a Feistel network structure with two Random Oracles (Hash functions) $G$ and $H$.
* **Inputs:** Message $m$, Random seed $r$.
* **Encryption Process:**
    1.  Pad message $m$ with zeros to fit block size.
    2.  Expand $r$ using $G$: $G(r)$.
    3.  Mask message: $X = (m || 00...0) \oplus G(r)$.
    4.  Hash masked message: $H(X)$.
    5.  Mask seed: $Y = r \oplus H(X)$.
    6.  **Result:** $X || Y$.
* **Security:** "All-or-Nothing". To recover $m$, you need $X$ and $Y$. $X$ depends on $r$, and $r$ depends on $X$ via $Y$.

**Primitives used:**
* **OS2IP:** Octet-Stream to Integer Primitive (Bytes $\to$ Int).
* **I2OSP:** Integer to Octet-Stream Primitive (Int $\to$ Bytes).
* **MGF1:** Mask Generation Function (based on a hash like SHA-256).

**Recommendation:** Always use **RSA-OAEP** for new applications.