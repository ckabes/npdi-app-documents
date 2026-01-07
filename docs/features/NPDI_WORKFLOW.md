# NPDI Workflow - Ticket Initiation

## Overview

When a ticket is initiated in the NPDI (New Product Development & Introduction) system, the official NPDI tracking number is recorded and displayed prominently in a banner. The ticket becomes locked to prevent further edits in this system.

## Ticket Number Flow

### Phase 1: Initial Creation
- **System Action**: Auto-generates ticket number
- **Format**: `NPDI-YYYY-####` (e.g., `NPDI-2025-0055`)
- **Status**: `SUBMITTED` or `DRAFT`
- **Visible**: In all dashboards, ticket details, and activity logs

### Phase 2: SKU Assignment (PMOps)
- **Status**: Changes to `IN_PROCESS` after SKU assignment
- **Ticket Number**: Uses system-generated number
- **Example**: `NPDI-2025-0055`

### Phase 3: NPDI Initiation (PMOps)
- **PMOps Action**: Enters official NPDI tracking number from external NPDI system
- **Input Example**: `100000000003154` (from external NPDI system)
- **System Action**:
  - Stores the official NPDI tracking number in `npdiTracking.trackingNumber`
  - Status changes to `NPDI_INITIATED`
  - Ticket becomes **locked and non-editable**
  - Activity log records the NPDI initiation
  - **Ticket number remains unchanged** (e.g., `NPDI-2025-0055`)

### Example

```
BEFORE NPDI Initiation:
  Ticket Number: NPDI-2025-0055 (system-generated)
  Status: IN_PROCESS

PMOps enters: 100000000003154 (from external NPDI system)

AFTER NPDI Initiation:
  Ticket Number: NPDI-2025-0055 (unchanged)
  Status: NPDI_INITIATED
  Official NPDI Number: 100000000003154 (displayed in banner)
```

## Activity History Entry

When NPDI is initiated, the system logs:

```
NPDI Initiated
Status: NPDI_INITIATED
Action: NPDI_INITIATED
Message: "NPDI initiated by [User Name]. Official NPDI Tracking Number: 100000000003154"
Details:
  - npdiTrackingNumber: 100000000003154
  - initiatedAt: 2025-01-11T...
```

## Where the Official NPDI Number Appears

After NPDI initiation, the **official NPDI tracking number** appears in:

1. **NPDI Banner** - Prominently displayed: "Official NPDI Number: 100000000003154"
2. **Activity History** - Shows the tracking number in the NPDI_INITIATED entry
3. **Database** - Stored in `npdiTracking.trackingNumber` field
4. **Exports/Reports** - Available in data exports

The **ticket number** (`NPDI-2025-0055`) remains unchanged and continues to appear in:
1. **Ticket Header** - Top of ticket details page
2. **Dashboard List** - All ticket listings
3. **Search Results** - Searchable by ticket number

## Important Notes

### Two Numbers, Two Purposes
- **Ticket Number** (e.g., `NPDI-2025-0055`): Internal system tracking number, never changes
- **Official NPDI Number** (e.g., `100000000003154`): External NPDI system tracking number, displayed in banner

### Traceability
- Both numbers are stored and accessible
- Activity history records when NPDI was initiated and by whom
- The official NPDI number is always visible in the banner for NPDI-initiated tickets

### Non-Editability
- Once NPDI is initiated, the ticket is **permanently locked**
- No fields can be edited in this system
- All further changes must be made in the external NPDI system

### Search Considerations
- Search by ticket number: `NPDI-2025-0055`
- The official NPDI number is searchable in activity history text

## Technical Implementation

### Database Fields
```javascript
{
  ticketNumber: "NPDI-2025-0055",  // System-generated, never changes
  npdiTracking: {
    trackingNumber: "100000000003154",  // Official NPDI number
    initiatedAt: Date,
    initiatedBy: "user@example.com"
  },
  status: "NPDI_INITIATED",
  statusHistory: [
    {
      action: "NPDI_INITIATED",
      reason: "NPDI initiated by [User]. Official NPDI Tracking Number: 100000000003154",
      details: {
        npdiTrackingNumber: "100000000003154",
        initiatedAt: "2025-01-11T..."
      }
    }
  ]
}
```

### Backend Logic
- The `ticketNumber` field is protected and cannot be changed after creation
- NPDI initiation stores the tracking number in `npdiTracking.trackingNumber`
- Status is set to `NPDI_INITIATED` which locks the ticket

### Frontend Display
- Ticket header shows `ticket.ticketNumber` (system number)
- NPDI banner shows `ticket.npdiTracking.trackingNumber` (official NPDI number)
- Activity history displays the official NPDI number from the details

## User Experience Flow

### For PMOps

1. **See Ticket** with system number: `NPDI-2025-0055`
2. **Complete to-dos**: Assign SKUs, base part number
3. **Status changes** to `IN_PROCESS`
4. **NPDI Initiation section appears** at top of page
5. **Enter official NPDI number** from external system: `100000000003154`
6. **Click "Initiate NPDI"**
7. **Success message**: "NPDI initiated! Official NPDI number: 100000000003154"
8. **Ticket header still shows**: `NPDI-2025-0055`
9. **Green banner displays**: "Official NPDI Number: 100000000003154"
10. **Activity log shows**: "NPDI initiated. Official NPDI Tracking Number: 100000000003154"

### For All Users

After NPDI initiation:
- Ticket number remains: `NPDI-2025-0055`
- Official NPDI number visible in banner: `100000000003154`
- Ticket is locked - no edits allowed
- Clear indication that NPDI has been initiated

## Security Considerations

- Only PMOps and Admin roles can initiate NPDI
- NPDI initiation is a one-way action (cannot be undone)
- Full audit trail maintained in activity history
