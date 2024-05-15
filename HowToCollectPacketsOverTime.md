---
title: How to Collect TCP Packets Over Time
layout: note
tags:
  - Linux
  - wireshark
  - tshark
  - dumpcap
  - tcp
  - network
---


How to rotate file capturing packets: Once per day, keep 7 days only.
How to filter packets: TCP to Specific IP and port only.

tshark and wireshark both use dumpcap.
dumpcap allows options to roll the captured logs.

```shell
dumpcap -i etho0 -w file.cap -b filesize:16384 -b files:1024 -b duration:3600 -f "tcp" -f "host 192.168.60.101" -f "port 502" &
```

- interface: etho0
- output file: file.cap
- files limited to 16 mb (16834 bytes)
- number of files limited to 1024, deleting the first after the 1024th file is created
- length of time for each file to capture would be 1 hour (60 minutes, or 3600 seconds)

Must run as root, and in a definately root owned directory, such as /home/

End the command with the ampersand so you can go about your day.
Regardless of the fact that you've commanded the shell to run dumcap in the background, the program will insist on outputting the number of packets captured wherever on your terminal you happen to be trying to work. But since you've sent it to the background, you can safely close that terminal and open a new one to escape this rude interruption.



