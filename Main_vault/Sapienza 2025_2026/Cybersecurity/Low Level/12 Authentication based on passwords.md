**Tags:** #Cybersecurity #LowLevel #Authentication #Passwords #EKE #Attacks #Salting

---

# 1. Password Fundamentals

## Definition
Password-based authentication relies on the factor "What you know". It pairs a **username** (identification) with a **password** (authentication).

## Characteristics
* **Entropy Constraints:**
    * Humans must remember passwords, limiting them to typeable characters and reasonable lengths.
    * Consequently, a password has significantly less entropy (randomness) than a cryptographic key of the same length (which can use any bit combination).
* **Leet Speak:**
    * Users often substitute characters to create "stronger" passwords that remain memorable (e.g., `h3110` for "hello", `p@55w0rd` for "password").
    * **Security Impact:** Attackers are aware of these substitutions. Rule-based dictionary attacks automatically check common leet speak variations, nullifying the perceived security benefit.

---

# 2. Attack Models

## A. Online Attack
* **Scenario:** The attacker interacts with the live system (e.g., a website login page).
* **Constraint:** The attacker is limited by the system's response time and defense mechanisms.
* **Defense:**
    * **Account Lockout:** Disable account after $N$ failed attempts.
    * **Rate Limiting:** Slow down responses for repeated failures.
    * **CAPTCHA:** Prevent automated bot attacks.
    * **[[10 Authentication#2 Authentication Factors|MFA (Multi-Factor Authentication)]]:** Require a second factor.

## B. Offline Attack
* **Scenario:** The attacker has stolen the password database (usually containing hashes).
* **Constraint:** Limited only by the attacker's hardware (CPU/GPU power). No network or system limits.
* **Impact:** Attackers can guess billions of passwords per second.
* **Defense:**
    * **Salting:** Prevents precomputed table attacks.
    * **Slow Hashing:** Computationally expensive functions (PBKDF2, bcrypt, Argon2) to reduce guessing speed.

---

# 3. Dictionary and Brute Force Attacks

## The Dictionary Attack
* **Method:** Instead of trying every combination of characters ($a, b, c...$), the attacker tries words from a list of common passwords (dictionary).
* **Reason:** Users rarely choose random strings; they choose words, names, or simple patterns.

## The Salt
A **Salt** is a unique, random string added to the password before hashing.
$$
\text{Stored Value} = (\text{Salt}, \text{Hash}(\text{Salt} || \text{Password}))
$$

**Why use Salt?**
1.  **Prevents Duplicate Hashes:** Without salt, two users with the password "123456" would have the same hash. Salt ensures their hashes are different.
2.  **Defeats Precomputed Attacks:** Prevents the use of **Rainbow Tables** (huge databases of precomputed hash chains).
3.  **Forces Per-User Cracking:** An attacker cannot run a single dictionary attack against the whole database. They must hash the entire dictionary *separately* for every single user (using that user's specific salt).

**What Salt Does NOT Do:**
* It does *not* increase the search space for a specific target. If an attacker targets Alice, they just grab her salt and brute-force her password. The effort to crack *one* password remains the same; the effort to crack *all* passwords increases linearly with the number of users.

---

# 4. Encrypted Key Exchange (EKE)

Standard Diffie-Hellman (DH) is vulnerable to Man-in-the-Middle (MITM) attacks because it lacks authentication. **EKE** (Bellovin & Merritt, 1992) adds authentication to DH using a password, without exposing the password to offline dictionary attacks if the protocol is observed.

## The Protocol Concept
EKE encrypts the Diffie-Hellman public parameters ($g^a$ and $g^b$) using a symmetric key derived from the user's password ($P$).

1.  **Alice:**
    * Generates random $a$.
    * Computes $g^a \pmod p$.
    * Encrypts $g^a$ using her password $P$: **$P\{g^a\}$**.
    * Sends to Bob: $A, P\{g^a\}$.

2.  **Bob:**
    * Uses the stored password $P$ to decrypt the message and recover $g^a$.
    * Generates random $b$.
    * Computes $g^b \pmod p$.
    * Encrypts $g^b$ using $P$: **$P\{g^b\}$**.
    * Sends to Alice.

3.  **Session Key:**
    * Both calculate the shared secret $K = g^{ab} \pmod p$.
    * They perform a challenge-response (e.g., encrypting a nonce) using $K$ to verify they established the same key.

## Security Logic
* **Passive Attacker:** Sees $P\{g^a\}$ and $P\{g^b\}$. To find the session key, they need $P$.
* **Active Attacker (MITM):** Can try to decrypt $P\{g^a\}$ with a guessed password $P'$.
    * If $P'$ is wrong, they get a random number $X$ instead of $g^a$.
    * They send $X$ to Bob.
    * Bob computes $K' = X^b$.
    * The authentication step (Challenge/Response) will fail because the keys don't match.
    * The attacker learns nothing about the correctness of $P'$ until the very end, preventing offline brute-force optimization.

## Augmented EKE
A variant where the server does not store the plaintext password (or equivalent key), but rather a verification value (like $g^P$). This prevents an attacker who steals the server database from masquerading as the user.

---

# 5. Authentication Outcomes

Protocols can result in two different states:

| Outcome | Description | Use Case |
| :--- | :--- | :--- |
| **Auth + Session Key** | The user is authenticated, and a high-entropy symmetric key is established to encrypt the subsequent session. | **TLS, SSH, VPNs** (Data protection required). |
| **Auth Only** | The user is authenticated, but no encrypted channel is established. Communication continues in plaintext or ends. | **Physical Access** (Door lock), **local login** (after auth, OS handles security). |