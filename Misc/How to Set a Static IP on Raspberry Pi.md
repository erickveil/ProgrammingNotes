---
title: How to Set a Static IP on Raspberry Pi
layout: note
tags:
  - Linux
  - raspi
  - network
---

To set a static IP on a Raspberry Pi, you need to modify the network configuration file. Here's how you can do it:

1. **Open the terminal** on your Raspberry Pi.

2. **Edit the `dhcpcd.conf` file**:
   You can use the `nano` text editor to edit the configuration file.

   ```bash
   sudo nano /etc/dhcpcd.conf
   ```

3. **Add static IP configuration**:
   Scroll down to the end of the file and add the following lines, modifying the IP address, router, and DNS to match your network's settings.

   ```bash
   interface eth0  # or wlan0 for Wi-Fi
   static ip_address=192.168.1.100/24  # Replace with your desired static IP
   static routers=192.168.1.1  # Your router's IP address
   static domain_name_servers=192.168.1.1 8.8.8.8  # Replace or add additional DNS if needed
   ```

   - Replace `eth0` with `wlan0` if you're using Wi-Fi.
   - Set `ip_address` to the static IP you want.
   - Set `routers` to your gateway or router's IP address.
   - Set `domain_name_servers` to your DNS server, which could be your router or a public DNS like Google's (8.8.8.8).

4. **Save the changes**:
   After editing the file, press `Ctrl + X` to exit, then `Y` to confirm saving the changes, and `Enter` to write to the file.

5. **Restart the networking service**:
   Apply the changes by restarting the DHCP client service.

   ```bash
   sudo systemctl restart dhcpcd
   ```

6. **Verify the static IP**:
   You can check if your Raspberry Pi has the static IP assigned by running:

   ```bash
   ip addr
   ```

This should display the network interfaces and show the assigned IP address for the interface you've configured (e.g., `eth0` or `wlan0`).

Your Raspberry Pi should now have the static IP you configured.