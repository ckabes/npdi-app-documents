# Backup and Recovery Guide

**Version:** 1.0.0
**Last Updated:** 2026-01-09

---

## Table of Contents

1. [Overview](#overview)
2. [Backup Strategy](#backup-strategy)
3. [MongoDB Backup Procedures](#mongodb-backup-procedures)
4. [Application File Backup](#application-file-backup)
5. [Automated Backup Setup](#automated-backup-setup)
6. [Recovery Procedures](#recovery-procedures)
7. [Disaster Recovery Plan](#disaster-recovery-plan)
8. [Testing Backups](#testing-backups)

---

## Overview

This guide covers backup and recovery procedures for the NPDI application, including:
- MongoDB database backups
- Application configuration backups
- Uploaded file backups
- Disaster recovery planning

### Critical Data

| Data Type | Location | Backup Priority |
|-----------|----------|-----------------|
| MongoDB database | MongoDB server | **Critical** |
| `.env` file | Application root | **Critical** |
| Log files | `/logs` | Medium |
| Uploaded files | `/uploads` (if applicable) | High |
| SSL certificates | `/etc/letsencrypt` | High |

---

## Backup Strategy

### Backup Types

| Type | Frequency | Retention | Purpose |
|------|-----------|-----------|---------|
| Full backup | Daily | 30 days | Complete data recovery |
| Incremental | Hourly | 7 days | Point-in-time recovery |
| Configuration | On change | Indefinite | System recovery |

### Storage Recommendations

1. **Local backup**: Fast recovery, vulnerable to hardware failure
2. **Network storage (NAS)**: Good balance of speed and redundancy
3. **Cloud storage (S3, Azure Blob)**: Off-site, disaster recovery
4. **Use at least 2 backup destinations**

---

## MongoDB Backup Procedures

### Manual Backup with mongodump

```bash
# Full database backup
mongodump --db npdi-app --out /backup/mongodb/$(date +%Y-%m-%d)

# With authentication
mongodump --db npdi-app \
  --username npdi_app \
  --password "your_password" \
  --authenticationDatabase npdi-app \
  --out /backup/mongodb/$(date +%Y-%m-%d)

# Compressed backup
mongodump --db npdi-app \
  --gzip \
  --archive=/backup/mongodb/npdi-$(date +%Y-%m-%d).gz
```

### Backup Specific Collections

```bash
# Backup only tickets
mongodump --db npdi-app \
  --collection producttickets \
  --out /backup/mongodb/tickets-$(date +%Y-%m-%d)

# Backup critical collections only
mongodump --db npdi-app \
  --collection producttickets \
  --collection users \
  --collection systemsettings \
  --out /backup/mongodb/critical-$(date +%Y-%m-%d)
```

### Point-in-Time Backup (Oplog)

For MongoDB replica sets:

```bash
# Backup with oplog for point-in-time recovery
mongodump --db npdi-app \
  --oplog \
  --out /backup/mongodb/$(date +%Y-%m-%d)
```

### MongoDB Atlas Backup

If using MongoDB Atlas:
1. Automatic backups are enabled by default
2. Access via Atlas Dashboard → Backup
3. Configure continuous backup or daily snapshots

---

## Application File Backup

### Configuration Files

```bash
# Create backup directory
mkdir -p /backup/npdi-config/$(date +%Y-%m-%d)

# Backup environment file
cp /var/www/npdi-app/.env /backup/npdi-config/$(date +%Y-%m-%d)/

# Backup PM2 config
cp /var/www/npdi-app/ecosystem.config.js /backup/npdi-config/$(date +%Y-%m-%d)/

# Backup nginx config
cp /etc/nginx/sites-available/npdi-app /backup/npdi-config/$(date +%Y-%m-%d)/
```

### Log Files (Optional)

```bash
# Archive logs older than 7 days
find /var/www/npdi-app/logs -name "*.log" -mtime +7 -exec gzip {} \;

# Backup compressed logs
cp /var/www/npdi-app/logs/*.gz /backup/npdi-logs/$(date +%Y-%m-%d)/
```

---

## Automated Backup Setup

### Backup Script

Create `/usr/local/bin/npdi-backup.sh`:

```bash
#!/bin/bash
#
# NPDI Application Backup Script
# Run daily via cron
#

set -e

# Configuration
BACKUP_DIR="/backup/npdi"
MONGO_DB="npdi-app"
RETENTION_DAYS=30
DATE=$(date +%Y-%m-%d)
TIMESTAMP=$(date +%Y-%m-%d_%H-%M-%S)

# Create backup directories
mkdir -p "$BACKUP_DIR/mongodb"
mkdir -p "$BACKUP_DIR/config"
mkdir -p "$BACKUP_DIR/logs"

echo "[$TIMESTAMP] Starting NPDI backup..."

# MongoDB backup
echo "[$TIMESTAMP] Backing up MongoDB..."
mongodump --db "$MONGO_DB" \
  --gzip \
  --archive="$BACKUP_DIR/mongodb/npdi-$DATE.gz"

if [ $? -eq 0 ]; then
    echo "[$TIMESTAMP] MongoDB backup completed: $BACKUP_DIR/mongodb/npdi-$DATE.gz"
else
    echo "[$TIMESTAMP] ERROR: MongoDB backup failed!"
    exit 1
fi

# Configuration backup
echo "[$TIMESTAMP] Backing up configuration..."
cp /var/www/npdi-app/.env "$BACKUP_DIR/config/.env-$DATE"
cp /var/www/npdi-app/ecosystem.config.js "$BACKUP_DIR/config/ecosystem.config.js-$DATE" 2>/dev/null || true

# Cleanup old backups
echo "[$TIMESTAMP] Cleaning up backups older than $RETENTION_DAYS days..."
find "$BACKUP_DIR/mongodb" -name "*.gz" -mtime +$RETENTION_DAYS -delete
find "$BACKUP_DIR/config" -name ".env-*" -mtime +$RETENTION_DAYS -delete

# Calculate backup size
BACKUP_SIZE=$(du -sh "$BACKUP_DIR/mongodb/npdi-$DATE.gz" | cut -f1)
echo "[$TIMESTAMP] Backup size: $BACKUP_SIZE"

echo "[$TIMESTAMP] Backup completed successfully!"
```

Make executable:

```bash
chmod +x /usr/local/bin/npdi-backup.sh
```

### Cron Schedule

```bash
# Edit crontab
crontab -e

# Add backup schedule
# Daily backup at 2:00 AM
0 2 * * * /usr/local/bin/npdi-backup.sh >> /var/log/npdi-backup.log 2>&1

# Weekly full backup on Sunday at 3:00 AM
0 3 * * 0 /usr/local/bin/npdi-backup.sh --full >> /var/log/npdi-backup.log 2>&1
```

### Cloud Backup (AWS S3)

Add to backup script:

```bash
# Upload to S3
aws s3 cp "$BACKUP_DIR/mongodb/npdi-$DATE.gz" \
  "s3://your-bucket/npdi-backups/mongodb/npdi-$DATE.gz"

# Sync to S3 with lifecycle management
aws s3 sync "$BACKUP_DIR" "s3://your-bucket/npdi-backups/" \
  --exclude "*.log"
```

---

## Recovery Procedures

### Restore from mongodump

#### Full Database Restore

```bash
# Stop the application first
pm2 stop npdi-app

# Restore from compressed archive
mongorestore --db npdi-app \
  --gzip \
  --archive=/backup/mongodb/npdi-2025-01-09.gz \
  --drop  # Drops existing collections before restore

# Or from directory
mongorestore --db npdi-app \
  /backup/mongodb/2025-01-09/npdi-app \
  --drop

# Restart application
pm2 start npdi-app
```

#### Restore Specific Collections

```bash
# Restore only tickets collection
mongorestore --db npdi-app \
  --collection producttickets \
  /backup/mongodb/2025-01-09/npdi-app/producttickets.bson
```

#### Restore with Authentication

```bash
mongorestore --db npdi-app \
  --username npdi_app \
  --password "your_password" \
  --authenticationDatabase npdi-app \
  --gzip \
  --archive=/backup/mongodb/npdi-2025-01-09.gz
```

### Restore Individual Documents

If you only need to restore specific documents:

```bash
# Start mongosh
mongosh npdi-app

# Export backup collection to find document
# Then manually insert or update
db.producttickets.insertOne({
  // document data
})
```

### Restore Configuration Files

```bash
# Stop application
pm2 stop npdi-app

# Restore .env
cp /backup/npdi-config/2025-01-09/.env /var/www/npdi-app/.env

# Restore PM2 config
cp /backup/npdi-config/2025-01-09/ecosystem.config.js /var/www/npdi-app/

# Restart
pm2 start npdi-app
```

---

## Disaster Recovery Plan

### Scenario 1: Database Corruption

**Symptoms:** Application errors, data inconsistency

**Recovery Steps:**

1. Stop application: `pm2 stop npdi-app`
2. Identify last known good backup
3. Restore database:
   ```bash
   mongorestore --db npdi-app --drop --gzip \
     --archive=/backup/mongodb/npdi-YYYY-MM-DD.gz
   ```
4. Verify data integrity
5. Restart application: `pm2 start npdi-app`

### Scenario 2: Server Failure

**Prerequisites:** Backups stored off-site (cloud/remote)

**Recovery Steps:**

1. Provision new server
2. Install prerequisites (Node.js, MongoDB, nginx)
3. Clone application code
4. Download latest backup from cloud storage
5. Restore database
6. Restore configuration files
7. Update DNS to point to new server
8. Verify application functionality

### Scenario 3: Accidental Data Deletion

**Recovery for deleted tickets:**

```bash
# Find document in backup
mongorestore --db npdi-app-recovery \
  --gzip \
  --archive=/backup/mongodb/npdi-YYYY-MM-DD.gz

# In mongosh, find the document
use npdi-app-recovery
db.producttickets.find({ ticketNumber: "NPDI-2025-0055" })

# Copy to production database
db.producttickets.find({ ticketNumber: "NPDI-2025-0055" }).forEach(function(doc) {
  db.getSiblingDB('npdi-app').producttickets.insertOne(doc);
})
```

### Recovery Time Objectives (RTO)

| Scenario | Target RTO | Notes |
|----------|------------|-------|
| Database corruption | 1 hour | Restore from backup |
| Server failure | 4 hours | New server provisioning |
| Data deletion | 30 minutes | Selective restore |
| Complete data center loss | 24 hours | Off-site recovery |

---

## Testing Backups

### Monthly Backup Verification

```bash
#!/bin/bash
# backup-test.sh - Monthly backup verification

BACKUP_FILE="/backup/mongodb/npdi-$(date +%Y-%m-%d).gz"
TEST_DB="npdi-app-test"

echo "Testing backup restore to $TEST_DB..."

# Restore to test database
mongorestore --db $TEST_DB \
  --gzip \
  --archive=$BACKUP_FILE \
  --drop

# Verify document counts
mongosh $TEST_DB --eval "
  print('Users:', db.users.countDocuments());
  print('Tickets:', db.producttickets.countDocuments());
  print('Settings:', db.systemsettings.countDocuments());
"

# Clean up test database
mongosh $TEST_DB --eval "db.dropDatabase()"

echo "Backup verification completed!"
```

### Verification Checklist

- [ ] Backup file exists and is not empty
- [ ] Backup can be decompressed without errors
- [ ] Database can be restored to test environment
- [ ] Document counts match expected values
- [ ] Random data spot checks pass
- [ ] Application functions correctly with restored data

---

## Backup Monitoring

### Email Notification Script

Add to backup script:

```bash
# At the end of backup script
if [ $? -eq 0 ]; then
    echo "NPDI backup completed successfully on $(date)" | \
      mail -s "NPDI Backup Success" admin@company.com
else
    echo "NPDI backup FAILED on $(date)" | \
      mail -s "ALERT: NPDI Backup Failed" admin@company.com
fi
```

### Monitoring Metrics

Track these metrics:
- Backup file size trends
- Backup duration
- Last successful backup time
- Storage space remaining

---

**Last Updated:** 2026-01-09
**Maintained by:** NPDI Development Team
