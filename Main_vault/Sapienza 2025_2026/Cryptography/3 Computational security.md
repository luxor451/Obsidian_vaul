# 3. Computational Security

**Tags:** #Cryptography #ComputationalSecurity #OneWayFunction #OWF #ImpagliazzoWorlds #PseudorandomGenerator #PRG #ComputationalIndistinguishability #NegligibleFunction #HybridArgument 
**Topic:** 3.1 One-way functions and Impagliazzo’s worlds & 3.2 Pseudorandom generators

---

## 3.1 One-way Functions and Impagliazzo’s Worlds

In previous chapters, we established that Information-Theoretic security (like the [[2 Information-theoretic cryptography#4. The One Time Pad (OTP)|One-Time Pad]]) has severe limitations:
* **Key Length:** Keys must be at least as long as the message ($|K| \ge |M|$).
* **Randomness:** We cannot extract more randomness than the [[2 Information-theoretic cryptography#Definition 2.5: Min-Entropy|min-entropy]] of the source.

To overcome these limitations and achieve practical security, we shift to **Computational Security**. This paradigm relies on two fundamental assumptions:
1.  **Bounded Adversary:** The adversary has limited resources (polynomial time and memory).
2.  **Hard Problems:** There exist problems that are easy to check but hard to solve (e.g., factoring large numbers).

**The Reductionist Approach:**
Security proofs typically follow a specific logic: *"If problem $X$ is hard for efficient solvers, then cryptosystem $\Pi$ is secure."*
By contrapositive: If $\Pi$ is broken, then we have found an efficient solution to $X$.

### The Role of $P \neq NP$
Ideally, we might assume $P \neq NP$. However, while $P \neq NP$ is necessary, it is **not sufficient** for cryptography. We need a stronger primitive called a **One-Way Function (OWF)**.
* **Concept:** A function $f$ is one-way if it is easy to compute ($f(x) \to y$) but hard to invert ($f^{-1}(y) \to x$).
* **Implication:** If OWFs exist, then $P \neq NP$. (If $P=NP$, we could invert any function by efficiently verifying the input-output pair).

### Impagliazzo’s Worlds
Russell Impagliazzo described five hypothetical "worlds" to categorize the state of complexity theory and cryptography.

| World | State of $P$ vs $NP$ | OWFs Exist? | Cryptography Status |
| :--- | :--- | :--- | :--- |
| **Algorithmica** | $P = NP$ | No | Impossible. No hard problems exist. |
| **Heuristica** | $P \neq NP$ | No | Impossible. Hard problems exist in the *worst case*, but are easy on *average*. |
| **Pessiland** | $P \neq NP$ | No | **The worst scenario.** Hard problems exist (we can't solve them), but they don't yield OWFs (we can't use them for security). |
| **Minicrypt** | $P \neq NP$ | **Yes** | **[[4 Symmetric-key encryption|Symmetric Crypto]] exists.** (PRGs, Private-key encryption). Public-key crypto is impossible. |
| **Cryptomania** | $P \neq NP$ | **Yes** | **[[6 Public-key encryption|Public-Key Crypto]] exists.** Secure communication is possible between parties who have never met. |

### Computational Model
We model efficient computation using **Probabilistic Polynomial Time (PPT)** Turing Machines. These machines run in time polynomial in the input length $n$ ($poly(n)$) and can use random bits.

**Negligible Functions:**
A function $\varepsilon: \mathbb{N} \to \mathbb{R}$ is **negligible** if it decreases faster than the inverse of any polynomial.
$$\forall c \in \mathbb{N}, \exists \lambda_0 \text{ such that } \forall \lambda > \lambda_0 : \varepsilon(\lambda) < \frac{1}{\lambda^c}$$

##### Definition 3.1: One-way functions
> [!def] Definition 3.1
> A function $f: \{0,1\}^n \to \{0,1\}^m$ is a **One-Way Function (OWF)** if:
> 1.  **Efficiency:** $f$ can be computed in $poly(n)$-time.
> 2.  **Hardness to Invert:** For every PPT algorithm $\mathcal{A}$, there exists a negligible function $negl(n)$ such that:
>     $$\mathbb{P}_{x \in_R U_n} [f(x') = y : y = f(x); x' \leftarrow \mathcal{A}(y)] \leq negl(n)$$
>
> *(Intuitively: The probability that $\mathcal{A}$ finds any preimage $x'$ mapping to $y$ is negligible.)*

---

## 3.2 Pseudorandom Generators

Modern systems rely on **Pseudorandomness**: efficient, deterministic algorithms that generate sequences that *look* unpredictable but are derived from a short, truly random seed.

### Computational Indistinguishability
We replace statistical distance (perfect security) with **Computational Indistinguishability** ($\approx_c$). This asks if a *computationally bounded* adversary can tell two distributions apart.

We define this over **Probability Ensembles** $X = \{X_n\}_{n \in \mathbb{N}}$ (sequences of distributions for increasing input lengths).

##### Definition 3.2: Computational indistinguishability
> [!def] Definition 3.2
> Two ensembles $X$ and $Y$ are **computationally indistinguishable** ($X \approx_c Y$) if for every PPT algorithm $D$ (the distinguisher), there is a negligible function $negl(n)$ such that:
> $$|\mathbb{P}[D(z) = 1 : z \in_R X_n] - \mathbb{P}[D(z) = 1 : z \in_R Y_n]| \leq negl(n)$$

**Properties:**
* **Transitivity:** If $X \approx_c Y$ and $Y \approx_c Z$, then $X \approx_c Z$.
* **Closure under Reductions:** For any efficient function $f$, if $X \approx_c Y$, then $f(X) \approx_c f(Y)$.

### The PRG Definition
A PRG expands a short random seed into a longer pseudorandom string.

##### Definition 3.3: Pseudorandom generator (PRG)
> [!def] Definition 3.3
> Let $G: \{0,1\}^n \to \{0,1\}^{n+\ell}$ be a deterministic function with expansion factor $\ell \geq 1$.
> $G$ is a **Pseudorandom Generator** if:
> 1.  **Efficiency:** $G$ can be computed in $poly(n)$-time.
> 2.  **Pseudorandomness:** The output is computationally indistinguishable from uniform:
>     $$G(U_n) \approx_c U_{n+\ell}$$

This means that although $G$ outputs a tiny subset of all possible strings of length $n+\ell$, no efficient adversary can distinguish that subset from the full uniform space.

### Discussion on PRGs
The definition of a PRG implies a very strong property: **Determinism**.
Since $G$ is a deterministic function, it maps a seed space of size $2^n$ to a much larger output space $2^{n+\ell}$. This means $G(U_n)$ is supported on a tiny, sparse subset of $\{0, 1\}^{n+\ell}$. Despite this sparsity, the output is indistinguishable from a truly uniform distribution to any efficient observer.

In practice, we use a **[[2 Information-theoretic cryptography#7. Randomness Extraction|Randomness Extractor]]** (from Chapter 2) to derive a short initial seed from a physical source, and then pass it to a PRG to "expand" it to any desired length.

### The Hybrid Argument
To prove that we can generate *many* pseudorandom bits, we rely on a powerful proof technique called the **Hybrid Argument**.
We define a sequence of distributions $H_0, \dots, H_\ell$ such that:
1.  **Start:** $H_0$ is the output of our candidate generator ($G_\ell(U_n)$).
2.  **End:** $H_\ell$ is the target uniform distribution ($U_{n+\ell}$).
3.  **Step:** Each neighbor $H_i$ is computationally indistinguishable from $H_{i+1}$ ($H_i \approx_c H_{i+1}$).
4.  **Structure:** Each $H_i$ is a "hybrid" mix of truly random bits and pseudorandom bits. As $i$ increases, we replace pseudorandom bits with truly random ones.

If we cannot distinguish neighbors, transitivity implies we cannot distinguish the start ($H_0$) from the end ($H_\ell$).

##### Theorem 3.1: PRG stretching
> [!abstract] Theorem 3.1
> Let $G : \{0, 1\}^n \to \{0, 1\}^{n+1}$ be a PRG (expands by 1 bit).
> Then, for any polynomial $\ell = poly(n)$, there exists a PRG $G_\ell : \{0, 1\}^n \to \{0, 1\}^{n+\ell}$.

**Proof:**
We construct the generator $G_\ell$ by iteratively applying $G$.
For each $i \in [0, \ell]$, define a function $f_i(s)$ that outputs a string $b_1 || \dots || b_\ell || s_\ell$:
##### Figure 3.1: Hybrid Construction

> [!example] Figure 3.1
> The idea behind the construction of the functions $f_0, \dots, f_\ell$.
> * **Red strings** represent truly random bits.
> 
>![[Pasted image 20260125171119.png]]


* **Algorithm for $f_i$:**
    1.  Input seed $s \in \{0, 1\}^n$.
    2.  Set $s_i = s$.
    3.  **Random Part ($j \le i$):** Set $b_j$ as truly random bits ($b_j \in_R U_1$).
    4.  **Pseudorandom Part ($j > i$):** Iteratively compute the next bits using $G$:
        $G(s_{j-1}) = s_j || b_j$, where $s_j \in \{0, 1\}^n$ is the next state and $b_j \in \{0, 1\}$ is the output bit.
    5.  Return $b_1 || \dots || b_\ell || s_\ell$.

**Distributions:**
Let $H_i$ be the distribution of the output of $f_i$.
* $H_\ell$: All bits are generated randomly (Step 3 covers all $j$). Thus $H_\ell = U_{n+\ell}$.
* $H_0$: No bits are random; all are generated by $G$. Thus $H_0 = G_\ell(U_n)$.

**Claim:** For each $i \in [0, \ell - 1]$, it holds that $H_i \approx_c H_{i+1}$.

**Proof of the Claim:**
We prove this by reduction. We will show that if an adversary can distinguish between the hybrids $H_i$ and $H_{i+1}$, they can use that ability to distinguish the output of the underlying PRG $G$ from a truly random string, which contradicts the security of $G$.

Fix an index $i \in [0, \ell - 1]$. By way of contradiction, suppose $H_i \not\approx_c H_{i+1}$. This means there exists a PPT distinguisher $D_i$ and a constant $c > 0$ such that:
$$
|\mathbb{P}[D_i(z) = 1 : z \in_R H_i] - \mathbb{P}[D_i(z) = 1 : z \in_R H_{i+1}]| > \frac{1}{n^c}
$$

We construct a new algorithm $D$ that breaks the PRG $G$ using $D_i$ as a subroutine.

**The Algorithm $D$:**
$D$ receives an input challenge string $w \in \{0, 1\}^{n+1}$. Its goal is to decide if $w$ was generated by $G(U_n)$ or if $w$ is truly random $U_{n+1}$.

1.  **Parse Input:**
    Let $w$ be parsed as $s || b$, where $s \in \{0, 1\}^n$ is the $n$-bit state part and $b \in \{0, 1\}$ is the 1-bit output part.
2.  **Generate Random Prefix:**
    Generate $i$ truly random bits: $b_1, \dots, b_i \in_R \{0, 1\}$.
    *(Note: These correspond to the "truly random" part shared by both hybrids up to index $i$.)*
3.  **Generate Pseudorandom Suffix:**
    Use the state $s$ from the input $w$ to continue the PRG expansion for the remaining steps ($j$ from $i+2$ to $\ell$).
    Run the generator starting from state $s$:
    * For $j = i+2$ to $\ell$, compute $(s_j, b_j) \leftarrow G(s_{j-1})$ (starting with $s_{i+1} = s$).
    * This produces the bits $b_{i+2}, \dots, b_\ell$ and the final state $s_\ell$.
4.  **Construct Hybrid String:**
    Assemble the full string $z'$ by stitching the parts together:
    * **Prefix:** $b_1 || \dots || b_i$ (Truly Random)
    * **Middle:** $b$ (The bit from the challenge $w$)
    * **Suffix:** $b_{i+2} || \dots || b_\ell || s_\ell$ (Generated from challenge state $s$)
    $$z' = b_1 || \dots || b_i || \mathbf{b} || b_{i+2} || \dots || b_\ell || s_\ell$$
5.  **Run Distinguisher:**
    Return the output of $D_i(z')$.

**Analysis:**
We analyze the distribution of the constructed string $z'$ in the two cases for $w$:

* **Case 1: $w$ is Pseudorandom ($w \in_R G(U_n)$)**
    Here, $w = G(s_{prev})$ for some random seed. This means $s$ is the correct next state and $b$ is the correct pseudorandom bit generated by $G$.
    Consequently, the bit $b$ (at position $i+1$) is pseudorandom, and the subsequent bits $b_{i+2} \dots$ are continuations of that same pseudorandom sequence.
    * Result: $z'$ is distributed exactly according to **$H_i$**.

* **Case 2: $w$ is Truly Random ($w \in_R U_{n+1}$)**
    Here, $b$ is a truly random bit, independent of $s$.
    Consequently, the bit $b$ (at position $i+1$) is truly random. The subsequent bits are generated from a random state $s$.
    * Result: $z'$ is distributed exactly according to **$H_{i+1}$**.

**Conclusion:**
The advantage of $D$ in distinguishing $G(U_n)$ from $U_{n+1}$ is exactly the advantage of $D_i$ in distinguishing $H_i$ from $H_{i+1}$.

$$
\begin{aligned}
&|\mathbb{P}[D(w) = 1 : w \in_R G(U_n)] - \mathbb{P}[D(w) = 1 : w \in_R U_{n+1}]| \\
&= |\mathbb{P}[D_i(z') = 1 : z' \in_R H_i] - \mathbb{P}[D_i(z') = 1 : z' \in_R H_{i+1}]| \\
&> \frac{1}{n^c}
\end{aligned}
$$

This contradicts the fact that $G$ is a secure PRG. Therefore, no such distinguisher $D_i$ can exist, and $H_i \approx_c H_{i+1}$.

By the transitivity of computational indistinguishability:
$$G_\ell(U_n) = H_0 \approx_c H_1 \approx_c \dots \approx_c H_\ell = U_{n+\ell}$$

This proves that $G_\ell$ is a valid PRG. $\blacksquare$

## 3.3 Hard-core Predicates and OWFs

We previously showed how to stretch a PRG from 1 bit to $poly(n)$ bits. The next logical step is constructing that initial 1-bit PRG.

**The Theoretical Equivalence:**
It is a fundamental result in cryptography that One-Way Functions (OWFs) and Pseudorandom Generators (PRGs) are equivalent.
$$\text{Existence of OWFs} \iff \text{Existence of PRGs}$$
This means building a PRG is exactly as hard as building a OWF. Since we don't know for sure if OWFs exist (we only assume they do), we rely on heuristic constructions in practice.

### From PRGs to OWFs
The "easier" direction is showing that if a PRG exists, it implies a OWF exists.

> [!abstract] Theorem 3.2: From PRGs to OWFs
> Let $G : \{0, 1\}^n \to \{0, 1\}^{2n}$ be a PRG. Then, there exists a OWF $f : \{0, 1\}^{2n} \to \{0, 1\}^{2n}$.

**Proof:**
Define $f$ as running $G$ on the first half of the input:
$$f(x_1 || \dots || x_{2n}) = G(x_1 || \dots || x_n)$$
We prove $f$ is a OWF by contradiction. Suppose $f$ is **not** one-way. Then there exists an adversary $\mathcal{A}$ that inverts $f$ with non-negligible probability $1/n^c$.

We construct a distinguisher $D$ to break the PRG $G$:
**Distinguisher $D(z)$:**
1.  Run $\mathcal{A}(z)$ to get a candidate preimage $b_1 || \dots || b_{2n}$.
2.  Check if $G(b_1 || \dots || b_n) = z$.
3.  If yes, output 1 (accept); otherwise output 0 (reject).

**Analysis:**
* If $z \in_R U_{2n}$ (Truly Random): The probability that a random string $z$ is in the image of $G$ is at most $2^n / 2^{2n} = 2^{-n}$. Thus, $D$ accepts with negligible probability.
* If $z \in_R G(U_n)$ (Pseudorandom): Since $z$ is a valid output of $G$, it has a valid preimage. By our assumption, $\mathcal{A}$ finds this preimage with probability $\ge 1/n^c$. Thus, $D$ accepts with non-negligible probability.

This distinguishes $G(U_n)$ from $U_{2n}$, contradicting the fact that $G$ is a PRG. $\blacksquare$

---

### Hard-Core Predicates

To prove the reverse (OWF $\implies$ PRG), we need the concept of **Hard-Core Bits**. These are specific bits of information about the input $x$ that are as hard to compute as inverting the function $f$ itself.

##### Definition 3.4: Hard-core predicate
> [!def] Definition 3.4
> Let $f : \{0, 1\}^n \to \{0, 1\}^m$ be a PPT function. A function $h : \{0, 1\}^n \to \{0, 1\}$ is a **hard-core predicate** for $f$ if for every PPT adversary $\mathcal{A}$:
> $$\mathbb{P}[\mathcal{A}(f(x)) = h(x) : x \in_R U_n] \leq \frac{1}{2} + negl(n)$$

**Equivalent Formulation:**
$h$ is hard-core for $f$ iff the pair $(f(x), h(x))$ is computationally indistinguishable from $(f(x), U_1)$.

### Three Fundamental Questions

1.  **Is there a universal hard-core predicate for *every* OWF?**
    **No.** For any candidate predicate $h^*$, we can construct a OWF $f'(x) = h^*(x) || f(x)$ where the predicate is explicitly revealed in the output, making it trivial to guess.

2.  **Does having a hard-core predicate imply the function is One-Way?**
    **No.** Consider $f(x_1 \dots x_n) = x_1 \dots x_{n-1}$ (drops the last bit). The predicate $h(x) = x_n$ is perfectly hidden (hard-core) because it is lost. However, $f$ is trivial to invert (just guess the last bit).

3.  **Does every OWF have a hard-core predicate?**
    **Yes (via modification).** The Goldreich-Levin theorem shows we can transform any OWF into one that has a specific hard-core predicate.

### The Goldreich-Levin Theorem
We can modify any OWF $f$ to create a new function $g$ that has a simple hard-core predicate based on the inner product.

> [!abstract] Theorem 3.3: Goldreich-Levin
> Let $f$ be a OWF. Define $g(x, r) = f(x) || r$, where $|x|=|r|=n$.
> Then, $g$ is a OWF with the hard-core predicate:
> $$h(x, r) = \oplus_{i=1}^n x_i r_i $$

---

### Constructing PRGs from OWFs

Ideally, we want to construct PRGs from One-Way Permutations (OWPs), which are bijective OWFs.

> [!check] Proposition 3.1: From OWP to PRG
> Let $f$ be a One-Way Permutation with a hard-core predicate $h$.
> Then, $G(s) = f(s) || h(s)$ is a PRG with expansion factor 1.

**Proof:**
1.  Since $f$ is a permutation, $f(U_n)$ is distributed identically to $U_n$.
2.  By the definition of a hard-core predicate, computing $h(s)$ given $f(s)$ is hard. Specifically, $(f(s), h(s)) \approx_c (f(s), U_1)$.
3.  Combining these: $G(U_n) = (f(U_n), h(U_n)) \equiv (U_n, h(U_n)) \approx_c (U_n, U_1) = U_{n+1}$. $\blacksquare$

This result can be generalized to all One-Way Functions (not just permutations), though the construction is more complex (Håstad, Impagliazzo, Levin, Luby '99).
##### From OWF to PRG

> [!abstract] Theorem 3.4: From OWF to PRG
> For every One-Way Function $f$, there exists a Pseudorandom Generator $G$ with expansion factor 1.

## 3.4 Solved Exercises

### Problem 3.1: PRG Security under XOR

**Question:**
For $m > n$, let $G : \{0, 1\}^n \to \{0, 1\}^m$ be a secure PRG.
Consider $G'$ defined as:
$$G'(s) = G(s) \oplus (0^{m-n} || s)$$
Prove or disprove that $G'$ is a secure PRG.

> [!fail] Verdict: Disprove
> $G'$ is not necessarily a secure PRG. We can construct a counter-example where the XOR operation reveals information (specifically, creates a constant bit).

**Counter-Example:**
Let $G^*$ be a secure PRG $G^* : \{0, 1\}^n \to \{0, 1\}^{m-1}$.
Construct $G(s)$ as:
$$G(s) = G^*(s_{1:n-1}) || s_n$$
*(Note: $s_{1:n-1}$ are the first $n-1$ bits, and $s_n$ is the last bit. This $G$ is valid if $G^*$ is valid).*

Now, analyze the output of $G'(s)$:
$$
\begin{aligned}
G'(s) &= G(s) \oplus (0^{m-n} || s) \\
&= (G^*(s_{1:n-1}) || s_n) \oplus (0^{m-n} || s_{1:n-1} || s_n) \\
\end{aligned}
$$
Focus on the **last bit** of the output:
* The last bit of $G(s)$ is $s_n$.
* The last bit of the padding term $(0^{m-n} || s)$ is also $s_n$.
* The last bit of $G'(s)$ is $s_n \oplus s_n = 0$.

**Adversary $\mathcal{A}$: **
1.  Check the last bit ($z_m$) of the input $z$.
2.  If $z_m = 0$, return 1 (likely pseudorandom).
3.  If $z_m = 1$, return 0 (must be random).

**Advantage:**
* $\mathbb{P}[\mathcal{A}(G'(U_n)) = 1] = 1$ (Last bit is always 0).
* $\mathbb{P}[\mathcal{A}(U_m) = 1] = 1/2$ (Last bit is 0 with probability 0.5).
* Difference $= 1 - 0.5 = 0.5$, which is non-negligible.

---

### Problem 3.2: OWF Combiner (Robustness)

**Question:**
Let $f_1, f_2, f_3 : \{0, 1\}^n \to \{0, 1\}^n$ be arbitrary functions. You know that **at least one** of them is a OWF, but you don't know which one.
Design a OWF $f$ using $f_1, f_2, f_3$ in a black-box manner.

> [!success] Solution
> We construct $f$ by concatenating the outputs of all three functions. If an adversary can invert the combination, they must be able to invert *every* component, including the one that is truly One-Way.

**Construction:**
Define $f : \{0, 1\}^{3n} \to \{0, 1\}^{3n}$ as:
$$f(x_1 || x_2 || x_3) = f_1(x_1) || f_2(x_2) || f_3(x_3)$$

**Proof (by Contradiction):**
Suppose $f$ is **not** a OWF. Then there exists an adversary $\mathcal{A}_f$ that inverts $f$ with non-negligible probability.
We can use $\mathcal{A}_f$ to build an adversary $\mathcal{A}_{f_i}$ that breaks *any* specific $f_i$.

**Reduction Algorithm $\mathcal{A}_{f_i}(y)$:**
1.  **Input:** A target value $y$ (where $y = f_i(x)$ for random $x$).
2.  **Setup:** Construct a target vector for the combined function $f$.
    * Set $y'_i = y$.
    * Set $y'_j = 0^n$ for the other indices $j \neq i$ (dummy values).
3.  **Call Oracle:** Run the adversary $\mathcal{A}_f$ on the input $Y = y'_1 || y'_2 || y'_3$.
4.  **Extract:** $\mathcal{A}_f$ returns a preimage $X = x'_1 || x'_2 || x'_3$ such that $f(X) = Y$.
5.  **Output:** Return $x'_i$.

**Analysis:**
Since $\mathcal{A}_f$ inverts $f$, it finds $x'_1, x'_2, x'_3$ such that $f_1(x'_1) = y'_1$, $f_2(x'_2) = y'_2$, and $f_3(x'_3) = y'_3$.
Specifically, it finds $x'_i$ such that $f_i(x'_i) = y$.
This means $\mathcal{A}_{f_i}$ successfully inverts $f_i$.

Since we know **at least one** function $f_k$ is actually a OWF, the existence of such a successful adversary $\mathcal{A}_{f_k}$ is a contradiction. Therefore, $f$ must be a OWF.