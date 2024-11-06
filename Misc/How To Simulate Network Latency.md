---
title: How to Simulate Network Latency
layout: note
tags:
  - network
  - socket
  - Linux
---

- First check ping
- Note ms delay
- Then set delay:

```
sudo tc qdisc add dev eth0 root netem delay 120ms
```

- Then check ping. Note how it is old ping plus the 120 we set.

- Then restore:

```
sudo tc qdisc del dev eth0 root netem
```

The tc program is part of the netem package.
It is a default package on "most Linux default installs" (confirmed on Fedora and Ubuntu), so shouldn't need to be installed.

As a bonus trick: If latency can't be simulated on either the sender or receiver, you can MITM the delay using an ssh pipe.
