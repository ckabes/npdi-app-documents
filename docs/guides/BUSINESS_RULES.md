# Business Rules & Workflow Documentation

**Version:** 1.0.0
**Last Updated:** 2026-01-09

---

## Table of Contents

1. [Overview](#overview)
2. [Ticket Workflow](#ticket-workflow)
3. [Status Transitions](#status-transitions)
4. [Role-Based Access](#role-based-access)
5. [SKU Assignment Rules](#sku-assignment-rules)
6. [Validation Rules](#validation-rules)
7. [Collaboration Rules](#collaboration-rules)
8. [Notification Rules](#notification-rules)

---

## Overview

This document defines the business rules that govern how the NPDI application operates. These rules are implemented in code but documented here for reference and maintenance.

### Key Principles

1. **Audit Trail**: All significant actions are recorded in status history
2. **Role-Based Access**: Permissions determine what users can view and edit
3. **Workflow Enforcement**: Tickets must follow defined status progression
4. **Data Integrity**: Validation prevents invalid data entry

---

## Ticket Workflow

### Workflow Diagram

```
┌─────────┐     ┌───────────┐     ┌────────────┐     ┌────────────────┐     ┌───────────┐
│  DRAFT  │────>│ SUBMITTED │────>│ IN_PROCESS │────>│ NPDI_INITIATED │────>│ COMPLETED │
└─────────┘     └───────────┘     └────────────┘     └────────────────┘     └───────────┘
     │               │                  │                    │
     │               │                  │                    │
     └───────────────┴──────────────────┴────────────────────┴───────> CANCELED
```

### Status Definitions

| Status | Description | Owner | Editable By |
|--------|-------------|-------|-------------|
| **DRAFT** | Ticket is being created | Product Manager | Creator, Collaborators |
| **SUBMITTED** | Awaiting PMOps review | PMOps Team | PM_OPS, ADMIN |
| **IN_PROCESS** | PMOps actively working | PMOps Team | PM_OPS, ADMIN |
| **NPDI_INITIATED** | Submitted to external NPDI | System | ADMIN only |
| **COMPLETED** | Product is live | N/A | ADMIN only (corrections) |
| **CANCELED** | Ticket was canceled | N/A | Not editable |

---

## Status Transitions

### Allowed Transitions

| From Status | To Status | Who Can Transition | Requirements |
|-------------|-----------|-------------------|--------------|
| DRAFT | SUBMITTED | Product Manager, ADMIN | Required fields filled |
| SUBMITTED | IN_PROCESS | PM_OPS, ADMIN | None |
| SUBMITTED | DRAFT | PM_OPS, ADMIN | Rejection reason required |
| IN_PROCESS | NPDI_INITIATED | PM_OPS, ADMIN | Part number assigned, NPDI tracking number |
| IN_PROCESS | SUBMITTED | PM_OPS, ADMIN | Return reason required |
| NPDI_INITIATED | COMPLETED | PM_OPS, ADMIN | None |
| Any (except COMPLETED) | CANCELED | PM_OPS, ADMIN | Cancellation reason required |

### Transition Actions

#### DRAFT → SUBMITTED

**Triggered by:** Product Manager clicks "Submit"

**Actions:**
1. Validate required fields (based on template `submissionRequirements`)
2. Update status to SUBMITTED
3. Record status change in history with `action: 'STATUS_CHANGE'`
4. Send Teams notification (if enabled)
5. Lock certain fields from Product Manager editing

**Validation Required:**
- `productName` - Must be provided
- `sbu` - Must be selected
- Template-specific required fields

#### SUBMITTED → IN_PROCESS

**Triggered by:** PMOps clicks "Accept" or "Start Processing"

**Actions:**
1. Update status to IN_PROCESS
2. Optionally assign to specific PMOps user
3. Record status change with reason
4. Send notification to Product Manager

#### IN_PROCESS → NPDI_INITIATED

**Triggered by:** PMOps clicks "Initiate NPDI"

**Requirements:**
- Part number must be assigned
- NPDI tracking number must be provided

**Actions:**
1. Update `npdiTracking.trackingNumber`
2. Update `npdiTracking.initiatedBy` and `initiatedAt`
3. Optionally update `ticketNumber` to match NPDI tracking number
4. Preserve original ticket number in status history
5. Update status to NPDI_INITIATED

#### Any → CANCELED

**Triggered by:** PMOps or Admin cancels ticket

**Requirements:**
- Reason must be provided

**Actions:**
1. Update status to CANCELED
2. Record reason in status history
3. Send notification to Product Manager
4. Ticket becomes read-only

---

## Role-Based Access

### Role Definitions

| Role | Primary Responsibilities |
|------|-------------------------|
| **PRODUCT_MANAGER** | Create tickets, fill product data, submit for processing |
| **PM_OPS** | Process tickets, assign SKUs, manage pricing, initiate NPDI |
| **ADMIN** | Full system access, user management, system configuration |

### Permission Matrix

| Feature | PRODUCT_MANAGER | PM_OPS | ADMIN |
|---------|-----------------|--------|-------|
| View own tickets | ✅ | ✅ | ✅ |
| View all tickets | ❌ | ✅ | ✅ |
| Create tickets | ✅ | ❌ | ✅ |
| Edit DRAFT tickets | ✅ (own) | ❌ | ✅ |
| Edit SUBMITTED+ tickets | ❌ | ✅ | ✅ |
| Chemical properties | Edit | View | Edit |
| Hazard classification | Edit | View | Edit |
| SKU variants | Edit | Edit | Edit |
| SKU assignment | ❌ | ✅ | ✅ |
| Pricing data | View | Edit | Edit |
| CorpBase data | Edit | View | Edit |
| Comments | Edit | Edit | Edit |
| Status history | View | Edit | Edit |
| Admin panel | ❌ | ❌ | ✅ |

### Ticket Visibility Rules

1. **PRODUCT_MANAGER** can see:
   - Tickets they created
   - Tickets where they are a collaborator

2. **PM_OPS** can see:
   - All tickets (any status)
   - Can filter by their service SBUs

3. **ADMIN** can see:
   - Everything

### Field-Level Permissions

Some fields have role-specific edit permissions:

| Field/Section | PRODUCT_MANAGER | PM_OPS | ADMIN |
|---------------|-----------------|--------|-------|
| Product Name | ✅ | ✅ | ✅ |
| Part Number | ❌ | ✅ | ✅ |
| SKU Assignment | ❌ | ✅ | ✅ |
| Pricing | View only | ✅ | ✅ |
| NPDI Tracking | ❌ | ✅ | ✅ |

---

## SKU Assignment Rules

### Part Number Assignment

**Who can assign:** PM_OPS, ADMIN only

**Process:**
1. Navigate to ticket in IN_PROCESS status
2. Enter base part number
3. System records:
   - `partNumber.baseNumber`
   - `partNumber.assignedBy` (email)
   - `partNumber.assignedByEmployeeId`
   - `partNumber.assignedAt` (timestamp)
4. Status history entry created with `action: 'SKU_ASSIGNMENT'`

### SKU Variant Rules

| Variant Type | Suffix Pattern | Example |
|--------------|---------------|---------|
| BULK | `-BULK` | 12345-BULK |
| PREPACK | Size (e.g., `100G`) | 12345-100G |
| CONF | `-CONF` | 12345-CONF |
| SPEC | `-SPEC` | 12345-SPEC |
| VAR | `-VAR` | 12345-VAR |

### Weight Matrix Auto-Population

When a prepack size is entered:
1. System looks up size in WeightMatrix
2. If found, auto-populates `grossWeight` and `netWeight`
3. User can override auto-populated values

### Pricing Calculation Rules

```
List Price = Standard Cost / (1 - Margin%)

Example:
- Standard Cost: $50
- Margin: 65%
- List Price = $50 / (1 - 0.65) = $50 / 0.35 = $142.86
```

---

## Validation Rules

### Required Fields for Submission

These fields must be filled before a ticket can move from DRAFT to SUBMITTED:

**Universal Requirements:**
- `productName`
- `sbu`

**Template-Specific Requirements:**

Defined in `TicketTemplate.submissionRequirements` array. Examples:

**DM1-CHEM Template:**
- `chemicalProperties.casNumber` (if applicable)
- `chemicalProperties.molecularFormula`

**AS6-BIOLOGIC Template:**
- `biologicalProperties.biologicalSource`
- `biologicalProperties.cloneName`

### Field-Level Validation

| Field | Validation Rule |
|-------|-----------------|
| `employeeId` | Format: `M` followed by digits (e.g., `M361549`) |
| `email` | Valid email format |
| `casNumber` | Format: `XXXXX-XX-X` (flexible first part) |
| `status` | Must be valid enum value |
| `priority` | Must be: `LOW`, `MEDIUM`, `HIGH`, `URGENT` |
| `ticketType` | Must be: `DM1-CHEM`, `AS6-BIOLOGIC`, etc. |

### Pre-Save Validation

The system performs these cleanups before saving:

1. **Empty enum cleanup**: Empty strings converted to `undefined` for enum fields
2. **Unit normalization**: Package units normalized (e.g., `ml` → `mL`)
3. **Timestamp update**: `updatedAt` automatically updated

---

## Collaboration Rules

### Adding Collaborators

**Who can add:** Ticket creator, existing collaborators, PM_OPS, ADMIN

**Process:**
1. Search for user by name or employee ID
2. Select user to add
3. System records:
   - Collaborator info (employeeId, email, name, role)
   - Who added them
   - Timestamp
4. Status history entry with `action: 'COLLABORATOR_ADDED'`

### Collaborator Permissions

Collaborators have the same permissions as the ticket creator for that ticket:
- Can view all ticket data
- Can edit fields based on ticket status and their role
- Can add comments
- Can add other collaborators

### Removing Collaborators

**Who can remove:** Ticket creator, PM_OPS, ADMIN

**Restrictions:**
- Cannot remove the ticket creator
- Status history entry created with `action: 'COLLABORATOR_REMOVED'`

---

## Notification Rules

### Teams Notifications

Notifications are sent when enabled in SystemSettings:

| Event | Setting | Recipients |
|-------|---------|------------|
| Ticket Created | `notifyOnTicketCreated` | PMOps channel |
| Status Change | `notifyOnStatusChange` | Creator, collaborators |
| Comment Added | `notifyOnCommentAdded` | Creator, collaborators |
| Assignment | `notifyOnAssignment` | Assignee |

### Email Notifications (if configured)

User preferences control email notifications per user:
- `notifications.email.newTicket`
- `notifications.email.statusChange`
- `notifications.email.comments`
- `notifications.email.assignments`

---

## Audit Trail Rules

### What Gets Recorded

Every significant action creates a status history entry:

```javascript
{
  status: String,           // Current status
  changedBy: String,        // Email (legacy)
  changedByEmployeeId: String, // Employee ID
  changedAt: Date,          // Timestamp
  reason: String,           // Optional reason
  action: String,           // Action type
  details: Mixed,           // Additional context
  userInfo: {               // User snapshot
    firstName: String,
    lastName: String,
    role: String
  }
}
```

### Action Types

| Action | When Recorded |
|--------|---------------|
| `TICKET_CREATED` | New ticket created |
| `STATUS_CHANGE` | Status transition |
| `SKU_ASSIGNMENT` | Part number assigned |
| `TICKET_EDIT` | Significant field changes |
| `COMMENT_ADDED` | New comment posted |
| `NPDI_INITIATED` | NPDI tracking added |
| `COLLABORATOR_ADDED` | User added as collaborator |
| `COLLABORATOR_REMOVED` | Collaborator removed |

### Retention

- Status history is never deleted
- Comments are never deleted (soft delete if needed)
- All changes are permanent and traceable

---

## Edge Cases & Special Rules

### Ticket Number Changes

When NPDI is initiated, the ticket number may change:
- Original: `NPDI-2025-0055`
- After NPDI: `10000030016545` (external NPDI number)
- Original number preserved in status history `details`

### Template Changes

- Tickets are locked to their template type after creation
- Cannot change a DM1-CHEM ticket to AS6-BIOLOGIC
- Template version is captured at ticket creation

### Concurrent Editing

- No built-in locking mechanism
- Last save wins
- Users should coordinate via comments

---

**Last Updated:** 2026-01-09
**Maintained by:** NPDI Development Team
