# 🚀 Installation Guide - Husserver Infrastructure

Complete step-by-step installation guide for setting up the Husserver infrastructure from scratch.

## 🔧 Prerequisites

### **System Requirements**
- **Operating System**: Ubuntu 20.04+ or similar Linux distribution
- **RAM**: Minimum 4GB, recommended 8GB+
- **Storage**: 50GB+ available disk space
- **CPU**: Multi-core processor recommended
- **Network**: Static IP address (recommended for database connections)

### **Software Requirements**
- Docker Engine 24.0+
- Docker Compose v2.0+
- Git
- Text editor (nano, vim, or similar)

---

## 📦 Phase 1: System Preparation

### **Step 1: Update System**
```bash
# Update package lists and system
sudo apt update && sudo apt upgrade -y

# Install essential packages
sudo apt install -y curl wget git nano htop unzip
```

### **Step 2: Install Docker**
```bash
# Remove old Docker installations
sudo apt remove -y docker docker-engine docker.io containerd runc

# Install Docker's official repository
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add current user to docker group
sudo usermod -aG docker $USER

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Verify installation
docker --version
docker-compose --version
```

### **Step 3: Install Portainer**
```bash
# Create Portainer volume
docker volume create portainer_data

# Deploy Portainer
docker run -d \
  -p 9000:9000 \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ee:latest

# Verify Portainer is running
docker ps | grep portainer
```

### **Step 4: Initial Portainer Setup**
1. Open browser: `http://[SERVER_IP]:9000`
2. Create admin account (username: admin, strong password)
3. Choose "Docker" as environment
4. Complete initial setup wizard

---

## 🗂️ Phase 2: Repository Setup

### **Step 1: Clone Repository**
```bash
# Navigate to desired directory
cd /opt

# Clone the repository
sudo git clone [YOUR_REPOSITORY_URL] husserver
sudo chown -R $USER:$USER husserver
cd husserver
```

### **Step 2: Verify Repository Structure**
```bash
# Check directory structure
tree -L 2

# Expected output:
# husserver/
# ├── infrastructure/
# ├── media-storage/
# ├── smart-home/
# ├── industrial/
# ├── utilities/
# ├── network/
# ├── gaming/
# └── portainer-configs/
```

---

## 🏗️ Phase 3: Infrastructure Deployment

### **Step 1: Create External Networks**
```bash
# Create shared networks
docker network create infrastructure-network
docker network ls | grep infrastructure
```

### **Step 2: Create External Volumes**
```bash
# Infrastructure volumes
docker volume create mariadb-unified-data
docker volume create timescaledb-unified-data
docker volume create redis-unified-data
docker volume create pgadmin-data

# Media storage volumes
docker volume create photoprism-originals
docker volume create photoprism-storage
docker volume create photoprism-import
docker volume create jellyfin-config
docker volume create jellyfin-cache
docker volume create filebrowser-config
docker volume create filebrowser-data
docker volume create jottacloud-config

# Smart home volumes
docker volume create emqx-data
docker volume create emqx-etc
docker volume create emqx-log
docker volume create scrypted-data
docker volume create scrypted-nvr

# Industrial volumes
docker volume create ignition-data
docker volume create ignition-logs
docker volume create opcua-config
docker volume create opcua-data
docker volume create modbus-config

# Utilities volumes
docker volume create codeserver-config
docker volume create codeserver-workspace
docker volume create rustdesk-hbbs
docker volume create rustdesk-hbbr
docker volume create uptime-kuma-data
docker volume create homer-config
docker volume create duplicati-config
docker volume create duplicati-backups

# Network volumes
docker volume create nginx-data
docker volume create nginx-letsencrypt
docker volume create nginx-logs
docker volume create fail2ban-config
docker volume create fail2ban-log
docker volume create authelia-config
docker volume create ddns-data

# Gaming volumes (if using gaming services)
docker volume create minecraft-creative-data
docker volume create minecraft-creative-mods
docker volume create minecraft-survival-data
docker volume create minecraft-survival-mods
docker volume create minecraft-friends-data
docker volume create minecraft-friends-mods
docker volume create beammp-data
docker volume create beammp-config
docker volume create beammp-logs
docker volume create pterodactyl-data
docker volume create game-stats-data

# Verify volumes created
docker volume ls | wc -l
```

### **Step 3: Prepare Directory Structure**
```bash
# Create host directories for volume mounts
sudo mkdir -p /opt/photos /opt/storage /opt/media/{movies,tv,music}
sudo mkdir -p /opt/projects /opt/backup-source
sudo chown -R $USER:$USER /opt/{photos,storage,media,projects,backup-source}
```

---

## 🔐 Phase 4: Security Configuration

### **Step 1: Generate Strong Passwords**
```bash
# Install password generator
sudo apt install -y pwgen

# Generate passwords for services (save these securely!)
echo "=== INFRASTRUCTURE PASSWORDS ===" > ~/husserver-passwords.txt
echo "MYSQL_ROOT_PASSWORD=$(pwgen -s 32 1)" >> ~/husserver-passwords.txt
echo "TIMESCALE_PASSWORD=$(pwgen -s 32 1)" >> ~/husserver-passwords.txt
echo "REDIS_PASSWORD=$(pwgen -s 32 1)" >> ~/husserver-passwords.txt
echo "PHOTOPRISM_DB_PASSWORD=$(pwgen -s 32 1)" >> ~/husserver-passwords.txt
echo "IGNITION_DB_PASSWORD=$(pwgen -s 32 1)" >> ~/husserver-passwords.txt
echo "HUSSERVER_DB_PASSWORD=$(pwgen -s 32 1)" >> ~/husserver-passwords.txt
echo "NGINXPM_DB_PASSWORD=$(pwgen -s 32 1)" >> ~/husserver-passwords.txt
echo "PGADMIN_PASSWORD=$(pwgen -s 32 1)" >> ~/husserver-passwords.txt
echo "REDIS_COMMANDER_PASSWORD=$(pwgen -s 32 1)" >> ~/husserver-passwords.txt

echo "" >> ~/husserver-passwords.txt
echo "=== APPLICATION PASSWORDS ===" >> ~/husserver-passwords.txt
echo "PHOTOPRISM_ADMIN_PASSWORD=$(pwgen -s 16 1)" >> ~/husserver-passwords.txt
echo "IGNITION_ADMIN_PASSWORD=$(pwgen -s 16 1)" >> ~/husserver-passwords.txt
echo "EMQX_ADMIN_PASSWORD=$(pwgen -s 16 1)" >> ~/husserver-passwords.txt
echo "MQTT_PASSWORD=$(pwgen -s 16 1)" >> ~/husserver-passwords.txt
echo "CODE_SERVER_PASSWORD=$(pwgen -s 16 1)" >> ~/husserver-passwords.txt
echo "DOZZLE_PASSWORD=$(pwgen -s 16 1)" >> ~/husserver-passwords.txt

# Secure the password file
chmod 600 ~/husserver-passwords.txt

echo "🔐 Passwords generated and saved to ~/husserver-passwords.txt"
echo "⚠️  IMPORTANT: Back up this file securely and remove it after configuration!"
```

---

## 📋 Phase 5: Stack Deployment in Portainer

### **Step 1: Deploy Infrastructure Stack (CRITICAL - FIRST!)**

1. **Navigate to Portainer**: http://[SERVER_IP]:9000
2. **Go to Stacks** → **Add Stack**
3. **Stack Name**: `husserver-infrastructure`
4. **Build Method**: Upload
5. **Upload File**: `infrastructure-corrected-compose.yml`
6. **Environment Variables**: Copy from `portainer-infrastructure-corrected-env.txt` and replace all placeholder passwords with generated ones
7. **Deploy the Stack**
8. **Verify**: Check that all containers are running and healthy (may take 2-3 minutes)

**Critical Verification:**
```bash
# Check infrastructure containers
docker ps | grep -E "(mariadb-unified|timescaledb-unified|redis-unified)"

# Test database connections
docker exec mariadb-unified mysql -u root -p[YOUR_PASSWORD] -e "SHOW DATABASES;"
docker exec timescaledb-unified psql -U homeassistant -d homeassistant -c "\l"
```

### **Step 2: Deploy Media Storage Stack**

1. **Add Stack**: `husserver-media-storage`
2. **Upload File**: `media-storage-optimized-compose.yml`
3. **Environment Variables**: Copy from `portainer-media-optimized-env.txt`
4. **Deploy and Verify**: Check PhotoPrism and Jellyfin containers start

### **Step 3: Deploy Smart Home Stack**

1. **Add Stack**: `husserver-smart-home`
2. **Upload File**: `smart-home-optimized-compose.yml`
3. **Environment Variables**: Copy from `portainer-smart-home-env.txt`
4. **Deploy and Verify**: Check EMQX and Scrypted containers

### **Step 4: Deploy Industrial Stack**

1. **Add Stack**: `husserver-industrial`
2. **Upload File**: `industrial-optimized-compose.yml`
3. **Environment Variables**: Copy from `portainer-industrial-optimized-env.txt`
4. **Deploy and Verify**: Check Ignition container starts

### **Step 5: Deploy Utilities Stack**

1. **Add Stack**: `husserver-utilities`
2. **Upload File**: `utilities-compose.yml`
3. **Environment Variables**: Copy from `portainer-utilities-env.txt`
4. **Deploy and Verify**: Check Code Server and monitoring tools

### **Step 6: Deploy Network Stack**

1. **Add Stack**: `husserver-network`
2. **Upload File**: `network-compose.yml`
3. **Environment Variables**: Copy from `portainer-network-env.txt`
4. **Deploy and Verify**: Check Nginx Proxy Manager starts

### **Step 7: Deploy Gaming Stack (Optional)**

1. **Add Stack**: `husserver-gaming`
2. **Upload File**: `gaming-compose.yml`
3. **Environment Variables**: Copy from `portainer-gaming-env.txt`
4. **Deploy and Verify**: Check Minecraft servers start

---

## ✅ Phase 6: Initial Configuration

### **Step 1: Access Services and Complete Setup**

#### **Database Administration**
- **phpMyAdmin**: http://[SERVER_IP]:8787 (root / your_mysql_password)
- **pgAdmin**: http://[SERVER_IP]:5050 (admin@husserver.local / your_pgadmin_password)

#### **Media Services**
- **PhotoPrism**: http://[SERVER_IP]:2342 (admin / your_photoprism_password)
  - Complete initial setup wizard
  - Configure photo import directories
- **Jellyfin**: http://[SERVER_IP]:8096
  - Complete initial setup wizard
  - Add media libraries

#### **Smart Home Services**
- **EMQX Dashboard**: http://[SERVER_IP]:18083 (admin / your_emqx_password)
  - Configure authentication
  - Set up MQTT users

#### **Industrial Services**
- **Ignition Gateway**: http://[SERVER_IP]:9088 (admin / your_ignition_password)
  - Complete license activation
  - Configure database connections

#### **Development Tools**
- **Code Server**: https://[SERVER_IP]:8444 (password: your_code_server_password)
  - Install required extensions
- **Dozzle**: http://[SERVER_IP]:8888 (admin / your_dozzle_password)

#### **Network Services**
- **Nginx Proxy Manager**: http://[SERVER_IP]:81
  - Complete initial admin account setup
  - Configure SSL certificates

### **Step 2: Configure External Home Assistant Connection**

On your dedicated Home Assistant hardware, update configuration:

```yaml
# configuration.yaml
recorder:
  db_url: "postgresql://homeassistant:[TIMESCALE_PASSWORD]@[SERVER_IP]:5432/homeassistant"
  purge_keep_days: 30
  auto_purge: true
  auto_repack: true

# For LTSS integration
ltss:
  db_url: "postgresql://homeassistant:[TIMESCALE_PASSWORD]@[SERVER_IP]:5432/homeassistant"
  chunk_time_interval: 2592000000000
  include:
    entity_globs:
      - "sensor.nordpool_*"
      - "sensor.energy_*"
      - "sensor.billige_strømtimer"
```

---

## 🔍 Phase 7: Verification and Testing

### **Step 1: Health Check All Services**
```bash
# Check all containers are running
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Check stack status in Portainer
# All stacks should show as "Active" with green indicators
```

### **Step 2: Test Database Connections**
```bash
# Test MariaDB
docker exec mariadb-unified mysql -u photoprism -p[PHOTOPRISM_DB_PASSWORD] photoprism -e "SELECT 'PhotoPrism DB OK';"

# Test TimescaleDB
docker exec timescaledb-unified psql -U homeassistant -d homeassistant -c "SELECT 'TimescaleDB OK';"

# Test Redis
docker exec redis-unified redis-cli --raw ping
```

### **Step 3: Verify Web Interfaces**
Visit all service URLs and confirm they load correctly. Create a quick checklist:

- [ ] phpMyAdmin loads and can connect to databases
- [ ] pgAdmin loads and can connect to TimescaleDB
- [ ] PhotoPrism interface loads and can scan photos
- [ ] Jellyfin interface loads (media libraries can be configured)
- [ ] EMQX dashboard loads and shows broker status
- [ ] Ignition Gateway loads (requires license for full functionality)
- [ ] Code Server loads and development environment works
- [ ] Nginx Proxy Manager admin interface loads

---

## 🔐 Phase 8: Security Hardening

### **Step 1: Clean Up Installation Files**
```bash
# Secure and remove password file after use
cp ~/husserver-passwords.txt ~/husserver-passwords-backup.txt
chmod 600 ~/husserver-passwords-backup.txt
rm ~/husserver-passwords.txt

echo "⚠️ Password backup stored in ~/husserver-passwords-backup.txt"
echo "⚠️ Store this file securely and remove from server when no longer needed"
```

### **Step 2: Configure Firewall (if not using Nginx Proxy Manager for all external access)**
```bash
# Install UFW if not present
sudo apt install -y ufw

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH
sudo ufw allow ssh

# Allow Portainer
sudo ufw allow 9000

# Allow specific service ports (adjust based on your external access needs)
sudo ufw allow 8787  # phpMyAdmin (consider restricting to local network)
sudo ufw allow 5432  # TimescaleDB (for external Home Assistant)

# Enable firewall
sudo ufw enable
```

### **Step 3: Set Up Regular Backups**
```bash
# Create backup script
cat > ~/backup-husserver.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/opt/backup-source/husserver-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$BACKUP_DIR"

# Database backups
docker exec mariadb-unified mysqldump -u root -p[YOUR_MYSQL_ROOT_PASSWORD] --all-databases > "$BACKUP_DIR/mariadb-backup.sql"
docker exec timescaledb-unified pg_dumpall -U homeassistant > "$BACKUP_DIR/timescaledb-backup.sql"

# Configuration backups
cp -r /opt/husserver "$BACKUP_DIR/configs"

echo "Backup completed: $BACKUP_DIR"
EOF

chmod +x ~/backup-husserver.sh

# Set up weekly backup cron job
(crontab -l 2>/dev/null; echo "0 2 * * 0 ~/backup-husserver.sh") | crontab -
```

---

## 🎉 Installation Complete!

### **🚀 Next Steps:**

1. **Document Your Configuration**: Save all passwords and configuration notes
2. **Test External Connections**: Verify Home Assistant can connect to TimescaleDB
3. **Configure Media Libraries**: Add your media collections to Jellyfin
4. **Set Up Monitoring**: Configure Uptime Kuma for service monitoring  
5. **Explore Services**: Familiarize yourself with each service's capabilities

### **📚 Additional Resources:**

- **Troubleshooting Guide**: `docs/TROUBLESHOOTING.md`
- **Service Documentation**: Each service has extensive online documentation
- **Community Support**: Service-specific forums and communities

### **⚠️ Important Notes:**

- **Backup Regularly**: Automated backups are configured, but verify they work
- **Monitor Resources**: Use `htop` and Docker stats to monitor system performance
- **Update Regularly**: Use Watchtower for automated updates or manual updates via Portainer
- **Security**: Regularly review and update passwords, especially for internet-facing services

**🎯 Your unified, efficient, and powerful home server infrastructure is now ready!**

---

*Installation guide version: 2.0 (September 2025)*