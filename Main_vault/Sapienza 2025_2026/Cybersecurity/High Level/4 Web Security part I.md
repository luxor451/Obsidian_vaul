# Web Security (Part I): Vulnerabilities, Injection Attacks, and Mitigations

**Tags:** #WebSecurity #CyberSecurity #OWASP #SQLInjection #RCE #SSRF #PathTraversal #AppSec #PenetrationTesting 
**Source:** ![[4 - Web Security (part I).pdf]]

---

## 1. The Landscape of Software Vulnerabilities

### 1.1 Defining Vulnerabilities
In the context of information security, a **vulnerability** is not merely a bug; it is a specific weakness that allows an attacker to reduce a system's information assurance. A true vulnerability exists at the intersection of three critical elements:
1.  **System Susceptibility:** A flaw, design defect, or bug must exist within the system.
2.  **Attacker Access:** An attacker must have a pathway to reach the flawed component.
3.  **Attacker Capability:** The attacker must possess the necessary tools, knowledge, or technique to exploit the flaw.

### 1.2 Causes and Fostering Factors
Vulnerabilities are rarely intentional. They arise from:
* **Implementation Bugs:** Coding errors (e.g., buffer overflows, race conditions).
* **Design Defects:** Flaws in the fundamental logic or architecture of the system.
* **Misconfigurations:** Insecure default settings or improper deployment parameters.
* **Software Aging:** Failure to update libraries or manage legacy code, which leads to "silent" vulnerabilities over time.

Several factors exacerbate these issues:
* **System Complexity:** As lines of code increase, the probability of bugs rises significantly. Estimates suggest there are approximately **20 flaws per 1000 lines of code**.
* **Connectivity:** The internet exposes systems to a global threat landscape, increasing the "Attacker Access" factor.
* **Incompetence:** A general lack of security training among developers often leads to avoidable mistakes.

### 1.3 The Vulnerability Life Cycle
The life cycle of a vulnerability determines the window of exposure and who bears the cost:
1.  **Creation:** The flaw is introduced during the development phase.
2.  **Discovery:** The flaw is found. This can be by "black hats" (malicious actors) or "white hats" (security researchers).
3.  **Exploitation:** Attacks begin. If this happens *before* a patch exists, it is known as a **Zero-Day** exploit.
4.  **Disclosure:** The vulnerability becomes public knowledge.
5.  **Patch Availability:** The vendor releases a fix.
6.  **Patch Application:** Users update their systems (often the slowest step in the chain).
7.  **Software EoL:** End of Life, where no further patches are issued.

**Disclosure Policies:**
The industry debates how vulnerabilities should be reported:
* **Full Vendor Disclosure:** Researchers report privately to the vendor. The vendor controls the timeline, promoting secrecy but potentially leaving users vulnerable if the vendor drags their feet.
* **Immediate Public Disclosure:** Researchers publish details immediately. This forces the vendor to act but exposes all users to immediate attacks (0-day).
* **Hybrid/Responsible Disclosure:** A middle ground (e.g., Google Project Zero's 90-day deadline). Vendors are given a specific grace period to fix the issue before it is automatically made public.

### 1.4 The Economics of Exploits
There is a thriving, lucrative market for vulnerabilities. Companies like **Zerodium** pay massive bounties for exclusive rights to high-impact exploits, often selling them to government agencies:
* **Mobile RCE (Android/iOS):** Up to **$2,500,000**.
* **Desktop/Server (Windows RCE):** Up to **$1,000,000**.
* **Browser RCE (Chrome/Safari):** Highly valued due to the difficulty of escaping modern sandboxes.
* **Server Software:** Exploits for Apache, Microsoft IIS, and OpenSSL can command hundreds of thousands of dollars.

---

## 2. Web Architecture and Threat Models

The web has shifted from simple document retrieval to complex, interactive applications. This complexity moves the security perimeter from the network firewall to the application layer itself.

### 2.1 The Attacker Taxonomy
* **Malware Attacker:** Operates directly on the victim's machine. They can bypass network protections because they are already "inside" (e.g., a virus modifying the browser or installing a root certificate).
* **Network Attacker:**
    * *Passive:* Eavesdrops on traffic (e.g., sniffing unencrypted HTTP on public Wi-Fi).
    * *Active:* Intercepts and modifies traffic (e.g., [[6 DNS Security#3. DNS Spoofing and Cache Poisoning|DNS poisoning]], Evil Twin attacks, Man-in-the-Middle).
* **Web Attacker:**
    * Controls a malicious site (e.g., `attacker.com`).
    * Exploits the trust relationship between the user and other sites.
    * **Gadget Attacker:** Uses an iframe to load malicious content within a legitimate page.
    * **Related-Domain Attacker:** Controls `attacker.example.com` to attack `target.example.com`.

### 2.2 Defense Strategies
Security relies on a **Defense in Depth** approach, layering multiple countermeasures:

#### Client-Side Defenses
* **Sandboxing:** Browsers isolate tabs and processes to prevent a crash or exploit in one site from affecting others or the underlying OS.
* **Site Isolation:** Ensures pages from different websites are put into different processes.
* **Note:** Older mechanisms like "XSS Filters" have been largely removed from modern browsers because they occasionally introduced new vulnerabilities (e.g., XS-Search).

#### Server-Side Defenses
* **Input Validation:** The first line of defense. Sanitizing all user data.
* **Prepared Statements:** The standard defense against SQL Injection.
* **Web Application Firewalls (WAF):** Specialized firewalls that inspect HTTP traffic for attack signatures.
* **[[5 Web Security part II#5. Cross-Site Request Forgery (CSRF)|CSRF Tokens]]:** Random values used to verify that a request originated from a legitimate form.

#### Hybrid/Protocol Defenses
* **[[5 Web Security part II#7. Secured Transport (HTTPS and HSTS)|HTTPS/HSTS]]:** Enforces encryption to defeat network attackers.
* **[[5 Web Security part II#4.2 Content Security Policy (CSP)|Content Security Policy (CSP)]]:** A header that restricts where the browser can load resources (scripts, images) from, neutralizing many XSS attacks.
* **[[5 Web Security part II#2. The Same Origin Policy (SOP)|Same-Origin Policy (SOP)]]:** The fundamental security model of the web, preventing script access across different domains.
* **Cookie Attributes:** `Secure` (HTTPS only), `HttpOnly` (inaccessible to JS), `SameSite` (CSRF protection).

---

## 3. Path Traversal (Directory Traversal)

**Path Traversal** (or Directory Traversal) is a vulnerability where an application allows user input to manipulate file paths, enabling access to files outside the intended directory (e.g., the web root).

### 3.1 The Mechanism
Web servers typically restrict access to a specific folder (e.g., `/var/www/html`). However, if an application blindly concatenates user input to a file path, attackers can use the dot-dot-slash sequence (`../`) to "climb up" the directory tree.

**Vulnerable PHP Example:**
```php
$page = $_GET['page'];
// Intended usage: pages/contact.txt
// Attack usage: pages/../../../etc/passwd
echo file_get_contents("pages/" . $page);
```
If the input is `../../../etc/passwd`, the server resolves the path relative to the root, exposing the system's user list (password file).

### 3.2 CTF Example: Challenge #02
* **Scenario:** A web app displays a flag based on the user's language preference.
* **Code Logic:**
    1.  Reads the `HTTP_ACCEPT_LANGUAGE` header (e.g., `en-US`).
    2.  Sanitizes input by replacing `../` with an empty string `""`.
    3.  Loads the file: `file_get_contents("flags/" . $lang)`.
* **The Flaw:** The sanitization is not recursive. It only runs once.
* **The Exploit:** An attacker sends a header like `....//`.
    * The sanitizer removes the inner `../`.
    * The remaining characters collapse to form a *new* `../`.
    * **Payload:** `....//....//....//flag`.
* **Lesson:** HTTP headers are user input and must be treated as untrusted. "Blacklisting" specific strings is often ineffective against clever encoding or manipulation.

### 3.3 Prevention
1.  **Avoid Direct File Access:** Use database IDs or mapped keys instead of filenames (e.g., `?page=1` maps to `contact.txt`).
2.  **Canonicalization:** Use functions like `realpath()` to resolve the path to its absolute form *before* checking it.
3.  **Validation:** Ensure the resolved path starts with the expected base directory.
    ```php
    $base = "/var/www/html/pages/";
    $path = realpath($base . $_GET['file']);
    if ($path && strncmp($path, $base, strlen($base)) === 0) { ... }
    ```
4.  **Least Privilege:** Run the web server in a `chroot` jail so it physically cannot access the OS root.

---

## 4. Command and Code Injection

These vulnerabilities allow attackers to execute arbitrary commands or code on the server, often leading to full system compromise.

### 4.1 Command Injection (OS Level)
This occurs when user input is passed to a system shell command.
* **Vulnerable Function:** `system()`, `shell_exec()`, `exec()`.
* **Example:** A script that pings an IP address.
    ```php
    system("ping -c 4 " . $_GET['ip']);
    ```
* **Attack:** The attacker uses shell separators (`;`, `|`, `&&`, `$()`) to inject a second command.
    * Input: `127.0.0.1; cat /etc/passwd`.
    * Result: The server pings localhost, *then* prints the password file.

### 4.2 Code Injection (Application Level)
This occurs when the application evaluates user input as code within the programming language (PHP, Python, etc.).
* **Vulnerable Function:** `eval()`.
* **Example:** A calculator app.
    ```php
    eval("echo " . $_GET['expr'] . ";");
    ```
* **Attack:** The attacker inputs valid PHP code.
    * Input: `file_get_contents('/etc/passwd')`.
    * Result: The code is executed by the interpreter.

### 4.3 Case Study: Moodle RCE (2018)
A real-world example in the Moodle e-learning platform.
* **Feature:** Teachers could create math quizzes with calculated questions.
* **Vulnerability:** The function evaluating the math formulas used `eval()`.
* **Exploit:** A malicious user (or attacker with compromised credentials) could inject PHP code into the quiz dataset definition. This code would execute whenever the quiz was accessed, granting full control over the server.

### 4.4 CTF Example: Challenge #03 (Smart Cat)
* **Scenario:** A "Smart Cat" debug interface that allows one ping.
* **Constraint:** The interface blocks spaces and standard separators.
* **Bypass:** Attackers must use alternative shell tricks.
    * Newline injection: `%0a`.
    * Using `$IFS` (Internal Field Separator) instead of spaces.
    * Payload: `127.0.0.1%0acat$IFS/flag`.

### 4.5 Prevention
* **Avoid Dangerous Functions:** Never use `eval()`. It is almost always a security risk.
* **Use Native Libraries:** Instead of shelling out to `ping`, use a language-native network library.
* **Input Sanitization:** If shell execution is unavoidable, use `escapeshellcmd()` or whitelist specific characters (e.g., only allow `[0-9.]` for IPs).

---

## 5. SQL Injection (SQLi)

SQL Injection occurs when untrusted user data is concatenated directly into a database query string, allowing the attacker to alter the query's logic.

### 5.1 Authentication Bypass
* **Code:** `SELECT * FROM users WHERE user='$u' AND pass='$p'`.
* **Attack:** User inputs `admin' --`.
* **Result:** `SELECT * FROM users WHERE user='admin' --' AND pass='...'`
    * The `'` closes the user string.
    * The `--` comments out the rest of the query (the password check).
    * **Outcome:** Logged in as admin without a password.

### 5.2 UNION-Based Injection
Used to retrieve data from other tables when the query results are visible.
* **Mechanism:** The `UNION` operator combines results from two `SELECT` statements.
* **Constraint:** Both queries must have the same number of columns.
* **Attack:** `' UNION SELECT username, password FROM users --`.
* **Outcome:** The application displays the original content (e.g., products) followed by the dumped user credentials.

### 5.3 Stacking Queries
Allows executing multiple distinct SQL statements in one request (requires specific DB configurations).
* **Attack:** `'; DROP TABLE users; --`.
* **Outcome:** The original query runs, followed immediately by the destructive `DROP TABLE` command.

### 5.4 Blind SQL Injection
Used when the application does not return database errors or data, but reacts differently to True/False conditions.
* **Boolean-Based:**
    * Injection: `id=1 AND substring(password, 1, 1) = 'a'`.
    * Observation: Does the page load normally (True) or return 404/Empty (False)?
* **Time-Based:**
    * Injection: `id=1; IF(substring(pass,1,1)='a') WAITFOR DELAY '0:0:10'`.
    * Observation: If the page takes 10 seconds to load, the first character is 'a'.

### 5.5 Second-Order SQL Injection (Stored)
The vulnerability lies not in the input, but in using previously stored data safely.
1.  **Step 1:** Attacker registers as user `admin' --`. The database stores this string.
2.  **Step 2:** A later process (e.g., password change) retrieves this username.
    * Query: `UPDATE users SET pass='123' WHERE user='$stored_user'`
3.  **Step 3:** The query becomes `... WHERE user='admin' --'`, changing the *real* admin's password.

### 5.6 CTF Example: Challenge #04 (CrimeMail)
* **Scenario:** A password hint feature susceptible to SQLi.
* **Recon:** Using `UNION SELECT` to dump `information_schema.columns` reveals the table structure (`users`, `pass_md5`, `pass_salt`).
* **Exploit:** Dump the MD5 hash and salt for the user "Collins Hackle".
* **Cracking:** The hint implies a weak password. A Python script using a dictionary (rockyou.txt) + the salt can crack the MD5 hash to recover the plaintext password.

### 5.7 Prevention
* **Prepared Statements (Parameterized Queries):** The only robust defense. The database engine treats the query template and the data separately.
    ```php
    $stmt = $db->prepare('SELECT * FROM users WHERE user = ?');
    $stmt->execute([$_POST["user"]]);
    ```
    Even if the input contains `' OR 1=1`, it is treated as a literal string, not code.
* **Least Privilege:** The database user used by the web app should not have permissions to `DROP TABLE` or access sensitive system tables.

---

## 6. Server-Side Request Forgery (SSRF)

SSRF is a vulnerability where an attacker forces the web server to make requests to unintended locations. This effectively turns the web server into a proxy for the attacker.

### 6.1 The Attack Surface
Modern apps often fetch URLs for legitimate reasons (webhooks, image imports, PDF generation).
* **Internal Network Recon:** Attackers can target private IP addresses (`127.0.0.1`, `192.168.x.x`) to access internal admin panels, databases, or intranets that are not exposed to the public internet.
* **Cloud Metadata:** In AWS, Google Cloud, and Azure, a special internal IP (`169.254.169.254`) is used by the instance to retrieve configuration data.
    * **Attack:** `http://169.254.169.254/latest/meta-data/`
    * **Impact:** Attackers can retrieve the server's IAM credentials and take over the cloud account.

### 6.2 Blind SSRF
The application performs the request but does not return the response body to the user.
* **Detection:** Attackers use an external logging server (like Burp Collaborator). If the server connects to the attacker's URL, the vulnerability exists.
* **Timing Attacks:** Scanning internal ports by measuring how long the server takes to respond (e.g., a connection to a closed port might fail instantly, while an open port takes longer).

### 6.3 CTF Example: Challenge #05
* **Scenario:** A Python Flask app that scans URLs.
* **Code Flaw:** The app uses Python 2's `urllib`. It attempts to block dangerous schemes like `file://` and `gopher://`.
* **Vulnerability 1 (Logic):** The signature check looks for the string "scan" inside the cookie but doesn't verify exact equality (`if "scan" in action`). Attackers can append "scan" to other actions.
* **Vulnerability 2 (Library Behavior):** In Python 2 `urllib`, passing a local path (e.g., `./flag.txt`) without a protocol automatically works as a file read.
* **Exploit:**
    1.  Generate a valid signature for `flag.txtread`.
    2.  Send a request with `action=readscan` (contains "scan" to pass check).
    3.  The server reads the local flag file thinking it's a URL.

### 6.4 Prevention
* **Allowlisting:** Strictly define which domains or IPs the server can contact (e.g., only `api.google.com`).
* **Disable Unused Schemas:** Configure the HTTP client to support only `http` and `https`, explicitly disabling `file://`, `gopher://`, `ftp://`.
* **[[Network Security#5.3 Network Segmentation|Network Segmentation]]:** Isolate web servers in a DMZ. Firewall rules should prevent the web server from initiating connections to the internal intranet or the cloud metadata service.