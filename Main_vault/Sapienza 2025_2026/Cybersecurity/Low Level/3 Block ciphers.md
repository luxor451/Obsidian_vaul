**Tags:** #Cybersecurity #LowLevel #Cryptography #BlockCiphers #AES #DES #FiniteFields #ModesOfOperation

---

# 1. Introduction to Block Ciphers

## Definition
A **Block Cipher** is a cryptographic protocol that encrypts fixed-size blocks of plaintext into ciphertext of the same size.
* **Inputs:** A block $P$ of plaintext of fixed size $h$ (e.g., 128 bits) and a key $k$ of fixed size.
* **Function:** A deterministic protocol $E_k$ produces a block $C$ as a function of $P$ and $k$.
* **Reversibility:** Decryption is the reverse transformation using the same key.

## Visual Analogy
* **Block Cipher:** Imagine the plaintext as a piece of **clay** and the key as a **custom mold**. The cipher shapes the clay into ciphertext. Using the same mold on the same clay always yields the exact same result (determinism).
* **[[2 Stream ciphers|Stream Cipher]]:** Acts like a **faucet** producing a continuous flow of bits (keystream).
* **Block Cipher:** Acts like a **stamping machine** processing chunks at a time.

## Block vs. Stream Ciphers
| Feature | Block Cipher | Stream Cipher |
| :--- | :--- | :--- |
| **Unit of Work** | Fixed-size blocks (e.g., 128 bits) | Single bits or bytes |
| **Ideal Use Case** | Structured data (Files, Databases, Disk Sectors) | Real-time communication |
| **Analogy** | Stamping Machine | Faucet |

**Why Block Ciphers for Structured Data?**
* **Files:** Stored in known formats (PDF, DOCX).
* **Databases:** Entries have regular sizes and are accessed in blocks.
* **Disk Sectors:** Operating systems read/write in 512-byte or 4096-byte blocks.

## Desired Properties
1.  **Invertibility:** Must be able to recover the original plaintext.
2.  **Avalanche Effect:** Small changes in input result in massive changes in output.
3.  **Non-linearity:** Avoids predictability and simple mathematical reversal.
4.  **Key Sensitivity:** Small key differences yield vastly different outputs.

---

# 2. History of Block Ciphers

## Early Standards (Late 20th Century)
* **DES (Data Encryption Standard) - 1976:**
    * 64-bit block, **56-bit key**.
    * Approved by the US Government but the key size was small enough for the NSA to brute-force.
    * *Broken:* In 1999, "Deep Crack" and distributed.net broke a DES key in 22 hours.
* **RC2 (1987):** Variable key size. Vulnerable to chosen-plaintext attacks.
* **IDEA (1991):** 64-bit block, 128-bit key. Strong but outdated.
* **Blowfish (1993):** Variable key (32-448 bits). Strong but deprecated.
* **RC5 (1994):** Variable parameters. Distributed.net is currently brute-forcing a 72-bit RC5 key.

## Iterating DES (3DES)
To fix DES's small key size, developers iterated the cipher.
* **3DES (Triple DES):** $C = E_{k3}(D_{k2}(E_{k1}(P)))$.
* **Security:** Theoretically $56 \times 3 = 168$ bits.
* **Meet-in-the-Middle (MITM) Attack:**
    * *Concept:* Brute force encryption from one side and decryption from the other, storing results in a huge lookup table to find a match.
    * *Impact:* Reduces effective security to **112 bits** ($2^{112}$).
    * *Memory:* Requires huge memory ($2^n$), making triple encryption the realistic upper bound.

## The Rise of AES
In 1997, NIST announced a competition to replace DES.
* **Finalists:** Rijndael, Serpent, Twofish, RC6, MARS.
* **Winner:** **Rijndael** (Joan Daemen & Vincent Rijmen), chosen in 2001.
* **Why?** Best balance of security ($8.5/10$), software speed ($9.5/10$), and hardware speed ($9/10$).

---

# 3. Mathematical Foundations

AES is built on specific number theory and algebraic structures to ensure security and efficiency.

## Number Theory Basics
**Euler's Theorem**
If $a$ and $n$ are coprime positive integers ($GCD(a,n)=1$), and $\phi(n)$ is Euler's totient function:
$$a^{\phi(n)} \equiv 1 \pmod n$$

**Bézout's Identity**
For nonzero integers $a$ and $b$ with $GCD(a,b)=d$, there exist integers $x$ and $y$ such that:
$$ax + by = d$$
* $x$ and $y$ can be computed using the **Extended Euclidean Algorithm**.

## Finite Fields (Galois Fields)
AES operates over a **Finite Field**, denoted $GF(p^k)$.
* **Theorem:** For every prime power $p^k$, there is a unique finite field containing exactly $p^k$ elements.
* **AES Field:** AES uses **$GF(2^8)$** (256 elements), allowing every byte to be treated as a mathematical object.

## Implementing $GF(p^k)$ Arithmetic
Elements are polynomials of degree $k-1$. For AES ($k=8$), operations are modulo an irreducible polynomial $f(x)$.

**1. Addition (XOR)**
Addition is bitwise **XOR** because coefficients are in $Z_2$ ($1+1=0$).
~~~text
Example in GF(2^5):
  x^3 + x + 1       -> (0, 1, 0, 1, 1)
+ x^4 + x^3 + x     -> (1, 1, 0, 1, 0)
--------------------------------------
  x^4 + 1           -> (1, 0, 0, 0, 1)
~~~

**2. Multiplication**
Multiplication involves polynomial multiplication followed by reduction modulo the irreducible polynomial $f(x)$.

**Detailed Calculation Example:**
* **Given:** $g(x) = (x^4 + x^3 + x + 1)(x^3 + x + 1)$ in $GF(2^5)$.
* **Modulus:** $f(x) = x^5 + x^4 + x^3 + x + 1$.

*Step A: Multiply*
$$(x^4 + x^3 + x + 1)(x^3 + x + 1)$$
$$= x^7 + x^5 + x^4 + x^6 + x^4 + x^3 + x^4 + x^2 + x + x^3 + x + 1$$
*Combine like terms (XOR: $2x^n = 0$):*
$$= x^7 + x^6 + x^5 + x^4 + x^2 + 1$$

*Step B: Modular Reduction*
Divide $(x^7 + x^6 + x^5 + x^4 + x^2 + 1)$ by $(x^5 + x^4 + x^3 + x + 1)$.
* Multiply divisor by $x^2$ to match $x^7$: $x^2(x^5 + ...) = x^7 + x^6 + x^5 + x^3 + x^2$.
* Subtract (XOR) this from the dividend:
$$(x^7 + x^6 + x^5 + x^4 + x^2 + 1) \oplus (x^7 + x^6 + x^5 + x^3 + x^2)$$
$$= x^4 + x^3 + 1$$

* **Final Result:** $1 + x^4 + x^3$.
* *Note:* Small fields often use lookup tables for efficiency.

---

# 4. AES Mechanics

AES is a symmetric block cipher designed for speed, simplicity, and compactness.

## Specifications
* **Block Size:** 128 bits (16 bytes).
* **Key Lengths:** 128, 192, or 256 bits.
* **State:** The 16 bytes are arranged in a $4 \times 4$ matrix.
$$
State = \begin{bmatrix}
A_{0,0} & A_{0,1} & A_{0,2} & A_{0,3} \\
A_{1,0} & A_{1,1} & A_{1,2} & A_{1,3} \\
A_{2,0} & A_{2,1} & A_{2,2} & A_{2,3} \\
A_{3,0} & A_{3,1} & A_{3,2} & A_{3,3}
\end{bmatrix}
$$

## Algorithm Structure
AES uses **10 rounds** for 128-bit keys.
1.  **Key Expansion:** Expands the 128-bit key into 11 round keys.
2.  **Initial Round:** `AddRoundKey`
3.  **Rounds 1-9:** `SubBytes` -> `ShiftRows` -> `MixColumns` -> `AddRoundKey`
4.  **Final Round:** `SubBytes` -> `ShiftRows` -> `AddRoundKey` (No MixColumns).

## The Four Transformations
1.  **SubBytes (Non-Linear Substitution):**
    * Operates on every byte separately.
    * $A_{i,j} \leftarrow (A_{i,j})^{-1}$ in $GF(2^8)$.
    * This S-Box provides the necessary **non-linearity**.

2.  **ShiftRows (Permutation):**
    * Cyclic shift of rows to mix data.
    * Row 0: 0 shifts. | Row 1: 1 shift left. | Row 2: 2 shifts left. | Row 3: 3 shifts left.

3.  **MixColumns (Diffusion):**
    * Columns are treated as polynomials over $GF(2^8)$ and multiplied by a fixed invertible polynomial $c(x)$.
    * $c(x) = 03x^3 + 01x^2 + 01x + 02$.

4.  **AddRoundKey:**
    * The State is XORed with the round-specific key.

---

# 5. Modes of Operation

Modes allow block ciphers to encrypt arbitrary amounts of data.

## 1. ECB (Electronic Codebook)
* **Mechanism:** Encrypts each block separately ($C_i = E_k(P_i)$).
* **Cons:** Identical plaintext blocks produce identical ciphertext. Does not conceal plaintext patterns (e.g., the "Tux" penguin image remains visible).
* **Verdict:** Insecure.
![[Pasted image 20260127181744.png]]

## 2. CBC (Cipher Block Chaining)
* **Mechanism:** $C_i = E_k(P_i \oplus C_{i-1})$.
* **IV:** Requires a random Initialization Vector (IV) for the first block.
* **Error Propagation:** A 1-bit error in ciphertext $C_{i-1}$ corrupts block $P_{i-1}$ and flips 1 bit in $P_i$.
* **Ciphertext Stealing:** Allows encryption of messages not divisible by block size without expanding ciphertext by rearranging the last two blocks.
![[Pasted image 20260127181757.png]]
## 3. OFB (Output Feedback)
* **Mechanism:** Turns the block cipher into a **Synchronous Stream Cipher**.
    * $O_i = E_k(O_{i-1})$ (Keystream generation)
    * $C_i = P_i \oplus O_i$
* **Properties:** 1-bit error in Ciphertext = 1-bit error in Plaintext (No propagation).
* **Security:** Reusing an IV destroys security.
![[Pasted image 20260127181829.png]]
![[Pasted image 20260127181838.png]]
## 4. CTR (Counter Mode)
* **Mechanism:** Encrypts a unique counter to generate keystream.
    * $Keystream_i = E_k(Seed + i)$
    * $C_i = P_i \oplus Keystream_i$
* **Properties:** Fully parallelizable (Encrypt & Decrypt) and supports Random Access.
![[Pasted image 20260127181933.png]]
## Summary Table

| Mode | Parallel Encrypt | Parallel Decrypt | Random Access | Ciphertext Bit Flip Effect |
| :--- | :---: | :---: | :---: | :--- |
| **ECB** | Yes | Yes | Yes | Block Ruined (Patterns Visible) |
| **CBC** | No | Yes | No | Block Ruined + 1 bit flipped in next |
| **OFB** | No | No | No | 1 Bit Flipped |
| **CTR** | **Yes** | **Yes** | **Yes** | 1 Bit Flipped |