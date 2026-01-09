# NPDI Application Dependencies

## Overview

This document provides a comprehensive inventory of external dependencies used in the NPDI Application, including their purpose, data access characteristics, and security features. This reference helps stakeholders understand how dependencies interact with application data and what protective mechanisms are in place.

**Document Date:** 2026-01-09
**Application:** MilliporeSigma NPDI Application
**Version:** 2.2

---

## 1. Dependency Inventory

### 1.1 Backend Dependencies (Production)

| Package | Version | Category | Purpose | Data Access |
|---------|---------|----------|---------|-------------|
| express | ^5.2.1 | Web Framework | HTTP server and routing | Request/Response handling only |
| mongoose | ^9.0.1 | Database ODM | MongoDB object modeling | Direct database access |
| axios | ^1.13.2 | HTTP Client | External API calls (PubChem, Palantir) | No database access |
| cors | ^2.8.5 | Security Middleware | Cross-origin resource sharing | No data access |
| helmet | ^8.1.0 | Security Middleware | HTTP security headers | No data access |
| express-rate-limit | ^8.2.1 | Security Middleware | API rate limiting | No data access |
| express-validator | ^7.3.1 | Validation | Input validation and sanitization | No data access |
| compression | ^1.8.1 | Performance | HTTP response compression | No data access |
| dotenv | ^17.2.3 | Configuration | Environment variable management | No data access |
| exceljs | ^4.4.0 | File Generation | Excel file creation (exports) | No database access |
| multer | ^2.0.2 | File Upload | Multipart form data handling | No database access |
| nodemailer | ^7.0.11 | Email | SMTP email notifications | No database access |
| apache-arrow | ^21.1.0 | Data Format | Arrow format for Palantir queries | No database access |
| csv-parser | ^3.2.0 | File Parsing | CSV file parsing | No database access |

### 1.2 Frontend Dependencies (Production)

| Package | Version | Category | Purpose | Data Access |
|---------|---------|----------|---------|-------------|
| react | ^19.2.1 | UI Framework | User interface rendering | No backend access |
| react-dom | ^19.2.1 | UI Framework | DOM rendering | No backend access |
| react-router-dom | ^7.10.1 | Routing | Client-side routing | No backend access |
| axios | ^1.13.2 | HTTP Client | API communication | API calls only |
| react-hook-form | ^7.68.0 | Form Management | Form state management | Client-side only |
| react-hot-toast | ^2.6.0 | UI Component | Toast notifications | Client-side only |
| @heroicons/react | ^2.2.0 | UI Assets | Icon library | Static assets only |
| react-markdown | ^10.1.0 | Content Rendering | Markdown to HTML rendering | Client-side only |
| remark-gfm | ^4.0.1 | Markdown Plugin | GitHub Flavored Markdown support | Client-side only |

### 1.3 Development Dependencies

| Package | Version | Category | Purpose | Production Deployment |
|---------|---------|----------|---------|----------------------|
| tailwindcss | ^4.1.17 | CSS Framework | Build-time CSS generation | Not deployed - build-time only |
| @tailwindcss/postcss | ^4.1.17 | CSS Processing | PostCSS plugin for Tailwind | Not deployed - build-time only |
| postcss | ^8.5.6 | CSS Processing | CSS transformations | Not deployed - build-time only |
| vite | ^7.2.6 | Build Tool | Frontend bundling and dev server | Not deployed |
| @vitejs/plugin-react | ^5.1.1 | Build Plugin | React support for Vite | Not deployed |
| nodemon | ^3.1.11 | Development | Auto-restart server on changes | Not deployed |
| eslint | ^9.39.1 | Code Quality | Static code analysis | Not deployed |
| concurrently | ^9.2.1 | Development | Run multiple dev commands | Not deployed |
| xlsx | ^0.18.5 | File Parsing | Excel file parsing (dev/import) | Not deployed |

### 1.4 Data Access Terminology

The "Data Access" column uses the following distinctions:

| Term | Meaning |
|------|---------|
| **Direct database access** | Reads from and writes to MongoDB |
| **No database access** | Does not interact with MongoDB, but may handle other data (files, external APIs, emails) |
| **No data access** | Does not interact with any application data; configuration or middleware only |
| **No backend access** | Frontend-only; cannot directly reach server or database |
| **Client-side only** | Executes in browser sandbox; no server interaction |
| **API calls only** | Communicates with backend via HTTP endpoints |

---

## 2. Database Access Layer: Mongoose

Mongoose is the only dependency with direct database access. The following mechanisms govern data interactions:

### 2.1 Schema Validation

```javascript
// Mongoose enforces strict schema validation at the application level
const productTicketSchema = new mongoose.Schema({
  ticketNumber: {
    type: String,
    unique: true,
    required: false
  },
  chemicalProperties: {
    casNumber: {
      type: String,
      required: true,
      match: /^\d{1,7}-\d{2}-\d$/  // CAS number format validation
    }
  }
});
```

**Validation Mechanisms:**
- Type validation before database writes
- Schema enforcement limits writable fields to those defined
- Custom validators for data entry rules
- Pre-save middleware hooks for data consistency

### 2.2 Query Pattern

```javascript
// Mongoose uses parameterized queries, not string concatenation
await ProductTicket.findOne({ ticketNumber: userInput });
// userInput is treated as data, not executable code
```

### 2.3 Data Sanitization
- MongoDB driver escapes special characters
- BSON encoding for data transmission
- No raw query string concatenation

### 2.4 Transaction Support
- MongoDB transactions for multi-document operations
- Atomic operations for data consistency
- Rollback capability on errors

### 2.5 Implementation in NPDI

**ProductTicket Model (`server/models/ProductTicket.js`):**
- Strict schema definitions with required fields
- Enum validation for status, priority, SBU fields
- RegEx pattern matching for CAS numbers
- Pre-save middleware for data consistency
- Automatic timestamp management
- Unique constraints on critical fields

**Controllers (`server/controllers/*.js`):**
- Input validation using express-validator
- Parameterized queries
- Error handling with try-catch blocks
- Sanitization of user inputs

---

## 3. HTTP Client: Axios

Axios handles external API communication without direct database access.

### 3.1 Characteristics
- Outbound HTTP requests only
- No direct database access
- Data persistence controlled by application code

### 3.2 Usage in NPDI
```javascript
// server/services/pubchemService.js
const enrichTicketData = async (casNumber) => {
  // Axios fetches external data
  const response = await axios.get(pubchemUrl);
  // Application code controls what gets saved
  return processedData;
};
```

### 3.3 Features
- Request/response interceptors for logging
- Timeout configuration
- SSL/TLS support for encrypted communication

---

## 4. Security Middleware

### 4.1 Express Framework
- Industry-standard web framework
- Routing and middleware functionality
- No direct data manipulation capabilities

**Weekly Downloads:** 50M+
**Publisher:** OpenJS Foundation

### 4.2 Helmet (Security Headers)
```javascript
app.use(helmet());
```

**Headers Applied:**
- Content Security Policy (XSS prevention)
- X-Frame-Options (Clickjacking prevention)
- X-Content-Type-Options (MIME sniffing prevention)
- Strict-Transport-Security (HTTPS enforcement)

### 4.3 CORS (Cross-Origin Resource Sharing)
```javascript
app.use(cors({
  origin: process.env.CLIENT_URL,
  credentials: true
}));
```

**Function:**
- Restricts API access to authorized origins
- Controls cross-origin requests

### 4.4 Express-Validator (Input Sanitization)
```javascript
const { validationResult } = require('express-validator');

const createTicket = async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  // Only validated data proceeds
};
```

**Function:**
- Input sanitization before database operations
- Type validation
- HTML escaping

### 4.5 Express-Rate-Limit
```javascript
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // requests per window
});
```

**Function:**
- Limits requests per IP address
- No data access or modification

---

## 5. Frontend Dependencies

### 5.1 Execution Environment
All frontend libraries execute in the browser sandbox with:
- No direct database connectivity
- No server-side code execution
- API access only through controlled endpoints

### 5.2 React (UI Framework)

**XSS Handling:**
```javascript
// React automatically escapes values in JSX
const userInput = '<script>alert("XSS")</script>';
<div>{userInput}</div>
// Renders as escaped text: &lt;script&gt;...
```

**Note:** `dangerouslySetInnerHTML` is not used in this application.

**Weekly Downloads:** 20M+
**Publisher:** Meta (Facebook)

### 5.3 Tailwind CSS

**Execution:** Build-time only

```javascript
// client/postcss.config.js - Build-time CSS processing
export default {
  plugins: {
    tailwindcss: {},  // Processes CSS during build
    autoprefixer: {}
  }
}
```

**Build Process:**
1. Developer writes HTML with Tailwind classes
2. Build tool (Vite) processes CSS at build time
3. PostCSS + Tailwind generate static CSS file
4. Generated CSS is bundled into production assets
5. No Tailwind code executes in production

**Production Output:**
```html
<!-- Static CSS file served -->
<link rel="stylesheet" href="/assets/index-abc123.css">

<!-- CSS contains only style declarations -->
.bg-blue-500 { background-color: rgb(59, 130, 246); }
.text-white { color: rgb(255, 255, 255); }
```

### 5.4 Form Libraries (react-hook-form)
- Client-side state management only
- No automatic data submission
- Server-side validation implemented separately

### 5.5 Icon Library (@heroicons/react)
- Static SVG icon components
- No data processing
- No network access

### 5.6 Markdown Rendering (react-markdown, remark-gfm)
- Converts markdown text to React components
- GitHub Flavored Markdown support (tables, task lists)
- Client-side rendering only
- No network access

---

## 6. Data Flow

### 6.1 Data Write Path

```
User Input (Browser)
    ↓
React Form (Client-side validation)
    ↓
HTTP POST via Axios (API call)
    ↓
Express Route Handler
    ↓
Express-Validator Middleware ← Validation checkpoint
    ↓
Controller Logic (cleanTicketData) ← Sanitization checkpoint
    ↓
Mongoose Schema Validation ← Schema checkpoint
    ↓
Mongoose Pre-save Middleware ← Consistency checkpoint
    ↓
MongoDB Write (BSON encoded, parameterized)
    ↓
Database (Persisted)
```

### 6.2 Data Read Path

```
User Request (Browser)
    ↓
React Component
    ↓
HTTP GET via Axios
    ↓
Express Route Handler
    ↓
Auth Middleware (API key validation) ← Authentication checkpoint
    ↓
Controller Logic (query parameters)
    ↓
Mongoose Query Builder (parameterized) ← Query checkpoint
    ↓
MongoDB Read
    ↓
JSON Response
    ↓
React Component Rendering
```

---

## 7. Application Security Implementation

### 7.1 Input Validation
```javascript
// server/controllers/productController.js
const createTicket = async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  ticketData = cleanTicketData(ticketData);
};
```

### 7.2 Data Sanitization
```javascript
// server/utils/enumCleaner.js
const cleanTicketData = (data) => {
  // Removes invalid enum values
  // Validates data structure
};
```

### 7.3 Error Handling
```javascript
try {
  await ticket.save();
} catch (error) {
  console.error('Create ticket error:', error);
  res.status(500).json({
    message: 'Server error'
    // Sensitive error details not exposed
  });
}
```

### 7.4 Environment Configuration
```javascript
// .env (not committed to repository)
MONGODB_URI=mongodb://localhost:27017/npdi
PORT=3000

// .gitignore includes .env
```

### 7.5 API Key Authentication
```javascript
// server/middleware/apiAuth.js
async function authenticateApiKey(req, res, next) {
  const apiKey = req.headers['x-api-key'];
  const keyHash = crypto.createHash('sha256').update(apiKey).digest('hex');
  const apiKeyRecord = await ApiKey.findOne({ keyHash });
  // Validation logic
}
```

---

## 8. Dependency Supply Chain

### 8.1 Package Statistics

| Package | Weekly Downloads | Publisher |
|---------|-----------------|-----------|
| Express | 50M+ | OpenJS Foundation |
| React | 20M+ | Meta (Facebook) |
| Mongoose | 3M+ | Automattic |
| Axios | 45M+ | Independent |

### 8.2 Version Management

```json
{
  "dependencies": {
    "express": "^5.2.1",
    "mongoose": "^9.0.1",
    "helmet": "^8.1.0"
  }
}
```

**Versioning Strategy:**
- Caret (^) allows patch updates
- Security fixes applied on `npm install`
- Major version changes require manual review

### 8.3 Production vs. Development Separation

```json
{
  "dependencies": {
    // Deployed to production
    "express": "^5.2.1",
    "mongoose": "^9.0.1"
  },
  "devDependencies": {
    // Not deployed to production
    "tailwindcss": "^4.1.17",
    "vite": "^7.2.6",
    "nodemon": "^3.1.11"
  }
}
```

**Production Deployment:**
```bash
npm install --production
# Installs only production dependencies
```

---

## 9. Dependency Summary by Category

| Category | Dependencies | Database Access | Production Execution |
|----------|-------------|-----------------|---------------------|
| Database ODM | Mongoose | Direct (controlled via schema) | Backend |
| Web Framework | Express | None | Backend |
| Security Middleware | Helmet, CORS, Rate-Limit, Validator | None | Backend |
| HTTP Client | Axios | None | Backend + Frontend |
| File Processing | ExcelJS, Multer, csv-parser, apache-arrow | None | Backend |
| Email | Nodemailer | None | Backend |
| UI Framework | React, React-DOM, React-Router-DOM | None | Frontend (browser) |
| UI Components | Heroicons, react-hot-toast | None | Frontend (browser) |
| Content Rendering | react-markdown, remark-gfm | None | Frontend (browser) |
| Form Management | react-hook-form | None | Frontend (browser) |
| CSS Framework | Tailwind CSS, PostCSS | None | Build-time only |
| Build Tools | Vite, @vitejs/plugin-react | None | Development only |
| Dev Tools | nodemon, eslint, concurrently | None | Development only |

---

## 10. References

### 10.1 Security Resources
- **OWASP Top 10:** https://owasp.org/www-project-top-ten/
- **Node.js Security:** https://nodejs.org/en/docs/guides/security/
- **Express Security:** https://expressjs.com/en/advanced/best-practice-security.html
- **MongoDB Security:** https://docs.mongodb.com/manual/administration/security-checklist/

### 10.2 Dependency Security Tools
- **npm audit:** Built-in security auditing
- **Snyk:** https://snyk.io
- **Dependabot:** https://github.com/dependabot
- **npm advisory database:** https://www.npmjs.com/advisories

### 10.3 Package Documentation
- **Mongoose:** https://mongoosejs.com/docs/validation.html
- **Helmet:** https://helmetjs.github.io/
- **Express-Validator:** https://express-validator.github.io/docs/
- **React:** https://reactjs.org/docs/

---

**Document Version:** 2.2
**Last Updated:** 2026-01-09
**Prepared By:** NPDI Development Team
