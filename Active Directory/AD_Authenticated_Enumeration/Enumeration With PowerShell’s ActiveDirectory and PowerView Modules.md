n Task 3, we relied heavily on the command prompt. Although the command prompt is quite useful, we can be more efficient if we have access to Windows PowerShell. This task is divided into two parts. In the first part, we explore the official ActiveDirectory module, while in the second part, we use the PowerView module from the PowerSploit framework.
Enumeration With the ActiveDirectory Module

The ActiveDirectory module is available on domain controllers. For other workstations and servers, you need to download Remote Server Administration Tools (RSAT) for Windows. Alternatively, certain repositories make the ActiveDirectory module available for easy installation without installing the whole RSAT.

To follow along, use the Terminal on the AttackBox to ssh into the Workstation with the IP address 10.211.12.20. We will use the following credentials:

Username: asrepuser1
Password: qwerty123!

Connecting over SSH is shown in the terminal below.
Then we need to start Windows PowerShell by executing powershell. Note that the PowerShell prompt starts with PS. The change from the Command Prompt to Windows PowerShell is shown in the terminal below.
To get started, check if the module is available; you can confirm by running Get-Module -ListAvailable ActiveDirectory in PowerShell. If it is available, you can easily import it using Import-Module ActiveDirectory. This is shown in the terminal below.

**User Enumeration**

You can get all users by running `Get-ADUser -Filter *` and watching PowerShell dump all accounts on your screen.
```PS
Get-ADUser -Filter *
```

![[25.png]]
![[28.png]]

```PS
Get-ADUser -Identity Administrator -Properties LastLogonDate,MemberOf,Title,Description,PwdLastSet
```


![[27.png]]
![[29.png]]
```PS
Get-ADUser -Filter "Name -like '*admin*'"
```
```PS
Get-ADGroup -Filter * | Select Name
```


![[30.png]]

![[31.png]]
![[32.png]]

```PS
Get-ADGroupMember -Identity "Remote Management Users"
```
```PS
Get-ADGroupMember -Identity "Domain Computers"
```
```PS
Get-ADDefaultDomainPasswordPolicy
```

	
![[33.png]]
![[34.png]]
![[35.png]]
You can check the various commands available via the official https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2025-ps 


![[36.png]]

![[37.png]]
![[38.png]]
![[39.png]]

Since the list is long, you might want to filter the output based on username. For example, to display usernames with admin inside them, we can specify `*admin*`
```PS
Get-DomainUser *admin*
```

![[40.png]]

![[41.png]]
```PS
Get-DomainGroup "*Admin*"
```


![[42.png]]

![[43.png]]

![[44.png]]
![[45.png]]
### How many groups did `Get-DomainGroup "*admin*"` return?
```PS
Get-DomainGroup "*Admin*" | Select-object "sameaccountname"
```

![[46.png]]
