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
