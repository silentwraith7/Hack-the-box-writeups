
# Code - Active Machine

## Enumeration

Did `nmap -sV -sC --min-rate 500 <ip address>`  
Found an SSH server and an HTTP server.

The HTTP server seems to be a Python code compiler application – uses Ace Editor and AJAX for calls to the backend.

Did directory fuzzing using `ffuf` but no useful directories.  
Tried subdomain fuzzing too but no subdomains were found.

Added BurpSuite proxy to see the requests being sent, tried intercepting and tinkering with inputs – nothing really worked, the inputs were sanitized.

So had to dig deep on how to exploit the Python editor and saw that we can print all the global variables used by printing 'globals()', it will print it too (had to refer a writeup).  
Ran:  
```python
print(globals())
```

This prints all the global stuff like all the imports.  
Saw that **SQLAlchemy** is imported, with two models: the `User` model and the `Code` model.

Since SQLAlchemy is imported, we can query directly in the editor using:  
```python
User.query.all()
```
Which returns that there are two users.

Now to get all the users in the table, ran:  
```python
print([{col.name: getattr(u, col.name) for col in User.__table__.columns} for u in User.query.all()])
```

For each user in the table, it takes the column name and value and puts it into an object/JSON.

Cracked the hash using hashcat:  
```bash
hashcat -a 0 -m 0 martin_user_hash.hash /usr/share/wordlists/rockyou.txt.gz
```
- `-a 0` specifies attack mode: straight dictionary attack  
- `-m 0` specifies raw MD5 hash

Got the password for the user `martin` and logged into SSH with that user.

## Privilege Escalation

Ran `sudo -l` and saw that the user has access to run one file as root: `/usr/bin/backy.sh`  
It requires a `task.json` file and will archive the backup of the target specified in the JSON.

Tried running it without changing anything – in the backup we get the **user flag**.

Then modified the `target` path in the JSON to something like `/home/../root`  
Because only paths starting with `/home` or `/var` are allowed.

It backed up `/root`, and we got the **root flag**.
