# Environment Variables Reference

**Version:** 1.1.0
**Last Updated:** 2026-01-09

---

## Table of Contents

1. [Overview](#overview)
2. [Quick Start](#quick-start)
3. [Server Environment Variables](#server-environment-variables)
4. [Client Environment Variables](#client-environment-variables)
5. [Security Considerations](#security-considerations)
6. [Environment-Specific Configuration](#environment-specific-configuration)

---

## Overview

The NPDI application uses environment variables for configuration. These are loaded from `.env` files using the `dotenv` package.

### File Locations

| File | Purpose |
|------|---------|
| `.env` | Active environment configuration (not committed to git) |
| `.env.example` | Template with all available variables (committed to git) |

### Creating Your Environment File

```bash
# Copy the example file
cp .env.example .env

# Edit with your values
nano .env
```

---

## Quick Start

Minimum required variables for local development:

```bash
# Required
MONGODB_URI=mongodb://localhost:27017/npdi-app
CLIENT_URL=http://localhost:5173

# Recommended for Security
ENCRYPTION_KEY=<generate with: node server/scripts/generateEncryptionKey.js>
```

---

## Server Environment Variables

### Database Configuration

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `MONGODB_URI` | **Yes** | - | MongoDB connection string |

**Example Values:**

```bash
# Local development
MONGODB_URI=mongodb://localhost:27017/npdi-app

# With authentication
MONGODB_URI=mongodb://username:password@localhost:27017/npdi-app

# MongoDB Atlas (production)
MONGODB_URI=mongodb+srv://user:pass@cluster.mongodb.net/npdi-app?retryWrites=true&w=majority
```

---

### Server Configuration

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `PORT` | No | `5000` | HTTP server port |
| `NODE_ENV` | No | `development` | Environment: `development`, `production`, `test` |
| `CLIENT_URL` | **Yes** | - | Frontend URL for CORS configuration |

**Example Values:**

```bash
PORT=5000
NODE_ENV=development
CLIENT_URL=http://localhost:5173

# Production
NODE_ENV=production
CLIENT_URL=https://npdi.company.com
```

**Effects of `NODE_ENV`:**

| Value | Behavior |
|-------|----------|
| `development` | Verbose logging, metrics endpoint accessible without token |
| `production` | Strict security, METRICS_TOKEN required for metrics endpoint |
| `test` | Test-specific behaviors |

---

### Security Configuration

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `ENCRYPTION_KEY` | **Recommended** | Fallback key | 64-character hex string for AES-256-GCM encryption |

**Generating an Encryption Key:**

```bash
# Using the provided script
node server/scripts/generateEncryptionKey.js

# Or using OpenSSL
openssl rand -hex 32
```

**Example:**

```bash
ENCRYPTION_KEY=a1b2c3d4e5f6...  # 64 hex characters
```

**What Gets Encrypted:**
- Azure OpenAI (LangDock) API keys in SystemSettings
- Palantir Foundry tokens in SystemSettings

**Warning:** Without `ENCRYPTION_KEY`, a fallback key is used with a console warning. This is NOT secure for production.

---

### Logging Configuration

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `LOG_LEVEL` | No | `info` | Minimum log level to record |

**Log Levels (in order of severity):**

```bash
LOG_LEVEL=error   # Only errors
LOG_LEVEL=warn    # Warnings and above
LOG_LEVEL=info    # Informational and above (default)
LOG_LEVEL=http    # HTTP requests and above
LOG_LEVEL=debug   # All logs including debug
```

---

### Metrics Endpoint Authentication

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `METRICS_TOKEN` | No | - | Token for `/api/metrics` endpoint authentication |

**Generating a Metrics Token:**

```bash
openssl rand -hex 32
```

**Example:**

```bash
METRICS_TOKEN=abc123def456...
```

**Note:** In development mode, the metrics endpoint is accessible without a token. In production, `METRICS_TOKEN` must be set.

---

### Integration Configuration

Integrations (Teams webhooks, Palantir, Azure OpenAI) are configured via the **Admin UI** in SystemSettings, not environment variables. This allows runtime configuration without redeployment.

**API Keys:** External API keys for third-party applications are stored in the database. Generate them with:

```bash
node server/scripts/generateApiKey.js "Application Name"
```

---

## Client Environment Variables

Client-side environment variables are prefixed with `VITE_` and exposed to the browser.

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `VITE_API_URL` | No | Auto-detected | Backend API URL |
| `VITE_APP_VERSION` | No | From package.json | Application version |
| `VITE_BUILD_TIME` | No | Build time | Build timestamp |

**Location:** `client/.env` or `client/.env.local`

**Example:**

```bash
VITE_API_URL=http://localhost:5000
VITE_APP_VERSION=1.3.0
```

**Accessing in Code:**

```javascript
const apiUrl = import.meta.env.VITE_API_URL;
const version = import.meta.env.VITE_APP_VERSION;
const isDev = import.meta.env.DEV;  // true in development
```

**Security Warning:** Never put secrets in `VITE_` variables - they are exposed to the browser!

---

## Security Considerations

### Sensitive Variables

These variables contain secrets and must NEVER be committed to git:

| Variable | Risk if Exposed |
|----------|----------------|
| `MONGODB_URI` (with credentials) | Database access |
| `ENCRYPTION_KEY` | Decrypt stored secrets |
| `METRICS_TOKEN` | Access system metrics |

### Best Practices

1. **Never commit `.env` files**
   ```bash
   # .gitignore should include:
   .env
   .env.local
   .env.*.local
   ```

2. **Use different values per environment**
   - Development: Local values, can use fallbacks
   - Staging: Production-like but separate credentials
   - Production: Unique, rotated credentials

3. **Rotate secrets regularly**
   - Generate new `ENCRYPTION_KEY` periodically
   - Rotate API keys and tokens

4. **Use environment-specific files**
   ```
   .env.development   # Local development
   .env.production    # Production (don't commit)
   .env.test          # Testing
   ```

---

## Environment-Specific Configuration

### Development

```bash
# .env.development
MONGODB_URI=mongodb://localhost:27017/npdi-app-dev
PORT=5000
NODE_ENV=development
CLIENT_URL=http://localhost:5173
LOG_LEVEL=debug
# ENCRYPTION_KEY not required (uses fallback with warning)
```

### Production

```bash
# .env.production (DO NOT COMMIT)
MONGODB_URI=mongodb+srv://prod-user:secure-password@cluster.mongodb.net/npdi-app?retryWrites=true&w=majority
PORT=5000
NODE_ENV=production
CLIENT_URL=https://npdi.company.com
LOG_LEVEL=info

# Required for production
ENCRYPTION_KEY=<64-character-hex-string>

# Recommended for production
METRICS_TOKEN=<32-character-hex-string>
```

### Validation

The application validates critical environment variables at startup:

| Variable | Validation |
|----------|------------|
| `MONGODB_URI` | Must be present and valid MongoDB URI |
| `ENCRYPTION_KEY` | Warns if missing (uses fallback) |
| `NODE_ENV` | Logs current environment |

---

## Complete Variable Summary

| Variable | Required | Default | Used By |
|----------|----------|---------|---------|
| `MONGODB_URI` | **Yes** | - | Server |
| `PORT` | No | `5000` | Server |
| `NODE_ENV` | No | `development` | Server |
| `CLIENT_URL` | **Yes** | - | Server (CORS) |
| `LOG_LEVEL` | No | `info` | Server |
| `ENCRYPTION_KEY` | Recommended | Fallback | Server |
| `METRICS_TOKEN` | No | - | Server |
| `VITE_API_URL` | No | Auto | Client |
| `VITE_APP_VERSION` | No | package.json | Client |
| `VITE_BUILD_TIME` | No | Build time | Client |

---

**Last Updated:** 2026-01-09
**Maintained by:** NPDI Development Team
