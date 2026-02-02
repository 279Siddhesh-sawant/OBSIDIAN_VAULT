This task will focus on manual enumeration using native CMD and PowerShell commands. This will include exploring token privileges, identifying domain and local users (including service accounts), analyzing logged-in sessions, and collecting valuable data from environment variables and the Windows Registry.

So, you’ve landed a shell on a Windows machine and want to discover the environment around you. For those more used to graphical applications, the command line may seem intimidating initially; however, once you learn the essential commands for your objectives, you will enjoy its versatility and power.

Imagine finding yourself in an unfamiliar office building at night; what would you do? With the mindset of a penetration tester (with proper written authorization), you will start your discovery journey. For instance, you might check the name of the office you landed in, look at other offices for nameplates, and check which doors are unlocked. Manual enumeration is the cyber-equivalent: you will check who you are, who else is around, and what doors (privileges) you have keys to.

In a way, you will play detective; you need to figure out where you are and what you can do. In this task, you will use built-in tools, CMD and PowerShell, to gather information as quietly as possible. Since you will be using built-in MS Windows tools, this tactic is known as living off the land (LOTL). Just like attackers, penetration testers often rely on native commands like net to enumerate users, groups, and configurations, often without triggering alarms. In this task, we will walk you through these native commands to map out the system and domain and fulfil our detective role.
Who Am I?

The first question any attacker (or philosopher) would ask is: “Who am I?” On a box, that’s the whoami command. To follow along, use the Terminal on the AttackBox to ssh into the Workstation with the IP address 10.211.12.20. We will use the following credentials:

    Username: asrepuser1
    Password: qwerty123!

Connecting over SSH is shown in the terminal below:

We just landed on a MS Windows Command Prompt. If your terminal shows tryhackme\asrepuser1, that’s the domain and username you are logged on with; WRK is the hostname. Now, go ahead and run:

![](Active%20Directory/AD_Authenticated_Enumeration/Images/4.png)
This command will output the current user’s username and domain, if applicable. In other words, if you see a domain name before the slash, you know that you are on a domain account, for example, DomainName\DomainUser. On the other hand, if you see a computer name, this would indicate a local user account, such as ComputerName\LocalUser.

To learn about the account’s groups and privileges, we can use whoami /all. This command will output a detailed page containing the account’s Security Identifier (SID), group memberships, and account privileges.

## Privileges

Let’s list some high privileges that can be pivotal in planning your next steps. The most interesting privileges to check for are:

SeImpersonatePrivilege: As mentioned already, this privilege allows a process to impersonate the security context of another user after authentication. The “potato” attack revolves around abusing this privilege.
SeAssignPrimaryTokenPrivilege: This privilege permits a process to assign the primary token of another user to a new process. It is used in conjunction with the SeImpersonatePrivilege privilege.
 SeBackupPrivilege: This privilege lets users read any file on the system, ignoring file permissions. Consequently, attackers can use it to dump sensitive files like the SAM or SYSTEM hive.
SeRestorePrivilege: This privilege grants the ability to write to any file or registry key without adhering to the set file permissions. Hence, it can be abused to overwrite critical system files or registry settings.
SeDebugPrivilege: This privilege allows the account to attach a debugger to any process. As a result, the attacker can use this privilege to dump memory from LSASS and extract credentials or even inject malicious code into privileged processes.

In brief, whoami /all informs you of your current power, be it due to group memberships or due to privileges. It is essential to note your findings as this tells your starting point.
System and Domain Information

Now that you have learned who you are, it is time to find out about the system you’re on. Some burning questions are: Is this machine part of an Active Directory domain? What is its hostname? What domain or workgroup is it in?

Let’s start with some basic commands. The following will help you learn about where you have “landed”:

    hostname
    systeminfo (requires administrator privileges)
    set

As the name implies, hostname prints the computer’s hostname. In many cases, the hostname can give you a hint about the role of the server. For instance, in many companies, it is not uncommon to see terms such as dc in the names of domain controllers and pc followed by digits as the names of computers joined to the domain.

On the other hand, systeminfo displays a wealth of computer information from the OS version to the installed hotfixes and passes through the domain or workgroup information. This information can be pretty valuable for penetration testers. Hint: You can filter the output using systeminfo | findstr /B "OS" to learn the OS name and version, while systeminfo | findstr /B "Domain" will return the name of the domain if it is part of an Active Directory.

Finally, we should not forget the information that can be discovered in environment variables. Just run set, and you will be presented with a non-trivial list of variables showing various information from the user’s home directory to the user's domain. You should note that the USERDOMAIN is set to the computer name unless the computer is joined to a domain. If you are on a PowerShell prompt, use Get-ChildItem Env: or simply dir env: instead of set.
![](Active%20Directory/AD_Authenticated_Enumeration/Images/5.png)

## Enumerating Users and Groups with NET commands (CMD)

So far, you know who you are on the host and who is (or isn’t) part of a particular domain. Furthermore, you know that your account belongs to certain groups and has specific privileges. It is time to discover more information about the domain itself; it is time to enumerate users and groups.

NET is a suite of commands for viewing and managing networked resources, and they run on the good old command prompt. Being present on every MS Windows system makes mastering them extremely useful and wise. There is a high chance that using NET will spare you from setting up new tools to gather information about the users, computers, groups, and other domain information. Even advanced adversaries use these for reconnaissance because they blend in with regular admin activity. Running NET HELP shows you all the available commands.

![](Active%20Directory/AD_Authenticated_Enumeration/Images/6.png)

Domain Users

First, try listing all the users in the domain using net user /domain. An example output is shown below. The list of users is long, so we only show the first two lines.

It should be noted that using net user will query the computer for local accounts and list them, while net user /domain will query the domain controller for a list of domain user accounts. Because the latter will list every user account in the Active Directory, this can take quite a long time, depending on the company’s size. Furthermore, the list can help discover interesting targets such as BackupAdmin.
![](Active%20Directory/AD_Authenticated_Enumeration/Images/7.png)
![](Active%20Directory/AD_Authenticated_Enumeration/Images/8.png)
From here, getting more information about any user account would be easy using net user <username> /domain. This command will return the full name, account status (active or inactive), information about the password, group memberships, and last logon time. The output gives you a good idea about the target account permissions and whether they are being used. An example is shown in the terminal below:

