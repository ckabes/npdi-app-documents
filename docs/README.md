# NPDI Portal Documentation

**Version:** 1.3.0
**Last Updated:** 2025-12-13

Welcome to the NPDI (New Product Development & Introduction) Portal documentation.

## Documentation Index

### Getting Started
- [Main README](../README.md) - Project overview and quick start
- [Setup Guide](guides/SETUP_GUIDE.md) - Installation and configuration
- [Maintenance Guide](MAINTENANCE_GUIDE.md) - System operations and troubleshooting

### API Documentation
- [API Documentation](api/API_DOCUMENTATION.md) - Complete API reference
- [API Quick Start](api/API_QUICKSTART.md) - Getting started with the API
- [API Key Setup](api/API_KEY_SETUP.md) - How to generate and use API keys
- [API Key Changelog](api/CHANGELOG_API_KEYS.md) - Version history for API features

### Architecture
- [Architecture Overview](architecture/ARCHITECTURE.md) - System design and components
- [Data Storage](architecture/DATA_STORAGE.md) - MongoDB/Mongoose data architecture and patterns
- [Ticket Data Flow](architecture/TICKET_DATA_FLOW.md) - Models, routes, and controllers interaction

### Features
- [Template Versioning](TEMPLATE_VERSIONING.md) - Ticket template system and versioning
- [PMOps SKU Workflow](features/PMOPS_SKU_WORKFLOW.md) - Part number assignment and SKU management
- [Quality Spec Parser](features/QUALITY_SPEC_PARSER.md) - Natural language parser for quality specifications
- [AI Content Generation](features/AI_CONTENT_GENERATION.md) - Azure OpenAI integration
- [AI Content Fields Reference](features/AI_CONTENT_FIELDS.md) - Field mapping and configuration
- [Azure OpenAI Setup](features/AZURE_OPENAI_SETUP.md) - Configuration guide
- [Azure OpenAI Models](features/AZURE_OPENAI_MODELS.md) - Available models
- [Similar Products Search](features/SIMILAR_PRODUCTS_SEARCH.md) - Find related products by CAS number
- [NPDI Workflow](features/NPDI_WORKFLOW.md) - Ticket workflow documentation
- [Collaborations](features/COLLABORATIONS.md) - Ticket collaboration and sharing
- [Feedback System](features/FEEDBACK_SYSTEM.md) - User feedback and bug reporting

### Integrations
- [Microsoft Teams Integration](integrations/TEAMS_INTEGRATION.md) - Teams webhook notifications
- [Palantir SQL Query API](Palantir-SQL-Query-API-Integration-Guide.md) - Query SAP MARA data from Palantir Foundry datasets

### Data Reference
- [SAP MARA to ProductTicket Mapping](features/SAP-MARA-to-ProductTicket-Mapping.md) - Field mapping from Palantir Foundry SAP MARA dataset
- [Plant Codes](PLANT_CODES.md) - Plant code reference data

### Security
- [API Key Storage](security/API_KEY_STORAGE.md) - How API keys are encrypted and stored
- [Dependency Security Assessment](security/DEPENDENCY_SECURITY_ASSESSMENT.md) - Security analysis of dependencies

### Analysis
- [SAP Loader Field Mapping Analysis](analysis/SAP_LOADER_FIELD_MAPPING_ANALYSIS.md) - SAP MDG-M field mapping

---

## Quick Links for Common Tasks

### For Developers
1. **First Time Setup**: Start with [Setup Guide](guides/SETUP_GUIDE.md)
2. **API Integration**: Read [API Quick Start](api/API_QUICKSTART.md)
3. **Understanding the System**: Review [Architecture Overview](architecture/ARCHITECTURE.md)
4. **Template System**: See [Template Versioning](TEMPLATE_VERSIONING.md)

### For Administrators
1. **Managing API Access**: See [API Key Setup](api/API_KEY_SETUP.md)
2. **Teams Notifications**: Configure using [Teams Integration](integrations/TEAMS_INTEGRATION.md)
3. **Palantir/SAP Integration**: [Palantir SQL Query API](Palantir-SQL-Query-API-Integration-Guide.md)

### For IT/Infrastructure
1. **System Maintenance**: [Maintenance Guide](MAINTENANCE_GUIDE.md)
2. **API Key Security**: [API Key Storage](security/API_KEY_STORAGE.md)

---

## Key Features Summary

- **Ticket Management**: Full CRUD with status workflow (DRAFT → SUBMITTED → IN_PROCESS → NPDI_INITIATED → COMPLETED)
- **Template System**: Discriminator pattern for Chemistry and Biologics templates
- **Collaborations**: Share tickets with other users for collaborative editing
- **Feedback System**: In-app bug reporting and feature requests
- **REST API**: External integration with API key authentication
- **PubChem Integration**: Automatic chemical data population
- **Palantir Foundry Integration**: Query SAP MARA data stored in Palantir Foundry datasets
- **Similar Products Search**: Find related products by CAS number
- **Quality Spec Parser**: Natural language to structured data conversion
- **AI Content Generation**: Azure OpenAI for product descriptions
- **Teams Notifications**: Webhook notifications for ticket events
- **SAP MDG-M Loader Export**: Generate SAP Material Master Data loader files
- **Excel Export**: PDP Checklists and Product Information Forms
- **Structured Logging**: Daily rotation with log levels

---

## Documentation Structure

```
docs/
├── README.md                         # This file (documentation index)
├── MAINTENANCE_GUIDE.md              # Operations and troubleshooting
├── TEMPLATE_VERSIONING.md            # Template system guide
├── PLANT_CODES.md                    # Plant code reference
├── Palantir-SQL-Query-API-Integration-Guide.md
├── api/                              # API documentation
│   ├── API_DOCUMENTATION.md          # Complete API reference
│   ├── API_QUICKSTART.md             # Quick start guide
│   ├── API_KEY_SETUP.md              # API key management
│   └── CHANGELOG_API_KEYS.md         # API changelog
├── architecture/                     # System architecture
│   ├── ARCHITECTURE.md               # Architecture overview
│   ├── DATA_STORAGE.md               # MongoDB/Mongoose data architecture
│   └── TICKET_DATA_FLOW.md           # Data flow diagrams
├── features/                         # Feature documentation
│   ├── PMOPS_SKU_WORKFLOW.md         # SKU management
│   ├── QUALITY_SPEC_PARSER.md        # Quality spec parser
│   ├── AI_CONTENT_GENERATION.md      # AI features
│   ├── AI_CONTENT_FIELDS.md          # AI field mapping
│   ├── AZURE_OPENAI_SETUP.md         # Azure setup
│   ├── AZURE_OPENAI_MODELS.md        # Model reference
│   ├── NPDI_WORKFLOW.md              # Workflow docs
│   ├── SIMILAR_PRODUCTS_SEARCH.md    # Similar products
│   ├── COLLABORATIONS.md             # Ticket collaboration
│   ├── FEEDBACK_SYSTEM.md            # User feedback system
│   └── SAP-MARA-to-ProductTicket-Mapping.md
├── guides/                           # How-to guides
│   └── SETUP_GUIDE.md                # Installation guide
├── integrations/                     # Third-party integrations
│   └── TEAMS_INTEGRATION.md          # Teams webhooks
├── security/                         # Security documentation
│   ├── API_KEY_STORAGE.md            # API key security
│   └── DEPENDENCY_SECURITY_ASSESSMENT.md
├── analysis/                         # Analysis documents
│   └── SAP_LOADER_FIELD_MAPPING_ANALYSIS.md
└── internal/                         # Internal documentation
    └── DOCUMENTATION_INVENTORY.md    # Documentation inventory
```

---

## Common Questions

**Q: How do I set up the development environment?**
A: See [Setup Guide](guides/SETUP_GUIDE.md)

**Q: How do I configure Teams notifications?**
A: Follow [Teams Integration](integrations/TEAMS_INTEGRATION.md)

**Q: How do I use the REST API?**
A: Begin with [API Quick Start](api/API_QUICKSTART.md)

**Q: What's the system architecture?**
A: Review [Architecture Overview](architecture/ARCHITECTURE.md)

**Q: How is data stored in the database?**
A: See [Data Storage](architecture/DATA_STORAGE.md)

**Q: How do I query SAP MARA data from Palantir Foundry?**
A: See [Palantir SQL Query API Integration](Palantir-SQL-Query-API-Integration-Guide.md)

**Q: How does the template system work?**
A: See [Template Versioning](TEMPLATE_VERSIONING.md)

**Q: How do I maintain and troubleshoot the system?**
A: See [Maintenance Guide](MAINTENANCE_GUIDE.md)

**Q: How does PMOps manage SKUs and part numbers?**
A: See [PMOps SKU Workflow](features/PMOPS_SKU_WORKFLOW.md)

**Q: How do I use the Quality Specifications parser?**
A: See [Quality Spec Parser](features/QUALITY_SPEC_PARSER.md)

---

## Contributing to Documentation

When adding new documentation:

1. **Choose the right folder:**
   - API-related: `docs/api/`
   - Architecture/design: `docs/architecture/`
   - Integration guides: `docs/integrations/`
   - How-to guides: `docs/guides/`
   - Security: `docs/security/`
   - Features: `docs/features/`

2. **Follow naming conventions:**
   - Use UPPERCASE for main docs (e.g., `FEATURE_NAME.md`)
   - Use descriptive names

3. **Update this index:**
   - Add your new document to the appropriate section
   - Include a brief description
   - Link to the document

4. **Use markdown best practices:**
   - Clear headings (H1, H2, H3)
   - Table of contents for long docs
   - Code examples in fenced code blocks
   - Links to related documentation

---

**Last Updated:** 2025-12-13
**Maintained by:** NPDI Development Team
