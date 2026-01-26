# Web Technologies: Architecture, Protocols, and Security

**Tags:** #WebSecurity #HTTP #HTML #OWASP #SQLInjection #XSS #CSRF #CyberSecurity
**Source:** ![[3 - Web Technologies.pdf]]

---

## 1. Introduction and History

The World Wide Web (WWW) is an information space where documents and other web resources are identified by URLs, interlinked by hypertext links, and accessible via the Internet.

### 1.1 Timeline of Evolution
* **1969:** ARPANET is commissioned by the DoD for research into networking.
* **1980:** Tim Berners-Lee writes *Enquire*, a notebook program that allows links between arbitrary nodes.
* **1989:** Berners-Lee circulates "Information Management: A Proposal" at CERN.
* **1990:** The first web server (`nxoc01.cern.ch`) and the first browser (`WorldWideWeb`) are developed.
* **1993:** Mosaic, the first graphical web browser, is released by NCSA, popularizing the web.
* **1994:** Netscape Navigator is released; the W3C (World Wide Web Consortium) is founded to standardize protocols.

---

## 2. Web Architecture

The Web relies on a client-server architecture where a **User Agent** (browser) requests resources from a **Web Server**.

### 2.1 Core Components
1.  **URL (Uniform Resource Locator):** A global identifier for resources. It specifies the protocol (e.g., `http`), the host (e.g., `www.example.com`), and the path to the resource.
2.  **HTTP (HyperText Transfer Protocol):** The application-layer protocol used to transfer resources.
3.  **HTML (HyperText Markup Language):** The standard markup language for creating web pages.

### 2.2 The HTTP Protocol
HTTP operates on a simple Request/Response model.
* **Connection:** The client opens a TCP connection to the server (typically port 80).
* **Request:** The client sends a message requesting a resource.
* **Response:** The server returns the resource (or an error) and closes the connection.



**Characteristics:**
* **Stateless:** The server does not retain information about previous client requests. Each transaction is independent.
* **Text-Based:** Messages are human-readable (though often compressed in transit).

**HTTP Methods:**
* `GET`: Retrieve a resource. Parameters are passed in the URL (query string).
* `POST`: Submit data to be processed (e.g., form data). Parameters are passed in the message body.
* `HEAD`: Retrieve headers only (no body).

**Status Codes:**
The server responds with a 3-digit code indicating the result:
* `1xx`: Informational.
* `2xx`: Success (e.g., `200 OK`).
* `3xx`: Redirection (e.g., `301 Moved Permanently`).
* `4xx`: Client Error (e.g., `404 Not Found`, `403 Forbidden`).
* `5xx`: Server Error (e.g., `500 Internal Server Error`).

### 2.3 State Management (Cookies)
Because HTTP is stateless, **Cookies** were introduced to maintain state (sessions) across multiple requests.
* **Mechanism:** The server sends a `Set-Cookie` header. The client stores it and sends it back in the `Cookie` header for subsequent requests to the same domain.
* **Types:**
    * *Session Cookies:* Stored in memory; deleted when the browser closes.
    * *Persistent Cookies:* Stored on disk with an expiration date.

---

## 3. Web Content Technologies

Web content has evolved from static text to dynamic, interactive applications.

### 3.1 Static Content
* **HTML:** Defines the structure of the page using tags (e.g., `<h1>`, `<p>`, `<a>`).
* **CSS (Cascading Style Sheets):** Describes the presentation (layout, colors, fonts) separately from the structure.

### 3.2 Client-Side Scripting
**JavaScript** allows code to run in the browser, enabling dynamic behavior.
* **DOM (Document Object Model):** An API that represents the HTML document as a tree structure. JavaScript can manipulate the DOM to change content without reloading the page.
* **AJAX (Asynchronous JavaScript and XML):** Allows the browser to send/receive data asynchronously, updating parts of the page without a full reload.

### 3.3 Server-Side Technologies
Dynamic pages are generated on the fly by the server.
* **CGI (Common Gateway Interface):** An early standard where the web server executes a separate program (script) to generate content. It was inefficient as it created a new process for every request.
* **Embedded Scripting:** Technologies like PHP, JSP (Java), and ASP (Microsoft) embed code directly into HTML files. The server processes this code before sending the result to the client.
* **Servlets:** Java classes that dynamically process requests and construct responses.

---

## 4. Web Security Vulnerabilities

The complexity of modern web applications has introduced numerous security risks. The **OWASP Top 10** is a standard awareness document for the most critical security risks.

### 4.1 [[4 Web Security part I#5. SQL Injection (SQLi)|SQL Injection (SQLi)]]
SQL Injection occurs when untrusted data is sent to an interpreter as part of a command or query. An attacker can trick the database into executing unintended commands.

* **Mechanism:** If an application constructs queries by string concatenation (e.g., `SELECT * FROM users WHERE name = '` + input + `'`), an attacker can input `' OR '1'='1`. The resulting query becomes `SELECT * FROM users WHERE name = '' OR '1'='1'`, which is always true, bypassing authentication.
* **Impact:** Data loss, unauthorized access, or [[7 DoS Attacks|denial of service]].
* **Defense:**
    * **Prepared Statements (Parameterized Queries):** Ensures the database treats user input as data, not executable code.
    * Input Validation and Sanitization.



### 4.2 [[5 Web Security part II#3. Cross-Site Scripting (XSS)|Cross-Site Scripting (XSS)]]
XSS flaws occur whenever an application includes untrusted data in a web page without proper validation or escaping. This allows attackers to execute malicious scripts in the victim's browser.

**Types of XSS:**
1.  **Reflected XSS:** The malicious script is part of the request (e.g., in a URL parameter) and is "reflected" back by the server in the response. It typically requires a phishing link.
2.  **Stored XSS (Persistent):** The malicious script is stored on the server (e.g., in a forum post). Every user who views the page executes the script.
3.  **DOM-based XSS:** The vulnerability exists in client-side code rather than server-side code. The attack payload is executed by modifying the DOM "environment" in the victim's browser.

**Defenses:**
* **Context-Sensitive Output Encoding:** Convert special characters into their HTML entity equivalents (e.g., `<` becomes `&lt;`) before rendering.
* **Content Security Policy (CSP):** An HTTP header that restricts the sources from which the browser is allowed to load resources.



### 4.3 [[5 Web Security part II#5. Cross-Site Request Forgery (CSRF)|Cross-Site Request Forgery (CSRF)]]
CSRF forces a logged-in victim's browser to send a forged HTTP request, including the victim's session cookie and other authentication information, to a vulnerable web application.

* **Mechanism:** An attacker embeds a malicious link or form in a page they control (e.g., `<img src="http://bank.com/transfer?to=attacker&amount=1000">`). If the victim visits this page while logged into the bank, the browser automatically sends the request with the victim's cookies.
* **Defense:**
    * **Anti-CSRF Tokens:** Include a unique, unpredictable token in every state-changing form/request. The server verifies this token before processing the request.
    * SameSite Cookie Attribute.

### 4.4 Other Injection Flaws
* **[[4 Web Security part I#4.1 Command Injection (OS Level)|Command Injection]]:** Occurs when user input is passed to system shells (e.g., `exec()`, `system()`). Attackers can execute OS commands (e.g., inputting `; rm -rf /`).
* **[[4 Web Security part I#3. Path Traversal (Directory Traversal)|Path Traversal]]:** Manipulating file paths (e.g., `../../etc/passwd`) to access files outside the intended directory.

---

## 5. Security Best Practices Summary

| Vulnerability | Key Mitigation Strategy |
| :--- | :--- |
| **SQL Injection** | Use **Prepared Statements** (Parameterized Queries). Do not concatenate strings for SQL. |
| **XSS** | **Escape/Encode** untrusted data based on output context (HTML, JS, CSS). Use **CSP**. |
| **CSRF** | Use **Anti-CSRF Tokens** (Synchronizer Token Pattern). |
| **Command Injection** | Avoid system calls; use library functions. Validate input against a whitelist. |
| **Data Exposure** | Encrypt data at rest and in transit (TLS). |