# Template Versioning Requirements

## Overview

This document defines the versioning requirements for ticket templates, form configurations, and view templates in the NPDI application.

## Naming Convention

**Format:** `<Division><Priority>-<Type>-<Version>`

- `<Division><Priority>`: Division code and priority level (e.g., DM1 for Chemistry, AS6 for Biologics)
- `<Type>`: Template type (CHEM for Chemistry, BIOLOGIC for Biologics, etc.)
- `<Version>`: Semantic version number (Major.Minor.Patch)

**Examples:**
- `DM1-CHEM-1.10.0` - Chemistry template, version 1.10.0
- `AS6-BIOLOGIC-1.0.0` - Biologics template, version 1.0.0

## Active Templates

**Current Active Templates:**
- `DM1-CHEM-1.10.0` - Chemistry products (default)
- `AS6-BIOLOGIC-1.0.0` - Biologic products

## Versioning Rules

Version numbers follow semantic versioning:

### Patch Version (x.x.X)

Increment for:
- Bug fixes
- Minor text changes
- Help text updates
- Typo corrections
- Clarifications

**Example:** `1.0.0` → `1.0.1`

### Minor Version (x.X.x)

Increment for:
- New optional fields
- New sections
- Non-breaking changes
- Additional field options
- Enhanced features that don't break existing tickets

**Example:** `1.0.0` → `1.1.0`

### Major Version (X.x.x)

Increment for:
- Breaking changes
- Field removals
- Required field changes
- Structural changes
- Changes that affect existing ticket data compatibility

**Example:** `1.0.0` → `2.0.0`

## Components That Must Match

The following components must use consistent naming:

1. **TicketTemplate** (MongoDB collection: `tickettemplates`)
   - `name` field must be `<Division><Priority>-<Type>-<Version>` (e.g., `DM1-CHEM-1.10.0`)

2. **FormConfiguration** (MongoDB collection: `formconfigurations`)
   - `name` field must match template naming convention
   - `version` field must be `<Version>` (e.g., "1.10.0")

3. **Seed Scripts** (`server/scripts/`)
   - `seedFormConfig.js` - Chemistry form configuration
   - `seedBiologicFormConfig.js` - Biologics form configuration
   - `seedDefaultTemplate.js` - Creates/updates chemistry template
   - `seedBiologicTemplate.js` - Creates/updates biologics template

4. **Dev Profiles** (`server/data/devProfiles.json`)
   - `assignedTemplate` field must reference template name (e.g., `DM1-CHEM`)
   - `templateId` must reference the correct TicketTemplate ObjectId

5. **View Templates** (`client/src/components/TicketView.jsx`)
   - Renders templates dynamically based on form configuration

## Update Checklist

When making changes to a template, follow this checklist:

### Before Making Changes

- [ ] Determine the appropriate version increment (patch/minor/major)
- [ ] Note the current version number
- [ ] Calculate the new version number

### Update Process

1. **Update Seed Script**
   ```bash
   # Edit server/scripts/seedFormConfig.js (for Chemistry)
   # or server/scripts/seedBiologicFormConfig.js (for Biologics)
   # Update the VERSION constant
   ```

2. **Run Seed Scripts**
   ```bash
   # For Chemistry template
   node server/scripts/seedFormConfig.js
   node server/scripts/seedDefaultTemplate.js

   # For Biologics template
   node server/scripts/seedBiologicFormConfig.js
   node server/scripts/seedBiologicTemplate.js
   ```

3. **Test**
   - Create a new ticket with the template
   - View an existing ticket
   - Edit an existing ticket
   - Verify form rendering
   - Verify view rendering

4. **Commit Changes**
   ```bash
   git add .
   git commit -m "Update template to DM1-CHEM-X.Y.Z"
   git push
   ```

### After Deployment

- [ ] Verify template is active in production
- [ ] Test ticket creation and viewing
- [ ] Update team on new template version
- [ ] Archive old template version if necessary

## Template Types

### Current Templates

| Template Name | Type | Version | Description | Status |
|--------------|------|---------|-------------|---------|
| DM1-CHEM-1.10.0 | Chemistry | 1.10.0 | Standard chemical product tickets | Active (Default) |
| AS6-BIOLOGIC-1.0.0 | Biologics | 1.0.0 | Biological product tickets | Active |

### Future Templates (Planned)

| Template Name | Type | Description |
|--------------|------|-------------|
| DM1-INSTR-1.0.0 | Instruments | Laboratory instrument tickets |
| DM1-MAT-1.0.0 | Materials | General materials tickets |

## Version History

### DM1-CHEM-1.10.0 (Current)
- **Date:** December 2025
- **Changes:** Current production version
  - Includes all core sections: Production Type, Basic Info, Vendor Info, Chemical Properties, Pricing, Composition, Quality Specs, CorpBase, Intellectual Property
  - Supports both Produced and Procured product types

### AS6-BIOLOGIC-1.0.0 (Current)
- **Date:** December 2025
- **Changes:** Initial biologics template
  - Specialized sections for biological products
  - Includes Host System, Expression System, Fermentation data
  - Supports both Produced and Procured product types

## Database Queries

### Check Current Template Version

```bash
mongosh npdi-app --eval "
db.tickettemplates.find({ isActive: true }, { name: 1, isDefault: 1 }).pretty()
"
```

### Check Form Configuration Version

```bash
mongosh npdi-app --eval "
db.formconfigurations.find({ isActive: true }, { name: 1, templateName: 1, version: 1 }).pretty()
"
```

### Find Tickets Using Template

```bash
mongosh npdi-app --eval "
db.producttickets.find({ 'template.name': /DM1-CHEM/ }).count()
"
```

## Best Practices

1. **Never** modify an active template version directly
2. **Always** increment version when making changes
3. **Keep** old template versions for reference
4. **Test** thoroughly before activating new version
5. **Document** all changes in version history
6. **Communicate** version updates to users
7. **Maintain** backwards compatibility when possible

## Troubleshooting

### Template Not Found

If users see "Template configuration not available":
1. Check that template exists in database
2. Verify template is marked as active
3. Check user's assignedTemplate in devProfiles.json
4. Verify FormConfiguration references correct templateName

### Version Mismatch

If template name and version don't match:
1. Run seed script to update FormConfiguration
2. Update TicketTemplate name in database
3. Update devProfiles.json references
4. Clear any caches and restart application

## Related Documentation

- [Form Configuration Guide](guides/FORM_CONFIGURATION_GUIDE.md) - How to modify form fields
- [Maintenance Guide](MAINTENANCE_GUIDE.md) - Section 11.6 on form configuration
- [Architecture Documentation](architecture/ARCHITECTURE.md) - Template system overview
- [Discriminator Pattern Plan](DISCRIMINATOR_PATTERN_REFACTORING_PLAN.md) - Future multi-type ticket system
