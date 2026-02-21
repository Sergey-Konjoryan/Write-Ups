## 1. Executive Summary
The goal was to compromise the W1R3S machine and obtain root access. The attack vector involved discovering sensitive information on an open FTP server, exploiting a Local File Inclusion (LFI) vulnerability in a CMS, and leveraging insecure sudo permissions for privilege escalation.

## 2. Reconnaissance & Enumeration

### 2.1 Network Discovery
An `nmap` scan was conducted to identify open services:
* **Port 21/tcp:** FTP (vsftpd 2.0.8 or later)
* **Port 22/tcp:** SSH (OpenSSH 7.2p2)
* **Port 80/tcp:** HTTP (Apache 2.4.18)
* **Port 3306/tcp:** MySQL

### 2.2 FTP Enumeration
The FTP service allowed **anonymous login**. 
* **Findings:** I found several files, including `creds.txt` and `worktodo.txt`.
* **Intelligence:** These files contained some hashed passwords and hints about the server setup.

### 2.3 Web Enumeration
Scanning directories with `gobuster` revealed an installation of **Cuppa CMS** in the `/administrator/` directory.

---

## 3. Exploitation

### 3.1 Vulnerability: Local File Inclusion (LFI)
The Cuppa CMS installation was found to be vulnerable to a known **Local File Inclusion (LFI)** exploit in the `urlConfig.php` file.



### 3.2 Gaining a Shell
Using the LFI vulnerability, I was able to read the `/etc/shadow` file:
`http://<target-ip>/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../etc/shadow`

1. **Cracking Hashes:** I extracted the password hash for the user `w1r3s`.
2. **John the Ripper:** The hash was cracked quickly, revealing the password: `computer`.
3. **SSH Access:** I logged in via SSH using these credentials.

---

## 4. Privilege Escalation

### 4.1 Sudo Enumeration
Once logged in as `w1r3s`, I checked the user's sudo privileges:
```bash
w1r3s@w1r3s:~$ sudo -l
(ALL : ALL) ALL
