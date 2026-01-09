# Form Configuration System

**Version:** 1.0.0
**Last Updated:** 2026-01-09

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Configuration Schema](#configuration-schema)
4. [Field Types](#field-types)
5. [Render Types](#render-types)
6. [Conditional Display](#conditional-display)
7. [Seed Scripts](#seed-scripts)
8. [Modifying Form Configuration](#modifying-form-configuration)
9. [Template Integration](#template-integration)

---

## Overview

The NPDI application uses a database-driven form configuration system that allows dynamic form generation without code changes. Form configurations define sections, fields, validation rules, and display logic.

### Key Benefits

- **No-code form changes**: Modify forms via database without deployments
- **Template-specific forms**: Different templates can have different field configurations
- **Conditional logic**: Fields can show/hide based on other field values
- **Versioning**: Form configurations are versioned for tracking changes

---

## Architecture

### Component Relationships

```
TicketTemplate
     │
     ├── formConfiguration (ref) ──> FormConfiguration
     │                                    │
     │                                    ├── sections[]
     │                                    │     │
     │                                    │     └── fields[]
     │                                    │
     │                                    └── version
     │
     └── submissionRequirements[] (field keys required for submission)
```

### Data Flow

```
FormConfiguration (DB)
        ↓
TicketTemplate.populate('formConfiguration')
        ↓
ProductTicketForm component
        ↓
DynamicFormRenderer component
        ↓
DynamicFormSection components
        ↓
Individual field components
```

---

## Configuration Schema

### FormConfiguration Model

```javascript
{
  name: String,           // e.g., "DM1-CHEM-1.10.0"
  description: String,    // Human-readable description
  version: String,        // Semantic version (e.g., "1.10.0")
  isActive: Boolean,      // Whether this config is active
  sections: [FormSection],
  createdBy: String,
  updatedBy: String,
  createdAt: Date,
  updatedAt: Date
}
```

### Section Schema

```javascript
{
  sectionKey: String,       // Unique identifier (e.g., "basic", "chemicalProperties")
  name: String,             // Display name (e.g., "Basic Information")
  description: String,      // Help text for section
  visible: Boolean,         // Whether section is shown
  collapsible: Boolean,     // Can be collapsed
  defaultExpanded: Boolean, // Initially expanded
  order: Number,            // Display order
  icon: String,             // Optional icon name
  renderType: String,       // How to render (keyValue, arrayTable, etc.)
  dataPath: String,         // Path to data in ticket object
  fields: [FormField],
  columns: [ColumnDefinition],    // For arrayTable render type
  subSections: [SubSection],      // For complex sections
  isCustom: Boolean
}
```

### Field Schema

```javascript
{
  fieldKey: String,         // Unique identifier (e.g., "productName", "chemicalProperties.casNumber")
  label: String,            // Display label
  type: String,             // Field type (text, select, number, etc.)
  required: Boolean,        // Whether field is required
  visible: Boolean,         // Whether field is shown
  editable: Boolean,        // Whether field can be edited
  defaultValue: Mixed,      // Default value
  placeholder: String,      // Placeholder text
  helpText: String,         // Help tooltip
  options: [FieldOption],   // For select/radio fields
  validation: {
    pattern: String,        // Regex pattern
    min: Number,            // Minimum value
    max: Number,            // Maximum value
    minLength: Number,
    maxLength: Number,
    step: Number
  },
  conditionalDisplay: {
    dependsOn: String,      // Field key to watch
    value: Mixed            // Value that triggers display
  },
  visibleWhen: {
    fieldKey: String,
    value: Mixed,
    values: [String]        // OR condition - show if any value matches
  },
  gridColumn: String,       // Layout: "full", "half", "third", "quarter"
  order: Number,
  isCustom: Boolean
}
```

---

## Field Types

### Basic Field Types

| Type | Description | Example |
|------|-------------|---------|
| `text` | Single-line text input | Product Name |
| `textarea` | Multi-line text input | Description |
| `number` | Numeric input | Molecular Weight |
| `email` | Email input with validation | Contact Email |
| `url` | URL input with validation | Product URL |
| `date` | Date picker | Launch Date |

### Selection Field Types

| Type | Description | Example |
|------|-------------|---------|
| `select` | Dropdown selection | SBU, Priority |
| `radio` | Radio button group | Production Type |
| `checkbox` | Checkbox (boolean) | Is Active |

### Field Configuration Examples

**Text Field:**
```javascript
{
  fieldKey: 'productName',
  label: 'Product Name',
  type: 'text',
  visible: true,
  editable: true,
  placeholder: 'Enter commercial/marketing product name',
  gridColumn: 'full',
  order: 1
}
```

**Select Field:**
```javascript
{
  fieldKey: 'priority',
  label: 'Priority',
  type: 'select',
  visible: true,
  editable: true,
  defaultValue: 'MEDIUM',
  gridColumn: 'half',
  order: 3,
  options: [
    { value: 'LOW', label: 'Low' },
    { value: 'MEDIUM', label: 'Medium' },
    { value: 'HIGH', label: 'High' },
    { value: 'URGENT', label: 'Urgent' }
  ]
}
```

**Radio Field:**
```javascript
{
  fieldKey: 'productionType',
  label: 'Production Type',
  type: 'radio',
  visible: true,
  editable: true,
  defaultValue: 'Produced',
  gridColumn: 'full',
  options: [
    { value: 'Produced', label: 'Produced' },
    { value: 'Procured', label: 'Procured' }
  ]
}
```

---

## Render Types

Sections can specify how their data should be rendered:

### keyValue (Default)

Standard form fields rendered as key-value pairs:
```javascript
{
  sectionKey: 'basic',
  renderType: 'keyValue',
  fields: [/* field definitions */]
}
```

### arrayTable

Data rendered as a table (for arrays like SKU variants):
```javascript
{
  sectionKey: 'skuVariants',
  renderType: 'arrayTable',
  dataPath: 'skuVariants',
  columns: [
    { key: 'sku', label: 'SKU', type: 'text' },
    { key: 'packageSize.value', label: 'Size', type: 'number' },
    { key: 'packageSize.unit', label: 'Unit', type: 'text' },
    { key: 'pricing.listPrice', label: 'List Price', type: 'currency', prefix: '$' }
  ]
}
```

### htmlContent

Rich HTML content display:
```javascript
{
  sectionKey: 'corpbaseData',
  renderType: 'htmlContent',
  dataPath: 'corpbaseData',
  htmlFields: ['productDescription', 'keyFeatures', 'applications']
}
```

### qualitySpecs

Quality specifications with test attributes:
```javascript
{
  sectionKey: 'quality',
  renderType: 'qualitySpecs',
  dataPath: 'quality',
  subSections: [
    {
      key: 'specifications',
      label: 'Specifications',
      renderType: 'arrayTable',
      columns: [/* columns */]
    }
  ]
}
```

### composition

Composition breakdown display:
```javascript
{
  sectionKey: 'composition',
  renderType: 'composition',
  dataPath: 'composition',
  showTotal: true,
  totalField: 'weightPercent'
}
```

---

## Conditional Display

### Simple Conditional

Show field based on another field's value:
```javascript
{
  fieldKey: 'vendorName',
  label: 'Vendor Name',
  type: 'text',
  visible: true,
  visibleWhen: {
    fieldKey: 'productionType',
    value: 'Procured'
  }
}
```

### Multiple Value Conditions

Show field if any of multiple values match:
```javascript
{
  fieldKey: 'manufacturingDetails',
  visibleWhen: {
    fieldKey: 'productionType',
    values: ['Produced', 'Mixed']  // Shows if either value
  }
}
```

### Section-Level Conditions

Entire sections can be conditionally displayed:
```javascript
{
  sectionKey: 'vendorInformation',
  name: 'Vendor Information',
  visible: true,
  // Section visibility controlled by DynamicFormRenderer
  // based on productionType value
}
```

---

## Seed Scripts

### Location

- `server/scripts/seedFormConfig.js` - DM1-CHEM template
- `server/scripts/seedBiologicFormConfig.js` - AS6-BIOLOGIC template

### Running Seed Scripts

```bash
# Seed DM1-CHEM form configuration
node server/scripts/seedFormConfig.js

# Seed AS6-BIOLOGIC form configuration
node server/scripts/seedBiologicFormConfig.js
```

### Seed Script Structure

```javascript
require('dotenv').config();
const mongoose = require('mongoose');
const FormConfiguration = require('../models/FormConfiguration');

// Version number - INCREMENT THIS when making changes
const VERSION = '1.10.0';

const defaultFormConfig = {
  name: `DM1-CHEM-${VERSION}`,
  description: 'Form configuration for NPDI chemical product tickets',
  version: VERSION,
  isActive: true,
  sections: [
    // Section definitions...
  ]
};

const seedFormConfig = async () => {
  await mongoose.connect(process.env.MONGODB_URI);

  // Deactivate existing configs
  await FormConfiguration.updateMany({}, { isActive: false });

  // Create or update the form config
  const result = await FormConfiguration.findOneAndUpdate(
    { name: defaultFormConfig.name },
    defaultFormConfig,
    { upsert: true, new: true }
  );

  console.log(`Form configuration seeded: ${result.name}`);
  await mongoose.disconnect();
};

seedFormConfig();
```

---

## Modifying Form Configuration

### Version Numbering

Follow semantic versioning:
- **Patch (x.x.X)**: Bug fixes, text changes, help text updates
- **Minor (x.X.x)**: New fields, new sections, non-breaking changes
- **Major (X.x.x)**: Breaking changes, field removals, structural changes

### Steps to Modify

1. **Edit the seed script** (`seedFormConfig.js` or `seedBiologicFormConfig.js`)

2. **Update the VERSION constant:**
   ```javascript
   const VERSION = '1.11.0'; // Was 1.10.0
   ```

3. **Make your changes** to sections/fields

4. **Run the seed script:**
   ```bash
   node server/scripts/seedFormConfig.js
   ```

5. **Test the changes** in the application

6. **Document changes** in comments or changelog

### Adding a New Field

```javascript
// In the appropriate section's fields array:
{
  fieldKey: 'newFieldName',
  label: 'New Field Label',
  type: 'text',
  visible: true,
  editable: true,
  placeholder: 'Enter value...',
  helpText: 'Help text explaining the field',
  gridColumn: 'half',
  order: 10,  // Adjust based on desired position
  isCustom: false
}
```

### Adding a New Section

```javascript
// In the sections array:
{
  sectionKey: 'newSection',
  name: 'New Section Name',
  description: 'Section description',
  visible: true,
  collapsible: true,
  defaultExpanded: false,
  order: 15,  // Adjust based on desired position
  isCustom: false,
  fields: [
    // Field definitions...
  ]
}
```

---

## Template Integration

### Linking Template to Form Configuration

```javascript
// When creating a template
const template = await TicketTemplate.create({
  name: 'DM1-CHEM',
  ticketType: 'DM1-CHEM',
  formConfiguration: formConfigId,  // Reference to FormConfiguration
  submissionRequirements: [
    'productName',
    'sbu',
    'chemicalProperties.casNumber'
  ]
});
```

### Submission Requirements

Templates define which fields are required for submission:

```javascript
{
  submissionRequirements: [
    'productName',                      // Top-level field
    'sbu',
    'chemicalProperties.casNumber',     // Nested field
    'chemicalProperties.molecularFormula'
  ]
}
```

These are checked when transitioning from DRAFT to SUBMITTED status.

### Fetching Form Configuration

```javascript
// In controller
const template = await TicketTemplate.findById(templateId)
  .populate('formConfiguration');

// formConfiguration now contains the full form config
const sections = template.formConfiguration.sections;
```

---

## Current Form Configurations

### DM1-CHEM (Chemistry)

**Sections:**
1. Production Type
2. Basic Information
3. Business Line
4. Distribution
5. Vendor Information (conditional)
6. Launch Timeline
7. Chemical Properties
8. Hazard Classification
9. Quality Specifications
10. Composition
11. Regulatory Information
12. SKU Variants
13. CorpBase Website Data
14. Pricing Data
15. Additional Information

### AS6-BIOLOGIC (Biologics)

**Sections:**
1. Production Type
2. Basic Information
3. Business Line
4. Distribution
5. Biological Properties
6. Physical Properties
7. Storage & Shipping
8. Applications
9. SKU Variants
10. CorpBase Website Data
11. Pricing Data

---

**Last Updated:** 2026-01-09
**Maintained by:** NPDI Development Team
