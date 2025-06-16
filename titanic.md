# Titanic - no walkthrough - active machine

## Basic Enumeration

Did `nmap -sC -sV`  

Found:  

```
22/tcp    open     ssh           OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 73:03:9c:76:eb:04:f1:fe:c9:e9:80:44:9c:7f:13:46 (ECDSA)
|_  256 d5:bd:1d:5e:9a:86:1c:eb:88:63:4d:5f:88:4b:7e:04 (ED25519)
80/tcp    open     http          Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://titanic.htb/
|_http-server-header: Apache/2.4.52 (Ubuntu)
```

Added the IP and domain in `/etc/hosts` and visited the website.

Couldn’t find anything useful by poking around in the website.

Using ffuf tried subdomain fuzzing:

```
ffuf -w SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt -u 'http://titanic.htb' -H "Host:FUZZ.titanic.htb" -fc 301
```

Had to filter out responses which had status 301 using `-fc` because most subdomains were returning 301 status.

Found a subdomain called `dev`.

Added the newly found subdomain to `/etc/hosts` and accessed the subdomain, which was a code repo.

Found a docker config file for a SQL DB and the YAML file contained the MySQL credentials — if I had added `-p-` I think I would have seen the SQL server.

Now trying to access the DB but it's only allowed to be accessed from inside the host.

Seems like Gitea was not the in.

Found a **LFI** vulnerability in the route `/download?ticket=` and I passed `../../../etc/passwd` to get all the users and found a user called `developer`. Got the user flag by giving `../../../home/developer/user.txt`.

Got the Gitea config here: `../../../../home/developer/gitea/data/gitea/conf/app.ini`.

Now dumped the Gitea DB by downloading the `gitea.db` file and got the salt and password for both the administrator and developer.

Since the password is in PBKDF2 hash, we convert the salt and passwd to the format Hashcat can use by using `gitea2hashcat.py` and then ran:

```
hashcat -m 10900 hashes.txt rockyou.txt
```

Where:
* `-m 10900` specifies the hash mode for PBKDF2-HMAC-SHA256.
* `hashes.txt` contains the converted hashes (one salt + hash combo per line).
* `rockyou.txt` is the wordlist.

Then got the developer user’s password.

Logged in to the SSH and got the user flag.

Checked `sudo -l` and it showed `NOPASSWD: ALL` which meant I can use sudo without needing a password.

Now I ran `sudo su -` to get the root shell and got the root flag.