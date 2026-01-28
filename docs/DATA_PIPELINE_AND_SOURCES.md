# Data Pipeline & Sources

**Date:** 2026-01-28  
**Model:** `sylvamo_mfg/sylvamo_manufacturing/v4`

---

## Overview

This document describes the data sources and transformations that populate the `sylvamo_mfg` data model with real Sylvamo production data.

## Data Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           SOURCE SYSTEMS                                     │
├─────────────────┬─────────────────┬─────────────────┬──────────────────────┤
│   SAP ERP       │   PPR System    │   SharePoint    │   Proficy            │
│   (Materials)   │   (Production)  │   (Quality)     │   (Lab Tests)        │
└────────┬────────┴────────┬────────┴────────┬────────┴──────────┬───────────┘
         │                 │                 │                   │
         ▼                 ▼                 ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         MICROSOFT FABRIC                                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │ PPV Snapshot│  │ PPR Hist    │  │ Roll Quality│  │ Event Tests         │ │
│  │ (SAP Data)  │  │ Reel/Roll   │  │ Reports     │  │ (Lab Data)          │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└────────┬────────────────┬────────────────┬────────────────┬─────────────────┘
         │                │                │                │
         ▼                ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           CDF RAW DATABASES                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│  raw_sylvamo_fabric/                                                         │
│    ├── ppv_snapshot          → MaterialCostVariance                          │
│    ├── ppr_hist_reel         → Reel                                          │
│    ├── ppr_hist_roll         → Roll                                          │
│    └── ppr_hist_package      → Package                                       │
│                                                                              │
│  raw_sylvamo_pilot/                                                          │
│    └── sharepoint_roll_quality → QualityResult                               │
│                                                                              │
│  raw_sylvamo_proficy/                                                        │
│    └── events_tests          → QualityResult (lab tests)                     │
└────────────────────────────────────────────────────────────────────────────┘
         │
         ▼ [Transformations]
┌─────────────────────────────────────────────────────────────────────────────┐
│                      sylvamo_mfg DATA MODEL (v4)                             │
├─────────────────────────────────────────────────────────────────────────────┤
│  Views:                                                                      │
│    Asset, Equipment, ProductDefinition, Recipe, Reel, Roll,                  │
│    Package, QualityResult, MaterialCostVariance                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Data Sources

### 1. SAP ERP Data (via Fabric)

| RAW Table | Description | Refresh |
|-----------|-------------|---------|
| `raw_sylvamo_fabric/ppv_snapshot` | Material cost and PPV data | Daily |

**Key Fields:**
- `material` - SAP material number
- `material_description` - Material name
- `material_type` - Classification (RAWM, HALB, etc.)
- `plant` - Plant code
- `current_ppv` / `prior_ppv` - Purchase price variance
- `current_standard_cost` / `prior_standard_cost` - Standard costs
- `ppv_snapshot_date` - Snapshot timestamp

### 2. PPR Production Data (via Fabric)

| RAW Table | Description | Records |
|-----------|-------------|---------|
| `raw_sylvamo_fabric/ppr_hist_reel` | Paper reel production history | 100+ |
| `raw_sylvamo_fabric/ppr_hist_roll` | Cut roll history | 19+ |
| `raw_sylvamo_fabric/ppr_hist_package` | Package/shipping history | 100+ |

**Reel Key Fields:**
- `reel_number` - Unique reel identifier (e.g., EM0010110008)
- `reel_manufactured_date` - Production date (YYYYMMDD)
- `reel_product_code` - Product type (W = Wove)
- `reel_average_basis_weight` - Basis weight measurement
- `reel_average_caliper` - Caliper measurement
- `reel_average_moisture` - Moisture percentage
- `reel_finished_weight` - Final weight in lbs
- `reel_status_ind` - Status code (C = Complete)

**Roll Key Fields:**
- `roll_number` - Unique roll identifier
- `roll_reel_number` - Parent reel reference
- `roll_basis_weight` - Basis weight
- `roll_caliper` - Caliper measurement
- `roll_current_weight` - Weight in lbs
- `roll_producing_machine` - Machine code (EMW01, etc.)

**Package Key Fields:**
- `pack_package_number` - Unique package ID
- `pack_number_rolls_in_package` - Roll count
- `pack_assembled_date` - Assembly date
- `pack_ship_date` - Shipping date
- `pack_current_inv_point` - Inventory location

### 3. SharePoint Quality Data

| RAW Table | Description | Records |
|-----------|-------------|---------|
| `raw_sylvamo_pilot/sharepoint_roll_quality` | Roll quality inspection reports | 21+ |

**Key Fields:**
- `title` - Roll ID being inspected
- `defect` - Defect code (e.g., "005 - Crushed Edge")
- `was_the_roll_rejected` - Yes/No
- `location` - Defect location (Top, Bottom, All)
- `who_is_entering` - Inspector name
- `created_by` - Source equipment (e.g., "Mill Floor Sheeter Sheeter 1")

### 4. Proficy Lab Data (Future)

| RAW Table | Description | Status |
|-----------|-------------|--------|
| `raw_sylvamo_proficy/events_tests` | Lab test results | Planned |
| `raw_sylvamo_proficy/tests` | Test definitions | Planned |

---

## Transformations

### Transform 1: RAW → Reel

**Source:** `raw_sylvamo_fabric/ppr_hist_reel`  
**Target:** `sylvamo_mfg:Reel/v2`

```sql
-- Transformation: populate_reels
SELECT
  CONCAT('reel:', reel_number) AS externalId,
  reel_number AS reelNumber,
  TO_TIMESTAMP(reel_manufactured_date, 'YYYYMMDD') AS productionDate,
  CAST(reel_finished_weight AS FLOAT) AS weight,
  CAST(reel_reel_width_num AS FLOAT) AS width,
  CAST(reel_actual_diameter_num AS FLOAT) AS diameter,
  reel_status_ind AS status,
  -- Product mapping based on basis weight
  CASE 
    WHEN reel_average_basis_weight < 22 THEN 'product:wove-20'
    ELSE 'product:wove-24'
  END AS productDefinition_externalId,
  'equip:emp01' AS equipment_externalId
FROM `raw_sylvamo_fabric`.`ppr_hist_reel`
```

### Transform 2: RAW → Roll

**Source:** `raw_sylvamo_fabric/ppr_hist_roll`  
**Target:** `sylvamo_mfg:Roll/v2`

```sql
-- Transformation: populate_rolls
SELECT
  CONCAT('roll:', roll_number) AS externalId,
  roll_number AS rollNumber,
  CAST(roll_width_num AS FLOAT) AS width,
  CAST(roll_original_diameter AS FLOAT) AS diameter,
  CAST(roll_current_weight AS FLOAT) AS weight,
  'Produced' AS status,
  'A' AS qualityGrade,
  CONCAT('reel:', roll_reel_number) AS reel_externalId
FROM `raw_sylvamo_fabric`.`ppr_hist_roll`
```

### Transform 3: RAW → Package

**Source:** `raw_sylvamo_fabric/ppr_hist_package`  
**Target:** `sylvamo_mfg:Package/v2`

```sql
-- Transformation: populate_packages
SELECT
  CONCAT('pkg:', pack_package_number) AS externalId,
  pack_package_number AS packageNumber,
  CAST(pack_number_rolls_in_package AS INT) AS rollCount,
  CASE 
    WHEN pack_ship_date IS NOT NULL AND TRIM(pack_ship_date) != '' THEN 'Shipped'
    WHEN pack_assembled_date IS NOT NULL THEN 'Assembled'
    ELSE 'Created'
  END AS status,
  TO_TIMESTAMP(pack_ship_date, 'YYYYMMDD') AS shippedDate,
  'asset:eastover' AS sourcePlant_externalId,
  'asset:sumpter' AS destinationPlant_externalId
FROM `raw_sylvamo_fabric`.`ppr_hist_package`
```

### Transform 4: RAW → MaterialCostVariance

**Source:** `raw_sylvamo_fabric/ppv_snapshot`  
**Target:** `sylvamo_mfg:MaterialCostVariance/v1`

```sql
-- Transformation: populate_material_cost_variance
SELECT
  CONCAT('mcv:', material) AS externalId,
  CAST(material AS STRING) AS material,
  material_description AS materialDescription,
  material_type AS materialType,
  plant,
  gl_account AS glAccount,
  units,
  CAST(current_quantity AS FLOAT) AS currentQuantity,
  CAST(prior_quantity AS FLOAT) AS priorQuantity,
  CAST(current_standard_cost AS FLOAT) AS currentStandardCost,
  CAST(prior_standard_cost AS FLOAT) AS priorStandardCost,
  CAST(current_unit_cost AS FLOAT) AS currentUnitCost,
  CAST(prior_unit_cost AS FLOAT) AS priorUnitCost,
  CAST(current_ppv AS FLOAT) AS currentPPV,
  CAST(prior_ppv AS FLOAT) AS priorPPV,
  CAST(current_ppv AS FLOAT) - CAST(prior_ppv AS FLOAT) AS ppvChange,
  TO_TIMESTAMP(ppv_snapshot_date) AS snapshotDate,
  CAST(ppv_surrogate_key AS STRING) AS surrogateKey
FROM `raw_sylvamo_fabric`.`ppv_snapshot`
```

### Transform 5: RAW → QualityResult

**Source:** `raw_sylvamo_pilot/sharepoint_roll_quality`  
**Target:** `sylvamo_mfg:QualityResult/v2`

```sql
-- Transformation: populate_quality_results
SELECT
  CONCAT('qr:', title) AS externalId,
  'Roll Quality Inspection' AS testName,
  'Visual Inspection' AS testMethod,
  COALESCE(defect, 'No defects') AS resultText,
  CASE WHEN was_the_roll_rejected = 'Yes' THEN FALSE ELSE TRUE END AS isInSpec,
  CONCAT('roll:', title) AS roll_externalId
FROM `raw_sylvamo_pilot`.`sharepoint_roll_quality`
```

---

## Data Refresh Schedule

| Transformation | Frequency | Window |
|----------------|-----------|--------|
| populate_reels | Daily | 2:00 AM EST |
| populate_rolls | Daily | 2:15 AM EST |
| populate_packages | Daily | 2:30 AM EST |
| populate_material_cost_variance | Daily | 3:00 AM EST |
| populate_quality_results | Hourly | On change |

---

## Data Quality Rules

### Roll ID Normalization
- SharePoint roll IDs may have 'EM' prefix stripped
- Transformation adds 'EM' prefix if missing for matching

### Timestamp Handling
- PPR dates in YYYYMMDD format → ISO 8601
- Null dates handled gracefully

### Material Deduplication
- Materials are unique by material number
- Latest snapshot retained on conflict

---

## Current Data Statistics

| Entity | Count | Source |
|--------|-------|--------|
| Asset | 2 | Static (Eastover, Sumpter) |
| Equipment | 3 | Static (EMP01, EMW01, Sheeter) |
| ProductDefinition | 2 | Derived from basis weight ranges |
| Reel | 50 | `ppr_hist_reel` |
| Roll | 19 | `ppr_hist_roll` |
| Package | 50 | `ppr_hist_package` |
| QualityResult | 21 | `sharepoint_roll_quality` |
| MaterialCostVariance | 176 | `ppv_snapshot` |
| **TOTAL** | **197** | Real production data |

---

*Document created: January 28, 2026*
