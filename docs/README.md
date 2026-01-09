# NPDI Portal Documentation

**Version:** 1.5.0
**Last Updated:** 2026-01-09

Welcome to the NPDI (New Product Development & Introduction) Portal documentation.

## Documentation Index

### Getting Started
- [Main README](../README.md) - Project overview and quick start
- [Setup Guide](guides/SETUP_GUIDE.md) - Installation and configuration
- [Glossary](reference/GLOSSARY.md) - Domain terms and abbreviations

### Development
- [Coding Standards](guides/CODING_STANDARDS.md) - Project conventions, naming patterns, file organization

### Operations & Maintenance
- [Maintenance Guide](MAINTENANCE_GUIDE.md) - System operations (for non-developers)
- [Troubleshooting Guide](guides/operations/TROUBLESHOOTING_GUIDE.md) - Common errors and solutions
- [Deployment Guide](guides/operations/DEPLOYMENT_GUIDE.md) - Production deployment
- [Backup and Recovery](guides/operations/BACKUP_AND_RECOVERY.md) - Backup procedures and disaster recovery
- [Logging and Monitoring](guides/operations/LOGGING_AND_MONITORING.md) - Log files, levels, analysis, alerts

### Business & Workflow
- [Business Rules](guides/BUSINESS_RULES.md) - Ticket workflow, status transitions, validation
- [NPDI Workflow](features/NPDI_WORKFLOW.md) - Ticket workflow documentation
- [PMOps SKU Workflow](features/PMOPS_SKU_WORKFLOW.md) - Part number assignment and SKU management

### API Documentation
- [API Documentation](api/API_DOCUMENTATION.md) - Complete API reference
- [API Quick Start](api/API_QUICKSTART.md) - Getting started with the API
- [API Key Setup](api/API_KEY_SETUP.md) - How to generate and use API keys
- [API Key Changelog](api/CHANGELOG_API_KEYS.md) - Version history for API features

### Architecture & Technical Reference
- [Architecture Overview](architecture/ARCHITECTURE.md) - System design and components
- [Data Storage](architecture/DATA_STORAGE.md) - MongoDB/Mongoose data architecture and patterns
- [State Management](architecture/STATE_MANAGEMENT.md) - React context, data flow, caching strategies
- [Ticket Data Flow](architecture/TICKET_DATA_FLOW.md) - Models, routes, and controllers interaction
- [Database Schema Reference](reference/DATABASE_SCHEMA_REFERENCE.md) - Complete MongoDB schema documentation
- [Component Catalog](reference/COMPONENT_CATALOG.md) - All 54 React components with props and usage
- [Environment Variables](reference/ENVIRONMENT_VARIABLES.md) - Configuration reference
- [Error Handling](reference/ERROR_HANDLING.md) - API error codes and response formats

### Features
- [Template Versioning](TEMPLATE_VERSIONING.md) - Ticket template system and versioning
- [Form Configuration](features/FORM_CONFIGURATION.md) - Dynamic form system and field configuration
- [Quality Spec Parser](features/QUALITY_SPEC_PARSER.md) - Natural language parser for quality specifications
- [AI Content Generation](features/AI_CONTENT_GENERATION.md) - Azure OpenAI integration
- [AI Content Fields Reference](features/AI_CONTENT_FIELDS.md) - Field mapping and configuration
- [Azure OpenAI Setup](features/AZURE_OPENAI_SETUP.md) - Configuration guide
- [Azure OpenAI Models](features/AZURE_OPENAI_MODELS.md) - Available models
- [Similar Products Search](features/SIMILAR_PRODUCTS_SEARCH.md) - Find related products by CAS number
- [Collaborations](features/COLLABORATIONS.md) - Ticket collaboration and sharing
- [Feedback System](features/FEEDBACK_SYSTEM.md) - User feedback and bug reporting

### Integrations
- [Microsoft Teams Integration](integrations/TEAMS_INTEGRATION.md) - Teams webhook notifications
- [Palantir SQL Query API](Palantir-SQL-Query-API-Integration-Guide.md) - Query SAP MARA data from Palantir Foundry datasets

### Data Reference
- [SAP MARA to ProductTicket Mapping](features/SAP-MARA-to-ProductTicket-Mapping.md) - Field mapping from Palantir Foundry SAP MARA dataset
- [Plant Codes](PLANT_CODES.md) - Plant code reference data
- [Glossary](reference/GLOSSARY.md) - Domain terms (NPDI, PMOps, SKU, SBU, MARA, etc.)

### Security
- [API Key Storage](security/API_KEY_STORAGE.md) - How API keys are encrypted and stored
- [NPDI Dependencies](security/NPDI_DEPENDENCIES.md) - Application dependency inventory and characteristics

### Analysis
- [SAP Loader Field Mapping Analysis](analysis/SAP_LOADER_FIELD_MAPPING_ANALYSIS.md) - SAP MDG-M field mapping

---

## Quick Links for Common Tasks

### For New Developers
1. **First Time Setup**: Start with [Setup Guide](guides/SETUP_GUIDE.md)
2. **Understand the Domain**: Read [Glossary](reference/GLOSSARY.md)
3. **Coding Standards**: Review [Coding Standards](guides/CODING_STANDARDS.md)
4. **Learn the Architecture**: Review [Architecture Overview](architecture/ARCHITECTURE.md)
5. **Database Schema**: See [Database Schema Reference](reference/DATABASE_SCHEMA_REFERENCE.md)
6. **React Components**: See [Component Catalog](reference/COMPONENT_CATALOG.md)
7. **Business Logic**: Read [Business Rules](guides/BUSINESS_RULES.md)

### For Ongoing Development
1. **API Integration**: Read [API Quick Start](api/API_QUICKSTART.md)
2. **State Management**: See [State Management](architecture/STATE_MANAGEMENT.md)
3. **Form System**: See [Form Configuration](features/FORM_CONFIGURATION.md)
4. **Template System**: See [Template Versioning](TEMPLATE_VERSIONING.md)
5. **Error Handling**: See [Error Handling](reference/ERROR_HANDLING.md)
6. **Environment Config**: See [Environment Variables](reference/ENVIRONMENT_VARIABLES.md)

### For Administrators
1. **Managing API Access**: See [API Key Setup](api/API_KEY_SETUP.md)
2. **Teams Notifications**: Configure using [Teams Integration](integrations/TEAMS_INTEGRATION.md)
3. **Palantir/SAP Integration**: [Palantir SQL Query API](Palantir-SQL-Query-API-Integration-Guide.md)

### For IT/Infrastructure
1. **Production Deployment**: [Deployment Guide](guides/operations/DEPLOYMENT_GUIDE.md)
2. **System Maintenance**: [Maintenance Guide](MAINTENANCE_GUIDE.md)
3. **Logging & Monitoring**: [Logging and Monitoring](guides/operations/LOGGING_AND_MONITORING.md)
4. **Troubleshooting**: [Troubleshooting Guide](guides/operations/TROUBLESHOOTING_GUIDE.md)
5. **Backup Procedures**: [Backup and Recovery](guides/operations/BACKUP_AND_RECOVERY.md)
6. **API Key Security**: [API Key Storage](security/API_KEY_STORAGE.md)

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
├── README.md                              # This file (documentation index)
├── MAINTENANCE_GUIDE.md                   # Operations and troubleshooting (non-dev)
├── TEMPLATE_VERSIONING.md                 # Template system guide
├── PLANT_CODES.md                         # Plant code reference
├── Palantir-SQL-Query-API-Integration-Guide.md
│
├── api/                                   # API documentation
│   ├── API_DOCUMENTATION.md               # Complete API reference
│   ├── API_QUICKSTART.md                  # Quick start guide
│   ├── API_KEY_SETUP.md                   # API key management
│   └── CHANGELOG_API_KEYS.md              # API changelog
│
├── architecture/                          # System architecture
│   ├── ARCHITECTURE.md                    # Architecture overview
│   ├── DATA_STORAGE.md                    # MongoDB/Mongoose data architecture
│   ├── STATE_MANAGEMENT.md                # React context and data flow
│   └── TICKET_DATA_FLOW.md                # Data flow diagrams
│
├── features/                              # Feature documentation
│   ├── PMOPS_SKU_WORKFLOW.md              # SKU management
│   ├── QUALITY_SPEC_PARSER.md             # Quality spec parser
│   ├── FORM_CONFIGURATION.md              # Dynamic form system
│   ├── AI_CONTENT_GENERATION.md           # AI features
│   ├── AI_CONTENT_FIELDS.md               # AI field mapping
│   ├── AZURE_OPENAI_SETUP.md              # Azure setup
│   ├── AZURE_OPENAI_MODELS.md             # Model reference
│   ├── NPDI_WORKFLOW.md                   # Workflow docs
│   ├── SIMILAR_PRODUCTS_SEARCH.md         # Similar products
│   ├── COLLABORATIONS.md                  # Ticket collaboration
│   ├── FEEDBACK_SYSTEM.md                 # User feedback system
│   └── SAP-MARA-to-ProductTicket-Mapping.md
│
├── guides/                                # How-to guides
│   ├── SETUP_GUIDE.md                     # Installation guide
│   ├── CODING_STANDARDS.md                # Project conventions and patterns
│   ├── BUSINESS_RULES.md                  # Workflow and validation rules
│   └── operations/                        # Operational guides
│       ├── TROUBLESHOOTING_GUIDE.md       # Error diagnosis and solutions
│       ├── DEPLOYMENT_GUIDE.md            # Production deployment
│       ├── BACKUP_AND_RECOVERY.md         # Backup and disaster recovery
│       └── LOGGING_AND_MONITORING.md      # Log files and monitoring
│
├── integrations/                          # Third-party integrations
│   └── TEAMS_INTEGRATION.md               # Teams webhooks
│
├── reference/                             # Reference documentation
│   ├── DATABASE_SCHEMA_REFERENCE.md       # MongoDB schema docs
│   ├── COMPONENT_CATALOG.md               # React component reference
│   ├── ERROR_HANDLING.md                  # API error codes and formats
│   ├── ENVIRONMENT_VARIABLES.md           # Environment configuration
│   └── GLOSSARY.md                        # Domain terminology
│
├── security/                              # Security documentation
│   ├── API_KEY_STORAGE.md                 # API key security
│   └── NPDI_DEPENDENCIES.md               # Dependency inventory
│
└── analysis/                              # Analysis documents
    └── SAP_LOADER_FIELD_MAPPING_ANALYSIS.md
```

---

## Common Questions

**Q: How do I set up the development environment?**
A: See [Setup Guide](guides/SETUP_GUIDE.md)

**Q: What do all these acronyms mean (NPDI, PMOps, SKU, etc.)?**
A: See [Glossary](reference/GLOSSARY.md)

**Q: How do I configure Teams notifications?**
A: Follow [Teams Integration](integrations/TEAMS_INTEGRATION.md)

**Q: How do I use the REST API?**
A: Begin with [API Quick Start](api/API_QUICKSTART.md)

**Q: What's the system architecture?**
A: Review [Architecture Overview](architecture/ARCHITECTURE.md)

**Q: How is data stored in the database?**
A: See [Data Storage](architecture/DATA_STORAGE.md) and [Database Schema Reference](reference/DATABASE_SCHEMA_REFERENCE.md)

**Q: What environment variables do I need?**
A: See [Environment Variables](reference/ENVIRONMENT_VARIABLES.md)

**Q: How do I query SAP MARA data from Palantir Foundry?**
A: See [Palantir SQL Query API Integration](Palantir-SQL-Query-API-Integration-Guide.md)

**Q: How does the template system work?**
A: See [Template Versioning](TEMPLATE_VERSIONING.md)

**Q: How do I troubleshoot errors?**
A: See [Troubleshooting Guide](guides/operations/TROUBLESHOOTING_GUIDE.md)

**Q: How do I deploy to production?**
A: See [Deployment Guide](guides/operations/DEPLOYMENT_GUIDE.md)

**Q: How do I back up the database?**
A: See [Backup and Recovery](guides/operations/BACKUP_AND_RECOVERY.md)

**Q: What are the business rules for ticket workflows?**
A: See [Business Rules](guides/BUSINESS_RULES.md)

**Q: How does PMOps manage SKUs and part numbers?**
A: See [PMOps SKU Workflow](features/PMOPS_SKU_WORKFLOW.md)

**Q: How do I use the Quality Specifications parser?**
A: See [Quality Spec Parser](features/QUALITY_SPEC_PARSER.md)

**Q: What are the project coding standards?**
A: See [Coding Standards](guides/CODING_STANDARDS.md)

**Q: How does the dynamic form system work?**
A: See [Form Configuration](features/FORM_CONFIGURATION.md)

**Q: How does React state management work in this app?**
A: See [State Management](architecture/STATE_MANAGEMENT.md)

**Q: What React components are available?**
A: See [Component Catalog](reference/COMPONENT_CATALOG.md)

**Q: Where are log files and how do I analyze them?**
A: See [Logging and Monitoring](guides/operations/LOGGING_AND_MONITORING.md)

**Q: How should I handle API errors?**
A: See [Error Handling](reference/ERROR_HANDLING.md)

---

## Contributing to Documentation

When adding new documentation:

1. **Choose the right folder:**
   - API-related: `docs/api/`
   - Architecture/design: `docs/architecture/`
   - Integration guides: `docs/integrations/`
   - How-to guides: `docs/guides/`
   - Operational guides: `docs/guides/operations/`
   - Reference material: `docs/reference/`
   - Security: `docs/security/`
   - Features: `docs/features/`

2. **Follow naming conventions:**
   - Use UPPERCASE for main docs (e.g., `FEATURE_NAME.md`)
   - Use descriptive names

3. **Include standard header:**
   ```markdown
   # Document Title

   **Version:** 1.0.0
   **Last Updated:** YYYY-MM-DD
   ```

4. **Update this index:**
   - Add your new document to the appropriate section
   - Include a brief description
   - Link to the document

5. **Use markdown best practices:**
   - Clear headings (H1, H2, H3)
   - Table of contents for long docs
   - Code examples in fenced code blocks
   - Links to related documentation

---

**Last Updated:** 2026-01-09
**Maintained by:** NPDI Development Team
