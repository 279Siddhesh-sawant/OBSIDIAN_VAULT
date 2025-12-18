### Q.1 Download the file flag.txt from the web root using wget from the Pwnbox. Submit the contents of the file as your answer.
![](Images/1.png)

### Q.2 Upload the attached file named upload_win.zip to the target using the method of your choice. Once uploaded, unzip the archive, and run "hasher upload_win.txt" from the command line. Submit the generated hash as your answer.
RDP to 10.129.201.55 (ACADEMY-MISC-MS02), with user "htb-student" and password "HTB_@cademy_stdnt!" 

Download the file from web and simply copy paste it to target machine. Unzip it using extract all on desktop only.

![](Images/2.png)


![](Images/3.png)

## File Transfer Method 1
### Check test file MD5 Hash
![](Images/4.png)


### Encode file to Base64
![](Images/5.png)


### We can copy this content and paste it into a Windows PowerShell terminal and use some PowerShell functions to decode it. Confirming the md5 hashes match

``` powershell
PS C:\htb> [IO.File]::WriteAllBytes("C:\Users\htb-student\Desktop\deoded.txt", [Convert]::FromBase64String("SGVsbG8K"))
```

``` powershell
PS C:\htb> Get-FileHash C:\Users\htb-student\Desktop\decoded.txt -Algorithm md5
```


![](Images/6.png)


