# PenTest Report: Tr0ll 1
**Category:** Boot2Root / CTF

### 1. Enumeration & PCAP Analysis
Found an open FTP service with anonymous login. Downloaded a `.pcap` file. Analyzing the traffic in **Wireshark** revealed a hidden directory path: `/sup3r_s3cr3t_d1r`.

### 2. Exploitation
A binary file found in the hidden directory contained a potential password string. Using this string, I performed a brute-force/logic test against the SSH service.
- **Result:** Gained SSH access as user `overflow`.

### 3. Privilege Escalation
Identified a world-writable **cronjob** running with root privileges. 
- **Method:** Injected a reverse shell command into the cron script.
- **Outcome:** After the cronjob execution, a **root** shell was caught on the listener.
