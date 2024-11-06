---
title: How to Get an SSH Server on Windows With Cygwin
layout: note
tags:
  - Windows
  - Cygwin
  - SSH
---

Resource:
https://docs.oracle.com/cd/E24628_01/install.121/e22624/preinstall_req_cygwin_ssh.htm#EMBSC281

Install openssh [[How to Install a new Cygwin package without reinstalling the entire program]]

Run command:

```
ssh-host-config
```

Answer questions:
Stricth Mode: no
Install as service: yes
Enter value of CYGWIN: binmode ntsec

Then login as a user in the home directory with your user (Windows?) password.

Should run on startup, or execute:

```
net start cygsshd
```

The stop and pause commands work for net, but the status command does not.

==MUST RUN CYGWIN AS ADMINISTRATOR==


