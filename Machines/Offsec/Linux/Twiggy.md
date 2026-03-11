Nmap scan
```sh
nmap -p- --min-rate 5000 -T4 -Pn 192.168.200.62             
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-16 12:02 IST
Stats: 0:00:19 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 48.79% done; ETC: 12:02 (0:00:20 remaining)
Nmap scan report for 192.168.200.62
Host is up (0.23s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
80/tcp   open  http
4505/tcp open  unknown
4506/tcp open  unknown
8000/tcp open  http-alt

Nmap done: 1 IP address (1 host up) scanned in 52.87 seconds
```
```sh
nmap -sC -sV -T4 -Pn -p 22,53,80,4505,4506,8000 192.168.200.62
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-16 12:03 IST
Nmap scan report for 192.168.200.62
Host is up (0.12s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 44:7d:1a:56:9b:68:ae:f5:3b:f6:38:17:73:16:5d:75 (RSA)
|   256 1c:78:9d:83:81:52:f4:b0:1d:8e:32:03:cb:a6:18:93 (ECDSA)
|_  256 08:c9:12:d9:7b:98:98:c8:b3:99:7a:19:82:2e:a3:ea (ED25519)
53/tcp   open  domain  NLnet Labs NSD
80/tcp   open  http    nginx 1.16.1
|_http-title: Home | Mezzanine
|_http-server-header: nginx/1.16.1
4505/tcp open  zmtp    ZeroMQ ZMTP 2.0
4506/tcp open  zmtp    ZeroMQ ZMTP 2.0
8000/tcp open  http    nginx 1.16.1
|_http-server-header: nginx/1.16.1
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Site doesn't have a title (application/json).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.88 seconds
```

After browsing the website on port 80, I tried the possible passwords on the admin login page, but it was not successful. Also we searched for exploits available on google but didn't find anything.
![[Pasted image 20260216122232.png]]

Later, I obtained the following information on port 8000.
![[Pasted image 20260216122354.png]]
![[Pasted image 20260216122819.png]]

We To determine the version, I used a `curl` command to send an HTTP request and inspect the response headers. 

![[Pasted image 20260216122612.png]]
I also started looking for exploits regarding the ZeroMQ service on port 4505 and 4506.
![[Pasted image 20260216123134.png]]
![[Pasted image 20260216123300.png]]
![[Pasted image 20260216123649.png]]
I downloaded the python file from the link below.
https://github.com/jasperla/CVE-2020-11651-poc?source=post_page-----8957372f9c36---------------------------------------
https://github.com/Al1ex/CVE-2020-11652

