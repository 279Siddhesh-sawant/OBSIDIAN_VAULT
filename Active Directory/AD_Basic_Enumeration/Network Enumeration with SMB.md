In this task, we assume that we have breached the perimeter of a corporate network and have access to a Linux-based box. We will be using the AttackBox; however, feel free to connect your favourite offensive Linux distribution over VPN.

We will focus on enumerating network shares using the Server Message Block (SMB) protocol. We will use various tools like Nmap to discover the relevant listening ports and identify services. Then, we will attempt to access their contents from the AttackBox using tools such as `smbclient` and `smbmap`. Furthermore, we will try to grab the contents of accessible SMB shares. Finally, we will mention a few more relevant tools. Let’s begin!

## Discovering Services

We will begin our “discovery” with the good old Nmap. We are mainly interested in MS Windows and Active Directory-related ports. We will limit our scan to the following ports:

- **TCP 88 (Kerberos)**: Kerberos uses this port for authentication in the Active Directory. From a penetration testing point of view, it can be a goldmine for ticket attacks like Pass-the-Ticket and Kerberoasting.
- **TCP 135 (RPC Endpoint Mapper)**: This TCP port is used for Remote Procedure Calls (RPC). It might be leveraged to identify services for lateral movement or remote code execution via DCOM.
- **TCP 139 (NetBIOS Session Service)**: This port is used for file sharing in older Windows systems. It can be abused for null sessions and information gathering.
- **TCP 389 (LDAP)**: This TCP port is used by the Lightweight Directory Access Protocol (LDAP). It is in plaintext and can be a prime target for enumerating AD objects, users, and policies.
- **TCP 445 (SMB)**: Critical for file sharing and remote admin; abused for exploits like EternalBlue, SMB relay attacks, and credential theft.
- **TCP 636 (LDAPS)**: This port is used by Secure LDAP. Although it is encrypted, it can still expose AD structure if misconfigured and can be abused via certificate-based attacks like AD CS exploitation.

We can use the Nmap scanner to check if any active services are listening on these ports, attempt to detect their versions with `-sV`, and allow default scripts to run with `-sC`. Our final command will be, `nmap -p 88,135,139,389,445,636 -sV -sC TARGET_IP`. The result of scanning the domain controller is shown below.

```sh
 nmap -A -T4 -sV 10.211.11.10 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-28 11:57 IST
Stats: 0:00:48 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 98.41% done; ETC: 11:58 (0:00:00 remaining)
Nmap scan report for 10.211.11.10
Host is up (0.17s latency).
Not shown: 986 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 57:d9:ac:9e:bc:f9:27:c9:8a:7f:01:28:72:9f:05:7a (RSA)
|   256 bc:21:8a:c6:9f:71:dc:8f:b7:e9:6b:71:71:cd:eb:ef (ECDSA)
|_  256 76:ba:01:61:c4:69:5b:c4:d6:8a:dd:8d:c7:32:7b:f3 (ED25519)
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-01-28 06:28:11Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: tryhackme.loc0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds  Windows Server 2019 Datacenter 17763 microsoft-ds (workgroup: TRYHACKME)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: tryhackme.loc0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=DC.tryhackme.loc
| Not valid before: 2026-01-26T06:31:45
|_Not valid after:  2026-07-28T06:31:45
| rdp-ntlm-info: 
|   Target_Name: TRYHACKME
|   NetBIOS_Domain_Name: TRYHACKME
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: tryhackme.loc
|   DNS_Computer_Name: DC.tryhackme.loc
|   DNS_Tree_Name: tryhackme.loc
|   Product_Version: 10.0.17763
|_  System_Time: 2026-01-28T06:28:35+00:00
|_ssl-date: 2026-01-28T06:28:48+00:00; 0s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=1/28%OT=22%CT=1%CU=30085%PV=Y%DS=2%DC=T%G=Y%TM=6979ACA
OS:7%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=106%TI=I%CI=I%II=I%SS=S%TS=
OS:U)SEQ(SP=104%GCD=1%ISR=10A%TI=I%CI=I%II=I%SS=S%TS=U)SEQ(SP=105%GCD=1%ISR
OS:=10A%TI=I%CI=I%II=I%SS=S%TS=U)SEQ(SP=107%GCD=1%ISR=10E%TI=I%CI=I%II=I%SS
OS:=S%TS=U)SEQ(SP=FE%GCD=1%ISR=104%TI=I%CI=I%II=I%SS=S%TS=U)OPS(O1=M509NW8N
OS:NS%O2=M509NW8NNS%O3=M509NW8%O4=M509NW8NNS%O5=M509NW8NNS%O6=M509NNS)WIN(W
OS:1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FF70)ECN(R=Y%DF=Y%T=80%W=FFFF%
OS:O=M509NW8NNS%CC=Y%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)T2(R=Y%DF=Y%
OS:T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y%DF=Y%T=80%W=0%S=Z%A=O%F=AR%O=%RD
OS:=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=80%W=0%S
OS:=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T7(R
OS:=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=80%IPL=164%UN=0%
OS:RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=80%CD=Z)

Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-os-discovery: 
|   OS: Windows Server 2019 Datacenter 17763 (Windows Server 2019 Datacenter 6.3)
|   Computer name: DC
|   NetBIOS computer name: DC\x00
|   Domain name: tryhackme.loc
|   Forest name: tryhackme.loc
|   FQDN: DC.tryhackme.loc
|_  System time: 2026-01-28T06:28:36+00:00
| smb2-time: 
|   date: 2026-01-28T06:28:38
|_  start_date: N/A
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

TRACEROUTE (using port 21/tcp)
HOP RTT       ADDRESS
1   176.29 ms 10.250.11.1
2   176.22 ms 10.211.11.10

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 61.78 seconds
```
The above open ports indicate an MS Windows Active Directory environment. The presence of Kerberos and LDAP ports might indicate that this is part of an Active Directory domain or even a domain controller.

## Listing SMB Shares

At this stage, we don’t have valid credentials. Let’s check the exposed SMB shares and see if we can connect to any of them anonymously. We will try an anonymous connection, also known as a null session because it uses no username or password, and see if we can get access. There are two good tools for enumerating SMB shares from a Linux box: `smbclient` and `smbmap`.

`smbclient` is a command-line tool that allows interaction with SMB shares and is part of the Samba suite. It is similar to an FTP client. You can use it to list, upload, download, and browse files on a remote SMB server. In the terminal below, we try to list the shares via the `-L` option, with no password, hence the `-N` option. We can see some interesting shares below running `smbclient -L //TARGET_IP -N`.

![](Active%20Directory/AD_Basic_Enumeration/Images/3.png)
Another tool is `smbmap`, a reconnaissance tool that enumerates SMB shares across a host. It can be used to display read and write permissions for each share. It’s instrumental for quickly identifying accessible or misconfigured shares without manually connecting to each one. Below is an example of running `smbmap -H TARGET_IP`. Note that `smbmap` is located in `/root/Desktop/Tools/Miscellaneous/smbmap` on the AttackBox.

![](Active%20Directory/AD_Basic_Enumeration/Images/4.png)

Running either of the above commands, we can notice that there are three non-standard shares that catch our attention: `AnonShare`, `SharedFiles` and `UserBackups`.

It is worth noting that you can also discover which shares grant access using Nmap. Using Nmap’s `smb-enum-shares` script, we can explore which shares give READ/WRITE, READ, or no access. The syntax is `nmap -p445 --script smb-enum-shares 10.211.11.10`.

## Accessing SMB Shares

Now that we have listed the shares, let’s attempt to access the ones that allow anonymous access. We might discover any interesting files that might help us gain access. We will target all the shares that showed `READ` access among their permissions when we ran `smbmap`. To use `smbclient` to connect to a share, you can use `smbclient //TARGET_IP/SHARE_NAME -N`. After connecting, we listed the files by issuing `ls` as shown below. Once you find the filename, you can download it using `get file_name`. In the terminal above, we did a `get Mouse_and_Malware.txt` to download the file to the AttackBox.
![](Active%20Directory/AD_Basic_Enumeration/Images/5.png)
![](Active%20Directory/AD_Basic_Enumeration/Images/6.png)
![](Active%20Directory/AD_Basic_Enumeration/Images/7.png)

Since we don’t have login credentials, we are trying to access the various shares without a password, i.e., with the `-N` option. However, if you have a username and password to access the SMB share, you can easily specify them with `--user=USERNAME --password=PASSWORD` or `-U 'username%password'`. Note that for domain accounts, you need to specify the domain using `-W`.

## What Can We Find in SMB Shares

An anonymous SMB share is a horrible idea. However, system administrators may have enabled it for legacy systems to work. Maybe an old printer needs to read a file, or an old scanner needs to write a file.

From a penetration testers’ point of view, SMB shares might contain configuration files, backup files, scripts, and even documents. After all, accessing a particular folder from any computer on the network is convenient. If the folder allows writing, it will only encourage users to upload more files. Some files might contain usernames or even credentials with a password that never got changed. In summary, it is essential to thoroughly search for any available shares as you might uncover useful hidden secrets.

## Other Tools and Resources

There are other tools that can be pretty useful as well. Examples include `impacket-smbclient`, which is a Python-based implementation of `smbclient` available from the Impacket toolkit. The Impacket toolkit is available on the AttackBox in the `/opt/impacket/examples/` directory.

[CrackMapExec](https://www.kali.org/tools/crackmapexec/) is not only for post-exploitation but also for enumeration. It includes many SMB modules for listing shares, testing credentials, and many others.

The `enum4linux` or `enum4linux-ng` is a powerful tool that performs extensive enumeration over SMB. You can try `enum4linux -a TARGET_IP` to get a trove of information. It might be a good idea to redirect the output to a file so that you can go through it slowly.

Finally, as mentioned earlier in this task, we should not forget Nmap with the `smb-enum-shares` script.