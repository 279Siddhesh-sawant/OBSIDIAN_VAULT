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
![](Images/Scrutiny1.png)
![](Images/Scrutiny2.png)
We added hostname to /etc/hosts file.

