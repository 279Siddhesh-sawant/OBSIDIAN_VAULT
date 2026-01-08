## Windows Vault and Credential Manager

[Credential Manager](https://learn.microsoft.com/en-us/windows-server/security/windows-authentication/credentials-processes-in-windows-authentication#windows-vault-and-credential-manager) is a feature built into Windows since `Server 2008 R2` and `Windows 7`. Thorough documentation on how it works is not publicly available, but essentially, it allows users and applications to securely store credentials relevant to other systems and websites. Credentials are stored in special encrypted folders on the computer under the user and system profiles ([MITRE ATT&CK](https://attack.mitre.org/techniques/T1555/004/)):

- `%UserProfile%\AppData\Local\Microsoft\Vault\`
- `%UserProfile%\AppData\Local\Microsoft\Credentials\`
- `%UserProfile%\AppData\Roaming\Microsoft\Vault\`
- `%ProgramData%\Microsoft\Vault\`
- `%SystemRoot%\System32\config\systemprofile\AppData\Roaming\Microsoft\Vault\`

Each vault folder contains a `Policy.vpol` file with AES keys (AES-128 or AES-256) that is protected by DPAPI. These AES keys are used to encrypt the credentials. Newer versions of Windows make use of `Credential Guard` to further protect the DPAPI master keys by storing them in secured memory enclaves ([Virtualization-based Security](https://learn.microsoft.com/en-us/windows-hardware/design/device-experiences/oem-vbs)).

Microsoft often refers to the protected stores as `Credential Lockers` (formerly `Windows Vaults`). Credential Manager is the user-facing feature/API, while the actual encrypted stores are the vault/locker folders. The following table lists the two types of credentials Windows stores:

![[ntds1.png]]

![[ntds2.png]]

It is possible to export Windows Vaults to `.crd` files either via Control Panel or with the following command. Backups created this way are encrypted with a password supplied by the user, and can be imported on other Windows systems.

![[ntds3.png]]

![Stored User Names and Passwords window showing a list of credentials with options to add, remove, or edit.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/308/img/CredMan_2.png)

## Enumerating credentials with cmdkey

We can use [cmdkey](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/cmdkey) to enumerate the credentials stored in the current user's profile:
![[ntds4.png]]
Stored credentials are listed with the following format:

![[ntds5.png]]

The first credential in the command output above, `virtualapp/didlogical`, is a generic credential used by Microsoft account/Windows Live services. The random looking username is an internal account ID. This entry may be ignored for our purposes.

The second credential, `Domain:interactive=SRV01\mcharles`, is a domain credential associated with the user SRV01\mcharles. `Interactive` means that the credential is used for interactive logon sessions. Whenever we come across this type of credential, we can use `runas` to impersonate the stored user like so:

![[ntds6.png]]

![[ntds7.png]]
## Extracting credentials with Mimikatz

There are many different tools that can be used to decrypt stored credentials. One of the tools we can use is [mimikatz](https://github.com/gentilkiwi/mimikatz). Even within `mimikatz`, there are multiple ways to attack these credentials - we can either dump credentials from memory using the `sekurlsa` module, or we can manually decrypt credentials using the `dpapi` module. For this example, we will target the LSASS process with `sekurlsa`:

![[ntds8.png]]

