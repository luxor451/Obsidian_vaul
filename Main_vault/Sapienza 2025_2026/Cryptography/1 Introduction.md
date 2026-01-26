# Introduction to Modern Cryptography

**Tags:** #Cryptography #Introduction #Confidentiality #Integrity #KerckhoffsPrinciple #SymmetricVsAsymmetric #SecurityDefinitions 
**Topic:** 1.1 Definitions, assumptions, and notation

---

## 1. What is Cryptography?

**Cryptography** is the branch of computer science that practices and studies techniques for secure communication in the presence of adversarial behavior.

It focuses on two primary goals:

### A. Confidential Communication
The property of making the original message **impossible to read** even if an outsider finds out the ciphertext (the encrypted message).

> **Figure 1.1: Loss of Confidentiality**
> Without encryption, "Eve" (the evildoer) can read the message $m$ sent by Alice.

![[Pasted image 20260124024559.png]]
### B. Message Integrity
The capability of ensuring that the original message has **not been altered** even if an outsider intercepts the ciphertext.

> **Figure 1.2: Loss of Integrity**
> Without encryption, Eve may intercept message $m$ and forward an altered message $m'$ to Bob.

![[Pasted image 20260124024614.png]]

---

## 2. The Evolution of Cryptography

* **Pre-1950s (The Art):** Cryptography was considered an "art for geniuses." It relied on individuals creating puzzles for other geniuses to solve.
* **Modern Days (The Science):** Cryptography has become a **mathematical science** with precise definitions and rigorous proofs.

### Types of Mathematical Proofs

In modern cryptography, tools and proofs are separated into two categories:

| Proof Type | Description | Resources |
| :--- | :--- | :--- |
| **Unconditional Proofs** | Proofs made without assumptions. They focus on showing something is possible regardless of efficiency. | Theoretically **infinite** resources. |
| **Conditional Proofs** | Uses real-world assumptions believed to be true to prove real-world results. | **Polynomial** (efficient) time. |

### The $P \neq NP$ Assumption
Conditional proofs often rely on the assumption that $P \neq NP$ (problems exist that are verifiable in polynomial time but *not* solvable in polynomial time).

* **Example:** Many cryptosystems assume [[5 Number theory in cryptography|Prime Factorization]] is hard to solve but easy to verify.  
* This implies assuming $FACTORING \in NP \cap P$.
* By making the "solution" (the key) hard to find but easy to verify, we create secure authentication phases.

> [!abstract] Theorem 1.1
> Every cryptosystem based on prime factorization is "secure" if $FACTORING \notin P$.
>
> **Proof (Sketch):**
> We prove this by **contrapositive**.
> 1. Suppose there is a cryptosystem $\Pi$ that is *not* secure.
> 2. This means there exists a polynomial-time algorithm $\mathcal{A}$ capable of "breaking" $\Pi$.
> 3. For each number $n \in \mathbb{N}$, we can forge a message $m$ based on $n$.
> 4. We use the output $\mathcal{A}(m)$ to find the prime factors of $n$.
> 5. **Conclusion:** If the system is breakable, then $FACTORING \in P$. Therefore, if Factoring is *not* in P, the system is secure.

---

## 3. Types of Cryptosystems

The goals of confidentiality and integrity are discussed under two main architectures:

1.  **[[4 Symmetric-key encryption|Symmetric Cryptography]]:**
    * Both ends of the communication share a **single common key**.
    * The key is random and unknown to any outsider.
2.  **[[6 Public-key encryption|Asymmetric Cryptography]]:**
    * Both ends possess a **Key Pair** $(pk, sk)$.
    * $pk$ (Public Key): Known by everyone.
    * $sk$ (Secret Key): Known only by the owner.

---

## 4. Notation

Throughout this course, the following notation is used to define cryptographic systems:

* **$\mathcal{M}$ (Message Space):** The set of all strings of messages that can be sent between two parties.
* **$\mathcal{C}$ (Ciphertext Space):** The set of all strings of ciphertexts that can be produced by a cryptosystem.
* **$\mathcal{K}$ (Key Space):** The set of all strings of keys that can be used in a cryptographic system.
* **$\mathcal{H}$ (Hash Function Space):** The set of all hash functions that can be used by a cryptosystem.