## 1. Executive Summary
Lord of the Root is a classic CTF machine simulating a hardened corporate infrastructure. The primary challenge lies in its multi-layered defense strategy:

## 2. Reconnaissance & Enumeration

### 2.1 Service Scanning
- **Port 22/tcp:** SSH (Filtered/Hidden via Port Knocking).
- **Port 80/tcp:** HTTP (Apache).

### 2.2 Port Knocking Bypass
Initial Nmap scans showed the SSH port as "filtered" by the firewall. Analysis of the connection banner and the hint "Easy as 1,2,3" indicated that a specific knocking sequence was required to trigger access.
- **Analysis:** By performing a full port scan (`nmap -p-`), three specific ports were identified as triggers for the firewall rule.
- **Bypass:** Used the `knock` utility to send packets to the target ports in the correct sequence.
- **Result:** The firewall rule was temporarily modified, granting access to port 22 for the attacker's IP address.

### 2.3 Web Discovery
- **Directory Brute-forcing:** Used `gobuster` to discover a hidden directory: `/secret_dir/`.
- **Finding:** Inside this directory was a login portal interacting with a backend database via a vulnerable PHP script.

---

## 3. Exploitation

### 3.1 Initial Access: SQL Injection (SQLi)
1. **Vulnerability:** The `id` parameter in the database query was found to be vulnerable to Union-based and Error-based SQLi.
2. **Exploitation:** Automated data extraction using `sqlmap`:
   - **Command:** `sqlmap -u "http://192.168.118.159/secret_dir/index.php?id=1" --dbms=mysql --dump`
3. **Credentials:** Successfully extracted usernames and password hashes from the system database.
4. **Shell Access:** After cracking the hash for the user `smeagol`, an SSH login was performed (after re-triggering the Port Knocking sequence).

---

## 4. Privilege Escalation

### 4.1 Vulnerability: Kernel Exploitation
- **Discovery:** Running `uname -a` revealed that the system was running an outdated version of Ubuntu 14.04 (Kernel 3.19.0-25-generic).
- **Vulnerability:** This specific kernel version is susceptible to the **CVE-2015-1328 (OverlayFS)** exploit.

### 4.2 Exploitation: Local Privilege Escalation
1. **Transfer:** The exploit source code (C) was uploaded to the target machine's `/tmp` directory.
2. **Compilation:** Compiled the exploit locally using `gcc`:
   ```bash
   gcc 39166.c -o bobm
