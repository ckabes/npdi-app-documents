# NPDI Portal Documentation Inventory

**Generated:** 2025-12-13
**Last Review:** 2025-12-13
**Repository:** npdi-app
**Application Version:** 1.3.0
**Status:** Current

This document provides a comprehensive inventory of all public documentation available in the NPDI Portal GitHub repository, including outlines, version numbers, accuracy status, and notes on any discrepancies found during review.

---

## Summary

| Category | Document Count | Status |
|----------|----------------|--------|
| Root Documentation | 2 | ✅ Current |
| Guides | 2 | ✅ Current |
| API Documentation | 4 | ✅ Current |
| Architecture | 3 | ✅ Current |
| Features | 12 | ✅ Current |
| Integrations | 2 | ✅ Current |
| Security | 2 | ✅ Current |
| Analysis | 1 | ✅ Current |
| **Total** | **28** | |

---

## Review Summary (2025-12-13)

### Updates Made During This Review

1. **README.md** - Updated to version 1.3.0:
   - Added SAP & CorpBase Integration section
   - Added missing admin components to project structure
   - Added missing services (sapExport/, corpbaseExport/, pifExportService.js)
   - Added Integration Endpoints section to API Endpoints
   - Added preview/ and modals/ component directories

2. **docs/README.md** - Updated:
   - Added version 1.3.0 and updated last updated date
   - Added Security and Analysis sections to documentation index
   - Added internal/ folder to documentation structure
   - Updated Key Features Summary with SAP/CorpBase features

3. **API_DOCUMENTATION.md** - Updated to version 1.3.0:
   - Added Section 5.4: Export SAP MDG-M Loader File endpoint
   - Added Section 5.5: Export CorpBase Batch Client Loader endpoint
   - Added Section 6: Integration Endpoints (CorpBase Substance ID Lookup)
   - Renumbered Admin Endpoints to Section 7
   - Added changelog entry for version 1.3.0

4. **Claude Attribution Removed** from 3 documents:
   - `docs/api/CHANGELOG_API_KEYS.md` - Changed to "NPDI Development Team"
   - `docs/security/DEPENDENCY_SECURITY_ASSESSMENT.md` - Changed to "NPDI Development Team", updated to v1.1
   - `docs/features/SAP-MARA-to-ProductTicket-Mapping.md` - Changed to "NPDI Development Team", updated to v1.1

5. **Schema Cleanup** - Removed unused fields from ProductTicket.js:
   - Removed `inventory` field from `skuVariantSchema` (completely unused)
   - Removed `biologicalRegulatorySchema` and `regulatory` field (unused in frontend and backend)
   - Removed standalone `standardCost` and `margin` fields (duplicated in `pricingData` and SKU pricing)
   - Deleted `server/models/schemas/biologicalRegulatorySchema.js`
   - Updated `server/models/schemas/index.js` exports
   - Updated `server/models/discriminators/AS6BiologicTicket.js`
   - Updated `server/utils/enumCleaner.js`

6. **Documentation Corrections**:
   - Removed "Regulatory Information" from README.md features list (schema exists for data export but no UI)

### Key Findings

The documentation was largely accurate. The main discrepancies were:

1. **Missing documentation for newer features** - Now documented
2. **Unused schema fields** - Cleaned up (see Schema Cleanup above)
3. **Regulatory feature overstated** - The `regulatoryInfo` schema exists for data export purposes, but there is no UI to edit these fields. Removed from feature list to avoid confusion.

### Verified Features

The following features mentioned in documentation were verified to exist in codebase:
- ✅ Feedback System - Feedback.js model, FeedbackManagement.jsx
- ✅ Collaborations - collaborators array in tickets, addCollaborators controller
- ✅ Similar Products Search - SimilarProductsPopup.jsx, searchSimilarProducts route
- ✅ Quality Spec Parser - ParserKnowledgeManager.jsx, ParserConfiguration.js
- ✅ AI Content Generation - aiContentService.js, azureOpenAIService.js
- ✅ Weight Matrix - WeightMatrix.js, WeightMatrixManagement.jsx
- ✅ SAP/CorpBase Export - sapExport/, corpbaseExport/ services

### Schema Fields Retained (Used in Data Export Only)

These fields exist in the schema and are used in data export but have no UI:
- `regulatoryInfo` - Used in dataExportService.js for Excel export
- `launchTimeline` - Used in dataExportService.js, pifExportService.js, sapLoaderGenerator.js

---

## Documentation Index

### 1. Root Documentation

#### README.md
- **Path:** `/README.md`
- **Version:** 1.3.0
- **Last Updated:** 2025-12-13
- **Accuracy:** ✅ Current
- **Outline:**
  1. Overview
  2. Features (Core, Chemical/Biological, SAP & CorpBase, Additional)
  3. Tech Stack (Backend, Frontend)
  4. Project Structure
  5. Documentation Links
  6. Getting Started
  7. Environment Variables
  8. User Roles & Access
  9. API Endpoints
  10. Logging
  11. Development
  12. Deployment
  13. Security Features
  14. License

#### docs/README.md
- **Path:** `/docs/README.md`
- **Version:** 1.3.0
- **Last Updated:** 2025-12-13
- **Accuracy:** ✅ Current
- **Outline:**
  1. Documentation Index
  2. Quick Links for Common Tasks
  3. Key Features Summary
  4. Documentation Structure
  5. Common Questions
  6. Contributing to Documentation

---

### 2. Guides

#### Setup Guide
- **Path:** `/docs/guides/SETUP_GUIDE.md`
- **Version:** Not specified
- **Accuracy:** ✅ Current

#### Maintenance Guide
- **Path:** `/docs/MAINTENANCE_GUIDE.md`
- **Version:** 1.2
- **Last Updated:** December 5, 2025
- **Accuracy:** ✅ Current

---

### 3. API Documentation

#### API Documentation
- **Path:** `/docs/api/API_DOCUMENTATION.md`
- **Version:** 1.3.0
- **Last Updated:** 2025-12-13
- **Accuracy:** ✅ Current

#### API Quick Start
- **Path:** `/docs/api/API_QUICKSTART.md`
- **Accuracy:** ✅ Current

#### API Key Setup
- **Path:** `/docs/api/API_KEY_SETUP.md`
- **Version:** 2.0.0
- **Accuracy:** ✅ Current

#### API Key Changelog
- **Path:** `/docs/api/CHANGELOG_API_KEYS.md`
- **Version:** 2.0.0
- **Last Updated:** 2025-10-13
- **Accuracy:** ✅ Current

---

### 4. Architecture Documentation

#### Architecture Overview
- **Path:** `/docs/architecture/ARCHITECTURE.md`
- **Version:** 1.2.0
- **Last Updated:** 2025-12-08
- **Accuracy:** ✅ Current

#### Data Storage
- **Path:** `/docs/architecture/DATA_STORAGE.md`
- **Last Updated:** 2025-12-08
- **Accuracy:** ✅ Current

#### Ticket Data Flow
- **Path:** `/docs/architecture/TICKET_DATA_FLOW.md`
- **Accuracy:** ✅ Current

---

### 5. Feature Documentation

| Document | Version | Status |
|----------|---------|--------|
| TEMPLATE_VERSIONING.md | - | ✅ Current |
| PLANT_CODES.md | - | ✅ Current |
| NPDI_WORKFLOW.md | - | ✅ Current |
| PMOPS_SKU_WORKFLOW.md | - | ✅ Current |
| QUALITY_SPEC_PARSER.md | 1.0 | ✅ Current |
| AI_CONTENT_GENERATION.md | 1.0.0 | ✅ Current |
| AI_CONTENT_FIELDS.md | - | ✅ Current |
| AZURE_OPENAI_SETUP.md | - | ✅ Current |
| AZURE_OPENAI_MODELS.md | - | ✅ Current |
| SIMILAR_PRODUCTS_SEARCH.md | 1.1 | ✅ Current |
| COLLABORATIONS.md | - | ✅ Current |
| FEEDBACK_SYSTEM.md | - | ✅ Current |

---

### 6. Integration Documentation

| Document | Version | Status |
|----------|---------|--------|
| TEAMS_INTEGRATION.md | 1.0 | ✅ Current |
| Palantir-SQL-Query-API-Integration-Guide.md | 1.0 | ✅ Current |

---

### 7. Security Documentation

| Document | Version | Status |
|----------|---------|--------|
| API_KEY_STORAGE.md | - | ✅ Current |
| DEPENDENCY_SECURITY_ASSESSMENT.md | 1.1 | ✅ Current |

---

### 8. Data Reference Documentation

| Document | Version | Status |
|----------|---------|--------|
| SAP-MARA-to-ProductTicket-Mapping.md | 1.1 | ✅ Current |

---

### 9. Analysis Documentation

| Document | Version | Status |
|----------|---------|--------|
| SAP_LOADER_FIELD_MAPPING_ANALYSIS.md | 2.0 | ✅ Current |

---

## Version Summary

| Document | Version |
|----------|---------|
| README.md | 1.3.0 |
| docs/README.md | 1.3.0 |
| API Documentation | 1.3.0 |
| API Key Setup | 2.0.0 |
| API Key Changelog | 2.0.0 |
| Architecture Overview | 1.2.0 |
| AI Content Generation | 1.0.0 |
| Similar Products Search | 1.1 |
| Quality Spec Parser | 1.0 |
| Teams Integration | 1.0 |
| Palantir SQL Query API | 1.0 |
| Dependency Security Assessment | 1.1 |
| SAP MARA Mapping | 1.1 |
| SAP Loader Field Mapping Analysis | 2.0 |
| Maintenance Guide | 1.2 |

---

## File Structure

```
docs/
├── README.md                                    # Documentation Index (v1.3.0)
├── MAINTENANCE_GUIDE.md                         # System operations guide (v1.2)
├── TEMPLATE_VERSIONING.md                       # Template system guide
├── PLANT_CODES.md                               # Plant code reference
├── Palantir-SQL-Query-API-Integration-Guide.md  # Palantir integration (v1.0)
├── api/
│   ├── API_DOCUMENTATION.md                     # Complete API reference (v1.3.0)
│   ├── API_QUICKSTART.md                        # Quick start guide
│   ├── API_KEY_SETUP.md                         # API key management (v2.0.0)
│   └── CHANGELOG_API_KEYS.md                    # API changelog (v2.0.0)
├── architecture/
│   ├── ARCHITECTURE.md                          # Architecture overview (v1.2.0)
│   ├── DATA_STORAGE.md                          # MongoDB/Mongoose patterns
│   └── TICKET_DATA_FLOW.md                      # Data flow diagrams
├── features/
│   ├── AI_CONTENT_FIELDS.md                     # AI field mapping
│   ├── AI_CONTENT_GENERATION.md                 # AI features (v1.0.0)
│   ├── AZURE_OPENAI_MODELS.md                   # Model reference
│   ├── AZURE_OPENAI_SETUP.md                    # Azure setup
│   ├── COLLABORATIONS.md                        # Ticket collaboration
│   ├── FEEDBACK_SYSTEM.md                       # User feedback system
│   ├── NPDI_WORKFLOW.md                         # Workflow docs
│   ├── PMOPS_SKU_WORKFLOW.md                    # SKU management
│   ├── QUALITY_SPEC_PARSER.md                   # Quality spec parser (v1.0)
│   ├── SAP-MARA-to-ProductTicket-Mapping.md     # Field mapping (v1.1)
│   └── SIMILAR_PRODUCTS_SEARCH.md               # Similar products (v1.1)
├── guides/
│   └── SETUP_GUIDE.md                           # Installation guide
├── integrations/
│   └── TEAMS_INTEGRATION.md                     # Teams webhooks (v1.0)
├── security/
│   ├── API_KEY_STORAGE.md                       # API key security
│   └── DEPENDENCY_SECURITY_ASSESSMENT.md        # Dependency security (v1.1)
├── analysis/
│   └── SAP_LOADER_FIELD_MAPPING_ANALYSIS.md     # SAP field analysis (v2.0)
└── internal/
    └── DOCUMENTATION_INVENTORY.md               # This file
```

---

**Document Generated:** 2025-12-13
**Review Completed By:** NPDI Development Team
**Next Review:** 2026-01-13
