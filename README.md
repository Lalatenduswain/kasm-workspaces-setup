# Kasm Workspaces Setup - Complete Installation Guide

<div align="center">

![Kasm Workspaces](https://img.shields.io/badge/Kasm-1.15.0-blue)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04%20LTS-orange)
![Docker](https://img.shields.io/badge/Docker-Required-blue)
![License](https://img.shields.io/badge/License-MIT-green)

**Containerized Browser Environment with Persistent Profiles**

Access Chrome, Edge, Firefox and 50+ applications through your web browser - isolated, secure, and accessible from anywhere.

[Features](#features) • [Installation](#installation) • [Usage](#usage) • [Troubleshooting](#troubleshooting) • [Documentation](#documentation)

</div>

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
  - [System Preparation](#system-preparation)
  - [Kasm Installation](#kasm-installation)
  - [Post-Installation Configuration](#post-installation-configuration)
- [Configuration](#configuration)
  - [User Accounts](#user-accounts)
  - [Persistent Profiles](#persistent-profiles)
  - [Session Timeout](#session-timeout)
  - [SSL Certificate](#ssl-certificate)
- [Usage](#usage)
  - [Accessing Kasm](#accessing-kasm)
  - [Launching Browsers](#launching-browsers)
  - [Using Persistent Profiles](#using-persistent-profiles)
  - [File Transfer](#file-transfer)
- [Available Workspaces](#available-workspaces)
- [Security](#security)
- [Management](#management)
  - [Service Control](#service-control)
  - [Database Management](#database-management)
  - [Docker Commands](#docker-commands)
  - [Log Management](#log-management)
- [Troubleshooting](#troubleshooting)
- [Advanced Configuration](#advanced-configuration)
- [Performance Tuning](#performance-tuning)
- [Backup and Restore](#backup-and-restore)
- [Upgrading](#upgrading)
- [FAQ](#faq)
- [Contributing](#contributing)
- [License](#license)
- [Support](#support)

---

## Overview

**Kasm Workspaces** is an open-source container streaming platform that enables you to run browsers and desktop applications in isolated Docker containers accessible through your web browser. This repository contains a complete installation and configuration guide for deploying Kasm Workspaces on Ubuntu 24.04 LTS with persistent user profiles and optimized settings.

### Why Use Kasm Workspaces?

- 🔒 **Security**: Isolate browsing sessions from your host system
- 🌐 **Accessibility**: Access from any device with a web browser
- 💾 **Persistent Profiles**: Keep your logins, bookmarks, and settings
- 🚀 **No VM Overhead**: Uses lightweight LXC containers
- 🎯 **Multi-Browser**: Chrome, Edge, Firefox, Brave, and more
- 📱 **Cross-Platform**: Works on desktop, tablet, and mobile
- 🔐 **Privacy**: Browse without leaving traces on your local machine

---

## Features

### ✅ Implemented Features

- **Full Kasm Workspaces 1.15.0 Installation**
- **HTTPS Access** via self-signed certificate (Let's Encrypt ready)
- **Persistent Browser Profiles** for Chrome and Edge
- **Extended Session Timeout** (8 hours)
- **Auto-Start on Boot** (systemd + Docker restart policies)
- **Multiple User Support** with role-based access
- **Tailscale Integration** for secure remote access
- **59 Pre-configured Workspaces** (browsers, dev tools, office apps)
- **Docker-based Architecture** for easy scaling
- **PostgreSQL Database** for configuration and user data
- **Redis Caching** for improved performance
- **File Upload/Download** capabilities
- **Clipboard Sharing** between host and container
- **Audio Support** (experimental)

### 🎯 Workspaces Included

**Browsers:**
- Google Chrome
- Microsoft Edge
- Mozilla Firefox
- Brave Browser
- Chromium

**Development Tools:**
- VS Code
- Sublime Text
- Atom Editor
- GitKraken
- Insomnia (API testing)

**Productivity:**
- LibreOffice
- GIMP (Image Editor)
- Inkscape (Vector Graphics)
- Discord
- FileZilla (FTP Client)

**Operating Systems:**
- Ubuntu Desktop
- CentOS 7
- Kali Linux
- And more...

---

## Architecture

### System Components

```
┌─────────────────────────────────────────────────────────┐
│                    User's Browser                        │
│                  (HTTPS: Port 443)                       │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              Kasm Proxy (Nginx)                          │
│          - SSL Termination                               │
│          - Load Balancing                                │
│          - WebSocket Proxy                               │
└────────────────────┬────────────────────────────────────┘
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
┌──────────┐  ┌──────────┐  ┌──────────┐
│   API    │  │ Manager  │  │  Guac    │
│  Server  │  │ Service  │  │ Gateway  │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │             │
     └─────────────┼─────────────┘
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
┌──────────┐  ┌──────────┐  ┌──────────┐
│PostgreSQL│  │  Redis   │  │  Agent   │
│ Database │  │  Cache   │  │ Service  │
└──────────┘  └──────────┘  └────┬─────┘
                                  │
                                  ▼
                        ┌──────────────────┐
                        │  Browser         │
                        │  Containers      │
                        │  (Chrome, Edge,  │
                        │   Firefox, etc.) │
                        └──────────────────┘
```

### Docker Containers

| Container | Purpose | Port | Restart Policy |
|-----------|---------|------|----------------|
| kasm_proxy | Nginx reverse proxy | 443 | always |
| kasm_api | REST API server | 8080 | always |
| kasm_manager | Session management | 8181 | always |
| kasm_db | PostgreSQL database | 5432 | always |
| kasm_redis | Redis cache | 6379 | always |
| kasm_agent | Container orchestration | 4444 | always |
| kasm_guac | Guacamole gateway | - | always |
| kasm_share | File sharing service | 8182 | always |

---

## Prerequisites

### Hardware Requirements

**Minimum:**
- CPU: 2 cores
- RAM: 4GB
- Disk: 50GB SSD
- Network: 10 Mbps

**Recommended:**
- CPU: 4+ cores (tested with 16 cores)
- RAM: 8GB+ (tested with 15GB)
- Disk: 100GB+ SSD
- Network: 100 Mbps

### Software Requirements

- **OS**: Ubuntu 24.04 LTS (Noble Numbat)
  - Also compatible with: Ubuntu 22.04, Ubuntu 20.04, Debian 11/12
- **Docker**: Version 20.10+ (will be installed during setup)
- **Docker Compose**: Version 2.0+ (will be installed during setup)
- **Kernel**: Linux 5.4+
- **Internet**: Required for initial installation

### Network Requirements

- **Port 443**: HTTPS access (configurable)
- **Firewall**: Configure to allow incoming connections on port 443
- **DNS**: Optional - for custom domain names
- **Tailscale** (optional): For secure remote access

---

## Installation

### Quick Install (TL;DR)

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Download Kasm installer
cd /tmp
curl -O https://kasm-static-content.s3.amazonaws.com/kasm_release_1.15.0.06fdc8.tar.gz

# Extract
tar -xf kasm_release_1.15.0.06fdc8.tar.gz

# Install with EULA acceptance
cd kasm_release
sudo bash install.sh -e

# Access at https://YOUR_IP
```

### Detailed Installation

#### Step 1: System Preparation

```bash
# Update package lists
sudo apt update && sudo apt upgrade -y

# Install required dependencies
sudo apt install -y curl wget software-properties-common

# Check system resources
echo "CPU Cores: $(nproc)"
echo "RAM: $(free -h | grep Mem | awk '{print $2}')"
echo "Disk: $(df -h / | tail -1 | awk '{print $4}') available"
```

#### Step 2: Download Kasm Workspaces

```bash
# Create working directory
mkdir -p ~/kasm-install
cd ~/kasm-install

# Download latest version (1.15.0)
curl -O https://kasm-static-content.s3.amazonaws.com/kasm_release_1.15.0.06fdc8.tar.gz

# Verify download
ls -lh kasm_release_1.15.0.06fdc8.tar.gz

# Extract archive
tar -xf kasm_release_1.15.0.06fdc8.tar.gz
cd kasm_release
```

#### Step 3: Run Installer

```bash
# View installation options
sudo bash install.sh -h

# Install with automatic EULA acceptance
sudo bash install.sh -e

# OR install with custom settings
sudo bash install.sh \
  -e \
  --admin-password "YourSecurePassword" \
  --user-password "YourUserPassword" \
  --proxy-port 443
```

**Installation Output:**
The installer will:
1. Check for Docker (install if missing)
2. Pull required Docker images
3. Initialize PostgreSQL database
4. Create default user accounts
5. Start all services
6. Display credentials and access URL

**Expected Duration:** 5-10 minutes (depending on internet speed)

#### Step 4: Verify Installation

```bash
# Check all containers are running
sudo docker ps --filter "name=kasm" --format "table {{.Names}}\t{{.Status}}"

# Expected output: 8 containers with "Up" status
# - kasm_proxy
# - kasm_api
# - kasm_manager
# - kasm_db
# - kasm_redis
# - kasm_agent
# - kasm_guac
# - kasm_share
```

#### Step 5: Initial Access

1. **Get your IP address:**
   ```bash
   # Local IP
   ip addr show | grep "inet " | grep -v 127.0.0.1

   # Or if using Tailscale
   tailscale ip -4
   ```

2. **Access Kasm:**
   - Open browser: `https://YOUR_IP`
   - Accept SSL certificate warning (self-signed)
   - You'll see the Kasm login page

3. **Default Credentials:**
   - Check installation output for generated passwords
   - Or use credentials you set during installation

---

## Post-Installation Configuration

### 1. Seed Default Workspace Images

The installation creates workspace definitions but doesn't download images. Let's add the default browsers and applications:

```bash
# Seed workspace image definitions
sudo /opt/kasm/current/bin/utils/db_init \
  -s /opt/kasm/current/conf/database/seed_data/default_images_amd64.yaml \
  -q localhost

# Restart services
sudo /opt/kasm/current/bin/stop
sudo /opt/kasm/current/bin/start
```

This adds 59 workspace definitions including Chrome, Firefox, Edge, and more.

### 2. Download Required Browser Images

```bash
# Pull Chrome (3.08 GB)
sudo docker pull kasmweb/chrome:1.15.0

# Pull Firefox (2.85 GB)
sudo docker pull kasmweb/firefox:1.15.0

# Pull Edge (3.32 GB)
sudo docker pull kasmweb/edge:1.15.0

# Pull Brave (2.92 GB)
sudo docker pull kasmweb/brave:1.15.0

# Verify images are downloaded
sudo docker images | grep kasmweb
```

### 3. Configure Auto-Start

Auto-start is already configured! Verify with:

```bash
# Check Docker service
systemctl is-enabled docker
# Output: enabled

# Check container restart policies
sudo docker inspect kasm_proxy kasm_api --format='{{.Name}}: {{.HostConfig.RestartPolicy.Name}}'
# Output: always for all containers
```

**What this means:**
- When your server reboots, Docker starts automatically
- Docker then starts all Kasm containers automatically
- Kasm will be accessible ~60 seconds after boot

---

## Configuration

### User Accounts

#### Default Accounts

After installation, you have two accounts:

| Username | Default Password | Role | Purpose |
|----------|-----------------|------|---------|
| admin@kasm.local | (generated) | Administrator | System configuration, user management |
| user@kasm.local | (generated) | User | Regular workspace usage |

#### Reset Password

If you need to reset a password:

```bash
# Method 1: Via Web Interface (preferred)
# Login as admin → Admin Panel → Users → Select user → Change Password

# Method 2: Via Database (if locked out)
# Get the user's salt
SALT=$(sudo docker exec kasm_db psql -U kasmapp -d kasm -t -c "SELECT salt FROM users WHERE username = 'admin@kasm.local';" | xargs)

# Generate new password hash (password: "newpassword")
HASH=$(printf "newpassword${SALT}" | sha256sum | cut -c-64)

# Update password and unlock account
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "UPDATE users SET pw_hash = '${HASH}', locked = false, failed_pw_attempts = 0 WHERE username = 'admin@kasm.local';"
```

**Password Hashing Details:**
- Method: SHA256(password + salt)
- Salt: UUID stored in `users.salt` column
- Hash: 64-character hex stored in `users.pw_hash` column
- NOT bcrypt (Kasm uses custom implementation)

#### Create New User

```bash
# Via Web Interface (recommended):
# Admin Panel → Users → Add User

# Via Database (advanced):
NEW_USER="newuser@kasm.local"
NEW_PASS="SecurePassword123"
NEW_SALT=$(cat /proc/sys/kernel/random/uuid)
NEW_HASH=$(printf "${NEW_PASS}${NEW_SALT}" | sha256sum | cut -c-64)

sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
INSERT INTO users (user_id, username, pw_hash, salt, disabled, locked, created)
VALUES (
  gen_random_uuid(),
  '${NEW_USER}',
  '${NEW_HASH}',
  '${NEW_SALT}',
  false,
  false,
  NOW()
);"
```

### Persistent Profiles

Enable persistent profiles to save browser data across sessions (logins, bookmarks, history, etc.).

#### Enable for Specific Workspace

```bash
# Enable for Chrome
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
UPDATE images
SET persistent_profile_path = '/home/kasm-user'
WHERE friendly_name = 'Chrome';"

# Enable for Edge
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
UPDATE images
SET persistent_profile_path = '/home/kasm-user'
WHERE friendly_name = 'Edge';"

# Enable for Firefox
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
UPDATE images
SET persistent_profile_path = '/home/kasm-user'
WHERE friendly_name = 'Firefox';"
```

#### Enable for All Users

```bash
# Check current setting
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
SELECT name, value
FROM group_settings
WHERE name = 'allow_persistent_profile';"

# Enable if not already enabled
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
UPDATE group_settings
SET value = 'True'
WHERE name = 'allow_persistent_profile';"
```

#### Verify Configuration

```bash
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
SELECT friendly_name, persistent_profile_path
FROM images
WHERE persistent_profile_path IS NOT NULL
  AND persistent_profile_path != '';"
```

### Session Timeout

Default session timeout is 1 hour. Increase for better user experience:

```bash
# Set to 8 hours (28800 seconds)
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
UPDATE group_settings
SET value = '28800'
WHERE name = 'keepalive_expiration';"

# Set to 24 hours (86400 seconds)
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
UPDATE group_settings
SET value = '86400'
WHERE name = 'keepalive_expiration';"

# Verify
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
SELECT name, value
FROM group_settings
WHERE name = 'keepalive_expiration';"
```

**Session Settings Explained:**

| Setting | Default | Recommended | Description |
|---------|---------|-------------|-------------|
| idle_disconnect | 20 min | 20-60 min | Disconnect after inactivity |
| keepalive_expiration | 3600s (1h) | 28800s (8h) | Total session lifetime |
| keepalive_expiration_action | delete | delete | Action when session expires |

### SSL Certificate

#### Option 1: Use Self-Signed (Default)

Kasm installs with a self-signed certificate. No action needed, but browsers will show warnings.

#### Option 2: Let's Encrypt (Recommended for Production)

```bash
# Install certbot
sudo apt install -y certbot

# Get certificate (replace with your domain)
sudo certbot certonly --standalone -d kasm.yourdomain.com

# Copy certificates to Kasm
sudo cp /etc/letsencrypt/live/kasm.yourdomain.com/fullchain.pem \
  /opt/kasm/current/certs/kasm_nginx.crt

sudo cp /etc/letsencrypt/live/kasm.yourdomain.com/privkey.pem \
  /opt/kasm/current/certs/kasm_nginx.key

# Set proper permissions
sudo chown kasm:kasm /opt/kasm/current/certs/kasm_nginx.*

# Restart proxy
sudo docker restart kasm_proxy
```

#### Option 3: Custom Certificate

```bash
# Copy your certificate and key
sudo cp /path/to/your/certificate.crt /opt/kasm/current/certs/kasm_nginx.crt
sudo cp /path/to/your/private.key /opt/kasm/current/certs/kasm_nginx.key

# Set ownership
sudo chown kasm:kasm /opt/kasm/current/certs/kasm_nginx.*

# Restart
sudo docker restart kasm_proxy
```

---

## Usage

### Accessing Kasm

1. **Open Web Browser**
   - Navigate to: `https://YOUR_SERVER_IP`
   - Or: `https://your-domain.com` (if configured)

2. **Accept SSL Certificate**
   - If using self-signed certificate, click "Advanced" → "Proceed to site"

3. **Login**
   - Username: `user@kasm.local`
   - Password: (your password)
   - Click "Sign In"

### Launching Browsers

1. **Select Workspace**
   - After login, you'll see the Workspaces page
   - Browse available applications
   - Click on desired browser (Chrome, Edge, Firefox, etc.)

2. **Configure Launch Options**
   - **✅ IMPORTANT**: Check "Persistent Profile" to save your data
   - Optional: Set custom resolution
   - Optional: Enable audio (experimental)

3. **Click "Launch"**
   - Container starts (5-10 seconds first time)
   - New tab opens with browser session
   - Use browser normally!

### Using Persistent Profiles

**To Keep Your Gmail Login Permanently:**

1. **First Launch:**
   ```
   Workspaces → Chrome → ✅ Check "Persistent Profile" → Launch
   ```

2. **Login to Gmail:**
   - Browser opens
   - Navigate to gmail.com
   - Login as normal
   - Your session is automatically saved!

3. **Next Time:**
   - Launch Chrome with "Persistent Profile" checked
   - You're automatically logged into Gmail!
   - All bookmarks, history, extensions preserved

**What Gets Saved:**
- ✅ Website logins (cookies/sessions)
- ✅ Bookmarks
- ✅ Browser history
- ✅ Extensions
- ✅ Settings & preferences
- ✅ Saved passwords
- ✅ Form autofill data

**Important Notes:**
- ⚠️ Must check "Persistent Profile" every launch
- ⚠️ Profile data is per-user (different users have separate profiles)
- ⚠️ Stored on server, survives container destruction
- ⚠️ Takes slightly longer to start (loading profile data)

### File Transfer

#### Upload Files to Workspace

1. **Method 1: Drag & Drop**
   - Drag file from your computer
   - Drop onto browser window
   - File appears in container's Downloads folder

2. **Method 2: Upload Button**
   - Click "Upload" icon in Kasm toolbar
   - Select file(s)
   - Files transfer to container

#### Download Files from Workspace

1. **Method 1: Download in Browser**
   - Download file in virtual browser
   - Click "Download" icon in Kasm toolbar
   - File transfers to your computer

2. **Method 2: Shared Clipboard**
   - Copy text/URLs in virtual browser
   - Paste on your local machine (and vice versa)

---

## Available Workspaces

### Pre-Configured (59 Total)

#### Web Browsers (5)
- **Chrome** - Google Chrome browser
- **Edge** - Microsoft Edge browser
- **Firefox** - Mozilla Firefox browser
- **Brave** - Privacy-focused Brave browser
- **Chromium** - Open-source Chromium

#### Development (15+)
- VS Code, Sublime Text, Atom
- GitKraken, Git GUI
- Insomnia (API testing)
- Postman
- Android Studio
- Eclipse IDE
- And more...

#### Productivity (10+)
- LibreOffice Suite
- OnlyOffice
- Thunderbird (Email)
- Remmina (Remote Desktop)
- FileZilla (FTP)
- And more...

#### Media & Design (8+)
- GIMP (Image Editor)
- Inkscape (Vector Graphics)
- Krita (Digital Painting)
- Audacity (Audio Editor)
- VLC Media Player
- And more...

#### Communication (5+)
- Discord
- Telegram
- Signal
- Zoom
- And more...

#### Operating Systems (10+)
- Ubuntu Desktop 22.04/20.04/18.04
- Debian 11/10
- CentOS 7/8
- Kali Linux
- Parrot Security
- And more...

### Custom Workspaces

Create your own workspace images! See [Advanced Configuration](#advanced-configuration).

---

## Security

### Immediate Actions (Required)

1. **Change Default Passwords**
   ```
   Admin Panel → Users → Select User → Change Password
   ```
   - Use strong passwords (16+ characters)
   - Mix uppercase, lowercase, numbers, symbols
   - Don't reuse passwords

2. **Enable Two-Factor Authentication**
   ```
   User Profile → Two-Factor Authentication → Enable
   ```
   - Use authenticator app (Google Authenticator, Authy)
   - Save recovery codes securely

3. **Review User Permissions**
   ```
   Admin Panel → Users → Review Roles
   ```
   - Use least privilege principle
   - Don't give admin access unnecessarily

### Firewall Configuration

#### Basic UFW Setup

```bash
# Install UFW
sudo apt install -y ufw

# Allow SSH (important - don't lock yourself out!)
sudo ufw allow 22/tcp

# Allow Kasm HTTPS
sudo ufw allow 443/tcp

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status verbose
```

#### Restrict to Tailscale Network

```bash
# Only allow Kasm from Tailscale IPs
sudo ufw delete allow 443/tcp
sudo ufw allow from 100.64.0.0/10 to any port 443

# Reload
sudo ufw reload
```

#### Fail2ban for Brute Force Protection

```bash
# Install fail2ban
sudo apt install -y fail2ban

# Create Kasm jail
sudo tee /etc/fail2ban/jail.d/kasm.conf <<EOF
[kasm-auth]
enabled = true
filter = kasm-auth
logpath = /opt/kasm/current/log/api_server.log
maxretry = 5
bantime = 3600
findtime = 600
EOF

# Create filter
sudo tee /etc/fail2ban/filter.d/kasm-auth.conf <<EOF
[Definition]
failregex = Authentication attempt invalid password for user: \(.*\)
ignoreregex =
EOF

# Restart fail2ban
sudo systemctl restart fail2ban
```

### Network Security

1. **Use HTTPS Only** - Already configured ✅
2. **Disable HTTP** - Not exposed ✅
3. **Use Strong Ciphers** - Default config ✅
4. **Enable HSTS** - Add to nginx config:
   ```bash
   # Edit nginx config
   sudo docker exec -it kasm_proxy vi /etc/nginx/conf.d/orchestrator.conf

   # Add under server block:
   add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

   # Restart
   sudo docker restart kasm_proxy
   ```

### Container Security

Kasm containers run with security best practices:
- ✅ Non-root user inside containers
- ✅ Limited capabilities
- ✅ Read-only root filesystem where possible
- ✅ Network isolation
- ✅ Resource limits (CPU/memory)

### Data Security

1. **Database Encryption**
   - Sensitive data is encrypted at rest
   - Connection uses SSL

2. **Backup Encryption**
   ```bash
   # Backup with encryption
   sudo /opt/kasm/current/bin/utils/db_backup -f /backup/kasm_backup.tar

   # Encrypt backup
   gpg --symmetric --cipher-algo AES256 /backup/kasm_backup.tar
   ```

3. **Log Security**
   ```bash
   # Restrict log permissions
   sudo chmod 640 /opt/kasm/current/log/*.log
   sudo chown kasm:kasm /opt/kasm/current/log/*.log
   ```

### Security Auditing

```bash
# Check for outdated images
sudo docker images | grep kasmweb

# Update to latest (see Upgrading section)

# Review user activity
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
SELECT username, created, last_session
FROM users
ORDER BY created DESC;"

# Review active sessions
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
SELECT u.username, k.created, k.port
FROM kasms k
JOIN users u ON k.user_id = u.user_id
WHERE k.current_state = 'running';"
```

---

## Management

### Service Control

```bash
# Start all Kasm services
sudo /opt/kasm/current/bin/start

# Stop all Kasm services
sudo /opt/kasm/current/bin/stop

# Restart all services
sudo /opt/kasm/current/bin/stop
sudo /opt/kasm/current/bin/start

# Check service status
sudo docker ps --filter "name=kasm"
```

### Database Management

#### Connect to Database

```bash
# Interactive psql
sudo docker exec -it kasm_db psql -U kasmapp -d kasm

# Run single query
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "SELECT COUNT(*) FROM users;"
```

#### Common Database Queries

```sql
-- List all users
SELECT username, disabled, locked, created FROM users;

-- List all workspace images
SELECT friendly_name, enabled FROM images ORDER BY friendly_name;

-- List active sessions
SELECT u.username, k.image_id, k.created
FROM kasms k
JOIN users u ON k.user_id = u.user_id
WHERE k.current_state = 'running';

-- Count sessions by user
SELECT u.username, COUNT(*) as session_count
FROM kasms k
JOIN users u ON k.user_id = u.user_id
GROUP BY u.username;

-- View group settings
SELECT name, value FROM group_settings ORDER BY name;
```

#### Backup Database

```bash
# Full backup
sudo /opt/kasm/current/bin/utils/db_backup -f /backup/kasm_$(date +%Y%m%d).tar

# Backup to specific location
sudo /opt/kasm/current/bin/utils/db_backup -f /mnt/backups/kasm_backup.tar

# Verify backup
tar -tzf /backup/kasm_*.tar | head -20
```

#### Restore Database

```bash
# Stop Kasm
sudo /opt/kasm/current/bin/stop

# Restore from backup
sudo /opt/kasm/current/bin/utils/db_restore -f /backup/kasm_backup.tar

# Start Kasm
sudo /opt/kasm/current/bin/start
```

### Docker Commands

```bash
# List all Kasm containers
sudo docker ps --filter "name=kasm" --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# View container logs
sudo docker logs kasm_api --tail 100
sudo docker logs kasm_proxy --tail 100
sudo docker logs kasm_db --tail 100

# Follow logs in real-time
sudo docker logs -f kasm_api

# Restart specific container
sudo docker restart kasm_api
sudo docker restart kasm_proxy

# Check container resource usage
sudo docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Inspect container configuration
sudo docker inspect kasm_api | jq '.[0].HostConfig'

# Execute command in container
sudo docker exec kasm_api ls -la /opt/kasm
sudo docker exec kasm_db psql --version
```

### Log Management

```bash
# View API logs
sudo tail -f /opt/kasm/current/log/api_server.log

# View client API logs
sudo tail -f /opt/kasm/current/log/client_api_server.log

# View manager logs
sudo tail -f /opt/kasm/current/log/manager_api_server.log

# View agent logs
sudo tail -f /opt/kasm/current/log/agent.log

# Search for errors
sudo grep -i error /opt/kasm/current/log/*.log

# Search for specific user activity
sudo grep "user@kasm.local" /opt/kasm/current/log/*.log

# Rotate logs manually
sudo docker exec kasm_api logrotate -f /etc/logrotate.conf

# Clean old logs (older than 30 days)
sudo find /opt/kasm/current/log -name "*.log.*" -mtime +30 -delete
```

---

## Troubleshooting

### Common Issues

#### 1. Cannot Access Web Interface

**Symptom:** Browser shows "Connection refused" or timeout

**Solutions:**

```bash
# Check if containers are running
sudo docker ps --filter "name=kasm"

# If containers are stopped, start them
sudo /opt/kasm/current/bin/start

# Check nginx proxy is running
sudo docker ps --filter "name=kasm_proxy"

# Check proxy logs for errors
sudo docker logs kasm_proxy --tail 50

# Verify port 443 is listening
sudo netstat -tlnp | grep :443

# Check firewall
sudo ufw status
```

#### 2. Login Failed / Invalid Password

**Symptom:** "Authentication failed" or "Invalid password"

**Solutions:**

```bash
# Check if account is locked
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
SELECT username, locked, failed_pw_attempts
FROM users
WHERE username = 'admin@kasm.local';"

# Unlock account
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
UPDATE users
SET locked = false, failed_pw_attempts = 0
WHERE username = 'admin@kasm.local';"

# Reset password (see Configuration section)
```

#### 3. Workspace Won't Launch

**Symptom:** Workspace launch fails or times out

**Solutions:**

```bash
# Check if image is downloaded
sudo docker images | grep kasmweb

# Pull image if missing
sudo docker pull kasmweb/chrome:1.15.0

# Check disk space
df -h

# Check agent is running
sudo docker ps --filter "name=kasm_agent"

# Check agent logs
sudo docker logs kasm_agent --tail 100

# Restart agent
sudo docker restart kasm_agent
```

#### 4. Persistent Profile Not Working

**Symptom:** Profile data not saved between sessions

**Solutions:**

```bash
# Verify persistent profiles are enabled
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
SELECT name, value
FROM group_settings
WHERE name = 'allow_persistent_profile';"

# Should return: True

# Check image has persistent path set
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
SELECT friendly_name, persistent_profile_path
FROM images
WHERE friendly_name = 'Chrome';"

# Should return: /home/kasm-user

# Make sure you're checking the box when launching!
```

#### 5. Session Expires Too Quickly

**Symptom:** Session ends after 1 hour

**Solution:**

```bash
# Increase timeout (see Configuration section)
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
UPDATE group_settings
SET value = '28800'
WHERE name = 'keepalive_expiration';"
```

#### 6. Language is Not English

**Symptom:** Interface shows in French or other language

**Solution:**

Kasm detects language from browser settings. Change browser language:
- Chrome/Edge: Settings → Languages → Move English to top
- Firefox: Settings → Language → Choose English

Then refresh the Kasm page.

#### 7. SSL Certificate Errors

**Symptom:** Browser shows "Your connection is not private"

**Solutions:**

This is normal with self-signed certificates:
1. Click "Advanced"
2. Click "Proceed to [your-ip] (unsafe)"

Or install proper certificate (see Configuration → SSL Certificate).

#### 8. Out of Disk Space

**Symptom:** Workspaces won't launch, database errors

**Solutions:**

```bash
# Check disk usage
df -h

# Clean up Docker
sudo docker system prune -a --volumes

# Remove old images
sudo docker images | grep kasmweb | grep -v 1.15.0 | awk '{print $3}' | xargs sudo docker rmi

# Check large files
sudo du -sh /opt/kasm/* | sort -h
```

### Advanced Troubleshooting

#### Enable Debug Logging

```bash
# Edit API config
sudo docker exec kasm_api vi /opt/kasm/current/conf/app/api.app.config.yaml

# Change log level to DEBUG
# Restart API
sudo docker restart kasm_api

# Watch debug logs
sudo tail -f /opt/kasm/current/log/api_server.log
```

#### Network Diagnostics

```bash
# Test internal networking
sudo docker exec kasm_api ping -c 3 kasm_db
sudo docker exec kasm_api ping -c 3 kasm_redis

# Check DNS resolution
sudo docker exec kasm_api nslookup google.com

# Test database connection
sudo docker exec kasm_api nc -zv kasm_db 5432
```

#### Container Health Checks

```bash
# Check container health status
sudo docker ps --format "table {{.Names}}\t{{.Status}}"

# Inspect specific container health
sudo docker inspect --format='{{json .State.Health}}' kasm_db | jq

# Run manual health check
sudo docker exec kasm_db pg_isready -U kasmapp
```

---

## Advanced Configuration

### Custom Workspace Images

Create your own workspace with specific applications:

```dockerfile
# Dockerfile
FROM kasmweb/core-ubuntu-focal:1.15.0

USER root

# Install your applications
RUN apt-get update && apt-get install -y \
    neovim \
    tmux \
    htop \
    && rm -rf /var/lib/apt/lists/*

# Custom configuration
COPY custom-config /home/kasm-user/.config/

USER 1000

# Expose VNC port
EXPOSE 6901
```

Build and add to Kasm:

```bash
# Build image
sudo docker build -t custom/my-workspace:1.0.0 .

# Add to Kasm via Admin Panel
# Admin → Workspaces → Add Workspace → Docker Registry
```

### LDAP/AD Integration

Configure authentication against Active Directory:

```bash
# Via Admin Panel:
# Admin → Authentication → LDAP Settings

# Or via database:
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
INSERT INTO auth_configs (...)
VALUES (...);"
```

### Multi-Server Deployment

Scale Kasm across multiple servers:

1. **Install on multiple servers**
2. **Configure shared database** (all point to same DB)
3. **Configure load balancer** (distribute traffic)
4. **Set server zones** for geo-distribution

See official docs: https://kasmweb.com/docs/latest/how_to/building_images.html

### Resource Limits

Limit CPU/memory per workspace:

```bash
# Via Admin Panel:
# Admin → Workspaces → Select Workspace → Edit
# Set: CPU Allocation, Memory Allocation

# Or via database:
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "
UPDATE images
SET cpu_allocation = 2,
    memory = 2048
WHERE friendly_name = 'Chrome';"
```

### Custom Branding

Replace Kasm logo and theme:

```bash
# Upload custom logo
sudo cp /path/to/logo.png /opt/kasm/current/www/img/

# Edit via Admin Panel:
# Admin → Branding → Upload Logo

# Custom CSS
sudo vi /opt/kasm/current/www/custom.css

# Add to page
sudo docker exec kasm_proxy vi /etc/nginx/conf.d/services.d/website.conf
```

---

## Performance Tuning

### Database Optimization

```bash
# Edit PostgreSQL config
sudo docker exec -it kasm_db vi /var/lib/postgresql/data/postgresql.conf

# Recommended changes for 15GB RAM system:
shared_buffers = 4GB
effective_cache_size = 12GB
work_mem = 64MB
maintenance_work_mem = 1GB
max_connections = 200

# Restart database
sudo docker restart kasm_db
```

### Redis Optimization

```bash
# Edit Redis config
sudo docker exec -it kasm_redis vi /etc/redis/redis.conf

# Set max memory
maxmemory 2gb
maxmemory-policy allkeys-lru

# Restart Redis
sudo docker restart kasm_redis
```

### Network Performance

```bash
# Enable BBR congestion control
echo "net.core.default_qdisc=fq" | sudo tee -a /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Increase connection backlog
echo "net.core.somaxconn=4096" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### Docker Optimization

```bash
# Edit Docker daemon config
sudo vi /etc/docker/daemon.json

# Add:
{
  "storage-driver": "overlay2",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 64000,
      "Soft": 64000
    }
  }
}

# Restart Docker
sudo systemctl restart docker

# Restart Kasm
sudo /opt/kasm/current/bin/start
```

---

## Backup and Restore

### Automated Backups

```bash
# Create backup script
sudo tee /usr/local/bin/kasm-backup.sh <<'EOF'
#!/bin/bash
BACKUP_DIR="/backup/kasm"
DATE=$(date +%Y%m%d_%H%M%S)
mkdir -p "$BACKUP_DIR"

# Backup database
/opt/kasm/current/bin/utils/db_backup -f "$BACKUP_DIR/kasm_db_$DATE.tar"

# Compress and encrypt
tar czf "$BACKUP_DIR/kasm_full_$DATE.tar.gz" \
  /opt/kasm/current/conf \
  /opt/kasm/current/certs \
  "$BACKUP_DIR/kasm_db_$DATE.tar"

# Encrypt
gpg --symmetric --cipher-algo AES256 "$BACKUP_DIR/kasm_full_$DATE.tar.gz"

# Cleanup
rm "$BACKUP_DIR/kasm_db_$DATE.tar"
rm "$BACKUP_DIR/kasm_full_$DATE.tar.gz"

# Keep only last 7 days
find "$BACKUP_DIR" -name "*.gpg" -mtime +7 -delete

echo "Backup completed: kasm_full_$DATE.tar.gz.gpg"
EOF

# Make executable
sudo chmod +x /usr/local/bin/kasm-backup.sh

# Add to cron (daily at 2 AM)
echo "0 2 * * * /usr/local/bin/kasm-backup.sh" | sudo crontab -
```

### Manual Backup

```bash
# Full backup
sudo /opt/kasm/current/bin/utils/db_backup -f /backup/kasm_$(date +%Y%m%d).tar

# Backup configuration
sudo tar czf /backup/kasm_config_$(date +%Y%m%d).tar.gz \
  /opt/kasm/current/conf \
  /opt/kasm/current/certs

# Backup persistent profiles (if used)
sudo tar czf /backup/kasm_profiles_$(date +%Y%m%d).tar.gz \
  /mnt/kasm_profiles  # Adjust path as needed
```

### Restore from Backup

```bash
# Stop Kasm
sudo /opt/kasm/current/bin/stop

# Restore database
sudo /opt/kasm/current/bin/utils/db_restore -f /backup/kasm_20251221.tar

# Restore configuration
sudo tar xzf /backup/kasm_config_20251221.tar.gz -C /

# Restore profiles (if needed)
sudo tar xzf /backup/kasm_profiles_20251221.tar.gz -C /

# Start Kasm
sudo /opt/kasm/current/bin/start
```

### Off-Site Backup

```bash
# Install rclone
sudo apt install -y rclone

# Configure remote (Google Drive, S3, etc.)
rclone config

# Sync backups to cloud
rclone sync /backup/kasm remote:kasm-backups

# Add to backup script
echo "rclone sync /backup/kasm remote:kasm-backups" | sudo tee -a /usr/local/bin/kasm-backup.sh
```

---

## Upgrading

### Before Upgrading

1. **Backup everything**
   ```bash
   sudo /opt/kasm/current/bin/utils/db_backup -f /backup/pre_upgrade_$(date +%Y%m%d).tar
   ```

2. **Check release notes**
   - Visit: https://kasmweb.com/docs/latest/release_notes.html
   - Review breaking changes

3. **Test in staging** (if possible)

### Upgrade Process

```bash
# Download new version
cd /tmp
curl -O https://kasm-static-content.s3.amazonaws.com/kasm_release_X.Y.Z.tar.gz

# Extract
tar -xf kasm_release_X.Y.Z.tar.gz
cd kasm_release

# Run upgrade script
sudo bash upgrade.sh

# Verify
sudo docker ps --filter "name=kasm"
```

### Rollback

If upgrade fails:

```bash
# Stop new version
sudo /opt/kasm/X.Y.Z/bin/stop

# Restore database backup
sudo /opt/kasm/current/bin/utils/db_restore -f /backup/pre_upgrade_YYYYMMDD.tar

# Start old version
sudo /opt/kasm/OLD_VERSION/bin/start
```

---

## FAQ

### General Questions

**Q: Is Kasm Workspaces free?**
A: Yes, the Community Edition is free and open-source. There's also a paid Enterprise Edition with additional features.

**Q: Can I use this for production?**
A: Yes, with proper security hardening and regular backups.

**Q: How many concurrent users can it handle?**
A: Depends on resources. Our test system (16 cores, 15GB RAM) can handle 10-20 concurrent browser sessions comfortably.

**Q: Do I need a domain name?**
A: No, you can access via IP address. Domain is recommended for Let's Encrypt SSL.

### Technical Questions

**Q: Can I run Kasm on Windows/macOS?**
A: Officially, only Linux is supported. However, you can run in WSL2 (Windows) or VM.

**Q: Can I use my own Docker registry?**
A: Yes, configure in Admin Panel → Workspaces → Docker Registry.

**Q: Does it support GPU passthrough?**
A: Yes, in Enterprise Edition. Community Edition does not support GPU.

**Q: Can I customize the desktop environment?**
A: Yes, you can build custom images with any desktop environment (GNOME, KDE, XFCE, etc.).

**Q: Is audio supported?**
A: Yes, but experimental. May have latency issues.

**Q: Can I use with VPN?**
A: Yes, works great with Tailscale, WireGuard, OpenVPN.

### Troubleshooting Questions

**Q: Why is my session so slow?**
A: Check network bandwidth, server resources (CPU/RAM), and consider increasing workspace resource limits.

**Q: Can I access from mobile?**
A: Yes! Works on iOS and Android browsers.

**Q: How do I delete old sessions?**
A: They auto-delete after timeout. Manual: Admin Panel → Sessions → Delete.

**Q: Can I have multiple displays?**
A: Enterprise Edition only.

---

## Contributing

We welcome contributions! Here's how:

### Reporting Issues

1. Check existing issues: https://github.com/Lalatenduswain/kasm-workspaces-setup/issues
2. Create new issue with:
   - Clear description
   - Steps to reproduce
   - Expected vs actual behavior
   - System details (OS, Kasm version, etc.)
   - Relevant logs

### Contributing Documentation

1. Fork the repository
2. Make changes to README.md
3. Test your changes
4. Submit pull request

### Contributing Code/Scripts

1. Fork the repository
2. Create feature branch: `git checkout -b feature/amazing-feature`
3. Commit changes: `git commit -m 'Add amazing feature'`
4. Push: `git push origin feature/amazing-feature`
5. Submit pull request

---

## License

This documentation is licensed under MIT License.

Kasm Workspaces is licensed under its own EULA. See: https://kasmweb.com/community-edition-license

---

## Support

### Official Kasm Support

- **Documentation**: https://kasmweb.com/docs/latest/
- **Community Forum**: https://forum.kasmweb.com/
- **Discord**: https://discord.gg/kasmweb
- **GitHub Issues**: https://github.com/kasmtech/workspaces-issues

### This Repository

- **Issues**: https://github.com/Lalatenduswain/kasm-workspaces-setup/issues
- **Discussions**: https://github.com/Lalatenduswain/kasm-workspaces-setup/discussions
- **Pull Requests**: https://github.com/Lalatenduswain/kasm-workspaces-setup/pulls

### Quick Help

**Installation help:** Check [Installation](#installation) section
**Login issues:** See [Troubleshooting → Login Failed](#2-login-failed--invalid-password)
**Workspace won't start:** See [Troubleshooting → Workspace Won't Launch](#3-workspace-wont-launch)
**Password reset:** See [Configuration → User Accounts](#reset-password)
**Persistent profiles:** See [Configuration → Persistent Profiles](#persistent-profiles)

---

## Acknowledgments

- **Kasm Technologies** - For creating this amazing platform
- **Community Contributors** - For sharing knowledge and improvements
- **You** - For using this guide!

---

## Changelog

### December 21, 2025
- ✅ Initial documentation
- ✅ Complete installation guide
- ✅ Persistent profiles configuration
- ✅ Security hardening guide
- ✅ Troubleshooting section
- ✅ Advanced configuration examples

---

<div align="center">

**Star this repository if it helped you! ⭐**

Made with ❤️ for the Kasm Community

[Back to Top](#kasm-workspaces-setup---complete-installation-guide)

</div>
