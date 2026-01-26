![[3.pdf]]

# Introduction
### Desired properties 
- **Invertibility**: must be able to recover original plaintext
- **Avalanche effect**: small changes = big output changes
- **Non-linearity**: avoids predictability and simple math reversal
- **Key sensitivity**: small key differences yield vastly different outputs

These features strengthen resistance to cryptanalytic attacks

### Applications

- File encryption
- Disk encryption (full disk, partitions)
- Encrypted RAM/memory
- Encrypted communication packets
- Database field/row encryption
- Deployed in virtually all modern security systems

Block ciphers work on *fixed-size* blocks (e.g., 128 bits)

### Iterating 
**DES** (data encryption standard) is a symmetric block cipher using 64-bit
blocks and a 56-bit key

Iterating can be used to make DES more robust, With 3 steps of encryption you can multiply by 3 the size of the space to search, that give the illusion of a key size $56 \cdot 3 \text{ bits} = 168 \text{ bits}$  

Iterating can be used (theoretically) with other ciphers
![[2025-10-02-152217_hyprshot.png]]
- Plaintext undergoes encryption repeatedly by underlying cipher
- Ideally, each stage uses a different key
- In practice triple cipher is usually
- C = $E_{k1}(E_{k2}(E_{k1}(P)))$ *EEE mode* or
- C = $E_{k1}(D_{k2}(E_{k1}(P)))$ *EDE mode*
- EDE is more common in practice

	But sometimes only two key are used in 3DES with identical key at the beginning and the the end, using more key can be demanding (storing, computational power...)

We don't use 2DES because of MITM (Meet-in-the-Middle)

### MITM (Meet-in-the-Middle)

*Not* to be confused with Man-in-the-Middle

- Requirements (assume EE)
	- Known plaintext/ciphertext pairs
	- $2^n$ encryptions + $2^n$ decryptions (2 keys of n bit), instead of $2^{2n}$ brute-force
	- $2^n$ memory space
- **Idea**: try all possible $2^n$ encryptions of the plaintext and all possible $2^n$ decryptions of the ciphertext. Encryptions stored into a lookup table
- Check for a pair of keys that transform the plaintext in the ciphertext. Test pair on other pairs plaintext/ciphertext
- Note: the method can be applied to all block ciphers

##### Can be used (theoretically) in the general case

- MITM can be (theoretically) applied on s iterations
- If key-length is $sn$, brute-forcing the iteration doesn't require $2 \cdot sn$ attempts, but only (about) $2^{\frac{sn}{2}}$ 
- MITM needs huge memory
- For 3 or more iterations, memory becomes impractical
- Triple encryption is the realistic upper bound

### AES (Advanced Encryption Standard)
##### General information
- Symmetric block cipher (block size: 128 bits = 16 bytes)
- Key lengths: 128, 192, or 256 bits
- Approved US standard (2001)
- Finite fields algebra

##### Some math results
**Euler's totient function**
$$\varphi(n) = \# \{x < n | x \wedge n = 1\}$$
**Euler Theorem**
$$
\forall a,n \in \mathbb{N^*}, \; a \wedge n = 1 \implies a^{\varphi(n)} \equiv 1 \pmod{n}
$$
**Bézout's identity**
$$\forall a,b\in \mathbb{N^*}, a \wedge b = d \implies \exists \ x,y \in \mathbb{Z}, ax + by = d$$
**Finite fields**
$$\mathbb{Z}/n\mathbb{Z} \text{ with } n\in \mathbb{N}$$
$$\mathbb{Z}/n\mathbb{Z} = \{[0], [1], [2], …, [n-1]\}$$

- here [i] is the equivalence class of integers congruent to i (mod n)
- for brevity people often write $\mathbb{Z}/n\mathbb{Z} =\{0, 1, 2, …, n-1\}$

- quotient set commonly called ring of integers modulo n
$$(\mathbb{Z}/m\mathbb{Z})^* = \text{multiplicative group modulo m}$$

- natural numbers mod m that are relatively prime (co-prime) to m 
- $(\mathbb{Z}/m\mathbb{Z})^* \subseteq (\mathbb{Z}/m\mathbb{Z})$
- **If $p \in \mathbb{P}$ then $\mathbb{Z}/p\mathbb{Z}$ is a field**

##### Exemple
Compute $2^{200} \pmod{127}$ Using Euler's Theorem

Since 127 is prime, $\varphi(127) = 126$ and $2^{200} = 2^{74} \cdot 2^{126}$
And $2^{74} = 2^{7^{10}} \cdot 2^4$
And finally, $127 = 2^7 - 1$
Thus 
$2^{200} \pmod{127} = 2^{\varphi(127)} \cdot 2^{7^{10}} \cdot 2^4 \pmod{127} = 1  \cdot 1^{10} \cdot 16 = 16$


We can now redefined the Euler's totient function : $$\varphi(n) = \#(\mathbb{Z}/n\mathbb{Z})^*$$
##### Galois Field $GF(p^k)$ or $\mathbb{F}_{p^k}$
**Theorem**
$$\forall p\in \mathbb{P}, \forall k \in \mathbb{N}^*, \exists! \text{ Finite field } GF(p^k) \text{ with } \#GF(p^k) = p^k$$
These are the *only* finite fields with these cardinalities

**Theorem**
Let $f \in (\mathbb{Z}/p\mathbb{Z})_k [X]$. 
The finite field $\mathbb{F}_{p^k}$ can be realized as the set of degree $k-1$ polynomial over $\mathbb{Z}/p\mathbb{Z}$, with addition and multiplication done modulo f

##### Exemple
In $\mathbb{F}_{p^5}$,
Addition : bit-wise *XOR* since ($1+1 = 0$)
$$\begin{align}
X^3 + X + 1 \text{ } (0,1,0,1,1)\\
+ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \\
X^4 + X^3 + X \ \ \ \ \ \ \ \ (1,1,0,1,0)\\
-------------- \\
X^4\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ +1 \ (1,0,0,0,1)
\end{align}
$$

Multiplication :![[2025-10-02-170015_hyprshot.png]]


### Back to AES
- Symmetric block cipher
- Key lengths: 128, 192, or 256 bits
- original Rijndael supports more lengths
Rationale
- Resistance to all known attacks
- Speed and code compactness
	- good for devices with limited computing power, e.g. smart cards
- Simplicity
##### Specifications

![[2025-10-02-170543_hyprshot.png]]![[2025-10-02-170553_hyprshot.png]]

##### High level code
```C
AES(State, Key)
KeyExpansion(Key, ExpandKey)
AddRoundKey(State, ExpandKey[0])
for (i = 1; i < R; i++) do
	Round(State, ExpandKey[i]);
end;
FinalRound(State, ExpandKey[R]);
```

- 128 bits AES uses 10 rounds, no shortcuts known for 6 rounds
	- The secret key is expanded from 128 bits to 10 round keys, 128 bits each
	- Each round changes the state, then XORs the round key (for longer keys, add one round for every extra 32 bits)
- Each rounds complicates things a little
- Overall, it seems infeasible to invert without the secret key (but easy given the key)
##### One Round :
1. Substitution
2. Shift rows
3. Mix column
4. XOR round key
##### Substitution (S-Box)

- Substitution operates on every byte separately: $A_{i,j}$ <-- $A_{i,j}^{-1}$ (multiplicative inverse in $GF(2^8)$ which is highly nonlinear)
- If $A_{i,j} = 0$, don’t change A_{i,j}$
- Clearly, the substitution is invertible
##### Shift of rows
![[2025-10-02-171955_hyprshot.png]]
Clearly, this operation *is reversible*
##### Mixing Columns
- Every state is considered as a polynomial over $GF(2^8)$
- Multiply with an invertible polynomial
- $03 X^3 + 01X^2 + 01X + 02 \pmod{X^4 + 1}$
- Inv = $0BX^3 + 0DX^2 + 09X + OE$
### [Animation](https://www.youtube.com/watch?v=gP4PqVGudtg)

# Block Cipher Modes of Operation

- Block cipher operate on fixed length block
	- Often 64 or 128 bit

They are *Many* modes of operation to permit encryption of messages of *any* length
### ECB (Electronic Code Book)

![[2025-10-08-132318_hyprshot.png]]
>[!WARNING] Only for teaching purpose, never use this in production

Encrypt each plaintext block separately 

**Properties**
- Simple and efficient
- Parallel implementation possible
- Does not conceal plaintext patterns
- Active attacks are possible (plaintext can be easily manipulated by removing, repeating, or interchanging blocks)
### CBC (Cipher Block Chaining)

![[2025-10-08-132427_hyprshot.png]]

- Previous ciphertext is $\oplus$ with current plaintext
- It uses a Seed (called *initialization vector*, or *IV*) to start the process
	- Seed = 0 safe in most but NOT all cases (e.g., assume the file with salaries is sent once a month, with the same seed we can detect changes in the salaries) therefore a random seed is better

The seed can be send as clear text w/o encryption
##### Decryption
![[2025-10-08-133341_hyprshot.png]]
If an error change one bit of $C_i$ (minimal error)
- Then $m_{i+1}$ changes in a predictable way (can be exploited by a adversary)
- *But* there are unpredictable changes in $m_i$
- Solution: always use *error detecting codes* (for example CRC) to check quality of transmission

**Properties**
- Asynchronous stream cipher
- Errors in one ciphertext block don’t propagate much
	-  a one-bit change to the ciphertext causes complete corruption of the corresponding block of plaintext, and inverts the corresponding bit in the following block of plaintext
- Conceals plaintext patterns
- No parallel encryption
- Yes parallel decryption
- Plaintext cannot be easily manipulated
- Standard in many systems: SSL, IPSec ...
- Message must be padded to a multiple of the cipher block size
	- One way to handle this issue is *ciphertext stealing* 
- A plaintext can be recovered from just two adjacent blocks of ciphertext
	- Therefore, decryption can be parallelized
	- Usually a message is encrypted once, but decrypted many times

### Ciphertext stealing
- General method that allows for processing of messages that are not evenly divisible into blocks
- Without resulting in any expansion of the ciphertext
- At the cost of slightly increased complexity
- Consists of altering processing of the last two blocks of plaintext, resulting in a reordered transmission of the last two blocks of ciphertext (and no ciphertext expansion)
- Suitable for ECB and CBC
- From [NIST](http://csrc.nist.gov/groups/ST/toolkit/BCM/documents/ciphertext%20stealing%20proposal.pdf) website 

##### Encryption procedure
- Apply standard CBC encryption to all complete blocks
- If the last block is partial, process the previous full block normally
-  Use the ciphertext of the previous block to fill the partial block (ciphertext stealing)
-  *Swap the last two ciphertext blocks*
-  Transmit the ciphertext as-is (possibly non-multiple of block size)
##### Decryption procedure
- *Requires knowledge of original plaintext length*
- Swap the last two ciphertext blocks back to original order
- Decrypt with CBC as usual
- Discard the extra bytes to recover the correct final block 

![[2025-10-08-141203_hyprshot.png]]
### OFB (Output FeedBack) 
- Makes a block cipher into a synchronous stream cipher: it generates keystream blocks, which are then $\oplus$ with the plaintext blocks to get the ciphertext
- Flipping a bit in the ciphertext produces a flipped bit in the plaintext at the same location. This property allows many error correcting codes to function normally even when applied before encryption
##### Encryption
![[2025-10-08-145159_hyprshot.png]]
It uses an IV as a seed for a sequence of data block
##### Decryption
![[2025-10-08-145331_hyprshot.png]]

##### Math
Because of the symmetry of the XOR operation, encryption and decryption are exactly the same
$$
\displaylines{
C_{i} = P_{i} \oplus O_{i} \\
P_{i} = C_{i} \oplus O_{i} \\
O_{i} = E_{k}(O_{i-1}) \\
O_{0} = \text{IV}}$$
**Some Important points**
- If $E_k$ is public (known to the adversary) then initial seed must be encrypted
- If $E$ is a cryptographic function that depends on a secret key, then initial seed can be sent in clear
- Initial seed must be modified for EVERY new encryption
- Extension: it can be modified in such a way that only k bits of each keystream block are used to compute the ciphertext (k-OFB)
**Properties**
- Synchronous stream cipher
- Errors in ciphertext do not propagate
- Pre-processing is possible
- Conceals plaintext patterns
- No parallel implementation known
- Active attacks by manipulating plaintext are possible
### CTR (Counter Mode)
a.k.a *Integer Counter Mode* (ICM) and *Segmented Integer Counter* (SIC)
![[2025-10-08-151026_hyprshot.png]]
**Similar to OFB**
- There are problems in repeated use of same seed (like OFB)
- CTR vs OFB: using CTR you can decrypt the message starting from block $i$ for any $i$

### IV (Initialization Vector)

- Most modes (except ECB) require an initialization vector, or IV
	- sort of "dummy block" to kick off the process for the first real block, and to provide some randomization for the process
	- no need for the IV to be secret, in most cases, but it is important that it is never reused with the same key
- For CBC reusing an IV leaks some information about the first block of plaintext, and about any common prefix shared by the two messages
- In CBC mode, the IV must, in addition, be unpredictable at encryption time
	- there is a TLS CBC IV attack
- *For OFB and CTR, reusing an IV destroys security*

# Analysis and comparison of block modes of operations

### Recap on block modes of operations

 - There are many block operation modes (more than 4)
- Used to overcome the limitation that the same algorithm with the same key always encrypts the same plaintext block to the same ciphertext
-  Since they are many, how to make a choice?
### Methodology of analysis
Some questions are relevant for most modes
1. Can it encrypt in parallel?
2. Can it decrypt in parallel?
3. Is preprocessing useful?
4. Does it support ciphertext direct addressing?
5. What consequences for a ciphertext bit flipping?
6. Does it turn block encryption to stream encryption?*
<figure>
  <img
  src="2025-10-08-152819_hyprshot.png"
  alt=".">
  <figcaption>Responses to the Question above for the 4 Mode seen here</figcaption>
</figure>

