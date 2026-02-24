## 1. Executive Summary
The compromise of the DC-1 machine was achieved by exploiting a critical SQL Injection vulnerability in Drupal 7 (**CVE-2014-3704**). This allowed for the creation of an administrative user, leading to Remote Code Execution (RCE). Final privilege escalation was performed via a misconfigured SUID bit on the `find` binary.

## 2. Reconnaissance & Enumeration
- **Network Scanning:** `nmap -sC -sV` identified port 80 running **Drupal 7**.
- **CMS Fingerprinting:** Confirmed the version was susceptible to the original "Drupalgeddon" SQLi.

## 3. Exploitation

### 3.1 Attack Vector: CVE-2014-3704 (SQL Injection)
The vulnerability lies in the `expandArguments` function in Drupal's database API, which allows an unauthenticated attacker to send specially crafted arrays to trigger a SQL injection.

**Exploitation Steps:**
1. **Payload Delivery:** Sent a POST request to the login page with a malicious array in the form parameters.
2. **Account Creation:** The SQLi was used to inject a new administrative user into the `users` table.
3. **Admin Access:** Logged into the Drupal dashboard using the newly created credentials.
4. **RCE via PHP Filter:** Enabled the "PHP Filter" module to execute arbitrary PHP code, establishing a **Reverse Shell** via netcat.



## 4. Privilege Escalation

### 4.1 Post-Exploitation
- Accessed the local file system as `www-data`.
- Enumerated SUID binaries using: `find / -perm -u=s -type f 2>/dev/null`.

### 4.2 SUID Find Exploitation
- Identified that `/usr/bin/find` had the SUID bit set.
- **Payload:** `find . -exec /bin/sh \; -quit`
- **Result:** Successfully spawned a shell with **root** privileges.

## 5. Conclusion & Remediation
- **Patching:** Update Drupal to version 7.32 or higher to mitigate CVE-2014-3704.
- **System Hardening:** Remove SUID permissions from administrative binaries like `find` that are not required for standard users.
- **Input Validation:** Ensure database APIs correctly sanitize array-based inputs.
