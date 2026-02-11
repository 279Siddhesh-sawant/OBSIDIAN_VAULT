Nmap scan
```sh
sudo nmap -p- --min-rate 5000 -T4 -Pn 10.81.191.55
[sudo] password for kali: 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-10 19:32 IST
Nmap scan report for 10.81.191.55
Host is up (0.18s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 16.71 seconds
```

```sh
sudo nmap -sC -sV -T4 -Pn -p 22,80,8080 10.81.191.55
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-10 19:33 IST
Nmap scan report for 10.81.191.55
Host is up (0.20s latency).

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 2a:72:3f:3e:11:64:ed:4d:c5:e4:1d:27:5a:93:ba:56 (ECDSA)
|_  256 7b:96:41:e0:cf:e3:4b:4d:6d:d5:cb:e0:c4:6a:b0:fc (ED25519)
80/tcp   open  http       nginx 1.18.0 (Ubuntu)
|_http-title: Hack Smarter Security
|_http-server-header: nginx/1.18.0 (Ubuntu)
8080/tcp open  http-proxy
|_http-title: Error
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 Not Found
|     Connection: close
|     Content-Length: 74
|     Content-Type: text/html
|     Date: Tue, 10 Feb 2026 14:04:07 GMT
|     <html><head><title>Error</title></head><body>404 - Not Found</body></html>
|   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SMBProgNeg, SSLSessionReq, Socks5, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Length: 0
|     Connection: close
|   GetRequest: 
|     HTTP/1.1 404 Not Found
|     Connection: close
|     Content-Length: 74
|     Content-Type: text/html
|     Date: Tue, 10 Feb 2026 14:04:05 GMT
|     <html><head><title>Error</title></head><body>404 - Not Found</body></html>
|   HTTPOptions: 
|     HTTP/1.1 404 Not Found
|     Connection: close
|     Content-Length: 74
|     Content-Type: text/html
|     Date: Tue, 10 Feb 2026 14:04:06 GMT
|_    <html><head><title>Error</title></head><body>404 - Not Found</body></html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8080-TCP:V=7.95%I=7%D=2/10%Time=698B3AD6%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,C9,"HTTP/1\.1\x20404\x20Not\x20Found\r\nConnection:\x20close\r
SF:\nContent-Length:\x2074\r\nContent-Type:\x20text/html\r\nDate:\x20Tue,\
SF:x2010\x20Feb\x202026\x2014:04:05\x20GMT\r\n\r\n<html><head><title>Error
SF:</title></head><body>404\x20-\x20Not\x20Found</body></html>")%r(HTTPOpt
SF:ions,C9,"HTTP/1\.1\x20404\x20Not\x20Found\r\nConnection:\x20close\r\nCo
SF:ntent-Length:\x2074\r\nContent-Type:\x20text/html\r\nDate:\x20Tue,\x201
SF:0\x20Feb\x202026\x2014:04:06\x20GMT\r\n\r\n<html><head><title>Error</ti
SF:tle></head><body>404\x20-\x20Not\x20Found</body></html>")%r(RTSPRequest
SF:,42,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Length:\x200\r\nConn
SF:ection:\x20close\r\n\r\n")%r(FourOhFourRequest,C9,"HTTP/1\.1\x20404\x20
SF:Not\x20Found\r\nConnection:\x20close\r\nContent-Length:\x2074\r\nConten
SF:t-Type:\x20text/html\r\nDate:\x20Tue,\x2010\x20Feb\x202026\x2014:04:07\
SF:x20GMT\r\n\r\n<html><head><title>Error</title></head><body>404\x20-\x20
SF:Not\x20Found</body></html>")%r(Socks5,42,"HTTP/1\.1\x20400\x20Bad\x20Re
SF:quest\r\nContent-Length:\x200\r\nConnection:\x20close\r\n\r\n")%r(Gener
SF:icLines,42,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Length:\x200\
SF:r\nConnection:\x20close\r\n\r\n")%r(Help,42,"HTTP/1\.1\x20400\x20Bad\x2
SF:0Request\r\nContent-Length:\x200\r\nConnection:\x20close\r\n\r\n")%r(SS
SF:LSessionReq,42,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Length:\x
SF:200\r\nConnection:\x20close\r\n\r\n")%r(TerminalServerCookie,42,"HTTP/1
SF:\.1\x20400\x20Bad\x20Request\r\nContent-Length:\x200\r\nConnection:\x20
SF:close\r\n\r\n")%r(TLSSessionReq,42,"HTTP/1\.1\x20400\x20Bad\x20Request\
SF:r\nContent-Length:\x200\r\nConnection:\x20close\r\n\r\n")%r(Kerberos,42
SF:,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Length:\x200\r\nConnect
SF:ion:\x20close\r\n\r\n")%r(SMBProgNeg,42,"HTTP/1\.1\x20400\x20Bad\x20Req
SF:uest\r\nContent-Length:\x200\r\nConnection:\x20close\r\n\r\n")%r(LPDStr
SF:ing,42,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Length:\x200\r\nC
SF:onnection:\x20close\r\n\r\n")%r(LDAPSearchReq,42,"HTTP/1\.1\x20400\x20B
SF:ad\x20Request\r\nContent-Length:\x200\r\nConnection:\x20close\r\n\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 93.16 seconds
```

Not much information, but still some useful stuff. We know that the target is a Linux machine running Ubuntu, and that it has 2 HTTP services listening on ports 80 and 8080.

Let’s take a look.

### Port 80
![](Images1/silver_platter1.png)
The website is a mostly static HTML website.

While clicking on the different tabs, we come across some useful information in the “contact” tab:
![](Images1/silver_platter2.png)
The contact tab leaks a potential username, as well as the name of some app called “Silverpeas”.

We can take note of this, and move on with our enumeration.

At this point, I ran Gobuster in the background while taking a look at port 8080. There were no interesting results there, so I’ll just move on to port 8080.

### Port 8080
![](Images1/silver_platter3.png)
Unfortunately, we get a 404 not found error when we visit port 8080.

Not much we can do here, so we should look for more content with gobuster:
![](Images1/silver_platter4.png)
![](Images1/silver_platter5.png)

We get two interesting endpoints, but upon researching them further, we discover that they are both, most likely, dead ends. /website returns a “Forbidden” error, and /console redirects us to a 404 page.

So, it seems like we are a bit stuck and don’t really have much of an attack surface.

However, recall that the message on the contact tab earlier mentioned “Silverpeas”. What if the directory that we need is /silverpeas ?

On port 80, we don’t find anything. But on port 8080…

![](Images1/silver_platter6.png)
Nice! Now we have something to work with.

### Attacking SilverPeas — Gaining Access as Admin

So, we are presented with a login page. We could try to brute-force the user “scr1ptkiddy” that was mentioned earlier, but these attacks are very loud and I usually save them for a last-resort.

We can begin by searching for “SilverPeas” default credentials on google:
`SilverAdmin/SilverAdmin`
Login failed.

Ok, the next step would be to research any vulnerabilities in SilverPeas. But to search effectively, we need to know the version. I was unable to identify the exact version, but the login page gives us a pretty good indication:
2022 is mentioned as the current year. So any CVE from 2023 and 2024 should be useful.

Simply searching for “SilverPeas CVEs” on google:
https://github.com/advisories/GHSA-4w54-wwc9-x62c
https://gist.github.com/ChrisPritchard/4b6d5c70d9329ef116266a6c238dcb2d
Reading through the details , it would seem that all we have to do, is to capture the login request in BurpSuite, and remove the password field from the request.
Let’s try:
![](Images1/silver_platter7.png)
![](Images1/silver_platter8.png)
![](Images1/silver_platter12.png)

Here, I was unable to make changes to request while intercepting. So I choose another method.
Now it’s time to see Where I can get my password because we already have the username . For this, they Delievered the clue for us :
Here, U can see they are referring to some kind of Wordlist other than rockyou.txt. They clearly mentioned about that ‘cool’ which is cuel in real. So let’s see
![](Images1/silver_platter9.png)
![](Images1/silver_platter10.png)
 Here u can see we got our custom wordlists and we have the username too so it is quite easy for us to go now…
## **Bruteforcing through Burp Suite..**
 Here After intercepting the request, We are bruteforcing for the Passowrd for the Username **“scr1ptkiddy**” and We got Our Password here which is **“adipiscing”.**
![](Images1/silver_platter11.png)
![](Images1/silver_platter13.png)
![](Images1/silver_platter28.png)

Here We got our Silverpeas page

- After looking at it, I thing I found interesting in it which is this “1 Unread notification”
- After clicking on it, Clicking on one or two more things ,
![](Images1/silver_platter14.png)
![](Images1/silver_platter15.png)
![](Images1/silver_platter16.png)
This Page comes, Where I can see this id Parameter. Also If u go to silverpeas Githhub , U can find the listing :

![](Images1/silver_platter17.png)
- In this there are 8 CVE’s contains all the information out there…
- Here at the 4th point ,I see one thing Where there is Broken access control…
- I also found something similar like that , so I thought let’s check to change the id parameter, When I changed the id parameter, I got to know something u wanna know …let’s see
![](Images1/silver_platter18.png)
- Here , I get some information out there..
- Here u see we got our username and password For the SSH service, Let’s Try to login through it

Username : tim
Password : cm0nt!md0ntf0rg3tth!spa(2timesdollar)w0rdagainlol

- **Let’s Try Login via SSH :**
![](Images1/silver_platter20.png)

![](Images1/silver_platter19.png)
![](Images1/silver_platter21.png)
![](Images1/silver_platter30.png)
## **Time To Privilege Escalation**

- Here Firstly We search the id parameter there in this, we got the adm. Now when we search this adm for linux , We get some of the information there :
![](Images1/silver_platter22.png)
That those /var/log files are enabled to read so it’s a good chance for us. 
**Lateral Movement to Tyler’s Account**

While enumerating the system as Tim, we notice there is another user account on the box: **tyler** (for example, an entry _/home/tyler_ exists, or maybe we saw “tyler” in _/etc/passwd_). Typically, privilege escalation on CTF boxes might involve moving laterally from one user to another (especially if the other user has higher privileges or is in the sudoers list). Here, **tyler** could be an administrative user, so we aim to access that account next.

Since we know Tim is in the _adm_ group and can read logs, a common tactic is to search through **system log files** for sensitive information like passwords. Administrators sometimes accidentally leave credentials in logs (for example, service startup logs, database logs, or failed login attempts). Given the presence of a user **tyler**, we specifically look for anything mentioning “tyler” in the logs. We can also search for keywords like “password” or “pass” in case a password was logged.

A convenient way to do this is using _grep_ on all files under _/var/log_. For instance, we can run:
Here we can use 2 commands
```sh
grep -ir "password"
```
This can collect all that words from these Log Files

![](Images1/silver_platter23.png)
![](Images1/silver_platter24.png)

### OR
```sh
grep -R -i "tyler" /var/log 2>/dev/null
```
This will recursively search for the string “tyler” in _/var/log_. We can also include multiple patterns (e.g., ‘tyler’ and ‘pass’) or search the auth logs specifically. In fact, checking **auth logs** is a good idea, since they record login attempts. We use this command to search across all authentication logs for any lines containing “tyler” or “pass”. Example:
```sh
cat /var/log/auth* | grep -ai -e 'tyler' -e 'pass' -e 'ssh'
```
The outputs will be long, but don’t let that intimidate you. Read it carefully, what you’re looking for is a line such as the following:
![](Images1/silver_platter25.png)
![](Images1/silver_platter26.png)
![](Images1/silver_platter27.png)
- Firstly, We switch to Our Tyler account, using the “su” command
tyler : - `_Zd_zx7N823/

**Privilege Escalation to Root**

Now that we are logged in as Tyler, we check what privileges this user has. A good practice is to run _sudo -l_ to list the sudo privileges for the current user.

We are prompted for Tyler’s password (which we already have). After entering it, the output shows Tyler’s sudo rights. The result is that **Tyler can** **run all commands as all users** (the sudoers entry is _tyler ALL=(ALL:ALL) ALL)_. In other words, Tyler is effectively an admin user who can use _sudo_ to do anything (this is typical if Tyler is in the _sudo_ group or otherwise configured with full sudo privileges). There appear to be no restrictions (and no mention of a password being required beyond the one we already have).

This means privilege escalation to root is straightforward: we can simply invoke a shell as root via sudo. Example:
![](Images1/silver_platter31.png)

```sh
sudo su
```

This command opens a root shell (since Tyler is allowed to run _su_ or any command as root). We could also do _sudo -i_ for an interactive root shell. Now, our _whoami_ returns **root**, indicating we have full superuser access to the system.

With root access, our final step is to retrieve the **Root Flag**. Typically, the root flag is stored in _/root/root.txt_ on TryHackMe machines. So we simply run:
![](Images1/silver_platter32.png)
![](Images1/silver_platter33.png)
![](Images1/silver_platter34.png)



