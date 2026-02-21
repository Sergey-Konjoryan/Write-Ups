## 1. Executive Summary
The objective was to gain root-level access to the Symfonos 1 virtual machine. The attack chain involved SMB enumeration to gather user information, exploiting a Local File Inclusion (LFI) vulnerability in a WordPress site, and escalating privileges via SMTP Log Poisoning followed by an insecure PATH vulnerability in a SUID binary.

## 2. Reconnaissance & Enumeration

### 2.1 Network Scanning
A full TCP port scan was performed using `nmap`:
* **Port 22/tcp:** SSH (OpenSSH 7.4p1)
* **Port 25/tcp:** SMTP (Postfix)
* **Port 80/tcp:** HTTP (Apache 2.4.25)
* **Port 139, 445/tcp:** SMB (Samba)

### 2.2 SMB Enumeration
Using `smbclient -L`, I identified an **anonymous** share.
* **Findings:** A file named `attention.txt` was retrieved.
* **Intelligence:** The file suggested potential credentials and confirmed the existence of a system user: **helios**.

## 3. Vulnerability Analysis & Exploitation

### 3.1 Web Vulnerability (LFI)
The web server hosted a WordPress site at `/h3l105`. Manual testing revealed a **Local File Inclusion (LFI)** vulnerability in a site parameter.

### 3.2 Initial Access: SMTP Log Poisoning
To gain Remote Code Execution (RCE), I performed a **Log Poisoning** attack:
1. Connected to SMTP (Port 25) and sent an email to the user `helios`.
2. Injected a PHP web shell into the email body: `<?php system($_GET['cmd']); ?>`.
3. Accessed the poisoned mail log via LFI at `/var/mail/helios`.
4. Executed a Python reverse shell to gain an interactive session as user `helios`.



## 4. Privilege Escalation

### 4.1 SUID Binary Analysis
Enumeration of SUID binaries revealed a custom tool: `/opt/statuscheck`.
* **Analysis:** Using `strings`, I discovered that the binary calls the `curl` command using a relative path rather than an absolute path.

### 4.2 Exploitation: PATH Hijacking
I manipulated the environment to hijack the execution flow:
1. Created a malicious `curl` script in `/tmp` that executes `/bin/sh`.
2. Modified the `$PATH` variable: `export PATH=/tmp:$PATH`.
3. Executed `/opt/statuscheck`.

**Result:** The system executed my malicious script with root privileges, granting a **Root Shell**.
