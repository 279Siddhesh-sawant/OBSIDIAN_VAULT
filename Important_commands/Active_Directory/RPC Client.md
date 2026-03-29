# 🔹 1. Domain Information

### 📌 `querydominfo`

rpcclient $ querydominfo

**What it does:**

- Displays basic domain details:
    - Domain name
    - Number of users/groups
    - Domain SID

**Why important:**

- Helps identify the **target domain structure**
- SID is useful for **RID-based enumeration**

---

# 🔹 2. User Enumeration

### 📌 `enumdomusers`

rpcclient $ enumdomusers

**What it does:**

- Lists all domain users with their **RID (Relative ID)**

**Output example:**

user:[Administrator] rid:[0x1f4]  
user:[john] rid:[0x3e8]

**Why important:**

- Gives valid usernames → useful for:
    - Password spraying
    - Bruteforce
    - Kerberoasting

---

### 📌 `queryuser <RID>`

rpcclient $ queryuser 0x3e8

**What it does:**

- Displays detailed info about a specific user:
    - Username
    - Full name
    - Last logon
    - Password last set
    - Account status

**Why important:**

- Identify:
    - Active users
    - Password age
    - Disabled accounts

---

# 🔹 3. Group Enumeration

### 📌 `enumdomgroups`

rpcclient $ enumdomgroups

**What it does:**

- Lists all domain groups with RIDs

---

### 📌 `querygroup <RID>`

rpcclient $ querygroup 0x200

**What it does:**

- Shows group details

---

### 📌 `querygroupmem <RID>`

rpcclient $ querygroupmem 0x200

**What it does:**

- Lists members of a group

**Why important:**

- Helps identify:
    - Admin users
    - Privileged accounts

---

# 🔹 4. Alias (Local Group) Enumeration

### 📌 `enumalsgroups builtin`

rpcclient $ enumalsgroups builtin

**What it does:**

- Lists built-in local groups (e.g., Administrators, Users)

---

### 📌 `queryaliasmem <RID>`

rpcclient $ queryaliasmem 0x220

**What it does:**

- Lists members of a local group

**Why important:**

- Identify:
    - Local admins
    - Privilege escalation targets

---

# 🔹 5. Password Policy

### 📌 `getdompwinfo`

rpcclient $ getdompwinfo

**What it does:**

- Displays domain password policy:
    - Minimum length
    - Complexity
    - Lockout settings

**Why important:**

- Helps in:
    - Password guessing strategy
    - Bruteforce planning

---

# 🔹 6. Share Enumeration

### 📌 `netshareenum`

rpcclient $ netshareenum

**What it does:**

- Lists shared folders on the target

**Why important:**

- Find:
    - Sensitive shares
    - Misconfigured permissions

---

# 🔹 7. Server Information

### 📌 `srvinfo`

rpcclient $ srvinfo

**What it does:**

- Displays server details:
    - OS version
    - Server name

**Why important:**

- Helps identify:
    - OS for exploit selection

---

# 🔹 8. SID Enumeration

### 📌 `lookupnames`

rpcclient $ lookupnames administrator

**What it does:**

- Converts username → SID

---

### 📌 `lookupsids`

rpcclient $ lookupsids S-1-5-21-XXXX

**What it does:**

- Converts SID → username

**Why important:**

- Useful in:
    - SID brute forcing
    - Expanding user lists

---

# 🔹 9. RID Cycling (Very Important 🔥)

### 📌 `lsaquery`

rpcclient $ lsaquery

**What it does:**

- Retrieves domain SID

---

Then manually enumerate users:

rpcclient $ lookupsids S-1-5-21-XXXX-500  
rpcclient $ lookupsids S-1-5-21-XXXX-501  
rpcclient $ lookupsids S-1-5-21-XXXX-1000

**Why important:**

- Works even when `enumdomusers` is blocked
- Helps discover hidden users

---

# 🔹 10. Session & Connection Info

### 📌 `enumdomains`

rpcclient $ enumdomains

**What it does:**

- Lists available domains

---

# 🔹 OSCP Tip (Very Important)

👉 Best enumeration flow inside `rpcclient`:

1. `lsaquery` → get SID
2. `enumdomusers` → get users
3. `enumdomgroups` → get groups
4. `querygroupmem` → find admins
5. `getdompwinfo` → password policy
6. `netshareenum` → shares