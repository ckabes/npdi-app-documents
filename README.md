# MilliporeSigma NPDI Portal

**Version:** 1.3.0
**Last Updated:** 2025-12-13

A New Product Development and Introduction (NPDI) ticket initiation application for capturing product data and facilitating workflow between Product Managers and PMOps at MilliporeSigma.

## Features

### Core Functionality
- **Product Ticket Management**: Create and manage development tickets for new chemical and biological products
- **Multi-SKU Support**: Handle prepack, -CONF, -SPEC, and -VAR SKUs per product
- **Profile-Based Access**: Separate interfaces for Product Managers, PM-OPS team, and Administrators
- **Status Workflow**: DRAFT → SUBMITTED → IN_PROCESS → NPDI_INITIATED → COMPLETED/CANCELED with audit trail
- **NPDI Integration**: Seamless transition to official NPDI system with ticket number synchronization
- **SBU Organization**: Tickets organized by Strategic Business Unit
- **Dynamic Form Configuration**: Database-driven form rendering with seed script management
- **Template-Based Ticket System**: Versioned ticket templates with semantic versioning
  - Current templates: DM1-CHEM (Chemistry), AS6-BIOLOGIC (Biologics)
  - Templates use semantic versioning (Major.Minor.Patch)
  - Template names are fully dynamic - loaded from database
  - Template-specific ticket views with discriminator pattern

### Chemical & Biological Data Management
- **Chemical Properties**: CAS numbers, molecular formulas, physical states, purity ranges
- **Biological Properties**: Host organisms, expression systems, protein sequences
- **Molecular Structure Viewer**: Professional 2D structure rendering using RDKit-JS
- **Quality Specifications Parser**: Natural language input with automatic formatting
- **Quality Tests Configuration**: Database-backed parser knowledge base (303 entries)
- **Hazard Classification**: GHS classes, signal words, transport classifications
- **Storage Conditions**: Temperature, humidity, light, and atmosphere requirements
- **PubChem Integration**: Automatic chemical data population from CAS numbers
- **Palantir Foundry Integration**: Query SAP MARA data stored in Palantir Foundry datasets via SQL Query API v2
- **Similar Products Search**: Find related products by CAS number
- **Weight Matrix Management**: Package size to weight conversion for SKU variants
- **Plant Codes & Business Lines**: Centralized hierarchy management
- **UNSPSC Classification**: Product classification tracking

### SAP & CorpBase Integration
- **SAP MDG-M Loader Export**: Generate SAP Material Master Data loader files
- **CorpBase Batch Client Export**: Generate CorpBase loader files for Sigma-Aldrich website
- **CorpBase Website Preview**: Live preview of product pages as they will appear on sigmaaldrich.com
- **CorpBase Definitions Management**: Admin UI for managing CorpBase field definitions
- **Attribute Fixed Text Management**: Admin UI for SAP attribute fixed text values

### Additional Features
- **Ticket Collaborations**: Share tickets with other users for collaborative editing
- **Commenting System**: Collaborative discussions with user attribution
- **User Feedback System**: In-app bug reporting and feature requests
- **Dashboard Analytics**: Statistics on ticket volumes, cycle times, SBU breakdowns
- **REST API**: External integration with API key authentication
- **User Preferences**: Customizable dashboard layouts and notification settings
- **System Settings**: Configurable security policies and integrations
- **Microsoft Teams Integration**: Webhook notifications for ticket events
- **AI Content Generation**: Azure OpenAI for CorpBase product descriptions
- **Data Export**: Excel export for PDP Checklists, PIF forms, and SAP/CorpBase loaders
- **Structured Logging**: File-based logging with daily rotation

## Tech Stack

### Backend
- **Node.js** with Express.js 5.x framework
- **MongoDB** with Mongoose 9.x ODM
  - Discriminator pattern for template-specific schemas
  - Strategic compound indexes for query optimization
  - Aggregation pipelines for analytics
- **Custom Logger** with daily file rotation and log levels
- **In-Memory LRU Cache** for frequently-accessed data
- **Express Validator** for input validation
- **Helmet** and rate limiting for security
- **Axios** for external API integration
- **Apache Arrow** for Palantir binary data parsing
- **Azure OpenAI** for AI-powered content generation

### Frontend
- **React 18** with modern hooks
- **React Router** for navigation
- **Tailwind CSS** with custom MilliporeSigma branding
- **Vite** for fast development and builds
- **Axios** for API communication
- **React Hot Toast** for notifications
- **Headless UI** for accessible components
- **Heroicons** for UI icons
- **RDKit-JS** for molecular structure rendering

## Project Structure

```
npdi-app/
├── server/                          # Backend API
│   ├── config/                      # Database configuration
│   ├── controllers/                 # Route handlers and business logic
│   │   ├── products/                # Product-specific controllers (modular)
│   │   │   ├── ticketCRUDController.js    # Ticket CRUD operations
│   │   │   ├── integrationController.js   # External integrations (PubChem, SAP)
│   │   │   ├── dashboardController.js     # Dashboard statistics
│   │   │   ├── exportController.js        # Excel export functions
│   │   │   └── helpers.js                 # Shared helper functions
│   │   ├── productController.js     # Main product controller
│   │   ├── ticketApiController.js   # External API endpoints
│   │   ├── adminController.js       # Admin dashboard statistics
│   │   ├── apiKeyController.js      # API key management
│   │   ├── systemSettingsController.js
│   │   └── userPreferencesController.js
│   ├── data/                        # Development profiles
│   ├── middleware/                  # Auth, validation, API auth
│   ├── models/                      # Database schemas (Mongoose)
│   │   ├── discriminators/          # Template-specific schemas
│   │   │   ├── ChemicalTicket.js    # Chemistry template fields
│   │   │   └── BiologicTicket.js    # Biologics template fields
│   │   ├── schemas/                 # Shared sub-schemas
│   │   │   ├── commentSchema.js
│   │   │   ├── statusHistorySchema.js
│   │   │   └── skuVariantSchema.js
│   │   ├── BaseTicket.js            # Base ticket schema
│   │   ├── ProductTicket.js         # Main ticket model
│   │   ├── TicketFactory.js         # Factory for creating typed tickets
│   │   ├── TicketTemplate.js        # Template definitions
│   │   └── ...                      # Other models
│   ├── routes/                      # API endpoint definitions
│   ├── services/                    # External integrations & utilities
│   │   ├── cacheService.js          # In-memory LRU cache
│   │   ├── pubchemService.js        # PubChem chemical data
│   │   ├── palantirService.js       # Palantir Foundry dataset queries
│   │   ├── aiContentService.js      # AI content generation
│   │   ├── azureOpenAIService.js    # Azure OpenAI integration
│   │   ├── teamsNotificationService.js
│   │   ├── dataExportService.js     # Excel exports
│   │   ├── pdpChecklistExportService.js  # PDP Checklist Excel export
│   │   ├── pifExportService.js      # Product Information Form export
│   │   ├── sapExport/               # SAP MDG-M loader generation
│   │   │   └── sapLoaderGenerator.js
│   │   └── corpbaseExport/          # CorpBase Batch Client generation
│   │       └── corpbaseLoaderGenerator.js
│   ├── utils/                       # Helper utilities
│   │   ├── logger.js                # Custom logging system
│   │   ├── enumCleaner.js           # Enum validation
│   │   ├── crudControllerFactory.js # Generic CRUD factory
│   │   ├── submissionValidator.js   # Ticket validation
│   │   └── encryption.js            # Data encryption
│   └── index.js                     # Server entry point
├── client/                          # Frontend React app
│   └── src/
│       ├── components/              # Reusable UI components
│       │   ├── admin/               # Admin components
│       │   │   ├── SystemSettings.jsx
│       │   │   ├── UserManagement.jsx
│       │   │   ├── ApiKeyManagement.jsx
│       │   │   ├── GenericCRUDManager.jsx
│       │   │   ├── ParserKnowledgeManager.jsx
│       │   │   ├── FeedbackManagement.jsx
│       │   │   ├── CorpBaseDefinitionsManagement.jsx
│       │   │   ├── AttributeFixedTextManagement.jsx
│       │   │   ├── WeightMatrixManagement.jsx
│       │   │   └── SimilarProductsPopup.jsx
│       │   ├── badges/              # Status/role badges
│       │   ├── common/              # Shared components (Modal, Loading)
│       │   ├── dashboard/           # Dashboard components
│       │   │   ├── StatusCardsGrid.jsx
│       │   │   ├── BreakdownCharts.jsx
│       │   │   ├── TicketTables.jsx
│       │   │   └── PerformanceMetricsRow.jsx
│       │   ├── forms/               # Form components
│       │   │   ├── ProductTicketForm.jsx
│       │   │   ├── ChemicalPropertiesForm.jsx
│       │   │   ├── BiologicalPropertiesForm.jsx
│       │   │   ├── SKUVariantsForm.jsx
│       │   │   ├── QualitySpecificationsForm.jsx
│       │   │   ├── CorpBaseDataForm.jsx
│       │   │   └── SectionNavigator.jsx
│       │   ├── preview/             # Preview components
│       │   │   └── CorpBaseWebsitePreview.jsx
│       │   ├── modals/              # Modal components
│       │   │   └── CorpBaseModal.jsx
│       │   ├── TicketView.jsx       # Template-driven ticket viewer
│       │   └── MoleculeViewerRDKit.jsx
│       ├── pages/                   # Page components
│       ├── context/                 # React contexts
│       │   └── AuthContext.jsx      # Authentication context
│       ├── services/                # API client
│       └── utils/                   # Utilities
│           ├── apiErrorHandler.js   # Error handling
│           ├── formValidationUtils.js
│           ├── dateFormatters.js
│           └── pricingCalculations.js
├── docs/                            # Public documentation
├── logs/                            # Application logs (gitignored)
└── README.md                        # This file
```

## Documentation

Comprehensive documentation is available in the [docs/](docs/) folder.

**Quick Links:**
- [Documentation Index](docs/README.md) - Complete documentation guide
- [Setup Guide](docs/guides/SETUP_GUIDE.md) - Installation and configuration
- [API Documentation](docs/api/API_DOCUMENTATION.md) - REST API reference
- [Architecture](docs/architecture/ARCHITECTURE.md) - System design
- [Template Versioning](docs/TEMPLATE_VERSIONING.md) - Ticket template system
- [Maintenance Guide](docs/MAINTENANCE_GUIDE.md) - Operations guide

## Getting Started

### Prerequisites
- Node.js 18+ (LTS recommended)
- MongoDB 5.0+
- npm

### Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd npdi-app
   ```

2. **Install dependencies**
   ```bash
   npm run setup
   # Or manually:
   npm install && cd client && npm install
   ```

3. **Environment Setup**
   ```bash
   cp .env.example .env
   # Edit .env with your configuration
   ```

4. **Start the application**
   ```bash
   # Development mode (server + client)
   npm run dev

   # Or run separately:
   npm run server:dev  # Backend on port 5000
   npm run client:dev  # Frontend on port 5173
   ```

5. **Access the application**
   - Frontend: http://localhost:5173
   - Backend API: http://localhost:5000/api
   - Health check: http://localhost:5000/api/health

## Environment Variables

```bash
# Database
MONGODB_URI=mongodb://localhost:27017/npdi-app

# Server Configuration
PORT=5000
NODE_ENV=development
CLIENT_URL=http://localhost:5173

# Logging
LOG_LEVEL=INFO  # DEBUG, INFO, WARN, ERROR

# Optional: Azure OpenAI
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com
AZURE_OPENAI_API_KEY=your-api-key
AZURE_OPENAI_DEPLOYMENT=your-deployment-name

# Optional: Palantir
PALANTIR_BASE_URL=https://your-instance.palantirfoundry.com
PALANTIR_CLIENT_ID=your-client-id
PALANTIR_CLIENT_SECRET=your-client-secret
```

## User Roles & Access

### Product Manager
- Create and submit product tickets
- View and edit own tickets (DRAFT/SUBMITTED status)
- Add comments and updates
- Access personal dashboard

### PM-OPS Team
- View all tickets across all SBUs
- Change ticket status and workflow
- Assign tickets to team members
- Edit ticket details at any stage
- Access analytics dashboard

### Administrator
- Full system access and configuration
- User and API key management
- System settings configuration
- All PM-OPS permissions

## API Endpoints

### Product Tickets (Web Interface)
- `POST /api/products` - Create ticket
- `GET /api/products` - List tickets
- `GET /api/products/:id` - Get ticket details
- `PUT /api/products/:id` - Update ticket
- `PATCH /api/products/:id/status` - Update status
- `GET /api/products/:id/export-pdp` - Export PDP Checklist
- `GET /api/products/:id/export-pif` - Export Product Information Form
- `GET /api/products/:id/export-sap-loader` - Export SAP MDG-M Loader
- `GET /api/products/:id/export-corpbase-loader` - Export CorpBase Batch Client

### Integration Endpoints
- `GET /api/products/cas-lookup/:casNumber` - PubChem CAS lookup
- `GET /api/products/similar-products/:casNumber` - Similar products search
- `GET /api/products/sap-search` - SAP MARA search via Palantir
- `POST /api/products/generate-corpbase-content` - AI content generation
- `POST /api/products/lookup-corpbase-substance` - CorpBase substance lookup

### External API (API Key Auth)
- `POST /api/v1/tickets` - Create ticket
- `GET /api/v1/tickets/:id` - Get ticket
- `PUT /api/v1/tickets/:id` - Update ticket
- `GET /api/v1/tickets` - List tickets

See [API Documentation](docs/api/API_DOCUMENTATION.md) for complete reference.

## Logging

The application uses a custom logging system with:
- **Daily log rotation** - New file each day
- **Log levels** - ERROR, WARN, INFO, HTTP, DEBUG
- **Structured output** - JSON metadata support
- **Console output** - Color-coded in development

View logs:
```bash
# Today's logs
tail -f logs/combined-$(date +%Y-%m-%d).log

# Errors only
tail -f logs/error-$(date +%Y-%m-%d).log
```

## Development

### Linting
```bash
npm run lint
```

### Building for Production
```bash
npm run build
```

## Deployment

### Production Checklist
1. Set `NODE_ENV=production`
2. Configure production MongoDB
3. Set secure environment variables
4. Build client: `npm run build`
5. Use process manager (PM2/systemd)
6. Configure reverse proxy (nginx)
7. Enable HTTPS
8. Set up log rotation and monitoring

### Docker Support
Use `docker-compose.yml` for local MongoDB:
```bash
docker-compose up -d  # Start MongoDB container
```

## Security Features

- **Profile-Based Authentication** - Development profile selection
- **API Key Authentication** - For external integrations
- **Rate Limiting** - Brute force protection
- **Input Validation** - Express Validator
- **CORS Configuration** - Controlled cross-origin access
- **Helmet Security Headers** - HTTP security
- **Role-Based Access Control** - Granular permissions

## License

Proprietary - MilliporeSigma Internal Use Only
