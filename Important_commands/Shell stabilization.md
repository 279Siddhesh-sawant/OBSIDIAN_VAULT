```sh
python3 -c 'import pty; pty.spawn("/bin/bash")'

Then, ctrl+z
stty raw -echo; fg  #press enter
reset
xterm
export TERM=xterm
stty size
stty rows 24 columns 80
```
