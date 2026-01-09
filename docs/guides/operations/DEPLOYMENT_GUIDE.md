# Production Deployment Guide

**Version:** 1.0.0
**Last Updated:** 2026-01-09

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Server Requirements](#server-requirements)
3. [Deployment Options](#deployment-options)
4. [Manual Deployment](#manual-deployment)
5. [Process Manager Setup](#process-manager-setup)
6. [Reverse Proxy Configuration](#reverse-proxy-configuration)
7. [SSL/TLS Configuration](#ssltls-configuration)
8. [Environment Configuration](#environment-configuration)
9. [Database Setup](#database-setup)
10. [Post-Deployment Verification](#post-deployment-verification)
11. [Monitoring & Maintenance](#monitoring--maintenance)

---

## Prerequisites

### Required Software

| Software | Minimum Version | Purpose |
|----------|-----------------|---------|
| Node.js | 16.x LTS | Runtime environment |
| npm | 8.x | Package management |
| MongoDB | 5.x | Database |
| Git | 2.x | Version control |
| nginx or Apache | Latest | Reverse proxy |

### Recommended Software

| Software | Purpose |
|----------|---------|
| PM2 | Process manager |
| certbot | SSL certificate management |
| fail2ban | Security hardening |
| ufw | Firewall management |

---

## Server Requirements

### Minimum Hardware

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU | 2 cores | 4+ cores |
| RAM | 4 GB | 8+ GB |
| Storage | 20 GB SSD | 50+ GB SSD |
| Network | 100 Mbps | 1 Gbps |

### Operating System

- Ubuntu 20.04 LTS or newer (recommended)
- CentOS 8 / RHEL 8 or newer
- Windows Server 2019 or newer (with adjustments)

---

## Deployment Options

### Option 1: Manual Deployment
Direct installation on a server with PM2 for process management.

### Option 2: Docker Deployment
Containerized deployment using Docker Compose.

### Option 3: Cloud Platform
- AWS Elastic Beanstalk
- Azure App Service
- Google Cloud Run

This guide focuses on **Option 1 (Manual Deployment)**.

---

## Manual Deployment

### Step 1: Server Preparation

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Node.js (via NodeSource)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Verify installation
node --version
npm --version

# Install build essentials (for native modules)
sudo apt install -y build-essential
```

### Step 2: Install MongoDB

```bash
# Import MongoDB GPG key
curl -fsSL https://pgp.mongodb.com/server-6.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-6.0.gpg --dearmor

# Add MongoDB repository
echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-6.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list

# Install MongoDB
sudo apt update
sudo apt install -y mongodb-org

# Start MongoDB
sudo systemctl start mongod
sudo systemctl enable mongod

# Verify MongoDB is running
sudo systemctl status mongod
```

### Step 3: Create Application User

```bash
# Create dedicated user
sudo useradd -m -s /bin/bash npdi
sudo passwd npdi

# Create application directory
sudo mkdir -p /var/www/npdi-app
sudo chown npdi:npdi /var/www/npdi-app
```

### Step 4: Clone Repository

```bash
# Switch to npdi user
sudo su - npdi

# Clone repository
cd /var/www/npdi-app
git clone <repository-url> .

# Or copy files from release
# scp -r npdi-app/* npdi@server:/var/www/npdi-app/
```

### Step 5: Install Dependencies

```bash
# Install server dependencies
npm install --production

# Install client dependencies and build
cd client
npm install
npm run build
cd ..
```

### Step 6: Configure Environment

```bash
# Create production environment file
cp .env.example .env
nano .env
```

**Production `.env` content:**

```bash
# Database
MONGODB_URI=mongodb://localhost:27017/npdi-app

# Server
PORT=5000
NODE_ENV=production
CLIENT_URL=https://npdi.yourdomain.com

# Security (REQUIRED)
ENCRYPTION_KEY=<generate with: node server/scripts/generateEncryptionKey.js>

# Logging
LOG_LEVEL=info

# Metrics (optional but recommended)
METRICS_TOKEN=<generate with: openssl rand -hex 32>
```

### Step 7: Initialize Database

```bash
# Initialize permissions
node server/scripts/initialize-permissions.js

# Seed form configurations (if fresh install)
node server/scripts/seedFormConfig.js
node server/scripts/seedBiologicFormConfig.js

# Create initial admin user (if needed)
# Use mongosh or create via the application
```

---

## Process Manager Setup

### PM2 Installation

```bash
# Install PM2 globally
sudo npm install -g pm2

# Generate PM2 startup script
pm2 startup
# Follow the instructions output
```

### PM2 Configuration

Create `ecosystem.config.js` in the application root:

```javascript
module.exports = {
  apps: [{
    name: 'npdi-app',
    script: 'server/index.js',
    instances: 'max',  // Use all CPU cores
    exec_mode: 'cluster',
    env_production: {
      NODE_ENV: 'production',
      PORT: 5000
    },
    error_file: '/var/log/npdi/pm2-error.log',
    out_file: '/var/log/npdi/pm2-out.log',
    log_file: '/var/log/npdi/pm2-combined.log',
    time: true,
    max_memory_restart: '1G',
    node_args: '--max-old-space-size=4096'
  }]
};
```

### Start Application with PM2

```bash
# Create log directory
sudo mkdir -p /var/log/npdi
sudo chown npdi:npdi /var/log/npdi

# Start application
pm2 start ecosystem.config.js --env production

# Save PM2 process list
pm2 save

# Check status
pm2 status
pm2 logs npdi-app
```

### PM2 Management Commands

```bash
pm2 restart npdi-app    # Restart application
pm2 stop npdi-app       # Stop application
pm2 reload npdi-app     # Zero-downtime reload
pm2 delete npdi-app     # Remove from PM2
pm2 monit               # Real-time monitoring
```

---

## Reverse Proxy Configuration

### Nginx Installation

```bash
sudo apt install -y nginx
sudo systemctl enable nginx
```

### Nginx Configuration

Create `/etc/nginx/sites-available/npdi-app`:

```nginx
upstream npdi_backend {
    server 127.0.0.1:5000;
    keepalive 64;
}

server {
    listen 80;
    server_name npdi.yourdomain.com;

    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name npdi.yourdomain.com;

    # SSL certificates (managed by certbot)
    ssl_certificate /etc/letsencrypt/live/npdi.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/npdi.yourdomain.com/privkey.pem;

    # SSL configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;

    # Client body size (for file uploads)
    client_max_body_size 50M;

    # Static files (React build)
    location / {
        root /var/www/npdi-app/client/dist;
        try_files $uri $uri/ /index.html;

        # Cache static assets
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }

    # API proxy
    location /api {
        proxy_pass http://npdi_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        proxy_read_timeout 300s;
        proxy_connect_timeout 75s;
    }
}
```

### Enable Site

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/npdi-app /etc/nginx/sites-enabled/

# Test configuration
sudo nginx -t

# Reload nginx
sudo systemctl reload nginx
```

---

## SSL/TLS Configuration

### Using Let's Encrypt (Recommended)

```bash
# Install certbot
sudo apt install -y certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d npdi.yourdomain.com

# Verify auto-renewal
sudo certbot renew --dry-run
```

### Certificate Renewal

Certbot automatically renews certificates. Verify cron job exists:

```bash
sudo systemctl status certbot.timer
```

---

## Environment Configuration

### Production Checklist

- [ ] `NODE_ENV=production`
- [ ] `ENCRYPTION_KEY` set (64 hex characters)
- [ ] `MONGODB_URI` points to production database
- [ ] `CLIENT_URL` matches your domain
- [ ] Database has authentication enabled
- [ ] SSL/TLS certificates installed
- [ ] Firewall configured
- [ ] Log rotation configured

### Security Hardening

```bash
# Configure firewall
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable

# Install fail2ban
sudo apt install -y fail2ban
sudo systemctl enable fail2ban
```

---

## Database Setup

### MongoDB Authentication

```bash
# Connect to MongoDB
mongosh

# Create admin user
use admin
db.createUser({
  user: "npdi_admin",
  pwd: "secure_password_here",
  roles: [{ role: "root", db: "admin" }]
})

# Create application user
use npdi-app
db.createUser({
  user: "npdi_app",
  pwd: "another_secure_password",
  roles: [{ role: "readWrite", db: "npdi-app" }]
})

# Enable authentication in /etc/mongod.conf
# security:
#   authorization: enabled

# Restart MongoDB
sudo systemctl restart mongod
```

Update `.env` with authenticated connection string:

```bash
MONGODB_URI=mongodb://npdi_app:password@localhost:27017/npdi-app?authSource=npdi-app
```

---

## Post-Deployment Verification

### Health Checks

```bash
# Check application is running
curl http://localhost:5000/api/health

# Check through reverse proxy
curl https://npdi.yourdomain.com/api/health

# Check PM2 status
pm2 status
pm2 logs npdi-app --lines 50
```

### Functional Tests

1. Access the application URL in browser
2. Login with test user
3. Create a draft ticket
4. Submit the ticket
5. Check logs for errors

### Performance Verification

```bash
# Check response time
curl -w "@curl-format.txt" -o /dev/null -s https://npdi.yourdomain.com/api/health

# Check memory usage
pm2 monit
```

---

## Monitoring & Maintenance

### Log Management

```bash
# Configure logrotate
sudo nano /etc/logrotate.d/npdi-app
```

```
/var/log/npdi/*.log
/var/www/npdi-app/logs/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0644 npdi npdi
    postrotate
        pm2 reloadLogs
    endscript
}
```

### Backup Schedule

```bash
# Add to crontab
crontab -e

# Daily database backup at 2 AM
0 2 * * * mongodump --db npdi-app --out /backup/mongodb/$(date +\%Y-\%m-\%d)
```

### Update Process

```bash
# Pull latest code
cd /var/www/npdi-app
git pull origin main

# Install any new dependencies
npm install --production

# Rebuild client
cd client
npm install
npm run build
cd ..

# Reload application (zero downtime)
pm2 reload npdi-app
```

---

**Last Updated:** 2026-01-09
**Maintained by:** NPDI Development Team
