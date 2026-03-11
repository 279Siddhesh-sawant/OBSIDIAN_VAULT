Nmap scan
```sh
nmap -p- --min-rate 5000 -T4 -Pn 192.168.192.178
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-04 15:17 IST
Warning: 192.168.192.178 giving up on port because retransmission cap hit (6).
Stats: 0:01:24 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 99.99% done; ETC: 15:18 (0:00:00 remaining)
Nmap scan report for 192.168.192.178
Host is up (3.7s latency).
Not shown: 49849 closed tcp ports (reset), 15684 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 92.17 seconds
```

```sh
nmap -sC -sV -T4 -Pn -p 22,80 192.168.192.178   
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-04 15:19 IST
Nmap scan report for 192.168.192.178
Host is up (0.13s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 62:36:1a:5c:d3:e3:7b:e1:70:f8:a3:b3:1c:4c:24:38 (RSA)
|   256 ee:25:fc:23:66:05:c0:c1:ec:47:c6:bb:00:c7:4f:53 (ECDSA)
|_  256 83:5c:51:ac:32:e5:3a:21:7c:f6:c2:cd:93:68:58:d8 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: ImageMagick Identifier
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.58 seconds
```

Visiting web server on port 80.
![](Image2.png)
When we upload valid png file, we discovered the version.
![](Image1.png)
We searched for public exploits and found below.
https://github.com/SudoIndividual/CVE-2023-34152
![](Image3.png)
![](Image4.png)
Uploading the created png file file to application. Before uploading nc started.
![](Image5.png)
We got the shell.
![](Image6.png)
Local flag.
![](Image7.png)

### Privilege Escalation
Tried sudo permission but no luck.
![](Image8.png)
Checked for SUID binary and found below.
![](Image9.png)
Searched on GTFobins.
![](Image10.png)

Found root flag.
![](Image11.png)
