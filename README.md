# Synology DSM IPv6 Remote Access via Asuswrt-Merlin

This repository guides you through enabling IPv6 remote access to your Synology DSM using an ASUS RT-BE86U router running Asuswrt-Merlin firmware.

---

## 📖 Overview

By default, Synology DSM can be reached locally via IPv6 but often blocked from the broader internet by router firewalls or ISP policies. This guide shows how to:

1. Flash your ASUS RT-BE86U with Asuswrt-Merlin.
2. Enable SSH and JFFS scripts on the router.
3. Deploy a `firewall-start` script to allow IPv6 TCP 5001 inbound to your NAS.
4. Validate and test external access (e.g., on 4G or other WAN).
5. Harden security post-configuration.

---

## 📋 Prerequisites

- **Synology NAS** with DSM 7.x and a global IPv6 address (e.g., `2406:...:4a66`).
- **ASUS RT-BE86U** router, flashed with **Asuswrt-Merlin 3006.102.4** (or later).
- Basic familiarity with SSH and router administration.

---

## 🚀 Steps

1. **Flash Asuswrt-Merlin**
   - Download the `RT-BE86U_3006.102.4_0-merlin.trx` firmware from https://www.asuswrt-merlin.net/download
   - In the stock ASUS GUI, navigate to **Administration → Firmware Upgrade**, upload and install.
   - After reboot, perform a **Factory Reset** to clear conflicts.

2. **Enable SSH & JFFS**
   - SSH: **Administration → System → Services**, enable SSH on LAN.
   - JFFS: **Administration → System → JFFS Partition**, check **Enable JFFS custom scripts and configs**, apply and reboot.

3. **Deploy the firewall script**
   - SSH into your router:
     ```bash
     ssh username@192.168.50.1
     ```
   - Create `/jffs/scripts/firewall-start` with:
     ```sh
     #!/bin/sh 
     NAS6="2406:....:4a66" # Replace with your NAS IPv6
     ip6tables -I FORWARD -p tcp -d "$NAS6" --dport 5001 -j ACCEPT
     ```
   - Make it executable:
     ```bash
     chmod +x /jffs/scripts/firewall-start
     ```
   - Reboot router.

4. **Validate**
   - SSH back to router and confirm:
     ```bash
     ip6tables -S FORWARD | grep 5001
     ```
   - On a WAN network (e.g., 4G), test:
     ```bash
     curl -6 -vk https://your-domain.com:5001/
     ```

5. **Security Hardening**
   - DSM: close HTTP (5000), force HTTPS (5001), enable 2FA.
   - Router: restrict IPv6 source prefixes in ip6tables if desired.

---
