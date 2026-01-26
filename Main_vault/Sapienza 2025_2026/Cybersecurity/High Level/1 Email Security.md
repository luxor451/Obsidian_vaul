# Email Security: Architecture, Protocols, and Cryptography

**Tags:** #EmailSecurity #NetworkSecurity #SMTP #PGP #DNS #Cryptography #CyberSecurity
**Source:** 
![[1 - Email security-1.pdf]]

---

## 1. Overview of the Email System

Electronic mail (email) is one of the oldest services on the Internet, originating in 1971 when Ray Tomlinson sent the first message across ARPANET using the "user@domain" addressing scheme.

### 1.1 Architecture
Modern email systems function on a **store-and-forward** model. This asynchronous nature means neither the sender nor the recipient needs to be online simultaneously; servers store messages until they can be forwarded or delivered.

The architecture consists of several distinct agents:
![[Pasted image 20260122185603.png]]

* **Message User Agent (MUA):** The client application used by the author to compose emails and by the recipient to read them (e.g., Outlook, Thunderbird, Webmail).
* **Mail Submission Agent (MSA):** Receives the email from the MUA, checks for standard formatting, and cooperates with the MTA for delivery.
* **Mail Transfer Agent (MTA):** Transfers messages from one computer to another using a client-server architecture. MTAs implement both client and server portions of SMTP.
* **Mail Delivery Agent (MDA):** Responsible for the final delivery of the message to the local recipient's environment (mailbox).
* **Message Store (MS):** Where the email is stored until retrieved by the user.



### 1.2 Protocols
The exchange of email relies on specific protocols for different stages of transmission:

1.  **SMTP (Simple Mail Transfer Protocol):** Defined originally in RFC 821 (1982) and updated in RFC 5321 (2008). It is used for transmitting messages across IP networks and by MUAs to submit mail to an MSA.
2.  **POP (Post Office Protocol):** Used by MUAs to download mail for offline usage.
3.  **IMAP (Internet Message Access Protocol):** Used by MUAs for online mail reading and management.

**Standard SMTP Interaction Flow:**
1.  **Connection:** Server answers with code `220`.
2.  **Greeting:** Client sends `HELO` (or `EHLO` for ESMTP).
3.  **Envelope:** Client specifies `MAIL FROM` (sender) and `RCPT TO` (recipient).
4.  **Data:** Client sends `DATA`, followed by the message content (headers + body), ending with a single dot `.` on a new line.
5.  **Termination:** Server confirms receipt (`250 OK`), and client sends `QUIT`.

**ESMTP (Extended SMTP):**
Introduced in RFC 5321 to replace the original SMTP. It allows for extensions like `STARTTLS` ([[5 Web Security part II#7. Secured Transport (HTTPS and HSTS)|Transport Layer Security]]), `8BITMIME` (data transmission), and `AUTH` (authentication).

---

## 2. Message Format

The Internet Message Format (IMF) is defined by **RFC 5322**. An email consists of a **Header** and a **Body**, separated by a single blank line.

### 2.1 The Header
Headers consist of fields with a name and a value (e.g., `Field-Name: Value`).
* **Mandatory Fields:** `From`, `To`, `Date`, `Message-ID` (globally unique identifier), and `Subject`.
* **Optional Fields:** `Cc` (Carbon Copy), `Bcc` (Blind Carbon Copy), `Reply-To`, `In-Reply-To` (for threading), and `Received` (tracking the route/hops).

> **Note on Tracking:** The `Received` field is added by each mail server processing the message. It is critical for tracing the path of an email.

### 2.2 MIME (Multipurpose Internet Mail Extensions)
Original email was restricted to 7-bit ASCII text. MIME (RFC 2045-2049) extends email to support non-ASCII character sets, images, audio, video, and multi-part bodies.

**Key MIME Headers:**
* `MIME-Version: 1.0`
* `Content-Type`: Defines the media type (e.g., `text/plain`, `text/html`, `image/jpeg`, `multipart/mixed`).
* `Content-Transfer-Encoding`: Defines how the binary data is converted to text for transmission.

**Encoding Schemes:**
1.  **Base64:** A binary-to-text scheme. It takes 3 bytes (24 bits) of data, splits them into four 6-bit groups, and maps them to a printable character set (`A-Z`, `a-z`, `0-9`, `+`, `/`). Padding with `=` is used if the data is not a multiple of 3 bytes.
2.  **Quoted-Printable:** Used for data that is mostly ASCII. Non-printing characters are represented by an `=` sign followed by their hexadecimal byte value (e.g., `=9D`).

**Email Example :**
![[Pasted image 20260122190139.png]]

---

## 3. Infrastructure Security (Sender Authentication)

Standard email protocols suffer from a lack of built-in security, leading to threats like Spam, Phishing, Spoofing, and Man-in-the-Middle attacks. Because SMTP allows the `From` header to be different from the actual sender, **Sender Authentication** technologies were developed to prevent spoofing.

### 3.1 SPF (Sender Policy Framework)
SPF (RFC 4408) allows a domain owner to specify which mail servers (IP addresses) are authorized to send email on behalf of that domain.

* **Mechanism:** The domain publishes a [[6 DNS Security|DNS]] TXT record.
* **Validation:** The receiving server checks the `Return-Path` (Envelope Sender) against the SPF record of that domain.
* **Example Record:** `v=spf1 mx include:_spf.google.com -all`
    * `mx`: Domain's MX servers are authorized.
    * `include`: Trusts another domain's SPF record (e.g., Google).
    * `-all`: Hard fail (reject all others). `~all` would be a soft fail.

**Limitations:** SPF breaks during **forwarding** because the intermediate server's IP is not listed in the original sender's SPF record. It also does not validate the visible `From` header, only the envelope sender.



### 3.2 DKIM (DomainKeys Identified Mail)
DKIM (RFC 4871) provides a cryptographic signature to ensure message integrity and authenticity.

* **Mechanism:**
    1.  **Signing:** The sender hashes specific headers and the body, encrypts the hash with their **private key**, and attaches it via the `DKIM-Signature` header.
    2.  **Publishing:** The sender publishes their **public key** in a DNS TXT record using a specific "selector".
    3.  **Verification:** The receiver retrieves the public key via DNS and verifies the signature.
* **Benefits:** Guarantees content integrity (no tampering) and non-repudiation of the domain.
* **Canonicalization:** Algorithms (simple or relaxed) are used to standardize headers/body before signing to allow for minor transport modifications (like whitespace) without breaking the signature.



### 3.3 DMARC (Domain-based Message Authentication, Reporting & Conformance)
DMARC (RFC 7489) unifies SPF and DKIM. It allows a domain to publish a policy on how receivers should handle emails that fail authentication.

* **Alignment:** DMARC introduces "Domain Alignment." It requires the domain in the `From` header to match the domain validated by SPF and/or the domain in the DKIM signature.
* **Policies (`p` tag):**
    * `none`: Monitor only (no action).
    * `quarantine`: Send to spam folder.
    * `reject`: Block the message entirely.
* **Reporting:** DMARC provides feedback loops via Aggregate (RUA) and Forensic (RUF) reports, allowing admins to monitor authentication failures.

### 3.4 ARC (Authenticated Received Chain)
ARC addresses the "forwarding problem" where legitimate indirect mail flows (mailing lists, forwarding) break SPF and DKIM.

* **Function:** It preserves the authentication results of the original message across intermediaries.
* **Header Fields:**
    1.  `ARC-Authentication-Results (AAR)`: Saves the authentication results (SPF/DKIM) from the previous hop.
    2.  `ARC-Message-Signature (AMS)`: Signs the message content and the AAR.
    3.  `ARC-Seal (AS)`: Signs the chain of ARC headers to prevent tampering.
* **Chain of Custody:** Each intermediary validates the previous chain and adds its own ARC headers. If the original authentication fails but the ARC chain is valid and trusted, the email can still be delivered.

---

## 4. End-to-End Security

While infrastructure security (SPF/DKIM/DMARC) protects the domain and transport, it does not guarantee confidentiality or end-to-end authentication between users. This requires encryption at the MUA level.

### 4.1 PGP (Pretty Good Privacy)
Created by Phil Zimmermann in 1991, PGP provides confidentiality, authentication, integrity, and non-repudiation. OpenPGP (RFC 4880) is the open standard.

**Confidentiality (Encryption):**
1.  Sender generates a random 128-bit **session key** ($K_s$).
2.  The message ($M$) is encrypted using $K_s$.
3.  The session key itself is encrypted using the recipient's **Public Key** ($PU_b$).
4.  The receiver uses their **Private Key** ($PR_b$) to decrypt $K_s$, then uses $K_s$ to decrypt the message.
    * *Formula:* $E[PU_b, K_s] + E[K_s, M]$

**Authentication (Digital Signature):**
1.  Sender creates a hash of the message $H(M)$.
2.  The hash is encrypted (signed) using the sender's **Private Key** ($PR_a$).
3.  The receiver decrypts the hash using the sender's **Public Key** ($PU_a$) and compares it to a newly calculated hash of the received message.
    * *Formula:* $E[PR_a, H(M)]$

**Confidentiality + Authentication:**
PGP can apply both: sign the message first, then encrypt the message and the signature together.



**Key Management (Web of Trust):**
PGP does not rely on centralized Certificate Authorities (CAs). Instead, it uses a **Web of Trust**:
* Users sign each other's public keys to vouch for their identity.
* A user trusts a key if it is signed by a "trusted introducer" or a chain of trusted individuals.
* **Key Rings:** Users maintain a "Public Key Ring" (keys of others) and a "Private Key Ring" (their own keys, protected by a passphrase).

### 4.2 S/MIME (Secure/Multipurpose Internet Mail Extensions)
S/MIME provides similar security guarantees to PGP but uses a different model for key management.
* **Infrastructure:** Relies on a centralized Public Key Infrastructure (PKI) with Certificate Authorities (CAs).
* **Certificates:** Uses X.509 certificates.
* **Trade-off:** Offers better enterprise interoperability but less user control compared to PGP's web of trust.

---

## 5. Summary of Threat Mitigation

| Threat Scenario | Mitigation Technology | Mechanism |
| :--- | :--- | :--- |
| **Spoofing (IP)** | **SPF** | Checks if the sending IP is authorized for the domain in the Envelope From. |
| **Tampering / Spoofing** | **DKIM** | Verifies content integrity and domain authorization via cryptographic signatures. |
| **Policy Enforcement** | **DMARC** | Tells the receiver what to do if SPF/DKIM fail (Reject/Quarantine) and aligns header domains. |
| **Forwarding Issues** | **ARC** | Preserves authentication results across intermediate hops (mailing lists, relays). |
| **Eavesdropping** | **PGP / S/MIME** | Encrypts the message body for end-to-end confidentiality. |
| **Identity Verification** | **PGP / S/MIME** | Digitally signs the message for end-to-end authentication. |
