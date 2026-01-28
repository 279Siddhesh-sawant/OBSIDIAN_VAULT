Password spraying is an attack technique where a small set of common passwords is tested across many accounts. Unlike brute-force attacks, password spraying avoids account lockouts by testing each account with only a few attempts, exploiting poor password practices common in many organisations. Password spraying is often effective because many organisations:

- Require frequent password changes, leading users to pick predictable patterns (for example, `Summer2025!`).
- Don't enforce their policies well.
- Reuse common passwords across multiple accounts.

Common password lists for spraying include:

- Seasonal passwords.
- Default passwords used by IT teams (`Password123`).
- Passwords leaked in data breaches, like `rockyou.txt`.

## Password Policy

Before we can start our attack, it is essential to understand our target's password policy. This will allow us to retrieve information about the minimum password length, complexity, and the number of failed attempts that will lock out an account.

**rpcclient**

We can use rpcclient via a null session to query the DC for the password policy:

`rpcclient -U "" 10.211.11.10 -N`

And then we can run the `getdompwinfo` command:

![](Active%20Directory/AD_Basic_Enumeration/Images/11.png)

**CrackMapExec**

**CrackMapExec** is a well-known network service exploitation tool that we will use throughout this module. It allows us to perform enumeration, command execution, and post-exploitation attacks in Windows environments. It supports various network protocols, such as SMB, LDAP, RDP, and SSH. If anonymous access is permitted, we can retrieve the password policy without credentials with the following command:

![](Active%20Directory/AD_Basic_Enumeration/Images/12.png)

## Performing Password Spraying Attacks

We have gathered a solid user list from our user enumeration in the previous task; we now need to create a small list of common passwords.  
Through our password policy enumeration, we saw that the password complexity is equal to 1:

- In **rpcclient**: `password_properties: 0x00000001`
- With **CrackMapExec**: `Password Complexity Flags: 000001`

This means that at least three of the following four conditions need to be respected for a password to be created:

1. Uppercase letters
2. Lowercase letters
3. Digits
4. Special characters

Also, passwords cannot contain the user's account name or parts of their full name exceeding two consecutive characters. We can refer to [this link](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh994562\(v=ws.11\)) to understand Microsoft's password complexity definitions.

Let's imagine that through some OSINT, we discovered that this company was in a data breach, and some of the known passwords were variations of the string "Password". We can create the following list, making sure to respect the password policy:

- `Password!`
- `Password1`
- `Password1!`
- `P@ssword`
- `Pa55word1`

We can use **CrackMapExec** to run our password spraying attack against the WRK computer:
![](Active%20Directory/AD_Basic_Enumeration/Images/13.png)