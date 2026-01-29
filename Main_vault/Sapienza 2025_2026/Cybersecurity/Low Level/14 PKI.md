**Tags:** #Cybersecurity #LowLevel #Cryptography #PKI #Certificates #X509 #TrustModels #DigitalSignatures

---

# 1. The Key Distribution Problem

## The Core Challenge
Public Key Cryptography provides strong mathematical assurances for confidentiality and integrity, assuming one condition is met: **Alice must possess Bob's genuine public key.**
* **The Assumption:** Most crypto textbooks assume the key exchange has already happened securely.
* **The Reality:** In a global network like the internet, Alice cannot physically meet every server administrator (like Bob) to verify their keys. She must retrieve Bob's key over a communication channel.
* **The Threat:** If the channel used to fetch the key is insecure, an attacker (Mallory) can intercept the request and send her *own* public key instead of Bob's.
    * Alice encrypts data with Mallory's key (thinking it's Bob's).
    * Mallory decrypts it, reads/modifies it, and re-encrypts it with Bob's real key to forward it.
    * Neither Alice nor Bob realizes the channel is compromised.
* **Conclusion:** The security of the entire system hinges not just on the strength of the encryption algorithms ([[7 Asymmetric encryption: the Diffie-Hellman intuition|RSA]], ECC), but on the **authenticity** of the public keys. Attacks rarely break the math; they exploit the trust infrastructure.

---

# 2. Trust Models

To solve the key distribution problem, we need a "Trust Model"—a systematic way to determine if a specific public key belongs to a specific identity.

## 1. Web of Trust (WoT)
* **Origin:** Popularized by PGP (Pretty Good Privacy) for email encryption.
* **Mechanism:** A decentralized, peer-to-peer trust model.
    * Users act as their own "Certification Authorities."
    * Alice signs Bob's key to vouch for him. Bob signs Carol's key.
    * **Transitive Trust:** If I trust Alice, and Alice trusts Bob, I might choose to trust Bob automatically.
* **Key Signing Parties:** Physical meetups where people verify identities (IDs) and sign each other's PGP keys to strengthen the web.
* **Pros:**
    * Highly resilient; no central point of failure.
    * Community-driven and resistant to government/corporate control.
* **Cons:**
    * **Complexity:** Requires users to understand keys, signatures, and trust levels.
    * **Scalability:** Hard to find a "trust path" to a stranger on the other side of the world.
    * **Subjectivity:** "Trust" is ill-defined (Does trusting Bob to be Bob mean I trust his ability to verify Carol?).

## 2. Trust On First Use (TOFU)
* **Origin:** The default model for **SSH** (Secure Shell).
* **Mechanism:**
    * **First Contact:** When connecting to a server for the very first time, the client has no way to verify the key. It asks the user to manually approve the key fingerprint.
    * **Pinning:** Once approved, the client saves (pins) the key locally.
    * **Subsequent Connections:** The client compares the presented key with the saved one. If it changes, a scary warning is displayed ("WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!").
* **Pros:**
    * Extremely simple and user-friendly after the initial setup.
    * Very effective for closed systems or developer tools.
* **Cons:**
    * **First Contact Vulnerability:** Vulnerable to MITM attacks during the very first connection attempt.
    * **Key Rotation Issues:** Legitimate key updates look exactly like attacks to the user.

## 3. Public Key Infrastructure (PKI)
* **Mechanism:** A strictly hierarchical, centralized trust model.
* **Root of Trust:** Relies on a small set of trusted organizations called **Certification Authorities (CAs)**.
* **Trust Anchors:** The public keys of these CAs come pre-installed in operating systems and browsers. If a CA signs Bob's key, my browser trusts Bob because it trusts the CA.
* **Status:** This is the dominant model for the World Wide Web (HTTPS/TLS).

---

# 3. PKI Components

PKI is not just software; it is a framework encompassing hardware, software, people, policies, and procedures.

## 1. Certification Authority (CA)
* The core entity responsible for the lifecycle of digital certificates.
* **Functions:**
    * Issues new certificates.
    * Signs certificates with its highly protected private key.
    * Publishes revocation information (CRLs/OCSP).
* **Security:** The CA's private key is the "Crown Jewels." If stolen, the entire trust hierarchy collapses. It is usually stored in a Hardware Security Module (HSM).

## 2. Registration Authority (RA)
* The entity that verifies the identity of the requester *before* the CA issues a certificate.
* **Role:** It acts as the "gatekeeper."
* **Workflow:**
    1. User applies for a cert.
    2. RA checks credentials (e.g., verifying domain ownership, checking passports for government IDs).
    3. RA tells CA: "This request is legitimate. Issue the cert."
* *Note:* In many commercial CAs, the RA function is automated and integrated into the CA software.

## 3. Certificate Repository
* A publicly accessible database or directory.
* **Content:**
    * Active digital certificates (for public retrieval).
    * Certificate Revocation Lists (CRLs).
* **Protocols:** Often accessed via LDAP or HTTP.

---

# 4. Digital Certificates (X.509)

A digital certificate is an electronic document that binds a Public Key to an Identity, signed by a trusted third party. The standard format is **X.509**.

## Structure of an X.509 Certificate
* **Version:** The version of the X.509 standard (usually v3).
* **Serial Number:** A unique integer assigned by the CA to distinguish this certificate from others it has issued.
* **Signature Algorithm:** The algorithm used by the CA to sign this cert (e.g., `sha256WithRSAEncryption`).
* **Issuer:** The Distinguished Name (DN) of the CA that issued the cert (e.g., "DigiCert Global Root CA").
* **Validity Period:**
    * `Not Before`: The date/time the cert becomes valid.
    * `Not After`: The expiration date/time.
* **Subject:** The Distinguished Name of the entity owning the key (e.g., `CN=www.google.com`, `O=Google LLC`, `C=US`).
* **Subject Public Key Info:** The actual public key and the algorithm used (e.g., RSA 2048-bit or ECDSA P-256).
* **Extensions (v3):** Critical for modern security.
    * **Key Usage:** Defines what the key can do (e.g., Digital Signature, Key Encipherment).
    * **Extended Key Usage (EKU):** (e.g., Server Authentication, Client Authentication).
    * **Basic Constraints:** Is this entity a CA? (True/False). Prevents a website from acting as a CA and signing other sites.
    * **Subject Alternative Name (SAN):** Allows one cert to secure multiple domains (e.g., `google.com`, `gmail.com`, `youtube.com`).
* **Digital Signature:** The cryptographic signature of the CA, covering all the fields above.

## The Chain of Trust
Certificates are rarely used in isolation; they form a chain.
1.  **Root Certificate:**
    * **Self-Signed:** The Issuer and Subject are the same.
    * Trusted implicitly because it is hard-coded in the software (Trust Store).
    * Kept offline for security.
2.  **Intermediate Certificate(s):**
    * Signed by the Root CA.
    * Used to sign End-Entity certificates.
    * Kept online to issue certs automatically.
    * *Purpose:* Protects the Root key. If an Intermediate is compromised, it can be revoked without replacing the Root.
3.  **Leaf (End-Entity) Certificate:**
    * Signed by an Intermediate CA.
    * Issued to the final user or server (e.g., `amazon.com`).
    * Cannot sign other certificates.

---

# 5. Certificate Lifecycle Management

Managing certificates is a continuous cycle.

1.  **Key Generation:** The applicant generates a key pair (Public/Private). The private key should *never* leave the applicant's server.
2.  **Registration (CSR):** The applicant creates a **Certificate Signing Request (CSR)** containing the Public Key and identity info, signs it with the Private Key (proof of possession), and sends it to the CA.
3.  **Verification:** The CA/RA validates the information based on the validation level (DV, OV, or EV).
4.  **Issuance:** The CA signs the public key and info, creating the X.509 certificate, and returns it.
5.  **Installation:** The admin installs the certificate on the web server or device.
6.  **Usage:** The certificate is used for TLS handshakes, email signing, etc.
7.  **Revocation:** If the private key is compromised or the business closes, the cert must be killed before its expiration date.
8.  **Renewal:** Certificates expire (currently ~398 days for web). They must be renewed (new key pair generated, new CSR sent) to maintain service.

---

# 6. Revocation: How to Trust, but Verify

If a private key is stolen, the certificate is still mathematically valid until it expires. Revocation mechanisms allow the CA to tell the world "Do not trust this anymore."

## 1. Certificate Revocation List (CRL)
* **Mechanism:** The CA publishes a distinct file (a "blacklist") containing the serial numbers of all revoked certificates.
* **Workflow:** The browser downloads the list and checks if the site's serial number is on it.
* **Pros:** Simple, standardized, easy to archive.
* **Cons:**
    * **Bandwidth:** CRLs can grow to be massive (megabytes), slowing down the initial connection.
    * **Latency:** CRLs are cached. If a revocation happens at 10:00 AM and the list publishes at 12:00 PM, there is a vulnerability window.

## 2. Online Certificate Status Protocol (OCSP)
* **Mechanism:** A real-time query protocol. The browser sends a specific request to the CA's OCSP responder: "Is Serial #12345 valid?"
* **Response:** "Good", "Revoked", or "Unknown".
* **Pros:** Lightweight (small packets), real-time status.
* **Cons:**
    * **Privacy:** The CA learns every site the user visits (because the browser queries the CA for every site).
    * **Reliability:** If the OCSP server is offline, browsers often "Fail Open" (accept the cert) to avoid breaking the internet, rendering the check useless.
* **Solution: OCSP Stapling:** The *web server* queries the OCSP responder periodically, gets a signed timestamped approval, and sends it ("staples it") to the client during the TLS handshake. This protects privacy and performance.

---

# 7. Types of Digital Certificates

Different use cases require different types of identity assurance.

## SSL/TLS Server Certificates
* **Domain Validated (DV):**
    * **Validation:** CA verifies only that the requester controls the domain (e.g., by uploading a file or adding a DNS record).
    * **Speed:** Automated, issued in minutes.
    * **Cost:** Often free (e.g., Let's Encrypt).
    * **Use:** Blogs, personal sites, small businesses.
* **Organization Validated (OV):**
    * **Validation:** CA verifies the legal existence of the organization (business registry check).
    * **Info:** Organization name appears in the certificate details.
    * **Use:** Corporate websites, internal systems.
* **Extended Validation (EV):**
    * **Validation:** Extremely strict. CA checks physical address, legal status, and verifies the request with corporate officers.
    * **UI:** Formerly triggered a "Green Bar" in browsers (now mostly removed due to lack of user impact).
    * **Use:** Banks, financial institutions (though usage is declining).

## Other Certificate Types
* **Client Certificates:**
    * Used for **Mutal TLS (mTLS)**. The server authenticates the user via a certificate instead of a password.
    * High security; often used in corporate VPNs and internal apps.
* **S/MIME Certificates:**
    * Used to encrypt and digitally sign emails.
    * Ensures the sender is legitimate and the email content hasn't been altered (Phishing defense).
* **Code Signing Certificates:**
    * Used by developers to sign software executables (.exe, .dmg).
    * Ensures the code comes from a known publisher and hasn't been infected with malware/viruses since build time.
    * Operating systems (Windows, macOS) warn or block unsigned code.
* **Device / IoT Certificates:**
    * Used for machine-to-machine authentication.
    * **Scale:** Issued in the millions for connected cars, smart meters, sensors.
    * **Automation:** Managed via protocols like **EST** (Enrollment over Secure Transport) or **ACME**.
* **Qualified Certificates (eIDAS):**
    * Specific to the European Union context.
    * Provides "Qualified Electronic Signatures" which have the exact same legal standing as a handwritten signature in court.
    * Requires storing keys on secure hardware (Smart Cards).
* **Self-Signed Certificates:**
    * Created by the user, not a trusted CA.
    * **Pros:** Free, infinite lifetime, instant.
    * **Cons:** Browsers display severe security warnings. No protection against MITM attacks (anyone can generate a self-signed cert for `google.com`).
    * **Use:** Testing environments, local development.