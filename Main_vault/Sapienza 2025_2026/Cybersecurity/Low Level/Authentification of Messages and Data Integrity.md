![[4.pdf]]

# Definition
**Integrity** : Trust in the unaltered state of information
**Data Integrity** : Ensures the content stays unchanged
**Origin Integrity (authenticity)** : Ensures the sender is legitimate

### Summary

- Integrity = unaltered, trustworthy data and source
- Critical in: healthcare, finance, law, journalism etc.
- Foundation for trust in digital systems
- Before any algorithm, the need for integrity is universal
- Integrity is perhaps the unique information security requirement that is always desired
# An integrity mechanism : MAC
![[2025-10-09-152117_hyprshot.png]]

### Definitions
- Authentication algorithm - $A$
- Verification algorithm - $V$ (“accept”/”reject”)
- Authentication key – $k$
- Message space (usually binary strings)
- Every message between Alice and Bob is a pair $(m, A_{k}(m))$
- $A_{k}(m)$ is called the authentication tag of $m$

- Requirement – $V_{k}(m,A_{k}(m)) = \text{accept}$
- The authentication algorithm is called MAC (Message Authentication Code)
- $A_{k}(m)$is frequently denoted $\text{MAC}_k(m)$
- Verification is by executing authentication on m and comparing with $\text{MAC}_k(m)$

>[!NOTE]  ABOUT 1:1
>More than a mere design choice, a MAC cannot assign a unique tag to each
message (i.e., be one-to-one) because this would require the set of possible
messages and the set of MAC values (tags) to have the same cardinality,
regardless of the tag length
>
>- Message space is vastly larger—potentially infinite—whereas tag
space is finite by design. [Pigeonhole principle](https://en.wikipedia.org/wiki/Pigeonhole_principle) : you cannot assign
unique tags from a finite set to an unbounded set of messages
> - There are additional reasons to prefer short,
fixed-length tags, including efficiency, ease of
implementation and constant-time verification

### Adversary’s goal
To produce a message–tag pair $(m, \text{MAC}_{k}(m))$ such that the verification function returns “accept”, i.e., $V_{k}(m,A_{k}(m)) = \text{accept}$

An adversary capable of controlling the communication channel
(e.g., a man-in-the-middle) can easily compromise data
integrity—i.e., alter the message during transmission—but cannot
ensure origin integrity (authenticity) without knowledge of the secret
key

In the standard threat model, the adversary is assumed to know
everything except the secret key k

# Birthday paradox and attack
The birthday paradox is the surprising probability result that in a group of just 23 people, there is a greater than 50% chance that at least two people share the same birthday

**Proof** :
Probability that n people have all different birthdays : 
$$P(n) = \prod_{k = 0}^{n-1}\left(\frac{365 -k}{365} \right)$$
Thus the Probability that at least two people share the same birthday is : $$1 - P(23) = 0.5073$$
### The Birthday Paradox in numbers
![[2025-10-13-141013_hyprshot.png]]
- Hashing can be seen as mapping people to birthdays, where the range size is 365
- After inserting the 23rd person into the group the probability of collision exceeds 50%

### The Birthday Paradox for length $m$ of range

- Assume the range is defined by an m-bit output space ($2^{m}$ digest)
- Then the probability that at least two message collide exceeds 50% when $k \approx 1.177 \cdot 2^{\frac{m}{2}}$ is the number of randomly chosen hashed messages being mapped into the range
- *The bound only depends on the range size*, but domain should have **at least $k$ element**
### Birthday attack
- Let's consider a cryptographic hash function that output $m$-bit values
- A *birthday attack* finds two distinct colliding inputs $x \neq y$ with high probability after hashing roughly $k \approx 1.177 \cdot 2^{\frac{m}{2}}$ inputs (Note that brute-force bound is $2^m$)
- In practice, 50% probability is exceeded after roughly $\sqrt{ 2^{m} }$ attempts

##### Procedure
```python
# Assume H known
map = ∅
while True:
	m = generate_random_message()
	h = H(m)
	if h in hash_map:
		return (map[h], m)
	hash_map[h] = m
```


> [!NOTE] Remarks
> - Expected number of iteration is $\approx 2^{ \frac{m}{2} }$ 
> - Attacker can't control colliding messages
> - The Procedure can't be used for finding an element colliding with a given message
> - Reduce iteration (from $2^{m}$ to $\approx 2^{ \frac{m}{2} }$)
> - Digest lengths of around 160 bits are deprecated, as they yield an expected effort of $2^{80}$ iterations, a threshold that is *now* feasible  

**note that**
$2^{80} \ = 1,208,925,819,614,629,174,706,176 \approx 10^{25}$
$2^{160} = 1,461,501,637,330,902,918,203,684,832,716,283,019,655,932,542,976 \approx 10^{49}$

# A hash exemple SHA-1 
### Overview

- Developed by: NIST & NSA, published in 1995 ([FIPS](https://en.wikipedia.org/wiki/Federal_Information_Processing_Standards) 180-1)
- Similar to MD4 and MD5
- $\#\text{message} < 2^{64}$
- $\#\text{digest} = 160$
- Original message is padded
![[2025-10-13-143316_hyprshot.png]]

The 160-bit message digest consists of five 32-bit words: A, B, C, D, and E.
- Before first stage: 
	- $A = 67452301_{16}$, 
	- $B = efcdab89_{16}$, 
	- $C = 98badcfe_{16}$, 
	- $D = 10325476_{16}$, 
	- $E = c3d2e1f0_{16}$.
- After last stage A|B|C|D|E is message digest
### High level scheme
![[2025-10-13-143856_hyprshot.png]]

### Processing one block
Block (512 bit, 5 32-bits words)
- 80 rounds: each round modifies the buffer (A,B,C,D,E)
**Round**: $(A,B,C,D,E) \leftarrow (E + f(t, B, C, D) + (A<<5) + Wt + Kt), A, (B<<30), C, D)$
- $t$ number of round, $<<$ denotes left shift
- $f(t,B,C,D)$ is a complicate nonlinear function
- $W_{t}$ is a 32-bit word obtained by expanding original words into 80 words (using shift and ex-or)
- $K_{t}$ constants

$$\begin{align}
K_{t} &= \lfloor 2^{30}\sqrt{ 2 } \rfloor   = 5a827999_{16} \ \ \ (0 \leq t \leq 19) \\
K_{t} &= \lfloor 2^{30}\sqrt{ 3 } \rfloor   = 6ed9eba1_{16} \ \ \ (20 \leq t \leq 39) \\
K_{t} &= \lfloor 2^{30}\sqrt{ 5 } \rfloor   = 8f1bbcdc_{16} \ \ \ (40 \leq t \leq 59) \\
K_{t} &= \lfloor 2^{30}\sqrt{ 10 } \rfloor   = ca62c1d6_{16} \ \ \ (60 \leq t \leq 79)
\end{align}$$
**Function f**
$$\begin{align}
f(t,B,C,D) = \ & (B \wedge C) \lor (B\land D) \ \ \ \ \ \ \ \ \ \ \ \  \ \ \ \ \ \ \ \ (0 \leq t \leq 19) \\ 
	 & B \oplus C\oplus D \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ (20 \leq t \leq 39)\\
     & (B \wedge C) \lor (B\land D) \lor (C \land D) \ \ (40 \leq t \leq 59)\\
	 & B \oplus  C \oplus D \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ (60 \leq t \leq 79)

\end{align}$$
**Words $w_{t}$**
$w_{0} \dots w_{15}$ are the original 512 bits
for $16 \leq t \leq 79$
- $w_{t} = (w_{t-3} + w_{t-8} + w_{t-14} + w_{t-16}) <<= 1$
- "<<=" denotes left bit rotation

##### Round t
![[2025-10-13-145501_hyprshot.png]]

### Stage
![[2025-10-13-150129_hyprshot.png]]

Summary on SHA-1
1. Pad initial message: final length must be $\equiv 448 \pmod{512}$ 
2. Last 64 bit are used to denote the message length
3. Initialize buffer of 5 words (160-bit) (A,B,C,D,E) (67452301, efcdab89, 98badcfe, 10325476, c3d2e1f0)
4. Process first block of 16 words (512 bits):
	1. expand the input block to obtain 80 words block $W_{0},W_1,W_2,…,W_{79}$ (ex-or and shift on the given 512 bits)
	2. initialize buffer (A,B,C,D,E)
	3. update the buffer (A,B,C,D,E): execute 80 rounds
	4. each round transforms the buffer
	5. the final value of buffer (H1 H2 H3 H4 H5) is the result
5. Repeat for following blocks using initial buffer (A+H1, B+H2,…)

> [!WARNING] SHA-1 is getting sunseted because it isn't secure enough anymore

