#### Running nmap scan
```
nmap -A -sV -sC -v 10.48.136.177
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-30 11:36 IST
NSE: Loaded 157 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 11:36
Completed NSE at 11:36, 0.00s elapsed
Initiating NSE at 11:36
Completed NSE at 11:36, 0.00s elapsed
Initiating NSE at 11:36
Completed NSE at 11:36, 0.00s elapsed
Initiating Ping Scan at 11:36
Scanning 10.48.136.177 [4 ports]
Completed Ping Scan at 11:36, 0.26s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 11:36
Completed Parallel DNS resolution of 1 host. at 11:36, 0.14s elapsed
Initiating SYN Stealth Scan at 11:36
Scanning 10.48.136.177 [1000 ports]
Discovered open port 80/tcp on 10.48.136.177
Discovered open port 445/tcp on 10.48.136.177
Discovered open port 21/tcp on 10.48.136.177
Discovered open port 22/tcp on 10.48.136.177
Discovered open port 111/tcp on 10.48.136.177
Discovered open port 139/tcp on 10.48.136.177
Discovered open port 2049/tcp on 10.48.136.177
Completed SYN Stealth Scan at 11:36, 3.12s elapsed (1000 total ports)
Initiating Service scan at 11:36
Scanning 7 services on 10.48.136.177
Completed Service scan at 11:36, 11.41s elapsed (7 services on 1 host)
Initiating OS detection (try #1) against 10.48.136.177
Initiating Traceroute at 11:36
Completed Traceroute at 11:36, 3.02s elapsed
Initiating Parallel DNS resolution of 2 hosts. at 11:36
Completed Parallel DNS resolution of 2 hosts. at 11:36, 0.31s elapsed
NSE: Script scanning 10.48.136.177.
Initiating NSE at 11:36
Completed NSE at 11:36, 5.88s elapsed
Initiating NSE at 11:36
Completed NSE at 11:36, 1.23s elapsed
Initiating NSE at 11:36
Completed NSE at 11:36, 0.00s elapsed
Nmap scan report for 10.48.136.177
Host is up (0.18s latency).
Not shown: 993 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         ProFTPD 1.3.5
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 88:9a:f5:b8:7f:04:ea:a3:a7:ad:96:e7:c2:1d:3e:e9 (RSA)
|   256 50:49:71:6b:56:44:27:5b:7d:8e:75:ee:8c:d2:88:26 (ECDSA)
|_  256 66:f6:4a:d9:65:24:c5:6f:43:f0:28:d6:6f:db:fe:59 (ED25519)
80/tcp   open  http        Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
| http-robots.txt: 1 disallowed entry 
|_/admin.html
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-server-header: Apache/2.4.41 (Ubuntu)
111/tcp  open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      34634/udp   mountd
|   100005  1,2,3      43490/udp6  mountd
|   100005  1,2,3      49909/tcp   mountd
|   100005  1,2,3      59679/tcp6  mountd
|   100021  1,3,4      40297/tcp   nlockmgr
|   100021  1,3,4      46791/tcp6  nlockmgr
|   100021  1,3,4      48275/udp6  nlockmgr
|   100021  1,3,4      49579/udp   nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
139/tcp  open  netbios-ssn Samba smbd 4
445/tcp  open  netbios-ssn Samba smbd 4
2049/tcp open  nfs         3-4 (RPC #100003)
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Uptime guess: 28.905 days (since Sat Nov  1 13:53:58 2025)
Network Distance: 3 hops
TCP Sequence Prediction: Difficulty=264 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-11-30T06:06:40
|_  start_date: N/A
| nbstat: NetBIOS name: , NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   <00>                 Flags: <unique><active>
|   <03>                 Flags: <unique><active>
|   <20>                 Flags: <unique><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|_  WORKGROUP<1e>        Flags: <group><active>

TRACEROUTE (using port 3389/tcp)
HOP RTT       ADDRESS
1   363.40 ms 192.168.128.1
2   ...
3   168.67 ms 10.48.136.177

NSE: Script Post-scanning.
Initiating NSE at 11:36
Completed NSE at 11:36, 0.00s elapsed
Initiating NSE at 11:36
Completed NSE at 11:36, 0.00s elapsed
Initiating NSE at 11:36
Completed NSE at 11:36, 0.00s elapsed
Read data files from: /usr/share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.30 seconds
           Raw packets sent: 1139 (50.902KB) | Rcvd: 1126 (45.806KB)
```
