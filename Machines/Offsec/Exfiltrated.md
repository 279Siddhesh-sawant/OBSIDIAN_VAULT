Nmap scan
```sh
nmap -p- --min-rate 5000 -T4 -Pn 192.168.169.163
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-16 21:04 IST
Warning: 192.168.169.163 giving up on port because retransmission cap hit (6).
Nmap scan report for 192.168.169.163
Host is up (0.090s latency).
Not shown: 64760 closed tcp ports (reset), 773 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 16.39 seconds
```

```sh
nmap -sC -sV -T4 -Pn -p 22,80 192.168.169.163   
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-16 21:04 IST
Nmap scan report for 192.168.169.163
Host is up (0.13s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1:99:4b:95:22:25:ed:0f:85:20:d3:63:b4:48:bb:cf (RSA)
|   256 0f:44:8b:ad:ad:95:b8:22:6a:f0:36:ac:19:d0:0e:f3 (ECDSA)
|_  256 32:e1:2a:6c:cc:7c:e6:3e:23:f4:80:8d:33:ce:9b:3a (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Did not follow redirect to http://exfiltrated.offsec/
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-robots.txt: 7 disallowed entries 
| /backup/ /cron/? /front/ /install/ /panel/ /tmp/ 
|_/updates/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.13 seconds
```

Visited web server on port 80.
![[Pasted image 20260216213612.png]]
![[Exfiltrated1.png]]
In the page source, we found that application is using Subrion 4.2.
![[Exfiltrated2.png]]
Here, we also did directory brute forcing and visiting on some of those directories like /login, /profile, /panel we found out the same subrion CMS version.
Later we searched for public exploits.
![[Exfiltrated3.png]]
We downloaded and run Arbitary file upload exploit.
![[Exfiltrated4.png]]
![[Exfiltrated5.png]]
We faced login failed response. So we tried default creds `admin : admin` and got success.
![[Exfiltrated6.png]]
