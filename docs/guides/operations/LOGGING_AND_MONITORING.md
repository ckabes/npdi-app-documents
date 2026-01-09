# Logging and Monitoring Guide

**Version:** 1.1.0
**Last Updated:** 2026-01-09

---

## Table of Contents

1. [Overview](#overview)
2. [Log Files](#log-files)
3. [Log Levels](#log-levels)
4. [Log Format](#log-format)
5. [Logger Usage](#logger-usage)
6. [Log Analysis](#log-analysis)
7. [Metrics Endpoint](#metrics-endpoint)

---

## Overview

The NPDI application uses a custom zero-dependency logging system built on Node.js built-in modules. Logs are written to files with automatic rotation.

### Key Features

- **Zero external dependencies**: Uses only Node.js built-in modules (`fs`, `path`, `util`)
- **Daily log rotation**: Log files are named with date stamps
- **Size-based rotation**: Files rotate when exceeding 10MB
- **Multiple log levels**: ERROR, WARN, INFO, HTTP, DEBUG
- **Console output**: Colored output in development mode

---

## Log Files

### File Locations

```
npdi-app/
└── logs/
    ├── combined-2026-01-09.log   # All log entries for the day
    ├── error-2026-01-09.log      # Error-level logs only for the day
    ├── combined-2026-01-08.log   # Previous day's logs
    └── error-2026-01-08.log      # Previous day's error logs
```

**Note:** The `logs/` directory is created automatically on first run.

### File Naming Pattern

- **Combined logs**: `combined-YYYY-MM-DD.log`
- **Error logs**: `error-YYYY-MM-DD.log`

### Rotation Behavior

| Trigger | Action |
|---------|--------|
| New day | New log file created with current date |
| File > 10MB | File rotated with timestamp suffix |
| Files > 30 days | Old files automatically deleted |

---

## Log Levels

### Level Hierarchy

| Level | Priority | Description | Use Case |
|-------|----------|-------------|----------|
| `ERROR` | 0 | Critical errors | Exceptions, failures requiring attention |
| `WARN` | 1 | Warning conditions | Non-critical issues, deprecations |
| `INFO` | 2 | Informational | Normal operations, state changes |
| `HTTP` | 3 | HTTP requests | Request/response logging |
| `DEBUG` | 4 | Debug information | Development debugging |

### Setting Log Level

**Environment Variable:**
```bash
# In .env file
LOG_LEVEL=info    # Default
LOG_LEVEL=debug   # Verbose logging
LOG_LEVEL=error   # Errors only
```

**Effect:** Only logs at or above the configured level are recorded.

```bash
LOG_LEVEL=warn    # Records: ERROR, WARN
LOG_LEVEL=info    # Records: ERROR, WARN, INFO
LOG_LEVEL=http    # Records: ERROR, WARN, INFO, HTTP
LOG_LEVEL=debug   # Records: All levels
```

---

## Log Format

### Standard Log Entry

```
[2026-01-09T14:30:45.123Z] [INFO] Operation completed successfully {"userId":"M361549","ticketId":"NPDI-2026-0055"}
```

### Format Structure

```
[timestamp] [LEVEL] message {metadata}
```

| Component | Format | Example |
|-----------|--------|---------|
| Timestamp | ISO 8601 | `2026-01-09T14:30:45.123Z` |
| Level | Uppercase | `INFO`, `ERROR`, `DEBUG` |
| Message | Free text | `Ticket created successfully` |
| Metadata | JSON object | `{"ticketId":"123","userId":"M361549"}` |

### Example Log Entries

**Info:**
```
[2026-01-09T14:30:45.123Z] [INFO] Ticket created successfully {"ticketId":"6789abc","ticketNumber":"NPDI-2026-0055","createdBy":"M361549"}
```

**Error:**
```
[2026-01-09T14:30:45.123Z] [ERROR] Database query failed {"error":"Connection timeout","operation":"findById","collection":"producttickets"}
```

**HTTP:**
```
[2026-01-09T14:30:45.123Z] [HTTP] GET /api/products 200 45ms {"method":"GET","url":"/api/products","status":200,"duration":45}
```

---

## Logger Usage

### Import and Basic Usage

```javascript
const logger = require('../utils/logger');

// Info level
logger.info('Ticket created successfully', {
  ticketId: ticket._id,
  ticketNumber: ticket.ticketNumber,
  createdBy: userId
});

// Warning level
logger.warn('Rate limit approaching', {
  currentCount: count,
  limit: MAX_REQUESTS
});

// Error level
logger.error('Failed to process ticket', {
  error: error.message,
  stack: error.stack,
  ticketId: req.params.id
});

// Debug level (only in development or when LOG_LEVEL=debug)
logger.debug('Query parameters', {
  filters: req.query,
  page: req.query.page
});
```

### HTTP Request Logging

The logger includes middleware for automatic HTTP request logging:

```javascript
// In server/index.js
const logger = require('./utils/logger');

// Bind the middleware to preserve 'this' context
app.use(logger.httpLogger.bind(logger));
```

This automatically logs all HTTP requests with method, URL, status code, and duration.

### Controller Pattern

```javascript
const getTicket = async (req, res) => {
  try {
    const ticket = await ProductTicket.findById(req.params.id);

    if (!ticket) {
      logger.warn('Ticket not found', {
        ticketId: req.params.id,
        userId: req.headers['x-user-employee-id']
      });
      return res.status(404).json({ message: 'Ticket not found' });
    }

    logger.info('Ticket retrieved', {
      ticketId: ticket._id,
      ticketNumber: ticket.ticketNumber
    });

    res.json(ticket);
  } catch (error) {
    logger.error('Error fetching ticket', {
      error: error.message,
      stack: error.stack,
      ticketId: req.params.id
    });
    res.status(500).json({ message: 'Internal server error' });
  }
};
```

---

## Log Analysis

### View Today's Logs

```bash
# View combined logs
cat logs/combined-$(date +%Y-%m-%d).log

# View error logs
cat logs/error-$(date +%Y-%m-%d).log

# Follow logs in real-time
tail -f logs/combined-$(date +%Y-%m-%d).log
```

### Common Log Queries

**View Recent Errors:**
```bash
tail -100 logs/error-$(date +%Y-%m-%d).log
```

**Search for Specific User:**
```bash
grep "M361549" logs/combined-$(date +%Y-%m-%d).log | tail -50
```

**Find All Errors Today:**
```bash
cat logs/error-$(date +%Y-%m-%d).log
```

**Count Errors by Message:**
```bash
grep "\[ERROR\]" logs/combined-$(date +%Y-%m-%d).log | cut -d']' -f3 | cut -d'{' -f1 | sort | uniq -c | sort -rn
```

**Find Slow Requests (>1000ms):**
```bash
grep -E "\"duration\":[0-9]{4,}" logs/combined-$(date +%Y-%m-%d).log
```

**Search for Specific Endpoint:**
```bash
grep "POST /api/products" logs/combined-$(date +%Y-%m-%d).log | tail -20
```

**View HTTP Status Distribution:**
```bash
grep "\[HTTP\]" logs/combined-$(date +%Y-%m-%d).log | grep -oE "\"status\":[0-9]+" | sort | uniq -c
```

### Log Analysis Script

```bash
#!/bin/bash
# log-stats.sh - Generate log statistics for today

DATE=$(date +%Y-%m-%d)
LOG_DIR="logs"

echo "=== Log Statistics for $DATE ==="
echo ""

if [ -f "$LOG_DIR/combined-$DATE.log" ]; then
  echo "Total log entries:"
  wc -l < "$LOG_DIR/combined-$DATE.log"

  echo ""
  echo "Entries by level:"
  grep -oE "\[(ERROR|WARN|INFO|HTTP|DEBUG)\]" "$LOG_DIR/combined-$DATE.log" | sort | uniq -c

  echo ""
  echo "Error count:"
  wc -l < "$LOG_DIR/error-$DATE.log" 2>/dev/null || echo "0"
else
  echo "No logs found for $DATE"
fi
```

---

## Metrics Endpoint

The application provides a metrics endpoint for monitoring system health.

### Endpoint Details

| Endpoint | Method | Authentication |
|----------|--------|----------------|
| `/api/metrics` | GET | `X-Metrics-Token` header |
| `/api/metrics/prometheus` | GET | `X-Metrics-Token` header |

### Authentication

Set the `METRICS_TOKEN` environment variable and include it in requests:

```bash
curl -H "X-Metrics-Token: your-token-here" http://localhost:5000/api/metrics
```

**Note:** In development mode (`NODE_ENV=development`), the metrics endpoint is accessible without authentication.

### JSON Response Format

```bash
curl http://localhost:5000/api/metrics
```

```json
{
  "timestamp": "2026-01-09T14:30:45.123Z",
  "system": {
    "nodeVersion": "v20.10.0",
    "platform": "linux",
    "arch": "x64",
    "pid": 12345,
    "uptime": 86400,
    "uptimeFormatted": "1d 0h 0m 0s"
  },
  "memory": {
    "heapUsedMB": 125,
    "heapTotalMB": 200,
    "rssMB": 250,
    "externalMB": 10,
    "heapUsagePercent": 62
  },
  "cpu": {
    "user": 1234567,
    "system": 234567,
    "totalMs": 1469
  },
  "database": {
    "connected": true,
    "readyState": 1,
    "readyStateDescription": "connected",
    "collections": 15,
    "host": "localhost"
  },
  "tickets": {
    "total": 1250,
    "byStatus": [
      {"_id": "DRAFT", "count": 45},
      {"_id": "SUBMITTED", "count": 120},
      {"_id": "IN_PROCESS", "count": 85},
      {"_id": "COMPLETED", "count": 1000}
    ],
    "byPriority": [
      {"_id": "LOW", "count": 200},
      {"_id": "MEDIUM", "count": 800},
      {"_id": "HIGH", "count": 250}
    ],
    "recentErrors": 0,
    "createdToday": 5
  },
  "performance": {
    "queryExecutionTime": 45
  }
}
```

### Prometheus Format

```bash
curl http://localhost:5000/api/metrics/prometheus
```

```
# HELP npdi_app_uptime_seconds Application uptime in seconds
# TYPE npdi_app_uptime_seconds gauge
npdi_app_uptime_seconds 86400

# HELP npdi_app_memory_heap_used_bytes Heap memory used in bytes
# TYPE npdi_app_memory_heap_used_bytes gauge
npdi_app_memory_heap_used_bytes 131072000

# HELP npdi_app_tickets_total Total number of tickets
# TYPE npdi_app_tickets_total gauge
npdi_app_tickets_total 1250

# HELP npdi_app_tickets_by_status Tickets grouped by status
# TYPE npdi_app_tickets_by_status gauge
npdi_app_tickets_by_status{status="DRAFT"} 45
npdi_app_tickets_by_status{status="SUBMITTED"} 120
npdi_app_tickets_by_status{status="COMPLETED"} 1000

# HELP npdi_app_database_connected Database connection status
# TYPE npdi_app_database_connected gauge
npdi_app_database_connected 1
```

### Key Metrics

| Metric | Description | Concern Threshold |
|--------|-------------|-------------------|
| `heapUsagePercent` | Memory usage | >80% |
| `database.connected` | Database status | `false` |
| `tickets.recentErrors` | Errors in last hour | >0 |
| `performance.queryExecutionTime` | API response time | >1000ms |

---

## Troubleshooting with Logs

### Debug High CPU/Memory

```bash
# Find resource-intensive operations (slow requests)
grep -E "\"duration\":[0-9]{4,}" logs/combined-$(date +%Y-%m-%d).log | tail -20

# Check for repeated rapid operations
grep "$(date +%Y-%m-%dT%H:%M)" logs/combined-$(date +%Y-%m-%d).log | wc -l
```

### Debug Database Issues

```bash
# Find database-related errors
grep -i "mongo\|database\|connection" logs/error-$(date +%Y-%m-%d).log
```

### Debug Authentication Issues

```bash
# Find auth failures
grep -i "unauthorized\|401\|forbidden\|403" logs/combined-$(date +%Y-%m-%d).log | tail -20

# Track specific user's requests
grep "M361549" logs/combined-$(date +%Y-%m-%d).log | tail -50
```

---

## Manual Log Cleanup

While the logger handles rotation automatically, you can manually clean up if needed:

```bash
# List log files by size
ls -lhS logs/

# Remove logs older than 30 days (handled automatically)
find logs/ -name "*.log" -mtime +30 -delete

# Check log directory size
du -sh logs/
```

---

**Last Updated:** 2026-01-09
**Maintained by:** NPDI Development Team
