**Tags:** #Cybersecurity #LowLevel #Cryptography #Integrity #MAC #Hashing #SHA1 #Attacks

---

# 1. Integrity Fundamentals

## Definition
**Integrity** is the trust in the unaltered state of information.
* **Goal:** Prevent undetected or unauthorized changes.
* It is a core part of the **[[1 Information security#The CIA Triad & Beyond|CIA Triad]]** (Confidentiality, Integrity, Availability).
* *Note:* Authenticity is orthogonal to secrecy; systems often require both.

## Why Integrity Matters (Examples)
1.  **Medical:** A patient record showing "10 mg" altered to "100 mg" could lead to severe or fatal errors.
2.  **Finance & Contracts:** Altering a transaction amount or editing a digital contract after signing.
3.  **Software & Information:** Injecting malware into a software update or subtly modifying news articles to spread disinformation.

## Threats to Integrity
* **Human Error:** Typos, wrong input.
* **Hardware Failure:** Bit rot, power loss.
* **Malicious Tampering:** Insider edits.
* **Transmission Errors:** Corrupted network packets.

## Two Facets of Integrity
1.  **Data Integrity:** Ensures the content stays unchanged.
2.  **Origin Integrity (Authenticity):** Ensures the sender is legitimate.

> **Reality Check:** Integrity cannot be guaranteed proactively; it can only be verified upon inspection. If verification fails, the data is typically rejected as recovery is generally not feasible.

---

# 2. Message Authentication Codes (MAC)

## Definitions
To ensure integrity against an active adversary (like "Fran" the forger), we use MACs.
* **Authentication Algorithm ($A$):** The MAC function.
* **Verification Algorithm ($V$):** Returns "accept" or "reject".
* **Key ($k$):** A secret authentication key.
* **Tag:** The output of the function, $A_k(m)$, often denoted as $MAC_k(m)$.

**Requirement:**

$V_k(m, MAC_k(m))$ = "accept"

## The Pigeonhole Principle & 1:1 Mapping
A MAC **cannot** assign a unique tag to each message (be one-to-one).
* The message space is potentially infinite.
* The tag space is finite (fixed length).
* **Pigeonhole Principle:** You cannot assign unique tags from a finite set to an unbounded set of messages.
* *Design Choice:* Fixed-length tags are preferred for efficiency and constant-time verification.

## Adversary Models (Forgery)
The adversary's goal is to produce a valid pair $(m, \tau)$ such that $V_k(m, \tau) = \text{"accept"}$. This is called a **Forgery**.

**Types of Forgery (ranked by power):**
1.  **Universal Forgery ($U$):** Adversary can create a valid tag for *any* given message. (Strongest ability).
2.  **Selective Forgery ($S$):** Adversary can create a valid tag for a *specific* message chosen *prior* to the attack.
3.  **Existential Forgery ($E$):** Adversary can create a valid tag for *at least one* message. The message might be meaningless (e.g., a random string). (Weakest goal).

**Security Implication:**
~~~text
U => S => E
~~~
Using the contrapositive: If we prevent Existential Forgery ($\neg E$), we automatically prevent Selective and Universal forgery. Therefore, secure schemes must be **existentially unforgeable**.

---

# 3. CBC-MAC (Cipher Block Chaining MAC)

## Mechanism
* Start with an all-zero seed (IV = 0).
* Apply [[3 Block ciphers#2. CBC (Cipher Block Chaining)|CBC]] encryption using secret key $k$.
* The last block of the ciphertext ($C_n$) is used as the MAC tag.

## Vulnerability
CBC-MAC is secure for fixed-length messages but **insecure for variable-length messages**.

**The Attack:**
If an attacker knows two valid pairs $(m, t)$ and $(m', t')$, they can generate a third valid message $m''$ that also produces tag $t'$.

1.  Let $m$ be a message with tag $t$.
2.  Let $m'$ be a message with tag $t'$.
3.  Construct $m''$ by concatenating $m$ and a modified version of $m'$.
4.  **Formula:**
	$m'' = m \ ||\  (\text{first\_block\_of\_m}` \ \oplus\  t) || \ \text{rest\_of\_m}'$ 

5.  The resulting tag for $m''$ will be $t'$.

---

# 4. Hash Functions

## Basics
Hash functions map large domains to smaller ranges (fingerprints/digests).
* **Avalanche Effect:** A 1-bit change in input causes a massive change in output.
* **Efficiency:** Fast to compute.
* **Fixed Length:** Output is always the same size (e.g., 160 bits, 256 bits).

## The Birthday Paradox
In a group of just **23 people**, there is a >50% chance that two share the same birthday. This is counterintuitive (23 << 365) but occurs because we compare *pairs* of people.

**Probability Calculation:**
The probability that $n$ people have unique birthdays is:
$$P(n) = \prod_{k=0}^{n-1} \left( \frac{365-k}{365} \right)$$
The probability of at least one collision is $1 - P(n)$.
* For $n=23$, $1 - P(23) \approx 0.5073$ (50.7%).

## The Birthday Attack
Applying this to cryptography:
* If a hash function has an output space of $m$ bits ($2^m$ possible digests).
* We find a collision with >50% probability after generating roughly $2^{m/2}$ hashes.

**Approximation Formula:**
$$
k \approx 1.177 \cdot 2^{m/2}
$$
**Attack Procedure (Pseudocode):**
~~~python
# Assume H is the hash function
map = {}
while True:
    m = generate_random_message()
    h = H(m)
    if h in map:
        return (map[h], m) # Collision found!
    map[h] = m
~~~
* *Significance:* Reduces brute-force effort from $2^m$ to $2^{m/2}$.
* *Example:* A 160-bit hash (SHA-1) has security of $2^{80}$, not $2^{160}$.

---

# 5. SHA-1 (Secure Hash Algorithm 1)

**Status:** Deprecated (Collision attacks are affordable). Google announced sunsetting SHA-1 in Chrome starting 2014.

## Basics
* **Output:** 160-bit digest.
* **Block Size:** 512 bits.
* **Max Message:** $< 2^{64}$ bits.
* **Structure:** Merkle-Damgård construction.

## Pre-processing (Padding)
1.  Append a `1` bit.
2.  Append `0` bits until length $\equiv 448 \pmod{512}$.
3.  Append 64 bits representing the original message length.

## Internal State
Uses a 160-bit buffer divided into five 32-bit words ($A, B, C, D, E$).
**Initial Constants (Hex):**
* $A = 67452301$
* $B = efcdab89$
* $C = 98badcfe$
* $D = 10325476$
* $E = c3d2e1f0$

## Operation
It processes data in 512-bit blocks. Each block goes through **80 rounds**.
1.  **Word Expansion:** The 16 32-bit words of the block ($w_0...w_{15}$) are expanded into 80 words ($W_0...W_{79}$).
$$
  W_t = (W_{t-3} \oplus W_{t-8} \oplus W_{t-14} \oplus W_{t-16}) \ll 1
$$
*(Note: $\ll$ denotes left circular rotation)*

2.  **Round Function:**
    Updates the buffer $(A, B, C, D, E)$ using a non-linear function $f$, a constant $K_t$, and the expanded word $W_t$.
$$\begin{align}
Temp &= (A << 5) + f(t,B,C,D) + E + W_t + K_t\\
E &= D\\
D &= C\\
C &= (B << 30) \\
B &= A\\
A &= Temp 
\end{align}
$$

3.  **Non-linear Functions $f(t, B, C, D)$:**
    * $0 \le t \le 19$: $(B \wedge C) \vee (\sim B \wedge D)$
    * $20 \le t \le 39$: $B \oplus C \oplus D$
    * $40 \le t \le 59$: $(B \wedge C) \vee (B \wedge D) \vee (C \wedge D)$
    * $60 \le t \le 79$: $B \oplus C \oplus D$

4.  **Round Constants ($K_t$):**
    * $0-19$: $5a827999$ ($\lfloor 2^{30}\sqrt{2} \rfloor$)
    * $20-39$: $6cd9eba1$ ($\lfloor 2^{30}\sqrt{3} \rfloor$)
    * $40-59$: $8f1bbcdc$ ($\lfloor 2^{30}\sqrt{5} \rfloor$)
    * $60-79$: $ca62c1d6$ ($\lfloor 2^{30}\sqrt{10} \rfloor$)

## Final Output
After processing all blocks, the final values of $A, B, C, D, E$ are concatenated to form the 160-bit hash.