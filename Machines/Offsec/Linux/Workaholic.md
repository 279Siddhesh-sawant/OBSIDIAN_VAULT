Nmap scan
```sh
nmap -p- --min-rate 5000 -T4 -Pn 192.168.164.229
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-22 15:01 IST
Stats: 0:01:03 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 96.80% done; ETC: 15:02 (0:00:02 remaining)
Nmap scan report for 192.168.164.229
Host is up (0.066s latency).
Not shown: 65505 filtered tcp ports (no-response), 27 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 65.95 seconds
```

```sh
nmap -sC -sV -T4 -Pn -p 21,22,80 192.168.164.229
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-22 15:05 IST
Nmap scan report for 192.168.164.229
Host is up (0.16s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 f2:5a:a9:66:65:3e:d0:b8:9d:a5:16:8c:e8:16:37:e2 (ECDSA)
|_  256 9b:2d:1d:f8:13:74:ce:96:82:4e:19:35:f9:7e:1b:68 (ED25519)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-title: Workaholic
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-generator: WordPress 6.7.2
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.37 seconds
```

Visiting web server on port 80.
![](Images/Workaholic1.png)
