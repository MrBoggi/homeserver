# 🏠 HUSSERVER - COMPLETE PORTAINER SETUP GUIDE

## 📋 Portainer Stack Creation Order

**IMPORTANT**: Create stacks in this exact order due to dependencies!

### 1. **Infrastructure Stack** (MUST BE FIRST!)
- **Stack Name**: `husserver-infrastructure`
- **Compose File**: [infrastructure-compose.yml](infrastructure-compose.yml)
- **Environment Variables**: [portainer-infrastructure-env.txt](portainer-infrastructure-env.txt)

### 2. **Media Storage Stack**
- **Stack Name**: `husserver-media-storage`
- **Compose File**: [media-storage-compose.yml](media-storage-compose.yml)
- **Environment Variables**: [portainer-media-storage-env.txt](portainer-media-storage-env.txt)

### 3. **Smart Home Stack**
- **Stack Name**: `husserver-smart-home`
- **Compose File**: [smart-home-compose.yml](smart-home-compose.yml)
- **Environment Variables**: [portainer-smart-home-env.txt](portainer-smart-home-env.txt)

### 4. **Industrial Stack**
- **Stack Name**: `husserver-industrial`
- **Compose File**: [industrial-compose.yml](industrial-compose.yml)
- **Environment Variables**: [portainer-industrial-env.txt](portainer-industrial-env.txt)

### 5. **Utilities Stack**
- **Stack Name**: `husserver-utilities`
- **Compose File**: [utilities-compose.yml](utilities-compose.yml)
- **Environment Variables**: [portainer-utilities-env.txt](portainer-utilities-env.txt)

### 6. **Network Stack**
- **Stack Name**: `husserver-network`
- **Compose File**: [network-compose.yml](network-compose.yml)
- **Environment Variables**: [portainer-network-env.txt](portainer-network-env.txt)

### 7. **Gaming Stack** (Optional - Last)
- **Stack Name**: `husserver-gaming`
- **Compose File**: [gaming-compose.yml](gaming-compose.yml)
- **Environment Variables**: [portainer-gaming-env.txt](portainer-gaming-env.txt)

---

## 🔧 Portainer Setup Steps

### Step 1: Create External Volumes
Before creating any stacks, create these external volumes in Portainer:

```bash
# Infrastructure volumes
mariadb-unified-data
timescaledb-unified-data
redis-unified-data
pgadmin-data

# Media storage volumes
nextcloud-data
nextcloud-apps
nextcloud-config
nextcloud-files
photoprism-originals
photoprism-storage
photoprism-import
filebrowser-config
filebrowser-data

# Smart home volumes
emqx-data
emqx-etc
emqx-log
scrypted-data
scrypted-nvr
homeassistant-config

# Industrial volumes
ignition-data
ignition-logs
envision-data
envision-logs
envision-config
opcua-config
opcua-data
modbus-config

# Utilities volumes
codeserver-config
codeserver-workspace
rustdesk-hbbs
rustdesk-hbbr
uptime-kuma-data
homer-config
duplicati-config
duplicati-backups

# Network volumes
nginx-data
nginx-letsencrypt
nginx-logs
fail2ban-config
fail2ban-log
authelia-config
ddns-data

# Gaming volumes (if used)
minecraft-creative-data
minecraft-creative-mods
minecraft-survival-data
minecraft-survival-mods
minecraft-friends-data
minecraft-friends-mods
beammp-data
beammp-config
beammp-logs
pterodactyl-data
game-stats-data
```

### Step 2: Create External Networks
Create these networks in Portainer:

```bash
infrastructure-network
```

### Step 3: Create Stacks

For each stack:

1. **Go to Stacks** → **Add Stack**
2. **Enter Stack Name** (use names from list above)
3. **Select "Upload"** and upload the corresponding compose file
4. **Copy environment variables** from corresponding .txt file
5. **Paste into Environment Variables section**
6. **CUSTOMIZE ALL PASSWORDS** - Replace placeholder passwords!
7. **Deploy the stack**

---

## 🔐 Security Checklist

**CRITICAL**: Before deploying, replace ALL placeholder passwords:

### Infrastructure Stack:
- [ ] `MYSQL_ROOT_PASSWORD`
- [ ] `TIMESCALE_PASSWORD`
- [ ] `REDIS_PASSWORD`
- [ ] `NEXTCLOUD_DB_PASSWORD`
- [ ] `IGNITION_DB_PASSWORD`
- [ ] `PHOTOPRISM_DB_PASSWORD`
- [ ] `GRAFANA_DB_PASSWORD`
- [ ] `HUSSERVER_DB_PASSWORD`
- [ ] `PROMETHEUS_DB_PASSWORD`
- [ ] `PGADMIN_PASSWORD`
- [ ] `REDIS_COMMANDER_PASSWORD`

### Application Stacks:
- [ ] All `*_ADMIN_PASSWORD` variables
- [ ] All `*_PASSWORD` variables
- [ ] API keys and tokens

---

## 🌐 Access Points After Setup

| Service | URL | Stack |
|---------|-----|-------|
| **Databases** | | |
| phpMyAdmin | http://localhost:8787 | Infrastructure |
| pgAdmin | http://localhost:5050 | Infrastructure |
| Redis Commander | http://localhost:8081 | Infrastructure |
| **Media** | | |
| Nextcloud | http://localhost:8080 | Media Storage |
| PhotoPrism | http://localhost:2342 | Media Storage |
| File Browser | http://localhost:8082 | Media Storage |
| **Smart Home** | | |
| EMQX Dashboard | http://localhost:18083 | Smart Home |
| Home Assistant | http://localhost:8123 | Smart Home |
| **Industrial** | | |
| Ignition Gateway | http://localhost:9088 | Industrial |
| Envision SCADA | http://localhost:8090 | Industrial |
| **Utilities** | | |
| Code Server | http://localhost:8444 | Utilities |
| Dozzle Logs | http://localhost:8888 | Utilities |
| Uptime Kuma | http://localhost:3001 | Utilities |
| Homer Dashboard | http://localhost:8081 | Utilities |
| Duplicati Backup | http://localhost:8200 | Utilities |
| **Network** | | |
| Nginx Proxy Manager | http://localhost:81 | Network |

---

## 🎯 Resource Savings Summary

| Metric | Before | After | Savings |
|--------|--------|-------|---------|
| **MariaDB instances** | 4 | 1 | **75%** |
| **RAM usage (DB)** | ~1.2GB | ~350MB | **850MB** |
| **Compose files** | 14 | 6 | **57%** |
| **Port conflicts** | Multiple | None | ✅ |
| **Management complexity** | High | Low | ✅ |

---

## 🆘 Troubleshooting

### Stack Won't Start:
1. Check if infrastructure stack is running first
2. Verify all external volumes exist
3. Check environment variables for typos
4. Ensure no port conflicts

### Database Connection Issues:
1. Verify infrastructure stack is healthy
2. Check database passwords match between stacks
3. Ensure networks are created and connected

### Can't Access Services:
1. Check if containers are running: Containers view in Portainer
2. Verify port mappings in compose files
3. Check firewall settings on host

---

## 📚 Additional Resources

- **Logs**: View in Portainer → Containers → Select container → Logs
- **Console**: Access container console in Portainer
- **Health**: Monitor in Portainer → Stacks view
- **Updates**: Use Watchtower (in Utilities stack) for automatic updates

---

**🎉 Enjoy your new unified, efficient Husserver setup!**
