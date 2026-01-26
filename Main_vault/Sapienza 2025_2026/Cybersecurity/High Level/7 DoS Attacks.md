# Denial of Service (DoS): Mechanisms, Evolution, and Defense

**Tags:** #DoS #DDoS #NetworkSecurity #CyberSecurity #Botnet #Amplification #Slowloris #Mirai #Availability
**Source:** ![[7 - DoS Attacks.pdf]]

---

## 1. Introduction to Denial of Service

A **Denial of Service (DoS)** attack is defined as any malicious attempt that reduces or eliminates the availability of a service. The primary objective is to make services, data, or resources unavailable to their intended, authorized users. Unlike intrusions, which target confidentiality or integrity (stealing data or modifying systems), DoS attacks strictly target **Availability**, a core component of the CIA Triad.

### 1.1 Impact and Severity
The consequences of a DoS attack are immediate and operational. They can range from minor annoyances to catastrophic business failures:
* **Complete Unavailability:** The service goes offline entirely (e.g., a web server returning 503 errors or timing out).
* **Severe Performance Degradation:** The service becomes incredibly slow, rendering it practically unusable.
* **SLA Violations:** Service Level Agreements are breached, leading to financial penalties.
* **Operational Stoppage:** For industrial or logistical targets (like airlines), DoS can stop physical operations (e.g., ticket sales, check-in systems).


### 1.2 Historical Evolution
The landscape of DoS attacks has evolved significantly over the last three decades, moving from simple script-kiddie tools to weaponized, nation-state grade infrastructure.

* **1990s:** The era of simple, single-source attacks. Tools like **Ping of Death** (exploiting packet size limits) and basic **SYN Floods** were common.
* **2000s:** The rise of distributed attacks (**DDoS**). Attackers began using botnets (compromised computers) and reflection techniques to amplify traffic.
* **2010s:** The shift toward **Application-Layer** attacks. Instead of just flooding pipes, attackers started targeting specific logic flaws (e.g., Slowloris).
* **2020s:** The era of **IoT Botnets** and hyper-volumetric attacks. Malware like **Mirai** turned cameras and routers into massive botnets, and attackers began abusing protocols like HTTP/2 for "Rapid Reset" attacks.

---

## 2. Threat Model and Motivations

Understanding the adversary is crucial for defense. A DoS threat model analyzes the attacker's goals, capabilities, and the victim's assets.

### 2.1 Attacker Goals
1.  **Disruption:** Simply causing chaos or stopping a service.
2.  **Extortion:** Demanding payment to stop or prevent an attack.
3.  **Distraction:** Launching a noisy DoS attack to distract security teams while a stealthier intrusion (data exfiltration) occurs in the background.

### 2.2 Motivations
* **Financial Extortion:** Groups like **DD4BC** (DDoS for Bitcoin) operate like racketeers. They launch a small "demonstration" attack and threaten a larger one unless a ransom is paid in cryptocurrency.
* **Hacktivism:** Political or ideological groups use DDoS as a form of protest. A famous example is **Operation Payback** (2010), where Anonymous attacked Visa, Mastercard, and PayPal for cutting off donations to WikiLeaks.
* **Competitive Sabotage:** Unscrupulous businesses attacking competitors. The **Spamhaus vs. CyberBunker** (2013) conflict is a prime example, where a massive DDoS was launched to protect spam-hosting business interests.
* **Cyberwarfare:** Nation-states using DDoS to cripple critical infrastructure. In February 2022, just before the invasion, coordinated waves of DDoS attacks hit Ukrainian government portals (Ministry of Defense) and banks (PrivatBank) to sow confusion and disrupt communications.

---

## 3. The Attack Surface

Attackers can target any layer of the stack, from the physical cable to the application logic.

### 3.1 Network Stack Layers
* **L3 (Network Layer):** Attacks aiming to saturate the bandwidth capacity of the link.
    * *Technique:* UDP Reflection/Amplification.
* **L4 (Transport Layer):** Attacks aiming to exhaust state tables on firewalls, load balancers, or operating systems.
    * *Technique:* TCP SYN Floods (exhausting connection backlogs).
* **L7 (Application Layer):** Attacks aiming to exhaust CPU or memory by forcing the server to perform expensive work.
    * *Technique:* HTTP GET floods on non-cacheable search endpoints.

### 3.2 Expanded Surfaces
* **Application Logic:** Poorly designed features can be weaponized. If a login page hashes passwords using expensive algorithms (like bcrypt) without rate limiting, an attacker can exhaust the CPU by sending thousands of login requests.
* **Shared Infrastructure:** Dependencies like [[6 DNS Security|DNS]] resolvers or CDNs are single points of failure. If the DNS provider is taken down (as seen in the Dyn attack), the service becomes unreachable even if the servers are fine.
* **Third-Party Dependencies:** External Identity Providers (IdP) or payment gateways.

---

## 4. Classification: DoS vs. DDoS

### 4.1 Single-Source DoS
The attack originates from a single machine.
* **Limitation:** The attacker is limited by their own upload bandwidth and CPU.
* **Mitigation:** Easy to block. Once the source IP is identified, a simple firewall rule stops the attack.

### 4.2 Distributed Denial of Service (DDoS)
The attack originates from thousands or millions of distributed sources (a **Botnet**).
* **Amplification:** The aggregate bandwidth of the botnet vastly exceeds the victim's capacity.
* **Attribution:** Difficult, as the traffic comes from "innocent" compromised devices (cameras, DVRs, routers) often spoofing IPs.
* **Mitigation:** Harder to block. You cannot simply block one IP; you must distinguish malicious traffic patterns from legitimate user traffic.


### 4.3 The Mirai Botnet Case Study
In 2016, the **Mirai** botnet changed the game. Unlike previous botnets made of PCs, Mirai scanned the internet for IoT devices (cameras, routers) with default credentials (e.g., admin/admin).
* **Scale:** It amassed hundreds of thousands of bots.
* **Impact:** It launched a 665 Gbps attack against security researcher Brian Krebs.
* **Legacy:** The source code was leaked ("Anna-senpai"), leading to variants like **Aisuru** (2025), which achieved record-breaking attacks (29.7 Tbps) using cloud-native concepts and carpet-bombing techniques.

---

## 5. Volumetric Attacks

These attacks aim to consume the available bandwidth of the target network. The goal is congestion: filling the pipe so legitimate packets are dropped.

### 5.1 UDP Floods
A network-layer attack where the attacker blasts the target with UDP packets.
* **Why UDP?** It is connectionless and stateless. The attacker does not need to complete a handshake (like in TCP), allowing for extremely high packet generation rates.
* **Mechanism:** The target receives packets on random ports. It checks for an application listening on that port, finds none, and generates an ICMP "Destination Unreachable" packet. This process consumes CPU and bandwidth.
* **Spoofing:** Attackers almost always spoof source IPs to hide their location and prevent return traffic from hitting them.
* **Carpet Bombing:** A variant where the attacker sprays traffic across the entire IP subnet rather than a single IP, making detection by standard thresholds difficult.

### 5.2 ICMP Floods
The attacker overwhelms the target with ICMP Echo Request ("Ping") packets.
* **Impact:** The victim spends CPU cycles parsing requests and generating Echo Replies. Bandwidth is consumed in both directions (Ingress and Egress).
* **Smurf Attack:** An ancient but classic variant.
    1.  Attacker spoofs the **Victim's IP** as the source.
    2.  Attacker sends a Ping to a network's **Broadcast Address**.
    3.  Every host on that network replies to the Victim.
    4.  Result: Traffic is multiplied by the number of hosts on the subnet.

### 5.3 Reflection and Amplification
These are the force multipliers of volumetric attacks. They exploit UDP-based services that do not verify the source IP.

**The Concept:**
1.  **Reflection:** The attacker sends a request to a third-party server (Reflector) but spoofs the source IP to be the Victim's IP. The Reflector sends the reply to the Victim. The attacker stays hidden.
2.  **Amplification:** The attacker chooses a protocol where the Response is much larger than the Request.

**Common Amplification Vectors:**
* **[[6 DNS Security#4.1 DDoS Amplification|DNS Amplification]]:**
    * *Mechanism:* Attacker sends a small query (60 bytes) for `ANY` records. The server responds with a massive answer (3000+ bytes).
    * *Factor:* 50x - 70x.
    * *NXDOMAIN Flood:* A variant where attackers query non-existent domains to exhaust the recursive resolver's resources.
* **NTP Amplification:**
    * *Mechanism:* The `MONLIST` command asks an NTP server for the details of the last 600 hosts that connected. A small command triggers a massive list response.
    * *Factor:* Up to 200x.
* **Memcached Amplification:**
    * *Mechanism:* Memcached servers (often exposed by mistake on port 11211 UDP) allow storing large blobs of data. Attackers verify a large value is stored, then request it with a spoofed packet.
    * *Factor:* Up to 50,000x. This powered the 1.35 Tbps attack on GitHub.


---

## 6. Protocol Attacks

These attacks exploit the stateful nature of protocols (TCP, IP) to exhaust resources on middleboxes (firewalls, load balancers) and servers. They target connection tables and memory buffers rather than raw bandwidth.

### 6.1 TCP SYN Flood
This attack exploits the TCP 3-way handshake.
1.  **Normal Handshake:** Client sends `SYN` $\to$ Server allocates memory (Backlog) and sends `SYN-ACK` $\to$ Client sends `ACK`. Connection established.
2.  **The Attack:** Attacker sends thousands of `SYN` packets.
3.  **The Trap:** The server allocates memory for each "half-open" connection and sends `SYN-ACK`.
4.  **The Impact:** The attacker never sends the final `ACK`. The server's connection queue (Backlog) fills up, and it stops accepting legitimate connections.

**Mitigation: SYN Cookies**
The server does *not* allocate memory upon receiving a SYN. Instead, it encodes the connection details into the Initial Sequence Number (ISN) of the `SYN-ACK`. Memory is only allocated when the client responds with the final `ACK` containing the valid math derived from that ISN.

### 6.2 ACK and RST Floods
* **ACK Flood:** Sending stray `ACK` packets to a server. The server wastes CPU looking up the connection table to match the packet to a session. If no session exists, it generates an RST, consuming egress bandwidth.
* **RST Flood:** Sending packets with the `RST` (Reset) flag. This forces firewalls and load balancers to process the termination logic, effectively purging valid connections or overloading the state engine.

### 6.3 Fragmentation Attacks
These attacks exploit how IP handles packets larger than the MTU (Maximum Transmission Unit).
* **Reassembly Exhaustion:** The attacker sends thousands of incomplete IP fragments. The victim's OS holds these fragments in memory, waiting for the missing pieces to reassemble the packet. The memory fills up, preventing legitimate packet processing.
* **FragmentSmack (2018):** A vulnerability in Linux/Windows where inefficient reassembly algorithms led to 100% CPU usage with very low attack traffic.

---

## 7. Application-Layer Attacks (L7)

These attacks mimic legitimate user behavior to exhaust specific application resources (CPU, Database connections, Thread pools). They are harder to detect because the traffic volume is often low and looks like valid HTTP.

### 7.1 HTTP Floods
A surge of GET or POST requests.
* **Targeting:** Attackers target "expensive" endpoints, such as search bars (hitting the DB), report generation (hitting CPU/Disk), or login pages (hitting hashing algorithms).
* **Cache Bypassing:** Attackers use random URL parameters (e.g., `?q=random123`) to ensure requests miss the CDN cache and hit the origin server directly.

### 7.2 Slowloris (Low and Slow)
Unlike floods, this attack uses minimal bandwidth.
* **Mechanism:** The attacker opens a TCP connection and sends a partial HTTP header:
    ```http
    GET / HTTP/1.1
    Host: target.com
    User-Agent: Mozilla...
    X-Keep-Alive: 300
    ```
* **The Hook:** The attacker never sends the final `\r\n\r\n` to complete the header. Instead, they send a random header line every 10 seconds to keep the connection alive.
* **Impact:** The server keeps the thread open, waiting for the request to finish. With just a few hundred connections, the attacker can exhaust the web server's maximum concurrent connection pool (e.g., Apache's `MaxClients`), blocking everyone else.


### 7.3 Algorithmic Complexity Attacks
Attacks that trigger worst-case performance scenarios in the code.
* **ReDoS (Regex DoS):** Sending a specific string that causes a Regular Expression engine to enter catastrophic backtracking (exponential time complexity $O(2^n)$).
* **Hash Collision Flooding:** Sending thousands of POST parameters with keys that hash to the same value (e.g., in Java or PHP). This degrades Hash Map lookups from $O(1)$ to $O(n)$ or $O(n^2)$, causing CPU saturation.
* **XML Bombs / Nested JSON:** sending deeply nested structures that consume massive RAM when parsed.

### 7.4 Crypto DoS
Abusing the asymmetry of cryptography. Verifying a signature or establishing a TLS connection is computationally cheaper for the client than the server.
* **TLS Handshake Floods:** Forcing the server to perform repeated expensive RSA/ECDHE calculations.
* **Renegotiation Attacks:** Continuously renegotiating the encryption key within a single connection.

---

## 8. Defenses and Mitigations

Defending against DoS requires a multi-layered architecture. No single box can stop all attacks.

### 8.1 Architecture & Network Design
* **Scrubbing Centers:** Divert traffic to a specialized provider (e.g., Cloudflare, Akamai) with massive bandwidth. They filter (scrub) the bad traffic and send only clean traffic to the origin.
* **Anycast:** Use Anycast routing to spread the attack traffic across multiple data centers globally, diluting its impact.
* **Hide the Origin:** Never expose the origin server's IP. Force all traffic through the CDN/WAF.

### 8.2 L3/L4 Mitigations
* **BCP-38 (Ingress Filtering):** ISPs should filter packets leaving their network. If a packet leaves a customer network with a source IP that doesn't belong to that network, drop it. This kills spoofing.
* **SYN Proxy:** A middlebox completes the TCP handshake with the client. Only when the connection is valid does it open a connection to the backend server.
* **Rate Limiting:** Prioritize traffic. Drop fragmented packets or specific UDP ports that are not used by the service.

### 8.3 Application Mitigations
* **Fail-Fast:** Enforce strict timeouts and limits on request sizes (headers, body). Don't let a connection hang for minutes.
* **Caching:** aggressive caching of static content and "stale-while-revalidate" strategies to keep serving content even if the backend is stressed.
* **Circuit Breakers:** Automatically stop sending requests to a failing backend component to prevent cascading failure.
* **CAPTCHAs:** Challenge/Response mechanisms to verify if the client is a human or a bot before allowing expensive operations (like login or search).

### 8.4 HTTP/2 & HTTP/3 Hygiene
* Cap the number of concurrent streams per connection.
* Monitor for "Rapid Reset" patterns (streams quickly opened and reset).
* Limit header table sizes to prevent compression bomb attacks.