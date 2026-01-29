**Tags:** #Cybersecurity #LowLevel #Authentication #PublicKey #WebAuthn #FIDO2 #Passkeys #Protocols

---

# 1. Public Key Authentication Principles

## Fundamentals
* **Key Pair:** Comprises a private key (secret) and a public key (shared).
* **Core Property:** Authentication is proving possession of the private key without revealing it.
* **Mechanism:** Generally uses a **Challenge-Response** flow.
    1.  Verifier sends a fresh random **Challenge** (nonce).
    2.  Prover **signs** the challenge with their Private Key.
    3.  Verifier checks the signature using the Prover's Public Key.
* **Benefit:** No shared secret is required between the prover and verifier.

## The Fake Public Key Problem
Public key authentication is only as strong as the trust in the public key itself.

**The Attack (Man-in-the-Middle):**
If the Verifier does not firmly associate the Public Key with the specific identity (Alice), an adversary can substitute their own key.
1.  Adversary tells Verifier: "I am Alice, here is my Public Key" (sends Adversary's PubKey).
2.  Verifier challenges Adversary.
3.  Adversary signs with Adversary's Private Key.
4.  Verifier checks signature against the fake key $\rightarrow$ Valid.
5.  **Result:** Full impersonation.

**Solutions:**
* **Certificates:** Digitally signed statements binding an identity to a public key ([[14 PKI|PKI]]).
* **Trust On First Use (TOFU):** Trust the key seen the first time (SSH style).
* **Out-of-Band (OOB):** Manual verification (QR codes, fingerprint comparison).

---

# 2. Needham-Schroeder Public Key Protocol

A protocol for mutual authentication using a trusted server (C) to distribute public keys.

## Protocol Steps
Let $K_{PX}$ be the Public Key of X, and $sig_C$ be the signature of the trusted server C.

1.  **A $\to$ C:** $A, B$ (Alice requests Bob's keys)
2.  **C $\to$ A:** $sig_C(B, K_{PB})$ (C sends Bob's signed public key)
3.  **A $\to$ B:** $K_{PB}(N_A, A)$ (Alice encrypts a nonce and her ID for Bob)
4.  **B $\to$ C:** $B, A$ (Bob requests Alice's keys)
5.  **C $\to$ B:** $sig_C(A, K_{PA})$ (C sends Alice's signed public key)
6.  **B $\to$ A:** $K_{PA}(N_A, N_B)$ (Bob proves he decrypted $N_A$, sends his own nonce $N_B$)
7.  **A $\to$ B:** $K_{PB}(N_B)$ (Alice proves she decrypted $N_B$)

## Vulnerability: Lowe's Attack (1995)
A Man-in-the-Middle (Trudy) can trick Alice if Alice initiates a session with Trudy.

1.  **A $\to$ T:** $K_{PT}(N_A, A)$ (Alice starts honest session with Trudy)
2.  **T $\to$ B:** $K_{PB}(N_A, A)$ (Trudy re-encrypts Alice's nonce for Bob, pretending to be Alice!)
3.  **B $\to$ A:** $K_{PA}(N_A, N_B)$ (Bob responds to Alice, thinking she initiated)
4.  **A $\to$ T:** $K_{PT}(N_B)$ (Alice decrypts the nonce, thinking it came from Trudy, and returns it encrypted for Trudy)
5.  **T $\to$ B:** $K_{PB}(N_B)$ (Trudy decrypts $N_B$ and re-encrypts for Bob)
* **Result:** Bob thinks he is talking to Alice, but Trudy is controlling the session.

## The Fix
Include the sender's identity in step 6.
* **Old:** $K_{PA}(N_A, N_B)$
* **New:** $K_{PA}(N_A, N_B, B)$
Alice will see that the message comes from B, but she was expecting a message from T, detecting the inconsistency.

---

# 3. Station-to-Station (STS) Protocol

An authenticated Key Exchange protocol based on Diffie-Hellman (DH), designed to prevent MITM attacks by adding signatures.

## Protocol Flow
1.  **A $\to$ B:** $g^x$ (Alice's DH half)
2.  **B $\to$ A:** $g^y, E_K(Sig_B(g^y, g^x))$
    * Bob calculates shared key $K = g^{xy}$.
    * Bob signs the DH values.
    * Bob encrypts the signature with $K$.
3.  **A $\to$ B:** $E_K(Sig_A(g^x, g^y))$
    * Alice calculates $K$, decrypts Bob's message, verifies signature.
    * Alice sends her encrypted signature.

**Key Features:**
* Provides **Perfect Forward Secrecy (PFS)** (if $x, y$ are ephemeral).
* Provides **Mutual Authentication**.
* Used in IPsec (IKE).

---

# 4. WebAuthn and FIDO2

**FIDO2** is the umbrella term for the modern passwordless authentication standard.
* **WebAuthn:** The W3C API built into browsers.
* **CTAP2:** The protocol for external authenticators (USB keys, phones) to talk to the computer.

## Roles
1.  **User:** The human.
2.  **User Agent:** The web browser.
3.  **Authenticator:** The security device (e.g., TouchID, YubiKey, Windows Hello).
    * **Platform Authenticator:** Built-in (Laptop fingerprint reader).
    * **Roaming Authenticator:** Removable (USB Security Key).
4.  **Relying Party (RP):** The website/server (e.g., google.com).

## The Registration Flow
1.  **Challenge:** RP sends a challenge and user info (ID, name).
2.  **Verification:** User unlocks Authenticator (PIN/Biometric).
3.  **Key Gen:** Authenticator generates a new key pair **unique to this origin** (RP ID).
4.  **Storage:**
    * Public Key is sent to RP.
    * Private Key is stored securely on the Authenticator.
    * *Note:* The Authenticator may generate a "Key Handle" (encrypted private key) to send to the server if it has limited storage.
5.  **Attestation:** Authenticator signs the data (proving its make/model) and sends it to RP.

## The Authentication Flow
1.  **Request:** RP sends a Challenge + List of allowed Credential IDs.
2.  **Verification:** User unlocks Authenticator (User Presence/Verification).
3.  **Signing:**
    * Authenticator finds the private key associated with the RP.
    * Signs the (Challenge + Client Data Hash).
    * Increments a counter (to prevent cloning).
4.  **Assertion:** Sends the signature (Assertion) to the RP.
5.  **Validation:** RP verifies signature with the stored Public Key.

## Security Properties
* **Phishing Resistant:** The protocol binds the key to the specific domain (Origin Binding). A fake site `g00gle.com` cannot trick the authenticator into signing a challenge meant for `google.com`.
* **Privacy:** Unique keys per site (Unlinkability).

## User Presence (UP) vs. User Verification (UV)
* **UP:** Simple touch. Proves a human is there (prevents malware auto-signing).
* **UV:** PIN or Biometric. Proves the *correct* human is there.

---

# 5. Passkeys

**Passkeys** are FIDO2 credentials designed to be **syncable** and usable across devices.

## Differences from Standard WebAuthn
* **Syncing:** Private keys are encrypted and synced via cloud (i.e., iCloud Keychain, Google Password Manager), rather than being bound to a single hardware chip.
* **Durability:** If you lose your phone, you don't lose access (restore from cloud backup).

## Cross-Device Authentication (Hybrid Transport)
Allows logging into a laptop using a passkey stored on a phone.
1.  **Discovery:** Laptop shows a QR code (containing crypto keys for a tunnel).
2.  **Connection:** Phone scans QR. Devices connect via **Bluetooth** (Local Proximity Check) to ensure the user is physically present (preventing remote phishing).
3.  **Tunnel:** A secure ephemeral channel is established (CTAP over WebSocket/Bluetooth).
4.  **Auth:** Phone performs biometric check, signs the challenge, and sends signature to Laptop.

## No PKI Required
Passkeys do not require a global Certificate Authority (CA).
* RP stores the user's public key directly during registration.
* Authentication is "Proof of Possession".
* Trust is established via the registration step ("TOFU" model).

## Comparison: Passwords vs. Passkeys

| Feature | Passwords | Passkeys |
| :--- | :--- | :--- |
| **Secret Type** | Shared Secret (User & Server know it) | Asymmetric Key Pair |
| **Server Storage** | Hashed Password (vulnerable to cracking) | Public Key (safe to leak) |
| **Phishing Risk** | High (User can type it anywhere) | **None** (Bound to origin domain) |
| **Entropy** | Low (Human memorable) | High (Cryptographic strength) |
| **Scope** | Often reused across sites | Unique key per domain |