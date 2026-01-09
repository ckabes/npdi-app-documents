# Error Handling Reference

**Version:** 1.0.0
**Last Updated:** 2026-01-09

---

## Table of Contents

1. [Overview](#overview)
2. [API Error Response Format](#api-error-response-format)
3. [HTTP Status Codes](#http-status-codes)
4. [Error Types](#error-types)
5. [Client-Side Error Handling](#client-side-error-handling)
6. [Server-Side Error Handling](#server-side-error-handling)
7. [Validation Error Patterns](#validation-error-patterns)
8. [Integration Error Handling](#integration-error-handling)

---

## Overview

The NPDI application uses a consistent error handling pattern across both client and server. This document describes the error formats, status codes, and handling patterns used throughout the application.

### Error Handling Flow

```
Client Request
     ↓
┌─────────────────┐
│  Express Route  │
└────────┬────────┘
         ↓
┌─────────────────┐
│   Validation    │  ← express-validator
└────────┬────────┘
         ↓
┌─────────────────┐
│   Controller    │  ← try/catch with logger
└────────┬────────┘
         ↓
┌─────────────────┐
│  Error/Success  │  → JSON response
└─────────────────┘
         ↓
┌─────────────────┐
│  Client Handler │  ← handleApiError utility
└────────┬────────┘
         ↓
┌─────────────────┐
│   Toast/UI      │  ← User feedback
└─────────────────┘
```

---

## API Error Response Format

### Standard Error Response

```json
{
  "success": false,
  "message": "Human-readable error message",
  "error": "Error type identifier (optional)"
}
```

### Validation Error Response

```json
{
  "success": false,
  "message": "Validation failed",
  "validationErrors": [
    "CAS Number must be in format: 123-45-6 (Field: chemicalProperties.casNumber)",
    "SKU type must be one of: BULK, CONF, SPEC, VAR, PREPACK (Field: skuVariants.0.type)"
  ]
}
```

### Submission Requirements Error Response

```json
{
  "success": false,
  "error": "Submission Requirements Not Met",
  "message": "The following required fields are missing values",
  "missingFields": [
    {
      "fieldKey": "productName",
      "fieldLabel": "Product Name",
      "sectionKey": "basic"
    },
    {
      "fieldKey": "chemicalProperties.casNumber",
      "fieldLabel": "CAS Number",
      "sectionKey": "chemicalProperties"
    }
  ]
}
```

### Success Response

```json
{
  "success": true,
  "data": { /* result data */ },
  "message": "Operation completed successfully"
}
```

---

## HTTP Status Codes

### Success Codes

| Code | Usage |
|------|-------|
| `200 OK` | Successful GET, PUT, PATCH requests |
| `201 Created` | Successful POST that creates a resource |

### Client Error Codes

| Code | Usage | Example |
|------|-------|---------|
| `400 Bad Request` | Validation errors, malformed requests | Invalid CAS number format |
| `401 Unauthorized` | Missing or invalid authentication | No API key provided |
| `403 Forbidden` | User lacks permission | PM trying to access admin panel |
| `404 Not Found` | Resource doesn't exist | Ticket not found |
| `409 Conflict` | Duplicate or conflict | Duplicate ticket number |
| `422 Unprocessable Entity` | Business rule violation | Submitting incomplete ticket |

### Server Error Codes

| Code | Usage | Example |
|------|-------|---------|
| `500 Internal Server Error` | Unexpected server errors | Database connection failed |
| `502 Bad Gateway` | External service error | PubChem API unavailable |
| `503 Service Unavailable` | Service temporarily down | Maintenance mode |
| `504 Gateway Timeout` | External service timeout | Palantir query timeout |

---

## Error Types

### Validation Errors (`400`)

**Trigger:** Request data fails express-validator checks

**Response:**
```json
{
  "message": "Validation failed",
  "validationErrors": [
    "CAS Number must be in format: 123-45-6 (Field: chemicalProperties.casNumber)"
  ]
}
```

**Common Validations:**
- CAS number format: `/^\d{1,7}-\d{2}-\d$/`
- SKU types: `BULK`, `CONF`, `SPEC`, `VAR`, `PREPACK`
- Package units: `mg`, `g`, `kg`, `mL`, `L`, `units`, `vials`, `plates`, `bulk`
- GHS classes: `H200-H299`, `H300-H399`, `H400-H499`
- Signal words: `WARNING`, `DANGER`

### Submission Requirement Errors (`400`)

**Trigger:** Attempting to submit a ticket without required fields

**Response:**
```json
{
  "error": "Submission Requirements Not Met",
  "missingFields": [
    { "fieldKey": "productName", "fieldLabel": "Product Name" }
  ]
}
```

### Authentication Errors (`401`)

**Trigger:** Missing or invalid credentials

**Scenarios:**
- Missing `X-API-Key` header for external API
- Expired API key
- Invalid API key hash

**Response:**
```json
{
  "message": "API key is required",
  "error": "Unauthorized"
}
```

### Permission Errors (`403`)

**Trigger:** User lacks required role/permission

**Scenarios:**
- Product Manager trying to assign SKUs
- Non-admin accessing admin panel
- User trying to edit another user's DRAFT ticket

**Response:**
```json
{
  "message": "Permission denied",
  "error": "Forbidden"
}
```

### Not Found Errors (`404`)

**Trigger:** Requested resource doesn't exist

**Response:**
```json
{
  "message": "Ticket not found",
  "error": "Not Found"
}
```

### Database Errors (`500`)

**Trigger:** MongoDB operation failures

**Logged Details:**
```javascript
logger.error('Database error', {
  error: error.message,
  stack: error.stack,
  operation: 'findById',
  collection: 'producttickets',
  id: ticketId
});
```

**Response:**
```json
{
  "message": "Internal server error"
}
```

### External Service Errors (`502`/`504`)

**Trigger:** PubChem, Palantir, or other integrations fail

**Response:**
```json
{
  "message": "External service unavailable",
  "service": "pubchem",
  "error": "Connection timeout"
}
```

---

## Client-Side Error Handling

### handleApiError Utility

**Location:** `client/src/utils/apiErrorHandler.js`

**Usage:**
```javascript
import { handleApiError } from '../utils/apiErrorHandler';

try {
  await productAPI.createTicket(data);
} catch (error) {
  handleApiError(error, {
    defaultMessage: 'Failed to create ticket',
    onMissingFields: (missingFields) => {
      setMissingRequiredFields(missingFields.map(f => f.fieldKey));
    },
    onValidationError: (fieldKeys) => {
      highlightFields(fieldKeys);
    }
  });
}
```

**Options:**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `defaultMessage` | string | "An error occurred" | Fallback error message |
| `showToast` | boolean | true | Show toast notification |
| `toastDuration` | number | 5000 | Toast display time (ms) |
| `onValidationError` | function | - | Callback with field keys |
| `onMissingFields` | function | - | Callback with missing fields |

**Return Value:**
```javascript
{
  type: 'missing_fields' | 'validation' | 'api_error' | 'unknown',
  message: string,
  fieldKeys: string[],
  validationErrors: string[],
  missingFields: object[]
}
```

### Simple Error Display

```javascript
import { showApiError } from '../utils/apiErrorHandler';

try {
  await productAPI.deleteTicket(id);
} catch (error) {
  showApiError(error, 'Failed to delete ticket');
}
```

### Toast Notifications

```javascript
import toast from 'react-hot-toast';

// Success
toast.success('Ticket created successfully');

// Error
toast.error('Failed to save changes');

// With options
toast.error('Missing required fields', { duration: 7000 });
```

---

## Server-Side Error Handling

### Controller Pattern

```javascript
const logger = require('../utils/logger');

const getTicket = async (req, res) => {
  try {
    const { id } = req.params;

    const ticket = await ProductTicket.findById(id);

    if (!ticket) {
      return res.status(404).json({
        success: false,
        message: 'Ticket not found'
      });
    }

    res.json({ success: true, data: ticket });

  } catch (error) {
    logger.error('Error fetching ticket', {
      error: error.message,
      stack: error.stack,
      ticketId: req.params.id,
      userId: req.headers['x-user-employee-id']
    });

    res.status(500).json({
      success: false,
      message: 'Internal server error'
    });
  }
};
```

### Validation Error Handling

```javascript
const { validationResult } = require('express-validator');

const createTicket = async (req, res) => {
  // Check validation results
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({
      success: false,
      message: 'Validation failed',
      validationErrors: errors.array().map(e => e.msg)
    });
  }

  // ... proceed with creation
};
```

### Submission Requirements Check

```javascript
const checkSubmissionRequirements = (ticket, template) => {
  const missingFields = [];

  for (const fieldKey of template.submissionRequirements) {
    const value = getFieldValue(ticket, fieldKey);
    if (!value || value === '') {
      missingFields.push({
        fieldKey,
        fieldLabel: getFieldLabel(template, fieldKey),
        sectionKey: getFieldSection(template, fieldKey)
      });
    }
  }

  return missingFields;
};

// In controller
if (newStatus === 'SUBMITTED') {
  const missingFields = checkSubmissionRequirements(ticket, template);
  if (missingFields.length > 0) {
    return res.status(400).json({
      success: false,
      error: 'Submission Requirements Not Met',
      message: 'The following required fields are missing values',
      missingFields
    });
  }
}
```

---

## Validation Error Patterns

### Field Path Notation

Validation errors reference fields using dot notation:

| Path | Field |
|------|-------|
| `productName` | Top-level field |
| `chemicalProperties.casNumber` | Nested object field |
| `skuVariants.0.type` | Array element field |
| `composition.components.0.casNumber` | Deeply nested |

### Common Validation Messages

| Field | Validation | Message |
|-------|------------|---------|
| CAS Number | Format check | "CAS Number must be in format: 123-45-6" |
| SKU Type | Enum check | "SKU type must be one of: BULK, CONF, SPEC, VAR, PREPACK" |
| Package Unit | Enum check | "Package size unit must be one of: mg, g, kg, mL, L..." |
| GHS Class | Enum check | "GHS class must be one of: H200-H299, H300-H399, H400-H499" |
| List Price | Number check | "List price must be a number" |

---

## Integration Error Handling

### PubChem Integration

```javascript
const lookupCAS = async (casNumber) => {
  try {
    const response = await axios.get(
      `https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/name/${casNumber}/JSON`,
      { timeout: 30000 }
    );
    return response.data;
  } catch (error) {
    if (error.code === 'ECONNABORTED') {
      throw new Error('PubChem request timed out');
    }
    if (error.response?.status === 404) {
      return null; // CAS not found
    }
    throw new Error('PubChem service unavailable');
  }
};
```

### Palantir Integration

```javascript
const queryPalantir = async (query) => {
  try {
    const response = await palantirClient.query(query);
    return response.data;
  } catch (error) {
    logger.error('Palantir query failed', {
      error: error.message,
      query,
      connectionStatus: 'failed'
    });

    // Update connection status in settings
    await SystemSettings.updateOne({}, {
      'integrations.palantir.connectionStatus': 'failed',
      'integrations.palantir.lastConnectionTest': new Date()
    });

    throw new Error('Palantir connection failed');
  }
};
```

### AI Content Generation

```javascript
const generateContent = async (productData, promptType) => {
  try {
    const response = await aiClient.generate(productData, promptType);
    return response;
  } catch (error) {
    if (error.response?.status === 429) {
      logger.warn('AI rate limit exceeded');
      throw new Error('AI service rate limit exceeded. Please try again later.');
    }
    if (error.response?.status === 401) {
      logger.error('AI authentication failed - check API key');
      throw new Error('AI service authentication failed');
    }
    throw new Error('AI content generation failed');
  }
};
```

---

## Error Logging

### Log Levels

| Level | Usage |
|-------|-------|
| `error` | Exceptions, failures requiring attention |
| `warn` | Non-critical issues, deprecations |
| `info` | Normal operations, state changes |
| `http` | HTTP request/response logging |
| `debug` | Development debugging |

### Error Log Format

```javascript
logger.error('Operation failed', {
  error: error.message,
  stack: error.stack,
  userId: req.headers['x-user-employee-id'],
  endpoint: req.originalUrl,
  method: req.method,
  body: sanitizeBody(req.body), // Remove sensitive data
  timestamp: new Date().toISOString()
});
```

---

**Last Updated:** 2026-01-09
**Maintained by:** NPDI Development Team
