# Coding Standards & Conventions

**Version:** 1.0.0
**Last Updated:** 2026-01-09

---

## Table of Contents

1. [Overview](#overview)
2. [Project Structure](#project-structure)
3. [Naming Conventions](#naming-conventions)
4. [JavaScript/Node.js Standards](#javascriptnodejs-standards)
5. [React Standards](#react-standards)
6. [API Design Standards](#api-design-standards)
7. [Database Standards](#database-standards)
8. [Error Handling Standards](#error-handling-standards)
9. [Documentation Standards](#documentation-standards)
10. [Git Workflow](#git-workflow)

---

## Overview

This document defines the coding standards and conventions used in the NPDI application. Following these standards ensures consistency, maintainability, and readability across the codebase.

### Tech Stack Summary

| Layer | Technology |
|-------|------------|
| Frontend | React 18, Vite, Tailwind CSS |
| Backend | Node.js, Express.js 5.x |
| Database | MongoDB with Mongoose ODM |
| Validation | express-validator |
| HTTP Client | Axios |

---

## Project Structure

### Root Directory

```
npdi-app/
├── client/                    # React frontend application
├── server/                    # Node.js backend application
├── logs/                      # Application log files
├── docs/                      # Documentation (synced to npdi-app-documents)
├── .env                       # Environment configuration (not in git)
├── .env.example               # Environment template
├── package.json               # Root package (server dependencies)
└── README.md                  # Project overview
```

### Server Structure

```
server/
├── config/                    # Configuration files
│   └── sapMappings/           # SAP field mapping configurations
├── controllers/               # Request handlers
│   └── products/              # Modular product controllers
├── data/                      # Static data files (JSON)
├── middleware/                # Express middleware
├── models/                    # Mongoose schemas
│   ├── discriminators/        # Template-specific ticket schemas
│   └── schemas/               # Reusable embedded schemas
├── routes/                    # Express route definitions
├── scripts/                   # CLI scripts (seeding, migrations)
├── services/                  # Business logic services
│   └── sapExport/             # SAP loader generation
├── utils/                     # Utility functions
└── index.js                   # Application entry point
```

### Client Structure

```
client/
├── src/
│   ├── components/            # React components
│   │   ├── admin/             # Admin panel components
│   │   ├── badges/            # Status/role badge components
│   │   ├── collaborations/    # Collaboration feature components
│   │   ├── common/            # Shared UI components
│   │   ├── dashboard/         # Dashboard-specific components
│   │   ├── forms/             # Form components
│   │   ├── modals/            # Modal dialogs
│   │   └── preview/           # Preview components
│   ├── context/               # React context providers
│   ├── data/                  # Static data (JSON)
│   ├── hooks/                 # Custom React hooks
│   ├── pages/                 # Page-level components
│   ├── services/              # API service layer
│   ├── styles/                # CSS files
│   └── utils/                 # Utility functions
├── public/                    # Static assets
└── index.html                 # HTML entry point
```

---

## Naming Conventions

### Files and Directories

| Type | Convention | Example |
|------|------------|---------|
| React Components | PascalCase.jsx | `ProductTicketForm.jsx` |
| Pages | PascalCase.jsx | `TicketDetails.jsx` |
| Hooks | camelCase.js with `use` prefix | `useFormCache.js` |
| Services | camelCase.js | `api.js`, `parserConfigService.js` |
| Utils | camelCase.js | `dateFormatters.js`, `pricingCalculations.js` |
| Controllers | camelCase + Controller.js | `adminController.js` |
| Routes | camelCase.js (plural nouns) | `products.js`, `users.js` |
| Models | PascalCase.js | `ProductTicket.js`, `User.js` |
| Schemas | camelCase + Schema.js | `skuVariantSchema.js` |
| Scripts | camelCase.js | `seedFormConfig.js` |

### Variables and Functions

| Type | Convention | Example |
|------|------------|---------|
| Variables | camelCase | `ticketNumber`, `casNumber` |
| Constants | UPPER_SNAKE_CASE | `MAX_FILE_SIZE`, `API_TIMEOUT` |
| Functions | camelCase (verb prefix) | `getTickets`, `createUser`, `handleSubmit` |
| Boolean variables | is/has/can prefix | `isActive`, `hasError`, `canEdit` |
| Event handlers | handle prefix | `handleClick`, `handleSubmit` |
| Async functions | Descriptive names | `fetchUserData`, `loadTickets` |

### React Components

| Type | Convention | Example |
|------|------------|---------|
| Component names | PascalCase | `ProductTicketForm` |
| Props | camelCase | `onSubmit`, `isLoading`, `ticketData` |
| State variables | camelCase | `loading`, `error`, `formData` |
| Context | PascalCase + Context | `AuthContext` |
| Hooks | use + PascalCase | `useAuth`, `useFormCache` |

### Database

| Type | Convention | Example |
|------|------------|---------|
| Collections | lowercase, plural | `producttickets`, `users` |
| Fields | camelCase | `ticketNumber`, `createdAt` |
| Enum values | UPPER_SNAKE_CASE | `DRAFT`, `IN_PROCESS` |
| Index names | field1_field2_1/-1 | `status_updatedAt_-1` |

---

## JavaScript/Node.js Standards

### Module Imports

```javascript
// 1. Node.js built-in modules
const fs = require('fs');
const path = require('path');

// 2. Third-party modules
const express = require('express');
const mongoose = require('mongoose');

// 3. Local modules
const ProductTicket = require('../models/ProductTicket');
const logger = require('../utils/logger');
```

### Function Definitions

```javascript
// Prefer arrow functions for callbacks
const getTickets = async (req, res) => {
  try {
    const tickets = await ProductTicket.find();
    res.json(tickets);
  } catch (error) {
    logger.error('Error fetching tickets', { error: error.message });
    res.status(500).json({ message: 'Server error' });
  }
};

// Export functions at the end of the file
module.exports = {
  getTickets,
  createTicket,
  updateTicket
};
```

### Async/Await Pattern

```javascript
// Always use try/catch with async/await
const processData = async () => {
  try {
    const result = await someAsyncOperation();
    return result;
  } catch (error) {
    logger.error('Operation failed', { error: error.message });
    throw error;  // Re-throw if caller should handle
  }
};
```

### Object Destructuring

```javascript
// Destructure in function parameters
const createTicket = async (req, res) => {
  const { productName, sbu, chemicalProperties } = req.body;
  // ...
};

// Destructure from objects
const { firstName, lastName, email } = user;
```

---

## React Standards

### Component Structure

```jsx
import React, { useState, useEffect, useMemo } from 'react';
import { useNavigate } from 'react-router-dom';
import { productAPI } from '../../services/api';
import toast from 'react-hot-toast';

/**
 * Component description
 * @param {Object} props - Component props
 * @param {string} props.ticketId - The ticket ID
 * @param {Function} props.onSave - Save callback
 */
const MyComponent = ({
  ticketId,
  onSave,
  isLoading = false,  // Default values in destructuring
  mode = 'view'
}) => {
  // 1. Hooks first
  const navigate = useNavigate();
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);

  // 2. Computed/memoized values
  const processedData = useMemo(() => {
    return data ? processData(data) : null;
  }, [data]);

  // 3. Effects
  useEffect(() => {
    fetchData();
  }, [ticketId]);

  // 4. Event handlers
  const handleSubmit = async (e) => {
    e.preventDefault();
    // ...
  };

  // 5. Render helpers (if needed)
  const renderContent = () => {
    if (loading) return <Loading />;
    return <div>{/* content */}</div>;
  };

  // 6. Return JSX
  return (
    <div className="container">
      {renderContent()}
    </div>
  );
};

export default MyComponent;
```

### Props Documentation

```jsx
/**
 * ProductTicketForm - Shared form component for creating and editing tickets
 *
 * @param {Object} props
 * @param {string} props.mode - 'create' or 'edit'
 * @param {Object} props.ticket - Ticket data for edit mode
 * @param {Object} props.template - Form template configuration
 * @param {Function} props.onSubmit - Submit handler
 * @param {Function} props.onCancel - Cancel handler
 */
```

### Custom Hooks

```javascript
// hooks/useFormCache.js
import { useState, useEffect } from 'react';

export const useFormCache = (key, initialValue) => {
  const [value, setValue] = useState(() => {
    const cached = localStorage.getItem(key);
    return cached ? JSON.parse(cached) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
};
```

### Context Pattern

```jsx
// context/AuthContext.jsx
import React, { createContext, useContext, useState } from 'react';

const AuthContext = createContext();

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);

  const value = {
    user,
    login: (userData) => setUser(userData),
    logout: () => setUser(null)
  };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
};
```

---

## API Design Standards

### Route Structure

```javascript
// RESTful route patterns
router.get('/products', controller.getAll);           // List
router.get('/products/:id', controller.getById);      // Get one
router.post('/products', controller.create);          // Create
router.put('/products/:id', controller.update);       // Full update
router.patch('/products/:id/status', controller.updateStatus);  // Partial update
router.delete('/products/:id', controller.delete);    // Delete

// Nested resources
router.post('/products/:id/comments', controller.addComment);
router.get('/products/:id/collaborators', controller.getCollaborators);
```

### Request Validation

```javascript
// Use express-validator for input validation
const { body, param, query } = require('express-validator');

router.post('/products', [
  body('productName').notEmpty().trim().withMessage('Product name is required'),
  body('sbu').notEmpty().trim().withMessage('SBU is required'),
  body('chemicalProperties.casNumber')
    .optional()
    .custom(value => {
      if (!value || value === '') return true;
      return /^\d{1,7}-\d{2}-\d$/.test(value);
    })
    .withMessage('Invalid CAS number format')
], controller.createTicket);
```

### Response Format

```javascript
// Success response
res.json({
  success: true,
  data: result,
  message: 'Operation completed successfully'
});

// Paginated response
res.json({
  data: items,
  pagination: {
    page: currentPage,
    limit: pageSize,
    total: totalCount,
    pages: Math.ceil(totalCount / pageSize)
  }
});

// Error response
res.status(400).json({
  success: false,
  message: 'Validation failed',
  errors: validationErrors
});
```

### API Service Layer (Client)

```javascript
// services/api.js
export const productAPI = {
  getTickets: (params) => apiClient.get('/products', { params }),
  getTicket: (id) => apiClient.get(`/products/${id}`),
  createTicket: (data) => apiClient.post('/products', data),
  updateTicket: (id, data) => apiClient.put(`/products/${id}`, data),
  updateStatus: (id, data) => apiClient.patch(`/products/${id}/status`, data)
};
```

---

## Database Standards

### Schema Definitions

```javascript
const mongoose = require('mongoose');

const ticketSchema = new mongoose.Schema({
  // Use descriptive field names
  ticketNumber: {
    type: String,
    unique: true,
    required: false
  },

  // Include enum validation
  status: {
    type: String,
    enum: ['DRAFT', 'SUBMITTED', 'IN_PROCESS', 'COMPLETED', 'CANCELED'],
    default: 'DRAFT'
  },

  // Use references for relationships
  createdByUser: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  },

  // Embed related data when appropriate
  chemicalProperties: chemicalPropertiesSchema,

  // Always include timestamps
  createdAt: {
    type: Date,
    default: Date.now
  },
  updatedAt: {
    type: Date,
    default: Date.now
  }
});

// Add indexes for query performance
ticketSchema.index({ status: 1, updatedAt: -1 });
ticketSchema.index({ createdBy: 1, status: 1 });
```

### Query Patterns

```javascript
// Use lean() for read-only queries
const tickets = await ProductTicket.find({ status: 'DRAFT' }).lean();

// Use select() to limit fields
const summary = await ProductTicket.find()
  .select('ticketNumber productName status createdAt')
  .lean();

// Use populate() for references
const ticket = await ProductTicket.findById(id)
  .populate('createdByUser', 'firstName lastName email')
  .populate('template');
```

---

## Error Handling Standards

### Server-Side Error Handling

```javascript
const logger = require('../utils/logger');

const controller = async (req, res) => {
  try {
    const result = await someOperation();
    res.json(result);
  } catch (error) {
    // Log with context
    logger.error('Operation failed', {
      error: error.message,
      stack: error.stack,
      userId: req.headers['x-user-employee-id'],
      endpoint: req.originalUrl
    });

    // Return appropriate status code
    if (error.name === 'ValidationError') {
      return res.status(400).json({ message: error.message });
    }

    res.status(500).json({ message: 'Internal server error' });
  }
};
```

### Client-Side Error Handling

```javascript
// API calls with error handling
try {
  const response = await productAPI.createTicket(data);
  toast.success('Ticket created successfully');
  navigate(`/tickets/${response.data._id}`);
} catch (error) {
  const message = error.response?.data?.message || 'An error occurred';
  toast.error(message);
  console.error('Create ticket error:', error);
}
```

---

## Documentation Standards

### Code Comments

```javascript
/**
 * Calculate the list price based on standard cost and margin
 * @param {number} standardCost - The standard cost per unit
 * @param {number} margin - The target margin percentage (0-100)
 * @returns {number} The calculated list price
 */
const calculateListPrice = (standardCost, margin) => {
  return standardCost / (1 - margin / 100);
};

// Use inline comments for non-obvious logic
const normalizedUnit = unit.toLowerCase();
// Special case: mL and L need capital L for consistency
if (normalizedUnit === 'ml') return 'mL';
```

### File Headers

```javascript
/**
 * Product Controller
 *
 * Handles all product-related API endpoints including:
 * - CRUD operations for product tickets
 * - CAS number lookups via PubChem
 * - Dashboard statistics
 *
 * @module controllers/products
 */
```

---

## Git Workflow

### Commit Messages

```
<type>: <short description>

[optional body with more details]

Types:
- feat: New feature
- fix: Bug fix
- docs: Documentation changes
- style: Code style changes (formatting, etc.)
- refactor: Code refactoring
- perf: Performance improvements
- chore: Maintenance tasks
```

**Examples:**

```
feat: Add collaborator management to tickets

fix: Resolve CAS number validation for edge cases

docs: Update API documentation with new endpoints

refactor: Extract SKU pricing calculations to utility
```

### Branch Naming

```
feature/description-here
fix/issue-description
docs/documentation-update
refactor/component-name
```

---

**Last Updated:** 2026-01-09
**Maintained by:** NPDI Development Team
