# Wazuh SIEM Incident Analysis & Digital Forensics Investigation

## 1. Executive Summary
A Digital Forensics and Incident Response (DFIR) investigation was conducted on an Ubuntu Server hosting the **Dibimbing Shop** web application after **Wazuh SIEM** triggered high-frequency security alerts[cite: 20, 21]. The investigation focused on log correlation, digital evidence preservation, server configuration auditing, and PHP source code review to determine the root cause, scope of impact, and required hardening measures[cite: 21, 22].

While forensic analysis confirmed **no successful unauthorized login, privilege escalation, malware deployment, or data exfiltration**, the investigation uncovered two active external attack vectors and two critical security misconfigurations/vulnerabilities requiring immediate remediation[cite: 23, 28].

---

## 2. Investigation Scope & Environment
* **Target Infrastructure:** Ubuntu Server (`srv974671`) running Apache Web Server & OpenSSH[cite: 20, 21, 24]
* **Monitoring Platform:** Wazuh SIEM[cite: 22]
* **Evidence Sources Analyzed:**
  * Wazuh Security Alerts & Rule Definitions[cite: 21, 24]
  * Linux Authentication Logs (`/var/log/auth.log`)[cite: 22, 24]
  * Apache Web Server Access Logs & VirtualHost Configurations (`/etc/apache2/`)[cite: 22, 26]
  * Application PHP Source Code (`admin_area/product.php`)[cite: 22, 27]
* **Rules of Engagement:** Read-only forensic analysis; live exploitation and modification of digital evidence were strictly prohibited[cite: 21].

---

## 3. Summary of Forensic Findings

| ID | Finding / Event | Category | Severity | Detection Source |
| :--- | :--- | :--- | :---: | :--- |
| **INC-01** | SSH Brute Force Attack[cite: 23, 24] | Active Attack / Incident[cite: 23] | **High**[cite: 23, 24] | Wazuh Rule ID `5551` & `auth.log`[cite: 24] |
| **INC-02** | Web Vulnerability Scanning (`.env` Enumeration)[cite: 23, 25] | Reconnaissance[cite: 23, 25] | **Informational**[cite: 23, 25] | Wazuh Rule ID `31151` & Apache Log[cite: 25] |
| **MIS-01** | Apache Directory Listing Enabled[cite: 23, 26] | Misconfiguration[cite: 23] | **Medium**[cite: 23, 26] | Apache Config Audit (`/etc/apache2/`)[cite: 26] |
| **VULN-01** | Insufficient File Upload Validation[cite: 23, 27] | Application Vulnerability[cite: 23] | **High**[cite: 23, 27] | PHP Source Code Review[cite: 27] |

---

## 4. Detailed Technical Analysis

### [INC-01] SSH Brute Force Attack — High[cite: 24]
* **Alert & Log Correlation:** Wazuh triggered **Rule ID 5551** (*PAM: Multiple failed logins in a small period of time*)[cite: 24]. Cross-referencing `/var/log/auth.log` confirmed repeated failed SSH authentication attempts originating from source IP `171.231.177.90` targeting multiple usernames[cite: 24].
* **Forensic Conclusion:** No corresponding `Accepted password` or `Accepted publickey` events were found for the attacker's IP, confirming the brute-force attempt was unsuccessful[cite: 24, 28].

### [INC-02] Automated Web Vulnerability Scanning — Informational[cite: 25]
* **Alert & Log Correlation:** Wazuh triggered **Rule ID 31151** due to rapid HTTP requests from source IP `34.34.229.26` probing for exposed environment files (`/utils/.env`, `/scripts/.env`, `/log/.env`, `/cache/.env`)[cite: 25].
* **Forensic Conclusion:** Apache access logs verified that all requests returned HTTP `404 Not Found`, indicating automated reconnaissance with zero sensitive file exposure[cite: 25].

### [MIS-01] Apache Directory Listing Misconfiguration — Medium[cite: 26]
* **Configuration Audit:** Inspection of `/etc/apache2/apache2.conf` and site configurations revealed the `Options Indexes FollowSymLinks` directive was enabled, publicly exposing internal asset directories such as `/images/`[cite: 26].
* **Impact:** Facilitates attacker reconnaissance by revealing internal directory structures and stored files[cite: 26].

### [VULN-01] Insufficient File Upload Validation — High[cite: 27]
* **Source Code Review:** Manual review of `admin_area/product.php` identified insecure file handling using `move_uploaded_file()` without validating file extensions, MIME types, or file size[cite: 27]:
  ```php
  if (!empty($_FILES[$img_key]['name'])) {
      $img_name = basename($_FILES[$img_key]['name']);$img_tmp = $_FILES[$img_key]['tmp_name'];
      move_uploaded_file($img_tmp, "product_images/$img_name");
  }
  ```[cite: 27]
* **Impact:** Exposes the server to potential Remote Code Execution (RCE) via malicious web shell uploads if administrative access is compromised[cite: 27].

---

## 5. Remediation & Hardening Plan
1. **SSH Hardening:** Enforce SSH key-based authentication, disable direct `root` login (`PermitRootLogin no`), and implement IP whitelisting or Fail2Ban/Wazuh Active Response[cite: 28].
2. **Web Server Hardening:** Remove the `Indexes` directive across all Apache configuration files (`Options -Indexes`) and implement rate-limiting against automated scanners[cite: 28].
3. **Secure File Upload Implementation:** Enforce strict allow-listing for file extensions and MIME types (`finfo_file()`), rename uploaded files using randomized hashes, and disable PHP execution within the `product_images/` directory[cite: 28].

---

## 6. Full Investigation Presentation
📄 The complete forensic presentation and visual log evidence can be viewed in [`report/Wazuh-Incident-Investigation-Arjuna.pdf`](report/Wazuh-Incident-Investigation-Arjuna.pdf).
