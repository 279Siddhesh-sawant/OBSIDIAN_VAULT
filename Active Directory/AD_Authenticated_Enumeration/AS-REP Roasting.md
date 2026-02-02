Like Kerberoasting, AS-REP Roasting dumps user account hashes that have Kerberos pre-authentication disabled. Unlike Kerberoasting, these users do not need to be service accounts—the only requirement is that the “Do not require Kerberos preauthentication” flag **(UF_DONT_REQUIRE_PREAUTH)** is set on the user account.

During standard Kerberos authentication, the user’s hash encrypts a timestamp, which the Key Distribution Center (KDC) decrypts to verify the user’s identity. However, if pre-authentication is disabled, the KDC skips this verification step and returns an encrypted AS-REP blob without confirming the user’s identity. This blob can then be captured and cracked offline to recover the user’s password.
AS-REP Roasting in Two Phases

AS-REP Roasting involves two main steps: enumeration and exploitation. First, identify vulnerable user accounts, then capture and crack the retrieved AS-REP hashes offline.
**Phase 1: Enumeration – Identifying Vulnerable Accounts**

In this phase, you identify user accounts within Active Directory that have Kerberos pre-authentication disabled. Accounts without pre-authentication allow anyone on the network to request a Kerberos ticket (specifically an AS-REP) without first proving their identity. As a result, encrypted hashes of the account passwords become exposed and vulnerable to offline attacks.
Tools Used

Rubeus (Windows only)

A powerful Windows-based tool designed explicitly for Kerberos-related security testing and enumeration. Rubeus automatically identifies vulnerable accounts and retrieves encrypted AS-REP hashes.

E**xample Command: Rubeus.exe asreproast**

This command scans Active Directory, identifies accounts with pre-authentication disabled, and retrieves hashes ready for offline cracking.

Impacket’s GetNPUsers.py (Linux/Windows)

Impacket provides a flexible Python script (GetNPUsers.py) to enumerate accounts in non-Windows environments. To test for the pre-authentication vulnerability, you must supply a users.txt file containing usernames.

Example Command: 

```sh
GetNPUsers.py tryhackme.loc/ -dc-ip 10.211.12.10 -usersfile users.txt -format hashcat -outputfile hashes.txt -no-pass
```
This command enumerates usernames listed in users.txt and collects AS-REP hashes for vulnerable accounts, saving them in hashes.txt for offline cracking.
![](Active%20Directory/AD_Authenticated_Enumeration/Images/1.png)

**Phase 2: Exploitation – Cracking Password Hashes and Accessing the Network**

Once encrypted hashes are obtained in the enumeration phase, the next step involves offline cracking. Recovering valid passwords allows authentication as compromised users, granting further access or privilege escalation within the targeted network.
Tool Used

Hashcat (Cross-platform)

Hashcat is a highly efficient, widely used tool for password cracking. For AS-REP hashes specifically, Hashcat uses cracking mode 18200.

Example Command: hashcat -m 18200 hashes.txt wordlist.txt

Here’s what this command does:

    -m 18200: Specifies the AS-REP Kerberos hash cracking mode.
    hashes.txt: Contains collected hashes from vulnerable accounts.
    wordlist.txt: Your chosen dictionary of possible passwords.

Let’s run hashcat against the dumped hashes.txt file. We will use rockyou.txt, which can be found at /usr/share/wordlists/rockyou.txt. The command will take a couple of minutes to crack the password. The terminal output below shows that the password has been cracked. To keep the fun in the exercise, we have replaced the cracked password with [PASSWORD REDACTED] in the output below:

![](Active%20Directory/AD_Authenticated_Enumeration/Images/2.png)

```sh
hashcat -m 18200 hashes.txt /usr/share/wordlist/rockyou.txt
```

![](Active%20Directory/AD_Authenticated_Enumeration/Images/3.png)
