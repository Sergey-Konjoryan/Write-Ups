## 1. Executive Summary
The compromise of Matrix 1 required a combination of web-based information gathering, credential brute-forcing, and restricted shell escapes. The primary entry point was discovered through source code analysis, leading to an SSH session that was initially restricted by `rbash`.

## 2. Reconnaissance & Enumeration

### 2.1 Service Scanning
A full port scan using `nmap` identified the following:
* **Port 80/tcp:** Apache Web Server.
* **Port 1000/tcp:** SSH (Moved from the default port 22).

### 2.2 Web Analysis
- **Source Code Investigation:** Deep inspection of the landing page's HTML and CSS revealed a hidden Base64 string in a comment.
- **Directory Discovery:** Using `gobuster`, I located the `/n00b/` directory, which contained further clues and a small custom wordlist.
- **Credential Harvesting:** By correlating hints found in the CSS files, the username `cypher` was identified.

---

## 3. Exploitation

### 3.1 Attack Vector: SSH Brute Force
With a confirmed username (`cypher`) and a target-specific wordlist:
1. **Tool:** Used `Hydra` against the SSH service on port 1000.
2. **Success:** Recovered the password and established a remote session.

### 3.2 Post-Exploitation: rbash Bypass
Upon login, the account was locked into an **rbash** (restricted bash) environment, preventing most commands and directory changes.
- **Bypass Technique:** Leveraged Python's `os` module to spawn a standard shell.
- **Payload:** `python -c 'import os; os.system("/bin/bash")'`
- **Result:** Successfully transitioned to a full interactive shell.



---

## 4. Privilege Escalation

### 4.1 Root Access
- **Enumeration:** Ran `sudo -l` and checked for SUID binaries.
- **Discovery:** Identified an insecure path or binary that allowed for execution of arbitrary commands as root.
- **Execution:** Exploited the misconfiguration to escalate from `cypher` to `root`.

**Flag:** Captured the final flag in `/root/flag.txt`.
