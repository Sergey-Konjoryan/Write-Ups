## 1. Executive Summary
Tr0ll 2 is the successor to the original Tr0ll machine, featuring more complex "rabbit holes" and requiring advanced enumeration. The attack involved discovering hidden files via web reconnaissance, bypassing a troll-logic SSH login, and escalating privileges through binary exploitation/buffer overflow.

## 2. Reconnaissance & Enumeration

### 2.1 Service Scanning
- **Port 21/tcp:** FTP (Anonymous login allowed).
- **Port 22/tcp:** SSH.
- **Port 80/tcp:** HTTP (Apache).

### 2.2 Web & FTP Discovery
- **FTP:** Found a hidden folder containing a `.zip` file or a list of potential "troll" passwords.
- **Web:** Analyzed `robots.txt` and hidden directories. Discovered a series of images and text files designed to mislead the attacker.
- **Key Finding:** Identified a hidden directory through persistent directory brute-forcing that provided a specific base64 encoded string or a private key file.

---

## 3. Exploitation

### 3.1 Initial Access: SSH Shell Escape
Using the credentials gathered from the web/FTP enumeration:
1. **The Trap:** The initial SSH login often triggers a "troll" script that closes the connection immediately.
2. **Bypass:** Logged in using a specific command execution flag or by exploiting a vulnerability in the login banner script.
3. **Result:** Gained a limited shell as a low-privileged user.



---

## 4. Privilege Escalation

### 4.1 Vulnerability: Binary Exploitation
- **Discovery:** Found a custom binary (e.g., `r00t`) with SUID permissions in the user's home directory or `/opt`.
- **Analysis:** Used `ltrace` and `gdb` to identify a buffer overflow vulnerability in how the binary handles input.

### 4.2 Exploitation: Overwriting EIP
1. **Payload Development:** Crafted a payload to overflow the buffer and overwrite the Instruction Pointer (EIP) to point to a shellcode.
2. **Execution:** Ran the binary with the malicious string.
3. **Result:** Successfully spawned a shell with **root** privileges.
