**Tags:** #Cryptography #SymmetricKeyEncryption #SKE #StreamCiphers #BlockCiphers #FeistelNetworks #DES #AES #CPA #CCA #PRF #HMAC #RandomOracleModel 

---

## 4.1 Games as security proofs

In Chapter 2 we gave the initial definitions of secure cryptographic schemes, without caring about the computation being achieved efficiently. In Chapter 3, instead, we discussed the basics of secure efficient computation. Now, we’re ready to merge the two ideas.

We start by discussing secure SKEs. Using the concept of [[3 Computational security#Definition 3.3: Pseudorandom generator (PRG)|pseudorandom generator (PRG)]] that we introduced in the previous chapter, we can define a PRG One Time Pad (PRG-OTP). Given $\mathcal{K} = \{0, 1\}^\lambda$ and $\mathcal{M} = \mathcal{C} = \{0, 1\}^{\lambda+\ell}$, we define:

$$
\begin{aligned}
Enc(K, m) &= G(K) \oplus m \\
Dec(K, c) &= G(K) \oplus c
\end{aligned}
$$

where $G : \{0, 1\}^\lambda \to \{0, 1\}^{\lambda+\ell}$ is a PRG and $\ell = poly(\lambda)$.

The use of a PRG allows us to "bypass" Shannon’s key-length constraint since the "pseudorandom key" generated through $G(K)$ is still at least as long as the message. However, this implies that our SKE doesn’t have real perfect secrecy, otherwise the theorem would be false. This idea moves us into defining a new concept of computationally secure SKE.

### Games and Adversaries
In this and many other cases, security definitions will be given in terms of games between two entities: a challenger and an adversary.
* **The Challenger:** Objective is proving that the scheme is secure by answering the questions posed by the adversary.
* **The Adversary:** Objective is to break the scheme.

The challenger wins if it is capable of responding to any possible series of questions without making the adversary able to break the scheme. If the challenger has a strategy to win the game, we consider the scheme to be secure. We assume that the two players are allowed only efficient computation.

Consider the following example. Let $Game^{1-time}_{\Pi, \mathcal{A}}(\lambda, b)$ be the 2-player game defined as follows:
1.  The adversary $\mathcal{A}$ sends two messages $m_0, m_1 \in \mathcal{M}$ to the challenger $\mathcal{C}$.
2.  $\mathcal{C}$ generates a key $K \in \mathcal{K}$ and sends $Enc_\Pi(K, m_b) = c$ back to $\mathcal{A}$.
3.  $\mathcal{A}$ analyzes $c$ and tries to guess if it was obtained through $m_0$ or $m_1$ by sending a bit $b' \in \{0, 1\}$.
4.  If $b' \neq b$, the challenger wins. Otherwise, the adversary wins.

> ![[Pasted image 20260125174801.png]]
> *Figure 4.1: Graphical representation of $Game^{1-time}_{\Pi, \mathcal{A}}(\lambda, b)$*

By the very definition of the game, it’s easy to see that if, for every pair of messages, the adversary is not capable of distinguishing which message got encrypted by the challenger, i.e. if they are playing $Game^{1-time}_{\Pi, \mathcal{A}}(\lambda, 0)$ or $Game^{1-time}_{\Pi, \mathcal{A}}(\lambda, 1)$, then the scheme can be considered 1-time computationally secure SKE.

##### Definition 4.1: 1-time computational security
> [!def] Definition 4.1
> Let $\Pi = (Enc, Dec)$ be a SKE. We say that $\Pi$ is **1-time computationally secure** if:
> $$Game^{1-time}_{\Pi, \mathcal{A}}(\lambda, 0) \approx_c Game^{1-time}_{\Pi, \mathcal{A}}(\lambda, 1)$$
> for all PPT adversaries $\mathcal{A}$.

To see how this game can be used for specific cryptosystems, we prove that PRG-OTP has 1-time computationally security.

##### Proposition 4.1
> [!check] Proposition 4.1
> **PRG-OTP has 1-time computational security.**

**Proof.**
Let $\ell = poly(n)$ be the expansion factor of the PRG $G$ used inside PRG-OTP. We use an argument similar to the hybrid argument. Let $Hyb(\lambda, b)$ be the 2-player game defined as follows:
1.  The adversary $\mathcal{A}$ sends two messages $m_0, m_1 \in \{0, 1\}^\lambda$ to the challenger $\mathcal{C}$.
2.  $\mathcal{C}$ picks $z \in_R U_{\lambda+\ell}$ and sends $z \oplus m_b = c$ back to $\mathcal{A}$.
3.  $\mathcal{A}$ analyzes $c$ and tries to guess if it was obtained through $m_0$ or $m_1$ by sending a bit $b' \in \{0, 1\}$.
4.  If $b' \neq b$, the challenger wins. Otherwise, the adversary wins.

We observe that the difference between $Game^{1-time}_{\Pi, \mathcal{A}}(\lambda, b)$ and $Hyb(\lambda, b)$ is given by $G(K)$ being replaced by a uniform at random string $z \in_R U_{\lambda+\ell}$. Since $G(U_\lambda) \approx_c U_{\lambda+\ell}$ by choice of $G$, the hybrid game and the original game are expected to also be indistinguishable from each other.

**Claim:** For each $b \in \{0, 1\}$ it holds that $Game^{1-time}_{\Pi, \mathcal{A}}(\lambda, b) \approx_c Hyb(\lambda, b)$.

**Proof of the claim.** By way of contradiction, assume that there is an adversary $\mathcal{A}'$ such that:
$$\mathbb{P} [\mathcal{A}'(b') = 1 : b' \leftarrow Game^{1-time}_{\Pi, \mathcal{A}}(\lambda, b)] - \mathbb{P} [\mathcal{A}'(b') = 1 : b' \leftarrow Hyb(\lambda, b)] \geq \frac{1}{\lambda^c}$$
for some $c > 0$. We build an adversary $D$ defined as follows:

**$D$ = "On input $z \in \{0, 1\}^{\lambda+\ell}$:**
1.  $D$ challenges $\mathcal{A}'$ to play the 1-time game (thus $D$ is the challenger and $\mathcal{A}'$ is the adversary). $D$ will use $z$ to compute $m_b \oplus z = c$ instead of choosing a random string.
2.  $D$ accepts if $\mathcal{A}'$ wins, otherwise $D$ rejects."

Since $\mathcal{A}'$ can distinguish the two games, it is also able to distinguish if the string $c$ passed by $D$ was computed with $z$ taken from $G(U_\lambda)$ or $U_{\lambda+\ell}$. This implies that:
$$\mathbb{P}[D(z) = 1 : z \in_R G(U_\lambda)] = \mathbb{P} [\mathcal{A}'(b') = 1 : b' \leftarrow Game^{1-time}_{\Pi, \mathcal{A}'}(\lambda, b)]$$
and:
$$\mathbb{P}[D(z) = 1 : z \in_R U_{\lambda+\ell}] = \mathbb{P} [\mathcal{A}'(b') = 1 : b' \leftarrow Hyb(\lambda, b)]$$
concluding that:
$$
\begin{aligned}
| \mathbb{P}[D(z) = 1 : z \in_R G(U_\lambda)] - \mathbb{P}[D(z) = 1 : z \in_R U_{\lambda+\ell}] | \\
= \mathbb{P} [\mathcal{A}'(b') = 1 : b' \leftarrow Game^{1-time}_{\Pi, \mathcal{A}'}(\lambda, b)] - \mathbb{P} [\mathcal{A}'(b') = 1 : b' \leftarrow Hyb(\lambda, b)] \\
\geq \frac{1}{\lambda^c}
\end{aligned}
$$
and thus contradicting the fact that $G$ is a PRG.

By definition, it’s easy to see that $Hyb(\lambda, 0) \equiv Hyb(\lambda, 1)$ since the distribution of $c$ is uniform and independent of $b$. Therefore, we conclude that:
$$Game^{1-time}_{\Pi, \mathcal{A}'}(\lambda, 0) \approx_c Hyb(\lambda, 0) \equiv Hyb(\lambda, 1) \approx_c Game^{1-time}_{\Pi, \mathcal{A}'}(\lambda, 1)$$


## 4.2 CPA-security and Pseudorandom Functions

In the previous section, we proved that PRG-OTP is 1-time computationally secure, meaning that it is secure against attackers that know only one message. However, this SKE’s weakness is ciphertext attacks.

**The Weakness (Known-Plaintext Attack):**
Suppose that an attacker knows a ciphertext-message pair $(m_1, c_1)$. If the challenger sends another ciphertext $c_2$ obtained from a message $m_2$, the attacker is able to retrieve the contents of $m_2$ through the SKE itself:

$$
\begin{aligned}
c_1 \oplus c_2 \oplus m_1 &= (G(K) \oplus m_1) \oplus (G(K) \oplus m_2) \oplus m_1 \\
&= (G(K) \oplus G(K)) \oplus (m_1 \oplus m_1) \oplus m_2 \\
&= 0 \oplus 0 \oplus m_2 = m_2
\end{aligned}
$$

The generalization of this type of attack is known as **Known-Plaintext Attack (KPA)** and the security counterpart is known as **t-time computational security**. We observe that KPAs require that the attacker only knows the pairs $(m_1, c_1), \dots, (m_t, c_t)$ but is not able to choose them.

**Chosen-Plaintext Attack (CPA):**
When the attacker is able to choose the pairs (e.g. they can choose messages and get their ciphertexts) we talk about **Chosen-Plaintext Attack (CPA)**.

In the CPA game, the adversary is given access to an **encryption oracle**, i.e., a sub-procedure with unlimited power that can be queried in order to get an answer. Each query costs $\Theta(1)$ computational steps. We observe that the oracle queries made by the attacker can be seen as "questions already answered".

Let $Game^{CPA}_{\Pi, \mathcal{A}}(\lambda, b)$ be the 2-player game defined as follows:

1.  The challenger $\mathcal{C}$ generates a key $K \in \mathcal{K}$.
2.  The adversary $\mathcal{A}$ queries $m_1, \dots, m_t \in \{0, 1\}^\lambda$, where $t = poly(\lambda)$, to an oracle for $Enc_\Pi(K, \cdot)$. The oracle answers with $c_1, \dots, c_t$.
3.  The adversary sends two messages $m^*_0, m^*_1 \in \{0, 1\}^\lambda$ to the challenger $\mathcal{C}$.
4.  $\mathcal{C}$ sends $Enc_\Pi(K, m^*_b) = c^*$ back to $\mathcal{A}$.
5.  $\mathcal{A}$ queries $m'_1, \dots, m'_s \in \{0, 1\}^\lambda$, where $s = poly(\lambda)$, to an oracle for $Enc_\Pi(K, \cdot)$. The oracle answers with $c'_1, \dots, c'_s$.
6.  $\mathcal{A}$ tries to guess if it was obtained through $m_0$ or $m_1$ by sending a bit $b' \in \{0, 1\}$.
7.  If $b' \neq b$, the challenger wins. Otherwise, the adversary wins.

> ![[Pasted image 20260125180159.png]]
> *Figure 4.2: Graphical representation of $Game^{CPA}_{\Pi, \mathcal{A}}(\lambda, b)$. Queries to the oracle can be substituted with queries to C, as if the two players already exchanged some pairs.*

##### Definition 4.2: CPA-security
> [!def] Definition 4.2
> Let $\Pi = (Enc, Dec)$ be an SKE. We say that $\Pi$ is **CPA-secure** if:
> $$Game^{CPA}_{\Pi, \mathcal{A}}(\lambda, 0) \approx_c Game^{CPA}_{\Pi, \mathcal{A}}(\lambda, 1)$$
> for all PPT adversaries $\mathcal{A}$.

### Why Deterministic Encryption Fails
It’s easy to see that the PRG-OTP is not CPA-secure since the output of the encryption scheme is deterministic (meaning that it will always return the same output when given the same input).

**The Attack:**
Consider the attacker that queries two messages $m_0, m_1 \in \{0, 1\}^n$. After obtaining the ciphertexts $c_0, c_1$ from $\mathcal{C}$, the attacker sends the final queries $m^*_0 = m_0$ and $m^*_1 = m_1$, obtaining the final ciphertext $c^*$. Since the scheme is deterministic, we know that either $c^* = c_0$ or $c^* = c_1$ must hold (depending on the value $b$ used by $\mathcal{C}$). Thus, the attacker returns 0 if $c^* = c_0$, otherwise they return 1. This allows the attacker to distinguish the two games with probability 1.

Clearly, this idea is not restricted to the PRG-OTP: it holds for any deterministic SKE!

##### Proposition 4.2
> [!fail] Proposition 4.2
> **No deterministic SKE can be CPA-secure.**

---

### Pseudorandom Functions (PRF)

In order to make a CPA-secure SKE, we need to substitute pseudorandom generators with **pseudorandom functions**, i.e., functions that are indistinguishable from a randomly selected one (thus they make their internal workings incomprehensible).

Let $\mathcal{R}(s \to t)$ denote the uniform distribution over the set of all functions from strings of $s$ bits to strings of $t$ bits. We define the following 2-player game $Game^{PRF}_{\mathcal{F}, \mathcal{A}}(\lambda, b)$:

1.  The challenger $\mathcal{C}$ generates a key $K \in \{0, 1\}^\lambda$ and sets $J = F_K$ if $b = 0$, otherwise $J \in_R \mathcal{R}(n \to n)$.
2.  The adversary $\mathcal{A}$ queries $x_1, \dots, x_t \in \{0, 1\}^n$ where $t = poly(n)$, to an oracle for $J(\cdot)$. The oracle answers with $y_1, \dots, y_t$.
3.  $\mathcal{A}$ tries to guess $b$ by sending a bit $b' \in \{0, 1\}$.
4.  If $b \neq b'$, the challenger wins. Otherwise, the adversary wins.

> ![[Pasted image 20260125180805.png]]
> *Figure 4.3: Graphical representation of $Game^{PRF}_{\mathcal{F}, \mathcal{A}}(\lambda, b)$.*

##### Definition 4.3: Pseudorandom functions
> [!def] Definition 4.3
> Let $\mathcal{F} = \{F_K : \{0, 1\}^n \to \{0, 1\}^m\}_{K \in \{0,1\}^\lambda}$ be a family of poly-time functions. We say that $\mathcal{F}$ is a **pseudorandom function family (PRF)** if:
> $$Game^{PRF}_{\mathcal{F}, \mathcal{A}}(\lambda, 0) \approx_c Game^{PRF}_{\mathcal{F}, \mathcal{A}}(\lambda, 1)$$
> for all PPT adversaries $\mathcal{A}$.

We observe that the above definition requires that $\mathcal{F}$ is a family of efficient deterministic functions: randomness is only used to choose a key that identifies which function of the family must be used. The uniform distribution $\mathcal{R}(n \to n)$, instead, also contains functions that aren’t poly-time computable. The idea behind the PRF game is to establish that each function of the family is computationally indistinguishable from a randomly chosen function.

In practical applications, many common encryption schemes (e.g. DES, 3DES, AES, ...) are based on on **pseudorandom permutations (PRP)**, i.e. PRFs that are invertible if the key is known. For now, we’ll give the idea behind how PRFs can be used in theory to produce CPA-secure SKEs.

**Using PRFs for CPA Security:**
First of all, we observe that we cannot simply swap PRGs with PRFs in order to get a good scheme since each function is deterministic. The trick here is to use a random initial value called **nonce** (short for "number used once"). As their name suggests, these values are used exactly once by the cryptosystem, granting that the same input will always give a different ciphertext.

Given a PRF $\mathcal{F} = \{F_K : \{0, 1\}^n \to \{0, 1\}^n\}_{K \in \{0,1\}^\lambda}$, let the **PRF One Time Pad (PRF-OTP)** be the scheme defined by the following operations:

1.  $Enc(K, m) = (r, F_K(r) \oplus m)$, where $r \in \{0, 1\}^n$ is a nonce.
2.  $Dec(K, (r, c)) = F_K(r) \oplus c$.

We observe that the encryption operation must also return the used nonce in order for the decryption operation to work. However, this is not an issue for CPA-security: since the nonce will never be used again by the scheme, the adversary gains no information on the key by knowing it.

##### Proposition 4.3
> [!check] Proposition 4.3
> **PRF-OTP is CPA-secure.**

**Proof:** Omitted (a hybrid argument suffices).

---

### 4.2.1 The GGM Tree

After discussing how PRFs can be used to achieve CPA-security, we prove that PRGs can be used to construct PRFs. In particular, we’ll give an explicit construction known as the **Goldreich-Goldwasser-Micali Tree (GGM Tree)**.

##### Definition 4.4: GGM Tree
> [!def] Definition 4.4
> Given a PRG $G : \{0, 1\}^\lambda \to \{0, 1\}^{2\lambda}$, let $G_0, G_1 : \{0, 1\}^\lambda \to \{0, 1\}^\lambda$ be two functions that respectively return the first and last $\lambda$ bits of the output of $G$, meaning that $G(k) = G_0(k) \,||\, G_1(k)$ for all $k \in \{0, 1\}^\lambda$.
> 
> The **GGM Tree function family** is defined as $\mathcal{F} = \{F_K : \{0, 1\}^n \to \{0, 1\}^\lambda\}_{K \in \{0,1\}^\lambda}$, where:
> $$F_K(x_1 || \dots || x_n) = G_{x_n}(G_{x_{n-1}}(\dots G_{x_2}(G_{x_1}(K))))$$

> ![[Pasted image 20260125181313.png]]
> *Figure 4.4: The GGM Tree for inputs of length $n = 3$. The path leading to the leaf represents the computation $F_K(011) = G_1(G_1(G_0(K)))$.*

To prove the security of this construction, we first need a lemma regarding the output of a PRG on multiple keys.

##### Lemma 4.1
> [!abstract] Lemma 4.1
> Let $G : \{0, 1\}^\lambda \to \{0, 1\}^{2\lambda}$ be a PRG. Given $K_1, \dots, K_t \in_R U_\lambda$, with $t = poly(\lambda)$ and $K_i \neq K_j$ for all $i \neq j$, it holds that:
> $$(G(K_1), \dots, G(K_t)) \approx_c (U_{2\lambda}, \dots, U_{2\lambda})$$

**Proof:**
We define $t$ hybrid distributions $H_0, \dots, H_t$ such that for each $i \in [t]$ it holds that:
$$H_i = (G(K_1), \dots, G(K_{t-i}), \underbrace{U_{2\lambda}, \dots, U_{2\lambda}}_{i \text{ times}})$$
We observe that $H_0 \equiv (G(K_1), \dots, G(K_t))$ and $H_t \equiv (U_{2\lambda}, \dots, U_{2\lambda})$.

**Claim:** For each $i \in [0, t - 1]$ it holds that $H_i \approx_c H_{i+1}$.

**Proof of the claim:**
Fix $i \in [0, t-1]$. We prove the claim by reducing the distinguishability of $G(K_{t-i})$ from $U_{2\lambda}$ to the distinguishability of $H_i$ from $H_{i+1}$. By way of contradiction, suppose that $H_i \not\approx_c H_{i+1}$, meaning that there is a distinguisher $D_i$ such that:
$$|\mathbb{P}[D_i(z) = 1 : z \in_R H_i] - \mathbb{P}[D_i(z) = 1 : z \in_R H_{i+1}]| > \frac{1}{n^c}$$
for some $c > 0$. Let $D$ be the algorithm defined as follows:

**$D$ = "On input $z \in \{0, 1\}^{2\lambda}$:**
1.  Extract $t - i$ keys $K_1, \dots, K_{t-i} \in_R U_\lambda$.
2.  Extract $i - 1$ strings $w_1, \dots, w_{i-1} \in_R U_{2\lambda}$.
3.  Return $D_i(G(K_1) \,||\, \dots \,||\, G(K_{t-i}) \,||\, z \,||\, w_1 \,||\, \dots \,||\, w_{i-1})$"

We observe that $H_i$ and $H_{i+1}$ differ only on the $i$-th string of $2n$ bits: the former contains an output of $G$ while the latter contains a truly random string. This implies that:
$$
\begin{aligned}
|\mathbb{P}[D(w) = 1 : w \in_R G(K_{t-i})] - \mathbb{P}[D(w) = 1 : w \in_R U_{2\lambda}]| \\
= |\mathbb{P}[D_i(z') = 1 : z' \in_R H_i] - \mathbb{P}[D_i(z') = 1 : z' \in_R H_{i+1}]| \\
> \frac{1}{n^c}
\end{aligned}
$$
concluding that $D$ is a distinguisher for $G$, raising a contradiction over $G$ being a PRG.
By transitivity of $\approx_c$, the claim concludes that:
$$(G(K_1), \dots, G(K_t)) \equiv H_0 \approx_c \dots \approx_c H_t \equiv (U_{2\lambda}, \dots, U_{2\lambda})$$

##### Theorem 4.1: The GGM theorem
> [!abstract] Theorem 4.1
> **GGM Tree is a PRF when constructed using a PRG.**

**Proof:**
Let $G : \{0, 1\}^\lambda \to \{0, 1\}^{2\lambda}$ be a PRG and let $G_0, G_1 : \{0, 1\}^\lambda \to \{0, 1\}^\lambda$ be the two functions such that $G(x) = G_0(x) \,||\, G_1(x)$. Let also $\mathcal{F} = \{F_K : \{0, 1\}^n \to \{0, 1\}^\lambda\}_{K \in \{0,1\}^\lambda}$ be the GGM function family.

Fix a key $K \in \{0, 1\}^\lambda$. We proceed by induction on the length $n$ of the input.

**Base Case ($n=1$):**
It holds that $F_K(x) = G_x(K)$. Since $U_{2\lambda} \approx_c G(K) \equiv (G_0(K), G_1(K))$, we know that $G_0(K) \approx_c U_\lambda$ and $G_1(K) \approx_c U_\lambda$. This concludes that, independently from the value of $x$, the output $F_K(x)$ is indistinguishable from a string taken from $U_\lambda$, making it indistinguishable from a random function taken from $\mathcal{R}(1 \to \lambda)$.

**Inductive Step:**
We now assume that the inductive hypothesis holds for $n - 1$, i.e., that the GGM family $\mathcal{F}' = \{F'_K : \{0, 1\}^{n-1} \to \{0, 1\}^\lambda\}_{K \in \{0,1\}^\lambda}$ is a PRF.
First, we observe that for each key $K \in \{0, 1\}^\lambda$, for each $x \in \{0, 1\}$ and for each $y \in \{0, 1\}^{n-1}$ it holds that $F_K(x \,||\, y) = G_x(F'_K(y))$.

We define the following three hybrid distributions:
* $Hyb_0$: The distribution over the function $F_K(x \,||\, y) = G_x(F'_K(y))$.
* $Hyb_1$: The distribution over the function $H(x \,||\, y) = G_x(R'(y))$ where $R' \in_R \mathcal{R}(n - 1 \to \lambda)$.
* $Hyb_2$: The distribution over the function $R \in_R \mathcal{R}(n \to \lambda)$.

To prove that $\mathcal{F}$ is also a PRF, we’ll show that $Hyb_0 \approx_c Hyb_1 \approx_c Hyb_2$.

**Claim 1: $Hyb_0 \approx_c Hyb_1$**
We reduce the distinguishability of $Game^{PRF}_{\mathcal{F}', \mathcal{A}}(\lambda, 0)$ from $Game^{PRF}_{\mathcal{F}', \mathcal{A}}(\lambda, 1)$ to the distinguishability of $Hyb_0$ from $Hyb_1$. By way of contradiction, suppose that $Hyb_0 \not\approx_c Hyb_1$, meaning that there is a PPT adversary $D_{01}$ such that:
$$|\mathbb{P}[D_{01}(z) = 1 : z \in_R Hyb_0] - \mathbb{P}[D_{01}(z) = 1 : z \in_R Hyb_1]| > \frac{1}{n^c}$$
for some $c > 0$. We define the attack strategy $A_{01}$ for $Game^{PRF}_{\mathcal{F}', \mathcal{A}}(\lambda, b)$ as follows.

1.  The challenger $\mathcal{C}$ generates a key $K \in \{0, 1\}^\lambda$ and sets $J = F'_K$ if $b = 0$, otherwise $H \in_R \mathcal{R}(n - 1 \to \lambda)$.
2.  $A_{01}$ challenges $D_{01}$ to distinguish $Hyb_0$ from $Hyb_1$.
3.  For each query $x \,||\, y$ – with $x \in \{0, 1\}$ and $y \in \{0, 1\}^{n-1}$ – made by $D_{01}$, the adversary $A_{01}$ forwards $y$ to the challenger $\mathcal{C}$.
4.  For each answer $z$ returned by $\mathcal{C}$, the adversary $A_{01}$ forwards $G_x(z)$ to $D_{01}$.
5.  When $D_{01}$ gives the final bit $b'$, $A_{01}$ forwards the same bit to $\mathcal{C}$.

Since $D_{01}$ can distinguish between $F_K$ and $R$, $A_{01}$ will be able to distinguish $F'_K$ from $R'$. This contradicts the fact that $\mathcal{F}'$ is a PRF, concluding that $Hyb_0 \approx_c Hyb_1$ must be true.

> ![[Pasted image 20260125182220.png]]
> *Figure 4.5: Graphical representation of the double-game attack used to prove Claim 1 of the GGM theorem.*

**Claim 2: $Hyb_1 \approx_c Hyb_2$**
We reduce the distinguishability of $(G(K_1), \dots, G(K_t))$ from $(U_{2\lambda}, \dots, U_{2\lambda})$ to the distinguishability of $Hyb_1$ from $Hyb_2$. By way of contradiction, suppose that there is an algorithm $D_{12}$ that distinguishes $Hyb_1$ from $Hyb_2$.
For each string $w \in \{0, 1\}^{2\lambda}$, let $w_0, w_1 \in \{0, 1\}^\lambda$ denote the first and last $\lambda$ bits of $w$. Let $A_{12}$ be the algorithm defined as follows:

**$A_{12}$ = "On input $z = z_1 || \dots || z_t$ with $z_i \in \{0, 1\}^{2\lambda}$:**
1.  $A_{12}$ challenges $D_{12}$ to distinguish $Hyb_1$ from $Hyb_2$, allowing $D_{12}$ exactly $t$ queries.
2.  For each query $x \,||\, y$ made by $D_{12}$, the adversary $A_{12}$ answers with $z^{x}_{i(y)}$, where $i(y) \in [1, t]$ is either the index that was used by $A_{12}$ if $D_{12}$ already queried $y$ or the next unused index.
3.  When $D_{12}$ gives the final bit $b'$, $A_{12}$ returns $b'$."

When $z_1, \dots, z_t \in U_{2\lambda}$, all values returned by $A_{12}$ to $D_{12}$ will be truly random ($Hyb_2$). When $z_1, \dots, z_t \in G(U_\lambda)$, all values will be outputs of $G$ ($Hyb_1$). This concludes that $A_{12}$ is a distinguisher for $(G(K_1), \dots, G(K_t))$ and $(U_{2\lambda}, \dots, U_{2\lambda})$, contradicting Lemma 4.1.

> ![[Pasted image 20260125182658.png]]
> *Figure 4.6: Graphical representation of the double-game attack used to prove Claim 2 of the GGM theorem.*

## 4.3 Encryption modes for SKEs

After proving that OWFs can be used to create PRFs through PRGs, we’re going to show that PRFs are enough to achieve practical symmetric-key encryption.

**Block Ciphers vs. PRFs vs. PRPs:**
For some practical applications, we strictly require that the pseudorandom functions are invertible, i.e., that they are **Pseudorandom Permutations (PRPs)**. Practical functions with this property are commonly called **Block Ciphers**.
* **Theoretical Perspective:** PRFs can be used to derive PRPs (Luby-Rackoff Construction).
* **Practical Perspective:** We use existing block ciphers like AES.

**Variable Input Length (VIL):**
Real-world encryption systems must work with messages of different lengths. We formally say these SKEs have **Variable Input Length (VIL)**, meaning $|m| = poly(\lambda)$.
To handle this, each message $m$ is split into $d$ blocks of a fixed length $n$ (typically a power of 2, e.g., 256 bits).
$$m = (m_1, \dots, m_d) \quad \text{where } m_i \in \{0, 1\}^n$$

### 4.3.1 Electronic Code Book (ECB)

The simplest encryption mode applies the PRF independently to each block.

##### Definition 4.5: Electronic Code Book (ECB) mode
> [!def] Definition 4.5
> Let $\mathcal{F} = \{F_K : \{0, 1\}^n \to \{0, 1\}^n\}_{K \in \{0,1\}^\lambda}$ be a PRF. Let $K \in \{0, 1\}^\lambda$ be a chosen key.
> On input $m = (m_1, \dots, m_d)$, the **ECB encryption mode** returns the ciphertext $c = (c_1, \dots, c_d)$ where:
> * For all $i \in [d]$, $c_i = F_K(m_i)$.

> ![[Pasted image 20260125183130.png]]
> *Figure 4.7: Graphical representation of ECB mode.*

**Security Flaw:**
Clearly, ECB mode **cannot** be CPA-secure because it is **deterministic**. If $m_i = m_j$, then $c_i = c_j$. This reveals patterns in the plaintext (e.g., the famous "Penguin" image).

### 4.3.2 Cipher Block Chaining (CBC)

To achieve CPA security, we introduce a **nonce** value, usually referred to as the **Initialization Vector (IV)**.
CBC uses **sequential encryption**: the output of the previous block affects the input of the current block. This propagates randomness and makes decryption unparallelizable (though encryption is also unparallelizable).

##### Definition 4.6: Cipher Block Chaining (CBC) mode
> [!def] Definition 4.6
> Let $\mathcal{F} = \{F_K : \{0, 1\}^n \to \{0, 1\}^n\}_{K \in \{0,1\}^\lambda}$ be a PRF. Let $K \in \{0, 1\}^\lambda$ be a chosen key.
> On input $m = (m_1, \dots, m_d)$, the **CBC encryption mode** returns the ciphertext $c = (c_0, c_1, \dots, c_d)$ where:
> * $c_0 \in_R U_n$ is a nonce value (IV).
> * For all $i \in [d]$, $c_i = F_K(c_{i-1} \oplus m_i)$.

> ![[Pasted image 20260125183321.png]]
> *Figure 4.8: Graphical representation of CBC mode.*

### 4.3.3 Cipher Feedback (CFB)

CFB is similar to CBC but changes the order of operations. It effectively turns the block cipher into a self-synchronizing stream cipher.
We observe that this idea is similar to our PRF-OTP example, but with chaining.

##### Definition 4.7: Cipher Feedback (CFB) mode
> [!def] Definition 4.7
> Let $\mathcal{F} = \{F_K : \{0, 1\}^n \to \{0, 1\}^n\}_{K \in \{0,1\}^\lambda}$ be a PRF. Let $K \in \{0, 1\}^\lambda$ be a chosen key.
> On input $m = (m_1, \dots, m_d)$, the **CFB encryption mode** returns the ciphertext $c = (c_0, c_1, \dots, c_d)$ where:
> * $c_0 \in_R U_n$ is a nonce value (IV).
> * For all $i \in [d]$, $c_i = F_K(c_{i-1}) \oplus m_i$.

> ![[Pasted image 20260125183356.png]]
> *Figure 4.9: Graphical representation of CFB mode.*

### 4.3.4 Output Feedback (OFB)

OFB generates a keystream independent of the plaintext message. This effectively makes it a synchronous stream cipher. The output of the PRF is forwarded to the next stage rather than the ciphertext.

##### Definition 4.8: Output Feedback (OFB) mode
> [!def] Definition 4.8
> Let $\mathcal{F} = \{F_K : \{0, 1\}^n \to \{0, 1\}^n\}_{K \in \{0,1\}^\lambda}$ be a PRF. Let $K \in \{0, 1\}^\lambda$ be a chosen key.
> On input $m = (m_1, \dots, m_d)$, the **OFB encryption mode** returns the ciphertext $c = (c_1, \dots, c_d)$ where:
> * $q_0 \in_R U_n$ is a nonce value (IV).
> * For all $i \in [d]$, $q_i = F_K(q_{i-1})$ (Keystream generation).
> * For all $i \in [d]$, $c_i = q_i \oplus m_i$.

> ![[Pasted image 20260125183520.png]]
> *Figure 4.10: Graphical representation of OFB mode.*

### 4.3.5 Counter (CTR) Mode

CTR mode turns a block cipher into a stream cipher. Instead of chaining previous outputs, it encrypts a **counter** that increments for each block.
* **Parallelizable:** Both encryption and decryption can be fully parallelized.
* **Random Access:** You can decrypt the $i$-th block without decrypting previous ones.

##### Definition 4.9: Counter (CTR) mode
> [!def] Definition 4.9
> Let $\mathcal{F} = \{F_K : \{0, 1\}^n \to \{0, 1\}^n\}_{K \in \{0,1\}^\lambda}$ be a PRF. Let $K \in \{0, 1\}^\lambda$ be a chosen key.
> On input $m = (m_1, \dots, m_d)$, the **CTR encryption mode** returns the ciphertext $c = (c_0, c_1, \dots, c_d)$ where:
> * $r \in_R U_n$ is a nonce value (IV).
> * $c_0 = r$.
> * For all $i \in [d]$, $c_i = F_K(r + i - 1) \oplus m_i$.
>   *(Note: The addition $r+i-1$ is typically performed modulo $2^n$).*

> ![[Pasted image 20260125183611.png]]
> *Figure 4.11: Graphical representation of CTR mode.*

**Security Note:**
CBC, CFB, OFB, and CTR can all be proven to be **CPA-secure** for Variable Input Length (VIL) messages, assuming the underlying block cipher is a secure PRP/PRF.

##### Theorem 4.2: CPA-security through PRFs
> [!abstract] Theorem 4.2
> **CBC, CFB, OFB, and CTR are CPA-secure for Variable Input Length (VIL) when constructed using a PRF.**

**Proof:**
We prove the theorem only for **CTR mode**.
Let $G(\lambda, b) := Game^{CPA}_{\Pi, \mathcal{A}}(\lambda, b)$, where $\Pi$ is the CTR mode scheme constructed through a PRF $\mathcal{F}$.

**Setup:**
Recall that for $G(\lambda, b)$, the challenge consists of:
* Two messages $m^*_0, m^*_1$ chosen by the adversary.
* Let the chosen message be $m^*_b = (m^*_{b,1}, \dots, m^*_{b,d^*})$.
* The answer is a ciphertext $c^* = (c^*_0, c^*_1, \dots, c^*_{d^*})$ where:
    * $c^*_0 = r^* \in_R U_n$ (The random IV).
    * $c^*_i = F_K(r^* + i - 1) \oplus m^*_{b,i}$.

**The Hybrid Strategy:**
We define a hybrid distribution $H(\lambda, b)$ for both values of $b$.
* **$H(\lambda, b)$**: The distribution obtained by replacing the pseudorandom function $F_K(\cdot)$ with a **truly random function** $R(\cdot) \in \mathcal{R}(n \to n)$ in the game.
* In $H(\lambda, b)$, encryption is performed as $c^*_i = R(r^* + i - 1) \oplus m^*_{b,i}$.

We will prove security in two steps (Claims).

---

**Claim 1: $G(\lambda, b) \approx_c H(\lambda, b)$**
We show that replacing the PRF with a Random Function is undetectable.

**Proof of Claim 1:**
We reduce the distinguishability of $F_K$ from $R$ to the distinguishability of $G(\lambda, b)$ from $H(\lambda, b)$.
Fix $b \in \{0, 1\}$. Suppose there is an adversary $\mathcal{A}$ that distinguishes $G(\lambda, b)$ from $H(\lambda, b)$ with non-negligible probability. We construct an adversary $\mathcal{A}'$ that breaks the PRF security.

**Adversary $\mathcal{A}'$ Strategy:**
1.  $\mathcal{A}'$ interacts with a Challenger $\mathcal{C}$ in the PRF Game. $\mathcal{C}$ provides access to an oracle $O(\cdot)$ which is either $F_K(\cdot)$ or $R(\cdot)$.
2.  $\mathcal{A}'$ acts as the challenger for $\mathcal{A}$ in the CPA game.
3.  **Encryption Queries:** When $\mathcal{A}$ requests an encryption of $m^{(i)}$ (split into blocks):
    * $\mathcal{A}'$ chooses a random nonce $r^{(i)}$.
    * $\mathcal{A}'$ queries its oracle $O$ on inputs $r^{(i)}, r^{(i)} + 1, \dots, r^{(i)} + d - 1$.
    * $\mathcal{A}'$ receives outputs $z^{(i)}_1, \dots, z^{(i)}_d$.
    * $\mathcal{A}'$ computes ciphertext blocks $c^{(i)}_j = z^{(i)}_j \oplus m^{(i)}_j$ and returns $c^{(i)}$ to $\mathcal{A}$.
4.  **Challenge:** When $\mathcal{A}$ sends $m^*_0, m^*_1$:
    * $\mathcal{A}'$ picks $r^*$ and queries $O$ for the keystream.
    * $\mathcal{A}'$ encrypts $m^*_b$ and returns $c^*$.
5.  **Guess:** $\mathcal{A}$ outputs a bit. $\mathcal{A}'$ outputs the same bit to $\mathcal{C}$.

**Analysis:**
* If $\mathcal{C}$ uses $F_K$, $\mathcal{A}$ sees exactly $G(\lambda, b)$.
* If $\mathcal{C}$ uses $R$, $\mathcal{A}$ sees exactly $H(\lambda, b)$.
* If $\mathcal{A}$ distinguishes the games, $\mathcal{A}'$ distinguishes the PRF from Random. This contradicts the definition of a PRF.

---

**Claim 2: $H(\lambda, b) \approx_c U_{n(d^*+1)}$**
We show that in the Random Function world, the ciphertext is statistically close to uniform random noise (assuming nonces don't collide).

**Proof of Claim 2:**
Consider the input values fed to the function $R$ during the game:
* **Challenge Phase:** $r^*, r^*+1, \dots, r^*+d^*-1$.
* **Query Phase (query $i$):** $r^{(i)}, r^{(i)}+1, \dots, r^{(i)}+d_i-1$.

Let $B$ be the binary **"Bad Event"** (nonce overlap).
$B=1$ if the sequence of counters used for the challenge overlaps with the sequence of counters used for any other encryption query.
$$B = \exists i, j, j^* \text{ such that } r^{(i)} + j = r^* + j^*$$

**Conditioning on $B=0$:**
If no overlap occurs ($B=0$), the inputs to the random function $R$ for the challenge are distinct from all other inputs.
* The outputs $R(r^* + j^*)$ are fresh, uniformly random strings $u_{j^*}$.
* The ciphertext blocks are $c^*_{j^*} = m^*_{b, j^*} \oplus u_{j^*}$.
* By the property of the One-Time Pad, $c^*$ is perfectly uniform and reveals no information about $m^*_b$.
* Thus, conditioned on $B=0$, $H(\lambda, b)$ is identical to the uniform distribution $U$.

**Bounding the Bad Event:**
We must calculate $\mathbb{P}[B=1]$.
Fix a query $i$. Assume max length $d_i = d^* = q$.
Overlap occurs if the interval $[r^{(i)}, r^{(i)} + q - 1]$ intersects $[r^*, r^* + q - 1]$.
This happens if $r^{(i)}$ falls within the range $[r^* - q + 1, r^* + q - 1]$.
There are $(r^* + q - 1) - (r^* - q + 1) + 1 = 2q - 1$ "bad" values for $r^{(i)}$.
Since $r^{(i)}$ is chosen uniformly from $2^n$:
$$\mathbb{P}[B_i = 1] \leq \frac{2q - 1}{2^n}$$

Applying the Union Bound over $q$ queries:
$$\mathbb{P}[B=1] \leq \sum_{i=1}^q \mathbb{P}[B_i=1] \approx \frac{q(2q)}{2^n} = \frac{2q^2}{2^n}$$

Since $q$ is polynomial in $\lambda$ and $2^n$ is exponential, this probability is negligible.
Thus, $SD(H(\lambda, b); U) \leq \mathbb{P}[B=1] \approx negl(\lambda)$.

**Conclusion:**
Combining the claims via transitivity:
$$G(\lambda, 0) \approx_c H(\lambda, 0) \approx_c U \approx_c H(\lambda, 1) \approx_c G(\lambda, 1)$$
Therefore, CTR mode is CPA-secure. $\blacksquare$

## 4.4 UFCMA-security and Universal Hash Families

After showing that common encryption schemes are CPA-secure, we focus on secure message authentication.
Consider a tag function $Tag : \mathcal{K} \times \mathcal{M} \to \mathcal{T}$.
**The Goal:** It should be computationally hard to forge a valid pair $(m^*, \tau^*)$ without knowing the key $K$.

### 4.4.1 Unforgeability under Chosen-Message Attacks (UFCMA)

We define security via the game $Game^{UFCMA}_{\Pi, \mathcal{A}}(\lambda)$:
1.  **Setup:** The challenger $\mathcal{C}$ generates a key $K$.
2.  **Queries:** The adversary $\mathcal{A}$ sends messages $m_1, \dots, m_t$ to an oracle for $Tag(K, \cdot)$ and receives $\tau_1, \dots, \tau_t$.
3.  **Forgery:** $\mathcal{A}$ outputs a pair $(m^*, \tau^*)$.
4.  **Win Condition:** $\mathcal{A}$ wins if:
    * $Tag(K, m^*) = \tau^*$
    * $m^* \neq m_i$ for all queried messages $i \in [t]$. (The message must be fresh).

> ![[Pasted image 20260125184815.png]]
> *Figure 4.12: Graphical representation of $Game^{UFCMA}_{\Pi, \mathcal{A}}(\lambda)$.*

##### Definition 4.10: UFCMA-security
> [!def] Definition 4.10
> Let $\Pi = (Tag)$ be a MAC. We say that $\Pi$ is **UFCMA-secure** if:
> $$\mathbb{P}[Game^{UFCMA}_{\Pi, \mathcal{A}}(\lambda) = 1] \leq negl(\lambda)$$
> for all PPT adversaries $\mathcal{A}$.

### 4.4.2 Constructing MACs from PRFs

Pseudorandom functions (PRFs) provide an intuitive way to build MACs. Since a PRF behaves like a random function, an adversary cannot predict the output for a new input.

##### Theorem 4.3: UFCMA-security through PRFs
> [!abstract] Theorem 4.3
> If $\mathcal{F} = \{F_K : \{0, 1\}^n \to \{0, 1\}^n\}_{K \in \{0,1\}^\lambda}$ is a PRF family, then $Tag(K, m) = F_K(m)$ is **UFCMA-secure for Fixed Input Length (FIL)** messages.

**Proof:**
Let $G(\lambda)$ be the real UFCMA game using $F_K$.
Let $H(\lambda)$ be a hybrid game using a truly random function $R(\cdot) \in \mathcal{R}(n \to n)$.
1.  **Indistinguishability:** $G(\lambda) \approx_c H(\lambda)$. If an adversary could distinguish them, they would distinguish the PRF from Random.
2.  **Unforgeability in H:** In $H(\lambda)$, the tag is a truly random string $R(m^*)$. Since $m^*$ is fresh, $R(m^*)$ has never been sampled. The probability of guessing it is $2^{-n}$.
Thus, the scheme is secure.

**The Problem with Variable Input Length (VIL):**
This simple construction fails if we want to authenticate messages of variable length (VIL). Simple domain extension tricks (concatenation, XOR) are **insecure**.
* **Concatenation Attack ($Tag(m) = (\tau_1, \dots, \tau_d)$):** Adversary queries $m_1 || m_2$ (gets $\tau_1, \tau_2$) and $m_3 || m_4$ (gets $\tau_3, \tau_4$). They can forge the tag for $m_1 || m_3$ as $\tau_1 || \tau_3$.
* **XOR Attack ($Tag(m) = \bigoplus \tau_i$):** Adversary queries $m_1 || m_1$ (gets $\tau_1$). They can forge the tag for $m_2 || m_2$ (also $\tau_1$) or other combinations.

---

### 4.4.3 Universal Hash Families

To handle VIL or long inputs, we use a **Hash-and-MAC** approach:
$$Tag(K, m) = F_K(h_s(m))$$
where $h_s$ compresses a long input to a short block, and $F_K$ secures it.
We need a special property for the hash family: **low collision probability**.

##### Definition 4.11: Universal Hash Families
> [!def] Definition 4.11
> Let $\mathcal{H} = \{h_s : \{0, 1\}^{nd} \to \{0, 1\}^n\}_{s \in \{0,1\}^\lambda}$ be a family of hash functions.
> $\mathcal{H}$ is **$\varepsilon$-almost universal** if $\forall m, m' \in \mathcal{M}$ with $m \neq m'$:
> $$\mathbb{P}_{s \in_R \{0,1\}^\lambda} [h_s(m) = h_s(m')] \leq \varepsilon$$

* **Almost Universal (AU):** $\varepsilon = negl(\lambda)$.
* **Perfectly Universal (PU):** $\varepsilon = 2^{-n}$ (The collision probability of a truly random function).

##### Theorem 4.4: Composition of PRFs with AU Hash Families
> [!abstract] Theorem 4.4
> Given a PRF family $\mathcal{F}$ and an AU hash family $\mathcal{H}$, the composed family $\mathcal{F}(\mathcal{H})$ defined as:
> $$\mathcal{F}(\mathcal{H}) = \{F_K \circ h_s : \{0, 1\}^{nd} \to \{0, 1\}^n\}_{(K,s)}$$
> is a **PRF family** (and thus a secure MAC).

**Proof:**
Consider the two games $G(\lambda, b) := Game^{PRF}_{\mathcal{F}(\mathcal{H}), \mathcal{A}}(\lambda, b)$ with $b \in \{0, 1\}$.
* $G(\lambda, 0)$ is the PRF game where we use the composed function $F_K(h_s(\cdot)) \in \mathcal{F}(\mathcal{H})$.
* $G(\lambda, 1)$ is the PRF game where we use a truly random function $R \in \mathcal{R}(nd \to n)$.

Let $H(\lambda)$ be a **hybrid distribution** where the game is played using the function $R'(h_s(\cdot))$, where $R' \in \mathcal{R}(n \to n)$ is a truly random function on the compressed domain.

**Step 1: $G(\lambda, 0) \approx_c H(\lambda)$**
It can be proven that $G(\lambda, 0) \approx_c H(\lambda)$ in the standard way. We reduce the distinguishability of the PRF $F_K$ from a random function $R'$ to the distinguishability of $G(\lambda, 0)$ from $H(\lambda)$. Since $\mathcal{F}$ is a PRF, this indistinguishability holds.

**Step 2: $H(\lambda) \approx_c G(\lambda, 1)$**
We must show that the composition of a Random Function and a Hash Function looks like a Random Function.
Let $m_1, \dots, m_t$ be the queries made by the adversary, where $t = poly(\lambda)$.
Let $B$ be the **"Bad Event"** such that $B = 1$ if there exists a collision in the hash outputs:
$$B = 1 \iff \exists i, j \in [t], i \neq j \text{ such that } h_s(m_i) = h_s(m_j)$$

**Conditioning on $B=0$:**
If no collisions occur ($B=0$), then all inputs to the internal function $R'$ are distinct. Since $R'$ is a truly random function, on distinct inputs it produces independent, uniformly random outputs. This is identical to the behavior of the truly random function $R$ in $G(\lambda, 1)$.
Thus, the statistical distance is bounded by the probability of the bad event:
$$SD(H(\lambda); G(\lambda, 1)) \leq \mathbb{P}[B = 1]$$

**Bounding the Bad Event:**
Using the Union Bound and the definition of an $\varepsilon$-Almost Universal hash family (where $\varepsilon = negl(\lambda)$):

$$
\begin{aligned}
\mathbb{P}[B = 1] &= \mathbb{P}_{s \in_R \{0,1\}^\lambda} [\exists i, j \in [t], i \neq j \text{ s.t. } h_s(m_i) = h_s(m_j)] \\
&\leq \sum_{i=1}^{t} \sum_{j=0, j \neq i}^{t} \mathbb{P}_{s \in \{0,1\}^\lambda} [h_s(m_i) = h_s(m_j)] \\
&\leq \binom{t}{2} \cdot negl(\lambda) \\
&\leq t^2 \cdot negl(\lambda) = negl(\lambda)
\end{aligned}
$$

**Conclusion:**
Since the probability of collision is negligible, $H(\lambda) \approx_c G(\lambda, 1)$.
Combining the steps: $G(\lambda, 0) \approx_c H(\lambda) \approx_c G(\lambda, 1)$. $\blacksquare$

---

### 4.4.4 Examples of Universal Hash Families

We can construct these efficiently using Galois Fields.

**1. Perfectly Universal (PU) via Scalar Product:**
Let input $m \in \mathbb{F}^d$ and seed $s \in \mathbb{F}^d$ (where $\mathbb{F} = GF(2^n)$).
$$h_s(m) = \langle s, m \rangle = \sum_{i=1}^d s_i m_i$$
This is **Perfectly Universal** ($\varepsilon = 2^{-d}$). A collision requires $\sum s_i (m_i - m'_i) = 0$, which occurs with probability $2^{-d}$ over random $s$.

**2. Almost Universal (AU) via Polynomial Evaluation:**
Let input $m \in \mathbb{F}^d$ define the coefficients of a polynomial, and seed $s \in \mathbb{F}$ be the point of evaluation.
$$h_s(m) = Q_m(s) = \sum_{i=1}^d m_i s^{i-1}$$
This is **Almost Universal** ($\varepsilon = (d-1)2^{-n}$). Two distinct polynomials of degree $d-1$ can intersect at most $d-1$ times.

**3. CBC-MAC:**
We can use the CBC encryption structure to define a hash function.

##### Definition 4.12: CBC-MAC
> [!def] Definition 4.12
> Let $\mathcal{F}$ be a PRF family. The **CBC-MAC** family $\mathcal{H}_{CBC}$ is defined as:
> $$h_s(m_1, \dots, m_d) = F_s(m_d \oplus F_s(m_{d-1} \oplus \dots F_s(m_1) \dots ))$$
> (Essentially the last block of CBC encryption with IV=0).

> ![[Pasted image 20260125185919.png]]
> *Figure 4.13: Graphical representation of CBC-MAC.*

**Properties:**
* $\mathcal{H}_{CBC}$ is a PRF itself.
* $\mathcal{H}_{CBC}$ is **Almost Universal** for VIL.
* The composition $\mathcal{F}(\mathcal{H}_{CBC})$ is UFCMA-secure for Variable Input Lengths.

##### Theorem 4.5
> [!abstract] Theorem 4.5
> Let $\mathcal{F}$ be a PRF family and $\mathcal{H}_{CBC}$ be the corresponding CBC-MAC hash family. Then:
> * $\mathcal{H}_{CBC}$ is a PRF.
> * $\mathcal{H}_{CBC}$ is AU for VIL.
> * $\mathcal{F}(\mathcal{H}_{CBC})$ is a PRF.
> * $\mathcal{F}(\mathcal{H}_{CBC})$ is UFCMA-secure for VIL.

## 4.5 CCA-security and non-malleability

In the previous sections we gave the idea behind computationally secure encryption and computationally secure message authentication. Now, we’re ready to combine the two concepts.

### Malleability
We say that an encryption scheme is **malleable** if it is possible to transform a ciphertext $c \in \mathcal{C}$ – of which we don’t know the original plaintext – into another ciphertext $\tilde{c}$ which decrypts to another plaintext related to the original one. Formally, given an encryption $c$ of a plaintext $m$, it is possible to generate another ciphertext $\tilde{c}$ of another plaintext $\tilde{m}$ that is in some way related to $m$.

**Example: PRG-OTP**
Consider again the PRG-OTP scheme, where $Enc(K, m) = G(K) \oplus m$. Suppose that we know a ciphertext $c$, without knowing the original message $m$. Then, for any value $t \in \{0, 1\}^n$ we can construct the ciphertext $\tilde{c}$ of the message $\tilde{m} = m \oplus t$ by XORing $c$ with $t$:
$$\tilde{c} = c \oplus t = (G(K) \oplus m) \oplus t = G(K) \oplus (m \oplus t) = Enc(K, \tilde{m})$$

Malleability is often an undesirable property in a general-purpose cryptosystem (e.g., digital auctions), since it allows an attacker to modify the contents of a message. It’s easy to see that even CPA-secure SKEs may be malleable.

**Example: PRF-OTP**
Consider the PRF-OTP scheme:
1.  $Enc(K, m) = (r, F_K(r) \oplus m)$
2.  $Dec(K, (r, c)) = F_K(r) \oplus c$

We already discussed how this scheme is CPA-secure. Consider now the ciphertext $(r, c)$ of an unknown message $m$. Let $\tilde{c} = c \oplus t$. Then:
$$(r, \tilde{c}) = (r, c \oplus t) = (r, F_K(r) \oplus (m \oplus t)) = Enc(K, m \oplus t)$$

### CCA-Security
In the computational security context, malleability attacks are known as **Chosen-Ciphertext Attacks (CCA)**.
Formally, CCA-security is defined through the game $Game^{CCA}_{\Pi, \mathcal{A}}(\lambda, b)$, similar to the one for CPA-security with the addition of **decryption queries**. The adversary can make decryption queries both before and after receiving the challenge ciphertext $c^*$. Obviously, the ciphertext sent in each decryption query must be strictly different from the challenge ciphertext ($c \neq c^*$).

> ![[Pasted image 20260125190314.png]]
*Figure 4.14: Graphical representation of $Game^{CCA}_{\Pi, \mathcal{A}}(\lambda, b)$.*

##### Definition 4.13: CCA-security
> [!def] Definition 4.13
> Let $\Pi = (Enc, Dec)$ be an SKE. We say that $\Pi$ is **CCA-secure** if:
> $$Game^{CCA}_{\Pi, \mathcal{A}}(\lambda, 0) \approx_c Game^{CCA}_{\Pi, \mathcal{A}}(\lambda, 1)$$
> for all PPT adversaries $\mathcal{A}$.

### Integrity (INT)
Since the CCA-game is complex due to decryption queries, we use a simpler property to prove security: **Integrity (INT)**.
This is the computational equivalent of message integrity: it must be hard for any PPT adversary to generate a **valid ciphertext** without knowing the key.

In $Game^{INT}_{\Pi, \mathcal{A}}(\lambda)$:
1.  The adversary makes $t = poly(\lambda)$ encryption queries.
2.  The adversary returns a ciphertext $c^*$ (where $c^* \neq c_i$ for all previous queries).
3.  The challenger computes $Dec(K, c^*)$.
4.  If $Dec(K, c^*) \neq \perp$ (valid message), the adversary wins. Otherwise, the challenger wins.

> ![[Pasted image 20260125190520.png]]
*Figure 4.15: Graphical representation of $Game^{INT}_{\Pi, \mathcal{A}}(\lambda)$*

##### Definition 4.14: Integrity
> [!def] Definition 4.14
> Let $\Pi = (Enc, Dec)$ be an SKE. We say that $\Pi$ satisfies **integrity** if:
> $$\mathbb{P}[Game^{INT}_{\Pi, \mathcal{A}}(\lambda) = 1] \leq negl(\lambda)$$
> for all PPT adversaries $\mathcal{A}$.

##### Theorem 4.6: CPA+INT implies CCA
> [!abstract] Theorem 4.6
> Let $\Pi = (Enc, Dec)$ be an SKE. Then, $\Pi$ is **CCA-secure** if it is **CPA-secure** and satisfies **integrity**.

**Proof (Sketch):**
We reduce both the distinguishability of the CPA game and the Integrity game to the CCA game.
We define a hybrid game $H(\lambda, b)$ identical to the CCA game but with modified decryption logic: if the adversary queries a ciphertext $\tilde{c}$ that was the output of a previous encryption query, return the known message $m$. Otherwise, return $\perp$.
1.  **Claim 1:** $H(\lambda, 0) \approx_c H(\lambda, 1)$. This follows directly from CPA-security (since the decryption oracle gives no new info).
2.  **Claim 2:** $Game^{CCA} \approx_c H$. The games only differ if the adversary submits a *fresh* valid ciphertext. If they do, they break Integrity. Since Integrity holds, this happens with negligible probability.

---

### Authenticated Encryption
To obtain a CCA-secure scheme, we combine a CPA-secure SKE ($\Pi_1$) with a secure MAC ($\Pi_2$).

**Combination Methods:**
1.  **Encrypt-and-MAC (E&M):** $c = Enc(m), \tau = Tag(m)$. Output $(c, \tau)$. (Used in old SSL/TLS).
2.  **MAC-then-Encrypt (MtE):** $c = Enc(m \,||\, Tag(m))$. Output $c$. (Used in SSH).
3.  **Encrypt-then-MAC (EtM):** $c = Enc(m), \tau = Tag(c)$. Output $(c, \tau)$. (Used in IPsec).

**Security Analysis:**
Only **Encrypt-then-MAC (EtM)** guarantees CCA security generically. This requires the MAC to be **Strong UFCMA (SUFCMA)** secure.
* **SUFCMA:** Requires that it is hard to forge a new tag $\tau^*$ even for a message $m$ that was already tagged (uniqueness of tags).

##### Definition 4.15: SUFCMA-security
> [!def] Definition 4.15
> Let $\Pi = (Tag)$ be a MAC. We say that $\Pi$ is **SUFCMA-secure** if:
> $$\mathbb{P}[Game^{SUFCMA}_{\Pi, \mathcal{A}}(\lambda) = 1] \leq negl(\lambda)$$
> for all PPT adversaries $\mathcal{A}$.

##### Theorem 4.7
> [!abstract] Theorem 4.7
> Let $\Pi' = (Enc', Dec')$ be an **EtM** SKE constructed from $\Pi_1$ and $\Pi_2$.
> Then, $\Pi'$ satisfies both **CPA-security** and **integrity** if $\Pi_1$ is CPA-secure and $\Pi_2$ is SUFCMA-secure.

**Proof:**
1.  **CPA-Security:** The adversary $\mathcal{A}_1$ uses the CPA game of $\Pi_1$. It generates its own MAC key $K_2$ to handle the tagging. If $\Pi'$ is broken, $\Pi_1$ is broken.
2.  **Integrity:** The adversary $\mathcal{A}_2$ uses the SUFCMA game of $\Pi_2$. It generates its own encryption key $K_1$. To win the integrity game, the adversary must provide a valid pair $(c^*, \tau^*)$. If the pair is valid, $\tau^*$ must be a valid tag for $c^*$. Since the pair is fresh, this constitutes a MAC forgery.

##### Corollary 4.1
> [!check] Corollary 4.1
> Let $\Pi'$ be an EtM SKE. Then, $\Pi'$ is **CCA-secure** if $\Pi_1$ is CPA-secure and $\Pi_2$ is SUFCMA-secure.

## 4.6 Block Ciphers and Feistel Networks

We have seen how to construct CCA-secure encryption using PRFs and MACs. However, there is a missing link: **Can we construct Pseudorandom Permutations (PRPs)?**
Real-world cryptosystems (Block Ciphers) rely on functions that are efficient to compute *and* invert. Since standard PRFs are not necessarily invertible, we need a way to build PRPs from PRFs.

### 4.6.1 Feistel Networks

The **Feistel Network** is a theoretical construction used to build a PRP from any PRF. It is the backbone of the **DES (Data Encryption Standard)** block cipher.
The core idea is to process the input in "rounds", splitting the data into two halves.

##### Definition 4.16: Feistel Function
> [!def] Definition 4.16
> Given a function $f : \{0, 1\}^n \to \{0, 1\}^n$, the **Feistel function** over $f$ is the function $\psi_f : \{0, 1\}^{2n} \to \{0, 1\}^{2n}$ such that:
> $$\psi_f(x, y) = (y, x \oplus f(y))$$
> with $x, y \in \{0, 1\}^n$.

> ![[Pasted image 20260125191400.png]]
*Figure 4.16: A Feistel function mechanism. Note that the left half becomes the right half, and the right half is XORed with the function output.*

**Invertibility:**
The Feistel function is invertible regardless of whether $f$ is invertible.
$$\psi^{-1}_f(x', y') = (y' \oplus f(x'), x')$$
Checking the inverse:
$$\psi^{-1}_f(\psi_f(x, y)) = \psi^{-1}_f(y, x \oplus f(y)) = ((x \oplus f(y)) \oplus f(y), y) = (x, y)$$

**Security Issue:**
A single Feistel round is **not** a PRP. The first half of the output is equal to the second half of the input ($y$), making it trivial to distinguish from random. Strength comes from repetition.

##### Definition 4.17: Feistel Network
> [!def] Definition 4.17
> Given $t$ functions $f_1, \dots, f_t : \{0, 1\}^n \to \{0, 1\}^n$, the **Feistel network** is the composite function:
> $$\psi_{f_1, \dots, f_t}(x, y) = \psi_{f_t}(\psi_{f_{t-1}}(\dots \psi_{f_1}(x, y)))$$

> ![[Pasted image 20260125191520.png]]
*Figure 4.17: A 3-layer Feistel network.*

### 4.6.2 The Luby-Rackoff Theorem

How many rounds are needed to make a Feistel Network secure?
* **1 Round:** Insecure (output $y$ leaks input $y$).
* **2 Rounds:** Insecure.
    * $\psi_{f_1, f_2}(x, y) = (x \oplus f_1(y), y \oplus f_2(x \oplus f_1(y)))$.
    * An attacker can query $(x, y)$ and $(x', y')$. If they choose inputs carefully, they can check if the XOR sum of the left halves matches $x \oplus x'$, distinguishing it from random.
* **3 Rounds:** **Secure (PRP).** This is the famous Luby-Rackoff result.

To prove this, we first need a lemma about "y-unique" queries.

##### Lemma 4.2
> [!abstract] Lemma 4.2
> Let $H$ be the distribution $\psi_{R_1, R_2}$ (2-round Feistel with random functions) and $H'$ be a truly random function $R : \{0, 1\}^{2n} \to \{0, 1\}^{2n}$.
> If all queries $(x_1, y_1), \dots, (x_q, y_q)$ have unique right halves ($y_i \neq y_j$), then $H$ and $H'$ are **computationally indistinguishable**.

##### Theorem 4.8: Luby-Rackoff Theorem
> [!abstract] Theorem 4.8
> Let $\mathcal{F}$ be a PRF family. Then, for any three independent keys $K_1, K_2, K_3$, the **3-layer Feistel network** $\psi_{F_{K_1}, F_{K_2}, F_{K_3}}$ is a **Pseudorandom Permutation (PRP)**.

**Proof:**
We use a sequence of hybrid distributions $H_1, \dots, H_4$:
1.  $H_1$: The real 3-round Feistel with PRFs $F_{K_1}, F_{K_2}, F_{K_3}$.
2.  $H_2$: 3-round Feistel with truly random functions $R_1, R_2, R_3$.
3.  $H_3$: A truly random *function* $R : \{0, 1\}^{2n} \to \{0, 1\}^{2n}$.
4.  $H_4$: A truly random *permutation* $P : \{0, 1\}^{2n} \to \{0, 1\}^{2n}$.

**Claim 1: $H_1 \approx_c H_2$**
Standard reduction: If we can distinguish PRFs from Random Functions, we can distinguish $H_1$ from $H_2$. Since $\mathcal{F}$ is a PRF, this holds.

**Claim 2: $H_3 \approx_c H_4$**
Distinguishing a random function from a random permutation is hard if the domain size $2^{2n}$ is large compared to the number of queries $q$. The collision probability ("Birthday Paradox") is $q^2 / 2^{2n}$, which is negligible.

**Claim 3: $H_2 \approx_c H_3$**
We view $H_2$ as $\psi_{R_2, R_3}(\psi_{R_1}(\cdot))$.
The first round $\psi_{R_1}$ randomizes the inputs.
Let $(x'_i, y'_i)$ be the output after the first round. We need the right halves $y'_i$ to be unique to apply Lemma 4.2.
$$y'_i = x_i \oplus R_1(y_i)$$
The probability of a collision $y'_i = y'_j$ (for distinct inputs) is bounded by the probability that the random function $R_1$ outputs specific values to cancel the XOR difference.
$$\mathbb{P}[y'_i = y'_j] \approx \frac{1}{2^n}$$
Summing over all pairs, the total probability of a "bad event" (collision) is negligible.
Conditioned on no collisions, Lemma 4.2 applies, making the remaining 2 rounds indistinguishable from a random function. $\blacksquare$

### 4.6.3 Practical Block Ciphers (DES & AES)

**Data Encryption Standard (DES):**
* Based on a **16-round Feistel Network**.
* **Key Length:** 56 bits (Too short! Easily brute-forced today).
* **Fix:** **3DES** applies DES three times with different keys ($k_{eff} \approx 112$ or $168$ bits). It is secure but slow.

**Advanced Encryption Standard (AES):**
* Replaced DES in 2001.
* **Not a Feistel Network.** It uses a **Substitution-Permutation Network (SPN)**.
* **Key Lengths:** 128, 192, or 256 bits.
* Highly efficient in hardware and software.

## 4.7 Collision-Resistant Hash Families

When we discussed domain expansion for secure MACs, we used hash families to "compress" long inputs into a short string compatible with a PRF. In that context (Hash-and-MAC), collisions were not a major issue because the output was hidden by a secret key inside the PRF.

However, in many cases (e.g., digital signatures, file integrity), we require **public** compressing hash functions without a secret key. Since the function maps a large domain to a smaller range, collisions inevitably exist (Pigeonhole Principle).
**The Requirement:** It must be computationally hard for a bounded adversary to *find* these collisions.

### 4.7.1 Defining Collision Resistance (CRH)

We define security via a game where the adversary tries to output a colliding pair for a randomly selected hash function from the family.

**The Game $Game^{CRH}_{\mathcal{H}, \mathcal{A}}(\lambda)$:**
1.  **Setup:** The challenger selects a key/seed $s \in_R U_\lambda$. (Note: In practice, this "key" is public parameters).
2.  **Attack:** The adversary outputs two distinct inputs $x, x' \in \mathcal{M}$ ($x \neq x'$).
3.  **Win Condition:** The adversary wins if $h_s(x) = h_s(x')$.

> ![[Pasted image 20260125192200.png]]
> *Figure 4.18: Graphical representation of $Game^{CRH}_{\mathcal{H}, \mathcal{A}}(\lambda)$.*

##### Definition 4.18: Collision-Resistant Hash
> [!def] Definition 4.18
> Let $\mathcal{H} = \{h_s : \{0, 1\}^{\ell(n)} \to \{0, 1\}^n\}_{s \in \{0,1\}^\lambda}$ be a hash family where $\ell(n) \gg n$.
> We say $\mathcal{H}$ is **collision-resistant (CRH)** if:
> $$\mathbb{P}[Game^{CRH}_{\mathcal{H}, \mathcal{A}}(\lambda) = 1] \leq negl(\lambda)$$
> for all PPT adversaries $\mathcal{A}$.

---

### 4.7.2 The Merkle-Damgård Transform

We often have a small, fixed-input-length (FIL) compression function (e.g., taking $2n$ bits to $n$ bits) and want to build a hash function for arbitrary lengths. The **Merkle-Damgård transform** is the standard method for this (used in MD5, SHA-1, SHA-2).

##### Definition 4.19: Merkle-Damgård Transform
> [!def] Definition 4.19
> Let $\mathcal{H} = \{h_s : \{0, 1\}^{2n} \to \{0, 1\}^n\}$ be a compression function.
> The **Merkle-Damgård transform** $H_{MD}$ works as follows for an input $x = (x_1, \dots, x_d)$ where each $x_i \in \{0, 1\}^n$:
> * Set $y_0 = 0^n$ (Initialization Vector).
> * For $i = 1$ to $d$: compute $y_i = h_s(y_{i-1} || x_i)$.
> * Output $y_d$.

> ![[Pasted image 20260125192310.png]]
> *Figure 4.19: The Merkle-Damgård transform structure.*

##### Proposition 4.4
> [!check] Proposition 4.4
> If the compression function $\mathcal{H}$ is collision-resistant, then $H_{MD}$ is collision-resistant for **Fixed Input Length (FIL)** (i.e., $d$ is fixed).

**Proof Sketch:**
We prove this by reduction. Suppose we find a collision in $H_{MD}$: two messages $x \neq x'$ such that $H_{MD}(x) = H_{MD}(x')$.
Let $y_0, \dots, y_d$ and $y'_0, \dots, y'_d$ be the intermediate values.
Since the final outputs match ($y_d = y'_d$) but inputs differ, we trace backwards. Let $i$ be the last index where the inputs to the compression function differed.
Then $h_s(y_{i-1} || x_i) = h_s(y'_{i-1} || x'_i)$.
This constitutes a collision in the underlying function $h_s$.

**The Flaw with Variable Input Length (VIL):**
This basic construction is **insecure** if message lengths vary.
* **Attack:** If $h_s(0^{2n}) = 0^n$, then $H_{MD}(x) = H_{MD}(0^n || x)$.
* **Length Extension:** Given $H(x)$, one can easily compute $H(x || suffix)$ without knowing $x$.

### 4.7.3 Strengthened Merkle-Damgård

To handle Variable Input Length (VIL), we perform **Merkle-Damgård Strengthening**: we append the length of the message $\langle d \rangle$ at the end. This ensures that no valid message is a prefix of another valid message.

##### Definition 4.20: Strengthened Merkle-Damgård Transform
> [!def] Definition 4.20
> Let $\mathcal{H} = \{h_s : \{0, 1\}^{2n} \to \{0, 1\}^n\}$. The **Strengthened Transform** $H^*_{MD}$ is defined as:
> * Input $x = (x_1, \dots, x_d)$.
> * Compute $y_d$ using the standard Merkle-Damgård chain.
> * **Final Step:** Output $y = h_s(y_d || \langle d \rangle)$, where $\langle d \rangle$ is the length encoding.

> ![[Pasted image 20260125192425.png]]
> *Figure 4.20: The Strengthened Merkle-Damgård transform with length padding.*

##### Theorem 4.9
> [!abstract] Theorem 4.9
> If $\mathcal{H}$ is collision-resistant, then $H^*_{MD}$ is collision-resistant for **Variable Input Length (VIL)**.

**Proof:**
Suppose we find a collision $H^*_s(x) = H^*_s(x')$.
* **Case 1 ($|x| \neq |x'|$):** The inputs to the final compression block are different because the length encodings differ ($\langle d \rangle \neq \langle d' \rangle$). Yet the outputs match. This is a collision in $h_s$.
* **Case 2 ($|x| = |x'|$):** If lengths are equal, the reduction proceeds exactly as in the standard Merkle-Damgård proof (tracing back to the first difference).

---

### 4.7.4 Other Constructions and Practical Usage

**Merkle Trees:**
Instead of a linear chain, we can hash blocks in a tree structure. This allows for parallel computation and efficient verification of sub-blocks.

> ![[Pasted image 20260125192636.png]]
> *Figure 4.21: A Merkle tree with 4 levels.*

**Practical Implementations:**
* **MD5, SHA-1, SHA-2:** Based on Merkle-Damgård. The compression functions are heuristic (block ciphers in specific modes).
* **SHA-3:** Based on a different structure called a **Sponge Construction**.

**Theoretical Constructions:**
We can build CRH families from:
* **Number Theoretic Assumptions:** Hardness of discrete log or factoring.
* **Block Ciphers (Davies-Meyer):** $h_s(x_1, x_2) = AES_{x_1}(x_2) \oplus x_2$. This is secure assuming AES behaves like an Ideal Cipher.

### 4.7.5 The Random Oracle Model (ROM)

Hash functions are often used to build MACs, e.g., $Tag(K, m) = H(K || m)$.
* If $H$ is a standard Merkle-Damgård hash, this is **insecure** due to length extension attacks.
* However, if we model $H$ as a **truly random function**, the scheme is secure.

This leads to a theoretical model used to analyze schemes that use hash functions as "black box" random sources.

##### Assumption 4.1: Random Oracle Model (ROM)
> [!def] Assumption 4.1
> In the **Random Oracle Model (ROM)**, we model a hash function $H$ as a truly random function accessible only via oracle queries.
> * **Consistency:** If $x$ has been queried before, return the same $y$.
> * **Randomness:** If $x$ is new, return a fresh uniform $y$.

This model simplifies proofs (e.g., for HMAC or RSA-OAEP) but is an idealization—real hash functions have structure.

## 4.8 Solved Exercises

### Problem 4.1: Constructing PRFs via Concatenation
**Question:**
Let $\mathcal{F} = \{F_K : \{0, 1\}^n \to \{0, 1\}^n\}_{K \in \{0,1\}^\lambda}$ be a PRF family.
Consider the family $\mathcal{F}' = \{F'_K : \{0, 1\}^{n-1} \to \{0, 1\}^{2n}\}_{K \in \{0,1\}^\lambda}$ defined as:
$$F'_K(m) = F_K(0 || m) \,||\, F_K(m || 1)$$
Prove or disprove that $\mathcal{F}'$ is a PRF family.

##### Proof (Disproof)
**Verdict:** The claim is **False**. $\mathcal{F}'$ is not a PRF.

We disprove it by constructing an adversary $\mathcal{A}$ that distinguishes $F'_K$ from a random function $R$ with high probability.

**Adversary $\mathcal{A}$:**
1.  **Query 1:** Choose $m_1 = 0^{n-1}$.
    The oracle returns $y_1 = J(m_1)$.
    Let's parse $y_1$ as $s_1 || t_1$, where $|s_1|=|t_1|=n$.
2.  **Query 2:** Choose $m_2 = 0^{n-2} || 1$.
    The oracle returns $y_2 = J(m_2)$.
    Let's parse $y_2$ as $s_2 || t_2$.
3.  **Check:** If $t_1 = s_2$, return 1 (Pseudorandom). Otherwise, return 0 (Random).

**Detailed Analysis:**
* **Case 1: $J = F'_K$ (Real World)**
    * Compute $y_1 = F'_K(0^{n-1}) = F_K(0 || 0^{n-1}) \,||\, F_K(0^{n-1} || 1)$.
        Thus, the second half is $t_1 = F_K(0^{n-1} || 1)$.
    * Compute $y_2 = F'_K(0^{n-2} || 1) = F_K(0 || 0^{n-2} || 1) \,||\, F_K(0^{n-2} || 1 || 1)$.
        Note that the first input is $0 || 0^{n-2} || 1 = 0^{n-1} || 1$.
        Thus, the first half is $s_2 = F_K(0^{n-1} || 1)$.
    * Since $t_1$ and $s_2$ are outputs of the deterministic function $F_K$ on the same input, $t_1 = s_2$ holds with **probability 1**.
* **Case 2: $J = R$ (Random World)**
    * The oracle uses a truly random function $R: \{0, 1\}^{n-1} \to \{0, 1\}^{2n}$.
    * $y_1$ and $y_2$ are results of $R$ on distinct inputs ($m_1 \neq m_2$).
    * Therefore, $y_1$ and $y_2$ are independent uniformly distributed strings.
    * The probability that the second $n$ bits of $y_1$ match the first $n$ bits of $y_2$ is:
        $$\Pr[t_1 = s_2] = \frac{1}{2^n}$$

**Conclusion:**
The advantage is $|1 - 2^{-n}|$, which is non-negligible. $\mathcal{F}'$ is not a PRF. $\blacksquare$

---

### Problem 4.2: Analyzing CPA Security
Let $F$ be a PRF and $G$ be a PRG. Analyze the CPA security of the following SKE schemes.

##### A. Scheme $\Pi_a$
* **Definition:** Given $m \in \{0, 1\}^{n+1}$, pick $r \in_R \{0, 1\}^n$, output $(r, G(r) \oplus m)$.
* **Verdict:** **Not CPA-secure** (Not even 1-time secure).

**Proof:**
Consider the 1-time adversary $\mathcal{A}$:
1.  $\mathcal{A}$ sends arbitrary messages $m^*_0, m^*_1$.
2.  Challenger returns $c^* = (r^*, z^*)$.
3.  Since $r^*$ is given in the clear, $\mathcal{A}$ computes the pad $p = G(r^*)$.
4.  $\mathcal{A}$ checks if $z^* \oplus p = m^*_0$. If true, return 0; else return 1.
This attack works with probability 1.

##### B. Scheme $\Pi_b$
* **Definition:** Given $m \in \{0, 1\}^n$, pick $K \in_R \{0, 1\}^\lambda$, output $F_K(0^n) \oplus m$.
* **Verdict:** **Not CPA-secure**.

**Proof:**
The padding term $P = F_K(0^n)$ depends only on the key, not the message or a nonce. It is constant across encryptions.
Consider the adversary $\mathcal{A}$:
1.  **Query:** Ask for encryption of $m = 0^n$.
    Challenger returns $c = F_K(0^n) \oplus 0^n = F_K(0^n)$.
    $\mathcal{A}$ now knows the secret pad $P = c$.
2.  **Challenge:** Send $m^*_0, m^*_1$.
    Challenger returns $c^* = P \oplus m^*_b$.
3.  **Decrypt:** $\mathcal{A}$ computes $m = c^* \oplus P$. If $m = m^*_0$, return 0.
This works with probability 1.

##### C. Scheme $\Pi_c$
* **Definition:** Given $m || b_m \in \{0, 1\}^{2n}$, pick $r \in_R \{0, 1\}^n$, output $(r, F_K(r) \oplus m, F_K(r+1) \oplus b_m)$.
* **Verdict:** **CPA-secure**.

**Proof:**
This scheme is exactly **Counter Mode (CTR)** encryption with a message length of 2 blocks.
* The nonce is $r$.
* Block 1 is encrypted with pad $F_K(r)$.
* Block 2 is encrypted with pad $F_K(r+1)$.
Since we proved in **Theorem 4.2** that CTR mode is CPA-secure for any polynomial length (VIL), this specific instance is also secure.

---

### Problem 4.3: Circular Security
**Definition:** $\Pi$ is **circularly secure** if $Game^{CPA} \approx_c Game^{CIRC}$, where $Game^{CIRC}$ gives the adversary $y_0 = Enc(K, K)$ at the start.

##### Prove/Disprove: CPA implies Circular Security
**Verdict:** **False.**

**Counter-Example Proof:**
Let $\Pi' = (Enc', Dec')$ be a standard CPA-secure scheme. We construct a modified scheme $\Pi$ that is CPA-secure but fails circular security.
Define $\Pi$ as:
$$
Enc(K, m) = \begin{cases} K & \text{if } m = K \\ Enc'(K, m) & \text{if } m \neq K \end{cases}
$$

**1. $\Pi$ is CPA-Secure:**
In the standard CPA game, the adversary does not know $K$. The probability that the adversary accidentally queries $m = K$ is at most $q/2^\lambda$ (negligible).
Conditioned on the event that $m \neq K$ for all queries, $\Pi$ behaves exactly like $\Pi'$, which is CPA-secure.
$$SD(Game^{CPA}_{\Pi}, Game^{CPA}_{\Pi'}) \le \Pr[\text{query } K] \le negl(\lambda)$$

**2. $\Pi$ is Not Circularly Secure:**
In the $Game^{CIRC}$, the challenger explicitly sends $y_0 = Enc(K, K)$.
By our definition, $Enc(K, K) = K$.
The adversary receives the secret key $K$ in plaintext immediately. They can use this key to decrypt the challenge ciphertext and win with probability 1.

---

### Problem 4.4: Domain Shrinking for PRPs
**Context:** We have a PRP $P : \mathcal{K} \times \mathcal{X} \to \mathcal{X}$. We construct $P' : \mathcal{K} \times \mathcal{X}' \to \mathcal{X}'$ (where $\mathcal{X}' \subseteq \mathcal{X}$) via **rejection sampling**:
Compute $y = P(K, x)$. While $y \notin \mathcal{X}'$, set $y = P(K, y)$. Output $y$.

##### Part 1: Efficiency
Let $t = |\mathcal{X}| / |\mathcal{X}'|$.
The process of finding a value inside $\mathcal{X}'$ is a Bernoulli trial sequence with success probability $p = 1/t$.
The number of iterations $I$ follows a geometric distribution.
**Expected evaluations:** $E[I] = \frac{1}{p} = t$.
**Condition for Efficiency:** The evaluation is polynomial-time if $t$ is polynomial in the bit-length $n$.
$$\frac{|\mathcal{X}|}{|\mathcal{X}'|} \le poly(n)$$

##### Part 2: Security Proof (Induction)
We prove $P'$ is a PRP by induction on the size difference $N = |\mathcal{X}| - |\mathcal{X}'|$.

**Base Case ($N=0$):**
$\mathcal{X} = \mathcal{X}'$. Then $P' = P$, which is a PRP by definition.

**Inductive Step:**
Assume the construction holds for any domain size difference $k < N$.
Consider the target domain $\mathcal{X}'$ such that $|\mathcal{X}| - |\mathcal{X}'| = N$.
Let $b_x \in \mathcal{X} \setminus \mathcal{X}'$ be one specific element outside the target set.
Define an intermediate set $\tilde{\mathcal{X}} = \mathcal{X}' \cup \{b_x\}$.
Let $\tilde{P}$ be the rejection-sampling permutation on $\tilde{\mathcal{X}}$. By the inductive hypothesis (since $|\mathcal{X}| - |\tilde{\mathcal{X}}| = N-1$), $\tilde{P}$ is a PRP.

Now, consider our target $P'$ (on $\mathcal{X}'$). It effectively runs $\tilde{P}$ and rejects only the single value $b_x$.
**Reduction:**
Suppose an adversary $\mathcal{A}$ breaks $P'$ on $\mathcal{X}'$. We can build an adversary $\mathcal{B}$ for $\tilde{P}$ on $\tilde{\mathcal{X}}$.
$\mathcal{B}$ simply runs $\mathcal{A}$. Note that $\mathcal{A}$ never queries $b_x$ (it's outside $\mathcal{X}'$).
However, the random permutation on $\tilde{\mathcal{X}}$ might map an element $x \in \mathcal{X}'$ to $b_x$. The random permutation on $\mathcal{X}'$ will never do this.
The statistical distance between a random permutation on $\mathcal{X}'$ and one on $\tilde{\mathcal{X}}$ (conditioned on outputs being in $\mathcal{X}'$) is negligible ($\approx 1/|\mathcal{X}'|$).
Thus, if $\tilde{P}$ is secure, $P'$ is secure.

---

### Problem 4.5: \$CPA vs CPA Security
**Definition:** SKE is **\$CPA-secure** if $Enc(K, m) \approx_c U$ (indistinguishable from random bits).

##### Prove/Disprove: CPA is equivalent to \$CPA
**Verdict:** **False.**

**Counter-Example Proof:**
Let $\Pi'$ be a \$CPA-secure scheme. Define $\Pi$ as:
$$Enc(K, m) = 0 \,||\, Enc'(K, m)$$

**1. $\Pi$ is CPA-Secure:**
The prefix '0' is deterministic and provides no information about $m$. If an adversary can distinguish $Enc(m_0)$ from $Enc(m_1)$ in $\Pi$, they rely on the suffix. This implies they can distinguish encryptions in $\Pi'$, which contradicts the hypothesis.

**2. $\Pi$ is NOT \$CPA-Secure:**
We distinguish real encryptions from random strings.
* **Adversary:** Query any message $m$. Check the first bit of the ciphertext $c$.
* **Real World ($b=0$):** $c = 0 || \dots$. First bit is always **0**.
* **Random World ($b=1$):** $c \in_R \{0, 1\}^{|c|}$. First bit is **0** or **1** with prob 0.5.
* **Advantage:** $|1 - 0.5| = 0.5$.

---

### Problem 4.6: Semantic Security for Random Messages (SSRM)
**Definition:** $\Pi$ is **SSRM** if $Enc(K, m_0) \approx_c Enc(K, m_1)$ where $m_0, m_1$ are sampled randomly by the challenger ($m \in_R \mathcal{M}$).

##### 1. Constructing Standard Security from SSRM
Given SSRM scheme $\Pi$, construct $\Pi^*$:
$$Enc^*(K, m) = (Enc(K, r), m \oplus r) \quad \text{where } r \in_R \{0,1\}^n$$

**Proof:**
By contradiction, suppose adversary $\mathcal{A}^*$ breaks $\Pi^*$. We build $\mathcal{A}$ to break SSRM of $\Pi$.
1.  $\mathcal{A}^*$ sends challenge $m^*_0, m^*_1$.
2.  $\mathcal{A}$ (playing SSRM) asks its challenger for a challenge.
    The SSRM challenger picks random $r_0, r_1$ and a bit $\beta$, returning $c = Enc(K, r_\beta)$.
3.  $\mathcal{A}$ picks a random bit $b'$ and constructs a ciphertext for $\mathcal{A}^*$:
    $$c^* = (c, m^*_{b'} \oplus r_\beta)$$
    *(Note: $\mathcal{A}$ doesn't know $r_\beta$, but constructs the tuple. Wait, the reduction requires $\mathcal{A}$ to construct a valid ciphertext for $\Pi^*$. The ciphertext is $(Enc(r), m \oplus r)$. $\mathcal{A}$ has $Enc(r_\beta)$. But $\mathcal{A}$ doesn't know $r_\beta$, so it cannot compute $m \oplus r_\beta$.)*

*Correction based on source text logic:*
The source text uses the fact that if $\beta$ (the SSRM random bit) matches $b$ (the adversary's choice), the reduction works.
Actually, the construction relies on the One-Time Pad property. $r$ is random. $m \oplus r$ is perfectly secure if $r$ is unknown. $Enc(K, r)$ hides $r$ because $\Pi$ is SSRM-secure (it hides random messages). If $Enc(K, r)$ doesn't leak $r$, then $r$ is effectively random to the adversary, so $m \oplus r$ is secure.

##### 2. SSRM does not imply Standard Security
**Counter-Example Proof:**
Let $\Pi'$ be a standard secure scheme. Define $\Pi$ as:
$$
Enc(K, m) = \begin{cases} K & \text{if } m = 0^n \\ Enc'(K, m) & \text{if } m \neq 0^n \end{cases}
$$

**1. $\Pi$ is SSRM:**
In the SSRM game, the challenger picks $m_0, m_1$ uniformly at random.
The probability that either message is $0^n$ is $2 \cdot 2^{-n}$, which is negligible.
Conditioned on $m \neq 0^n$, the scheme is secure.

**2. $\Pi$ is Not Standard CPA-Secure:**
The adversary *chooses* the messages.
* Adversary chooses $m_0 = 0^n, m_1 = 1^n$.
* If $b=0$, ciphertext is $K$.
* Adversary decrypts trial ciphertexts or recognizes the key structure immediately.