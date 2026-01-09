# User Feedback & Bug Reporting System

## Overview

A floating feedback widget that allows users to report bugs, critical failures, and feature requests directly from any page in the application. All submissions are logged to a dedicated MongoDB collection and viewable by administrators through a new tab in the Admin Dashboard.

## User Experience

### Floating Feedback Button
- Fixed-position circular button in the bottom-right corner of all pages
- Visible on every page after authentication
- Subtle hover animation with tooltip: "Report Bug or Request Feature"
- High z-index to float above all content

### Feedback Submission Flow
1. User clicks the floating button
2. Modal opens with category selection:
   - 🐛 **BUG** - "Something isn't working correctly"
   - 🚨 **CRITICAL FAILURE** - "App crashed or data was lost"
   - 💡 **FEATURE REQUEST** - "Suggest an improvement"
3. User fills out:
   - Title (required, max 100 characters)
   - Description (required, min 20 characters)
4. On submit, technical context is automatically captured
5. Success toast with feedback reference number
6. Modal closes automatically

---

## Technical Architecture

### Data Model

**Collection:** `feedbacks`

**File:** `server/models/Feedback.js`

```javascript
{
  // Core feedback data
  feedbackId: String,           // Auto-generated: FB-YYYYMMDD-XXXX
  feedbackType: String,         // 'BUG' | 'CRITICAL_FAILURE' | 'FEATURE_REQUEST'
  title: String,                // Short summary (max 100 chars)
  description: String,          // Detailed description

  // Status tracking
  status: String,               // 'NEW' | 'IN_PROGRESS' | 'RESOLVED' | 'CLOSED' | 'WONT_FIX'
  priority: String,             // 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL'

  // User information (snapshot at submission time)
  submittedBy: {
    firstName: String,
    lastName: String,
    email: String,
    role: String,
    employeeId: String,
    sbu: String
  },

  // Automatic technical context
  technicalContext: {
    currentPage: String,        // Route/path when submitted
    fullUrl: String,            // Complete URL with query params
    userAgent: String,          // Browser and OS info
    screenResolution: String,   // "1920x1080"
    devicePixelRatio: Number,   // For retina detection
    platform: String,           // OS platform
    language: String,           // Browser language
    timezone: String,           // User's timezone
    isOnline: Boolean,          // Network status
    sessionDuration: Number,    // Seconds since login
    memoryInfo: String,         // Device memory if available
    connectionType: String,     // Network connection type
    appVersion: String          // Client version if available
  },

  // Error tracking
  clientErrors: [{
    message: String,            // Error message
    stack: String,              // Stack trace
    componentStack: String,     // React component stack
    timestamp: Date,            // When error occurred
    url: String                 // Page where error occurred
  }],

  // Server logs (populated on creation)
  serverLogs: [{
    timestamp: Date,
    level: String,              // ERROR, WARN, INFO, etc.
    message: String,
    metadata: Object
  }],

  // Admin management
  adminNotes: String,           // Internal notes from admin
  resolvedBy: {
    firstName: String,
    lastName: String,
    email: String
  },
  resolvedAt: Date,

  // Timestamps
  createdAt: Date,
  updatedAt: Date
}
```

### Priority Auto-Assignment
| Feedback Type | Default Priority |
|---------------|------------------|
| CRITICAL_FAILURE | CRITICAL |
| BUG | MEDIUM |
| FEATURE_REQUEST | LOW |

---

## API Endpoints

**Base Path:** `/api/feedback`

| Method | Endpoint | Access | Description |
|--------|----------|--------|-------------|
| `POST` | `/` | Authenticated | Submit new feedback |
| `GET` | `/` | Admin only | List all feedback (paginated) |
| `GET` | `/stats` | Admin only | Get feedback statistics |
| `GET` | `/:id` | Admin only | Get feedback details |
| `PATCH` | `/:id` | Admin only | Update status/notes |
| `DELETE` | `/:id` | Admin only | Delete feedback |

### Query Parameters for GET /

| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | Number | Page number (default: 1) |
| `limit` | Number | Items per page (default: 20) |
| `feedbackType` | String | Filter by type |
| `status` | String | Filter by status |
| `priority` | String | Filter by priority |
| `startDate` | Date | Filter from date |
| `endDate` | Date | Filter to date |
| `search` | String | Search in title/description |
| `sortBy` | String | Sort field (default: createdAt) |
| `sortOrder` | String | 'asc' or 'desc' (default: desc) |

---

## Client-Side Implementation

### Files to Create

| File | Description |
|------|-------------|
| `client/src/components/common/FeedbackButton.jsx` | Floating button component |
| `client/src/components/common/FeedbackModal.jsx` | Submission modal with form |
| `client/src/components/admin/FeedbackManagement.jsx` | Admin dashboard view |
| `client/src/utils/errorLogger.js` | Client-side error capture utility |

### Files to Modify

| File | Change |
|------|--------|
| `client/src/services/api.js` | Add `feedbackAPI` service |
| `client/src/components/Layout.jsx` | Add FeedbackButton to layout |
| `client/src/pages/AdminDashboard.jsx` | Add Feedback tab |
| `client/src/components/common/index.js` | Export new components |

### Error Logger Utility

Captures JavaScript errors globally for inclusion in feedback:

```javascript
// client/src/utils/errorLogger.js

class ErrorLogger {
  constructor(maxErrors = 20) {
    this.errors = [];
    this.maxErrors = maxErrors;
    this.sessionStart = Date.now();
    this.init();
  }

  init() {
    // Capture unhandled errors
    window.onerror = (message, source, lineno, colno, error) => {
      this.pushError({
        message,
        stack: error?.stack,
        source,
        lineno,
        colno,
        timestamp: new Date()
      });
    };

    // Capture unhandled promise rejections
    window.onunhandledrejection = (event) => {
      this.pushError({
        message: event.reason?.message || 'Unhandled Promise Rejection',
        stack: event.reason?.stack,
        timestamp: new Date()
      });
    };
  }

  pushError(error) {
    this.errors.push({
      ...error,
      url: window.location.href
    });
    if (this.errors.length > this.maxErrors) {
      this.errors.shift();
    }
  }

  getErrors() { return [...this.errors]; }
  clearErrors() { this.errors = []; }
  getSessionDuration() { return Math.floor((Date.now() - this.sessionStart) / 1000); }
}

export const errorLogger = new ErrorLogger();
```

### Technical Context Collection

```javascript
// Collected automatically on feedback submission
const collectTechnicalContext = () => ({
  currentPage: window.location.pathname,
  fullUrl: window.location.href,
  userAgent: navigator.userAgent,
  screenResolution: `${window.innerWidth}x${window.innerHeight}`,
  devicePixelRatio: window.devicePixelRatio,
  platform: navigator.platform,
  language: navigator.language,
  timezone: Intl.DateTimeFormat().resolvedOptions().timeZone,
  isOnline: navigator.onLine,
  sessionDuration: errorLogger.getSessionDuration(),
  memoryInfo: navigator.deviceMemory ? `${navigator.deviceMemory}GB` : 'N/A',
  connectionType: navigator.connection?.effectiveType || 'unknown'
});
```

---

## Server-Side Implementation

### Files to Create

| File | Description |
|------|-------------|
| `server/models/Feedback.js` | Mongoose model |
| `server/controllers/feedbackController.js` | Controller with CRUD operations |
| `server/routes/feedback.js` | Express routes |

### Files to Modify

| File | Change |
|------|--------|
| `server/models/index.js` | Export Feedback model |
| `server/index.js` | Register feedback routes |

### Server Log Retrieval

When feedback is submitted, the server retrieves recent log entries:

```javascript
// In feedbackController.js
const getRecentServerLogs = async (userEmail, limit = 50) => {
  const logDir = path.join(__dirname, '../../logs');
  const today = new Date().toISOString().split('T')[0];
  const logFile = path.join(logDir, `combined-${today}.log`);

  if (!fs.existsSync(logFile)) return [];

  const logs = [];
  const lines = fs.readFileSync(logFile, 'utf-8').split('\n').filter(Boolean);

  // Get last N lines, filter by user if email provided
  const relevantLines = lines.slice(-limit * 2).filter(line => {
    if (!userEmail) return true;
    return line.includes(userEmail) || line.includes('[ERROR]') || line.includes('[WARN]');
  }).slice(-limit);

  return relevantLines.map(line => {
    try {
      return JSON.parse(line);
    } catch {
      return { raw: line };
    }
  });
};
```

---

## Admin Dashboard Integration

### New Tab Configuration

```javascript
// In AdminDashboard.jsx tabs array
{
  id: 'feedback',
  name: 'User Feedback',
  icon: ChatBubbleBottomCenterTextIcon,
  badge: newFeedbackCount  // Shows count of NEW items
}
```

### FeedbackManagement Component Features

1. **Stats Dashboard**
   - Total feedback count
   - Count by status (NEW, IN_PROGRESS, RESOLVED, etc.)
   - Count by type (BUG, CRITICAL_FAILURE, FEATURE_REQUEST)
   - Recent submission trend

2. **Filterable Table**
   - Columns: ID, Type (icon), Title, User, Status, Priority, Created
   - Quick filters for type, status, priority
   - Date range picker
   - Search by user email or description
   - Sortable columns
   - Pagination

3. **Detail Modal**
   - Full feedback content
   - User information card
   - Technical context display
   - Collapsible client errors viewer with stack traces
   - Collapsible server logs viewer
   - Status update dropdown
   - Admin notes text area
   - Resolution button

---

## Security Considerations

1. **Authentication Required** - Only authenticated users can submit feedback
2. **Admin-Only Access** - Only admins can view all feedback
3. **User Data Snapshot** - User info is copied at submission time, not referenced
4. **Log Sanitization** - Sensitive data in logs should be filtered before storage
5. **Rate Limiting** - Consider adding rate limits to prevent spam
