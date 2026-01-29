# Firewalls and Network Defense Architectures

**Tags:** #cybersecurity #firewalls #network-security #architecture #DMZ #NAT #packet-filtering #proxy

---

# 1. Introduction to Firewalls

## Concept
A **Firewall** is a security system designed to separate a local network (trusted) from an external network (untrusted, typically the Internet).
* **Core Idea:** Isolate the internal network from external threats while allowing authorized communication.
* **Placement:** Typically situated at the perimeter, linking the internal network (Intranet) to the WAN (Wide Area Network).

## Functions
1.  **Traffic Control:** Monitors and controls both inbound (incoming) and outbound (outgoing) traffic.
2.  **[[1 Information security#Access Control|Access Control]]:** Only authorized traffic is allowed to pass through based on a defined security policy.
3.  **Hiding:** It hides the internal network topology from the external world.
4.  **Logging:** Controls and monitors access to services for auditing purposes.

## The "Personal Firewall"
* A software firewall installed directly on an end-user machine.
* Protects the individual device rather than the network perimeter.
* Should be immune to attacks itself.

---

# 2. Limitations: What Firewalls Cannot Do

While critical, firewalls are not a silver bullet.
1.  **Bypassed Traffic:** Does not protect against attacks that bypass the firewall (e.g., internal dial-up access, side-channels).
2.  **Internal Threats:** Does not protect against attacks originating *within* the protected network (Insider threats).
3.  **[[2 Network Security#4 Malware|Malware]]:** Cannot block all [[2 Network Security#4 Malware|viruses, worms, or trojans]], especially if they are embedded in allowed traffic (like HTTP) or depend on specific OS vulnerabilities.
4.  **Application Vulnerabilities:** Poorly written applications (buffer overflows) or bad protocol design (WEP) remain vulnerable if the traffic is allowed through.
5.  **[[7 DoS Attacks|Denial of Service (DoS)]]:** generally limited effectiveness against volumetric DoS attacks.

---

# 3. Types of Firewalls

Firewalls are categorized by the layer of the OSI model they operate on and their state awareness.

## 1. Packet-Filtering Router
* **Mechanism:** Inspects the header of each packet against a set of rules (Access Control Lists - ACLs).
* **Operation:**
    * Checks Source/Destination IP.
    * Checks Protocol (TCP/UDP/ICMP).
    * Checks Source/Destination Ports.
    * **Stateless:** Decisions are made on a packet-by-packet basis without memory of previous packets.
* **Pros:** Fast, low cost, transparent to users.
* **Cons:** Hard to configure complex rules, no context awareness (can't tell if a packet is part of an established flow), vulnerable to IP spoofing.

## 2. Stateful Inspection Packet Filter
* **Mechanism:** Tracks the state of active connections to make smarter decisions.
* **Operation:**
    * Maintains a state table containing {Source IP, Dest IP, Ports, Sequence Numbers}.
    * A packet is allowed only if it matches a defined rule OR corresponds to an existing active connection.
    * *Example:* An incoming TCP packet with the `ACK` flag set is only allowed if it matches a sequence number expected from an established internal session.
* **Pros:** More secure than simple packet filters, handles dynamic protocols (like FTP) better.

## 3. Application-Level Gateway (Proxy)
* **Mechanism:** Acts as a middleman (relay) for specific application protocols.
* **Operation:**
    * Client connects to Proxy $\rightarrow$ Proxy connects to Destination.
    * The proxy understands the application protocol (e.g., HTTP, FTP, Telnet, [[1 Email Security|SMTP]]) and can inspect the *payload* (content).
    * *Example:* An SMTP proxy can strip malicious attachments from emails before forwarding them.
* **Pros:** High security (can strip malicious commands), logging at application level.
* **Cons:** High overhead (CPU intensive), requires specific proxy software for each protocol, not transparent to users.

## 4. Circuit-Level Gateway
* **Mechanism:** Sets up two TCP connections (one between user and gateway, one between gateway and host).
* **Operation:**
    * Relays TCP segments from one connection to the other without examining the contents.
    * Often implemented via SOCKS.
* **Pros:** Faster than Application Proxies.
* **Cons:** Does not inspect the data payload.

---

# 4. Deep Dive: Packet Filtering Examples

Packet filters use a rule table to decide the fate of a packet. Rules are processed top-to-bottom.

## Example Rule Set
Here is how a packet filter might implement a policy to **Block inbound connections** but **Allow outbound web access**:

| Rule # | Source IP | Dest IP | Protocol | Dest Port | Action | Explanation |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | Internal Net | Any | TCP | 80 (HTTP) | **Allow** | Allow internal users to browse the web. |
| **2** | Any | Internal Net | TCP | Any | **Block** | Block all unsolicited incoming TCP connections. |
| **3** | Any | Any | UDP | Any | **Block** | Block all UDP traffic (default deny). |

* **Scenario A:** An internal user (192.168.1.5) tries to access Google (8.8.8.8) on port 80.
    * Matches **Rule 1**. Action: **Allow**.
* **Scenario B:** An external attacker tries to connect to an internal server on port 22 ([[0 Introduction#Secure Protocols|SSH]]).
    * Fails Rule 1. Matches **Rule 2**. Action: **Block**.

---

# 5. Firewall Architectures

How firewalls are deployed to create secure network zones.

## 1. Bastion Host
* **Definition:** A computer system specifically hardened to withstand attacks.
* **Placement:** Directly exposed to the untrusted network (Public Internet).
* **Role:** Hosts publicly accessible services (Web Server, [[6 DNS Security|DNS]], Mail).
* **Hardening:** All unnecessary services, accounts, and programs are removed to minimize the attack surface.

## 2. Dual-Homed Bastion Host
* **Structure:** A Bastion Host with **two** network interfaces (NICs).
    * Interface 1: Connected to the Internet.
    * Interface 2: Connected to the Private Network.
* **Function:** Physically isolates the internal network from the external one. No direct packet forwarding (routing) is allowed between the interfaces; all traffic must be proxied or filtered by the host software.

## 3. Screened Host Architecture
* **Components:**
    1.  **Packet-Filtering Router:** Connected to the Internet.
    2.  **Bastion Host:** Located on the internal network.
* **Flow:**
    * The router is configured to force all incoming traffic to go *only* to the Bastion Host.
    * Internal clients must connect to the Internet via the Bastion Host (Proxy).
* **Risk:** If the Bastion Host is compromised, the intruder is already inside the internal network and can attack other internal hosts directly.

## 4. Screened Subnet Architecture (DMZ)
* **Concept:** Creates an isolated intermediate network called a **Demilitarized Zone (DMZ)**.
* **Components:**
    1.  **Outside Router:** Protects the DMZ from the Internet.
    2.  **Bastion Hosts:** Public servers (Web, Mail, DNS) placed inside the DMZ.
    3.  **Inside Router:** Protects the Internal Network from the DMZ.
* **Security:**
    * The Internal Network is invisible to the Internet.
    * Only the DMZ is reachable from outside.
    * **Defense in Depth:** If a Bastion Host in the DMZ is compromised, the attacker is still stuck in the DMZ. They must break through the **Inside Router** to reach the private network.

---

# 6. Network Address Translation (NAT)

## Function
* **Hiding:** NAT hides the IP addresses of internal hosts, making the internal topology opaque to attackers.
* **Mapping Types:**
    * **Static (1-to-1):** Maps a private IP to a specific public IP. Used for servers that need a consistent external address.
    * **Dynamic / Masquerading (N-to-1):** Maps multiple private IPs to a single public IP using different source ports. Used for client workstations sharing an internet connection.

## Route Filtering
* Routers should be configured to **not advertise** internal private routes (e.g., 192.168.x.x) to the public Internet.
* **Anti-Spoofing:** Routers should drop packets coming *from* the Internet that claim to have a source IP belonging to the internal network (Prevent attacker from advertising that the shortest route to an internal host lies through him).

---

# 7. Implementation Issues

* **Complexity:** As networks grow, firewall rulesets become massive and contradictory. Misconfiguration is a primary vulnerability.
* **Protocol Interference:** Firewalls often break protocols that embed IP addresses in their payload (e.g., FTP, SIP/VoIP) or require secondary dynamic ports.
* **Performance:** Deep Packet Inspection (DPI) and Application Proxies introduce significant latency.
* **False Sense of Security:** Having a firewall does not remove the need for patching software or securing endpoints.