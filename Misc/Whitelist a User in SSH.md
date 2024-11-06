---
title: Whitelist a User in SSH
layout: note
tags:
  - SSH
---

Making an exception in ssh config for the key:

If you have the server set up to require keys, you can create an exception for a client at a specific IP to require a password instead.

First get the public IP of your machine from the command line:

```shell
curl ipecho.net/plain; echo
```

Then on the server, you can add the exception to the end of /etc/ssh/sshd_config:

```shell
Match Address myclientaddress.com
    PasswordAuthentication yes
```

Then restart the sshd:

```shell
sudo systemctl restart sshd.service
systemctl status sshd.service
```

Now all connection attempts from that IP will request a password instead of the key.
I don't think there's a way to just allow uninhibited connections from a specific IP
if the server is set up to require a key.
