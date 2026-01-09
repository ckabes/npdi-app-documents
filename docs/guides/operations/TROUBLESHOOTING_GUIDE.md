# Troubleshooting Guide

**Version:** 1.0.0
**Last Updated:** 2026-01-09

---

## Table of Contents

1. [Quick Diagnostics](#quick-diagnostics)
2. [Startup Issues](#startup-issues)
3. [Database Issues](#database-issues)
4. [Authentication Issues](#authentication-issues)
5. [API Issues](#api-issues)
6. [Frontend Issues](#frontend-issues)
7. [Integration Issues](#integration-issues)
8. [Performance Issues](#performance-issues)
9. [Log Analysis](#log-analysis)
10. [Common Error Messages](#common-error-messages)

---

## Quick Diagnostics

### Health Check Endpoints

```bash
# Server health check
curl http://localhost:5000/api/health

# Database connectivity
curl http://localhost:5000/api/health/db

# Full system status (requires METRICS_TOKEN)
curl -H "Authorization: Bearer YOUR_METRICS_TOKEN" http://localhost:5000/api/metrics
```

### Quick Status Commands

```bash
# Check if server is running
lsof -i :5000

# Check if MongoDB is running
mongosh --eval "db.adminCommand('ping')"

# Check Node.js version
node --version  # Should be 16+

# Check recent logs
tail -100 logs/combined.log
tail -100 logs/error.log
```

---

## Startup Issues

### Server Won't Start

#### Error: "EADDRINUSE: address already in use"

**Cause:** Port 5000 is already in use by another process.

**Solutions:**

```bash
# Find process using port 5000
lsof -i :5000

# Kill the process
kill -9 <PID>

# Or use a different port
PORT=5001 npm run server:dev
```

#### Error: "Cannot find module"

**Cause:** Dependencies not installed or corrupted.

**Solutions:**

```bash
# Clean install
rm -rf node_modules package-lock.json
npm install

# Also for client
cd client
rm -rf node_modules package-lock.json
npm install
```

#### Error: "MongooseServerSelectionError"

**Cause:** Cannot connect to MongoDB.

**Solutions:**

1. Check MongoDB is running:
   ```bash
   sudo systemctl status mongod
   # or
   mongod --version
   ```

2. Start MongoDB if not running:
   ```bash
   sudo systemctl start mongod
   ```

3. Verify connection string in `.env`:
   ```bash
   MONGODB_URI=mongodb://localhost:27017/npdi-app
   ```

4. Check if MongoDB port is accessible:
   ```bash
   telnet localhost 27017
   ```

---

## Database Issues

### Connection Timeout

**Symptoms:** API requests hang, then fail with timeout errors.

**Causes:**
- MongoDB not running
- Network issues
- Connection pool exhausted

**Solutions:**

1. Restart MongoDB:
   ```bash
   sudo systemctl restart mongod
   ```

2. Check MongoDB logs:
   ```bash
   tail -100 /var/log/mongodb/mongod.log
   ```

3. Verify connection string format:
   ```bash
   # Local
   MONGODB_URI=mongodb://localhost:27017/npdi-app

   # With auth
   MONGODB_URI=mongodb://user:password@localhost:27017/npdi-app?authSource=admin

   # Atlas
   MONGODB_URI=mongodb+srv://user:password@cluster.mongodb.net/npdi-app
   ```

### Duplicate Key Error

**Error:** `E11000 duplicate key error collection`

**Cause:** Attempting to insert a document with a value that already exists for a unique field.

**Solutions:**

1. For tickets with duplicate `ticketNumber`:
   ```javascript
   // In mongosh
   db.producttickets.find({ ticketNumber: "NPDI-2025-0001" })
   ```

2. For users with duplicate `employeeId` or `email`:
   ```javascript
   db.users.find({ employeeId: "M361549" })
   ```

3. Remove duplicates if safe to do so:
   ```javascript
   // Careful! Verify before deleting
   db.producttickets.deleteOne({ _id: ObjectId("...") })
   ```

### Schema Validation Errors

**Error:** `ValidationError: ProductTicket validation failed`

**Cause:** Data doesn't match schema constraints (enum values, required fields).

**Common Issues:**

1. Invalid enum value:
   ```
   Error: 'xyz' is not a valid enum value for path 'status'
   ```

   **Solution:** Check valid enum values in schema documentation.

2. Required field missing:
   ```
   Error: Path 'productName' is required
   ```

   **Solution:** Ensure required fields are provided.

---

## Authentication Issues

### User Cannot Login

**Symptoms:** User selects profile but gets error or blank screen.

**Checklist:**

1. Verify user exists in database:
   ```javascript
   // In mongosh
   db.users.findOne({ employeeId: "M361549" })
   ```

2. Check user is active:
   ```javascript
   db.users.findOne({ employeeId: "M361549", isActive: true })
   ```

3. Verify user has valid role:
   ```javascript
   // Role must be: PRODUCT_MANAGER, PM_OPS, or ADMIN
   db.users.findOne({ employeeId: "M361549" }).role
   ```

4. Check browser console for errors.

### Permission Denied

**Symptoms:** User can login but cannot access certain features.

**Solutions:**

1. Check user's role in database:
   ```javascript
   db.users.findOne({ employeeId: "M361549" }).role
   ```

2. Verify permissions for role:
   ```javascript
   db.permissions.findOne({ role: "PRODUCT_MANAGER" })
   ```

3. Reinitialize permissions if corrupted:
   ```bash
   node server/scripts/initialize-permissions.js
   ```

---

## API Issues

### 401 Unauthorized (API Key)

**Error:** External API request returns 401.

**Solutions:**

1. Verify API key is active:
   ```javascript
   // In mongosh
   db.apikeys.findOne({ keyPrefix: "abc12345" })
   ```

2. Check key hasn't expired:
   ```javascript
   db.apikeys.findOne({ keyPrefix: "abc12345" }).expiresAt
   ```

3. Verify correct header format:
   ```bash
   curl -H "X-API-Key: your-full-api-key" http://localhost:5000/api/v1/tickets
   ```

4. Generate new key if needed:
   ```bash
   node server/scripts/generateApiKey.js "Application Name"
   ```

### 400 Bad Request

**Error:** API returns validation error.

**Solutions:**

1. Check request body format:
   ```bash
   # Use proper JSON content type
   curl -X POST \
     -H "Content-Type: application/json" \
     -H "X-API-Key: your-key" \
     -d '{"productName": "Test", "sbu": "775"}' \
     http://localhost:5000/api/v1/tickets
   ```

2. Review required fields in API documentation.

3. Check server logs for validation details:
   ```bash
   tail -50 logs/combined.log | grep "validation"
   ```

### 500 Internal Server Error

**Solutions:**

1. Check error logs:
   ```bash
   tail -100 logs/error.log
   ```

2. Look for stack trace in combined log:
   ```bash
   grep -A 20 "Error" logs/combined.log | tail -40
   ```

3. Common causes:
   - Database connection issues
   - Null pointer exceptions
   - External service failures

---

## Frontend Issues

### Blank White Screen

**Cause:** JavaScript error preventing React from rendering.

**Solutions:**

1. Open browser DevTools (F12) and check Console tab.

2. Clear browser cache:
   - Chrome: Ctrl+Shift+Delete → Clear cached images and files

3. Rebuild client:
   ```bash
   cd client
   npm run build
   ```

4. Check for missing environment variables:
   ```bash
   # client/.env should exist if custom config needed
   ls -la client/.env
   ```

### Form Not Saving

**Symptoms:** Changes to ticket forms not persisting.

**Solutions:**

1. Check browser console for API errors.

2. Verify network requests in DevTools → Network tab.

3. Check if ticket is in editable state:
   - DRAFT tickets: Fully editable by creator
   - SUBMITTED tickets: Limited editing based on role

4. Check local storage isn't corrupted:
   ```javascript
   // In browser console
   localStorage.clear()
   ```

### Stale Data / Not Updating

**Symptoms:** Changes made by other users not appearing.

**Solutions:**

1. Hard refresh: Ctrl+Shift+R (or Cmd+Shift+R on Mac)

2. Clear React Query cache:
   ```javascript
   // In browser console (if available)
   window.__REACT_QUERY_DEVTOOLS_GLOBAL_CACHE__.clear()
   ```

3. Restart the development server.

---

## Integration Issues

### PubChem Not Working

**Symptoms:** CAS lookup returns no data or errors.

**Checklist:**

1. Check integration is enabled:
   ```javascript
   // In mongosh
   db.systemsettings.findOne().integrations.pubchem.enabled
   ```

2. Test PubChem directly:
   ```bash
   curl "https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/name/64-17-5/JSON"
   ```

3. Check for rate limiting (too many requests).

4. Verify CAS number format: `XXXXX-XX-X`

### Palantir Connection Failed

**Symptoms:** MARA lookup returns connection error.

**Solutions:**

1. Verify Palantir is enabled and configured:
   ```javascript
   db.systemsettings.findOne().integrations.palantir
   ```

2. Check token is valid (encrypted in DB):
   - Re-enter token via Admin UI if expired

3. Verify hostname is correct:
   ```javascript
   db.systemsettings.findOne().integrations.palantir.hostname
   // Should be: palantir.mcloud.merckgroup.com
   ```

4. Test connectivity:
   ```bash
   curl -I https://palantir.mcloud.merckgroup.com
   ```

5. Run diagnostic script:
   ```bash
   node test-palantir-connection.js
   ```

### Teams Notifications Not Sending

**Solutions:**

1. Check Teams integration is enabled:
   ```javascript
   db.systemsettings.findOne().integrations.teams.enabled
   ```

2. Verify webhook URL is configured:
   ```javascript
   db.systemsettings.findOne().integrations.teams.webhookUrl
   ```

3. Test webhook manually:
   ```bash
   curl -X POST \
     -H "Content-Type: application/json" \
     -d '{"text": "Test message from NPDI"}' \
     "YOUR_WEBHOOK_URL"
   ```

4. Check if specific notifications are enabled:
   ```javascript
   db.systemsettings.findOne().integrations.teams.notifyOnStatusChange
   ```

---

## Performance Issues

### Slow API Responses

**Diagnosis:**

1. Check response times in logs:
   ```bash
   grep "HTTP" logs/combined.log | tail -20
   ```

2. Look for slow queries:
   ```bash
   grep -i "slow" logs/combined.log
   ```

**Solutions:**

1. Add missing indexes (check DATABASE_SCHEMA_REFERENCE.md).

2. Enable caching in SystemSettings:
   ```javascript
   db.systemsettings.updateOne({}, { $set: { "performance.cache.enabled": true } })
   ```

3. Check MongoDB performance:
   ```javascript
   // In mongosh
   db.producttickets.stats()
   db.producttickets.getIndexes()
   ```

### High Memory Usage

**Diagnosis:**

```bash
# Check Node.js memory
ps aux | grep node

# Monitor in real-time
top -p $(pgrep -f "node server")
```

**Solutions:**

1. Restart the server (quick fix):
   ```bash
   npm run server:dev
   ```

2. Check for memory leaks in custom code.

3. Increase Node.js memory limit if needed:
   ```bash
   NODE_OPTIONS="--max-old-space-size=4096" npm start
   ```

---

## Log Analysis

### Log File Locations

| File | Contents |
|------|----------|
| `logs/combined.log` | All log entries |
| `logs/error.log` | Errors only |
| `logs/YYYY-MM-DD-combined.log` | Daily rotated logs |

### Useful Log Queries

```bash
# Recent errors
tail -100 logs/error.log

# Specific endpoint issues
grep "POST /api/products" logs/combined.log | tail -20

# User activity
grep "M361549" logs/combined.log | tail -20

# Database operations
grep -i "mongo" logs/combined.log | tail -20

# HTTP 500 errors
grep "500" logs/combined.log | tail -20

# Today's errors
grep "$(date +%Y-%m-%d)" logs/error.log
```

---

## Common Error Messages

| Error Message | Likely Cause | Solution |
|---------------|--------------|----------|
| `ECONNREFUSED` | Service not running | Start MongoDB/server |
| `MongooseServerSelectionError` | MongoDB unreachable | Check MongoDB status |
| `JsonWebTokenError` | Invalid or expired token | Re-authenticate |
| `ValidationError` | Schema validation failed | Check required fields |
| `E11000 duplicate key` | Unique constraint violation | Remove duplicate |
| `ENOENT: no such file` | Missing file | Check file path |
| `Encryption key not set` | Missing ENCRYPTION_KEY | Add to .env |
| `Rate limit exceeded` | Too many requests | Wait or increase limit |
| `Network Error` | CORS or connectivity | Check CLIENT_URL |

---

## Getting Help

If you cannot resolve an issue:

1. **Check existing documentation** in `/docs`

2. **Search logs** for error details:
   ```bash
   grep -r "your error message" logs/
   ```

3. **Submit feedback** using the in-app feedback button (captures context automatically)

4. **Contact support** with:
   - Error message (exact text)
   - Steps to reproduce
   - Relevant log entries
   - Browser/environment details

---

**Last Updated:** 2026-01-09
**Maintained by:** NPDI Development Team
