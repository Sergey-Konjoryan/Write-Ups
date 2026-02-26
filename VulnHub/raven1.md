## 1. Executive Summary
The compromise of Raven 1 involved a multi-stage attack starting with web enumeration, exploiting a critical vulnerability in the PHPMailer library (CVE-2016-10033) to gain a reverse shell, and escalating privileges by exploiting a misconfigured MySQL service using User Defined Functions (UDF).

## 2. Reconnaissance & Enumeration

### 2.1 Service Scanning
An `nmap` scan identified the following active services:
* **Port 22/tcp:** SSH
* **Port 80/tcp:** HTTP (Apache 2.4.7, WordPress)
* **Port 111/tcp:** RPCbind
* **Port 443/tcp:** HTTPS

### 2.2 Web Discovery
* **Technology:** The site runs on WordPress.
* **Enumeration:** Using `gobuster`, I discovered a `/vendor/` directory containing a `PATH` file and a `VERSION` file for PHPMailer.
* **Vulnerability:** The PHPMailer version was identified as **5.2.17**, which is vulnerable to Remote Code Execution (RCE).

---

## 3. Exploitation

### 3.1 Attack Vector: PHPMailer RCE (CVE-2016-10033)
Using a python exploit for CVE-2016-10033, I targeted the `contact.php` page.
1. **Payload:** The exploit leverages the `mail()` function to inject a malicious sender address that writes a PHP shell into the web root.
2. **Execution:** After triggering the exploit, I accessed the generated shell at `/contact.php` and established a reverse shell.
3. **Result:** Gained initial access as the `www-data` user.

### 3.2 Post-Exploitation
- Found `wp-config.php` containing database credentials: `root:R@v3n1backdoor`.
- Located 3 out of 4 flags during the file system traversal.

---

## 4. Privilege Escalation

### 4.1 MySQL UDF Exploitation
The MySQL service was running as `root`, and I had the credentials for the MySQL root user.
1. **Verification:** Checked if MySQL could load external libraries (`secure_file_priv` was empty).
2. **Exploit:** Used the **MySQL User Defined Function (UDF) Dynamic Library** exploit.
   * Uploaded `lib_mysqludf_sys.so` to the target.
   * Created a function in MySQL: `CREATE FUNCTION sys_exec RETURNS int SONAME 'lib_mysqludf_sys.so';`
3. **Root Shell:** Executed system commands with root privileges:
   ```sql
   select sys_exec('cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash');
