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
launched burp suite

