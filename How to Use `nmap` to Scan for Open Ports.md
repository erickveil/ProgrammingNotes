---
title: How to Use `nmap` to Scan for Open Ports
layout: note
tags:
  - Linux
  - shell
  - terminal
  - command
  - bash
---

To scan for open and listening ports on the remote machine with IP address `192.168.0.152` using `nmap` from your macOS terminal, follow these steps:

1. Open your Terminal application.

2. Run the `nmap` command with the desired options. A basic scan to find open ports on the remote machine can be done with:

   ```sh
   nmap 192.168.0.152
   ```

3. For more detailed information, including service versions and OS detection, you can use the following options:

   ```sh
   sudo nmap -sS -sV -O 192.168.0.152
   ```

   - `-sS`: TCP SYN scan, which is faster and stealthier than a full TCP connection scan.
   - `-sV`: Service/version detection, which provides information about the software versions running on the open ports.
   - `-O`: OS detection, which attempts to determine the operating system of the remote machine.

4. If you want a quick scan for the most common ports, use:

   ```sh
   nmap -F 192.168.0.152
   ```

   - `-F`: Fast scan, which scans fewer ports than the default scan.

5. For scanning all 65535 TCP ports, use:

   ```sh
   sudo nmap -p- 192.168.0.152
   ```

   - `-p-`: Scans all 65535 ports.

After running the desired `nmap` command, `nmap` will perform the scan and display the results in your terminal. This will show which ports are open and listening on the remote machine.
