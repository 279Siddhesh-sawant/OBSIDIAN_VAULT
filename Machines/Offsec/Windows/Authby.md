Nmap scan
```sh
nmap -p- --min-rate 5000 -T4 -Pn 192.168.225.46  
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-14 10:10 IST
Nmap scan report for 192.168.225.46
Host is up (0.059s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT     STATE SERVICE
21/tcp   open  ftp
242/tcp  open  direct
3145/tcp open  csi-lfap
3389/tcp open  ms-wbt-server

Nmap done: 1 IP address (1 host up) scanned in 26.49 seconds
```

```sh
nmap -sC -sV -T4 -Pn -p 21,242,3145,3389 192.168.225.46
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-14 10:11 IST
Nmap scan report for 192.168.225.46
Host is up (0.10s latency).

PORT     STATE SERVICE    VERSION
21/tcp   open  ftp        zFTPServer 6.0 build 2011-10-17
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| total 9680
| ----------   1 root     root      5610496 Oct 18  2011 zFTPServer.exe
| ----------   1 root     root           25 Feb 10  2011 UninstallService.bat
| ----------   1 root     root      4284928 Oct 18  2011 Uninstall.exe
| ----------   1 root     root           17 Aug 13  2011 StopService.bat
| ----------   1 root     root           18 Aug 13  2011 StartService.bat
| ----------   1 root     root         8736 Nov 09  2011 Settings.ini
| dr-xr-xr-x   1 root     root          512 Mar 14 11:41 log
| ----------   1 root     root         2275 Aug 08  2011 LICENSE.htm
| ----------   1 root     root           23 Feb 10  2011 InstallService.bat
| dr-xr-xr-x   1 root     root          512 Nov 08  2011 extensions
| dr-xr-xr-x   1 root     root          512 Nov 08  2011 certificates
|_dr-xr-xr-x   1 root     root          512 Oct 11 00:16 accounts
242/tcp  open  http       Apache httpd 2.2.21 ((Win32) PHP/5.3.8)
|_http-title: 401 Authorization Required
| http-auth: 
| HTTP/1.1 401 Authorization Required\x0D
|_  Basic realm=Qui e nuce nuculeum esse volt, frangit nucem!
|_http-server-header: Apache/2.2.21 (Win32) PHP/5.3.8
3145/tcp open  zftp-admin zFTPServer admin
3389/tcp open  tcpwrapped
| rdp-ntlm-info: 
|   Target_Name: LIVDA
|   NetBIOS_Domain_Name: LIVDA
|   NetBIOS_Computer_Name: LIVDA
|   DNS_Domain_Name: LIVDA
|   DNS_Computer_Name: LIVDA
|   Product_Version: 6.0.6001
|_  System_Time: 2026-03-14T04:42:14+00:00
|_ssl-date: 2026-03-14T04:42:29+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=LIVDA
| Not valid before: 2025-10-09T17:16:18
|_Not valid after:  2026-04-10T17:16:18
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.77 seconds
```

First, when accessing port 242 (HTTP), it presented a Basic Authentication prompt requesting a username and password.
![](Images/Authby1.png)

Since the Basic Auth credentials were unknown, I explored other services — specifically, attempting to access the FTP service using anonymous login, which turned out to be successful.
![](Images/Authby2.png)
Inside the `accounts` directory, there were three files named after users: `offsec`, `anonymous`, and `admin`.
![](Images/Authby3.png)
I then attempted to log in to the FTP server using those usernames, with the password set to match the username. As a result, I successfully logged in using the credentials `admin:admin`.
![](Images/Authby4.png)
