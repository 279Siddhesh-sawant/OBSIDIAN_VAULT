Nmap scan
```sh
nmap -p- --min-rate 5000 -T4 -Pn 192.168.192.33
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-05 16:54 IST
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
Warning: 192.168.192.33 giving up on port because retransmission cap hit (6).
Nmap scan report for 192.168.192.33
Host is up (0.96s latency).
Not shown: 41684 filtered tcp ports (no-response), 23849 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 94.69 seconds
```

```sh
nmap -sC -sV -T4 -Pn -p 22,80 192.168.192.33   
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-05 16:56 IST
Nmap scan report for 192.168.192.33
Host is up (0.19s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u2 (protocol 2.0)
| ssh-hostkey: 
|   3072 c9:c3:da:15:28:3b:f1:f8:9a:36:df:4d:36:6b:a7:44 (RSA)
|   256 26:03:2b:f6:da:90:1d:1b:ec:8d:8f:8d:1e:7e:3d:6b (ECDSA)
|_  256 fb:43:b2:b0:19:2f:d3:f6:bc:aa:60:67:ab:c1:af:37 (ED25519)
80/tcp open  http    Apache httpd 2.4.56 ((Debian))
|_http-title: MZEE-AV - Check your files
|_http-server-header: Apache/2.4.56 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.00 seconds
```

Visiting web server on port 80.
![](Mzeeav1.png)

Directory brute forcing.
![](Mzeeav2.png)
![](Mzeeav3.png)
![](Mzeeav4.png)
I downloaded and extracted them on my working directory :
![](Mzeeav5.png)
If we cat the upload.php :
![](Mzeeav6.png)

This PHP code allows users to upload files to a server, but it only accepts **PE (Portable Executable)** files. Executable files on Windows (e.g., `.exe`, `.dll`) are in PE format, and every PE file is identified by the **"MZ"** characters at the beginning. In hexadecimal (hex) format, "MZ" appears as **4D5A**.

Now, let’s break down the code **step by step** in a simple way:

1. **Getting the File Name**

  
$filename = $_FILES[‘file’][‘name’];

The user uploads a file via an HTML form.

$_FILES[‘file’][‘name’] gets the name of the uploaded file and stores it in the $filename variable.  
For example, if the user uploads malicious.exe, $filename = malicious.exe

**2. Defining the File Save Locations**

$tmp_location = “upload/file.tmp”;  
$location = “upload/”.$filename;

- $tmp_location defines where the file will be saved **temporarily**: “upload/file.tmp”.
- $location defines where the file will be saved **permanently**: “upload/malicious.exe”.

**Purpose:**

- The file is first uploaded as file.tmp.
- If it’s a valid PE file, it’s saved with its original name.

**3. Moving the File to the Temporary Location**

move_uploaded_file($_FILES[‘file’][‘tmp_name’], $tmp_location);

- The move_uploaded_file() function moves the file from the server’s temporary folder ($_FILES[‘file’][‘tmp_name’]) to the upload/file.tmp file.
- After this step, the file is now on the server but not permanently saved yet.

**4. Reading the First 2 Bytes (Magic Bytes)**

$F = fopen($tmp_location, “r”);  
 $magic = fread($F, 2);  
 fclose($F);

- The fopen() function opens the file in **read** mode (“r”).
- The fread($F, 2) command reads the **first 2 bytes** of the file and stores them in the $magic variable.
- The fclose() function closes the file.

**Why the First 2 Bytes?**

- PE files start with the **“MZ”** characters.
- Computers read files in bytes, and each character equals one byte.
- “MZ” = 2 bytes, so checking the first 2 bytes is enough.

📌 **What Does “MZ” Mean?**

- The “MZ” signature refers to **Mark Zbikowski**, a Microsoft engineer.
- In the 1980s, while designing DOS executable files, he added his initials at the start of every executable file.
- These initials indicate that the file is a **PE (Portable Executable)** file.

**5. Converting the Bytes to Hex Format**

$magicbytes = strtoupper(substr(bin2hex($magic), 0, 4));

The purpose of this line is to convert the “MZ” characters into hexadecimal format. Here’s how it works:
**Definitions**

1. **Magic Bytes:** Special bytes used to identify the type of a file. For example:

- PE files: **“MZ”** → **4D5A**
- PNG images: **“PNG”** → **89504E47**
- PDF documents: **“%PDF”** → **25504446**

**2. PE (Portable Executable):** A file format used for executable files (EXE, DLL, SYS, etc.) on Windows operating systems.

**3. Hexadecimal (Hex):** A number system that uses digits 0–9 and letters A-F. For example:

- M → **4D**
- Z → **5A**

**4. ASCII:** A system that converts characters into numerical values. For example:

- M = 77 (decimal), **4D** (hex)
- Z = 90 (decimal), **5A** (hex)
-

##  Summary

This PHP code allows only PE files to be uploaded to the server. Here’s how it works:

1. The file is first uploaded as a temporary file.
2. The first 2 bytes are read to check if they match **“MZ” (4D5A)**.
3. If the file is valid, it’s saved permanently with its original name.
4. If it’s invalid, an error message is displayed, and the process is terminated.

Now, it’s time to design your file to bypass this check :

From the revshells.com, I copied PentestMonkey’s PHP Reverse Shell as revshell.php and added the “MZ” bytes at the beginning of the code.
![](Mzeeav7.png)
Then upload it :
![](Mzeeav8.png)
![](Mzeeav9.png)
Start the nc and visit the uploaded path **/upload** directory :
![](Mzeeav10.png)
![](Mzeeav11.png)
Found the local flag.
![](Mzeeav12.png)

### Privilege Escalation
Ran linpeas.
![](Mzeeav13.png)
 I found a strange suid bit :
![](Mzeeav15.png)
Let’s check its version to find out if there is a potential exploit :
![](Mzeeav16.png)
This output shows that when you run the `./fileS` command, it actually runs the GNU `find` command. So, there might be something in the `fileS` file that calls the `find` command.

**What’s Happening?**

- Normally, the `./fileS --version` command should display the version of the `fileS` file itself.
- But here, you’re getting the output `find (GNU findutils) 4.8.0`, which indicates that `fileS` is somehow running the `find` command.

**Why Is This Happening?**

- The `fileS` file might be a symlink to the `find` command or could be a script that calls the `find` command.

So, this file is caling for the find command. It’s a suid bit and if we check the GTFOBins :
![](Mzeeav17.png)
We can leverage this by :
![](Mzeeav18.png)


