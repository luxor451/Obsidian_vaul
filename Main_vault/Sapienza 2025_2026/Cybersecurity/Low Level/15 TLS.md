**Tags:** #Cybersecurity #LowLevel #Cryptography #TLS #SSL #IPsec #IKE #VPN #NetworkSecurity

---

# 1. Transport Layer Security (TLS)

## Introduction
* **Definition:** TLS (and its predecessor SSL) are cryptographic protocols designed to secure communications over computer networks.
* **Goal:** Provide end-to-end confidentiality, integrity, and authentication for data exchanged between applications.
* **Layering:** TLS runs on top of TCP (Transport Layer) and below the Application Layer (e.g., HTTP $\rightarrow$ HTTPS).

## Core Threats Addressed
1.  **Eavesdropping:** Interception of communication to read data.
2.  **Tampering:** Unauthorized modification of data in transit.
3.  **Spoofing:** Impersonating a sender or receiver.
4.  **Replay Attacks:** Reusing captured data to trick the recipient.

## Authentication Models
* **Unilateral Authentication:** Standard on the web. The Client validates the Server's certificate. The Client remains anonymous at the TLS layer.
* **Mutual Authentication (mTLS):** Both Client and Server present [[14 PKI|X.509 certificates]]. Common in enterprise systems (Zero Trust) but rare on the public web.

---

# 2. TLS Architecture & Phases

TLS consists of two main layers: the **Handshake Protocol** (negotiation) and the **Record Protocol** (transport).

## Phase 1: Negotiation (Handshake)
Peers agree on cryptographic parameters (Cipher Suites) and establish shared secrets.

**The Cipher Suite:**
A combination of algorithms usually presented in a specific string format, e.g., `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384`.
* **Key Exchange:** ECDHE (Elliptic Curve Diffie-Hellman Ephemeral).
* **Authentication:** RSA (Signatures).
* **Encryption:** AES-256 (in GCM mode).
* **MAC/PRF:** SHA-384.

## Phase 2: Authentication & Key Exchange
* **Server Auth:** The server sends its Certificate chain. The client verifies the chain against its Root CA store.
* **Key Generation:** A "Pre-Master Secret" is exchanged/generated, from which the Master Secret and Session Keys are derived.

## Phase 3: Secure Transport (Record Protocol)
Once the handshake is complete, the Record Protocol takes over to protect application data.
1.  **Fragmentation:** Breaks data into manageable blocks ($2^{14}$ bytes max).
2.  **Compression:** (Optional and usually disabled due to attacks like CRIME/BREACH).
3.  **MAC:** Adds a Message Authentication Code for integrity (if not using AEAD).
4.  **Encryption:** Encrypts the data using the negotiated symmetric key.
5.  **Header:** Adds a header (Type, Version, Length).

---

# 3. TLS Handshake Protocol Flow

## TLS 1.2 Handshake (Full)
1.  **ClientHello:**
    * Sends highest supported TLS version.
    * Sends Client Random (32 bytes).
    * Sends List of supported Cipher Suites.
2.  **ServerHello:**
    * Selects the Protocol Version and Cipher Suite.
    * Sends Server Random.
3.  **Server Certificate:** Sends X.509 certificate chain.
4.  **ServerKeyExchange:** (If using Diffie-Hellman) Sends DH parameters and signature.
5.  **ServerHelloDone:** Indicates end of messages.
6.  **ClientKeyExchange:** Client sends the Pre-Master Secret (encrypted with Server's PubKey) OR Client's DH parameters.
7.  **ChangeCipherSpec:** Signals "switching to encrypted mode".
8.  **Finished:** Authenticated hash of all previous handshake messages (to prevent tampering).
9.  **Server ChangeCipherSpec & Finished:** Server confirms.

## TLS 1.3 Improvements
TLS 1.3 is a major overhaul focusing on speed (1-RTT) and security.
* **Removed:** Weak primitives (RSA key transport, SHA-1, CBC mode, RC4).
* **Speed:** The handshake is reduced to 1 Round Trip Time (1-RTT).
* **Forward Secrecy:** Mandatory. RSA Key Exchange (static) is removed; Ephemeral Diffie-Hellman (DHE/ECDHE) is required.
* **HelloRetryRequest:** Used only if the client didn't guess the server's preferred key share correctly.

---

# 4. IP Security (IPsec)

## Definition
IPsec is a suite of protocols that secures IP communications by authenticating and encrypting each IP packet of a communication session.
* **Layer:** Network Layer (Layer 3).
* **Transparency:** It protects all protocols running on top (TCP, UDP, ICMP) transparently to applications.

## Security Services
* **Confidentiality:** Encrypts packet payload.
* **Integrity:** Verifies packet has not been modified.
* **Authentication:** Verifies the source of the packet.
* **Anti-Replay:** Prevents replay of old packets via sequence numbers.

## Architecture Components
1.  **Security Protocols:** AH and ESP.
2.  **Key Management:** IKE (Internet Key Exchange).
3.  **Databases:** SAD (Security Association Database) and SPD (Security Policy Database).

---

# 5. IPsec Protocols: AH vs. ESP

## 1. Authentication Header (AH)
* **Goal:** Provides Integrity and Origin Authentication.
* **No Encryption:** The data is readable (plaintext).
* **Mechanism:** Computes a MAC (ICV) over the *entire* packet (IP Header + Payload).
* **Issue:** Because it protects the IP Header (including Source/Dest IP), it breaks **NAT** (Network Address Translation). NAT changes IP headers, which invalidates the AH signature.
* **Protocol ID:** 51.

## 2. Encapsulating Security Payload (ESP)
* **Goal:** Confidentiality, Integrity, and Authentication.
* **Mechanism:**
    * Encrypts the payload.
    * Adds an ESP Header and Trailer.
    * Authenticates the Encrypted Payload + Header (but *not* the outer IP Header).
* **NAT Traversal:** Works better with NAT because the outer IP header is not part of the integrity check.
* **Protocol ID:** 50.

---

# 6. IPsec Modes of Operation

## 1. Transport Mode
* **Scope:** Protects the payload of the IP packet (TCP/UDP segment).
* **Header:** Original IP header is kept.
* **Use Case:** Host-to-Host communication (End-to-End).
* **Overhead:** Low (adds only AH/ESP headers).

## 2. Tunnel Mode
* **Scope:** Protects the *entire* original IP packet.
* **Mechanism:**
    * Encrypts the original IP packet.
    * Encapsulates it in a **New IP Packet** with a new header (Gateway IPs).
* **Use Case:** Site-to-Site VPNs (Gateway-to-Gateway) or Remote Access VPNs (Host-to-Gateway).
* **Privacy:** Hides internal network topology (private IPs).

---

# 7. Security Associations (SA)

## Definition
An SA is a logical connection (agreement) between two devices defining how to protect traffic.
* **Simplex:** SAs are one-way. Bi-directional communication requires **two** SAs (one A $\rightarrow$ B, one B $\rightarrow$ A).
* **Identification:** An SA is uniquely identified by the triple:
    ~~~text
    < SPI, Destination IP, Protocol ID >
    ~~~
    * **SPI (Security Parameter Index):** A 32-bit tag in the packet header to pick the right key.

## Databases
1.  **SPD (Security Policy Database):**
    * Defines **What** to protect.
    * Rules: `DISCARD` ([[16 Firewalls|Firewall]] drop), `BYPASS` (Cleartext), `PROTECT` (Apply IPsec).
    * If `PROTECT` is matched, the system looks for an active SA.
2.  **SAD (Security Association Database):**
    * Defines **How** to protect.
    * Stores active keys, algorithms, SPIs, sequence numbers.

---

# 8. Internet Key Exchange (IKE)

IKE is the protocol used to dynamically set up Security Associations (SAs). It uses UDP port 500.

## IKEv1 vs IKEv2
* **IKEv1:** Complex, defined in multiple RFCs. Two phases (Main/Aggressive + Quick Mode).
* **IKEv2:** Simplified, faster, more reliable, built-in NAT traversal.

## IKEv2 Exchanges
1.  **IKE_SA_INIT:**
    * Negotiates crypto proposals (algorithms).
    * Performs Diffie-Hellman key exchange.
    * Exchanges Nonces.
    * *Result:* An encrypted channel for control messages.
2.  **IKE_AUTH:**
    * Authenticates the peers (Certificates or Pre-Shared Keys).
    * Establishes the first Child SA (for data traffic).
3.  **CREATE_CHILD_SA:**
    * Used for re-keying (changing keys periodically).
    * Used to create additional SAs for different traffic streams.
4.  **INFORMATIONAL:**
    * Keep-alives (Dead Peer Detection).
    * Error notifications.
    * Delete SA notifications.

## Summary Flow
~~~text
Initiator                         Responder
   |                                 |
   | ------ IKE_SA_INIT Request ---> | (Crypto, Nonce, DH)
   | <----- IKE_SA_INIT Response --- | (Crypto, Nonce, DH)
   |                                 |
   | -- {IKE_AUTH, SAi2, TSi..} ---> | (Encrypted Identity + Child SA setup)
   | <--- {IKE_AUTH, SAr2, TSr..} -- | (Encrypted Identity + Child SA setup)
   |                                 |
   | <======= IPsec Tunnel ========> | (Secure Data Transfer)
~~~