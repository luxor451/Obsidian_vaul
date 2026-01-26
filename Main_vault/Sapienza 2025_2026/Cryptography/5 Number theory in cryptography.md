**Tags:** #Cryptography #NumberTheory #ModularArithmetic #Groups #Rings #Fields #EuclideanAlgorithm #RSA #Primes #EulerTotient #DiscreteLogarithm #DiffieHellman 

---

## 5.1 Introduction

To build advanced cryptographic tools (like [[6 Public-key encryption|Public-Key Cryptography]]), we need specific **hard problems** (e.g., Factoring, Discrete Log). Number theory provides the algebraic structures and hardness assumptions required.

### Algebraic Structures
We work with structures like groups, rings, and fields.
* **Groups $(G, +)$:** A set $G$ with an operation $+$ that satisfies:
    * **Associativity:** $(a+b)+c = a+(b+c)$.
    * **Neutral Element:** $\exists 0 \in G$ such that $a+0 = a$.
    * **Inverse:** $\forall a \in G, \exists -a \in G$ such that $a + (-a) = 0$.
    * Example: $(\mathbb{Z}_n, +)$.
* **Rings $(G, +, \cdot)$:** An abelian group under $+$ with an associative multiplication $\cdot$ that has a neutral element ($1$). Distributivity holds.
    * Example: $(\mathbb{Z}_n, +, \cdot)$.
* **Fields:** A ring where every non-zero element has a multiplicative inverse.
    * Example: $(\mathbb{Z}_p, +, \cdot)$ where $p$ is prime.

**Modular Arithmetic:**
$a \equiv b \pmod n$ if $n$ divides $a-b$ (i.e., $a = b + kn$).

### Invertibility in $\mathbb{Z}_n$
Let $\mathbb{Z}^*_n$ be the set of **invertible elements** in $\mathbb{Z}_n$. An element $a$ is invertible iff it is coprime to $n$.

##### Proposition 5.1
> [!check] Proposition 5.1
> Given $a \in \mathbb{Z}_n$, it holds that $a \in \mathbb{Z}^*_n \iff \gcd(a, n) = 1$.

**Proof:**
* ($\Rightarrow$) If $a$ is invertible, $\exists b$ such that $ab \equiv 1 \pmod n$, so $ab - kn = 1$. The GCD $d$ must divide the linear combination $ab - kn$, so $d$ divides 1. Thus $d=1$.
* ($\Leftarrow$) If $\gcd(a, n) = 1$, by Bézout's identity $\exists k_1, k_2$ such that $k_1 a + k_2 n = 1$. Thus $k_1 a \equiv 1 \pmod n$, so $k_1$ is the inverse.

**Euler's Totient Function $\phi(n)$:**
* $\mathbb{Z}^*_n = \{a \in \mathbb{Z}_n \mid \gcd(a, n) = 1\}$.
* $|\mathbb{Z}^*_n| = \phi(n)$.
* For a prime $p$, $\mathbb{Z}^*_p = \{1, \dots, p-1\}$, so $\phi(p) = p-1$.

---

### The Euclidean Algorithm

To compute the GCD efficiently, we use the Euclidean Algorithm. It relies on the following lemma:

##### Lemma 5.1
> [!abstract] Lemma 5.1
> Given $a, b \in \mathbb{N}$ such that $a \ge b > 0$:
> $$\gcd(a, b) = \gcd(b, a \pmod b)$$

##### Algorithm 5.1: Euclidean Algorithm
> [!algo] Algorithm 5.1
> **Input:** $a, b \in \mathbb{Z}$ with $0 < a \le b$.
> **Output:** $\gcd(a, b)$.
> 1. Set $r_0 := b$ and $r_1 := a$. Set $i = 0$.
> 2. While $r_i \neq 0$:
>    * Compute $r_{i+1} := r_{i-1} \pmod{r_i}$.
>    * Increment $i$.
> 3. Return $r_{i-1}$.

**Complexity:** $O(|a| + |b|)$ bit operations (linear in the number of bits).

**Extended Euclidean Algorithm:**
We can work backwards to find coefficients $k, h$ such that $\gcd(a, b) = ak + bh$ (Bézout's identity).
If $\gcd(a, n) = 1$, finding $ak + hn = 1$ implies $ak \equiv 1 \pmod n$, giving us the **modular inverse** $a^{-1} = k$.

---

### Modular Exponentiation

We frequently need to compute $g^x \pmod n$ for very large $x$.
Using the **Square-and-Multiply** algorithm (based on the binary representation of $x$), we can compute this in **polynomial time** (specifically $O(\log x)$ multiplications).

$$a^b \equiv a^{\sum b_i 2^i} \equiv \mathbb{P}od (a^{2^i})^{b_i} \pmod n$$

---

### Generating Prime Numbers

Cryptosystems like [[6 Public-key encryption#6.2 RSA Encryption and Trapdoor Permutations|RSA]] require large prime numbers.
**Prime Number Theorem:** The number of primes up to $x$, denoted $\pi(x)$, is approx $x / \ln x$.
* A random $\lambda$-bit number is prime with probability $\approx 1/\lambda$.
* We can generate primes by repeatedly sampling random numbers and testing them.

**Primality Testing:**
We use probabilistic tests based on number-theoretic properties.

**Fermat's Little Theorem:**
> If $p$ is prime, then for all $a \in \{1, \dots, p-1\}$:
> $$a^{p-1} \equiv 1 \pmod p$$

**Euler's Theorem (Generalization):**
> For any $n$, if $a \in \mathbb{Z}^*_n$:
> $$a^{\phi(n)} \equiv 1 \pmod n$$
> $$a^b \equiv a^{b \pmod{\phi(n)}} \pmod n$$

**Algorithms:**
1.  **Fermat Test:** Check if $a^{n-1} \equiv 1 \pmod n$.
    * *Problem:* **Carmichael Numbers** (composite numbers that pass for all $a$).
2.  **Miller-Rabin:** Efficient probabilistic test. Guaranteed to eventually terminate and correct with high probability.
3.  **AKS:** Deterministic polynomial-time test (slower than Miller-Rabin).

**Conclusion:** We can efficiently generate large primes for cryptography.

## 5.2 The DL, CDH, and DDH Assumptions

We formally introduce the number-theoretic hardness assumptions that underpin public-key cryptography.

### The Discrete Logarithm (DL) Problem
Consider the equation $a^x = b$.
* **Continuous Setting ($\mathbb{R}$):** Solved easily via logarithms: $x = \log_a b$.
* **Discrete Setting ($\mathbb{Z}_p$):** Consider $g^x \equiv y \pmod p$. Due to the cyclic nature of the group, there are infinite solutions for the exponent (modulo the group order). Finding the specific $x$ used is computationally difficult.

**Example:** $2^x \equiv 4 \pmod 5$.
$4 \equiv 2^2 \equiv 2^6 \equiv 2^{10} \dots$
Solving for $x$ involves reversing the modular exponentiation, which is believed to be hard (sub-exponential time best known).

##### Assumption 5.1: DL Assumption
> [!def] Assumption 5.1
> Given a prime $p$ and a generator $g$, the **Discrete Logarithm assumption** states that for all PPT adversaries $\mathcal{A}$:
> $$\mathbb{P}[\mathcal{A}(g, p, y) = x : x \in_R \mathbb{Z}_p, y \equiv g^x \pmod p] \leq negl(\lambda)$$

This implies the existence of a One-Way Function (OWF): $f(x) = g^x \pmod p$.

---

### Diffie-Hellman Key Exchange

Diffie and Hellman used the DL assumption to propose a revolutionary protocol allowing two parties to establish a shared secret over a public channel.

##### Algorithm 5.2: Diffie-Hellman Key Exchange
> [!algo] Algorithm 5.2
> **Goal:** Alice and Bob generate a shared key $K$.
> 1.  **Setup:** Alice picks a prime $p$ and a generator $g$ of $\mathbb{Z}^*_p$. She shares $(g, p)$ with Bob.
> 2.  **Alice:** Samples secret $x \in_R \mathbb{Z}_{p-1}$. Computes $X = g^x \pmod p$. Sends $X$ to Bob.
> 3.  **Bob:** Samples secret $y \in_R \mathbb{Z}_{p-1}$. Computes $Y = g^y \pmod p$. Sends $Y$ to Alice.
> 4.  **Key Derivation:**
>     * Alice computes $K = Y^x = (g^y)^x = g^{xy} \pmod p$.
>     * Bob computes $K = X^y = (g^x)^y = g^{xy} \pmod p$.


---

### Hardness Assumptions for Diffie-Hellman

Security against **Passive Attacks** (eavesdropping) relies on the hardness of computing $g^{xy}$ given only $g^x$ and $g^y$.

#### Computational Diffie-Hellman (CDH)
Is the DL assumption enough? Not necessarily. Even if an attacker cannot find $x$ (DL problem), they might still be able to compute the shared key $g^{xy}$. We need a specific assumption for this.

##### Assumption 5.2: CDH Assumption
> [!def] Assumption 5.2
> Given a cyclic group $G$ of order $q$ and generator $g$, the **CDH assumption** states:
> $$\mathbb{P}[\mathcal{A}(G, g, q, g^x, g^y) = g^{xy} : x, y \in_R \mathbb{Z}_q] \leq negl(\lambda)$$

* **Relation:** $DL \implies CDH$ (If you can solve DL, you can find $x$ and compute $(g^y)^x = g^{xy}$).
* **Converse:** Unknown (but widely believed false).

#### Decisional Diffie-Hellman (DDH)
CDH ensures the adversary cannot *compute* the key. However, it does not guarantee the key looks *random*. An adversary might distinguish $K$ from a random string, leaking information.
We require the shared secret to be indistinguishable from a random element in $G$.

##### Assumption 5.3: DDH Assumption
> [!def] Assumption 5.3
> Given $(G, g, q)$, the **DDH assumption** states that the following distributions are computationally indistinguishable:
> $$(g^x, g^y, g^{xy}) \approx_c (g^x, g^y, g^z)$$
> where $x, y, z \in_R \mathbb{Z}_q$.

* **Relation:** $CDH \implies DDH$ (If you can distinguish, you might be able to compute, or vice versa? Actually, if you can compute $g^{xy}$, you can distinguish it from $g^z$. So $DDH \implies CDH$).

---

### The Failure of DDH in $\mathbb{Z}^*_p$

Ideally, we would use $\mathbb{Z}^*_p$. However, **DDH does NOT hold in $\mathbb{Z}^*_p$.**

##### Proposition 5.2
> [!fail] Proposition 5.2
> **The DDH assumption fails for $G = \mathbb{Z}^*_p$.**

**Proof:**
This relies on **Quadratic Residues**.
Let $QR_p = \{y \in \mathbb{Z}^*_p \mid \exists x, x^2 \equiv y \pmod p\}$.
This is the set of elements that are perfect squares modulo $p$.
* **Property:** $y \in QR_p \iff y^{\frac{p-1}{2}} \equiv 1 \pmod p$ (Euler's Criterion).
* **Observation:** In $\mathbb{Z}^*_p$, exactly half the elements are in $QR_p$.

**Distinguisher $\mathcal{A}$:**
1.  Receive tuple $(X, Y, Z) = (g^x, g^y, Z)$.
2.  Check if $Z \in QR_p$ by computing $Z^{\frac{p-1}{2}} \pmod p$.
3.  **Analysis:**
    * If $Z = g^{xy}$: $Z \in QR_p$ if *either* $x$ or $y$ is even (probability $3/4$).
    * If $Z = g^z$ (Random): $Z \in QR_p$ with probability $1/2$.
4.  **Advantage:** $|3/4 - 1/2| = 1/4$, which is non-negligible.

**The Fix:**
To assume DDH holds, we must work in a subgroup where these attacks don't work.
1.  **Safe Primes:** Use a subgroup of prime order $q$ inside $\mathbb{Z}^*_p$ where $p = 2q+1$.
2.  **Elliptic Curves:** Use Elliptic Curve groups, which do not suffer from this specific index calculus structure.

---

### Active Security (Man-in-the-Middle)

The assumptions above only protect against passive eavesdropping.
**Man-in-the-Middle (MitM) Attack:**
An active attacker (Eve) intercepts $X$ from Alice and replaces it with $X'$. She intercepts $Y$ from Bob and replaces it with $Y'$.
* Alice computes key $K_{AE}$.
* Bob computes key $K_{EB}$.
* Eve knows both and proxies the traffic.

**Solution:** Hardness assumptions alone cannot fix this. We need **Authentication** (e.g., using a Master Key, MACs, or Digital Signatures) to verify the origin of $X$ and $Y$.

## 5.3 Building Crypto-Tools through Number Theory

We previously established the chain of implications: $DDH \to CDH \to DL \to OWF \to PRG \to PRF \to PRP \to CRH$.
However, we don't need to go through the full generic chain (which is often inefficient). We can construct primitives directly from number-theoretic assumptions.

### 1. Constructing a PRG (from DDH)
Consider a group $G$ of prime order $q$ where the **DDH assumption** holds.
Define the function $G_{G,g,q} : \mathbb{Z}_q \to G^3$ as:
$$G_{G,g,q}(x, y) = (g^x, g^y, g^{xy})$$
* **Input:** Random $x, y \in \mathbb{Z}_q$.
* **Output:** Tuple in $G^3$.
* **Security:** Under DDH, $(g^x, g^y, g^{xy}) \approx_c (g^x, g^y, g^z) \approx_c U_{G^3}$.
* **Stretch:** If elements of $G$ can be encoded efficiently, this expands randomness.

**Improved Construction (Stretch $t$):**
$$G^t_{G,g,q}(x, y_1, \dots, y_t) = (g^x, g^{y_1}, g^{xy_1}, g^{y_2}, g^{xy_2}, \dots, g^{y_t}, g^{xy_t})$$

### 2. Constructing a PRF (from DDH)
We can build a PRF efficiently using a closed-form construction similar to the GGM tree logic but without recursion.
Define the family $\mathcal{F} = \{F_{\vec{a}} : \{0, 1\}^n \to G\}_{\vec{a} \in \mathbb{Z}_q^{n+1}}$ where:
$$F_{\vec{a}}(x_1, \dots, x_n) = g^{a_0 \cdot \prod_{i=1}^n a_i^{x_i}}$$
*(Note: The source text describes a slightly different product structure $g^{a_0 \prod a_i^{x_i}}$ or similar exponentiation logic resembling the Naor-Reingold PRF).*

### 3. Constructing a CRH (from DL)
We can build a **Collision-Resistant Hash** family directly from the **Discrete Log (DL)** assumption.
Define $H_{g_1, g_2} : \mathbb{Z}_q \times \mathbb{Z}_q \to G$ as:
$$H_{g_1, g_2}(x_1, x_2) = g_1^{x_1} g_2^{x_2}$$
where $g_1, g_2 \in G$.

**Security Proof (Reduction):**
Suppose an adversary finds a collision: $(x_1, x_2) \neq (x'_1, x'_2)$ such that $H(x_1, x_2) = H(x'_1, x'_2)$.
$$g_1^{x_1} g_2^{x_2} = g_1^{x'_1} g_2^{x'_2} \iff g_1^{x_1 - x'_1} = g_2^{x'_2 - x_2}$$
If we know the discrete log relation between $g_1$ and $g_2$ (e.g., $g_2 = g_1^y$), we can solve for $y$:
$$g_1^{x_1 - x'_1} = (g_1^y)^{x'_2 - x_2} \implies x_1 - x'_1 \equiv y(x'_2 - x_2) \pmod q$$
$$y \equiv (x_1 - x'_1)(x'_2 - x_2)^{-1} \pmod q$$
Thus, finding a collision allows us to solve the Discrete Log problem.

---

## 5.4 Solved Exercises

### Problem 5.1: Generalized Number-Theoretic Hash
**Question:**
Let $G$ be a cyclic group of prime order $q$. Let $n$ be polynomial.
Consider the hash family $\mathcal{H}_{G,n}$ parametrized by $g_1, \dots, g_n \in G$:
$$H_{g_1, \dots, g_n}(x_1, \dots, x_n) = \prod_{i=1}^n g_i^{x_i}$$
Prove that $\mathcal{H}_{G,n}$ is collision-resistant assuming the **DL assumption** for $G$.

##### Proof
**Verdict:** The scheme is secure.

**Reduction Strategy:**
We construct an adversary $\mathcal{A}_{DL}$ that uses a collision-finder $\mathcal{A}_{CRH}$ to solve the Discrete Log problem.

1.  **Setup:**
    * $\mathcal{A}_{DL}$ receives a DL challenge $(g, p, y)$ where $y = g^\alpha$. Goal: Find $\alpha$.
    * $\mathcal{A}_{DL}$ chooses a random index $i \in_R [n]$.
    * Set $g_i = y$.
    * For all $j \neq i$, sample known exponents $r_j \in_R \mathbb{Z}_q$ and set $g_j = g^{r_j}$.
    * Send parameters $(g_1, \dots, g_n)$ to $\mathcal{A}_{CRH}$.

2.  **Attack:**
    * $\mathcal{A}_{CRH}$ returns a collision $(x_1, \dots, x_n)$ and $(x'_1, \dots, x'_n)$.
    * Condition: $\prod g_k^{x_k} = \prod g_k^{x'_k}$ and inputs differ.

3.  **Solving for DL:**
    Rearrange the collision equation:
    $$g_i^{x_i} \cdot \prod_{j \neq i} g_j^{x_j} = g_i^{x'_i} \cdot \prod_{j \neq i} g_j^{x'_j}$$
    $$g_i^{x_i - x'_i} = \prod_{j \neq i} g_j^{x'_j - x_j}$$
    Substitute known values ($g_i = y = g^\alpha$ and $g_j = g^{r_j}$):
    $$(g^\alpha)^{x_i - x'_i} = \prod_{j \neq i} (g^{r_j})^{x'_j - x_j}$$
    $$g^{\alpha(x_i - x'_i)} = g^{\sum_{j \neq i} r_j(x'_j - x_j)}$$
    In exponents modulo $q$:
    $$\alpha(x_i - x'_i) \equiv \sum_{j \neq i} r_j(x'_j - x_j) \pmod q$$

4.  **Success Condition:**
    If $x_i - x'_i \not\equiv 0 \pmod q$, we can invert it to find $\alpha$:
    $$\alpha = (x_i - x'_i)^{-1} \cdot \sum_{j \neq i} r_j(x'_j - x_j) \pmod q$$
    Since the collision guarantees the vectors differ at *some* position, and $\mathcal{A}_{DL}$ chose $i$ uniformly at random, the probability that the difference occurs at index $i$ is at least $1/n$.

**Conclusion:**
If $\mathcal{A}_{CRH}$ succeeds with non-negligible probability $\epsilon$, $\mathcal{A}_{DL}$ succeeds with probability $\epsilon/n$, which contradicts the DL assumption. $\blacksquare$