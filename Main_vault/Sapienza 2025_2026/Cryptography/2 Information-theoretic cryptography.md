**Tags:** #Cryptography #InformationTheoretic #PerfectSecrecy #OneTimePad #Shannon #Entropy #MinEntropy #RandomnessExtraction #MessageAuthenticationCodes #MAC 

---

## 1. Symmetric-key Encryption (SKE)

Symmetric cryptography relies on a **shared secret key** used for both encryption and decryption. It is widely used due to its speed and efficiency, especially for large volumes of data.

### Core Components
A Symmetric-key Encryption (SKE) scheme consists of three main algorithms:
1.  **Key Generation:** A shared secret key $K \in_R \mathcal{K}$ is chosen uniformly at random.
2.  **Encryption:** A function $Enc : \mathcal{K} \times \mathcal{M} \to \mathcal{C}$ that transforms plaintext into ciphertext.
3.  **Decryption:** A function $Dec : \mathcal{K} \times \mathcal{C} \to \mathcal{M}$ that transforms ciphertext back into plaintext.

> [!check] Definition 2.1: Correctness in SKEs
> An SKE $\Pi = (Enc, Dec)$ is said to be **correct** if for every message $m \in \mathcal{M}$ and every key $K \in_R \mathcal{K}$ it holds that:
> $$Dec(K, Enc(K, m)) = m$$

### Architecture and Adversarial Model
The following diagram illustrates the standard SKE flow where Eve (the adversary) intercepts the ciphertext but does not know the key.

> **Figure 2.1:** Example of SKE with perfect secrecy. Eve intercepts $c$, but cannot derive $m$ without $K$.
![[Pasted image 20260124025818.png]]

---

## 2. Kerckhoffs’s Principle

Originally proposed in the 19th century, this is a foundational concept in modern cryptography.

> [!quote] Kerckhoffs’s Principle
> The security of a cryptographic system should depend **only on the secrecy of the key**. The system must remain secure even if everything else about it (the algorithms, the hardware, the implementation) is public knowledge.

Claude Shannon later succinctly restated this as: **“The enemy knows the system”**.

---

## 3. Perfect Secrecy

Claude Shannon proposed a formal definition of secrecy that fully respects Kerckhoffs's principle.

**The Concept:**
A cryptosystem has perfect secrecy if the ciphertext reveals **no additional information** about the plaintext. Knowing the ciphertext $c$ should not change the probability that the message was $m$.

> [!abstract] Definition 2.2: Perfect Secrecy
> Let $\Pi = (Enc, Dec)$ be an SKE. Let $M$ be a random variable over $\mathcal{M}$ and $C = Enc(K, M)$ for a key $K \in_R \mathcal{K}$.
> 
> $\Pi$ has **perfect secrecy** if $\forall m \in \mathcal{M}$ and $\forall c \in \mathcal{C}$ (where $\mathbb {P}[C = c] > 0$):
> $$\mathbb {P}[M = m] = \mathbb {P}[M = m \mid C = c]$$

---

## 4. The One Time Pad (OTP)

Shannon proved that perfect secrecy is achievable. The **One Time Pad (OTP)** is the simplest system that satisfies [[#3. Perfect Secrecy|this definition]].

### Setup
We assume the message, key, and ciphertext are binary strings of the same length $n$.
* $\mathcal{M} = \mathcal{K} = \mathcal{C} = \{0, 1\}^n$

### Functions
The system uses the bit-wise XOR ($\oplus$) operation:
* **Encryption:** $Enc(K, m) = K \oplus m$
* **Decryption:** $Dec(K, c) = K \oplus c$

### Proof of Correctness
Due to the properties of XOR (specifically $A \oplus A = 0$ and $A \oplus 0 = A$):
$$Dec(K, Enc(K, m)) = K \oplus (K \oplus m) = (K \oplus K) \oplus m = 0 \oplus m = m$$

### Equivalent Definitions of Perfect Secrecy

To fully understand perfect secrecy, we can look at it through three equivalent lenses.

> [!abstract] Lemma 2.1: Equivalent Definitions
> Let $\Pi = (Enc, Dec)$ be an SKE. Let $M$ be a random variable over $\mathcal{M}$ and $C = Enc(K, M)$ for a key $K \in_R \mathcal{K}$. The following statements are **equivalent**:
> 
> 1. $\Pi$ has **perfect secrecy**.
> 2. $M$ and $C$ are **independent**.
> 3. $\forall m, m' \in \mathcal{M}$ and $\forall c \in \mathcal{C}$ it holds that:
>    $$\mathbb{P}_{K \in_R \mathcal{K}}[Enc(K, m) = c] = \mathbb{P}_{K \in_R \mathcal{K}}[Enc(K, m') = c]$$

#### Proof of Equivalence

**1. $\implies$ 2. (Perfect Secrecy $\to$ Independence)**
Suppose $\Pi$ has perfect secrecy. By the definition of conditional probability:
$$\mathbb{P}[M = m] = \mathbb{P}[M = m \mid C = c] = \frac{\mathbb{P}[M = m, C = c]}{\mathbb{P}[C = c]}$$
Multiplying both sides by $\mathbb{P}[C=c]$:
$$\mathbb{P}[M = m] \cdot \mathbb{P}[C = c] = \mathbb{P}[M = m, C = c]$$
This is the definition of statistical independence between $M$ and $C$.

**2. $\implies$ 3. (Independence $\to$ Equal Probability)**
Suppose $M$ and $C$ are independent. Fix $m, m' \in \mathcal{M}$ and $c \in \mathcal{C}$.
$$
\begin{aligned}
\mathbb{P}_{K \in_R \mathcal{K}}[Enc(K, m) = c] &= \mathbb{P}_{K \in_R \mathcal{K}}[Enc(K, M) = c \mid M = m] \\
&= \mathbb{P}_{K \in_R \mathcal{K}}[C = c \mid M = m] \\
&= \mathbb{P}_{K \in_R \mathcal{K}}[C = c] \quad \text{(due to independence)}
\end{aligned}
$$
Proceeding in the same way for $m'$, we get $\mathbb{P}[C=c]$. Thus:
$$\mathbb{P}[Enc(K, m) = c] = \mathbb{P}[Enc(K, m') = c]$$

**3. $\implies$ 1. (Equal Probability $\to$ Perfect Secrecy)**
Assume the third statement holds. Fix $m \in \mathcal{M}$ and $c \in \mathcal{C}$.

> **Claim:** $\mathbb{P}[C = c] = \mathbb{P}[C = c \mid M = m]$
> 
> *Proof of Claim:*
> Using the Law of Total Probability:
> $$\mathbb{P}[C = c] = \sum_{m' \in \mathcal{M}} \mathbb{P}[C = c, M = m']$$
> $$= \sum_{m' \in \mathcal{M}} \mathbb{P}[C = c \mid M = m'] \cdot \mathbb{P}[M = m']$$
> $$= \sum_{m' \in \mathcal{M}} \mathbb{P}[Enc(K, m') = c] \cdot \mathbb{P}[M = m']$$
> 
> Using the hypothesis (Statement 3), we replace $m'$ with $m$:
> $$= \sum_{m' \in \mathcal{M}} \mathbb{P}[Enc(K, m) = c] \cdot \mathbb{P}[M = m']$$
> $$= \mathbb{P}[Enc(K, m) = c] \cdot \underbrace{\sum_{m' \in \mathcal{M}} \mathbb{P}[M = m']}_{1}$$
> $$= \mathbb{P}[Enc(K, m) = c] = \mathbb{P}[C = c \mid M = m] \quad \blacksquare$$

Now, using the definition of conditional probability:
$$\mathbb{P}[M = m \mid C = c] \cdot \mathbb{P}[C = c] = \mathbb{P}[M = m, C = c] = \mathbb{P}[C = c \mid M = m] \cdot \mathbb{P}[M = m]$$
Rearranging gives:
$$\mathbb{P}[M = m] = \frac{\mathbb{P}[M = m \mid C = c] \cdot \mathbb{P}[C = c]}{\mathbb{P}[C = c \mid M = m]}$$
Using our **Claim** ($\mathbb{P}[C=c] = \mathbb{P}[C=c \mid M=m]$), the fraction cancels out, leaving:
$$\mathbb{P}[M = m] = \mathbb{P}[M = m \mid C = c]$$
This confirms perfect secrecy.

---

> [!check] Proposition 2.1
> **The One Time Pad (OTP) has perfect security.**

### Proof of Perfect Secrecy for OTP

We will prove that the One Time Pad (OTP) satisfies the third equivalent definition of [[#Equivalent Definitions of Perfect Secrecy|Lemma 2.1]] (Equal Probability).

**Proof:**
Fix two messages $m, m' \in \mathcal{M}$ and a ciphertext $c \in \mathcal{C}$.
By the definition of OTP ($Enc(K, m) = K \oplus m$) and the properties of XOR:

$$
\begin{aligned}
\mathbb{P}_{K \in_R \mathcal{K}}[Enc(K, m) = c] &= \mathbb{P}_{K \in_R \mathcal{K}}[K \oplus m = c] \\
&= \mathbb{P}_{K \in_R \mathcal{K}}[K = m \oplus c] \\
&= 2^{-n} \quad (\text{since } K \text{ is uniform})
\end{aligned}
$$

Through the same argument for $m'$:
$$\mathbb{P}_{K \in_R \mathcal{K}}[Enc(K, m') = c] = 2^{-n}$$

Since both probabilities are equal ($2^{-n}$), OTP has perfect secrecy. $\blacksquare$

---

## 5. Limitations of Perfect Secrecy

While perfect secrecy is ideal, it comes with a significant inherent limitation: **The key cannot be shorter than the message.**

> [!abstract] Theorem 2.1: Shannon’s Perfect Secrecy Theorem
> Let $\Pi = (Enc, Dec)$ be a non-trivial perfectly secret SKE. Then, it holds that:
> $$|\mathcal{K}| \geq |\mathcal{M}|$$

**Proof:**
Suppose $\Pi$ is a non-trivial perfectly secret system.
1.  Fix any $c \in \mathcal{C}$ such that $\mathbb{P}[C = c] > 0$ (required for non-triviality).
2.  Let $\mathcal{M}'$ be the set of possible decryptions of $c$:
    $$\mathcal{M}' = \{Dec(K, c) \mid K \in \mathcal{K}\}$$
    Note that $|\mathcal{M}'| \le |\mathcal{K}|$ (since each key produces at most one decryption).
3.  **Contradiction Assumption:** Suppose $|\mathcal{K}| < |\mathcal{M}|$.
    Then $|\mathcal{M}'| \le |\mathcal{K}| < |\mathcal{M}|$.
4.  This implies $\exists m \in \mathcal{M} \setminus \mathcal{M}'$. There is a message $m$ that cannot be the result of decrypting $c$ with any valid key.
5.  Therefore, $\mathbb{P}[M = m \mid C = c] = 0$.
6.  However, for uniform messages, $\mathbb{P}[M = m] = \frac{1}{|\mathcal{M}|} > 0$.
7.  Since $\mathbb{P}[M = m] \neq \mathbb{P}[M = m \mid C = c]$, perfect secrecy is violated. $\blacksquare$

---

## 6. Message Authentication Codes (MACs)

We now move to the second key goal of cryptography: **Message Integrity**.
The simplest way to achieve integrity is through **Message Authentication Codes (MACs)**.

### Integrity-Only Model
This model focuses solely on integrity, not secrecy.
* It uses a **deterministic tagging function** $Tag : \mathcal{K} \times \mathcal{M} \to \mathcal{T}$.
* $\mathcal{T}$ is the **Tag Space**.

> **Figure 2.2:** Example of an integrity-only MAC. Alice sends the message $m$ and the computed tag $\tau = Tag(K, m)$.
![[Pasted image 20260124031900.png]]
### Verification Process
1.  Bob receives $(m, \tau)$.
2.  Bob re-computes $\tau' = Tag(K, m)$ using the shared key.
3.  If $\tau' = \tau$, the message is accepted as authentic.

### Unforgeability
A MAC is secure only if it is **unforgeable**:
1.  Hard to forge a valid tag $\tau$ for $m$ without $K$.
2.  Hard to forge a valid tag $\tau$ for $m$ **even if** pairs $(m', \tau')$ are known.

If an attacker can forge tags, they can intercept $(m, \tau)$, alter $m$ to $m_{new}$, forge $\tau_{new}$, and send $(m_{new}, \tau_{new})$ to Bob, successfully fooling him.

> [!def] Definition 2.3: $t$-time $\varepsilon$-statistical security
> A MAC $\Pi = (Tag)$ has **$t$-time $\varepsilon$-statistical security** when for all distinct messages $m, m_1, \dots, m_t \in \mathcal{M}$ and all tags $\tau, \tau_1, \dots, \tau_t \in \mathcal{T}$:
> 
> $$\mathbb{P}_{K \in_R \mathcal{K}}[Tag(K, m) = \tau \mid Tag(K, m_1) = \tau_1, \dots, Tag(K, m_t) = \tau_t] \leq \varepsilon$$

**Key Implications:**
* Even if an adversary sees $t$ valid message-tag pairs, the probability of successfully forging a new pair $(m, \tau)$ is at most $\varepsilon$.
* Ideally, $\varepsilon$ is small and $t$ is large.
* $\varepsilon$ cannot be 0 because a random guess always succeeds with probability at least $1/|\mathcal{T}|$.

> [!abstract] Theorem 2.2
> Any $t$-time $2^{-\lambda}$-statistically secure MAC must have a key of size:
> $$(t + 1)\lambda$$

*(Proof Omitted)*

### Pairwise Independent Hash Functions

A **1-time statistically secure MAC** is achievable through **pairwise independent hash functions**. These are families of functions where each pair forms a set of independent random variables.

> [!def] Definition 2.4: Pairwise Independent Hash Functions
> Let $\mathcal{H} = \{h_K : \mathcal{M} \to \mathcal{T}\}_{K \in \mathcal{K}}$ be a family of hash functions.
> $\mathcal{H}$ is **pairwise independent** if $\forall m, m' \in \mathcal{M}$ with $m \neq m'$, the distribution $(h_K(m), h_K(m'))$ is uniform over $\mathcal{T} \times \mathcal{T}$ when $K \in_R \mathcal{K}$.

> [!abstract] Theorem 2.3
> Let $\mathcal{H} = \{h_K : \mathcal{M} \to \mathcal{T}\}_{K \in \mathcal{K}}$ be a family of pairwise independent hash functions.
> Then, $\mathcal{H}$ induces a **1-time $1/|\mathcal{T}|$-statistically secure MAC**.

**Proof:**
Fix $m \in \mathcal{M}$ and $\tau \in \mathcal{T}$. Let the MAC be defined as $Tag(K, m) = h_K(m)$.
Since the joint probability is uniformly distributed, each individual hash function is uniformly distributed:
$$\mathbb{P}[Tag(K, m) = \tau] = \mathbb{P}[h_K(m) = \tau] = \frac{1}{|\mathcal{T}|}$$

By pairwise independence, for $m \neq m'$ and $\forall \tau, \tau' \in \mathcal{T}$:
$$
\begin{aligned}
\mathbb{P}_{K \in_R \mathcal{K}}[Tag(K, m') = \tau', Tag(K, m) = \tau] &= \mathbb{P}[Tag(K, m') = \tau'] \cdot \mathbb{P}[Tag(K, m) = \tau] \\
&= \frac{1}{|\mathcal{T}|} \cdot \frac{1}{|\mathcal{T}|}
\end{aligned}
$$

Using conditional probability:
$$
\mathbb{P}[Tag(K, m') = \tau' \mid Tag(K, m) = \tau] = \frac{\frac{1}{|\mathcal{T}|^2}}{\frac{1}{|\mathcal{T}|}} = \frac{1}{|\mathcal{T}|} \quad \blacksquare
$$

---

### Constructing Pairwise Independent Families

We can easily construct such families using prime numbers.

> [!check] Proposition 2.2
> Given a prime $p \in \mathbb{P}$, let $\mathcal{M} = \mathcal{T} = \mathbb{Z}_p$ and $\mathcal{K} = \mathbb{Z}_p^2$.
> Then, the family $\mathcal{H} = \{h_{(a,b)}\}_{(a,b) \in \mathbb{Z}_p^2}$ where **$h_{(a,b)}(m) = am + b \pmod p$** is pairwise independent.

**Proof:**
Fix $m, m' \in \mathbb{Z}_p$ with $m \neq m'$ and $\tau, \tau' \in \mathbb{Z}_p$.
We express the event $h_{(a,b)}(m) = \tau$ and $h_{(a,b)}(m') = \tau'$ as a linear system:
$$
\begin{pmatrix} m & 1 \\ m' & 1 \end{pmatrix} \begin{pmatrix} a \\ b \end{pmatrix} = \begin{pmatrix} \tau \\ \tau' \end{pmatrix}
$$
Since $m \neq m'$, the determinant $m - m' \neq 0$, meaning the matrix is invertible.
Thus, there is exactly one unique pair $(a, b)$ for any pair $(\tau, \tau')$.
$$
\mathbb{P}_{(a,b) \in_R \mathbb{Z}_p^2} \left[ \begin{pmatrix} a \\ b \end{pmatrix} = \begin{pmatrix} m & 1 \\ m' & 1 \end{pmatrix}^{-1} \begin{pmatrix} \tau \\ \tau' \end{pmatrix} \right] = \frac{1}{|\mathbb{Z}_p^2|}
$$

---

## 7. Randomness Extraction

Randomness is crucial for secure cryptography, but true randomness is hard to generate.
**Randomness Extraction** is the process of producing a short, unpredictable sequence of bits (e.g., 256 bits) from a physical source (noise, humidity, etc.). This seed is often expanded using a **Pseudorandom Generator (PRG)**.

### Von Neumann’s Extractor
This extractor yields a **fair random coin** from an **unpredictable unfair one**.
Let $B \in \{0, 1\}$ be an unfair coin with $\mathbb{P}[B=0] = p \neq 1/2$.

**Procedure:**
1. Sample two bits $b_1, b_2$ from $B$.
2. If $b_1 = b_2$, discard and repeat ($Y = ?$).
3. If $b_1 = 0, b_2 = 1$, output $Y = 1$.
4. If $b_1 = 1, b_2 = 0$, output $Y = 0$.

**Analysis:**
The probability of outputting 0 or 1 is now equal:
$$\mathbb{P}[Y=0] = p(1-p) \quad \text{and} \quad \mathbb{P}[Y=1] = (1-p)p$$
The probability of failure ($b_1 = b_2$) decreases exponentially with the number of attempts $m$, ensuring $Y$ eventually becomes a fair coin.

### Generalizing Extraction: Min-Entropy
We want to design extractors for *any* distribution. To do this, we measure the "goodness" of a source using **Min-Entropy**. This represents the minimum amount of information (in bits) provided by observing the variable.

> [!def] Definition 2.5: Min-Entropy
> Given a random variable $X$, the min-entropy is:
> $$H_\infty(X) = -\log \max_x \mathbb{P}[X = x]$$

* **Comparison to Standard Entropy:** Standard entropy uses the *expected value* of $\log(1/p_i)$. Min-entropy uses the *minimum* value of $\log(1/p_i)$, which corresponds to the most likely outcome ($\max p_i$).
* **Example:**
    * If $X \sim U_n$ (uniform), $H_\infty(X) = n$.
    * If $X$ is constant, $H_\infty(X) = 0$.

### Impossibility of Deterministic Extraction

We asked if there exists a deterministic extractor $Ext^*$ that outputs a uniform distribution from any source with sufficient min-entropy. The answer is **negative**.

> [!fail] Proposition 2.3
> There is **no** extractor $Ext$ such that for every random variable $X$ over $\{0, 1\}^n$ with $H_\infty(X) \geq n - 1$, it holds that $Ext(X)$ is a uniform distribution over $\{0, 1\}$.

**Proof:**
Let $Ext : \{0, 1\}^n \to \{0, 1\}$ be any extractor. Let $b \in \{0, 1\}$ be the output that maximizes the size of the preimage (the set of inputs that map to $b$):
$$b = \arg \max_{b' \in \{0,1\}} |Ext^{-1}(b')|$$

By the **Pigeonhole Principle**, the size of this set must be at least half the input space:
$$|Ext^{-1}(b)| \geq \frac{|\{0,1\}^n|}{2} = 2^{n-1}$$

Now, let $X$ be a random variable uniform over this set $Ext^{-1}(b)$.
* Since $X$ is uniform over a set of size $2^{n-1}$, its min-entropy is $H_\infty(X) \geq n-1$.
* However, $Ext(X)$ is always $b$. This is a constant distribution, which is definitely **not uniform**.

Thus, for any extractor, there exists a specific high-entropy source that makes it fail. $\blacksquare$

---

### $\varepsilon$-Closeness

Since perfect uniformity is impossible, we aim for a distribution that is "close enough."

> [!def] Definition 2.6: $\varepsilon$-closeness
> Let $X, X'$ be two random variables over the same set. We say $X$ and $X'$ are **$\varepsilon$-close** ($X \sim_\varepsilon X'$) if their **Statistical Distance (SD)** is at most $\varepsilon$:
> 
> $$SD(X; X') = \frac{1}{2} \sum_x |\mathbb{P}[X = x] - \mathbb{P}[X' = x]| \leq \varepsilon$$

**Adversarial Interpretation:**
$\varepsilon$-closeness implies that even an **unbounded adversary** $\mathcal{A}$ cannot distinguish between $X$ and $X'$ with an advantage greater than $\varepsilon$:
$$|\mathbb{P}[\mathcal{A}(x) = 1 : x \in_R X] - \mathbb{P}[\mathcal{A}(x) = 1 : x \in_R X']| \leq \varepsilon$$

---

### Seeded Extractors

To bypass the impossibility result of [[#Impossibility of Deterministic Extraction|Proposition 2.3]], we introduce a public random **seed**.

> [!def] Definition 2.7: Seeded Extractor
> Let $S$ be a random variable (the **seed**) over $\{0, 1\}^d$.
> A function $Ext : \{0, 1\}^d \times \{0, 1\}^n \to \{0, 1\}^\ell$ is a **$(k, \varepsilon)$-extractor** if for every random variable $X$ with $H_\infty(X) \geq k$:
> 
> $$(S, Ext(S, X)) \sim_\varepsilon (S, U_\ell)$$
> 
> where $S \sim U_d$ (uniform seed) and $U_\ell$ is uniform output.

**Note:** The definition requires the output to be close to uniform **even given the seed**. This allows the seed to be public.

### Collision Probability

We measure the "goodness" of a distribution using **Collision Probability**: the probability that two independent samples from the same variable are equal.
$$Col(Y) = \mathbb{P}[Y = Y'] = \sum_{y \in \mathcal{Y}} \mathbb{P}[Y = y]^2$$

We now prove a crucial relationship: if the collision probability is low (close to that of a uniform distribution), the statistical distance from uniform is small.

> [!abstract] Proposition 2.4
> Let $Y$ be a random variable over $\mathcal{Y}$. Assume $Col(Y) \leq \frac{1}{|\mathcal{Y}|} (1 + 4\varepsilon^2)$ for some $\varepsilon > 0$.
> Then, $SD(Y, U) \leq \varepsilon$, where $U$ is uniform over $\mathcal{Y}$.

**Proof:**
Recall the definition of Statistical Distance between $Y$ and Uniform $U$:
$$SD(Y; U) = \frac{1}{2} \sum_{y \in \mathcal{Y}} \left|\mathbb{P}[Y = y] - \frac{1}{|\mathcal{Y}|}\right|$$

Let $q_y = \mathbb{P}[Y = y] - \frac{1}{|\mathcal{Y}|}$.
Let $s_y = \text{sign}(q_y) \in \{-1, 1\}$.
We can rewrite SD using vector notation vectors $\vec{q}$ and $\vec{s}$ of length $|\mathcal{Y}|$:
$$SD = \frac{1}{2} \sum q_y s_y = \frac{1}{2} \langle \vec{q}, \vec{s} \rangle$$

By the **Cauchy-Schwarz inequality** ($\langle \vec{q}, \vec{s} \rangle \le \|\vec{q}\| \cdot \|\vec{s}\|$):
$$SD \leq \frac{1}{2} \sqrt{\sum q_y^2} \cdot \sqrt{\sum s_y^2}$$

1.  **Analyze $\|\vec{s}\|$:** Since $s_y^2 = 1$, the sum is simply the number of elements: $\sum_{y} s_y^2 = |\mathcal{Y}|$.
2.  **Analyze $\|\vec{q}\|$ (Claim):**
    $$
    \begin{aligned}
    \sum q_y^2 &= \sum \left(\mathbb{P}[Y=y] - \frac{1}{|\mathcal{Y}|}\right)^2 \\
    &= \sum \left(\mathbb{P}[Y=y]^2 - \frac{2}{|\mathcal{Y}|}\mathbb{P}[Y=y] + \frac{1}{|\mathcal{Y}|^2}\right) \\
    &= \underbrace{\sum \mathbb{P}[Y=y]^2}_{Col(Y)} - \frac{2}{|\mathcal{Y}|}\underbrace{\sum \mathbb{P}[Y=y]}_{1} + \sum \frac{1}{|\mathcal{Y}|^2} \\
    &= Col(Y) - \frac{2}{|\mathcal{Y}|} + \frac{1}{|\mathcal{Y}|} = Col(Y) - \frac{1}{|\mathcal{Y}|}
    \end{aligned}
    $$
    Using the hypothesis $Col(Y) \leq \frac{1}{|\mathcal{Y}|}(1 + 4\varepsilon^2)$:
    $$\sum q_y^2 \leq \frac{1}{|\mathcal{Y}|} + \frac{4\varepsilon^2}{|\mathcal{Y}|} - \frac{1}{|\mathcal{Y}|} = \frac{4\varepsilon^2}{|\mathcal{Y}|}$$

**Conclusion:**
Substitute these back into the Cauchy-Schwarz inequality:
$$
SD \leq \frac{1}{2} \sqrt{\frac{4\varepsilon^2}{|\mathcal{Y}|}} \cdot \sqrt{|\mathcal{Y}|} = \frac{1}{2} \cdot \frac{2\varepsilon}{\sqrt{|\mathcal{Y}|}} \cdot \sqrt{|\mathcal{Y}|} = \varepsilon \quad \blacksquare
$$

### The Left-Over Hash Lemma

This lemma connects min-entropy, pairwise independence, and extraction. It states that applying a pairwise independent hash function to a source with sufficient min-entropy yields a distribution close to uniform.

> [!abstract] Lemma 2.2: Left-Over Hash Lemma
> Let $\mathcal{H} = \{h_S : \{0,1\}^n \to \{0,1\}^\ell\}_{S \in \{0,1\}^d}$ be a pairwise independent hash family.
> Let $X$ be a random variable with $H_\infty(X) \geq k$.
> Then, for each seed $S \in \{0,1\}^d$, the function **$Ext(S, X) = h_S(X)$** is a **$(k, \varepsilon)$-extractor** provided:
> 
> $$k \geq \ell + 2 \log \left(\frac{1}{\varepsilon}\right) - 2$$

**Proof:**
Fix $S \in \{0,1\}^d$. Let $Y = (S, Ext(S, X)) = (S, h_S(X))$.
Let $Y'$ be an independent copy: $Y' = (S', h_{S'}(X'))$.

Calculate the **Collision Probability** of $Y$:
$$
\begin{aligned}
Col(Y) &= \mathbb{P}[Y = Y'] \\
&= \mathbb{P}[S = S', h_S(X) = h_{S'}(X')] \\
&= \mathbb{P}[S = S', h_S(X) = h_{S}(X')] \quad (\text{since } S\text{ and } S' \text{ are independent from } X \text{ and } X') \\
&= \mathbb{P}[S = S'] \cdot \mathbb{P}[h_S(X) = h_S(X')] \\
&= 2^{-d} \cdot \mathbb{P}[h_S(X) = h_S(X')]
\end{aligned}
$$

We split the probability based on whether $X = X'$:
$$
\begin{aligned}
\mathbb{P}[h_S(X) = h_S(X')] &= \mathbb{P}[X = X', h_S(X) = h_S(X')] + \mathbb{P}[X \neq X', h_S(X) = h_S(X')] \\
&= \mathbb{P}[X = X'] + \mathbb{P}[X \neq X', h_S(X) = h_S(X')]
\end{aligned}
$$

1.  **Case $X = X'$:** Since $H_\infty(X) \geq k$, the collision probability is bounded by $2^{-k}$. Thus $\mathbb{P}[X = X'] \leq 2^{-k}$.
2.  **Case $X \neq X'$:** Since $\mathcal{H}$ is pairwise independent, the probability of collision for distinct inputs is uniform over the output space size $2^\ell$. Thus probability is $2^{-\ell}$.

Substituting back:
$$
\begin{aligned}
Col(Y) &\leq 2^{-d} (2^{-k} + 2^{-\ell}) \\
&\leq \frac{1}{2^{d+\ell}} (2^{\ell-k} + 1)
\end{aligned}
$$

Using the hypothesis for $k$:
$$
Col(Y) \leq \frac{1}{|Y|} (1 + 4\varepsilon^2)
$$
By [[#Collision Probability|Proposition 2.4]], this implies $SD(Y, U) \leq \varepsilon$. $\blacksquare$

---



## 8. Exercises

### Problem 2.1: Uniform Ciphertext Distribution vs. Perfect Secrecy

**Question:**
Prove or disprove: A SKE $\Pi = (Enc, Dec)$ has perfect secrecy **if and only if** $\forall c_0, c_1 \in \mathcal{C}$, it holds that $\mathbb{P}[C = c_0] = \mathbb{P}[C = c_1]$.

> [!fail] Verdict: False
> The claim implies that perfect secrecy *requires* every possible string in the ciphertext space to occur with equal probability. We can construct a counter-example where this is not true, yet perfect secrecy is maintained.

**Counter-Example Construction:**
Let $\mathcal{M} = \mathcal{K} = \{0, 1\}^n$ and $\mathcal{C} = \{0, 1\}^{n+1}$.
Define the scheme $\Pi$:
* **Encryption:** $Enc(K, m) = 0 \,||\, (m \oplus K)$. (Prepend a '0' bit).
* **Decryption:** $Dec(K, b \,||\, c') = c' \oplus K$. (Ignore the first bit).

**Disproof:**
1.  **Check Uniformity Condition:**
    * Any ciphertext $c$ starting with $1$ (e.g., $1 || \dots$) has probability $\mathbb{P}[C=c] = 0$ because $Enc$ always outputs a $0$ prefix.
    * Any ciphertext $c$ starting with $0$ has probability $\mathbb{P}[C=c] = 2^{-n}$.
    * Since $0 \neq 2^{-n}$, the ciphertext distribution is **not uniform** over $\mathcal{C}$.
2.  **Check Perfect Secrecy:**
    * Despite the non-uniform ciphertext distribution, does valid ciphertext reveal info about $m$?
    * For any valid ciphertext $c = 0 || c'$, we have:
        $$\mathbb{P}[M=m \mid C=c] = \mathbb{P}[M=m \mid 0 || (m \oplus K) = 0 || c'] = \mathbb{P}[K = m \oplus c'] = 2^{-n}$$
    * Since the posterior probability equals the prior probability, $\Pi$ **has perfect secrecy**.

---

### Problem 2.2: Perfect Secrecy vs. Indistinguishability

##### 1. Concept

The "Game" is a thought experiment used to define security formally. It models a scenario where an attacker ($\mathcal{A}$) tries to determine which of two messages was encrypted.

If the attacker cannot win this game with a probability better than random guessing ($1/2$), the encryption scheme ($\Pi$) is considered secure.

##### 2. The Procedure

The game, denoted as $Game^{1-time}_{\Pi, \mathcal{A}}(\lambda, b)$, typically proceeds in three phases:

1.  **Setup Phase:**
    * The **Adversary** ($\mathcal{A}$) chooses two distinct messages, $m_0$ and $m_1$, of the same length from the message space $\mathcal{M}$.
    * $\mathcal{A}$ sends these messages to the **Challenger**.

2.  **Challenge Phase:**
    * The **Challenger** generates a random key $K \in_R \mathcal{K}$.
    * The Challenger selects a bit $b \in \{0, 1\}$.
        * If the game is run with $b=0$, they encrypt $m_0$.
        * If the game is run with $b=1$, they encrypt $m_1$.
    * The Challenger computes the ciphertext $c^* = Enc(K, m_b)$.
    * The Challenger sends $c^*$ back to the Adversary.

3.  **Guess Phase:**
    * The Adversary analyzes $c^*$ and outputs a guess $b' \in \{0, 1\}$.
    * If $b' = b$, the Adversary **wins**.


**Question:**
Prove that a SKE $\Pi$ has perfect secrecy **if and only if** for every unbounded adversary $\mathcal{A}$, the advantage in the **One-Time Indistinguishability Game** is 0.
$$Game_{\Pi, \mathcal{A}}^{1-time}(\lambda, 0) \equiv Game_{\Pi, \mathcal{A}}^{1-time}(\lambda, 1)$$

> [!success] Proof
> We need to show that the adversary's probability of guessing the correct bit $b$ is exactly $1/2$ regardless of the messages chosen.

**Step 1: Define the Winning Condition**
The games are indistinguishable iff for all adversaries $\mathcal{A}'$:
$$
\left| \mathbb{P}[\mathcal{A}' \text{ outputs } 1 \mid b=0] - \mathbb{P}[\mathcal{A}' \text{ outputs } 1 \mid b=1] \right| = 0
$$
This implies that the distribution of ciphertexts produced by $m_0$ is identical to the distribution produced by $m_1$.

**Step 2: Link to Probability**
The condition above holds if and only if for **all** message pairs $m_0, m_1 \in \mathcal{M}$ chosen by the adversary, and for **any** ciphertext $c^* \in \mathcal{C}$:
$$\mathbb{P}[Enc(K, m_0) = c^*] = \mathbb{P}[Enc(K, m_1) = c^*]$$

**Step 3: Conclusion**
This equation is exactly the **Third Equivalent Definition of Perfect Secrecy** (from [[#Equivalent Definitions of Perfect Secrecy|Lemma 2.1]]).
Therefore, Perfect Secrecy $\iff$ Indistinguishable Ciphertexts. $\blacksquare$

---

### Problem 2.3: Statistical Insecurity of Affine MACs

**Question:**
Given a prime $p$, let $\mathcal{M} = \mathcal{T} = \mathbb{Z}_p$ and $\mathcal{K} = \mathbb{Z}_p^2$.
Prove that the hash family $\mathcal{H} = \{h_{(a,b)}(m) = am + b \pmod p\}$ **cannot** be a [[#Unforgeability|2-time statistically secure MAC]].

> [!abstract] Analysis
> A MAC is "2-time secure" if seeing 2 valid pairs doesn't help forge a 3rd. We will show that seeing 2 pairs allows the adversary to **fully recover the key**, making forgery trivial (probability 1).

**Attack Scenario:**
1.  **Observation:** The adversary observes two valid message-tag pairs $(m_1, \tau_1)$ and $(m_2, \tau_2)$ where $m_1 \neq m_2$.
2.  **System of Equations:** The adversary sets up the linear system:
    $$
    \begin{cases}
    a \cdot m_1 + b \equiv \tau_1 \pmod p \\
    a \cdot m_2 + b \equiv \tau_2 \pmod p
    \end{cases}
    $$
3.  **Key Recovery:**
    * Subtract the second equation from the first:
        $$a(m_1 - m_2) \equiv \tau_1 - \tau_2 \pmod p$$
    * Since $m_1 \neq m_2$ and $p$ is prime, $(m_1 - m_2)$ has a modular inverse (since $\mathbb{Z_{p}}$ is a field)
    * Solve for $a$:
        $$a \equiv (\tau_1 - \tau_2)(m_1 - m_2)^{-1} \pmod p$$
    * Solve for $b$ using $a$:
        $$b \equiv \tau_1 - a \cdot m_1 \pmod p$$
4.  **Forgery:**
    The adversary now possesses the specific key $K = (a, b)$. They can compute $\tau = am + b$ for *any* new message $m$ with probability $1$.

**Conclusion:**
$$\mathbb{P}_{K}[Tag(m) = \tau \mid (m_1, \tau_1), (m_2, \tau_2)] = 1$$
Since $1 > \varepsilon$ for any negligible $\varepsilon$, the scheme is **not** 2-time secure.

---

### Problem 2.4: Constructing a 3-wise Independent Hash Family

**Question:**
Construct a 3-wise independent hash function family and prove its correctness.

> [!example] Construction
> Given prime $p$, let $\mathcal{M} = \mathcal{T} = \mathbb{Z}_p$ and $\mathcal{K} = \mathbb{Z}_p^3$.
> Define the family using a quadratic polynomial:
> $$\mathcal{H} = \{h_{(a,b,c)}(m) = am^2 + bm + c \pmod p\}_{(a,b,c) \in \mathbb{Z}_p^3}$$

**Proof of 3-wise Independence:**
We must prove that for any 3 distinct messages $x_1, x_2, x_3$ and any 3 target values $\tau_1, \tau_2, \tau_3$, the probability of mapping $x_i \to \tau_i$ is uniform ($1/|\mathcal{K}|$).

**1. Set up the System:**
$$
\begin{cases}
a x_1^2 + b x_1 + c = \tau_1 \\
a x_2^2 + b x_2 + c = \tau_2 \\
a x_3^2 + b x_3 + c = \tau_3
\end{cases}
\implies
\begin{pmatrix} x_1^2 & x_1 & 1 \\ x_2^2 & x_2 & 1 \\ x_3^2 & x_3 & 1 \end{pmatrix} 
\begin{pmatrix} a \\ b \\ c \end{pmatrix} = 
\begin{pmatrix} \tau_1 \\ \tau_2 \\ \tau_3 \end{pmatrix}
$$

**2. Analyze the Matrix (Vandermonde):**
The matrix on the left is a **Vandermonde Matrix**. Its determinant is given by the product of the differences of the inputs:
$$\det(V) = (x_1 - x_2)(x_1 - x_3)(x_2 - x_3)$$

**3. Invertibility:**
Since $x_1, x_2, x_3$ are distinct values in a field $\mathbb{Z}_p$:
* $(x_i - x_j) \neq 0$ for all $i \neq j$.
* Therefore, $\det(V) \neq 0$.
* The matrix is invertible.

**4. Probability Calculation:**
Because the matrix is invertible, there exists exactly **one unique key triplet** $(a, b, c)$ that satisfies the system for any given $\tau$'s.
$$\mathbb{P}_{(a,b,c) \in_R \mathbb{Z}_p^3} \left[ \text{System holds} \right] = \frac{\text{\#valid keys}}{\text{\#total keys}} = \frac{1}{|\mathbb{Z}_p^3|}$$

This matches the definition of 3-wise independence. $\blacksquare$