**Tags:** #Cybersecurity #LowLevel #InfoSec #Cryptography #Learning #History #Attacks

---

# 1. Foundations of Information Security

### The CIA Triad & Beyond
Information security is built on three core requirements, often known as the **CIA Triad**:
* **Confidentiality:** Keeping information secret from unauthorized access.
* **[[4 Authentication of messages and data integrity|Integrity]]:** Protecting information from being altered or tampered with during transmission or storage.
* **Availability:** Ensuring information and systems are accessible when needed.

**Additional Requirements:**
* **[[10 Authentication|Authentication]]:** Verifying the identity of a user or system.
* **Non-repudiation:** Ensuring a party cannot deny an action (e.g., preventing a sender from denying they sent a message).
* **Accounting:** Tracking usage and actions for auditing purposes.

### Security vs. Cryptography
It is critical to distinguish between these two fields:
* **Cryptography** traditionally deals with the secrecy of information (making text unreadable).
* **Real Security** focuses on preventing fraud. This includes handling message modifications and ensuring user authentication.
* **The Relationship:** Security often uses cryptography as a tool, but encryption rarely lives alone. Without authentication, encryption cannot guarantee security because an attacker could simply impersonate a valid user.

---

# 2. Pre-Computer Cryptography (Historical Primitives)

Before modern computers, "old" cryptography relied on physical or mechanical methods. In all these examples, the sender and recipient must share a secret piece of information (the Key).

### **The Scytale Cipher (Sparta, 700 BCE)**
* **Mechanism:** A strip of parchment is wrapped spirally around a wooden rod (the scytale). The message is written lengthwise along the rod. When unwound, the letters appear scrambled and random.
* **Decryption:** To read the message, the recipient must possess a rod of the **exact same diameter**. Wrapping the parchment around the matching rod realigns the letters.
* **Concept:** This is an early form of a **Transposition Cipher** (rearranging characters).

### **The Pharaoh Approach (Steganography)**
This method focused on hiding the message's existence rather than scrambling it. This is technically **Steganography**, not cryptography.
* **The Process:**
    1.  The sender shaves a slave's head.
    2.  The secret message is tattooed or carved directly onto the slave's skull.
    3.  The sender waits for the hair to grow back, completely hiding the text.
    4.  The slave is sent to the recipient.
* **Decryption:** The recipient shaves the slave's head again to reveal the message.
* **Trade-off:** It offers high secrecy but extremely slow transmission speed (limited by hair growth and travel time).

### **The Caesar Cipher**
* **Mechanism:** A substitution cipher where the alphabet is shifted by a predetermined number of positions (the key).
    * *Example:* With a shift of 3, 'A' becomes 'D', 'B' becomes 'E', etc.
* **Attack:** The keyspace is tiny (only 26 possible shifts). It is easily broken by **Brute Force**: an attacker simply tests all 26 shifts until the output makes sense.

### **Alphabetic Substitution**
* **Mechanism:** Instead of a uniform shift, every letter is replaced by a different letter using a random permutation (e.g., A=Z, B=M, C=L).
* **Keyspace:** This offers a massive number of possible keys ($26!$, which is approx. $4 \times 10^{26}$), making manual brute force impossible.
* **Weakness:** It preserves the statistical structure of the language (e.g., if 'E' is the most common letter in plaintext, its symbol will be the most common in ciphertext), making it vulnerable to **Frequency Analysis**.

### **The Enigma Machine (WWII)**
* **Mechanism:** An electromechanical device used by the Germans, featuring rotating electrical rotors.
* **Polyalphabetic Nature:** It produced a different substitution for every keystroke. If you typed 'A' three times, it might encrypt to 'X', then 'P', then 'L'. This flattened the frequency distribution.
* **Vulnerability:** Although advanced for its time, its keyspace ($\approx 10^{23}$ to $10^{26}$) is considered small by modern standards. It was broken by the Allies (Bletchley Park) and can be easily broken by today's computers.

---

# 3. Cryptanalysis: Frequency Analysis

Frequency analysis is the art of breaking substitution ciphers by analyzing the statistical occurrence of letters.

### **Language Fingerprints**
Every language has a specific, public distribution of letter usage.
* **English:** The most frequent letters are **E, T, A, O, N, I**.
    * The letter 'E' appears roughly **12-14%** of the time.
* **Other Languages:**
    * *French:* E, A, S, I, T.
    * *German:* E, N, I, S, R, A.

### **The Attack Workflow**
1.  **Clean:** Remove spaces, punctuation, and special symbols from the ciphertext.
2.  **Normalize:** Convert all text to the same case (e.g., all uppercase) to ensure consistency.
3.  **Count:** Calculate the frequency of every individual character in the ciphertext.
4.  **Align:** Match the most frequent ciphertext letters to the known language profile.
    * *Example:* If symbol '#' appears 13% of the time, assume '#' = 'E'.
5.  **Refine (Pattern Matching):** Use language rules to confirm the key.
    * **Common Pairs (Digrams):** Look for symbols representing TH, QU, CH, LL.
    * **Positions:** In English, 'Q' is almost always followed by 'U'.
    * **Short Words:** Look for 2 or 3-letter patterns representing "it", "is", "to", "the".

### **Limitations**
This technique fails against:
* **Short messages:** There is not enough data to form a reliable statistical profile.
* **Polyalphabetic ciphers:** Because the same plaintext letter maps to different ciphertext letters, the frequency counts are flattened.
* **Modern encryption:** Algorithms like AES are designed to produce output that looks statistically random.

---

# 4. Threat Models & Adversaries

### **The Communication Model**
* **Parties:** Alice (sender) and Bob (recipient).
* **Goal:** Send a message $M$ confidentially over a reliable communication line.
* **Components:** They share an encryption scheme ($E, D$) and keys ($K_1, K_2$).
* **Symmetric Encryption:** If $K_1 = K_2$, the system is symmetric (preferred for confidentiality).

### **The Adversary (Trudy/Eve)**
* **Passive Attack:** The adversary eavesdrops on the communication line. They read the messages but do not alter them.
* **Active Attack:** The adversary intervenes in the communication. They can:
    * Modify messages sent by Alice.
    * Send fake messages claiming to be Alice.

### **Kerckhoff's Principle**
A fundamental rule of modern cryptography:
> "A cryptosystem should be designed to be secure, even if all its details, except for the key, are publicly known."
* **Implication:** Security must rely *entirely* on the secrecy of the **key**, not on the secrecy of the algorithm. We assume the enemy knows exactly how the system works (Shannon's Maxim).

---

## 5. Attack Categorization

Attacks are often defined by the "Oracle"—what information the adversary possesses or can generate.

| Acronym | Attack Type | Description |
| :--- | :--- | :--- |
| **COA** | **Ciphertext-Only** | The attacker has only the ciphertext. They must deduce the plaintext or key with no other info. This is the hardest context for an attacker. |
| **KPA** | **Known-Plaintext** | The attacker has samples of pairs: (Plaintext, Ciphertext). They use these pairs to reverse-engineer the key. |
| **CPA** | **Chosen-Plaintext** | The attacker can encrypt arbitrary plaintexts of their choice to see the output. This allows them to study how the cipher behaves with specific inputs. |
| **CCA** | **Chosen-Ciphertext** | The attacker can decrypt arbitrary ciphertexts (except the target one) to see the result. |

**Other Attack Vectors:**
* **Brute-Force:** Trying all possible keys until the correct one is found.
    * *The Math:* A key of $n$ bits has $2^n$ possibilities.
    * *Speed:* Modern computers can test $10^6$ to $10^{15}$ keys/sec.
    * *Security:* A 128-bit key would take $>10^{19}$ years to break even at 1 trillion attempts/sec.
* **Side-Channel:** Exploiting physical leakage from the device rather than the math. This includes power consumption, timing variations, or electromagnetic radiation.
* **Replay Attack:** Capturing a valid message (like a login token) and resending it later to trick the system.

---

# 6. Keys vs. Passwords

While both are used to protect secrets, they are fundamentally different tools with different properties.

### **Passwords**
* **Origin:** Chosen by humans.
* **Constraint:** Must be memorable and **typable** (fit on a standard keyboard).
* **Entropy:** Low. The search space is limited to the character set ($T^n$) rather than the full binary space.
* **Vulnerabilities:** Susceptible to social engineering (phishing) and brute-force/dictionary attacks due to predictability.
* **Analogy:** Like a **PIN** typed into a keypad.

### **Cryptographic Keys**
* **Origin:** Generated by machines (RNG).
* **Properties:**
    * High entropy (randomness).
    * Fixed length (e.g., 128 or 256 bits).
    * Stored as raw binary data (often displayed as Hex), not text.
* **Analogy:** The **physical metal key** that actually turns the mechanism of a vault.

### **How They Work Together**
Systems like PGP, Signal, or WhatsApp often combine them:
1.  The system generates a high-security **Cryptographic Key** to encrypt your data.
2.  The user provides a **Password**.
3.  The system uses the Password to encrypt the Cryptographic Key (often using Key Derivation Functions like **Argon2** or **PBKDF2**).
4.  *Result:* The security is usable (password) but strong (key), though it is ultimately limited by the strength of the user's password.