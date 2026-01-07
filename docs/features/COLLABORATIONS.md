# Collaborations Feature

## Overview

The Collaborations feature allows Product Managers to grant edit access to other users on their tickets. This enables team collaboration while maintaining a full audit trail of all changes made by collaborators.

## Key Features

- **Collaborations Page**: View all tickets where you have edit access as a collaborator
- **Add Collaborators Modal**: Searchable user list for adding multiple collaborators
- **Two Integration Points**: Add collaborators from Section Navigator or Ticket Details
- **Full Audit Trail**: All collaborator additions, removals, and edits are tracked in history

## User Experience

### For Ticket Owners (Product Managers)

1. **Create/Edit a ticket** in DRAFT or SUBMITTED status
2. **Add collaborators** via:
   - Floating button below Section Navigator
   - "Manage Collaborators" button in ticket header
3. **Search users** by name or email in the modal
4. **Select multiple users** to add as collaborators
5. **View collaborator list** with ability to remove

### For Collaborators

1. **Navigate to Collaborations page** in the main navigation
2. **View all tickets** where you have edit access
3. **Click a ticket** to open and edit it
4. **Make changes** - all edits are tracked with your name in history

### For PMOps/Admin

- Full edit access to all tickets (unchanged)
- Can add/remove collaborators on any ticket
- Can view collaboration history in audit trail

## Where Collaborators Appear

### 1. Collaborations Page
- Route: `/collaborations`
- Shows table of all tickets where user is a collaborator
- Columns: Ticket #, Product Name, Owner, Status, Priority, Added Date
- Click row to navigate to ticket

### 2. Ticket Details Page
- Collaborators section shows all current collaborators
- "Manage Collaborators" button (for owner/PMOps/Admin)
- Badge showing collaborator count

### 3. Section Navigator
- Floating "Add Collaborators" button at bottom
- Only visible to ticket owner on DRAFT/SUBMITTED tickets

### 4. Activity History
- `COLLABORATOR_ADDED` entries (green badge)
- `COLLABORATOR_REMOVED` entries (red badge)
- Shows who was added/removed and by whom

## Permissions

### Who Can Add Collaborators
- Ticket owner (createdBy user)
- PMOps role
- Admin role

### Who Can Edit as Collaborator
- Users explicitly added to the `collaborators` array
- Edit access is revoked when removed from collaborators

### What Collaborators Can Edit
- All editable fields that a Product Manager can edit
- Changes tracked with collaborator's name in history
- Cannot add/remove other collaborators (only owner/PMOps/Admin)

## Technical Implementation

### Data Model

#### ProductTicket Schema Addition
```javascript
collaborators: [{
  userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  employeeId: String,
  email: String,
  firstName: String,
  lastName: String,
  role: String,
  addedBy: {
    employeeId: String,
    firstName: String,
    lastName: String
  },
  addedAt: { type: Date, default: Date.now }
}]
```

#### Status History Actions
```javascript
action: {
  type: String,
  enum: [
    'TICKET_CREATED',
    'STATUS_CHANGE',
    'SKU_ASSIGNMENT',
    'TICKET_EDIT',
    'COMMENT_ADDED',
    'NPDI_INITIATED',
    'COLLABORATOR_ADDED',    // NEW
    'COLLABORATOR_REMOVED'   // NEW
  ]
}
```

### API Endpoints

#### Add Collaborator
```
POST /products/:id/collaborators
Body: { userId: "user_id" }
Response: Updated ticket with new collaborator
```

#### Remove Collaborator
```
DELETE /products/:id/collaborators/:userId
Response: Updated ticket without collaborator
```

#### Get Collaborative Tickets
```
GET /products/collaborations
Query: status, page, limit
Response: Paginated list of tickets where user is collaborator
```

#### Search Users
```
GET /users/search?q=searchTerm
Response: Array of matching users (excludes current user)
```

### Client Components

#### AddCollaboratorModal
```jsx
<AddCollaboratorModal
  isOpen={boolean}
  onClose={() => {}}
  ticketId={string}
  existingCollaborators={Array}
  onCollaboratorsAdded={(collaborators) => {}}
/>
```

**Features:**
- Debounced search input
- Multi-select user list
- Selected users shown as removable chips
- Prevents adding existing collaborators
- Loading and error states

#### CollaboratorList
```jsx
<CollaboratorList
  collaborators={Array}
  onRemove={(userId) => {}}
  canRemove={boolean}
/>
```

**Features:**
- Display collaborator name, email, role
- Show who added and when
- Remove button (if permitted)

### File Structure

#### New Files
```
client/src/components/collaborations/
├── AddCollaboratorModal.jsx
├── CollaboratorList.jsx
└── index.js

client/src/pages/
└── CollaborationsPage.jsx
```

#### Modified Files
```
server/models/ProductTicket.js
server/models/schemas/statusHistorySchema.js
server/controllers/products/ticketCRUDController.js
server/routes/products.js
client/src/services/api.js
client/src/components/forms/SectionNavigator.jsx
client/src/pages/TicketDetails.jsx
client/src/App.jsx
```

## Activity History Examples

### Collaborator Added
```
Action: COLLABORATOR_ADDED (Green)
Message: "Added John Doe (john.doe@company.com) as collaborator"
User: Jane Smith (Product Manager)
Timestamp: 2025-01-15 10:30 AM
```

### Collaborator Removed
```
Action: COLLABORATOR_REMOVED (Red)
Message: "Removed John Doe (john.doe@company.com) as collaborator"
User: Jane Smith (Product Manager)
Timestamp: 2025-01-15 11:45 AM
```

### Edit by Collaborator
```
Action: TICKET_EDIT (Orange)
Message: "Ticket updated by collaborator"
User: John Doe (Collaborator)
Timestamp: 2025-01-15 02:15 PM
```

## Security Considerations

- Only authorized users can add/remove collaborators
- Collaborator access is ticket-specific, not global
- All actions logged in audit trail
- Collaborators cannot escalate their own permissions
- Removal is immediate - no grace period

## Implementation Status

All phases have been completed:

- [x] **Phase 1**: Backend schema and API updates
- [x] **Phase 2**: Client API service updates
- [x] **Phase 3**: AddCollaboratorModal component
- [x] **Phase 4**: CollaboratorList component
- [x] **Phase 5**: SectionNavigator button integration
- [x] **Phase 6**: TicketDetails collaborators UI
- [x] **Phase 7**: CollaborationsPage
- [x] **Phase 8**: Route and navigation
- [x] **Phase 9**: History display for new action types

## Database Migration

No migration required - new `collaborators` field will be undefined on existing tickets, which is handled gracefully as an empty array.

## Related Documentation

- [ARCHITECTURE.md](../architecture/ARCHITECTURE.md) - System architecture
- [TICKET_DATA_FLOW.md](../architecture/TICKET_DATA_FLOW.md) - Ticket data flow
- [NPDI_WORKFLOW.md](./NPDI_WORKFLOW.md) - NPDI initiation workflow
