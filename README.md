
# VPS Server Security Guide

This is a step-by-step guide to secure your VPS server right after you purchase it. Following this guide will help you secure your server and reduce the risk of attacks.

---

## 1. Access the VPS Server
After purchasing your server, you'll need to access it via SSH.

```bash
ssh root@your-server-ip
```

Make sure you use a **strong password** or, ideally, **SSH key-based authentication**.

---

## 2. Update the System
Start by updating your server's software. This ensures that you have the latest security patches and updates.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt dist-upgrade -y
```

---

## 3. Create a Non-Root User
For security, never log in as the `root` user directly. Instead, create a non-root user with `sudo` privileges.

```bash
adduser yourusername
usermod -aG sudo yourusername
```

---

## 4. Set Up SSH Key Authentication (Recommended)
SSH keys are much more secure than passwords. Here’s how you can set them up.

1. On your local machine, generate an SSH key (if you don't have one already):

   ```bash
   ssh-keygen -t rsa -b 4096
   ```

   Follow the prompts to save the key in the default location (`~/.ssh/id_rsa`).

2. Copy the public key to your server:

   ```bash
   ssh-copy-id yourusername@your-server-ip
   ```

3. Now, you can log in without needing a password:

   ```bash
   ssh yourusername@your-server-ip
   ```

---

## 5. Disable Root SSH Login
Disabling root login ensures that attackers can’t try to brute-force the `root` account.

1. Open the SSH configuration file:
   
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Find and modify the following line:
   
   ```bash
   PermitRootLogin no
   ```

3. Save and exit the file (CTRL+X, Y, Enter).

4. Restart SSH service:

   ```bash
   sudo systemctl restart sshd
   ```

---

## 6. Install and Configure a Firewall (UFW)

UFW (Uncomplicated Firewall) is a simple and effective way to control which ports are open on your server.

1. Install UFW:

   ```bash
   sudo apt install ufw
   ```

2. Allow SSH connections (so you don’t lock yourself out):

   ```bash
   sudo ufw allow OpenSSH
   ```

3. Enable UFW:

   ```bash
   sudo ufw enable
   ```

4. Check the status to see which ports are open:

   ```bash
   sudo ufw status
   ```

5. Allow additional services as needed, e.g., HTTP/HTTPS for a web server:

   ```bash
   sudo ufw allow http
   sudo ufw allow https
   ```

---

## 7. Disable Unused Services

Go through the services running on your server and disable any that are unnecessary. 

1. Check active services:

   ```bash
   sudo systemctl list-units --type=service
   ```

2. Disable a service you don’t need, e.g., `telnet`:

   ```bash
   sudo systemctl stop telnet
   sudo systemctl disable telnet
   ```

---

## 8. Install Fail2Ban (Protection Against Brute Force Attacks)

**Fail2Ban** will monitor log files for repeated failed login attempts and automatically block IPs that try to brute-force login.

1. Install Fail2Ban:

   ```bash
   sudo apt install fail2ban
   ```

2. Start the service:

   ```bash
   sudo systemctl start fail2ban
   sudo systemctl enable fail2ban
   ```

3. Check the status:

   ```bash
   sudo systemctl status fail2ban
   ```

---

## 9. Keep Your System Secure with Automatic Updates

Set up automatic security updates to keep your system patched.

1. Install the unattended-upgrades package:

   ```bash
   sudo apt install unattended-upgrades
   ```

2. Configure automatic security updates:

   ```bash
   sudo dpkg-reconfigure --priority=low unattended-upgrades
   ```

---

## 10. Set Up SSH Rate Limiting
Prevent brute force attacks by limiting the number of SSH login attempts.

1. Install `iptables` to rate-limit SSH:

   ```bash
   sudo apt install iptables
   ```

2. Add the rule:

   ```bash
   sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
   sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 5 -j DROP
   ```

3. Save the iptables configuration:

   ```bash
   sudo iptables-save > /etc/iptables/rules.v4
   ```

---

## 11. Secure Shared Memory
Prevent certain types of exploits by securing the `/dev/shm` directory.

1. Add the following line to `/etc/fstab`:

   ```bash
   sudo nano /etc/fstab
   ```

   Add:

   ```
   tmpfs /dev/shm tmpfs defaults,noexec,nosuid 0 0
   ```

2. Save and exit, then remount:

   ```bash
   sudo mount -o remount /dev/shm
   ```

---

## 12. Regular Backups
Backups are critical in case your server is compromised. Set up a backup solution (e.g., **rsync**, **duplicity**, or a cloud backup service).

1. You can back up files like this:

   ```bash
   rsync -avz /your-files /path/to/backup/
   ```

---

## 13. Monitor Server Activity

Set up tools to monitor server activity, logs, and system health:

- **Logwatch**: Summarizes log activity.

   ```bash
   sudo apt install logwatch
   ```

- **Sysstat**: Monitors CPU usage, memory, etc.

   ```bash
   sudo apt install sysstat
   ```

---

## 14. Disable USB / External Devices (Optional)  
If your server doesn’t need USB or external device support, disable it to prevent certain types of attacks.

```bash
sudo systemctl mask dev-sda.device
```

---

## 15. Set Up Two-Factor Authentication (Optional, but Recommended)

For even more security, set up 2FA on your server (e.g., Google Authenticator for SSH).

1. Install `pam_google_authenticator`:

   ```bash
   sudo apt install libpam-google-authenticator
   ```

2. Configure it for your user:

   ```bash
   google-authenticator
   ```

---

## 16. Review Server Logs Regularly
Check logs often to spot any suspicious activity:

```bash
sudo tail -f /var/log/auth.log
```

---

## 17. Regularly Review & Patch
Periodically check for new updates:

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Conclusion:
With these steps, you’ve set a solid foundation for securing your VPS server. This guide can be your go-to reference for securing a fresh VPS and maintaining its security over time.
