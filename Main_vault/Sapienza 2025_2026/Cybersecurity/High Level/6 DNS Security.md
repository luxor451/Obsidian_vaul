# DNS Security: Vulnerabilities, Attacks, and DNSSEC

**Tags:** #DNSSecurity #NetworkSecurity #DNSSEC #CachePoisoning #DDoS #Tunneling #FastFlux #CyberSecurity #DoH #DoT
**Source:** ![[6 - DNS Security.pdf]]

---

## 1. The Domain Name System (DNS) Overview

The Domain Name System (DNS) is the phonebook of the Internet, implementing a massive, distributed database responsible for translating human-readable hostnames (like `www.google.com`) into IP addresses (like `142.250.184.4`) that computers use to communicate.

### 1.1 The Namespace Hierarchy
DNS is organized as a strict tree hierarchy rooted at the "dot" (`.`).

* **Root (`.`):** The top of the hierarchy. It consists of 13 logical server clusters (named A through M) managed by IANA (Internet Assigned Numbers Authority).
* **Top-Level Domains (TLD):** The next level down, managed by Registries (e.g., Verisign manages `.com`). Examples include `.com`, `.edu`, `.org`, and country codes like `.it` or `.uk`.
* **Second-Level Domains (SLD):** Domains managed by Registrars or specific owners (e.g., `google.com`, `uniroma1.it`).
* **Sub-domains:** Further divisions managed by the organization (e.g., `maps.google.com`).


### 1.2 The Concept of Delegation
The DNS does not store all data in one place. It relies on **delegation**. The parent zone (e.g., `.com`) does not know the IP address of `www.example.com`. Instead, it stores a pointer—specifically, **Name Server (NS)** records—that tells the resolver: "I don't know the IP, but here is the server authorized to answer for `example.com`."

### 1.3 The Resolution Process
When a user queries a domain, the resolution follows a recursive process:
1.  **User (Stub Resolver):** Sends a query to a Recursive Resolver (usually provided by the ISP or a public provider like 8.8.8.8).
2.  **Recursive Resolver:** If the answer is not in its cache, it starts from the Root.
    * Queries Root $\to$ Referred to TLD (e.g., `.com`).
    * Queries TLD $\to$ Referred to Authoritative Server (e.g., `ns1.google.com`).
    * Queries Authoritative Server $\to$ Receives the final IP (`A` record).
3.  **Response:** The Recursive Resolver sends the IP back to the user and caches it for a specific duration (TTL).

### 1.4 Packet Format
Understanding the DNS packet is crucial for understanding attacks. DNS typically runs over **UDP** (for speed) but can use TCP (for large responses or zone transfers).
![[Pasted image 20260123021108.png]]
**Key Header Fields:**
* **Transaction ID (TXID):** A 16-bit identifier (0–65,535) used to match a request with its corresponding reply.
* **Flags:**
    * `QR`: Query (0) or Response (1).
    * `AA`: Authoritative Answer (1 if the server "owns" the domain).
    * `RD`: Recursion Desired (set by the client).
* **Counts:** Number of Questions, Answers, Authority records, and Additional records.

---

## 2. Vulnerabilities in Standard DNS

The original DNS protocol (RFC 1035) was designed in the 1980s with scalability in mind, not security. It lacks built-in authentication or encryption.

### 2.1 The CIA Triad Status
1.  **Confidentiality:** **Non-existent.** DNS queries are sent in cleartext (UDP/53). Any passive observer (ISP, government, local attacker) can spy on every domain a user visits.
2.  **Integrity:** **Weak.** Integrity relies solely on the 16-bit Transaction ID. If an attacker can guess this ID, they can forge a response.
3.  **Availability:** **Generally High** (due to Anycast and caching), but vulnerable to amplification attacks and resource exhaustion.

### 2.2 The Threat Landscape
* **Surveillance:** Passive monitoring of user browsing habits.
* **Cache Poisoning:** Injecting fake data into a recursive resolver's cache, redirecting users to malicious sites.
* **Spoofing:** Impersonating an authoritative server.
* **[[7 DoS Attacks#4.2 Distributed Denial of Service (DDoS)|DDoS]]:** Using DNS to overwhelm victims.

---

## 3. DNS Spoofing and Cache Poisoning

This is the most critical integrity attack against DNS. The goal is to force a resolver to cache a malicious IP address for a legitimate domain (e.g., making `bankofamerica.com` point to an attacker's server).

### 3.1 The Mechanics of Spoofing
Since DNS uses UDP, it is connectionless. A resolver accepts a response if three conditions are met:
1.  **Destination IP:** Matches the resolver's IP.
2.  **Destination Port:** Matches the source port of the query.
3.  **Transaction ID (TXID):** Matches the ID sent in the query.

If an attacker sends a forged response that satisfies these criteria *before* the legitimate authoritative server responds, the resolver accepts the fake data. This is a **Race Condition**.

### 3.2 Difficulty Factors
To succeed, the attacker must guess:
* **TXID:** 16 bits $\to$ 65,536 possibilities.
* **Source Port:** Modern resolvers randomize the source port (approx. 60,000 possibilities).
* **Total Entropy:** $2^{16} \times 2^{16} \approx 4$ billion combinations.

While 4 billion sounds high, in a high-bandwidth network environment, a "birthday attack" approach can succeed surprisingly quickly.

### 3.3 The Kaminsky Attack (Subdomain Flooding)
Traditional poisoning requires waiting for the TTL of a record to expire before trying again. Dan Kaminsky discovered a method to bypass this restriction, allowing for continuous attempts.

**Attack Steps:**
1.  **Trigger:** The attacker forces the target resolver to query a *random subdomain* of the target zone (e.g., `random123.bank.com`).
2.  **Cache Miss:** Since `random123` doesn't exist in the cache, the resolver queries the authoritative server for `bank.com`.
3.  **Flood:** The attacker floods the resolver with thousands of fake responses.
    * **The Bait:** The response claims to be for `random123.bank.com`.
    * **The Poison (Glued Record):** The "Additional Section" of the fake response contains a malicious record: "`bank.com` is handled by Name Server `ns.evil.com`" or maps the real NS to a malicious IP.
4.  **Success:** If the attacker wins the race for *any* of the random subdomains, the resolver caches the malicious Name Server record for the entire `bank.com` zone.
5.  **Impact:** All future queries for `bank.com` are sent to the attacker's server. This is a **Full Domain Hijack**.


---

## 4. DNS as an Attack Vector (Malware & DDoS)

Attackers not only exploit DNS to redirect users; they also abuse the protocol itself to support their infrastructure.

### 4.1 DDoS Amplification
DNS is ideal for **[[7 DoS Attacks#5.3 Reflection and Amplification|Reflective DDoS attacks]]** because it uses UDP (easy to spoof source IP) and has a large response-to-request ratio.
* **Mechanism:** An attacker sends a small query (60 bytes) to an open DNS resolver, spoofing the source IP to be the Victim's IP.
* **Query:** Usually `ANY` or `TXT`, requesting a large record.
* **Response:** The resolver sends a massive response (3000+ bytes) to the Victim.
* **Amplification Factor:** 50x to 70x. A 1 Gbps attack stream becomes 50 Gbps hitting the victim.

### 4.2 DNS Tunneling
[[16 Firewalls|Firewalls]] often block arbitrary TCP/UDP traffic but allow DNS queries (port 53) to pass through. Attackers exploit this to create a covert communication channel (Command & Control or Data Exfiltration).
* **Encoding:** Data is encoded into the hostname.
    * *Query:* `secret-password-data.evil.com`.
    * *Routing:* The query is forwarded hierarchically until it reaches the attacker's Authoritative Server for `evil.com`.
    * *Decoding:* The attacker logs the query and extracts the "secret-password-data".
* **Response:** The attacker sends commands back via `TXT` records or fake IP addresses (`A` records).

### 4.3 Domain Generation Algorithms (DGA)
Botnets need to contact their Command & Control (C2) servers. If they use a hardcoded domain (`evil.com`), defenders can blacklist it easily. To evade this, malware uses DGAs.
* **Function:** The malware runs a deterministic algorithm (based on time, date, or a crypto seed) to generate hundreds or thousands of random domain names every day (e.g., `adjk234.com`, `xkv991.com`).
* **Synchronization:** The attacker runs the same algorithm and registers only *one* of those domains.
* **Connection:** The bot tries to resolve the domains in the list until one succeeds.
* **Types:**
    * *Time-based:* Uses system date/time.
    * *Dictionary-based:* Concatenates random words (e.g., `shutter-island-dream.com`) to bypass entropy filters.

### 4.4 DNS Fast-Flux
This technique hides the location of the malicious server (C2 or Phishing site) behind a constantly changing network of compromised hosts (proxies).
* **Method:** The attacker sets a very short TTL (e.g., 60 seconds) for the malicious domain.
* **Rotation:** Every time the domain is queried, the DNS server returns a different set of IP addresses. These IPs belong to infected home computers (bots) that act as reverse proxies, forwarding traffic to the actual backend server.
* **Double Flux:** The attacker rotates both the `A` records (web servers) and the `NS` records (name servers), making the infrastructure extremely difficult to take down.

---

## 5. DNSSEC: The Security Extension

**DNSSEC (DNS Security Extensions)** was designed to add **Authentication** and **Integrity** to DNS. It prevents spoofing and cache poisoning by digitally signing data.

**What DNSSEC Provides:**
1.  **Origin Authentication:** Proves the data came from the true zone owner.
2.  **Data Integrity:** Proves the data wasn't tampered with in transit.
3.  **Authenticated Denial of Existence:** Cryptographically proves that a domain *does not* exist (preventing attackers from spoofing "Not Found" errors).

**What DNSSEC Does NOT Provide:**
* **Confidentiality:** Data is still sent in cleartext.
* **DoS Protection:** In fact, the larger packet sizes makes DNSSEC a potent tool for DDoS amplification.

### 5.1 New Record Types
DNSSEC introduces several new records to handle cryptography:
1.  **RRSIG (Resource Record Signature):** The actual digital signature. It does not sign individual records; it signs an **RRset** (a bundle of all records of the same type, e.g., all `A` records for `www.example.com`).
2.  **DNSKEY:** Contains the Public Key used to verify the RRSIG.
3.  **DS (Delegation Signer):** A hash of a child zone's key, stored in the parent zone. This links the trust chain.
4.  **NSEC / NSEC3:** Records used to prove a name doesn't exist.

### 5.2 Key Management: KSK and ZSK
To balance security and performance, DNSSEC uses a dual-key system:
* **Zone Signing Key (ZSK):**
    * Used to sign the actual zone data (A, MX, CNAME records).
    * Shorter key (e.g., 1024-bit) for faster computation.
    * Rolled over frequently (e.g., monthly) by the zone administrator.
* **Key Signing Key (KSK):**
    * Used ONLY to sign the DNSKEY RRset (which contains the ZSK and KSK public keys).
    * Longer, stronger key (e.g., 2048-bit).
    * Rolled over rarely (e.g., yearly) because it requires updating the parent zone (sending the new DS record to the Registrar).

### 5.3 The Chain of Trust
How do we trust the keys? Through a chain of delegations starting from the Root.
1.  **Trust Anchor:** The resolver is pre-configured with the Public Key of the Root Zone.
2.  **Root Verification:** The resolver uses the Root Key to verify the `DS` record for `.com`.
3.  **TLD Verification:** The `DS` record for `.com` verifies the KSK of the `.com` zone.
4.  **SLD Verification:** The `.com` zone contains a `DS` record for `example.com`, which validates `example.com`'s KSK.
5.  **Data Verification:** `example.com`'s KSK validates its ZSK; the ZSK validates the `A` record for `www.example.com`.

**Root Key Ceremony:**
The Private Key for the DNS Root is managed by ICANN with extreme physical security. It is generated/accessed during a scripted "Key Signing Ceremony" that is streamed publicly to ensure transparency and trust.

### 5.4 Authenticated Denial of Existence (NSEC vs. NSEC3)
If a user asks for `ghost.example.com` and it doesn't exist, how does the server prove it without signing a "Does Not Exist" message dynamically (which is expensive)?

* **NSEC:** The server returns a signed record saying: "There are no names between `ftp.example.com` and `mail.example.com`."
    * *Vulnerability:* **Zone Enumeration (Zone Walking).** An attacker can query `a.example.com`, then `b.example.com`, and use the NSEC responses to map out the entire zone file.
* **NSEC3:** Solves the enumeration problem by hashing the names.
    * The record says: "There are no names whose hash falls between `Hash(A)` and `Hash(B)`."
    * This prevents easy mapping of the zone while still proving non-existence.


---

## 6. Privacy and Future Protocols

Since DNSSEC does not hide *what* you are visiting, new protocols have emerged to secure the transport layer.

### 6.1 DNS over TLS (DoT) - RFC 7858
* **Port:** 853 (TCP).
* **Mechanism:** Wraps DNS packets inside a TLS tunnel.
* **Pros:** Provides full Confidentiality, Integrity, and Authentication between the client and the resolver.
* **Cons:** Uses a dedicated port (easy to block by firewalls). Higher latency due to TCP handshake.

### 6.2 DNS over HTTPS (DoH) - RFC 8484
* **Port:** 443 (TCP).
* **Mechanism:** Encodes DNS queries as HTTP/2 requests.
* **Pros:** Indistinguishable from normal HTTPS web traffic. Very hard for censorship bodies or network admins to block without blocking the entire web.
* **Cons:** Bypass enterprise security policies (since firewalls can't see the DNS requests). Can be used by malware for covert channels (C2) that are hard to detect.

---

## 7. Summary Table of DNS Defenses

| Threat | Defense Technology | Mechanism |
| :--- | :--- | :--- |
| **Spoofing / Poisoning** | **DNSSEC** | Cryptographic signatures (RRSIG) ensure data origin and integrity. |
| **Surveillance** | **DoT / DoH** | Encrypts the channel between client and resolver. |
| **Zone Enumeration** | **NSEC3** | Hashes domain names in denial-of-existence records. |
| **DDoS Amplification** | **RRL (Response Rate Limiting)** | DNS servers limit the rate of responses to a single requester. |
| **Tunneling / DGA** | **Traffic Analysis** | Analyzing entropy, query length, and frequency (Machine Learning). |