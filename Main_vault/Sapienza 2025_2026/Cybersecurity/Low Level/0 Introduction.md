**Tags:** #Cybersecurity #LowLevel #InfoSec #Syllabus #Logistics #CIA #SafetyVsSecurity #Exam

---

## 1. Logistics and Resources

**Digital Platforms:**
* **Google Classroom:** Code `baomarw4`.
* [**Wnebsite:**](https://sites.google.com/diag.uniroma1.it/cybersecurity/) 
    * Contains the Calendar, Textbook info, and the **Log**.
    * **Crucial:** The "Log 2025" on the website is the official reference for the course content.

**Materials:**
* **Slides:** Will be available both in advance (tentatively) and in retrospect.
* **Warning:** The versions may differ slightly, so tracking the Log is essential.

**Lecture Organization:**
* **Modality:** Lectures are held **only in presence**.
* **Timing Proposal:** The standard lecture slot is divided as follows:
    * **Part 1:** 67.5 minutes
    * **Break:** 30 minutes (Thursday break might be shorter)
    * **Part 2:** 67.5 minutes

---

## 2. Exam and Grading

The course is split into two parts, and the final grade is a weighted combination of both.

**The Exam Structure:**
1.  **Written Test:** **Mandatory** for everyone.
2.  **Oral Exam:** **Optional**.
    * *Risk:* The oral exam can result in a positive grade adjustment or a *negative* one.

**Grading Formula:**
The final grade is calculated based on the weight of the ECTS credits:
~~~math
Final Grade = (2/3) * Grade_dAmore + (1/3) * Grade_Querzoni
~~~

**Administrative Rules:**
* **Partial Grades:** The system handles partial grades, even for students not yet formally enrolled in the Master's degree (e.g., waiting for Bachelor graduation).
* **Reset Policy:** Grades are valid until the end of the academic year; a reset occurs after the **September call**.

---

## 3. Syllabus (Tentative 2025)

The syllabus is expected to follow the 2024 structure, focusing on the theoretical foundations of security and cryptography.

1.  **Information Security:** Introduction to the field, definitions, and scope.
2.  **Symmetric Encryption:** Cryptography where the same key is used for encryption and decryption.
3.  **Data Integrity:** Mechanisms to ensure data has not been altered (hashing).
4.  **Public Key Encryption:** Asymmetric cryptography (pairs of public/private keys).
5.  **Digital Signatures and PKI:**
    * Using crypto to verify authenticity.
    * **PKI (Public Key Infrastructure):** The system managing certificates and trust.
6.  **CSPRNG:** Cryptographically Secure Pseudo-Random Number Generators (essential for key generation).
7.  **Authentication:** verifying the identity of entities.
8.  **Secret Sharing:** Techniques to split a secret among participants (e.g., Shamir's Secret Sharing) so it can only be reconstructed by a quorum.
9.  **Access Control:** Policies and mechanisms to restrict access to resources.
10. **Secure Protocols:**
    * **TLS (Transport Layer Security)**
    * **IPSec (Internet Protocol Security)**
    * **SSH (Secure Shell)**
11. **[[16 Firewalls|Firewalls]]:** Network security barriers.

---

## 4. Fundamental Concepts

The course starts by defining what it actually means to be "Secure".

**The CIA Triad:**
The three pillars of Information Security requirements:
1.  **Confidentiality:** Only authorized users can read the data.
2.  **Integrity:** The data cannot be modified without detection.
3.  **Availability:** The system/data is available when needed.



**Safety vs. Security:**
A critical distinction made early in the course:
* **Safety:** Protecting the system against accidental failures or environmental hazards (reliability).
* **Security:** Protecting the system against **intentional, malicious attacks** (adversaries).

**Thesis Opportunities:**
The instructor noted that there are "Many and demanding" Master Theses available in this field for interested students.