**Tags:** #Cryptography #PublicKeyEncryption #PKE #Asymmetric #RSA #ElGamal #TrapdoorPermutations #DiffieHellman #CPA #CCA #HybridEncryption #PKI #DigitalSignatures 

---

## 6.1 The Asymmetric Key Paradigm

In the previous chapter, we introduced key exchange. Unlike **[[4 Symmetric-key encryption|Symmetric-Key Encryption (SKE)]]** where both parties share the same key, **Public-Key Encryption (PKE)** uses a pair of keys $(pk, sk)$:
* **Public Key ($pk$):** Used for **encryption**. Can be shared openly.
* **Secret Key ($sk$):** Used for **decryption**. Must be kept private.

> ![[Pasted image 20260126003650.png]]
*Figure 6.1: Example of public-key encryption.*

**Formal Definition:**
Since we need to explicitly generate these paired keys, a PKE scheme is formally defined as a triple $\Pi = (KGen, Enc, Dec)$:
1.  **$KGen$:** Key Generation algorithm.
2.  **$Enc(pk, m)$:** Encryption algorithm.
3.  **$Dec(sk, c)$:** Decryption algorithm.

**Public Key Infrastructure (PKI):**
The security relies on Alice having the *correct* public key for Bob. If an attacker performs a Man-in-the-Middle attack on the key distribution, security is lost. This requires **[[#6.3 Digital Signatures|Digital Signatures]]** and **PKI**, which we will discuss later.

---

### Security Definitions for PKE

We adapt our previous security games (CPA, CCA) for the asymmetric setting. The main difference is that the **Challenger sends the Public Key ($pk$)** to the Adversary at the start of the game.

**Observation on Encryption Oracles:**
In PKE, we **do not** need to provide the adversary with an encryption oracle. Since the adversary knows $pk$, they can compute $Enc(pk, \cdot)$ themselves.
However, for encryption to be secure, it **must be randomized**. If encryption were deterministic, the adversary could simply encrypt $m_0$ and $m_1$ themselves and check which one matches the challenge ciphertext $c^*$.

#### 1. CPA Security
Let $Game^{CPA}_{\Pi, \mathcal{A}}(\lambda, b)$ be defined as:
1.  **Setup:** Challenger runs $KGen$ to get $(pk, sk)$ and sends $pk$ to $\mathcal{A}$.
2.  **Challenge:** $\mathcal{A}$ chooses $m^*_0, m^*_1$. Challenger returns $c^* = Enc(pk, m^*_b)$.
3.  **Guess:** $\mathcal{A}$ outputs $b'$.

> ![[Pasted image 20260126003806.png]]
*Figure 6.2: Graphical representation of $Game^{CPA}_{\Pi, \mathcal{A}}(\lambda, b)$. Note the absence of encryption queries.*

#### 2. CCA Security
Let $Game^{CCA}_{\Pi, \mathcal{A}}(\lambda, b)$ be defined as:
1.  **Setup:** Challenger sends $pk$ to $\mathcal{A}$.
2.  **Phase 1:** $\mathcal{A}$ has access to a **Decryption Oracle** $Dec(sk, \cdot)$.
3.  **Challenge:** $\mathcal{A}$ receives $c^* = Enc(pk, m^*_b)$.
4.  **Phase 2:** $\mathcal{A}$ may query the Decryption Oracle (except on $c^*$).
5.  **Guess:** $\mathcal{A}$ outputs $b'$.

> ![[Pasted image 20260126003832.png]]
*Figure 6.3: Graphical representation of $Game^{CCA}_{\Pi, \mathcal{A}}(\lambda, b)$.*

---

### The ElGamal Encryption Scheme

We can construct a PKE scheme directly from the **[[5 Number theory in cryptography#Assumption 5.3: DDH Assumption|DDH Assumption]]**. This is the ElGamal scheme.

##### Algorithm 6.1: ElGamal PKE Scheme
> [!algo] Algorithm 6.1
> **Setup:** A cyclic group $G$ of order $q$ with generator $g$.
>
> * **$KGen(G, g, q)$:**
>     1. Sample $x \in_R \mathbb{Z}_q$.
>     2. Set $sk = x$.
>     3. Set $pk = g^x$.
>     4. Output $(pk, sk)$.
>
> * **$Enc(pk, m)$:**
>     1. Sample a random ephemeral key $r \in_R \mathbb{Z}_q$.
>     2. Compute $c_1 = g^r$.
>     3. Compute $c_2 = pk^r \cdot m = (g^x)^r \cdot m$.
>     4. Output $c = (c_1, c_2)$.
>
> * **$Dec(sk, (c_1, c_2))$:**
>     1. Compute $s = (c_1)^{sk} = (g^r)^x = g^{rx}$.
>     2. Compute $m = c_2 \cdot s^{-1}$.
>     3. Output $m$.

**Correctness:**
$$Dec(sk, c) = c_2 \cdot (c_1^{sk})^{-1} = (g^{xr} \cdot m) \cdot (g^{rx})^{-1} = m$$

##### Theorem 6.1: Security of ElGamal
> [!abstract] Theorem 6.1
> **The ElGamal PKE scheme is CPA-secure under the DDH assumption.**

**Proof:**
We use a hybrid argument.
Let $G(\lambda, b)$ be the real CPA game. In this game, the ciphertext is $(g^r, g^{xr} \cdot m_b)$.
Define a hybrid game $H(\lambda, b)$ where the encryption is:
$$Enc_H(pk, m) = (g^r, g^z \cdot m_b)$$
where $z \in_R \mathbb{Z}_q$ is a truly random exponent.

1.  **Indistinguishability ($G \approx_c H$):**
    Suppose adversary $\mathcal{A}$ distinguishes $G$ from $H$. We build a distinguisher $\mathcal{A}'$ for the **DDH Assumption**.
    * $\mathcal{A}'$ receives a DDH tuple $(X, Y, Z)$ (where $X=g^x, Y=g^r$, and $Z$ is either $g^{xr}$ or $g^z$).
    * $\mathcal{A}'$ sets $pk = X$ and sends it to $\mathcal{A}$.
    * When $\mathcal{A}$ sends $m_0, m_1$, $\mathcal{A}'$ returns the ciphertext $c^* = (Y, Z \cdot m_b)$.
    * If $Z = g^{xr}$, this is the real game (ElGamal encryption).
    * If $Z = g^z$, this is the hybrid game (Random One-Time Pad).
    * Thus, distinguishing the games is equivalent to distinguishing DDH tuples.

2.  **Information Theoretic Security in $H$:**
    In $H(\lambda, b)$, the term masking the message is $g^z$. Since $z$ is uniform random, $g^z$ is uniform random in $G$.
    This acts like a One-Time Pad. $c_2 = g^z \cdot m_b$ reveals absolutely no information about $m_b$.
    Therefore, $H(\lambda, 0) \equiv H(\lambda, 1)$.

**Conclusion:**
$$G(\lambda, 0) \approx_c H(\lambda, 0) \equiv H(\lambda, 1) \approx_c G(\lambda, 1)$$

---

### CCA Insecurity of ElGamal (Malleability)

ElGamal is **not CCA-secure** because it is **malleable**.
The ciphertext structure allows algebraic manipulation.

**Attack:**
1.  Receive challenge ciphertext $c^* = (c^*_1, c^*_2) = (g^r, pk^r \cdot m_b)$.
2.  Adversary wants to check if $m_b = m_0$.
3.  Adversary creates a new ciphertext $c'$ by multiplying the second component by a known value $m'$:
    $$c' = (c^*_1, c^*_2 \cdot m')$$
4.  Submit $c'$ to the Decryption Oracle. (Note: $c' \neq c^*$, so this is allowed).
5.  Oracle decrypts:
    $$Dec(c') = (c^*_2 \cdot m') \cdot (c^*_1)^{-sk} = (pk^r \cdot m_b \cdot m') \cdot (g^{rx})^{-1} = m_b \cdot m'$$
6.  Adversary receives $m_{result} = m_b \cdot m'$. They compute $m_{result} \cdot (m')^{-1}$ to retrieve $m_b$ exactly.

## 6.2 RSA Encryption and Trapdoor Permutations

We study the computational security of the ever-famous **Rivest-Shamir-Adleman (RSA)** public-key encryption scheme.
The security of RSA relies on the hardness of **integer factorization**: given a composite number $n = pq$, find the primes $p, q$.

### 6.2.1 The Factoring and RSA Assumptions

##### Assumption 6.1: FACTORING Assumption
> [!def] Assumption 6.1
> The **FACTORING assumption** states that for all PPT adversaries $\mathcal{A}$:
> $$\mathbb{P}[\mathcal{A}(n) = (p, q) : p, q \in_R \mathbb{P}, n = pq] \leq negl(\lambda)$$
> where $p, q$ are $\lambda$-bit primes.

##### Algorithm 6.2: RSA PKE Scheme
> [!algo] Algorithm 6.2
> The RSA PKE scheme $\Pi = (KGen, Enc, Dec)$ is defined as:
> * **$KGen(1^\lambda)$:**
>     1. Generate two large primes $p, q$ of approximately $\lambda$ bits.
>     2. Compute $n = pq$ and $\phi(n) = (p-1)(q-1)$.
>     3. Choose $e$ such that $\gcd(e, \phi(n)) = 1$.
>     4. Compute $d \equiv e^{-1} \pmod{\phi(n)}$.
>     5. Output $pk = (n, e)$ and $sk = (n, d)$.
> * **$Enc(pk, m)$:** Output $c \equiv m^e \pmod n$.
> * **$Dec(sk, c)$:** Output $m \equiv c^d \pmod n$.

**Correctness:**
From Euler's Theorem, if $m \in \mathbb{Z}^*_n$:
$$c^d \equiv (m^e)^d \equiv m^{ed} \equiv m^{1 + k\phi(n)} \equiv m \cdot (m^{\phi(n)})^k \equiv m \cdot 1^k \equiv m \pmod n$$

**Security Issues with "Plain" RSA:**
* **Deterministic:** Since $Enc(pk, m)$ is deterministic, plain RSA is **not CPA-secure**. An adversary can simply compute $m_0^e$ and $m_1^e$ to distinguish ciphertexts.
* **Fix (Padding):** We must introduce randomness.
    * **PKCS #1.5:** Encodes message as $\bar{m} = m \,||\, r$ where $r$ is a random string.
    * Requirement: $r$ must have $\omega(\log \lambda)$ bits to avoid brute-force attacks.

### 6.2.2 Trapdoor Permutations (TDP)

The RSA function $f(x) = x^e \pmod n$ is the standard example of a **Trapdoor Permutation (TDP)**.
A TDP is a function that is easy to compute but hard to invert—unless you possess a secret "trapdoor" (the secret key).

##### Definition 6.1: Trapdoor Permutation
> [!def] Definition 6.1
> A triple $(KGen, f, f^{-1})$ is a **Trapdoor Permutation** if:
> * $(pk, sk) \in_R KGen(1^\lambda)$.
> * $f(pk, \cdot) : \{0, 1\}^n \to \{0, 1\}^n$ is a **One-Way Function** (hard to invert given only $pk$).
> * $f^{-1}(sk, y)$ efficiently recovers $x$ such that $f(pk, x) = y$.

>![[Pasted image 20260126005109.png]]
*Figure 6.4: Graphical representation of a trapdoor permutation. Easy to go forward, hard to go back without the trapdoor.*

##### Assumption 6.2: RSA Assumption
> [!def] Assumption 6.2
> The RSA function $f(x) = x^e \pmod n$ is a Trapdoor Permutation.
> (Note: This implies the FACTORING assumption, but is strictly stronger. It assumes computing $e$-th roots modulo $n$ is hard).

### 6.2.3 Constructing CPA-Secure PKE from TDP

We can build a CPA-secure encryption scheme for a single bit using a TDP and its **Hard-Core Predicate** $h$.

##### Theorem 6.2
> [!abstract] Theorem 6.2
> Let $(KGen, f, f^{-1})$ be a TDP and $h$ be a hard-core predicate for $f$.
> Define $\Pi = (KGen, Enc, Dec)$ as:
> * **$Enc(pk, m)$:** Pick random $r$. Output $(f(pk, r), h(pk, r) \oplus m)$.
> * **$Dec(sk, (c_0, c_1))$:** Recover $r = f^{-1}(sk, c_0)$. Output $m = c_1 \oplus h(pk, r)$.
>
> Then, $\Pi$ is **CPA-secure** for 1-bit messages.

**Proof:** (See Problem 6.1 in exercises).
*Idea:* $f(pk, r)$ hides $r$ (TDP property), and $h(pk, r)$ looks random given $f(pk, r)$ (Hard-core property). Thus, $h(pk, r) \oplus m$ acts like a one-time pad.

### 6.2.4 CCA Security: RSA-OAEP

To achieve CCA security (and handle longer messages), we use **Optimal Asymmetric Encryption Padding (OAEP)** (standardized in PKCS #2.0).
This uses a Feistel-like structure to randomize the input before applying the RSA function.

##### Algorithm 6.3: OAEP PKE Scheme
> [!algo] Algorithm 6.3
> Let $\Pi_{RSA}$ be the RSA primitive. Let $G, H$ be random oracles (hash functions).
> **Encryption of $m$:**
> 1. Sample random $r \in \{0, 1\}^{\lambda}$.
> 2. Pad message to get $m'$.
> 3. **Feistel Step:**
>    * $s = m' \oplus G(r)$
>    * $t = r \oplus H(s)$
> 4. **RSA Step:**
>    * $c = (s || t)^e \pmod n$.

> ![[Pasted image 20260126005434.png]]
*Figure 6.5: The OAEP padding scheme structure.*

##### Theorem 6.3
> [!abstract] Theorem 6.3
> The OAEP PKE scheme is **CCA-secure** under the RSA assumption in the **Random Oracle Model (ROM)**.

**Proof:** Omitted.

## 6.3 Digital Signatures

After proving that CCA-security is achievable also for public-key cryptography, we’re left with constructing the equivalent of MACs for PKEs. In the context of asymmetric encryption, we talk about **Digital Signatures (DS)** schemes.

The critical difference between MACs and DSs is that the signature is **publicly verifiable**: we don’t need to know the secret key that was used for signing to verify validity. This gives birth to a somewhat "mirror" concept of public-key encryption:
* **Sign:** Use the **secret key** ($sk$) to sign the message.
* **Verify:** Use the **public key** ($pk$) to verify it.

> ![[Pasted image 20260126005610.png]]
> *Figure 6.6: Example of digital signature. Alice signs with sk, Bob verifies with pk.*

### Public Key Infrastructure (PKI)
Just like MACs, digital signatures are used to authenticate a message to guarantee its integrity. Moreover, the asymmetric paradigm allows us to form a hierarchy of authentications.
If Alice wants to know Bob’s public key without being subject to a Man-in-the-Middle attack, they can ask for the key from a **Certificate Authority (CA)** to which Bob previously sent his key in an authenticated manner.
* The CA is authenticated by another CA of "higher grade".
* This chain continues until a **Root CA** (highest grade of authority).
This hierarchy is known as the **Public Key Infrastructure (PKI)**.

### RSA Digital Signatures
The simplest digital signature scheme can be constructed through RSA itself. The idea is to use the **decryption function as the signing function** and the **encryption function as the verifying function**. This works because the public and secret keys are inverses of each other ($f(f^{-1}(x)) = x$).

##### Algorithm 6.4: RSA DS scheme
> [!algo] Algorithm 6.4
> The RSA DS scheme $\Pi = (KGen, Sign, Vrfy)$ is defined as follows:
> * **$KGen(1^\lambda)$:** Generate $(pk, sk)$ where $pk = (n, e)$ and $sk = (n, d)$, with $n = pq$ and $ed \equiv 1 \pmod{\phi(n)}$.
> * **$Sign(sk, m)$:** Output signature $\sigma \equiv m^d \pmod n$.
> * **$Vrfy(pk, (m, \sigma))$:** Output 1 if $m \equiv \sigma^e \pmod n$, and 0 otherwise.

### Security of Digital Signatures (UFCMA)
For the computational security of digital signatures, we use the concept of **Unforgeability under Chosen-Message Attacks (UFCMA)**.
The game is identical to the one for MACs, with the addition that the challenger sends the **public key** to the adversary at the start.

**The Game $Game^{UFCMA}_{\Pi, \mathcal{A}}(\lambda)$:**
1.  Challenger generates $(pk, sk)$ and sends $pk$ to $\mathcal{A}$.
2.  $\mathcal{A}$ requests signatures on messages $m_1, \dots, m_t$.
3.  Challenger returns signatures $\sigma_1, \dots, \sigma_t$.
4.  $\mathcal{A}$ outputs a forgery $(m^*, \sigma^*)$.
5.  $\mathcal{A}$ wins if $Vrfy(pk, m^*, \sigma^*) = 1$ and $m^*$ was not previously queried.

> ![[Pasted image 20260126005835.png]]
> *Figure 6.7: Graphical representation of $Game^{UFCMA}_{\Pi, \mathcal{A}}(\lambda)$ for PKEs.*

**Insecurity of Basic RSA Signatures:**
It’s easy to see that the standard RSA digital signature is **not** UFCMA-secure.
Given two messages $m_1$ and $m_2$, let $\sigma_1$ and $\sigma_2$ be their signatures. Observe that:
$$(\sigma_1 \sigma_2)^e \equiv \sigma_1^e \sigma_2^e \equiv m_1 m_2 \pmod n$$
Therefore, an adversary can query $m_1$ and $m_2$, get their signatures, and then forge a valid signature for the message $m^* = m_1 m_2$ (which is $\sigma^* = \sigma_1 \sigma_2$).

---

### Full-Domain Hash (FDH)

We can fix the homomorphic issue by **hashing** the message before signing it. Generally, we can use any Trapdoor Permutation (TDP) instead of just RSA.

##### Algorithm 6.5: FDH DS scheme
> [!algo] Algorithm 6.5
> Let $(KGen, f, f^{-1})$ be a TDP and let $H$ be a hash function.
> The **Full-Domain Hash (FDH)** DS scheme $\Pi_{FDH}$ is defined as follows:
> * **$KGen(1^\lambda)$:** Generate $(pk, sk)$.
> * **$Sign(sk, m)$:** $\sigma = f^{-1}(sk, H(m))$.
> * **$Vrfy(pk, m, \sigma)$:** Output 1 if $H(m) = f(pk, \sigma)$, and 0 otherwise.

##### Theorem 6.4
> [!abstract] Theorem 6.4
> **The FDH signature scheme is UFCMA-secure in the Random Oracle Model (ROM) when constructed through a TDP.**

**Proof:**
This proof uses the **Random Oracle Model (ROM)**, assuming $H$ is a truly random function managed by an oracle.
We build a reduction from the UFCMA security of $\Pi$ to the One-Way Function (OWF) security of the TDP $f$.
Suppose there is an adversary $\mathcal{A}_H$ that forges a signature with probability $1/n^c$. We build an adversary $\mathcal{A}$ to invert $f$.

1.  **Setup:** The challenger sends a challenge $(pk, y)$ to $\mathcal{A}$ (where $y = f(sk, x)$ for unknown $x$). $\mathcal{A}$ must find $x$.
2.  $\mathcal{A}$ forwards $pk$ to $\mathcal{A}_H$.
3.  **Simulation of Random Oracle $H$:**
    $\mathcal{A}_H$ makes $q$ queries to the hash oracle. $\mathcal{A}$ must answer them.
    $\mathcal{A}$ guesses an index $j \in_R [q]$ (hoping this will be the index of the forgery message $m^*$).
    * **For query $i = j$:** Set $H(m_i) = y$ (the challenge value).
    * **For query $i \neq j$:** Pick random $x_i \in \{0, 1\}^n$, calculate $y_i = f(pk, x_i)$, and set $H(m_i) = y_i$.
4.  **Signing Queries:**
    $\mathcal{A}_H$ asks for signatures on $m_i$.
    * If $i \neq j$: $\mathcal{A}$ knows the preimage $x_i$ (since it generated it). It returns $\sigma_i = x_i$.
    * If $i = j$: $\mathcal{A}$ cannot sign this because it doesn't know $f^{-1}(y)$. It aborts.
5.  **Forgery:**
    $\mathcal{A}_H$ outputs a forgery $(m^*, \sigma^*)$.
    If $m^*$ corresponds to the $j$-th query, then $H(m^*) = y$.
    Since the signature is valid, $f(pk, \sigma^*) = H(m^*) = y$.
    Therefore, $\sigma^*$ is the preimage of $y$, solving the OWF challenge.

**Probability Analysis:**
The reduction works if $\mathcal{A}$ correctly guesses which message $\mathcal{A}_H$ will forge (index $j$). This happens with probability $1/q$.
$$\mathbb{P}[\mathcal{A} \text{ wins}] = \frac{1}{q} \cdot \mathbb{P}[\mathcal{A}_H \text{ wins}] > \frac{1}{q n^c}$$
This is non-negligible, proving the security of FDH. $\blacksquare$

## 6.4 Solved Exercises

### Problem 6.1: Hard-Core Predicate Encryption
**Problem:** Prove Theorem 6.2.
*Theorem 6.2 Recapped:* Let $(KGen, f, f^{-1})$ be a TDP and $h$ be a hard-core predicate for $f$. The scheme $Enc(pk, m) = (f(pk, r), h(pk, r) \oplus m)$ is CPA-secure for 1-bit messages.

##### Proof
**Verdict:** The theorem holds. We prove it by contradiction.

Suppose there exists a CPA adversary $\mathcal{A}_{CPA}$ that distinguishes the encryptions with probability at least $1/2 + 1/n^c$. We build an adversary $\mathcal{A}_{HPC}$ that predicts the hard-core predicate $h_{pk}$.

**Adversary $\mathcal{A}_{HPC}(pk, y)$:**
1.  **Input:** Challenger $\mathcal{C}_{HPC}$ sends $(pk, y)$ where $y = f(pk, x)$ for unknown $x$. Goal: Guess $h(pk, x)$.
2.  **Setup:** $\mathcal{A}_{HPC}$ forwards $pk$ to $\mathcal{A}_{CPA}$.
3.  **Challenge:** $\mathcal{A}_{CPA}$ sends messages $m^*_0, m^*_1 \in \{0, 1\}$.
4.  **Simulation:**
    * $\mathcal{A}_{HPC}$ picks a random bit $b^\# \in_R \{0, 1\}$ and a random challenge bit $b \in_R \{0, 1\}$.
    * It constructs the ciphertext $c^* = (y, b^\# \oplus m^*_b)$.
    * Sends $c^*$ to $\mathcal{A}_{CPA}$.
    * (Note: Effectively, $\mathcal{A}_{HPC}$ is implicitly guessing that $h(pk, x) = b^\#$).
5.  **Guess:** $\mathcal{A}_{CPA}$ outputs a guess $b'$.
6.  **Output:** If $b' = b$, $\mathcal{A}_{HPC}$ outputs $b^\#$ (assuming the guess was correct). Else, it outputs $1 - b^\#$.

**Probability Analysis:**
Let $H(x)$ be the true hard-core bit.
* **Case 1: $b^\# = H(x) \oplus m^*_b \oplus m^*_{b'}$** (Simulation is perfect).
    $\mathcal{A}_{CPA}$ sees a valid encryption of $m^*_b$.
    $\mathbb{P}[b' = b] \ge 1/2 + \epsilon$.
* **Case 2: $b^\# \neq \dots$** (Simulation is random).
    $\mathcal{A}_{CPA}$ sees a valid encryption of $m^*_{1-b}$. (Or essentially random noise if analyzed simply).
    $\mathbb{P}[b' = b] = 1/2$.

Aggregating the probabilities, if $\mathcal{A}_{CPA}$ has non-negligible advantage, $\mathcal{A}_{HPC}$ predicts $H(x)$ with probability $> 1/2 + \epsilon'$, contradicting the definition of a hard-core predicate. $\blacksquare$

---

### Problem 6.2: Signature Scheme modification
**Problem:**
Let $\Pi = (KGen, Sign, Vrfy)$ be a secure signature scheme. Consider $\Pi'$:
* $Sign'(sk, m) = (r, Sign(sk, m \oplus r), Sign(sk, r))$ for $r \leftarrow_R \{0, 1\}^n$.
* $Vrfy'(pk, m, (r, \sigma_0, \sigma_1)) = Vrfy(pk, m \oplus r, \sigma_0) \land Vrfy(pk, r, \sigma_1)$.

Prove or disprove: $\Pi'$ is UFCMA-secure assuming $\Pi$ is UFCMA-secure.

##### Solution
**Verdict:** **No**, the scheme is **not** UFCMA-secure.

**Attack Strategy:**
We can forge a signature for a new message by combining valid signatures of previous messages.
1.  **Query 1:** Ask for signature on $m_1$.
    Oracle returns $\sigma'_1 = (r_1, \sigma^{(1)}_0, \sigma^{(1)}_1)$ where:
    * $\sigma^{(1)}_0 = Sign(sk, m_1 \oplus r_1)$
    * $\sigma^{(1)}_1 = Sign(sk, r_1)$
2.  **Query 2:** Ask for signature on $m_2 = m_1 \oplus r_1$.
    Oracle returns $\sigma'_2 = (r_2, \sigma^{(2)}_0, \sigma^{(2)}_1)$ where:
    * $\sigma^{(2)}_0 = Sign(sk, m_2 \oplus r_2) = Sign(sk, m_1 \oplus r_1 \oplus r_2)$
    * $\sigma^{(2)}_1 = Sign(sk, r_2)$
3.  **Forgery:** We want to sign a new message $m^* = m_1 \oplus r_2$.
    We construct the signature $\sigma^* = (r_1, \sigma^{(2)}_0, \sigma^{(1)}_1)$.
    * Check $r$: Uses $r_1$.
    * Check $\sigma_0$: $Vrfy(pk, m^* \oplus r_1, \sigma^{(2)}_0)$.
        Note that $m^* \oplus r_1 = (m_1 \oplus r_2) \oplus r_1$.
        This matches the message signed in $\sigma^{(2)}_0$ ($m_1 \oplus r_1 \oplus r_2$). **Valid.**
    * Check $\sigma_1$: $Vrfy(pk, r_1, \sigma^{(1)}_1)$. **Valid.**

The adversary wins. $\Pi'$ is insecure.

---

### Problem 6.3: Strong Binding Signatures
**Definition (Strong Binding):** No adversary can output $(pk, m, pk', m', \sigma)$ such that $(pk, m) \neq (pk', m')$ and the signature $\sigma$ verifies for both.

**Problem:** Combine a Collision-Resistant Hash (CRH) $\mathcal{H}$ and a UFCMA signature $\Pi$ to build $\Pi^*$ that is both UFCMA-secure and Strongly Binding.

##### Solution
**Construction $\Pi_{\mathcal{H}}$:**
* $KGen(1^\lambda) = (pk, sk) \leftarrow \Pi.KGen$. Also pick seed $s \in_R \{0, 1\}^\lambda$.
* $Sign((pk, sk), m)$:
    1. Compute $h = H_s(pk || m)$.
    2. Compute $\sigma' = Sign'(sk, h)$.
    3. Output $\Sigma = (h, \sigma')$.
* $Vrfy(pk, m, (h, \sigma'))$:
    1. Check if $h = H_s(pk || m)$.
    2. Check if $Vrfy'(pk, h, \sigma') = 1$.

**Claim 1: $\Pi_{\mathcal{H}}$ is UFCMA-secure.**
*Proof:* If an adversary forges a signature for $m^*$, they produce valid $(h^*, \sigma^*)$.
This means $Vrfy'(pk, h^*, \sigma^*) = 1$.
Since $h^* = H_s(pk || m^*)$, finding a valid $\sigma^*$ for $h^*$ implies forging a signature in the underlying scheme $\Pi$. Since $\Pi$ is secure, this is negligible.

**Claim 2: $\Pi_{\mathcal{H}}$ has Strong Binding.**
*Proof:* Suppose an adversary outputs $(pk, m, pk', m', (h, \sigma))$ such that the signature is valid for both.
* Validity 1: $h = H_s(pk || m)$.
* Validity 2: $h = H_s(pk' || m')$.
* Implies: $H_s(pk || m) = H_s(pk' || m')$.
* Since $(pk, m) \neq (pk', m')$, the inputs $pk || m$ and $pk' || m'$ differ.
* This is a **collision** in $H_s$.
Since $\mathcal{H}$ is collision-resistant, this happens with negligible probability.

---

### Problem 6.4: Anonymous PKE (Key Privacy)
**Problem:**
1.  Define Key Privacy (Anonymity) for PKE.
2.  Show ElGamal is anonymous.
3.  Show RSA is **not** anonymous.

##### 1. Definition (ANON)
A PKE is anonymous if an adversary cannot distinguish which public key was used to encrypt a message.
**$Game^{ANON}_{\Pi, \mathcal{A}}(\lambda, b)$:**
* Challenger sends $pk_0, pk_1$.
* Adversary sends $m^*$.
* Challenger returns $c^* = Enc(pk_b, m^*)$.
* Adversary guesses $b$.

##### 2. ElGamal is Anonymous
**Proof (Sketch):**
ElGamal Ciphertext: $c = (g^r, pk^r \cdot m)$.
* If $b=0$, $c = (g^r, (g^{x_0})^r \cdot m)$.
* If $b=1$, $c = (g^r, (g^{x_1})^r \cdot m)$.
Under **DDH**, $(g^r, g^{rx})$ looks like $(g^r, g^z)$.
Thus, both ciphertexts look like $(g^r, g^z \cdot m)$ (pure random noise), indistinguishable from each other.

##### 3. RSA is NOT Anonymous
**Proof:**
RSA Ciphertext: $c = (m^e \pmod n)$.
* Let $pk_0 = (n_0, e_0)$ and $pk_1 = (n_1, e_1)$. Assume $n_0 < n_1$.
* **Attack:** Adversary sends message $m^*$. Challenger returns $c^*$.
    * If $c^* \ge n_0$, it **cannot** be encrypted with $pk_0$ (since outputs mod $n_0$ are always $< n_0$).
    * Thus, if $c^* \ge n_0$, output $b=1$. Else, guess.
* **Advantage:** The probability that $c \pmod{n_1}$ falls in the range $[n_0, n_1)$ is roughly $(n_1 - n_0)/n_1$. Since $n_0, n_1$ are random primes of similar length, this gap is large enough to give a significant advantage.

---

### Problem 6.5: CCA-1 Security
**Definition (CCA-1):** Decryption queries allowed *before* challenge, but not *after*.
**Questions:**
1.  CPA $\implies$ CCA-1?
2.  CCA-1 $\implies$ CPA?
3.  CCA-1 $\implies$ CCA?

##### Solutions

**1. CPA $\implies$ CCA-1? (False)**
**Counter-Example:**
Let $\Pi$ be a CPA-secure scheme. Define $\Pi'$:
* $Enc' = Enc$.
* $Dec'(sk, c)$: If $c = 0^n$, return $sk$. Else, return $Dec(sk, c)$.
$\Pi'$ is still CPA-secure (adversary cannot query decryption, and random encryption hitting $0^n$ is negligible).
However, in CCA-1, the adversary queries $c = 0^n$, gets $sk$, and breaks the scheme instantly.

**2. CCA-1 $\implies$ CPA? (True)**
CCA-1 allows *more* adversary capabilities (decryption queries) than CPA. If a scheme is secure against CCA-1, it is trivially secure against the weaker CPA adversary (who just chooses not to make decryption queries).

**3. CCA-1 $\implies$ CCA? (False)**
CCA-1 does not protect against malleability *after* seeing the challenge.
**Counter-Example:** **ElGamal**.
* ElGamal is CCA-1 secure (under appropriate assumptions/variants).
* ElGamal is **not** CCA secure (malleable). We can modify the challenge ciphertext $c^*$ into $c'$ and decrypt it in the second phase (allowed in CCA, not relevant in CCA-1).