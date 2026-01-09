# Component Catalog

**Version:** 1.1.0
**Last Updated:** 2026-01-09

---

## Table of Contents

1. [Overview](#overview)
2. [Component Directory Structure](#component-directory-structure)
3. [Form Components](#form-components)
4. [Admin Components](#admin-components)
5. [Dashboard Components](#dashboard-components)
6. [Common Components](#common-components)
7. [Modal Components](#modal-components)
8. [Badge Components](#badge-components)
9. [Preview Components](#preview-components)

---

## Overview

The NPDI application contains 54 React components organized by feature area. This catalog provides a reference for all components, their purposes, and key props.

### Component Naming Conventions

| Pattern | Example | Purpose |
|---------|---------|---------|
| `*Form.jsx` | `ProductTicketForm.jsx` | Form input components |
| `*Modal.jsx` | `AddCollaboratorModal.jsx` | Dialog/modal components |
| `*Badge.jsx` | `StatusBadge.jsx` | Status indicator badges |
| `*Management.jsx` | `UserManagement.jsx` | Admin CRUD interfaces |
| `*Popup.jsx` | `MARASearchPopup.jsx` | Popup dialogs |

---

## Component Directory Structure

```
client/src/components/
├── Layout.jsx                # Main layout wrapper
├── Loading.jsx               # Loading indicator
├── MoleculeViewerRDKit.jsx   # Molecule structure viewer
├── TicketView.jsx            # Ticket detail view
├── admin/                    # Admin panel components
├── badges/                   # Status/role badge components
├── collaborations/           # Collaboration feature
├── common/                   # Shared UI components
├── dashboard/                # Dashboard-specific components
├── forms/                    # Form components
│   └── index.js              # Barrel export
├── modals/                   # Modal dialogs
└── preview/                  # Preview/display components
```

---

## Form Components

### ProductTicketForm

**File:** `components/forms/ProductTicketForm.jsx`
**Lines:** ~1,700

Main form component for creating and editing product tickets.

**Props:**

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `mode` | `'create' \| 'edit'` | No | Form mode (default: 'create') |
| `ticket` | Object | No | Existing ticket data for edit mode |
| `template` | Object | No | Form template configuration |
| `loadingTemplate` | boolean | No | Template loading state |
| `register` | Function | Yes | react-hook-form register |
| `handleSubmit` | Function | Yes | react-hook-form submit handler |
| `control` | Object | Yes | react-hook-form control |
| `setValue` | Function | Yes | react-hook-form setValue |
| `watch` | Function | Yes | react-hook-form watch |
| `errors` | Object | Yes | Form validation errors |
| `isDirty` | boolean | Yes | Form dirty state |
| `onSubmit` | Function | Yes | Form submission handler |
| `onCancel` | Function | Yes | Cancel handler |
| `user` | Object | Yes | Current user |
| `missingRequiredFields` | string[] | No | Fields missing for submission |
| `attemptedSubmit` | boolean | No | Whether submit was attempted |

---

### ChemicalPropertiesForm

**File:** `components/forms/ChemicalPropertiesForm.jsx`

Form section for chemical property data (CAS, formula, etc.).

**Props:**

| Prop | Type | Description |
|------|------|-------------|
| `register` | Function | react-hook-form register |
| `setValue` | Function | react-hook-form setValue |
| `watch` | Function | react-hook-form watch |
| `errors` | Object | Validation errors |
| `onCASLookup` | Function | CAS lookup handler |
| `loading` | boolean | Loading state |

---

### BiologicalPropertiesForm

**File:** `components/forms/BiologicalPropertiesForm.jsx`

Form section for biological product properties.

---

### QualitySpecificationsForm

**File:** `components/forms/QualitySpecificationsForm.jsx`

Quality specifications with natural language parsing.

**Features:**
- Natural language input parsing
- Test attribute/method normalization
- Structured data output

---

### PricingCalculationForm

**File:** `components/forms/PricingCalculationForm.jsx`

Pricing calculation with margin/cost inputs.

**Props:**

| Prop | Type | Description |
|------|------|-------------|
| `register` | Function | react-hook-form register |
| `setValue` | Function | react-hook-form setValue |
| `watch` | Function | react-hook-form watch |
| `skuIndex` | number | Index of SKU variant |

---

### SKUVariantsForm

**File:** `components/forms/SKUVariantsForm.jsx`

Array form for managing SKU variants.

**Features:**
- Dynamic add/remove SKU variants
- Package size configuration
- Pricing per variant
- Weight matrix auto-population

---

### CorpBaseDataForm

**File:** `components/forms/CorpBaseDataForm.jsx`

Website/marketing content form with AI generation.

**Props:**

| Prop | Type | Description |
|------|------|-------------|
| `register` | Function | react-hook-form register |
| `setValue` | Function | react-hook-form setValue |
| `watch` | Function | react-hook-form watch |
| `onGenerateContent` | Function | AI content generation handler |

---

### DynamicFormRenderer

**File:** `components/forms/DynamicFormRenderer.jsx`

Renders form sections based on template configuration.

**Props:**

| Prop | Type | Description |
|------|------|-------------|
| `sections` | Section[] | Form section configurations |
| `register` | Function | react-hook-form register |
| `control` | Object | react-hook-form control |
| `watch` | Function | react-hook-form watch |
| `errors` | Object | Validation errors |
| `isReadOnly` | boolean | Disable editing |

---

### DynamicFormSection

**File:** `components/forms/DynamicFormSection.jsx`

Renders individual form section with fields.

---

### SectionNavigator

**File:** `components/forms/SectionNavigator.jsx`

Side navigation for form sections with validation indicators.

---

### ProductHierarchySelector

**File:** `components/forms/ProductHierarchySelector.jsx`

Hierarchical dropdown for product classification.

---

### UNSPSCSelector

**File:** `components/forms/UNSPSCSelector.jsx`

UNSPSC code selector with search.

---

### PlantCodeAutocomplete

**File:** `components/forms/PlantCodeAutocomplete.jsx`

Autocomplete for plant code selection.

---

### BusinessLineAutocomplete

**File:** `components/forms/BusinessLineAutocomplete.jsx`

Autocomplete for business line selection.

---

### SearchableAutocomplete

**File:** `components/forms/SearchableAutocomplete.jsx`

Generic searchable autocomplete component.

---

## Admin Components

### SystemSettings

**File:** `components/admin/SystemSettings.jsx`
**Lines:** ~2,000

Main admin settings panel with tabs for all configuration areas.

**Tabs:**
- General Settings
- Integrations (PubChem, Palantir, Teams, AI)
- AI Prompts Configuration
- Performance Settings

---

### UserManagement

**File:** `components/admin/UserManagement.jsx`

User CRUD management interface.

---

### ApiKeyManagement

**File:** `components/admin/ApiKeyManagement.jsx`

API key generation and management.

**Features:**
- Generate new keys
- View key prefixes
- Enable/disable keys
- Set expiration

---

### PermissionsManagement

**File:** `components/admin/PermissionsManagement.jsx`

Role-based permission configuration.

---

### FeedbackManagement

**File:** `components/admin/FeedbackManagement.jsx`

View and manage user feedback/bug reports.

---

### GPHManagement

**File:** `components/admin/GPHManagement.jsx`

Product hierarchy CSV upload and management.

---

### PlantCodesManager

**File:** `components/admin/PlantCodesManager.jsx`

Plant code CRUD management.

---

### BusinessLineManager

**File:** `components/admin/BusinessLineManager.jsx`

Business line CRUD management.

---

### WeightMatrixManagement

**File:** `components/admin/WeightMatrixManagement.jsx`

Weight matrix configuration for package sizes.

---

### AttributeFixedTextManagement

**File:** `components/admin/AttributeFixedTextManagement.jsx`

SAP attribute picklist upload and management.

---

### ParserKnowledgeManager

**File:** `components/admin/ParserKnowledgeManager.jsx`

Quality spec parser knowledge base management.

---

### MARASearchPopup

**File:** `components/admin/MARASearchPopup.jsx`

Popup for searching SAP MARA data via Palantir.

**Props:**

| Prop | Type | Description |
|------|------|-------------|
| `isOpen` | boolean | Modal visibility |
| `onClose` | Function | Close handler |
| `onSelect` | Function | Selection handler with MARA data |
| `initialSearchType` | string | Default search type |
| `initialSearchValue` | string | Default search value |

---

### SimilarProductsPopup

**File:** `components/admin/SimilarProductsPopup.jsx`

Find similar products by CAS number.

---

### GenericCRUDManager

**File:** `components/admin/GenericCRUDManager.jsx`

Reusable CRUD interface component.

**Props:**

| Prop | Type | Description |
|------|------|-------------|
| `title` | string | Panel title |
| `items` | array | Data items |
| `columns` | array | Table column definitions |
| `onAdd` | Function | Add handler |
| `onEdit` | Function | Edit handler |
| `onDelete` | Function | Delete handler |

---

## Dashboard Components

### StatusCardsGrid

**File:** `components/dashboard/StatusCardsGrid.jsx`

Grid of status summary cards.

---

### TicketTables

**File:** `components/dashboard/TicketTables.jsx`

Ticket listing tables with sorting/filtering.

---

### BreakdownCharts

**File:** `components/dashboard/BreakdownCharts.jsx`

Charts for ticket statistics breakdown.

---

### ProcessingTimesCard

**File:** `components/dashboard/ProcessingTimesCard.jsx`

Processing time statistics display.

---

### PerformanceMetricsRow

**File:** `components/dashboard/PerformanceMetricsRow.jsx`

Performance metrics summary row.

---

### MetricModals

**File:** `components/dashboard/MetricModals.jsx`

Detail modals for metric drill-down.

---

## Common Components

### LoadingSpinner

**File:** `components/common/LoadingSpinner.jsx`

Inline loading spinner component.

---

### Modal

**File:** `components/common/Modal.jsx`

Base modal dialog component.

**Props:**

| Prop | Type | Description |
|------|------|-------------|
| `isOpen` | boolean | Visibility state |
| `onClose` | Function | Close handler |
| `title` | string | Modal title |
| `size` | `'sm' \| 'md' \| 'lg' \| 'xl'` | Modal size |
| `children` | ReactNode | Modal content |

---

### FeedbackButton

**File:** `components/common/FeedbackButton.jsx`

Floating feedback submission button.

---

### FeedbackModal

**File:** `components/common/FeedbackModal.jsx`

User feedback submission modal.

---

### HelpDocumentation

**File:** `components/common/HelpDocumentation.jsx`

In-app help documentation viewer.

---

### UserForm

**File:** `components/common/UserForm.jsx`

User create/edit form component.

---

## Modal Components

### SubmitConfirmationModal

**File:** `components/modals/SubmitConfirmationModal.jsx`

Confirmation dialog for ticket submission.

---

## Badge Components

### StatusBadge

**File:** `components/badges/StatusBadge.jsx`

Ticket status indicator badge.

**Props:**

| Prop | Type | Description |
|------|------|-------------|
| `status` | string | Status value |
| `size` | `'sm' \| 'md' \| 'lg'` | Badge size |

---

### ActiveStatusBadge

**File:** `components/badges/ActiveStatusBadge.jsx`

Active/inactive status badge.

---

### PriorityBadge

**File:** `components/badges/PriorityBadge.jsx`

Ticket priority indicator.

---

### RoleBadge

**File:** `components/badges/RoleBadge.jsx`

User role indicator badge.

---

### GenericBadge

**File:** `components/badges/GenericBadge.jsx`

Configurable generic badge component.

---

## Preview Components

### TicketView

**File:** `components/TicketView.jsx`
**Lines:** ~2,000

Read-only ticket detail view with all sections.

---

### CorpBaseWebsitePreview

**File:** `components/preview/CorpBaseWebsitePreview.jsx`

Preview of website content as it would appear on MilliporeSigma.com.

---

### MoleculeViewerRDKit

**File:** `components/MoleculeViewerRDKit.jsx`

2D molecule structure viewer using RDKit.

---

## Collaboration Components

### CollaboratorList

**File:** `components/collaborations/CollaboratorList.jsx`

Display list of ticket collaborators.

---

### AddCollaboratorModal

**File:** `components/collaborations/AddCollaboratorModal.jsx`

Modal for adding collaborators to tickets.

---

## Pages

**Location:** `client/src/pages/`

| Page | File | Description |
|------|------|-------------|
| Dashboard | `Dashboard.jsx` | Main ticket dashboard |
| Create Ticket | `CreateTicket.jsx` | New ticket form |
| Edit Ticket | `EditTicket.jsx` | Ticket edit form |
| Ticket Details | `TicketDetails.jsx` | Ticket view/detail |
| Admin Panel | `AdminPanel.jsx` | Admin settings |
| Profile Select | `ProfileSelect.jsx` | Dev profile selection |
| Not Found | `NotFound.jsx` | 404 page |

---

**Last Updated:** 2026-01-09
**Maintained by:** NPDI Development Team
