# PenTest Report: Metasploitable 2
**Difficulty:** Beginner
**Objective:** Gain Root Access

### 1. Enumeration
A full TCP port scan was performed using `nmap -sV -p-`. Multiple vulnerable services were identified.
- **Critical Finding:** Port 21/TCP running **vsftpd 2.3.4**.

### 2. Exploitation
The vsftpd 2.3.4 version is known to contain a backdoor triggered by a specific character sequence in the username.
- **Tool:** Metasploit Framework (`exploit/unix/ftp/vsftpd_234_backdoor`).
- **Execution:** The exploit was launched against the target, successfully triggering the backdoor.
- **Result:** Immediate **root** shell access.

### 3. Mitigation
- Update vsftpd to the latest secure version.
- Implement a firewall to restrict access to management ports.
