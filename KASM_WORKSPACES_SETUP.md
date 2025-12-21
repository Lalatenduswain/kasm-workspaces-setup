# Kasm Workspaces Installation & Setup Guide

## Table of Contents
- [Overview](#overview)
- [System Information](#system-information)
- [Installation Summary](#installation-summary)
- [Access Information](#access-information)
- [How to Use Kasm Workspaces](#how-to-use-kasm-workspaces)
- [Available Workspaces](#available-workspaces)
- [Auto-Start Configuration](#auto-start-configuration)
- [Management Commands](#management-commands)
- [Troubleshooting](#troubleshooting)
- [Security Recommendations](#security-recommendations)

---

## Overview

**Kasm Workspaces** is a container streaming platform that allows you to run browsers and desktop applications in isolated Docker containers and access them through your web browser.

**Benefits:**
- Access browsers without installing them locally
- Isolated environment for security
- No VM overhead - uses LXC containers
- Access from anywhere via web browser
- Multiple browser options available

---

## System Information

**Host System:**
- OS: Ubuntu 24.04.3 LTS (Noble Numbat)
- Platform: Linux 6.14.0-37-generic
- CPU: 16 cores
- RAM: 15GB
- Disk: 196GB total, 96GB available
- Installation Directory: `/opt/kasm/1.15.0`

**Kasm Version:** 1.15.0

**Network:**
- Tailscale IP: `100.94.23.26`
- HTTPS Port: `443`

---

## Installation Summary

Kasm Workspaces was installed with the following components:

1. **Database**: PostgreSQL 12 (container: `kasm_db`)
2. **API Server**: Kasm API (container: `kasm_api`)
3. **Web Proxy**: Nginx (container: `kasm_proxy`)
4. **Manager**: Kasm Manager (container: `kasm_manager`)
5. **Agent**: Kasm Agent (container: `kasm_agent`)
6. **Guacamole**: Remote desktop gateway (container: `kasm_guac`)
7. **Share Service**: File sharing (container: `kasm_share`)
8. **Redis**: Caching layer (container: `kasm_redis`)

**Installation Date:** December 21, 2025

---

## Access Information

### Web Interface URL
```
https://100.94.23.26
```

**Note:** You will see a SSL certificate warning because Kasm uses a self-signed certificate. This is normal - click "Advanced" and proceed to the site.

### Login Credentials

**Admin Account:**
- Username: `admin@kasm.local`
- Password: `password`
- Use this for: System administration, adding users, configuring workspaces

**User Account:**
- Username: `user@kasm.local`
- Password: `password`
- Use this for: Regular workspace usage

**⚠️ IMPORTANT:** Change these default passwords immediately after first login!

---

## How to Use Kasm Workspaces

### Step 1: Access Kasm
1. Open your web browser
2. Navigate to: `https://100.94.23.26`
3. Accept the security certificate warning
4. You'll see the Kasm login page

### Step 2: Login
1. Enter your username (e.g., `user@kasm.local`)
2. Enter your password (default: `password`)
3. Click "Login"

### Step 3: Launch a Browser
1. After login, you'll see the **Workspaces** page
2. Browse available workspaces (Chrome, Firefox, Edge, etc.)
3. Click on the browser you want to use
4. Click the **"Launch"** button
5. Wait a few seconds for the container to start
6. The browser will open in a new tab/window in your browser!

### Step 4: Use the Virtual Browser
- The browser runs completely isolated in a container
- You can browse the internet, download files, etc.
- Files can be uploaded/downloaded to/from the container
- When done, close the tab or click "Destroy Session"

---

## Available Workspaces

The following browser and application workspaces are pre-installed:

### Browsers
- **Chrome** - Google Chrome web browser
- **Firefox** - Mozilla Firefox web browser
- **Edge** - Microsoft Edge web browser
- **Brave** - Privacy-focused Brave browser
- **Chromium** - Open-source Chromium browser

### Applications
- **Discord** - Voice, video, and text chat
- **FileZilla** - FTP/FTPS client
- **GIMP** - Image editor
- **Insomnia** - API design platform
- **CentOS 7** - Full Linux desktop
- And many more...

---

## Auto-Start Configuration

✅ **Kasm is configured to start automatically on system boot!**

### Current Configuration:
- **Docker Service**: Enabled (starts on boot)
- **All Kasm Containers**: Restart policy set to `always`

### What This Means:
- When your Ubuntu VM reboots, Docker starts automatically
- Docker then starts all Kasm containers automatically
- No manual intervention needed
- Kasm will be accessible at `https://100.94.23.26` about 30-60 seconds after VM boot

### Verify Auto-Start:
```bash
# Check Docker is enabled
systemctl is-enabled docker

# Check container restart policies
sudo docker inspect kasm_proxy kasm_api kasm_db --format='{{.Name}}: {{.HostConfig.RestartPolicy.Name}}'
```

---

## Management Commands

### Check Kasm Status
```bash
sudo docker ps --filter "name=kasm" --format "table {{.Names}}\t{{.Status}}"
```

### Start Kasm Services
```bash
sudo /opt/kasm/1.15.0/bin/start
```

### Stop Kasm Services
```bash
sudo /opt/kasm/1.15.0/bin/stop
```

### Restart Kasm Services
```bash
sudo /opt/kasm/1.15.0/bin/stop
sudo /opt/kasm/1.15.0/bin/start
```

### View Container Logs
```bash
# API logs
sudo docker logs kasm_api --tail 50

# Proxy logs
sudo docker logs kasm_proxy --tail 50

# Database logs
sudo docker logs kasm_db --tail 50

# Manager logs
sudo docker logs kasm_manager --tail 50
```

### Database Access
```bash
# Connect to PostgreSQL database
sudo docker exec -it kasm_db psql -U kasmapp -d kasm

# List all users
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "SELECT username, disabled, locked FROM users;"

# List all workspaces
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "SELECT friendly_name, enabled FROM images;"
```

---

## Troubleshooting

### Cannot Access Web Interface

**Issue:** Cannot connect to `https://100.94.23.26`

**Solutions:**
1. Check if containers are running:
   ```bash
   sudo docker ps --filter "name=kasm"
   ```

2. Restart Kasm services:
   ```bash
   sudo /opt/kasm/1.15.0/bin/stop
   sudo /opt/kasm/1.15.0/bin/start
   ```

3. Check nginx proxy logs:
   ```bash
   sudo docker logs kasm_proxy --tail 100
   ```

### Login Issues

**Issue:** "Invalid password" or "Connection failed"

**Solutions:**
1. Check if account is locked:
   ```bash
   sudo docker exec kasm_db psql -U kasmapp -d kasm -c "SELECT username, locked, failed_pw_attempts FROM users WHERE username = 'admin@kasm.local';"
   ```

2. Unlock account:
   ```bash
   sudo docker exec kasm_db psql -U kasmapp -d kasm -c "UPDATE users SET locked = false, failed_pw_attempts = 0 WHERE username = 'admin@kasm.local';"
   ```

3. Reset password to "password":
   ```bash
   # Get the salt
   SALT=$(sudo docker exec kasm_db psql -U kasmapp -d kasm -t -c "SELECT salt FROM users WHERE username = 'admin@kasm.local';" | xargs)

   # Generate hash (password + salt)
   HASH=$(printf "password${SALT}" | sha256sum | cut -c-64)

   # Update password
   sudo docker exec kasm_db psql -U kasmapp -d kasm -c "UPDATE users SET pw_hash = '${HASH}', locked = false, failed_pw_attempts = 0 WHERE username = 'admin@kasm.local';"
   ```

### Workspace Won't Launch

**Issue:** Browser workspace fails to start

**Solutions:**
1. Check Docker has enough resources (disk space, memory)
   ```bash
   df -h
   free -h
   ```

2. Check if image is enabled:
   ```bash
   sudo docker exec kasm_db psql -U kasmapp -d kasm -c "SELECT friendly_name, enabled FROM images WHERE friendly_name = 'Chrome';"
   ```

3. Check agent logs:
   ```bash
   sudo docker logs kasm_agent --tail 100
   ```

### Language Settings

**Issue:** Interface is in French (or another language)

**Solution:** Kasm detects language from your browser settings. To change to English:

1. **In Browser (Edge/Chrome):**
   - Go to Settings → Languages
   - Move English to the top of the preferred languages list
   - Refresh Kasm page

2. **In Kasm (if available):**
   - Click profile icon (top-right)
   - Go to Settings/Paramètres
   - Look for language/langue option

---

## Security Recommendations

### 1. Change Default Passwords Immediately
```bash
# Access Kasm as admin
# Go to: Admin Panel → Users → Select user → Change Password
```

### 2. Use Strong Passwords
- Minimum 16 characters
- Mix of uppercase, lowercase, numbers, and symbols
- Don't reuse passwords from other services

### 3. Enable Two-Factor Authentication
- In user profile → "Two-Factor Authentication"
- Use an authenticator app (Google Authenticator, Authy, etc.)

### 4. Create Individual User Accounts
- Don't share the admin account
- Create separate accounts for each person
- Assign appropriate permissions

### 5. Configure Firewall (if needed)
```bash
# Only allow Tailscale traffic to port 443
sudo ufw allow from 100.0.0.0/8 to any port 443
sudo ufw enable
```

### 6. Regular Updates
```bash
# Check for Kasm updates periodically
# Visit: https://kasmweb.com/docs/latest/upgrade.html
```

### 7. SSL Certificate (Optional)
Replace the self-signed certificate with a proper SSL certificate:
- Use Let's Encrypt for Tailscale domains
- Or use your own trusted certificate

---

## Additional Resources

### Official Kasm Documentation
- Main Docs: https://kasmweb.com/docs/latest/
- Installation Guide: https://kasmweb.com/docs/latest/install.html
- User Guide: https://kasmweb.com/docs/latest/guide/user.html
- Admin Guide: https://kasmweb.com/docs/latest/guide/admin.html

### Community
- GitHub: https://github.com/kasmtech
- Discord: https://discord.gg/kasmweb
- Forum: https://forum.kasmweb.com/

### Support
- Issues: https://github.com/kasmtech/workspaces-issues/issues
- Documentation: https://kasmweb.com/docs

---

## Quick Reference

| Item | Value |
|------|-------|
| **URL** | https://100.94.23.26 |
| **Admin User** | admin@kasm.local |
| **Regular User** | user@kasm.local |
| **Default Password** | password (CHANGE THIS!) |
| **Installation Path** | /opt/kasm/1.15.0 |
| **Start Command** | sudo /opt/kasm/1.15.0/bin/start |
| **Stop Command** | sudo /opt/kasm/1.15.0/bin/stop |
| **Database Container** | kasm_db |
| **Web Proxy Container** | kasm_proxy |

---

## Installation History

This Kasm Workspaces installation was completed on December 21, 2025.

### What Was Done:
1. ✅ Installed Kasm Workspaces 1.15.0
2. ✅ Configured database with PostgreSQL
3. ✅ Set up HTTPS with self-signed certificate
4. ✅ Created admin and user accounts
5. ✅ Installed default browser workspaces (Chrome, Firefox, Edge, Brave, etc.)
6. ✅ Configured auto-start on system boot
7. ✅ Tested and verified all services working

### Access Verified:
- ✅ Web interface accessible via Tailscale
- ✅ Login working with both admin and user accounts
- ✅ Browser workspaces launching successfully
- ✅ Auto-start configured and tested

---

**Document Created:** December 21, 2025
**Last Updated:** December 21, 2025
**Kasm Version:** 1.15.0
**System:** Ubuntu 24.04.3 LTS
