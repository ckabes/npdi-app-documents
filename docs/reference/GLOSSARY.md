# NPDI Application Glossary

**Version:** 1.0.0
**Last Updated:** 2026-01-09

---

## Table of Contents

1. [Business Terms](#business-terms)
2. [Product Terms](#product-terms)
3. [Technical Terms](#technical-terms)
4. [Workflow Terms](#workflow-terms)
5. [Integration Terms](#integration-terms)
6. [Abbreviations](#abbreviations)

---

## Business Terms

### NPDI
**New Product Development and Introduction**

The formal process at MilliporeSigma for introducing new products to the catalog. This application serves as the intake system that collects product information before submission to the official NPDI system.

### PMOps / PM-OPS
**Product Management Operations**

The operations team responsible for processing submitted product tickets, assigning SKUs, managing pricing, and coordinating with other departments for product setup in SAP.

### Product Manager
A user role responsible for creating and submitting new product requests. Product Managers fill out ticket forms with product details and submit them to PMOps for processing.

### SBU
**Strategic Business Unit**

Organizational divisions within MilliporeSigma. Products are categorized by SBU:
- **775** - Life Science
- **P90** - Process Solutions
- **440** - Electronics
- **P87** - Healthcare
- **P89** - (Additional SBU)
- **P85** - (Additional SBU)

### Business Line
A subdivision within an SBU representing a product category or market segment. Examples include `DM1` (Chemistry), `AS6` (Biologics/Antibodies).

### GPH
**Global Product Hierarchy**

A classification system for organizing products. GPH Product Line (`YYD_GPHPL`) is used for SAP categorization.

---

## Product Terms

### Ticket
A product development request containing all information needed to create a new product in the catalog. Tickets progress through a workflow from DRAFT to COMPLETED.

### Template
A form configuration that defines what fields are available for a ticket type. Templates use the discriminator pattern to support different product categories:
- **DM1-CHEM** - Chemistry products
- **AS6-BIOLOGIC** - Biological products (antibodies, proteins)

### SKU
**Stock Keeping Unit**

A unique identifier for a specific product variant. SKU types include:
- **BULK** - Bulk quantity
- **PREPACK** - Pre-packaged sizes (100G, 1KG, etc.)
- **CONF** - Configured variant (-CONF suffix)
- **SPEC** - Specification variant (-SPEC suffix)
- **VAR** - Generic variant (-VAR suffix)

### Part Number / Base Number
The primary identifier assigned to a product by PMOps. Format varies by product type. SKU variants append suffixes to this base number.

### CAS Number
**Chemical Abstracts Service Registry Number**

A unique numerical identifier for chemical substances. Format: `XXXXX-XX-X` (e.g., `64-17-5` for ethanol).

### Material Number
The SAP system identifier for a product. Distinct from the Part Number used externally.

### UNSPSC
**United Nations Standard Products and Services Code**

A global classification system for products and services used for e-commerce and procurement.

---

## Technical Terms

### Discriminator Pattern
A MongoDB/Mongoose pattern where multiple document types share a collection but have type-specific schemas. The `ticketType` field determines which schema is applied:

```
BaseTicket (common fields)
├── DM1ChemTicket (chemistry fields)
└── AS6BiologicTicket (biology fields)
```

### Embedded Document
In MongoDB, a document nested within another document. Used for complex data like `chemicalProperties`, `skuVariants`, etc., rather than creating separate collections.

### Form Configuration
Database-driven form field definitions that control what appears in ticket forms. Enables dynamic UI without code changes.

### Parser Configuration
Knowledge base entries used by the Quality Specification Natural Language Parser to normalize test attributes and methods.

### Seed Script
A script that populates the database with initial data. Examples:
- `seedFormConfig.js` - Form field definitions
- `seedBiologicFormConfig.js` - Biologic template configuration

---

## Workflow Terms

### Ticket Status

| Status | Description |
|--------|-------------|
| **DRAFT** | Initial state. Ticket is being created/edited by Product Manager. |
| **SUBMITTED** | Product Manager has submitted for review. Awaiting PMOps processing. |
| **IN_PROCESS** | PMOps is actively working on the ticket (assigning SKUs, etc.). |
| **NPDI_INITIATED** | Ticket has been submitted to the external NPDI system. |
| **COMPLETED** | All processing complete. Product is live in catalog. |
| **CANCELED** | Ticket was canceled (with reason recorded). |

### Status History
An audit trail recording all status changes, who made them, when, and why. Stored as an embedded array in each ticket.

### Priority Levels

| Priority | Description |
|----------|-------------|
| **LOW** | Standard processing |
| **MEDIUM** | Normal priority (default) |
| **HIGH** | Expedited processing needed |
| **URGENT** | Requires immediate attention |

### Collaborator
A user granted access to view or edit a ticket they didn't create. Used for cross-team collaboration.

---

## Integration Terms

### PubChem
A free chemistry database from NIH/NCBI. The application integrates with PubChem to auto-populate chemical data from CAS numbers.

### Palantir Foundry
A data integration platform used at MilliporeSigma. The NPDI application queries SAP MARA data stored in Palantir datasets.

### SAP
**Systems, Applications, and Products**

Enterprise resource planning (ERP) software. Key SAP tables referenced:
- **MARA** - General Material Data (master material table)
- **MARC** - Plant Data for Material
- **MVKE** - Sales Data for Material
- **MBEW** - Material Valuation

### MARA
The primary SAP material master table. Contains core product information that can be imported into NPDI tickets.

### SAP MDG-M Loader
**Master Data Governance - Material**

An SAP mechanism for bulk importing material master data. The application generates loader files in the required format.

### CorpBase
The corporate website/e-commerce platform where product information is published. The `corpbaseData` section contains marketing content.

### LangDock / Azure OpenAI
AI services used for generating product descriptions, key features, and other marketing content. LangDock is the enterprise gateway to Azure OpenAI services.

### Teams Webhook
Microsoft Teams integration for sending notifications about ticket events (submissions, status changes, etc.).

---

## Abbreviations

| Abbreviation | Full Term |
|--------------|-----------|
| API | Application Programming Interface |
| CAS | Chemical Abstracts Service |
| COA | Certificate of Analysis |
| CRUD | Create, Read, Update, Delete |
| ERP | Enterprise Resource Planning |
| GHS | Globally Harmonized System (of hazard classification) |
| GPH | Global Product Hierarchy |
| IP | Intellectual Property |
| MDG-M | Master Data Governance - Material |
| MERN | MongoDB, Express, React, Node.js |
| MQ | MilliporeSigma Quality (level designation) |
| NPDI | New Product Development and Introduction |
| ODM | Object Document Mapper (Mongoose) |
| PMOps | Product Management Operations |
| QC | Quality Control |
| RUO | Research Use Only |
| SAP | Systems, Applications, and Products |
| SBU | Strategic Business Unit |
| SKU | Stock Keeping Unit |
| SMTP | Simple Mail Transfer Protocol |
| TSCA | Toxic Substances Control Act |
| UI | User Interface |
| UOM | Unit of Measure |
| UNSPSC | United Nations Standard Products and Services Code |

---

## Quality & Regulatory Terms

### MQ Quality Levels

| Level | Description |
|-------|-------------|
| **MQ100** | Research grade |
| **MQ200** | Analytical grade |
| **MQ300** | High purity |
| **MQ400** | Ultra-high purity |
| **MQ500** | Pharmaceutical grade |
| **MQ600** | Highest purity available |

### GHS Class
**Globally Harmonized System** hazard classification ranges:
- **H200-H299** - Physical hazards
- **H300-H399** - Health hazards
- **H400-H499** - Environmental hazards

### Signal Words
GHS hazard severity indicators:
- **DANGER** - More severe hazards
- **WARNING** - Less severe hazards

### Production Type

| Type | Description |
|------|-------------|
| **Produced** | Manufactured internally |
| **Procured** | Purchased from external vendor |

### Distribution Type

| Type | Description |
|------|-------------|
| **Standard** | Regular distribution through normal channels |
| **Purchase on Demand** | Ordered specifically for customer |
| **Dock to Stock** | Direct from vendor to customer |

---

## Brand Names

MilliporeSigma operates under several brands:

| Brand | Focus Area |
|-------|------------|
| **Sigma-Aldrich** | Research chemicals, biochemicals |
| **SAFC** | Custom manufacturing, pharma |
| **Supelco** | Analytical standards, chromatography |
| **Milli-Q** | Water purification |
| **Millipore** | Filtration, life science |
| **BioReliance** | Biopharmaceutical testing |
| **Calbiochem** | Biochemicals, inhibitors |
| **Merck** | Parent company brand |

---

**Last Updated:** 2026-01-09
**Maintained by:** NPDI Development Team
