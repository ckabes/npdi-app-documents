# SAP Loader File Analysis and NPDI Field Mapping

## Document Overview

**Source Files:**
- `ZIP__SLI_FERT_HF075040P35_2025-03-14 1.xlsx` - Equipment/Membrane product (FERT type)
- `ZIP_IND_PMC2013_Elevate_Jigani_2025-12-9.xlsx` - Chemical substance (ZSTA/IND type) - **Most similar to DM1-CHEM**

**Analysis Date:** December 9, 2025
**Purpose:** Comprehensive analysis of SAP MDG-M loader file structure and mapping to NPDI data models

---

## 1. SAP Loader File Structure

### 1.1 Sheets Overview (Both Files Share Same Structure)

| Sheet Name | Purpose | Description |
|------------|---------|-------------|
| **MDG-M** | Master Data Governance - Material (Legacy Blue) | Core material master data |
| **Next Classification** | NEXT Platform batch classification | Batch class characteristics |
| **Next Plant** | NEXT Platform plant data | Plant-specific material data (MARC) |
| **Next Dist Model** | Distribution model reference | Regional distribution configuration |
| **ERP** | ERP system routing | Maps subsidiaries to SAP/Oracle systems |
| **Next Proportional Unit** | Proportional UoM config | Batch-specific UoM handling |
| **PRD_PRE Stock** | Production/Pre-production stock | Legacy P95 plant data |
| **PRD_PRE 3rd Party** | Third-party stock | Legacy 3rd party vendor data |
| **Sales BOM** | Sales Bill of Materials | Kit/bundle component structure |
| **Oracle** | Oracle ERP data | Oracle-specific fields |
| **P01** | P01 System (Coming Soon) | Future SAP P01 integration |
| **P95** | P95 System (Coming Soon) | Future SAP P95 integration |

### 1.2 MDG-M Sheet Structure (Three-Row Header)

- **Row 1:** Display/Friendly Names (human-readable labels)
- **Row 2:** SAP Technical Field Names (MARA/custom YYD/ZZD fields)
- **Row 3:** Variable Field Names (loader tool placeholders like `<tbx2ProductNumber>`)
- **Row 4+:** Data rows (BULK row first, then SKU variants)

### 1.3 Key Difference: MDG-M (Legacy Blue) vs NEXT (Purple Platform)

The loader files support **two SAP platforms**:

| Aspect | MDG-M (Blue/Legacy) | NEXT (Purple Platform) |
|--------|---------------------|------------------------|
| Field Prefix | Standard MARA + YYD_ custom | YYD_ and ZZD_ custom |
| Material Type Field | MTART | YYD_MTNEX |
| Plant Data | Separate tabs | "Next Plant" tab |
| Classification | MDG-M tab | "Next Classification" tab |
| Distribution | Legacy routing | "Next Dist Model" tab |

---

## 2. File Comparison: FERT vs IND (Chemical)

### 2.1 ZIP_IND (Chemical Substance) - DM1-CHEM Mapping Target

**Product:** PMC2013 (cataCXium® A)
**CAS Number:** 321921-71-5
**Material Type:** ZSTA (IND/Intermediate)
**NEXT Material Type:** HALB (Semi-finished)

This file represents a **pure chemical substance** and maps directly to the **DM1-CHEM** ticket type in NPDI.

### 2.2 ZIP_FERT (Finished Equipment)

**Product:** HF075040P35 (Hi-Flow™ Plus membrane)
**Material Type:** ZSTA → FERT (Finished Product)
**NEXT Material Type:** FERT

This file represents a **finished equipment product** (membrane) - would map to a different ticket type.

### 2.3 Key Field Differences

| Field | ZIP_IND (Chemical) | ZIP_FERT (Equipment) | Notes |
|-------|-------------------|---------------------|-------|
| YYD_CASNR | 321921-71-5 | (empty) | CAS only for chemicals |
| MTART | ZSTA | ZSTA | Same material type |
| YYD_MTNEX | HALB | FERT | Different NEXT type |
| MEINS | G (Grams) | M (Meters) | Different base UoM |
| YYD_QASEG | 300 | 400 | Quality segment differs |
| YYD_ARTTY | (not set) | EQUI | Article type: Equipment |
| YYD_MATAT | (not set) | EQUCH | Material attribute |
| SPART | 86 | 86 | Same LS division |

---

## 3. Complete MDG-M Field Analysis (93 Fields)

### 3.1 Core Identification Fields

| Col | Technical Name | Description | FERT Sample | IND Sample | NPDI Mapping |
|-----|---------------|-------------|-------------|------------|--------------|
| 3 | MATNR | Material Number | HF075040P35 | PMC2013-BULK | `internalProductCode` + variant |
| 4 | MTART | Material Type (Legacy) | ZSTA | ZSTA | Derived from ticket type |
| 5 | YYD_MTNEX | Material Type NEXT | FERT | HALB | Derived from ticket type |
| 11 | BISMT | Old Material Number | HF075040P35 | PMC2013 | `internalProductCode` |
| 12 | YYD_LSIND | Web Indexing | HF075040P35 | PMC2013 | `internalProductCode` |

### 3.2 Organizational Fields

| Col | Technical Name | Description | FERT Sample | IND Sample | NPDI Mapping |
|-----|---------------|-------------|-------------|------------|--------------|
| 6 | SPART | Life Science Division | 86 | 86 | **sbu** (needs mapping) |
| 7 | YYD_YGART | Article Number | 612525 | SIAL99 | **Derived from SBU** |
| 8 | YYD_ARREL | Article Relation | YES | - | Fixed value |
| 9 | YYD_INHSC | Inheritance Schema | V1 | 99 | Configuration |
| 20 | PRDHA | Product Hierarchy (GPH) | MCA001LFK000 | PMC036AJQ397 | **Build from category** |
| 21 | YYD_YSBU | SBU Code | 635 | P79 | **sbu** |
| 22 | YYD_SOSUB | Material Source Subsidiary | 1937 | 2123 | Plant configuration |

### 3.3 Description Fields

| Col | Technical Name | Description | FERT Sample | IND Sample | NPDI Mapping |
|-----|---------------|-------------|-------------|------------|--------------|
| 14 | YYD_MAKTX | Material Description (40 char) | Hi-Flow™ Plus membrane... | cataCXium® A | **productName** (truncate) |
| 15 | YYD_YTX01 | Material Longname | Hi-Flow™ Plus membrane... | cataCXium® A | **productDescription** |

### 3.4 Chemical-Specific Fields (Critical for DM1-CHEM)

| Col | Technical Name | Description | IND Sample | NPDI Mapping |
|-----|---------------|-------------|------------|--------------|
| 19 | YYD_CASNR | CAS Number | 321921-71-5 | **casNumber** |
| 10 | KZUMW | Environmentally Relevant | X | **regulatoryInfo** or fixed |

### 3.5 Physical/Measurement Fields

| Col | Technical Name | Description | FERT Sample | IND Sample | NPDI Mapping |
|-----|---------------|-------------|-------------|------------|--------------|
| 16 | MEINS | Base Unit of Measure | M | G | `variants[].packageSize.unit` |
| 23 | MTPOSMARA | General Item Category | NORM | NORM | Fixed |
| 24 | BRGEWMARA | Gross Weight | 0.06 | 1 | `variants[].grossWeight.value` |
| 25 | NTGEW | Net Weight | 0.059 | 1 | `variants[].netWeight.value` |
| 26 | GEWEI_MAT | Weight Unit | kg | g | `variants[].grossWeight.unit` |
| 27 | VOLUMMARA | Volume | (empty) | (empty) | `variants[].volume.value` |
| 28 | VOLEHMARA | Volume Unit | (empty) | (empty) | `variants[].volume.unit` |
| 29 | GROES | Size/Dimensions | (empty) | (empty) | `variants[].packageSize` |
| 37 | MEABMMARA | Unit of Dimension | (empty) | (empty) | Configuration |
| 38 | LAENGMARA | Length | (empty) | (empty) | (not in NPDI) |
| 39 | BREITMARA | Width | (empty) | (empty) | (not in NPDI) |
| 40 | HOEHEMARA | Height | (empty) | (empty) | (not in NPDI) |

### 3.6 Packaging Fields

| Col | Technical Name | Description | Sample | NPDI Mapping |
|-----|---------------|-------------|--------|--------------|
| 30 | YYD_YPAGR | Package Group | (varies) | (not in NPDI - config) |
| 31 | YYD_YME01 | Alternative UoM 1 | (varies) | Derived |
| 32 | YYD_PACCO | Package Content | (varies) | `variants[].packageSize.value` |
| 33 | YYD_YINHA | Content UoM | (varies) | `variants[].packageSize.unit` |
| 34 | YYD_VPMAT | Packaging Material | (varies) | (not in NPDI - config) |
| 35 | YYD_VHART | Packaging Type | (varies) | (not in NPDI - config) |
| 36 | MAGRVMARA | Material Group Packaging | (varies) | Configuration |

### 3.7 Quality Management Fields

| Col | Technical Name | Description | FERT Sample | IND Sample | NPDI Mapping |
|-----|---------------|-------------|-------------|------------|--------------|
| 51 | YYD_QASEG | Quality Segment | 400 | 300 | **quality.gradeLevel** |
| 52 | YYD_QMPUR | QM Procurement Active | X | (empty) | Derived from quality |
| 71 | YYD_QCRES | QC Resample | (empty) | (empty) | Quality config |
| 72 | YYD_TESTD | Test Days | (empty) | (empty) | Quality config |

**Quality Segment Mapping (YYD_QASEG → NPDI gradeLevel):**
| SAP Value | Description | NPDI gradeLevel |
|-----------|-------------|-----------------|
| 100 | Technical Grade | 100 |
| 200 | Puriss | 200 |
| 300 | ACS/Reagent Grade | 300 |
| 400 | Analytical Grade | 400 |
| 500 | Biological Grade | 500 |
| 600 | Pharma Grade | 600 |

### 3.8 Storage and Handling Fields

| Col | Technical Name | Description | FERT Sample | IND Sample | NPDI Mapping |
|-----|---------------|-------------|-------------|------------|--------------|
| 60 | YYD_RAUBE | Storage Conditions | (empty) | 01 | **storageShipping.storageConditions** |
| 61 | YYD_TEMPB | Temperature Conditions | W1 | W1 | **storageShipping.tempCondition** |
| 62 | YYD_MHDRZ | Min Remaining Shelf Life | 1 | (empty) | **quality.minShelfLife** |
| 63 | YYD_MHDHB | Total Shelf Life | 0 | (empty) | **quality.totalShelfLife** |
| 64 | YYD_IPRKZ | Period Indicator | D | (empty) | Configuration |
| 65 | YYD_RDMHD | Rounding Rule Shelf Life | (empty) | (empty) | Configuration |

**Temperature Condition Mapping (YYD_TEMPB):**
| SAP Value | Description | NPDI storageConditions |
|-----------|-------------|------------------------|
| W1 | Room Temperature | Room Temperature (15-25°C) |
| K1 | Refrigerated | Refrigerated (2-8°C) |
| K2 | Frozen | Frozen (-20°C) |
| K3 | Deep Frozen | Deep Frozen (-80°C) |

### 3.9 Batch Management Fields

| Col | Technical Name | Description | Sample | NPDI Mapping |
|-----|---------------|-------------|--------|--------------|
| 58 | XCHPFMARA | Batch Management | X | Fixed for chemicals |
| 66 | YYD_YBAT2 | Batch Determination | (varies) | Configuration |

### 3.10 Branding and Marketing Fields

| Col | Technical Name | Description | FERT Sample | IND Sample | NPDI Mapping |
|-----|---------------|-------------|-------------|------------|--------------|
| 17 | YYD_MROLE | Material Role | PB | PB | Fixed |
| 18 | MATKL | Material Group | 3510999 | Y0001 | **Derived from category** |
| 44 | NUMTP1 | EAN Category | YE | (empty) | Configuration |
| 45 | YYD_YLOGO | Portfolio Brand | MP | SA | **Derived from SBU** |
| 46 | YYD_LSWEB | Brand ID for NEXT | MI | SI | **Derived from SBU** |
| 70 | YYD_TRADE | Trademark | YMDA.50.00513 | (empty) | (not in NPDI - config) |

### 3.11 Commercial/Operational Fields

| Col | Technical Name | Description | FERT Sample | IND Sample | NPDI Mapping |
|-----|---------------|-------------|-------------|------------|--------------|
| 47 | YYD_COGSI | COGS/ICM Indicator | X | X | Fixed |
| 48 | YYD_SERFL | Serial Number Flag | (empty) | - | Configuration |
| 49 | YYD_YGMP2 | GMP Level 2 | (empty) | (empty) | **regulatory.gmpLevel** |
| 50 | YYD_GMPKZ | GMP Indicator | (empty) | (empty) | **regulatory.gmpRequired** |
| 53 | YYD_APORL | APO Relevance | 4 | 0 | Configuration |
| 54 | YYD_ROYAL | Royalty Indicator | (empty) | (empty) | NEW: royaltyFlag |
| 55 | YYD_PRDOR | Product Origin | NA | NA | **countryOfOrigin** |
| 59 | YYD_EKWSL | Purchasing Value Key | ZGLB | ZGLB | Configuration |
| 67 | YYD_MEDRE | Media Relevance | 00 | 00 | Configuration |
| 68 | YYD_YKZLP | (Unknown) | (empty) | (empty) | Configuration |
| 69 | YYD_SELLF | Sellable Flag | 1 | 2 | Derived from status |
| 73 | YYD_MFRPN | Supplier Material Number | (empty) | (empty) | (not in NPDI) |
| 74 | YYD_EXPID | Export ID | (empty) | (empty) | **regulatory.exportId** |

### 3.12 Status and Lifecycle Fields

| Col | Technical Name | Description | FERT Sample | IND Sample | NPDI Mapping |
|-----|---------------|-------------|-------------|------------|--------------|
| 13 | MSTAE | X-Plant Material Status | 20 | 30 | Derived from ticket status |
| 56 | YYD_NPSDT | NPI Start Date | (empty) | (empty) | **createdAt** |
| 57 | YYD_NPEDT | NPI End Date | (empty) | (empty) | **targetLaunchDate** |

**Material Status Mapping (MSTAE):**
| SAP Value | Description | NPDI Status |
|-----------|-------------|-------------|
| 20 | Active - Full Sales | approved |
| 30 | Active - Limited | approved |
| 40 | Blocked | blocked |
| 90 | Phase Out | deprecated |

### 3.13 Classification and Attribute Fields

| Col | Technical Name | Description | FERT Sample | IND Sample | NPDI Mapping |
|-----|---------------|-------------|-------------|------------|--------------|
| 75 | ZZD_DFC | DFC Code | (empty) | (empty) | Configuration |
| 76 | ZZD_DOM | DOM Code | (empty) | (empty) | Configuration |
| 77 | YYD_GLMTY | Global Material Type | FIN | UFG | Derived from type |
| 78 | YYD_ARTTY | Article Type | EQUI | (empty) | Fixed per template |
| 79 | YYD_RESUB | Receiving Subsidiary | 1937 | 2123 | Plant config |
| 80 | YYD_LAYID | Layout ID | (empty) | (empty) | Configuration |
| 81 | YYD_LAYNA | Layout Name | (empty) | (empty) | Configuration |
| 82 | YYD_YIFRS | Global Revenue Classification | G | G | Fixed |
| 83 | YYD_GLOPH | Global Operational Hierarchy | 3MIE11CMSL00 | (empty) | Derived |
| 92 | YYD_MATAT | Material Attribute | EQUCH | (empty) | Fixed per template |

### 3.14 Organization Assignment Fields

| Col | Technical Name | Description | FERT Sample | IND Sample | NPDI Mapping |
|-----|---------------|-------------|-------------|------------|--------------|
| 84 | YYD_YORGA_GDP | Global Distribution Point | 09D | (empty) | Configuration |
| 85 | YYD_YORGA_GSP | Global Supply Point | N79 | (empty) | Configuration |
| 86 | YYD_YORGA_PPL | Organization Assignment PPL (Plant) | IE11 | IN03 | `primaryPlant` |
| 87 | YYD_YORGA_PRM | Organization Assignment PRM | PRM00847 | PRM00801 | Configuration |
| 88 | YYD_YORGA_PRP | Organization Assignment PRP | PRP00262 | PRP00772 | Configuration |

### 3.15 Oracle Integration Fields

| Col | Technical Name | Description | FERT Sample | IND Sample | NPDI Mapping |
|-----|---------------|-------------|-------------|------------|--------------|
| 89 | YYD_ORAST | Oracle Status | MTO | (empty) | Oracle config |
| 90 | YYD_ORATL | Oracle Template | 9001 | (empty) | Oracle config |
| 91 | YYD_OSLCR | Oracle Lot Exp Control | 4 | (empty) | Oracle config |
| 93 | SLED_BBD | Shelf Life Expiration | (varies) | (empty) | **quality.expirationPolicy** |

---

## 4. Next Classification Sheet Analysis

This sheet defines batch-level characteristics for the NEXT platform.

### 4.1 Structure

| Column | Technical Name | Description |
|--------|---------------|-------------|
| 3 | MATNR | Material Number |
| 4 | KLART | Class Type (023 = Batch) |
| 5 | CLASS | Class Name |
| 6 | ATNAM | Characteristic Name |
| 7 | ATWRT | Characteristic Value |

### 4.2 Key Batch Classifications

| Characteristic | Description | Values | NPDI Mapping |
|---------------|-------------|--------|--------------|
| QB_EXP_RETEST_CONTROL | Expiration/Retest Control | EXP, RET, N/A | **quality.expirationPolicy** |
| INTERNAL_LABEL_ANIMAL_ORIGIN | Animal Origin Indicator | ANIMAL, NON-ANIMAL, CONTAINS | **regulatory.animalOrigin** |

---

## 5. Next Plant Sheet Analysis (134 Fields)

The "Next Plant" sheet contains plant-specific material data (equivalent to SAP MARC table).

### 5.1 Key Plant Fields

| Col | Technical Name | Description | NPDI Mapping |
|-----|---------------|-------------|--------------|
| 3 | ZITEMID | NPDI Number | **ticket._id** |
| 4 | YYD_PRODTYP | Product Type | Derived from template |
| 5 | ZZDISTMDL | Distribution Model | Configuration |
| 7 | MATNR | Material Number | **internalProductCode** |
| 8 | WERKS | Plant Code | **NEW: plantCode** |
| 9 | LGORT | Storage Location | Configuration |
| 10 | VKORG | Sales Organization | Configuration |
| 13 | ZZPRODMAN | Product Manager | **primaryContact** |
| 22 | ZZMINSLEDSLS | Min Remaining Shelf Life Sales | Quality config |
| 23 | ZZBATCBUFF | Batch Buffer Time | Quality config |
| 24 | ZZSLEDTYPE | SLED Type | Quality config |
| 26 | ZZSCMODEL | Supply Chain Model | Configuration |

### 5.2 MRP/Planning Fields

| Col | Technical Name | Description | Default |
|-----|---------------|-------------|---------|
| 51 | DISGR | MRP Group | FORE/0002 |
| 53 | DISMM | MRP Type | X0 |
| 55 | DISPO | MRP Controller | Various |
| 56 | DISLS | Lot Sizing Procedure | EX/12 |
| 63 | BESKZ | Procurement Type | E |
| 64 | KZECH | Batch Entry | 3 |

### 5.3 Quality Management Fields

| Col | Technical Name | Description | NPDI Mapping |
|-----|---------------|-------------|--------------|
| 89 | QMATA | QM Material Authorization | Quality config |
| 92 | QMPUR | QM Procurement Active | Derived |
| 93 | SSQSS | QM Control Key | Fixed |

### 5.4 Valuation Fields

| Col | Technical Name | Description | NPDI Mapping |
|-----|---------------|-------------|--------------|
| 94 | BKLAS | Valuation Class | Configuration |
| 95 | STPRS | Standard Price | `variants[].pricing.standardCost` |
| 96 | PEINH | Price Unit | Configuration |
| 97 | VPRSV | Price Control | Fixed (S) |

---

## 6. NPDI Data Model Analysis

### 6.1 BaseTicket Schema Fields

| NPDI Field | Type | Description | SAP Target |
|------------|------|-------------|------------|
| productName | String | Full product name | YYD_MAKTX (40 char) |
| productDescription | String | Detailed description | YYD_YTX01 |
| casNumber | String | CAS Registry Number | **YYD_CASNR** |
| mdlNumber | String | MDL Number | Not in loader |
| molecularFormula | String | Chemical formula | Not in loader |
| molecularWeight | Number | Molecular weight | Not in loader |
| internalProductCode | String | SKU/Internal code | MATNR, BISMT |
| linkedProducts | Array | Related products | Not in loader |
| sbu | String | Strategic Business Unit | **YYD_YSBU, SPART** |
| primaryContact | ObjectId | Contact reference | ZZPRODMAN |

### 6.2 Chemical Properties Schema (DM1-CHEM)

| NPDI Field | Type | SAP Mapping Candidate | Status |
|------------|------|----------------------|--------|
| appearance | String | Not in loader | Gap |
| color | String | Not in loader | Gap |
| odor | String | Not in loader | Gap |
| physicalForm | Enum | Not in loader | Gap |
| meltingPoint.value | Number | Not in loader | Gap |
| boilingPoint.value | Number | Not in loader | Gap |
| flashPoint.value | Number | Not in loader | Gap |
| density | Number | Not in loader | Gap |
| solubility | String | Not in loader | Gap |
| phValue | Number | Not in loader | Gap |
| vaporPressure | Number | Not in loader | Gap |
| autoIgnitionTemp | Number | Not in loader | Gap |

**Key Finding:** Chemical property fields are NOT in the SAP loader file. These would be maintained in SAP EHS (Environment, Health & Safety) module or separate regulatory systems.

### 6.3 Hazard Classification Schema

| NPDI Field | Type | SAP Mapping | Status |
|------------|------|-------------|--------|
| ghsHazardClass | Array | Not in loader | Gap - EHS data |
| ghsSignalWord | String | Not in loader | Gap - EHS data |
| pictograms | Array | Not in loader | Gap - EHS data |
| hStatements | Array | Not in loader | Gap - EHS data |
| pStatements | Array | Not in loader | Gap - EHS data |
| dotHazardClass | String | Not in loader | Gap - EHS data |
| unNumber | String | Not in loader | Gap - EHS data |
| packingGroup | String | Not in loader | Gap - EHS data |

### 6.4 Regulatory Info Schema

| NPDI Field | Type | SAP Mapping | Status |
|------------|------|-------------|--------|
| tsca | Boolean | Not in loader | Gap - Regulatory system |
| einecs | Boolean | Not in loader | Gap |
| reach | Boolean | Not in loader | Gap |
| canadaDSL | Boolean | Not in loader | Gap |
| japanMITI | Boolean | Not in loader | Gap |
| australiaAICS | Boolean | Not in loader | Gap |
| koreaECL | Boolean | Not in loader | Gap |
| chinaIECSC | Boolean | Not in loader | Gap |
| philippinesPICCS | Boolean | Not in loader | Gap |

### 6.5 Quality Schema

| NPDI Field | Type | SAP Mapping | Status |
|------------|------|-------------|--------|
| gradeLevel | Enum | **YYD_QASEG** | Mappable |
| purityGrade | String | Not in loader | Gap |
| purityValue | Number | Not in loader | Gap |
| certifications | Array | Not in loader | Gap |
| lotNumberFormat | String | Not in loader | Gap |
| expirationPolicy | String | **QB_EXP_RETEST_CONTROL** | Mappable (Classification) |

### 6.6 Storage/Shipping Schema

| NPDI Field | Type | SAP Mapping | Status |
|------------|------|-------------|--------|
| storageConditions | String | **YYD_RAUBE, YYD_TEMPB** | Mappable |
| shippingClass | String | Not directly in MDG-M | Gap |
| specialHandling | Array | Not in loader | Gap |

### 6.7 SKU Variant Schema

| NPDI Field | Type | SAP Mapping | Status |
|------------|------|-------------|--------|
| variants[].sku | String | **MATNR** | Mappable |
| variants[].size | String | **GROES** | Mappable |
| variants[].unit | String | **MEINS** | Mappable |
| variants[].price | Number | Not in MDG-M | Gap (KONP table) |

---

## 7. Field Mapping Matrix: NPDI → SAP MDG-M

### 7.1 Direct Mappings (Ready to Implement)

| Priority | NPDI Source | SAP Target | Transform | Notes |
|----------|-------------|------------|-----------|-------|
| P1 | `productName` | YYD_MAKTX | Truncate to 40 char | Direct |
| P1 | `productName` / `productDescription` | YYD_YTX01 | Full text | Longname |
| P1 | `casNumber` | YYD_CASNR | Format validation | Chemical only |
| P1 | `internalProductCode` | MATNR, BISMT, YYD_LSIND | Direct | Multiple fields |
| P1 | `sbu` | YYD_YSBU | Code mapping | Lookup table needed |
| P1 | `primaryPlant` | YYD_YORGA_PPL, WERKS | Direct | Plant code |
| P1 | `variants[].grossWeight.value` | BRGEWMARA | Direct | Per variant |
| P1 | `variants[].grossWeight.unit` | GEWEI_MAT | Unit mapping | g→G, kg→KG |
| P1 | `variants[].netWeight.value` | NTGEW | Direct | Per variant |
| P1 | `variants[].volume.value` | VOLUMMARA | Direct | Per variant |
| P1 | `variants[].volume.unit` | VOLEHMARA | Unit mapping | mL→ML, L→L |
| P1 | `variants[].packageSize` | GROES, YYD_PACCO, YYD_YINHA | Format as size string | Per variant |
| P1 | `variants[].pricing.standardCost` | STPRS | Direct | Per variant |
| P2 | `quality.gradeLevel` | YYD_QASEG | Enum mapping | See mapping table |
| P2 | `storageShipping.storageConditions` | YYD_TEMPB | Enum mapping | See mapping table |
| P2 | `variants[].packageSize.unit` | MEINS | Unit code mapping | Standard UoM codes |

### 7.2 Derived/Calculated Fields

| SAP Field | Derivation Logic | Source |
|-----------|------------------|--------|
| MTART | `ZSTA` for chemicals | Fixed per template |
| YYD_MTNEX | `HALB` for bulk, `FERT` for finished | Derived from SKU type |
| SPART | Map from SBU code | sbu → division mapping |
| MATKL | Build from SBU + category | Lookup table |
| PRDHA | Product hierarchy from category | Category hierarchy |
| YYD_YGART | Article number from SBU | SBU → article mapping |
| YYD_YLOGO | Brand from SBU | SBU → brand mapping |
| YYD_LSWEB | Web brand from SBU | SBU → web brand mapping |
| YYD_GLMTY | `UFG` for chemical, `FIN` for finished | Template-based |
| KZUMW | `X` for chemicals | Fixed |
| XCHPFMARA | `X` for batch-managed | Fixed |

### 7.3 Existing NPDI Fields (Already Available)

These fields already exist in NPDI and can be directly mapped:

| SAP Field | NPDI Field | Schema Location | Notes |
|-----------|------------|-----------------|-------|
| BRGEWMARA | `variants[].grossWeight.value` | skuVariantSchema | Has value/unit structure |
| GEWEI_MAT | `variants[].grossWeight.unit` | skuVariantSchema | enum: ['mg', 'g', 'kg', 'lb', 'oz'] |
| NTGEW | `variants[].netWeight.value` | skuVariantSchema | Has value/unit structure |
| VOLUMMARA | `variants[].volume.value` | skuVariantSchema | Has value/unit structure |
| VOLEHMARA | `variants[].volume.unit` | skuVariantSchema | enum: ['mL', 'L', 'µL', 'gal', 'fl oz'] |
| YYD_YORGA_PPL | `primaryPlant` | BaseTicket | Plant code string |
| STPRS | `variants[].pricing.standardCost` | skuVariantSchema | Standard cost per variant |
| GROES | `variants[].packageSize` | skuVariantSchema | Has value/unit structure |

**Supporting Reference Models:**
- **PlantCode** - Reference data for valid plant codes (code, description)
- **WeightMatrix** - Gross weight lookup by package size (maps to SAP BRGEW/GEWEI)

### 7.4 Fields Potentially Requiring New NPDI Schema Additions

| SAP Field | Proposed NPDI Field | Schema | Priority |
|-----------|---------------------|--------|----------|
| LAENGMARA | `dimensions.length` | skuVariantSchema | Low |
| BREITMARA | `dimensions.width` | skuVariantSchema | Low |
| HOEHEMARA | `dimensions.height` | skuVariantSchema | Low |
| YYD_MFRPN | `supplierPartNumber` | BaseTicket | Low |

### 7.5 Fields NOT in Loader (Maintained Elsewhere)

These fields exist in NPDI but are NOT part of the SAP MDG-M loader:

| NPDI Field Category | Reason | Where Maintained |
|--------------------|--------|------------------|
| Chemical properties (melting point, density, etc.) | SAP EHS | EHS module |
| Hazard classifications (GHS, DOT) | SAP EHS | EHS module |
| Regulatory registrations (TSCA, REACH) | SAP EHS | EHS module |
| Purity specifications | QM specs | QM module |
| Pricing | SD Pricing | KONP/Pricing tables |

### 7.6 Configuration-Only Fields (Not from NPDI)

These fields have fixed values or come from system configuration:

| SAP Field | Value/Source | Notes |
|-----------|--------------|-------|
| YYD_INHSC | 99 or V1 | Inheritance schema config |
| YYD_ARREL | YES or - | Article relation config |
| YYD_MROLE | PB | Material role (fixed) |
| MTPOSMARA | NORM | Item category (fixed) |
| YYD_COGSI | X | COGS indicator (fixed) |
| YYD_PRDOR | NA | Product origin (fixed) |
| YYD_YIFRS | G | Revenue classification (fixed) |

---

## 8. SBU/Division Mapping Table

Based on sample data analysis:

| SBU Code | SPART (Division) | YYD_YGART | YYD_YLOGO | YYD_LSWEB | Description |
|----------|------------------|-----------|-----------|-----------|-------------|
| P79 | 86 | SIAL99 | SA | SI | Sigma-Aldrich Chemicals |
| 635 | 86 | 612525 | MP | MI | MilliporeSigma |
| 775 | 86 | (varies) | (varies) | (varies) | Life Science |

---

## 9. Implementation Architecture

### 9.1 Export Data Flow

```
NPDI Ticket (DM1-CHEM)
        │
        ▼
┌───────────────────┐
│  Field Mapping    │
│  Service          │
│  - Direct maps    │
│  - Enum lookups   │
│  - Derivations    │
└───────────────────┘
        │
        ▼
┌───────────────────┐
│  Validation       │
│  - Required check │
│  - Format check   │
│  - Missing report │
└───────────────────┘
        │
        ▼
┌───────────────────┐
│  Excel Generator  │
│  - MDG-M sheet    │
│  - Classification │
│  - Plant data     │
└───────────────────┘
        │
        ▼
   SAP Loader File (.xlsx)
```

### 9.2 Multi-Row Generation

For a single NPDI ticket with variants, the loader generates multiple rows:

```
NPDI Ticket: cataCXium® A (PMC2013)
  ├── Variant: 1G   → Row: PMC2013-1G (FERT)
  ├── Variant: 5G   → Row: PMC2013-5G (FERT)
  ├── Variant: 25G  → Row: PMC2013-25G (FERT)
  └── Bulk Material → Row: PMC2013-BULK (HALB)
```

### 9.3 Proposed Service Structure

```javascript
// services/sapExport/index.js
module.exports = {
  SAPLoaderExporter,      // Main export orchestrator
  FieldMapper,            // NPDI → SAP field mapping
  EnumMapper,             // Enum value transformations
  ValidationService,      // Required field validation
  ExcelGenerator,         // XLSX file generation
  MappingConfig           // Configuration tables
};
```

---

## 10. Appendix

### A. Sample MDG-M Row for Chemical Product

```javascript
{
  // Col 3-12: Identification
  MATNR: 'PMC2013-BULK',
  MTART: 'ZSTA',
  YYD_MTNEX: 'HALB',
  SPART: '86',
  YYD_YGART: 'SIAL99',
  YYD_ARREL: '-',
  YYD_INHSC: '99',
  KZUMW: 'X',
  BISMT: 'PMC2013',
  YYD_LSIND: 'PMC2013',

  // Col 13-19: Description & Classification
  MSTAE: '30',
  YYD_MAKTX: 'cataCXium® A',
  YYD_YTX01: 'cataCXium® A',
  MEINS: 'G',
  YYD_MROLE: 'PB',
  MATKL: 'Y0001',
  YYD_CASNR: '321921-71-5',

  // Col 20-26: Hierarchy & Weight
  PRDHA: 'PMC036AJQ397',
  YYD_YSBU: 'P79',
  YYD_SOSUB: '2123',
  MTPOSMARA: 'NORM',
  BRGEWMARA: '1',
  NTGEW: '1',
  GEWEI_MAT: 'g',

  // Col 45-52: Branding & Quality
  YYD_YLOGO: 'SA',
  YYD_LSWEB: 'SI',
  YYD_COGSI: 'X',
  YYD_QASEG: '300',

  // Col 58-61: Batch & Storage
  XCHPFMARA: 'X',
  YYD_EKWSL: 'ZGLB',
  YYD_RAUBE: '01',
  YYD_TEMPB: 'W1',

  // Col 77-88: Global Classification
  YYD_GLMTY: 'UFG',
  YYD_RESUB: '2123',
  YYD_YIFRS: 'G',
  YYD_YORGA_PPL: 'IN03',
  YYD_YORGA_PRM: 'PRM00801',
  YYD_YORGA_PRP: 'PRP00772'
}
```

### B. Variable Field Placeholders Reference

The loader tool uses placeholders in Row 3 that are replaced with actual values:

| Placeholder | Description | Source |
|-------------|-------------|--------|
| `<tbx2ProductNumber>` | Material Number | NPDI internalProductCode |
| `<tbx2MaterialName>` | Long Material Name | NPDI productName |
| `<tbx5CAS>` | CAS Number | NPDI casNumber |
| `<tbx2NetWeight>` | Net Weight | NPDI netWeight |
| `<tbx2NetVolume>` | Net Volume | NPDI volume |
| `<cbx2CountryOfOrigin>` | Country of Origin | NPDI countryOfOrigin |
| `<tbx2SupplierMaterialNumber>` | Supplier Part # | NPDI supplierPartNumber |
| `<tbx1NPDI>` | NPDI Ticket ID | NPDI ticket._id |

### C. Related SAP Tables

| Table | Description | Loader Sheet |
|-------|-------------|--------------|
| MARA | General Material Data | MDG-M |
| MAKT | Material Descriptions | MDG-M (YYD_MAKTX, YYD_YTX01) |
| MARC | Plant Data | Next Plant |
| MARD | Storage Location Data | Next Plant |
| MVKE | Sales Data | Next Plant |
| MBEW | Material Valuation | Next Plant (STPRS) |
| INOB | Class Link | Next Classification |
| AUSP | Characteristic Values | Next Classification |

---

*Document Version: 2.0*
*Generated: December 9, 2025*
*Last Updated: December 9, 2025*
