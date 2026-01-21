## Enumerating the Password Policy - from Linux - Credentialed

As stated in the previous section, we can pull the domain password policy in several ways, depending on how the domain is configured and whether or not we have valid domain credentials. With valid domain credentials, the password policy can also be obtained remotely using tools such as [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) or `rpcclient`.

![](IMages/ERPP1.png)

## Enumerating the Password Policy - from Linux - SMB NULL Sessions

Without credentials, we may be able to obtain the password policy via an SMB NULL session or LDAP anonymous bind. The first is via an SMB NULL session. SMB NULL sessions allow an unauthenticated attacker to retrieve information from the domain, such as a complete listing of users, groups, computers, user account attributes, and the domain password policy. SMB NULL session misconfigurations are often the result of legacy Domain Controllers being upgraded in place, ultimately bringing along insecure configurations, which existed by default in older versions of Windows Server.

When creating a domain in earlier versions of Windows Server, anonymous access was granted to certain shares, which allowed for domain enumeration. An SMB NULL session can be enumerated easily. For enumeration, we can use tools such as `enum4linux`, `CrackMapExec`, `rpcclient`, etc.

We can use [rpcclient](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html) to check a Domain Controller for SMB NULL session access.

Once connected, we can issue an RPC command such as `querydominfo` to obtain information about the domain and confirm NULL session access.

#### Using rpcclient

![](IMages/ERPP2.png)
We can also obtain the password policy. We can see that the password policy is relatively weak, allowing a minimum password of 8 characters.

#### Obtaining the Password Policy using rpcclient
![](IMages/ERPP3.png)

Let's try this using [enum4linux](https://labs.portcullis.co.uk/tools/enum4linux). `enum4linux` is a tool built around the [Samba suite of tools](https://www.samba.org/samba/docs/current/man-html/samba.7.html) `nmblookup`, `net`, `rpcclient` and `smbclient` to use for enumeration of windows hosts and domains. It can be found pre-installed on many different penetration testing distros, including Parrot Security Linux. Below we have an example output displaying information that can be provided by `enum4linux`. Here are some common enumeration tools and the ports they use:

![](IMages/ERPP4.png)
#### Using enum4linux
![](IMages/ERPP5.png)

The tool [enum4linux-ng](https://github.com/cddmp/enum4linux-ng) is a rewrite of `enum4linux` in Python, but has additional features such as the ability to export data as YAML or JSON files which can later be used to process the data further or feed it to other tools. It also supports colored output, among other features

#### Using enum4linux-ng

![](IMages/ERPP6.png)

Enum4linux-ng provided us with a bit clearer output and handy JSON and YAML output using the `-oA` flag.

#### Displaying the contents of ilfreight.json
![](IMages/ERPP7.png)

## Enumerating Null Session - from Windows

It is less common to do this type of null session attack from Windows, but we could use the command `net use \\host\ipc$ "" /u:""` to establish a null session from a windows machine and confirm if we can perform more of this type of attack.

#### Establish a null session from windows

![](IMages/ERPP8.png)

![](IMages/ERPP9.png)

## Enumerating the Password Policy - from Linux - LDAP Anonymous Bind

[LDAP anonymous binds](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/anonymous-ldap-operations-active-directory-disabled) allow unauthenticated attackers to retrieve information from the domain, such as a complete listing of users, groups, computers, user account attributes, and the domain password policy. This is a legacy configuration, and as of Windows Server 2003, only authenticated users are permitted to initiate LDAP requests. We still see this configuration from time to time as an admin may have needed to set up a particular application to allow anonymous binds and given out more than the intended amount of access, thereby giving unauthenticated users access to all objects in AD.

With an LDAP anonymous bind, we can use LDAP-specific enumeration tools such as `windapsearch.py`, `ldapsearch`, `ad-ldapdomaindump.py`, etc., to pull the password policy. With [ldapsearch](https://linux.die.net/man/1/ldapsearch), it can be a bit cumbersome but doable. One example command to get the password policy is as follows:

#### Using ldapsearch
![](IMages/ERPP10.png)
Note: In newer versions of `ldapsearch`, the `-h` parameter was deprecated in favor for `-H`.

Here we can see the minimum password length of 8, lockout threshold of 5, and password complexity is set (`pwdProperties` set to `1`).

---

## Enumerating the Password Policy - from Windows

If we can authenticate to the domain from a Windows host, we can use built-in Windows binaries such as `net.exe` to retrieve the password policy. We can also use various tools such as PowerView, CrackMapExec ported to Windows, SharpMapExec, SharpView, etc.

Using built-in commands is helpful if we land on a Windows system and cannot transfer tools to it, or we are positioned on a Windows system by the client, but have no way of getting tools onto it. One example using the built-in net.exe binary is:

#### Using net.exe
![](IMages/ERPP11.png)
Here we can glean the following information:

- Passwords never expire (Maximum password age set to Unlimited)
- The minimum password length is 8 so weak passwords are likely in use
- The lockout threshold is 5 wrong passwords
- Accounts remained locked out for 30 minutes

This password policy is excellent for password spraying. The eight-character minimum means that we can try common weak passwords such as `Welcome1`. The lockout threshold of 5 means that we can attempt 2-3 (to be safe) sprays every 31 minutes without the risk of locking out any accounts. If an account has been locked out, it will automatically unlock (without manual intervention from an admin) after 30 minutes, but we should avoid locking out `ANY` accounts at all costs.

PowerView is also quite handy for this:

#### Using PowerView

![](IMages/ERPP12.png)
PowerView gave us the same output as our `net accounts` command, just in a different format but also revealed that password complexity is enabled (`PasswordComplexity=1`).

As with Linux, we have many tools at our disposal to retrieve the password policy while on a Windows system, whether it is our attack system or a system provided by the client. PowerView/SharpView are always good bets, as are CrackMapExec, SharpMapExec, and others. The choice of tools depends on the goal of the assessment, stealth considerations, any anti-virus or EDR in place, and other potential restrictions on the target host. Let's cover a few examples.

---

## Analyzing the Password Policy

We've now pulled the password policy in numerous ways. Let's go through the policy for the INLANEFREIGHT.LOCAL domain piece by piece.

- The minimum password length is 8 (8 is very common, but nowadays, we are seeing more and more organizations enforce a 10-14 character password, which can remove some password options for us, but does not mitigate the password spraying vector completely)
- The account lockout threshold is 5 (it is not uncommon to see a lower threshold such as 3 or even no lockout threshold set at all)
- The lockout duration is 30 minutes (this may be higher or lower depending on the organization), so if we do accidentally lockout (avoid!!) an account, it will unlock after the 30-minute window passes
- Accounts unlock automatically (in some organizations, an admin must manually unlock the account). We never want to lockout accounts while performing password spraying, but we especially want to avoid locking out accounts in an organization where an admin would have to intervene and unlock hundreds (or thousands) of accounts by hand/script
- Password complexity is enabled, meaning that a user must choose a password with 3/4 of the following: an uppercase letter, lowercase letter, number, special character (`Password1` or `Welcome1` would satisfy the "complexity" requirement here, but are still clearly weak passwords).

The default password policy when a new domain is created is as follows, and there have been plenty of organizations that never changed this policy:
![](IMages/ERPP13.png)
## Next Steps

Now that we have the password policy in hand, we need to create a target user list to perform our password spraying attack. Remember that sometimes we will not be able to obtain the password policy if we are performing external password spraying (or if we are on an internal assessment and cannot retrieve the policy using any of the methods shown here). In these cases, we `MUST` exercise extreme caution not to lock out accounts. We can always ask our client for their password policy if the goal is as comprehensive an assessment as possible. If asking for the policy does not fit the expectations of the assessment or the client does not want to provide it, we should run one, max two, password spraying attempts (regardless of whether we are internal or external) and wait over an hour between attempts if we indeed decide to attempt two. While most organizations will have a lockout threshold of 5 bad password attempts, a lockout duration of 30 minutes and accounts will automatically unlock, we cannot always count on this being normal. I have seen plenty of organizations with a lockout threshold of 3, requiring an admin to intervene and unlock accounts manually.

`We do not want to be the pentester that locks out every account in the organization!`

Let's now prepare to launch our password spraying attacks by gathering a list of target users.
