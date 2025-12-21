# Project: Kasm Workspaces Browser Environment Setup
Last Updated: December 21, 2025 - 13:00 IST

## Current Sprint
- Goal: Set up containerized browser environment for secure, isolated web browsing
- Progress: 100% ✅ Complete

## This Session (December 21, 2025)
✅ Completed:
- Installed Kasm Workspaces 1.15.0 on Ubuntu 24.04.3 LTS VM
- Configured all services (Database, API, Proxy, Manager, Agent, Guacamole, Redis)
- Set up HTTPS access via Tailscale (https://100.94.23.26)
- Created and configured user accounts (admin@kasm.local, user@kasm.local)
- Reset passwords using SHA256 hashing with salt (password: "password")
- Unlocked locked admin account after failed login attempts
- Installed default workspace images (59 workspaces total)
- Downloaded and verified Chrome workspace (3.08GB)
- Downloaded and verified Edge workspace (3.32GB)
- Configured auto-start on system boot (Docker + container restart policies)
- **Enabled persistent profiles** for Chrome and Edge browsers
- Increased session timeout from 1 hour to 8 hours
- Created comprehensive documentation (KASM_WORKSPACES_SETUP.md)

🚧 In Progress:
- None - project complete

⏭️ Next Steps:
1. User should login and test persistent profile feature with Chrome/Edge
2. Login to Gmail with "Persistent Profile" checkbox enabled
3. Test session persistence by closing and relaunching browser
4. Consider changing default passwords for security
5. Optional: Configure firewall rules to restrict access to Tailscale network only
6. Optional: Replace self-signed SSL certificate with Let's Encrypt

⚠️ Blockers/Notes:
- None
- Language interface defaults to French (browser language detection) - user can change in browser settings
- Persistent profiles must be manually enabled each launch by checking the checkbox
- Session timeout: 8 hours (28800 seconds)
- Idle disconnect: 20 minutes

## System Details
**Environment:**
- Host: Ubuntu 24.04.3 LTS (Noble Numbat)
- CPU: 16 cores
- RAM: 15GB
- Disk: 196GB total, 96GB available
- Network: Tailscale IP 100.94.23.26

**Kasm Installation:**
- Version: 1.15.0
- Installation Path: /opt/kasm/1.15.0
- Access URL: https://100.94.23.26
- Admin User: admin@kasm.local / password
- Regular User: user@kasm.local / password

**Docker Containers:**
- kasm_db (PostgreSQL 12)
- kasm_api (Kasm API Server)
- kasm_proxy (Nginx)
- kasm_manager (Manager Service)
- kasm_agent (Agent Service)
- kasm_guac (Guacamole)
- kasm_share (File Sharing)
- kasm_redis (Redis Cache)

**Workspaces Installed:**
- Chrome (3.08GB) ✅ Downloaded
- Edge (3.32GB) ✅ Downloaded
- Firefox, Brave, Chromium, Discord, GIMP, FileZilla, and 50+ others (configured, not downloaded)

## Tech Stack
- Kasm Workspaces 1.15.0
- Docker & Docker Compose
- PostgreSQL 12
- Nginx (reverse proxy with SSL)
- Redis
- Tailscale (networking)
- Ubuntu 24.04 LTS

## Database Configuration Notes
**Password Hashing:**
- Method: SHA256(password + salt)
- Salt stored in `users.salt` column
- Hash stored in `users.pw_hash` column
- NOT bcrypt (initial attempt was incorrect)

**Persistent Profiles:**
- Enabled in `group_settings.allow_persistent_profile = True`
- Chrome: `images.persistent_profile_path = '/home/kasm-user'`
- Edge: `images.persistent_profile_path = '/home/kasm-user'`
- Data persists across session destruction
- User must check "Persistent Profile" checkbox when launching

**Session Settings:**
- `keepalive_expiration = 28800` (8 hours)
- `idle_disconnect = 20` (20 minutes)
- `keepalive_expiration_action = delete`

## Files Created
- `/home/ehs/browser/KASM_WORKSPACES_SETUP.md` - Complete setup documentation
- `/home/ehs/browser/CLAUDE.md` - This session tracking file

## Useful Commands
**Kasm Management:**
```bash
# Start services
sudo /opt/kasm/1.15.0/bin/start

# Stop services
sudo /opt/kasm/1.15.0/bin/stop

# Check status
sudo docker ps --filter "name=kasm"
```

**Database Access:**
```bash
# Connect to database
sudo docker exec -it kasm_db psql -U kasmapp -d kasm

# List users
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "SELECT username, locked, disabled FROM users;"

# List workspaces
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "SELECT friendly_name, enabled FROM images;"
```

**Password Reset (if needed):**
```bash
# Get salt
SALT=$(sudo docker exec kasm_db psql -U kasmapp -d kasm -t -c "SELECT salt FROM users WHERE username = 'admin@kasm.local';" | xargs)

# Generate hash
HASH=$(printf "newpassword${SALT}" | sha256sum | cut -c-64)

# Update password
sudo docker exec kasm_db psql -U kasmapp -d kasm -c "UPDATE users SET pw_hash = '${HASH}', locked = false, failed_pw_attempts = 0 WHERE username = 'admin@kasm.local';"
```

## Resume Instructions
If resuming this work in a new session:
1. Read `/home/ehs/browser/KASM_WORKSPACES_SETUP.md` for complete setup details
2. Kasm should be auto-running (check with `sudo docker ps | grep kasm`)
3. Access via https://100.94.23.26
4. Login credentials in this file above
5. Persistent profiles are configured - just need user to enable when launching

## Success Criteria ✅
- [x] Kasm Workspaces installed and running
- [x] Accessible via HTTPS on Tailscale network
- [x] User accounts created and working
- [x] Chrome and Edge browsers available
- [x] Persistent profiles configured
- [x] Auto-start on boot enabled
- [x] Documentation created
- [x] Session timeout extended to 8 hours
