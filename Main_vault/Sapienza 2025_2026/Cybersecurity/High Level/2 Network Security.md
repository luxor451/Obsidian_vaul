# Network Security: Intrusion Detection, Segmentation, and SIEM

**Tags:** #NetworkSecurity #IDS #IPS #ZeroTrust #SIEM #CyberSecurity #VLAN #Malware
**Source:** ![[2 - Network security.pdf]]

---

## 1. Introduction to Intrusion Detection

In the modern landscape of cybersecurity, shielding mechanisms like firewalls and VPNs are no longer sufficient to guarantee 100% protection. The rise of new attack techniques, "silent" vulnerabilities, misconfigurations, and malicious insiders necessitates a proactive approach. Security must involve continuous monitoring to detect unauthorized activities.

**Intrusion Detection** is defined by NIST as the process of monitoring events occurring in a computer system or network and analyzing them for signs of intrusions. An intrusion is defined as an attempt to compromise the confidentiality, integrity, or availability of a resource, or to bypass the security mechanisms of a computer or network.

### 1.1 Attacks vs. Intrusions
It is important to distinguish between an attempt and a success:
* **Attack:** An intrusion attempt. It may be blocked by effective shielding.
* **Intrusion:** A successful attack. It represents a malicious, externally induced operational fault.

---

## 2. Taxonomy of Attacks

Attacks can be classified based on several dimensions, including the type of attack, the network connections involved, the source, the environment, and the level of automation.

### 2.1 Classification by Attack Type

**1. [[7 DoS Attacks|Denial of Service (DoS)]]**
The goal of a DoS attack is to shut down a network, computer, or process, denying the use of resources to authorized users. It is a direct attack on **availability**.
* **Strategies:** Consumption of scarce resources (bandwidth, disk space, CPU), disrupting network connectivity, or physical destruction/alteration of network components.

**2. Probing and Scanning**
The goal is to identify valid IP addresses within a domain and collect information about them, such as open ports, operating systems, and running services.
* **Significance:** This is often a precursor to a more targeted attack. It provides the attacker with a list of potential vulnerabilities (e.g., outdated services).
* **Tools:** Nmap, IPsweep, Portsweep.
* **Detection:** Fast, noisy scans are easy to detect (e.g., looking for N connections in T seconds). Stealthy, slow scans are much more challenging to identify.

**3. Compromises**
The goal is to break into a system to gain privileged access.
* **R2L (Remote to Local):** An attacker capable of sending packets to a machine (but without an account) gains access, often by exploiting vulnerabilities like buffer overflows or password guessing.
* **U2R (User to Root):** An attacker who already has a standard user account exploits vulnerabilities in the OS or local programs to elevate privileges to administrator (root) level.

**4. Malware (Viruses, Worms, Trojans, Rootkits)**
* **Viruses:** Programs that persist by attaching themselves to other executables. They typically require human interaction (e.g., clicking a file) to replicate.
* **Worms:** Self-replicating programs that aggressively spread through a network without human intervention. They can spread via email, file sharing, or direct network connections.
* **Trojan Horses:** Malicious programs disguised as benign software. The user executes the attack on purpose, believing they are installing legitimate software.
* **Rootkits:** Software used to maintain illicit access to a compromised machine (persistence). They often operate at the OS level to hide their presence and may open backdoors (RATs) for the attacker.

### 2.2 Other Classification Dimensions
* **Automation:**
    * *Automated:* Tools scan the internet indiscriminately.
    * *Semi-automated:* Scripts scan and compromise, while a human handler directs the specific targets.
    * *Manual:* High effort, requires deep knowledge, usually targeted and hard to detect.
* **Source:** Single source (e.g., port scan) vs. Multiple sources (e.g., [[7 DoS Attacks#4.2 Distributed Denial of Service (DDoS)|DDoS]]).
* **Environment:** Host, Network, Wireless, or P2P environments.

---

## 3. Monitoring Infrastructure

To detect attacks, IT systems must generate monitoring data. There are three primary sources of data used for security analysis:


### 3.1 Network Flows
Traffic flows are **metadata** records of communication sessions. They capture the "envelope" of the conversation but not the content (payload).
* **The 5-Tuple:** The basic identification criteria for a flow are Source IP, Destination IP, Source Port, Destination Port, and Protocol (TCP/UDP).
* **Additional Data:** Byte counts, packet counts, start/end times, TCP flags (SYN, ACK, FIN), and Autonomous System (AS) numbers.

### 3.2 Event and System Logs
Assets produce logs regarding their internal activities.
* **Sources:** Servers, workstations, network appliances, and applications.
* **Value:** Error logs, such as failed authentication attempts, are high-value indicators of malicious activity.

### 3.3 Full Packet Capture (FPC)
FPC captures the actual **payload** of network flows.
* **Utility:** Allows analysts to reconstruct attacks perfectly, analyze malware payloads, or recover extracted data. It is essential for digital forensics.
* **Limitations:** It requires massive storage and high-performance hardware. It is impractical to record everything; usually, FPC is triggered only by alerts or deployed on critical segments.

---

## 4. Intrusion Detection Systems (IDS)

An IDS follows a general framework: sensors collect data from the monitored system, an analysis engine (supported by a knowledge base) processes events, and a response component triggers alarms or actions based on the system state.


### 4.1 Desired Characteristics (KPIs)
* **Detection Rate (True Positive Rate):** The percentage of actual attacks detected.
* **False Alarm Rate (False Positive Rate):** The percentage of normal connections incorrectly flagged as attacks.
* **Accuracy:** The overall correctness of the system (attacks detected + normal events ignored).
* **Precision:** The trustworthiness of an alert.
* **Timeliness:** The total time from the occurrence of the intrusion to the triggering of the alert (includes processing and propagation time).
* **Fault Tolerance:** The ability of the IDS to resist attacks against itself (e.g., flooding the IDS with noise to mask a real attack).

**ROC Curve:** The Receiver Operating Characteristic curve illustrates the trade-off between the detection rate and the false alarm rate. A perfect IDS would have a 100% detection rate with 0% false alarms.

![[Pasted image 20260122194238.png]]


### 4.2 Categorization of IDSs

#### By Monitored System
1.  **Host-Based (HIDS):** Monitors a single host (logs, file systems, processes).
    * *Pros:* Can analyze encrypted traffic (after decryption on the host), file integrity, and code execution sandbox.
    * *Cons:* Requires complex tuning, consumes host resources, and lacks network context.
    * *Evolution:* **EDR (Endpoint Detection and Response)** is the active evolution of HIDS, offering automated response (isolation, process killing) and centralized aggregation.
2.  **Network-Based (NIDS):** Monitors traffic segments (e.g., at a gateway or DMZ).
    * *Deployment:* Can be **In-line** (all traffic passes through it, enables blocking) or **Passive** (monitors a copy/mirror of traffic).
    * *Pros:* Stealthy (no IP address), low impact on existing systems.
    * *Cons:* Cannot analyze encrypted traffic without a proxy; high resource usage for deep inspection.
3.  **Wireless (WIDS):** Monitors wireless medium for rogue access points, de-authentication attacks, and MAC spoofing. Deployment is difficult due to imprecise physical boundaries.

#### By Detection Methodology
1.  **Misuse Detection (Signature-Based):**
    * Uses a database of known attack "fingerprints" (signatures).
    * *Pros:* Low false positives for known attacks.
    * *Cons:* Cannot detect zero-day (unknown) attacks; requires constant updates.
    * *Examples:* Snort, Suricata, AntiVirus.
2.  **Anomaly Detection (Behavioral):**
    * Models the "normal" behavior of the system; deviations are flagged as attacks.
    * *Methods:* Statistical models, Machine Learning (Neural Networks, HMMs), Rule-based baselines, Profiling.
    * *Pros:* Can detect unknown attacks.
    * *Cons:* High false positive rate (normal behavior can change).
3.  **Compound Detection:** Maintains models for both normal and abnormal behaviors and compares events against both.


#### By Reaction Type
* **Passive (IDS):** Reports alarms to administrators.
* **Active (IPS - Intrusion Prevention System):** Performs non-destructive reactions like patching or firewall rule injection, or aggressive actions like dropping packets, terminating connections (sending TCP Reset), or throttling traffic.

---

## 5. Network Defense Strategies

### 5.1 The Decline of "Border Defense"
Traditional network design relies on "Internal Trust": a hard outer shell (firewall) protects a soft trusted interior. This is flawed because modern boundaries are blurred (Cloud, Remote Work) and once the perimeter is breached, attackers move freely.

### 5.2 Zero Trust
Zero Trust is a security model assuming **no user or device is trusted by default**, regardless of location.
* **Principle:** "Never trust, always verify."
* **Key Components:**
    * **Verify Explicitly:** Use all data points (identity, location, device health) for every access request.
    * **Least Privilege:** Restrict permissions to the bare minimum.
    * **Assume Breach:** Design the network to contain breaches.
* **Implementation:** Requires strong Identity and Access Management (IAM), extensive monitoring, and microsegmentation.

### 5.3 Network Segmentation
Segmentation divides the network into isolated protection domains to limit lateral movement.

1.  **Physical Segmentation:**
    * Hardware-level separation (e.g., air gaps, **Data Diodes** which allow one-way traffic only).
    * *Pros:* Extremely secure.
    * *Cons:* Inflexible, expensive, leads to device proliferation.
    
2.  **Logical Segmentation (VLANs):**
    * Software-defined boundaries using Layer 2 switches (802.1Q).
    * **VLANs (Virtual Local Area Networks):** Logically multiplex broadcast domains on shared infrastructure. Traffic between VLANs must pass through a Layer 3 device (Router/Firewall) where ACLs can be applied.
    * *Pros:* Flexible, dynamic.
    * *Cons:* Software vulnerabilities in switches can break isolation.
    
3.  **Microsegmentation:**
    * Fine-grained isolation at the workload or application level.
    * Crucial for Zero Trust. Policies can be based on IP, Application ID, User Identity, or Process.
    * *Pros:* Limits lateral movement effectively in cloud/hybrid environments.
    * *Cons:* Complex policy management.

---

## 6. Security Information and Event Management (SIEM)

In a complex infrastructure with HIDS, NIDS, Firewalls, and Servers all generating logs, a unified view is required.

**SIEM** is a management layer positioned above existing security controls. It collects, aggregates, and correlates low-level data to produce high-level actionable intelligence.

### 6.1 The SIEM Process
1.  **Input:** Logs (from servers, apps), Network Flows, Security Alerts (from IDS/IPS).
2.  **Processing:**
    * **Context:** Adds business context (e.g., "This IP belongs to the Finance Server").
    * **Knowledge:** Applies threat intelligence and correlation rules.
3.  **Output:** Actionable Intelligence (Prioritized Alarms, Reports).


### 6.2 Example Scenario
Consider a log entry showing a database copy initiation.
* *Without SIEM:* It looks like a standard data transfer.
* *With SIEM:* The system correlates the source IP (Accounting IT), the User ID (Sales Sync Account), and the Destination (Remote Host in Boston). The SIEM recognizes that the "Accounting IT" machine should not be using the "Sales Sync" credentials to move data to an external host, flagging it as a valid business activity or a potential anomaly based on pre-defined policy and context.

**Benefits of SIEM:**
* **Unified Visibility:** Single pane of glass for security status.
* **Incident Response:** Real-time detection and prioritization.
* **Compliance:** centralized logging for regulatory reporting.