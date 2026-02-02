# Appendix: sylvamo_mfg_core Data Model

> **Status:** Draft for Discussion  
> **Last Updated:** January 31, 2026  
> **Space:** sylvamo_mfg_core_schema  
> **Instance Space:** sylvamo_mfg_core_instances

This appendix documents the `sylvamo_mfg_core` data model, which extends the Cognite Data Model (CDM) with manufacturing-specific entities for paper production.

---

## Entity Relationship Diagram

```mermaid
erDiagram
    Asset ||--o{ Asset : "parent/children"
    Asset ||--o{ MfgTimeSeries : "timeSeries"
    Asset ||--o{ Reel : "reels"
    Asset ||--o{ Event : "events"
    Asset ||--o{ CogniteFile : "files"
    
    Reel ||--o{ Roll : "rolls"
    Reel ||--o{ Event : "events"
    Reel ||--o{ RollQuality : "qualityResults"
    
    Roll ||--o{ Event : "events"
    Roll ||--o{ RollQuality : "qualityResults"
    
    Asset {
        string name
        string description
        string assetType
        string sapFunctionalLocation
        string plantCode
        string workCenterCode
        string workCenterDescription
    }
    
    MfgTimeSeries {
        string name
        string description
        boolean isStep
        enum type
        string measurementType
        string piTagName
        timeseries timeSeries
    }
    
    Event {
        string name
        string description
        timestamp startTime
        timestamp endTime
        string eventType
        string eventSubtype
        float resultValue
        string resultText
        string unit
        boolean isInSpec
        string sourceSystem
    }
    
    Reel {
        string reelNumber
        timestamp productionDate
        float weight
        float width
        float diameter
        string status
        string gradeCode
        float machineSpeed
        float avgBasisWeight
        float avgCaliper
        float avgMoisture
        int splices
    }
    
    Roll {
        string rollNumber
        float weight
        float width
        float diameter
        string status
        string qualityGrade
        timestamp cutDate
        float basisWeight
        float caliper
        float moisture
        int linearFootage
        string paperMachine
        string producingMachine
    }
    
    CogniteFile {
        string name
        string description
        string mimeType
        string directory
        boolean isUploaded
        timestamp uploadedTime
    }
    
    Material {
        string materialCode
        string materialDescription
        string materialType
        string materialGroup
        string unitOfMeasure
        boolean isActive
    }
    
    RollQuality {
        string testName
        string testMethod
        timestamp testDate
        float resultValue
        string resultText
        string unitOfMeasure
        float specMin
        float specMax
        float specTarget
        boolean isInSpec
        string defectType
        string sourceSystem
    }
```

---

## Relationship Flowchart

```mermaid
flowchart TB
    subgraph core [Core Entities]
        Asset[Asset]
        TimeSeries[MfgTimeSeries]
        Event[Event]
    end
    
    subgraph production [Production Entities]
        Reel[Reel]
        Roll[Roll]
        Material[Material]
    end
    
    subgraph quality [Quality]
        RollQuality[RollQuality]
    end
    
    subgraph cdm [CDM Entities]
        CogniteFile[CogniteFile]
    end
    
    Asset -->|parent| Asset
    Asset -->|reels| Reel
    Asset -->|events| Event
    Asset -->|timeSeries| TimeSeries
    Asset -->|files| CogniteFile
    
    Reel -->|asset| Asset
    Reel -->|rolls| Roll
    Reel -->|events| Event
    Reel -->|qualityResults| RollQuality
    
    Roll -->|reel| Reel
    Roll -->|events| Event
    Roll -->|qualityResults| RollQuality
    
    Event -->|asset| Asset
    Event -->|reel| Reel
    Event -->|roll| Roll
    
    RollQuality -->|roll| Roll
    RollQuality -->|reel| Reel
```

---

## CDM Inheritance Diagram

```mermaid
classDiagram
    direction LR
    
    class CogniteDescribable {
        <<CDM>>
        name
        description
        tags
        aliases
    }
    
    class CogniteSourceable {
        <<CDM>>
        sourceId
        source
    }
    
    class CogniteAsset {
        <<CDM>>
        parent
        root
        path
    }
    
    class CogniteActivity {
        <<CDM>>
        startTime
        endTime
        assets
    }
    
    class CogniteTimeSeries {
        <<CDM>>
        isStep
        type
        unit
    }
    
    CogniteDescribable <|-- Asset
    CogniteSourceable <|-- Asset
    CogniteAsset <|-- Asset
    
    CogniteDescribable <|-- Event
    CogniteSourceable <|-- Event
    CogniteActivity <|-- Event
    
    CogniteDescribable <|-- Reel
    CogniteSourceable <|-- Reel
    
    CogniteDescribable <|-- Roll
    CogniteSourceable <|-- Roll
    
    CogniteDescribable <|-- MfgTimeSeries
    CogniteSourceable <|-- MfgTimeSeries
    CogniteTimeSeries <|-- MfgTimeSeries
    
    CogniteDescribable <|-- Material
    CogniteSourceable <|-- Material
    
    CogniteDescribable <|-- RollQuality
    CogniteSourceable <|-- RollQuality
```

---

## Entity Summary

| Entity | Implements | Custom Container | Key Relationships |
|--------|------------|------------------|-------------------|
| **Asset** | CogniteAsset, CogniteDescribable, CogniteSourceable | MfgAsset | parent/children (hierarchy), timeSeries, reels, events, files |
| **MfgTimeSeries** | CogniteTimeSeries, CogniteDescribable, CogniteSourceable | MfgTimeSeries | assets, timeSeries (CDF link) |
| **Event** | CogniteActivity, CogniteDescribable, CogniteSourceable | MfgEvent | asset, reel, roll |
| **Reel** | CogniteDescribable, CogniteSourceable | MfgReel | asset, rolls, events, qualityResults |
| **Roll** | CogniteDescribable, CogniteSourceable | MfgRoll | reel, events, qualityResults |
| **Material** | CogniteDescribable, CogniteSourceable | Material | (standalone) |
| **RollQuality** | CogniteDescribable, CogniteSourceable | RollQuality | roll, reel |
| **CogniteFile** | CogniteDescribable, CogniteSourceable | (CDM) | assets |

---

## Key Design Decisions

1. **CDM Integration**: All entities implement CDM interfaces (CogniteDescribable, CogniteSourceable) for consistency
2. **Asset Hierarchy**: Uses CogniteAsset for parent/children relationships with SAP functional locations
3. **Event Unification**: Single Event entity for all event types (lab tests, quality inspections, downtime, work orders)
4. **Reel/Roll Model**: ISA-95 aligned - Reel as Batch, Roll as MaterialLot
5. **Time Series Link**: MfgTimeSeries uses `timeseries` property type for CDF preview/sparkline functionality

---

## Current Data Statistics

| Entity | Count | Source |
|--------|-------|--------|
| Asset | 44,898 | SAP Asset Hierarchy |
| MfgTimeSeries | 3,532 | PI Server |
| Event | 92,000+ | SAP, Proficy, Fabric |
| Reel | 83,600+ | Fabric PPR |
| Roll | 1,000+ | Fabric PPR |
| CogniteFile | 97 | SharePoint |
| Material | TBD | SAP |
| RollQuality | TBD | SharePoint |

---

## GraphQL Query Example

```graphql
{
  listAsset(first: 10) {
    items {
      externalId
      name
      assetType
      sapFunctionalLocation
      parent {
        externalId
        name
      }
      children {
        items {
          externalId
          name
        }
      }
      events {
        items {
          externalId
          eventType
          startTime
        }
      }
    }
  }
}
```

---

> **Note:** This is a working document for discussion purposes. The model may evolve based on feedback.
