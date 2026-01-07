# NPDI Portal Data Storage Architecture

This document provides comprehensive documentation of how data is stored, structured, and accessed in the NPDI Portal application.

## Table of Contents

1. [Why MongoDB and Mongoose](#why-mongodb-and-mongoose)
2. [Database Connection](#database-connection)
3. [Collections Overview](#collections-overview)
4. [Discriminator Pattern](#discriminator-pattern)
5. [Schema Design Patterns](#schema-design-patterns)
6. [Data Access Patterns](#data-access-patterns)
7. [Indexes and Performance](#indexes-and-performance)
8. [Data Security](#data-security)

---

## Why MongoDB and Mongoose

### MongoDB: The Document Database

MongoDB was chosen as the database for the NPDI Portal for several key reasons:

1. **Flexible Schema**: Product tickets have varying data structures based on product type (chemical vs. biological). MongoDB's document model allows different tickets to have different fields without requiring schema migrations.

2. **Nested Data Structures**: Product data naturally contains nested objects (chemical properties, quality specifications, SKU variants). MongoDB stores these as embedded documents, eliminating the need for complex JOINs.

3. **Rapid Development**: The schema-less nature allows quick iteration during development. New fields can be added without database migrations.

4. **Rich Query Capabilities**: MongoDB's aggregation framework supports complex analytics (dashboard statistics, ticket filtering, user lookups) in a single query.

5. **JSON-Native**: Since the application uses a JavaScript/Node.js stack with a React frontend, MongoDB's BSON (Binary JSON) format provides seamless data flow without type conversions.

### Mongoose: The ODM Layer

Mongoose provides an Object Document Mapping (ODM) layer that brings structure and safety to MongoDB:

1. **Schema Enforcement**: While MongoDB is schema-less, Mongoose enforces schemas at the application level, providing type validation, required fields, and enum constraints.

2. **Discriminator Pattern**: Mongoose's discriminator feature enables inheritance patterns, allowing different ticket types (chemical, biologic) to share a base schema while having type-specific fields.

3. **Middleware/Hooks**: Pre-save and post-save hooks enable automatic data transformation, validation, and audit trail generation.

4. **Population**: Mongoose's populate feature enables JOIN-like queries for referenced documents (users, templates).

5. **Static and Instance Methods**: Custom methods can be added to models for reusable business logic.

---

## Database Connection

The database connection is configured in `server/config/database.js`:

```javascript
const mongoose = require('mongoose');

const connectDB = async () => {
  const conn = await mongoose.connect(
    process.env.MONGODB_URI || 'mongodb://localhost:27017/npdi-app',
    { serverSelectionTimeoutMS: 5000 }
  );
};
```

**Environment Variable**: `MONGODB_URI` specifies the connection string. Defaults to local MongoDB on port 27017.

---

## Collections Overview

The NPDI Portal uses the following MongoDB collections:

### Core Business Data

| Collection | Model | Purpose |
|------------|-------|---------|
| `producttickets` | ProductTicket/BaseTicket | Main ticket storage (chemical, biologic) |
| `tickettemplates` | TicketTemplate | Template definitions linking to form configurations |
| `formconfigurations` | FormConfiguration | Dynamic form section and field definitions |

### User & Preferences

| Collection | Model | Purpose |
|------------|-------|---------|
| `users` | User | User profiles with roles and template assignments |
| `userpreferences` | UserPreferences | Per-user display, notification, and workflow preferences |

### System Configuration

| Collection | Model | Purpose |
|------------|-------|---------|
| `systemsettings` | SystemSettings | Global settings, integrations (PubChem, Teams, Palantir, AI) |
| `apikeys` | ApiKey | External API authentication keys |
| `parserconfigurations` | ParserConfiguration | Quality spec parser knowledge base |

### Reference Data

| Collection | Model | Purpose |
|------------|-------|---------|
| `plantcodes` | PlantCode | Manufacturing plant reference codes |
| `businesslines` | BusinessLine | Business line hierarchy |
| `producthierarchies` | ProductHierarchy | SIAL product hierarchy classifications |
| `weightmatrices` | WeightMatrix | Package size to weight conversion data |
| `permissions` | Permission | Role-based access control definitions |

---

## Discriminator Pattern

The discriminator pattern is a key architectural feature that enables different ticket types to share a common base while having type-specific fields. **The discriminator key uses template names** (e.g., `DM1-CHEM`, `AS6-BIOLOGIC`) for direct mapping between templates and ticket schemas.

### How It Works

```
┌─────────────────────────────────────────────────────────────┐
│                    BaseTicket Schema                         │
│  - Common fields (productName, status, sbu, skuVariants)    │
│  - Shared audit trail (statusHistory, comments)             │
│  - discriminatorKey: 'ticketType'                           │
│  - collection: 'producttickets'                             │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌───────────────┐    ┌────────────────┐    ┌────────────────┐
│  DM1-CHEM     │    │ AS6-BIOLOGIC   │    │ Future Types   │
│  (Chemical)   │    │  (Biologic)    │    │                │
│               │    │                │    │ DM2-CHEM-RESELL│
│ chemicalProps │    │ biologicalProps│    │ AS7-RECOMBINANT│
│ hazardClass   │    │ physicalProps  │    │ etc.           │
│ quality       │    │ storageShipping│    │                │
│ composition   │    │ applications   │    │                │
│ regulatoryInfo│    │ regulatory     │    │                │
└───────────────┘    └────────────────┘    └────────────────┘
```

### Template-Based Discriminators

The discriminator key uses **template names** instead of generic categories:
- `DM1-CHEM` for chemical products (maps to DM1-CHEM template)
- `AS6-BIOLOGIC` for biological products (maps to AS6-BIOLOGIC template)

This design allows:
- **Direct mapping**: Template name IS the ticket type - no translation needed
- **Easy extensibility**: Adding `DM2-CHEM-RESELL` creates a new discriminator with its own schema
- **Right-sized schemas**: Each template type has only the fields it needs

### Implementation

**Base Ticket** (`server/models/BaseTicket.js`):
```javascript
const baseTicketSchema = new mongoose.Schema({
  // Common fields shared by all ticket types
  ticketNumber: { type: String, unique: true },
  productName: { type: String, required: true },
  status: { type: String, enum: ['DRAFT', 'SUBMITTED', ...] },
  sbu: { type: String, required: true },
  skuVariants: [skuVariantSchema],
  statusHistory: [statusHistorySchema],
  comments: [commentSchema],
  // ... more common fields
}, {
  discriminatorKey: 'ticketType',  // Template name (e.g., 'DM1-CHEM')
  collection: 'producttickets'      // All types stored in same collection
});

const BaseTicket = mongoose.model('Ticket', baseTicketSchema);
```

**DM1-CHEM Ticket Discriminator** (`server/models/discriminators/DM1ChemTicket.js`):
```javascript
const {
  chemicalPropertiesSchema,
  hazardClassificationSchema,
  qualitySchema,
  compositionSchema,
  regulatoryInfoSchema
} = require('../schemas');

const dm1ChemSchema = new mongoose.Schema({
  chemicalProperties: chemicalPropertiesSchema,
  hazardClassification: hazardClassificationSchema,
  quality: qualitySchema,
  composition: compositionSchema,
  regulatoryInfo: regulatoryInfoSchema
});

// Template name as discriminator key
const DM1ChemTicket = BaseTicket.discriminator('DM1-CHEM', dm1ChemSchema);
```

**AS6-BIOLOGIC Ticket Discriminator** (`server/models/discriminators/AS6BiologicTicket.js`):
```javascript
const {
  biologicalPropertiesSchema,
  physicalPropertiesSchema,
  storageShippingSchema,
  applicationsSchema,
  biologicalRegulatorySchema
} = require('../schemas');

const as6BiologicSchema = new mongoose.Schema({
  biologicalProperties: biologicalPropertiesSchema,
  physicalProperties: physicalPropertiesSchema,
  storageShipping: storageShippingSchema,
  applications: applicationsSchema,
  regulatory: biologicalRegulatorySchema
});

// Template name as discriminator key
const AS6BiologicTicket = BaseTicket.discriminator('AS6-BIOLOGIC', as6BiologicSchema);
```

### Using the TicketFactory

The `TicketFactory` (`server/models/TicketFactory.js`) provides a unified interface for creating and querying tickets. **Controllers must use TicketFactory** instead of instantiating models directly:

```javascript
const TicketFactory = require('./models/TicketFactory');

// Get ticket type from user's template
const ticketType = TicketFactory.getTypeFromTemplate(userTemplate);  // 'DM1-CHEM'

// Get the correct model and create ticket
const TicketModel = TicketFactory.getModel(ticketType);
const ticket = new TicketModel({
  productName: 'Acetonitrile',
  sbu: '775',
  chemicalProperties: { casNumber: '75-05-8' }
});
await ticket.save();

// Or use createTicket for simple cases
const ticket = await TicketFactory.createTicket('DM1-CHEM', ticketData);

// Get model for type-specific queries
const DM1ChemTicket = TicketFactory.getModel('DM1-CHEM');
const chemicals = await DM1ChemTicket.find({ 'chemicalProperties.physicalState': 'Liquid' });

// Query all tickets regardless of type
const allTickets = await TicketFactory.findAll({ status: 'SUBMITTED' });

// Get type distribution
const distribution = await TicketFactory.getTypeDistribution();
// Returns: [{ _id: 'DM1-CHEM', count: 150 }, { _id: 'AS6-BIOLOGIC', count: 45 }]
```

### Shared Sub-Schemas

Field definitions are extracted into reusable schemas in `server/models/schemas/`:

| Schema | Used By | Description |
|--------|---------|-------------|
| `chemicalPropertiesSchema` | DM1-CHEM | CAS, molecular formula, physical state, etc. |
| `hazardClassificationSchema` | DM1-CHEM | GHS class, hazard statements, signal word |
| `qualitySchema` | DM1-CHEM | MQ level, quality attributes |
| `compositionSchema` | DM1-CHEM | Chemical components, weight percentages |
| `regulatoryInfoSchema` | DM1-CHEM | FDA, EPA, REACH, TSCA |
| `biologicalPropertiesSchema` | AS6-BIOLOGIC | Source, clone, isotype, reactivity |
| `physicalPropertiesSchema` | AS6-BIOLOGIC | Form, concentration, pH |
| `storageShippingSchema` | AS6-BIOLOGIC | Shelf life, sensitivity |
| `applicationsSchema` | AS6-BIOLOGIC | Intended use, techniques |
| `biologicalRegulatorySchema` | AS6-BIOLOGIC | Quality level, GMP, FDA/CE |

### Benefits of Template-Based Discriminators

1. **Single Collection**: All ticket types in one collection enables cross-type queries and aggregations
2. **Direct Mapping**: Template name = ticket type - no translation layer needed
3. **Right-Sized Schemas**: Each template has only the fields it needs (no null fields)
4. **Type Safety**: Each type has its own validated schema with template-specific required fields
5. **Easy Extensibility**: New templates get their own discriminator without affecting existing ones
6. **Automatic Routing**: Mongoose automatically instantiates the correct model based on `ticketType`

### Adding a New Template Type

To add a new template (e.g., `DM2-CHEM-RESELL`):

1. Create discriminator: `server/models/discriminators/DM2ChemResellTicket.js`
2. Add to registry in `TicketFactory.js`
3. Export from `server/models/discriminators/index.js`
4. Run migration if needed

---

## Schema Design Patterns

### Embedded Documents

Complex nested data is stored as embedded documents for atomic operations and query efficiency:

```javascript
// SKU Variants embedded in ticket
skuVariants: [{
  type: { type: String, enum: ['BULK', 'CONF', 'SPEC', 'VAR', 'PREPACK'] },
  packageSize: { value: Number, unit: String },
  pricing: {
    standardCost: Number,
    listPrice: Number,
    currency: { type: String, default: 'USD' }
  }
}]

// Status history embedded for audit trail
statusHistory: [{
  status: String,
  changedByEmployeeId: String,
  changedAt: { type: Date, default: Date.now },
  reason: String,
  action: { type: String, enum: ['TICKET_CREATED', 'STATUS_CHANGE', ...] },
  userInfo: { firstName: String, lastName: String, role: String }
}]
```

### Referenced Documents

For data that needs independent management or is shared across documents:

```javascript
// User reference on ticket
createdByUser: {
  type: mongoose.Schema.Types.ObjectId,
  ref: 'User'
}

// Template reference
template: {
  type: mongoose.Schema.Types.ObjectId,
  ref: 'TicketTemplate'
}
```

### Enum Validation

Enums enforce valid values at the schema level:

```javascript
status: {
  type: String,
  enum: ['DRAFT', 'SUBMITTED', 'IN_PROCESS', 'NPDI_INITIATED', 'COMPLETED', 'CANCELED'],
  default: 'DRAFT'
}

physicalState: {
  type: String,
  enum: ['Solid', 'Liquid', 'Gas', 'Powder', 'Crystal']
}
```

### Pre-Save Hooks

Automatic data transformation and validation:

```javascript
// Normalize enum fields before validation
baseTicketSchema.pre('validate', function() {
  // Convert empty strings to undefined to avoid enum validation errors
  if (this.productScope && this.productScope.scope === '') {
    this.productScope.scope = undefined;
  }

  // Normalize units (mL, L capitalization)
  if (this.skuVariants) {
    this.skuVariants.forEach(variant => {
      if (variant.packageSize?.unit === 'ml') {
        variant.packageSize.unit = 'mL';
      }
    });
  }
});

// Update timestamp on save
baseTicketSchema.pre('save', function(next) {
  this.updatedAt = new Date();
  next();
});

// Auto-generate ticket number for new tickets
productTicketSchema.pre('save', async function() {
  if (this.isNew && !this.ticketNumber) {
    const year = new Date().getFullYear();
    const count = await mongoose.model('ProductTicket').countDocuments();
    this.ticketNumber = `NPDI-${year}-${String(count + 1).padStart(4, '0')}`;
  }
});
```

### Static Methods

Reusable query methods on models:

```javascript
// Find chemical tickets by CAS number
chemicalTicketSchema.statics.findByCAS = function(casNumber) {
  return this.findOne({ 'chemicalProperties.casNumber': casNumber });
};

// Get or create user preferences
userPreferencesSchema.statics.getOrCreate = async function(userId, isEmployeeId = false) {
  const query = isEmployeeId
    ? { userEmployeeId: userId }
    : { $or: [{ userEmployeeId: userId }, { userId }] };
  let preferences = await this.findOne(query);
  if (!preferences) {
    preferences = await this.create(isEmployeeId ? { userEmployeeId: userId } : { userId });
  }
  return preferences;
};

// Singleton pattern for system settings
systemSettingsSchema.statics.getSettings = async function() {
  let settings = await this.findOne();
  if (!settings) {
    settings = await this.create({});
  }
  return settings;
};
```

---

## Data Access Patterns

### Creating Data

**Creating a Ticket**:
```javascript
// From ticketCRUDController.js
const createTicket = async (req, res) => {
  // Determine ticket type from user's template
  const userTemplate = userRecord?.ticketTemplate;
  const ticketType = TicketFactory.getTypeFromTemplate(userTemplate);

  let ticketData = {
    ...req.body,
    ticketType,
    createdByEmployeeId: currentUser.employeeId,
    createdByUser: userRecord?._id,
    template: userRecord?.ticketTemplate?._id
  };

  // Clean and validate data
  ticketData = cleanTicketData(ticketData);

  // Create and save
  const ticket = new ProductTicket(ticketData);
  ticket.statusHistory = [{
    status: ticket.status,
    changedByEmployeeId: currentUser.employeeId,
    action: 'TICKET_CREATED',
    userInfo: currentUser
  }];

  await ticket.save();
};
```

### Reading Data

**Single Document with Population**:
```javascript
const ticket = await ProductTicket.findOne({ _id: id })
  .populate('createdByUser', 'firstName lastName email employeeId')
  .populate({
    path: 'template',
    populate: { path: 'formConfiguration' }
  });
```

**Aggregation Pipeline for Lists**:
```javascript
const pipeline = [
  { $match: filter },
  // Lookup user by employeeId
  {
    $lookup: {
      from: 'users',
      localField: 'createdByEmployeeId',
      foreignField: 'employeeId',
      as: 'createdByEmployeeIdLookup'
    }
  },
  { $unwind: { path: '$createdByEmployeeIdLookup', preserveNullAndEmptyArrays: true } },
  // Facet for pagination
  {
    $facet: {
      tickets: [
        { $sort: { updatedAt: -1 } },
        { $skip: skip },
        { $limit: limit },
        { $project: { ticketNumber: 1, productName: 1, status: 1, ... } }
      ],
      total: [{ $count: 'count' }]
    }
  }
];

const result = await ProductTicket.aggregate(pipeline);
```

### Updating Data

**Partial Update with History Tracking**:
```javascript
const updateTicket = async (req, res) => {
  const ticket = await ProductTicket.findOne({ _id: id });

  let updateData = cleanTicketData(req.body);
  const oldStatus = ticket.status;
  const newStatus = updateData.status;

  // Apply updates
  Object.assign(ticket, updateData);

  // Track status change
  if (newStatus && newStatus !== oldStatus) {
    ticket.statusHistory.push({
      status: newStatus,
      changedByEmployeeId: currentUser.employeeId,
      reason: `Status changed from ${oldStatus} to ${newStatus}`,
      action: 'STATUS_CHANGE',
      userInfo: currentUser
    });
  }

  await ticket.save();  // Pre-save hooks run here
};
```

### Deleting Data

Tickets are typically not deleted but rather set to CANCELED status for audit trail preservation. For true deletion:

```javascript
await TicketFactory.deleteById(id);
```

---

## Indexes and Performance

### Strategic Indexes

Indexes are defined on schemas to optimize common query patterns:

```javascript
// Type-based queries (discriminator pattern)
baseTicketSchema.index({ ticketType: 1 });
baseTicketSchema.index({ ticketType: 1, status: 1 });
baseTicketSchema.index({ ticketType: 1, createdAt: -1 });

// Dashboard and list queries
baseTicketSchema.index({ status: 1, updatedAt: -1 });
baseTicketSchema.index({ status: 1, sbu: 1 });
baseTicketSchema.index({ status: 1, priority: 1 });

// User queries
baseTicketSchema.index({ createdBy: 1, status: 1 });
baseTicketSchema.index({ assignedTo: 1, status: 1 });

// Date-based queries
baseTicketSchema.index({ createdAt: -1 });
baseTicketSchema.index({ sbu: 1, createdAt: -1 });

// Chemical-specific indexes
chemicalTicketSchema.index({ 'chemicalProperties.casNumber': 1 });
chemicalTicketSchema.index({ 'chemicalProperties.pubchemCID': 1 });

// Biologic-specific indexes
biologicTicketSchema.index({ 'biologicalProperties.cloneName': 1 });
biologicTicketSchema.index({ 'biologicalProperties.isotype': 1 });
```

### Query Optimization Tips

1. **Use Projections**: Only select needed fields in list queries
2. **Use Aggregation Pipelines**: Combine multiple operations in one database round-trip
3. **Use $facet**: Get paginated results and total count in single query
4. **Avoid $where**: Use native MongoDB operators instead
5. **Index Compound Queries**: Create compound indexes for commonly used filter combinations

---

## Data Security

### Encryption at Rest

Sensitive fields are encrypted using AES-256-GCM before storage:

```javascript
// In SystemSettings model
systemSettingsSchema.pre('save', function(next) {
  if (this.isModified('integrations.langdock.apiKey')) {
    const apiKey = this.integrations?.langdock?.apiKey;
    if (apiKey && !encryption.isEncrypted(apiKey)) {
      this.integrations.langdock.apiKey = encryption.encrypt(apiKey);
    }
  }

  if (this.isModified('integrations.palantir.token')) {
    const token = this.integrations?.palantir?.token;
    if (token && !encryption.isEncrypted(token)) {
      this.integrations.palantir.token = encryption.encrypt(token);
    }
  }
  next();
});
```

### API Key Security

API keys are never stored in plain text:

```javascript
// In ApiKey model
apiKeySchema.statics.generateKey = function() {
  return crypto.randomBytes(32).toString('hex');
};

apiKeySchema.statics.hashKey = function(key) {
  return crypto.createHash('sha256').update(key).digest('hex');
};

// Only store the hash, show only prefix to users
apiKeySchema.methods.toJSON = function() {
  const obj = this.toObject();
  delete obj.keyHash;  // Never expose hash
  return obj;
};
```

### Data Sanitization

Input data is cleaned before storage:

```javascript
// From enumCleaner.js
const cleanTicketData = (ticketData) => {
  // Remove empty strings from enum fields
  // Normalize case for specific fields
  // Validate and sanitize user input
  return cleanedData;
};
```

---

## Model File Reference

### Core Models

| File | Description |
|------|-------------|
| `server/models/BaseTicket.js` | Base ticket schema with discriminator configuration |
| `server/models/ProductTicket.js` | Legacy model with all fields (backward compatibility) |
| `server/models/TicketFactory.js` | Factory for creating and querying typed tickets |
| `server/models/TicketTemplate.js` | Template definitions |
| `server/models/FormConfiguration.js` | Dynamic form configurations |
| `server/models/User.js` | User profiles |
| `server/models/UserPreferences.js` | User preferences |
| `server/models/SystemSettings.js` | Global system settings |
| `server/models/ApiKey.js` | API key management |
| `server/models/ParserConfiguration.js` | Quality parser knowledge base |
| `server/models/index.js` | Barrel export for all models |

### Discriminators (Template-Based)

| File | Discriminator Key | Description |
|------|-------------------|-------------|
| `server/models/discriminators/DM1ChemTicket.js` | `DM1-CHEM` | Chemical product fields |
| `server/models/discriminators/AS6BiologicTicket.js` | `AS6-BIOLOGIC` | Biological product fields |
| `server/models/discriminators/index.js` | - | Barrel export for discriminators |

### Shared Sub-Schemas

| File | Used By | Description |
|------|---------|-------------|
| `server/models/schemas/skuVariantSchema.js` | All | SKU variant definitions |
| `server/models/schemas/statusHistorySchema.js` | All | Audit trail entries |
| `server/models/schemas/commentSchema.js` | All | User comments |
| `server/models/schemas/chemicalPropertiesSchema.js` | DM1-CHEM | CAS, molecular properties |
| `server/models/schemas/hazardClassificationSchema.js` | DM1-CHEM | GHS hazard data |
| `server/models/schemas/qualitySchema.js` | DM1-CHEM | MQ quality attributes |
| `server/models/schemas/compositionSchema.js` | DM1-CHEM | Chemical composition |
| `server/models/schemas/regulatoryInfoSchema.js` | DM1-CHEM | Regulatory compliance |
| `server/models/schemas/biologicalPropertiesSchema.js` | AS6-BIOLOGIC | Source, clone, isotype |
| `server/models/schemas/physicalPropertiesSchema.js` | AS6-BIOLOGIC | Form, concentration, pH |
| `server/models/schemas/storageShippingSchema.js` | AS6-BIOLOGIC | Storage requirements |
| `server/models/schemas/applicationsSchema.js` | AS6-BIOLOGIC | Intended use, techniques |
| `server/models/schemas/biologicalRegulatorySchema.js` | AS6-BIOLOGIC | GMP, FDA/CE compliance |
| `server/models/schemas/index.js` | - | Barrel export for schemas |

---

## Related Documentation

- [Architecture Overview](ARCHITECTURE.md) - System design and components
- [Ticket Data Flow](TICKET_DATA_FLOW.md) - How data flows through the system
- [API Documentation](../api/API_DOCUMENTATION.md) - REST API reference
- [Template Versioning](../TEMPLATE_VERSIONING.md) - Ticket template system

---

**Last Updated:** 2025-12-08
**Maintained by:** NPDI Development Team
