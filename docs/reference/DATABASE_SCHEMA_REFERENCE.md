# Database Schema Reference

**Version:** 1.0.0
**Last Updated:** 2026-01-09
**Database:** MongoDB with Mongoose ODM

---

## Table of Contents

1. [Overview](#overview)
2. [Core Models](#core-models)
   - [ProductTicket](#productticket)
   - [BaseTicket](#baseticket)
   - [User](#user)
   - [TicketTemplate](#tickettemplate)
3. [Discriminator Models](#discriminator-models)
   - [DM1ChemTicket](#dm1chemticket)
   - [AS6BiologicTicket](#as6biologicticket)
4. [Configuration Models](#configuration-models)
   - [FormConfiguration](#formconfiguration)
   - [SystemSettings](#systemsettings)
   - [Permission](#permission)
   - [ParserConfiguration](#parserconfiguration)
5. [Reference Data Models](#reference-data-models)
   - [PlantCode](#plantcode)
   - [BusinessLine](#businessline)
   - [ProductHierarchy](#producthierarchy)
   - [WeightMatrix](#weightmatrix)
   - [AttributeFixedText](#attributefixedtext)
6. [Supporting Models](#supporting-models)
   - [ApiKey](#apikey)
   - [Feedback](#feedback)
   - [UserPreferences](#userpreferences)
7. [Embedded Schemas](#embedded-schemas)
8. [Indexes](#indexes)
9. [Relationships](#relationships)

---

## Overview

The NPDI application uses MongoDB as its database with Mongoose as the ODM (Object Document Mapper). The schema design uses several patterns:

- **Discriminator Pattern**: Used for ticket types (DM1-CHEM, AS6-BIOLOGIC) to share common fields while allowing type-specific extensions
- **Embedded Documents**: Complex nested data (chemical properties, SKU variants) are embedded rather than referenced
- **Versioned Data**: Some collections (ProductHierarchy, AttributeFixedText) use versioning for bulk data updates
- **Singleton Pattern**: SystemSettings uses a single document to store application configuration

### Collections Overview

| Collection | Model | Purpose |
|------------|-------|---------|
| `producttickets` | ProductTicket, BaseTicket | Product development tickets (legacy and discriminator) |
| `users` | User | User accounts and roles |
| `tickettemplates` | TicketTemplate | Ticket form templates |
| `formconfigurations` | FormConfiguration | Dynamic form field definitions |
| `systemsettings` | SystemSettings | Application-wide settings |
| `permissions` | Permission | Role-based access control |
| `apikeys` | ApiKey | External API authentication |
| `feedbacks` | Feedback | User bug reports and feature requests |
| `userpreferences` | UserPreferences | Per-user settings |
| `plantcodes` | PlantCode | Manufacturing plant reference data |
| `businesslines` | BusinessLine | Business line reference data |
| `producthierarchies` | ProductHierarchy | Product classification hierarchy |
| `weightmatrixes` | WeightMatrix | Package size to weight mappings |
| `attributefixedtexts` | AttributeFixedText | SAP attribute picklist data |
| `parserconfigurations` | ParserConfiguration | Quality spec parser knowledge base |

---

## Core Models

### ProductTicket

**Collection:** `producttickets`
**File:** `server/models/ProductTicket.js`

The main ticket model for product development requests. Contains comprehensive product information.

#### Fields

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `ticketNumber` | String | No | Auto-generated | Unique ticket ID (format: `NPDI-YYYY-XXXX`) |
| `productName` | String | **Yes** | - | Name of the product |
| `ticketType` | String (enum) | No | `'DM1-CHEM'` | Type: `DM1-CHEM`, `AS6-BIOLOGIC`, `instrument`, `consumable` |
| `productionType` | String (enum) | No | `'Produced'` | `Produced` or `Procured` |
| `sbu` | String | **Yes** | - | Strategic Business Unit code |
| `primaryPlant` | String | No | - | Primary manufacturing plant code |
| `status` | String (enum) | No | `'DRAFT'` | Workflow status (see [Business Rules](../guides/BUSINESS_RULES.md)) |
| `priority` | String (enum) | No | `'MEDIUM'` | `LOW`, `MEDIUM`, `HIGH`, `URGENT` |
| `createdBy` | String | No | - | Creator's email address |
| `createdByEmployeeId` | String | No | - | Creator's employee ID (e.g., `M361549`) |
| `createdByUser` | ObjectId (ref: User) | No | - | Reference to User document |
| `assignedTo` | String | No | - | Assignee's email address |
| `assignedToEmployeeId` | String | No | - | Assignee's employee ID |
| `template` | ObjectId (ref: TicketTemplate) | No | - | Reference to template used |
| `brand` | String (enum) | No | - | Brand: `Sigma-Aldrich`, `SAFC`, `Supelco`, etc. |
| `countryOfOrigin` | String | No | - | Country of origin |
| `currency` | String | No | `'USD'` | Currency code |
| `createdAt` | Date | No | `Date.now` | Creation timestamp |
| `updatedAt` | Date | No | `Date.now` | Last update timestamp |

#### Embedded Objects

| Field | Schema | Description |
|-------|--------|-------------|
| `productScope` | Object | Distribution scope (`Worldwide`, `US only`, etc.) |
| `distributionType` | Object | Distribution configuration (COA, labeling) |
| `retestOrExpiration` | Object | Expiration/retest date configuration |
| `businessLine` | Object | Business line and GPH product line |
| `vendorInformation` | Object | Vendor details for procured products |
| `intellectualProperty` | Object | Patent/license information |
| `partNumber` | Object | Assigned part number and metadata |
| `baseUnit` | Object | Base unit of measure |
| `npdiTracking` | Object | External NPDI system tracking |
| `corpbaseData` | Object | Website/marketing content |
| `pricingData` | Object | Pricing configuration |
| `launchTimeline` | Object | Launch date and milestones |
| `chemicalProperties` | chemicalPropertiesSchema | Chemical data (CAS, formula, etc.) |
| `hazardClassification` | hazardClassificationSchema | GHS hazard data |
| `quality` | qualitySchema | Quality specifications |
| `composition` | compositionSchema | Component composition |
| `regulatoryInfo` | regulatoryInfoSchema | Regulatory compliance |
| `biologicalProperties` | biologicalPropertiesSchema | Biological product data |
| `physicalProperties` | physicalPropertiesSchema | Physical characteristics |
| `storageShipping` | storageShippingSchema | Storage/shipping requirements |
| `applications` | applicationsSchema | Product applications |

#### Arrays

| Field | Type | Description |
|-------|------|-------------|
| `skuVariants` | [skuVariantSchema] | Package sizes and pricing |
| `collaborators` | [Object] | Users with access to this ticket |
| `statusHistory` | [statusHistorySchema] | Audit trail of status changes |
| `comments` | [commentSchema] | Discussion comments |

#### Indexes

```javascript
{ status: 1, updatedAt: -1 }           // Dashboard queries
{ status: 1, sbu: 1 }                  // Filtered lists
{ status: 1, priority: 1 }             // Priority filtering
{ createdBy: 1, status: 1 }            // User's tickets
{ assignedTo: 1, status: 1 }           // Assignment queries
{ createdAt: -1 }                      // Date sorting
{ 'chemicalProperties.casNumber': 1 } // CAS lookup
{ sbu: 1, createdAt: -1 }              // SBU reports
{ 'collaborators.employeeId': 1, status: 1 } // Collaborator queries
```

#### Pre-Save Hooks

1. **Validation hook**: Cleans up empty enum values, normalizes SKU variant units
2. **Timestamp hook**: Updates `updatedAt`, adds initial status history entry
3. **Ticket number generation**: Auto-generates unique ticket numbers for new tickets

---

### BaseTicket

**Collection:** `producttickets` (shared with discriminators)
**File:** `server/models/BaseTicket.js`

Base schema for the discriminator pattern. Contains common fields shared across all ticket types.

#### Purpose

- Enables cross-type queries: `BaseTicket.find({})` returns all ticket types
- Discriminator key: `ticketType` determines which model handles the document
- Type-specific models extend this with additional fields

#### Static Methods

| Method | Parameters | Returns | Description |
|--------|------------|---------|-------------|
| `findByType` | `ticketType`, `filter` | Query | Find tickets by type with optional filters |
| `getTypeDistribution` | - | Promise<Array> | Aggregation of ticket counts by type |

---

### User

**Collection:** `users`
**File:** `server/models/User.js`

User account information and role assignment.

#### Fields

| Field | Type | Required | Unique | Description |
|-------|------|----------|--------|-------------|
| `employeeId` | String | **Yes** | **Yes** | Format: `M` + numbers (e.g., `M361549`) |
| `email` | String | **Yes** | **Yes** | Email address (lowercase) |
| `firstName` | String | **Yes** | No | First name |
| `lastName` | String | **Yes** | No | Last name |
| `role` | String (enum) | **Yes** | No | `PRODUCT_MANAGER`, `PM_OPS`, `ADMIN` |
| `sbu` | String (enum) | Conditional | No | Required for `PRODUCT_MANAGER` role |
| `isActive` | Boolean | No | No | Account active status (default: `true`) |
| `ticketTemplate` | ObjectId (ref: TicketTemplate) | No | No | Default template for user |
| `lastLogin` | Date | No | No | Last login timestamp |
| `createdAt` | Date | No | No | Account creation date |
| `updatedAt` | Date | No | No | Last update date |

#### SBU Values

- `Life Science`
- `Process Solutions`
- `Electronics`
- `Healthcare`

#### Indexes

```javascript
{ employeeId: 1 }  // Primary lookup (unique)
{ email: 1 }       // Email lookup (unique)
```

---

### TicketTemplate

**Collection:** `tickettemplates`
**File:** `server/models/TicketTemplate.js`

Defines ticket form templates that determine available fields and validation.

#### Fields

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `name` | String | **Yes** | - | Unique template name |
| `description` | String | No | `''` | Template description |
| `ticketType` | String (enum) | **Yes** | `'DM1-CHEM'` | Associated ticket type |
| `formConfiguration` | ObjectId (ref: FormConfiguration) | **Yes** | - | Form field definitions |
| `isDefault` | Boolean | No | `false` | Default template flag |
| `isActive` | Boolean | No | `true` | Template active status |
| `submissionRequirements` | [String] | No | `[]` | Required field keys for submission |
| `createdBy` | String | No | `'system'` | Creator email |
| `createdByEmployeeId` | String | No | `null` | Creator employee ID |
| `updatedBy` | String | No | `'system'` | Last updater email |
| `updatedByEmployeeId` | String | No | `null` | Last updater employee ID |

#### Pre-Save Hook

When `isDefault` is set to `true`, all other templates are automatically set to `isDefault: false`.

---

## Discriminator Models

### DM1ChemTicket

**Collection:** `producttickets`
**File:** `server/models/discriminators/DM1ChemTicket.js`
**Discriminator Key:** `'DM1-CHEM'`

Template-specific discriminator for chemical product tickets.

#### Additional Fields

Extends BaseTicket with:

| Field | Schema | Description |
|-------|--------|-------------|
| `chemicalProperties` | chemicalPropertiesSchema | CAS, formula, molecular weight, etc. |
| `hazardClassification` | hazardClassificationSchema | GHS hazard data |
| `quality` | qualitySchema | Quality specifications and attributes |
| `composition` | compositionSchema | Component breakdown |
| `regulatoryInfo` | regulatoryInfoSchema | FDA, EPA, REACH compliance |

#### Indexes

```javascript
{ 'chemicalProperties.casNumber': 1 }  // CAS lookup
{ 'chemicalProperties.pubchemCID': 1 } // PubChem lookup
```

#### Static Methods

| Method | Parameters | Returns | Description |
|--------|------------|---------|-------------|
| `findByCAS` | `casNumber` | Promise | Find by CAS registry number |
| `findByFormula` | `formula` | Query | Search by molecular formula |

---

### AS6BiologicTicket

**Collection:** `producttickets`
**File:** `server/models/discriminators/AS6BiologicTicket.js`
**Discriminator Key:** `'AS6-BIOLOGIC'`

Template-specific discriminator for biological product tickets.

#### Additional Fields

Extends BaseTicket with:

| Field | Schema | Description |
|-------|--------|-------------|
| `biologicalProperties` | biologicalPropertiesSchema | Source, clone, isotype |
| `physicalProperties` | physicalPropertiesSchema | Form, concentration, pH |
| `storageShipping` | storageShippingSchema | Storage requirements |
| `applications` | applicationsSchema | Intended use, techniques |

#### Indexes

```javascript
{ 'biologicalProperties.cloneName': 1 }      // Clone lookup
{ 'biologicalProperties.isotype': 1 }        // Isotype lookup
{ 'biologicalProperties.biologicalSource': 1 } // Source lookup
```

#### Static Methods

| Method | Parameters | Returns | Description |
|--------|------------|---------|-------------|
| `findByClone` | `cloneName` | Promise | Find by clone name |
| `findByIsotype` | `isotype` | Query | Search by isotype |
| `findByReactivity` | `species` | Query | Search by species reactivity |

---

## Configuration Models

### FormConfiguration

**Collection:** `formconfigurations`
**File:** `server/models/FormConfiguration.js`

Defines dynamic form sections and fields for ticket templates.

#### Fields

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `name` | String | **Yes** | `'Product Ticket Form'` | Form name |
| `description` | String | No | - | Form description |
| `version` | String | **Yes** | `'1.0.0'` | Semantic version |
| `isActive` | Boolean | No | `true` | Active status |
| `sections` | [formSectionSchema] | No | - | Form sections |
| `createdBy` | String | No | `'system'` | Creator |
| `updatedBy` | String | No | `'system'` | Last updater |

#### Section Schema

| Field | Type | Description |
|-------|------|-------------|
| `sectionKey` | String | Unique section identifier |
| `name` | String | Display name |
| `description` | String | Section description |
| `visible` | Boolean | Visibility flag |
| `collapsible` | Boolean | Can be collapsed |
| `defaultExpanded` | Boolean | Initially expanded |
| `order` | Number | Display order |
| `fields` | [formFieldSchema] | Fields in this section |
| `renderType` | String (enum) | `keyValue`, `arrayTable`, `htmlContent`, etc. |
| `dataPath` | String | Path to data in ticket object |
| `columns` | [columnDefinitionSchema] | Column definitions for tables |

#### Field Schema

| Field | Type | Description |
|-------|------|-------------|
| `fieldKey` | String | Unique field identifier |
| `label` | String | Display label |
| `type` | String (enum) | `text`, `textarea`, `number`, `select`, `radio`, `checkbox`, `date`, `email`, `url` |
| `required` | Boolean | Required for validation |
| `visible` | Boolean | Visibility flag |
| `editable` | Boolean | Can be edited |
| `defaultValue` | Mixed | Default value |
| `placeholder` | String | Placeholder text |
| `helpText` | String | Help tooltip |
| `options` | [fieldOptionSchema] | Options for select/radio |
| `validation` | Object | Validation rules (pattern, min, max, etc.) |
| `conditionalDisplay` | Object | Conditional visibility rules |
| `gridColumn` | String | Layout: `full`, `half`, `third`, `quarter` |
| `order` | Number | Display order |

---

### SystemSettings

**Collection:** `systemsettings`
**File:** `server/models/SystemSettings.js`

Singleton document for application-wide configuration.

#### Structure

```javascript
{
  general: {
    systemName: String,        // Default: 'NPDI Application'
    systemDescription: String,
    companyName: String,
    supportEmail: String
  },
  tickets: {
    enableStatusHistory: Boolean,  // Default: true
    enableComments: Boolean        // Default: true
  },
  integrations: {
    pubchem: { enabled, timeout, cacheTime, autoPopulation },
    teams: { enabled, webhookUrl, notifyOnStatusChange, ... },
    langdock: { enabled, apiKey (encrypted), environment, model, ... },
    palantir: { enabled, token (encrypted), datasetRID, hostname, ... }
  },
  aiPrompts: {
    productDescription: { enabled, prompt, maxWords, temperature },
    websiteTitle: { enabled, prompt, maxChars, temperature },
    metaDescription: { ... },
    keyFeatures: { ... },
    applications: { ... },
    targetIndustries: { ... }
  },
  templateAiPrompts: Map,  // Template-specific AI prompt overrides
  performance: {
    cache: { enabled, timeout },
    files: { maxFileSize, maxFilesPerTicket }
  },
  lastUpdatedBy: String,
  version: Number
}
```

#### Encryption

The following fields are encrypted at rest using AES-256-GCM:
- `integrations.langdock.apiKey`
- `integrations.palantir.token`

#### Static Methods

| Method | Parameters | Returns | Description |
|--------|------------|---------|-------------|
| `getSettings` | - | Promise | Get or create settings document |
| `updateSettings` | `updates`, `userId` | Promise | Deep merge updates |

#### Instance Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `getDecryptedApiKey()` | String | Decrypt LangDock API key |
| `getDecryptedPalantirToken()` | String | Decrypt Palantir token |

---

### Permission

**Collection:** `permissions`
**File:** `server/models/Permission.js`

Role-based access control configuration.

#### Fields

| Field | Type | Required | Unique | Description |
|-------|------|----------|--------|-------------|
| `role` | String (enum) | **Yes** | **Yes** | `PRODUCT_MANAGER`, `PM_OPS`, `ADMIN` |
| `privileges` | Object | No | No | Permission matrix |

#### Privileges Structure

Each privilege area has `view` and `edit` boolean flags:

- `tickets` - Ticket access
- `drafts` - Draft ticket access
- `skuVariants` - SKU variant management
- `skuAssignment` - Part number assignment
- `chemicalProperties` - Chemical data
- `hazardClassification` - Hazard data
- `corpbaseData` - Website content
- `pricingData` - Pricing information
- `comments` - Commenting
- `statusHistory` - Status history
- `adminPanel` - Admin dashboard

#### Default Permissions

| Role | Key Capabilities |
|------|-----------------|
| `PRODUCT_MANAGER` | Create/edit own tickets, view pricing (no edit) |
| `PM_OPS` | Edit submitted tickets, assign SKUs, manage pricing |
| `ADMIN` | Full access to all features |

---

### ParserConfiguration

**Collection:** `parserconfigurations`
**File:** `server/models/ParserConfiguration.js`

Knowledge base for the quality specification natural language parser.

#### Fields

| Field | Type | Required | Unique | Description |
|-------|------|----------|--------|-------------|
| `configType` | String (enum) | **Yes** | **Yes** | `testAttribute`, `testMethod`, `defaultMethod` |
| `entries` | [parserEntrySchema] | No | No | Parser knowledge entries |
| `version` | Number | No | `1` | Version for cache invalidation |
| `totalEntries` | Number | No | `0` | Computed count |
| `customEntriesCount` | Number | No | `0` | User-added entries count |
| `lastModifiedBy` | String | No | `'system'` | Last modifier |

#### Entry Schema

| Field | Type | Description |
|-------|------|-------------|
| `key` | String | Lowercase key for matching |
| `value` | String | Properly formatted value |
| `category` | String | Category (chemical, biological, etc.) |
| `description` | String | Optional description |
| `isCustom` | Boolean | User-added vs seeded |

---

## Reference Data Models

### PlantCode

**Collection:** `plantcodes`
**File:** `server/models/PlantCode.js`

Manufacturing plant reference data.

| Field | Type | Required | Unique | Description |
|-------|------|----------|--------|-------------|
| `code` | String | **Yes** | **Yes** | Plant code (uppercase) |
| `description` | String | **Yes** | No | Plant description |
| `active` | Boolean | No | `true` | Active status |
| `metadata.extractedFrom` | String | No | - | Source file |
| `metadata.lastExtracted` | Date | No | - | Extraction date |
| `metadata.extractedBy` | ObjectId (ref: User) | No | - | User who extracted |

---

### BusinessLine

**Collection:** `businesslines`
**File:** `server/models/BusinessLine.js`

Business line reference data.

| Field | Type | Required | Unique | Description |
|-------|------|----------|--------|-------------|
| `code` | String | **Yes** | **Yes** | Business line code |
| `description` | String | **Yes** | No | Description |
| `active` | Boolean | No | `true` | Active status |
| `metadata` | Object | No | - | Source and extraction info |

---

### ProductHierarchy

**Collection:** `producthierarchies`
**File:** `server/models/ProductHierarchy.js`

Versioned product classification hierarchy.

#### Fields

| Field | Type | Description |
|-------|------|-------------|
| `metadata.source` | String | Source file name |
| `metadata.version` | Number | Version number |
| `metadata.uploadedBy` | ObjectId (ref: User) | Uploader |
| `divisions` | Mixed | Full hierarchy structure |
| `isActive` | Boolean | Current active version |
| `stats` | Object | Record counts |

#### Hierarchy Structure

```
divisions → units → fields → lines → groups → mainGroups → prodhTree
```

#### Static Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `getActive()` | Promise | Get active hierarchy |
| `createNewVersion(data, uploadedBy)` | Promise | Create new version, deactivate old |

---

### WeightMatrix

**Collection:** `weightmatrixes`
**File:** `server/models/WeightMatrix.js`

Package size to weight mappings based on SAP MARA data.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `size` | String | **Yes** | Package size (e.g., "100G", "1L") |
| `grossWeight` | Number | **Yes** | Gross weight value (BRGEW) |
| `weightUnit` | String (enum) | **Yes** | Unit: `MG`, `G`, `KG`, `UG` |
| `normalizedSize` | Object | No | Normalized size for comparison |
| `normalizedGrossWeight` | Object | No | Weight converted to grams |

---

### AttributeFixedText

**Collection:** `attributefixedtexts`
**File:** `server/models/AttributeFixedText.js`

Versioned SAP attribute picklist data (38K+ entries).

#### Fields

| Field | Type | Description |
|-------|------|-------------|
| `metadata.version` | Number | Version number |
| `metadata.uploadedBy` | ObjectId (ref: User) | Uploader |
| `attributeTypeIndex` | Mixed | Lookup map: typeId → elements |
| `uniqueAttributeTypes` | [Object] | List of attribute types |
| `isActive` | Boolean | Current active version |
| `stats` | Object | Record counts |

#### Static Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `getActive()` | Promise | Get active version |
| `getPicklistByType(typeId)` | Promise<Array> | Get values for attribute type |
| `getUniqueAttributeTypes()` | Promise<Array> | Get all attribute types |
| `createNewVersion(data, uploadedBy)` | Promise | Create new version |

---

## Supporting Models

### ApiKey

**Collection:** `apikeys`
**File:** `server/models/ApiKey.js`

External API authentication keys.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | String | **Yes** | Key name |
| `description` | String | No | Description |
| `keyHash` | String | **Yes** | SHA-256 hash of key (never store plaintext) |
| `keyPrefix` | String | **Yes** | First 8 characters for display |
| `isActive` | Boolean | No | Active status (default: `true`) |
| `permissions` | [String] | No | `read`, `write`, `admin` |
| `lastUsedAt` | Date | No | Last usage timestamp |
| `usageCount` | Number | No | Usage counter |
| `rateLimit.requestsPerHour` | Number | No | Rate limit (default: 1000) |
| `expiresAt` | Date | No | Expiration date |
| `ipWhitelist` | [String] | No | Allowed IP addresses |
| `application` | String | No | Application name |

#### Static Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `generateKey()` | String | Generate 64-character hex key |
| `hashKey(key)` | String | SHA-256 hash of key |

#### Instance Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `isExpired()` | Boolean | Check expiration |
| `isValid()` | Boolean | Check active and not expired |
| `recordUsage()` | Promise | Increment usage counter |

---

### Feedback

**Collection:** `feedbacks`
**File:** `server/models/Feedback.js`

User bug reports and feature requests.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `feedbackId` | String | Auto | Format: `FB-YYYYMMDD-XXXX` |
| `feedbackType` | String (enum) | **Yes** | `BUG`, `CRITICAL_FAILURE`, `FEATURE_REQUEST` |
| `title` | String | **Yes** | Title (max 100 chars) |
| `description` | String | **Yes** | Description (min 20 chars) |
| `isRead` | Boolean | No | Read status (default: `false`) |
| `submittedBy` | Object | No | User info snapshot |
| `currentPage` | String | No | Page where submitted |
| `clientErrors` | [Object] | No | Captured client-side errors |
| `serverLogs` | [Object] | No | Server logs at submission |

---

### UserPreferences

**Collection:** `userpreferences`
**File:** `server/models/UserPreferences.js`

Per-user settings and preferences.

#### Structure

```javascript
{
  userId: String,           // Legacy email
  userEmployeeId: String,   // Employee ID (preferred)
  display: {
    timezone: String,       // Default: 'America/New_York'
    dateFormat: String,     // 'MM/DD/YYYY', 'DD/MM/YYYY', 'YYYY-MM-DD'
    timeFormat: String,     // '12-hour', '24-hour'
    theme: String,          // 'light', 'dark', 'auto'
    language: String
  },
  notifications: {
    email: { enabled, newTicket, statusChange, comments, ... },
    browser: { enabled, newTicket, statusChange, ... }
  },
  dashboard: {
    defaultView: String,    // 'grid', 'list', 'kanban'
    itemsPerPage: Number,   // 10-100
    showCompletedTickets: Boolean,
    defaultFilter: Object,
    defaultSort: Object
  },
  ticketForm: {
    saveAsDraftByDefault: Boolean,
    autoSaveInterval: Number,  // 10-300 seconds
    defaultPriority: String,
    showHelpText: Boolean
  },
  accessibility: { reducedMotion, highContrast, fontSize, screenReader },
  advanced: { showDeveloperInfo, enableExperimentalFeatures, compactMode },
  sbu: { preferred, rememberSelection },
  ticketDefaults: { defaultBusinessLine, defaultSBU },
  pmOps: { serviceSBUs, dashboardFilters, thresholds },
  aiPrompts: { useCustomPrompts, customPrompts }
}
```

---

## Embedded Schemas

**File:** `server/models/schemas/`

Reusable embedded schemas:

| Schema | File | Used In |
|--------|------|---------|
| `skuVariantSchema` | `skuVariantSchema.js` | ProductTicket, BaseTicket |
| `chemicalPropertiesSchema` | `chemicalPropertiesSchema.js` | ProductTicket, DM1ChemTicket |
| `biologicalPropertiesSchema` | `biologicalPropertiesSchema.js` | ProductTicket, AS6BiologicTicket |
| `hazardClassificationSchema` | `hazardClassificationSchema.js` | ProductTicket, DM1ChemTicket |
| `qualitySchema` | `qualitySchema.js` | ProductTicket, DM1ChemTicket |
| `compositionSchema` | `compositionSchema.js` | ProductTicket, DM1ChemTicket |
| `regulatoryInfoSchema` | `regulatoryInfoSchema.js` | ProductTicket, DM1ChemTicket |
| `physicalPropertiesSchema` | `physicalPropertiesSchema.js` | ProductTicket, AS6BiologicTicket |
| `storageShippingSchema` | `storageShippingSchema.js` | ProductTicket, AS6BiologicTicket |
| `applicationsSchema` | `applicationsSchema.js` | ProductTicket, AS6BiologicTicket |
| `statusHistorySchema` | `statusHistorySchema.js` | ProductTicket, BaseTicket |
| `commentSchema` | `commentSchema.js` | ProductTicket, BaseTicket |

---

## Indexes

### Performance Indexes

| Collection | Index | Purpose |
|------------|-------|---------|
| `producttickets` | `{ status: 1, updatedAt: -1 }` | Dashboard queries |
| `producttickets` | `{ createdBy: 1, status: 1 }` | User's tickets |
| `producttickets` | `{ 'chemicalProperties.casNumber': 1 }` | CAS lookup |
| `producttickets` | `{ ticketType: 1, status: 1 }` | Type-based queries |
| `users` | `{ employeeId: 1 }` | Primary user lookup |
| `apikeys` | `{ keyHash: 1 }` | API key validation |
| `feedbacks` | `{ feedbackType: 1, isRead: 1 }` | Feedback filtering |

---

## Relationships

### Entity Relationships

```
User ─────────────┬──────────────> TicketTemplate
                  │
                  └──────────────> UserPreferences (1:1)

TicketTemplate ────────────────> FormConfiguration

ProductTicket ─────┬─────────────> User (createdByUser)
                   │
                   ├─────────────> TicketTemplate (template)
                   │
                   └─────────────> User[] (collaborators)

Permission ────────────────────> Role (1:1 per role)

ProductHierarchy ──────────────> User (uploadedBy)
AttributeFixedText ────────────> User (uploadedBy)
```

### Discriminator Pattern

```
BaseTicket (Ticket)
    │
    ├── DM1ChemTicket (ticketType: 'DM1-CHEM')
    │
    └── AS6BiologicTicket (ticketType: 'AS6-BIOLOGIC')
```

---

**Last Updated:** 2026-01-09
**Maintained by:** NPDI Development Team
