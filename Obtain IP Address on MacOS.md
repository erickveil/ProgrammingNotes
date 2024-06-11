---
title: Obtain IP Address on MacOS
layout: note
tags:
  - bash
  - command
  - macOS
  - shell
---

Typically on a Linux system, I'll grab the IP Address fairly easily using `ipconfig`. But I've found that the output on my  macOS is very lengthy and not all relevent.

You can find your local IP address on macOS using the command line by using this command:

   ```bash
   ipconfig getifaddr en0
   ```

This command will typically give you the local IP address of your primary network interface (usually `en0` for Ethernet or Wi-Fi).

If you are connected via Wi-Fi and the above command does not work, you can try:

   ```bash
   ipconfig getifaddr en1
   ```

