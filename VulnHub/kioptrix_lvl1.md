# PenTest Report: Kioptrix Level 1 (#1)
**Objective:** Privilege Escalation to Root

### 1. Reconnaissance
Network discovery via `nmap -sC -sV` revealed:
- **Port 80:** Apache 1.3.20
- **Port 139:** Samba 2.2.1a

### 2. Vulnerability Analysis
Researching **Samba 2.2.1a** using `searchsploit` indicated a vulnerability to the "trans2open" buffer overflow attack.

### 3. Exploitation & Root
- **Vector:** Samba exploitation via `trans2open`.
- **Payload:** `linux/x86/shell_reverse_tcp`.
- **Execution:** Established a reverse shell. Due to the nature of the vulnerability, the shell was spawned with **root** privileges.

### 4. Conclusion
The machine was compromised due to an outdated Samba service. Legacy systems should be isolated or decommissioned.
