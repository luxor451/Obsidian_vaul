
![[5.pdf]]
# Introduction
CHF needs to be
- collision resistance
- irreversibility

>[!NOTE] Remark
>hard means computationally infeasible
>- infeasible means that computation would take too much time (years, or more) 
>- birthday bound allows saying that "short" fingerprints are feasible
>	- today short means 160 bits or less 
>	- SHA-1 was a cryptographic function, until 2014

### Collision resistance
- A hash function $h: D \rightarrow R$ is called **weakly collision resistance** for $x \in D$ if it is *NP-hard* to find $x'\neq x$ but $h(x) = h(x')$
- A hash function $h: D \rightarrow R$ is called **strongly collision resistant** if it is *NP-hard* to find $x, x’$ such that $x’\neq x$ but $h(x)=h(x’)$

### Difference between **weak** and **strong** 
- The strong form requests that it is hard to find messages that collide
	- there is no input
	- it is pure existence, i.e., no other condition is put
- the weak form has one message in input
	- it requests to find a colliding message
- **strong** and **weak** terms used because strong $\implies$ weak
##### Proof 
we show $\neg weak \implies \neg strong$ (equivalent)
given $h$, suppose there is an polynomial algorithm $A_{h} : A_{h}(x) = x'$ such that $h(x) = h(x')$ 
we construct a polynomial algorithm $B_{h}$ such that $B_{h}() = (x,x')$ such that $h(x) = h(x')$ :
1. Arbitrarily choose x
2. return $(x,A_{h}(x))$
### Requirements of CHF
- (strong) collision resistance
- non-reversibility it is hard to find, given $f(M)$, $f^{-1}(f(M))$

> [!NOTE] Remark
> Note that for no function a math proof that it is cryptographic hashing has been found, but there are several functions conjectured to be cryptographic


### Terminological note
In modern cryptography terminology has been updated
- **Preimage resistance**: non-reversibility
- **Second preimage resistance**: what was traditionally called weak collision resistance
- **Collision resistance**: what was traditionally called strong collision resistance

### Summary on cryptographic hashing functions
- Deterministic
- Efficient
- Preimage resistance
- Second preimage resistance
- Collision resistance
- Avalanche effect
- Fixed output length

##### Security of cryptographic hashing functions

- Birthday bound holds even for cryptographic hashing functions
	- Same number of collisions, just better distributed and random
	- Birthday attack in theory is possible
- The improvement is the (much) greater difficulty in finding collisions
- Preimage resistance offers several security applications
	- Privacy, authentication, blockchain etc.
	- Used for deriving keys from passwords

# Design and Construction

### Recap on cryptographic hashing functions

- Maps input of any length to a fixed-size digest
- Properties :
	- Preimage resistance
	- Second preimage resistance
	- Collision resistance
- Applications :
	- Integrity
	- Digital signatures
	- Password hashing

### Design goals
- Output should be:
	- Unpredictable
	- Uniformly distributed
	- Sensitive to input changes (avalanche effect)
- Efficient computation
- Secure against known attack strategies

### Historical design
- MD5 – Broken, deprecated
- SHA-1 – Vulnerable to collisions
- SHA-2 (it's a family) – Still widely used
- SHA-3 (Keccak, a family) – Sponge-based, more secure

| Name  |                                                                                                                       Basic Info                                                                                                                       |
| :---: | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|  MD5  |                                       Once widely used, but now recognized as insecure for typical security purposes due to known collision attacks. Avoid for new applications requiring cryptographic security                                       |
| SHA-1 |                    While not as easily broken as MD5, practical collision attacks have been demonstrated. It's largely considered unsuitable for new applications requiring strong collision resistance (e.g., digital signatures)                     |
| SHA-2 |                                This is a family of hash functions (e.g., SHA-256, SHA-512). It's currently considered secure for most applications and is widely implemented in various security protocols and systems                                 |
| SHA-3 | Also known as Keccak, this is a family of hash functions that uses a  different underlying design called a sponge construction. It was developed to offer an alternative to SHA-2 with a distinct cryptographic foundation, enhancing overall security |

### Merkle–Damgård construction
- Processes message in blocks
- Uses a compression function
- Adds padding
- Final hash = last output
- Strength: simple and modular
- Used in the design of SHA-1 and SHA-2 (and others)
- Weakness: length extension attacks (later)

##### Diagram
![[2025-10-15-135748_hyprshot.png]]
$H: D \rightarrow R \ \ (\text{fixed sets}, \text{typically} \{0,1\}^n and \{0,1\}^m )$

##### Function H
- It is called Compression Function
- Must behave like a pseudo-random function
- Common structures
- Davies-Meyer (block cipher-based)
- Requirements
	- Non-linearity
	- Diffusion
	- Confusion
##### Example
**Davies-Meyer** :
- Denote by $O_{i}$ the output of block $i$
- $H = E_{M_{i}} (O_{i-1}) \oplus O_{i-1}$
- $E$ is a block cipher, $M_{i}$ (message block) is used as a key

 [[Authentification of Messages and Data Integrity#A hash exemple SHA-1|SHA-1/SHA-2]]

  >[!NOTE] Remarks
  >• The seed is usually constant
  >- Typically, padding (including text length of original message) is used to ensure a multiple of $n$ (size of domain of $H$)
  >- Claim: *if the basic function $H$ is collision resistant, then so is its extension*
  >- Input message length should be arbitrary. In practice it is usually up to $2^{64}$, which is good enough for all practical purposes 
  >- Block length is usually 512 bits
  >- Output length should be larger than 160 bits to prevent birthday attacks
  

### Sponge construction
- The Sponge Construction (used by SHA-3/Keccak) processes data like a sponge: *absorbing input* and *squeezing out* the hash
- It uses an internal state (rate + capacity) and a permutation function $f$. Data is absorbed into the rate $r$ via $\oplus$, followed by a transformation by $f$. Output is then squeezed from $r$, with $f$ applied between extractions. The capacity portion remains hidden for security
- Advantages :
	- Resistant to Length Extension Attacks
	- Flexible: Creates fixed-length hashes (e.g., SHA3-256) and eXtendable Output Functions (XOFs)

##### SHA-3 overview
- Based on Keccak algorithm
	- *not affected by length extension attack*
- 1600-bit internal state
- Permutation-based rounds
- Higher diffusion per round
- Post-quantum suitability
- Available
##### Keccak functions
- SHA-3 family
	- SHA3-224, SHA3-256, SHA3-384, SHA3-512
- SHAKE128, SHAKE256
	- they are XOFs (eXtendable Output Functions)
	- XOFs take the desired length as input sometimes faster than SHA-3

### Most used today
- It is SHA-256
- Member of family SHA-2, with output of 256 bits
	- SHA-224, SHA-384, SHA-512 and truncated versions
- Still secure, fast, easy
- **Based on Merkle-Damgård construction**
- Standardized: it is approved by FIPS 180-4 and recommended by many international security standards

# CHF : applications

- In addition to integrity (currently focused and non-finished)
- *Digital signatures (later)*
- *Passwords (later)*
- Pseudo-random number generators
- Key derivation functions (KDF)
- Blockchain
- Privacy
- Disk optimization
- *PRNG (later)*
### Key Derivation Functions

-  Cryptographic algorithms used to generate one or more secret keys from a shared secret or password
-  Their main purpose is to transform input data that may have low entropy or structure into cryptographically strong keys suitable for encryption or other security mechanisms
##### Definition
Algorithm that takes as input
- a base secret (e.g., a password, a key)
- optionally a salt (random value)
- possibly other context-specific parameters
It outputs
- one or more pseudo random keys of desired length
Often hash-based (not necessarily)

**very popular**: get a secret key from a password
##### Properties
- Strengthen weak inputs
- Generate multiple keys from a single master (key separation)
- Increase entropy using salting and iteration
- Make brute-force attacks more expensive

### Blockchain
- Complex and articulated subject deserving more space
- Blockchain is a distributed database or ledger that maintains a continuously growing list of records, called blocks, which are securely linked together using cryptography
- Each block typically contains
	- A list of transactions or records
	- A timestamp
	- A cryptographic hash of the previous block
	- A nonce (in proof-of-work systems)

##### The blockchain is immutable
- Each block’s hash depends on the content of the previous block, forming an unbroken chain: if any block is altered, all subsequent blocks become invalid unless recomputed
- Computation is carried out through Proof of Work (there are other methods)
	-  PoW consists in solving a difficult computational puzzle
- How it works (simplified) :
	1. A transaction is broadcast to the network
	2. Nodes group pending transactions into a block
	3. The block is validated (by PoW)
	4. The validated block is added to the blockchain
	5. All nodes update their copy of the chain

##### PoW
Generic case
- Find a nonce such that: Hash(block data || nonce) < Target
- nonce = number used once
- This is called a hash-based puzzle. It has no shortcut, brute-force is needed
- Target is known (and slowly updated) with several mechanisms
- People (miners) work trying to solve the puzzle
### Hashing functions in privacy

|         Use Case          |                        How hashing support privacy                         |
| :-----------------------: | :------------------------------------------------------------------------: |
|  **Data anonymization**   | Hashes can obscure identifiers (e.g. names,<br>emails) before data sharing |
|   **Password storage**    |  Stores only a hashed version of the password,<br>not the password itself  |
| **Zero-Knowledge Proofs** |        Hashes are used to commit to a value without<br>revealing it        |
|  **Private blockchains**  |    Transactions may include hashed identifiers to<br>pseudonymize users    |
```C
(a++)++
(++a)+(++b
``