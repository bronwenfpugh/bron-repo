# Equipment Selection PRD (Revised)

## Overview 
Create a webapp for residential HVAC contractors that matches users' ServiceTitan pricebook equipment to the home's Manual J load using equipment nominal output. The app prioritizes speed and simplicity while providing precise, reliable equipment recommendations with clear guidance.

**Key Success Metrics:**
- Equipment selection completed in under 2 minutes
- Clear guidance provided for all recommendations
- Handles typical pricebooks of 50-500 equipment items efficiently

## User Stories
- As a busy contractor, I want to quickly enter my Manual J loads and see all viable equipment from my ServiceTitan pricebook in under 2 minutes
- As a contractor with basic HVAC knowledge, I want clear guidance on why each piece of equipment is recommended and any special considerations
- As a contractor, I want to see all viable options (not just "best" picks) so I can make the final decision based on price, availability, and customer preferences

## Implementation Phases 

### Phase 1: Core Equipment Matching (MVP)
**Goal: Get contractors from load input to equipment list in 60 seconds**

**Features:**
- Simple load input form (Total heating BTU, Total cooling BTU, Sensible cooling BTU)
- Basic equipment filtering by nominal capacity
- Clear equipment recommendations with sizing guidance
- ServiceTitan pricebook API integration (mock data for development)

**Equipment Types Supported:**
- Furnaces (heating only)
- Air conditioners (cooling only) 
- Heat pumps (heating & cooling)

### Phase 2: Advanced Filtering & Guidance
**Goal: Add precision filters and comprehensive guidance**

**Features:**
- Additional load inputs (design temperatures, elevation, humidity)
- Equipment filters (brand, staging, distribution type, AFUE)
- Advanced sizing guidance and warnings
- Support for boilers and combination systems

### Phase 3: Enhanced User Experience
**Goal: Polish and optimize for daily use**

**Features:**
- Improved UI/UX based on user feedback
- Performance optimizations for large pricebooks
- Advanced guidance for complex scenarios
- Export/sharing capabilities

---

## Data Model

### Core Equipment Schema
The equipment data structure should be optimized for fast filtering and clear display:

```typescript
interface Equipment {
  // Primary Identifiers
  id: string;                    // Unique ServiceTitan equipment ID
  name: string;                  // Display name (e.g., "Carrier 58CVA090")
  brand: string;                 // Manufacturer name
  model: string;                 // Model number/series
  price: number;                 // Cost in dollars
  
  // Visual Assets
  imageUrl: string;              // Equipment photo URL
  imageAlt: string;              // Alt text for accessibility
  
  // Equipment Classification
  equipmentType: EquipmentType;  // Primary equipment category
  systemType: SystemType;        // What loads it handles
  distributionType: DistributionType; // How it distributes conditioned air/water
  
  // Capacity Data (for sizing calculations)
  capacities: {
    heating?: HeatingCapacity;   // Only present if equipment provides heating
    cooling?: CoolingCapacity;   // Only present if equipment provides cooling
  };
  
  // Technical Specifications
  specifications: {
    staging: StagingType;        // Single, two-stage, or variable
    afue?: number;              // For furnaces/boilers (0-100 percentage)
    seer?: number;              // For cooling equipment
    hspf?: number;              // For heat pumps
    refrigerant?: string;       // R-410A, R-32, etc.
  };
  
  // Physical Properties
  physical?: {
    cabinetSize?: {
      width: number;            // Inches
      height: number;           // Inches  
      depth: number;            // Inches
    };
    weight?: number;            // Pounds
  };
  
  // Matching Information (for combo systems)
  matching?: {
    isAhriMatched: boolean;
    indoorModel?: string;
    outdoorModel?: string;
    matchingSetId?: string;     // Groups matched equipment together
  };
}

// Enums for type safety and filtering
enum EquipmentType {
  FURNACE = 'furnace',
  AC = 'ac', 
  HEAT_PUMP = 'heat_pump',
  BOILER = 'boiler',
  FURNACE_AC_COMBO = 'furnace_ac_combo'
}

enum SystemType {
  HEATING = 'heating',
  COOLING = 'cooling', 
  HEATING_AND_COOLING = 'heating_and_cooling'
}

enum DistributionType {
  DUCTED = 'ducted',
  DUCTLESS = 'ductless',
  HYDRONIC = 'hydronic',
  MIXED = 'mixed'
}

enum StagingType {
  SINGLE_STAGE = 'single_stage',
  TWO_STAGE = 'two_stage',
  VARIABLE_SPEED = 'variable_speed'
}

// Capacity interfaces for clear data handling
interface HeatingCapacity {
  inputBtu: number;             // Rated input capacity
  outputBtu: number;            // Actual heating output (input * efficiency)
  nominalTons?: number;         // For heat pumps (outputBtu / 12000)
}

interface CoolingCapacity {
  totalBtu: number;             // Total cooling capacity
  sensibleBtu: number;          // Sensible cooling capacity
  latentBtu: number;            // Latent cooling capacity (calculated)
  nominalTons: number;          // totalBtu / 12000
}
```

### Manual J Load Input Schema
Simplified for Phase 1, expandable for Phase 2:

```typescript
interface ManualJLoad {
  // Core Load Data (Phase 1)
  totalHeatingBtu: number;      // Required
  totalCoolingBtu: number;      // Required  
  sensibleCoolingBtu: number;   // Required
  
  // Calculated Values (auto-computed)
  latentCoolingBtu: number;     // totalCooling - sensibleCooling
  sensibleHeatRatio: number;    // sensibleCooling / totalCooling
  
  // Design Conditions (Phase 2)
  designConditions?: {
    outdoorWinterDb: number;    // °F
    outdoorSummerDb: number;    // °F  
    indoorWinterDb: number;     // °F (default 70)
    outdoorSummerDb: number;    // °F (default 75)
    relativeHumidity: number;   // % (default 50)
    elevation: number;          // feet above sea level
  };
  
  // System Characteristics (Phase 2)
  systemInfo?: {
    isZoned: boolean;           // Multiple zones/thermostats
    hasExistingDuctwork: boolean;
    ductworkCondition?: 'poor' | 'fair' | 'good' | 'excellent';
  };
}
```

### Equipment Sizing Logic Schema
Encapsulate sizing rules for maintainability:

```typescript
interface SizingRule {
  equipmentType: EquipmentType;
  loadType: 'heating' | 'cooling';
  conditions: SizingCondition[];
  sizeRange: {
    minPercent: number;         // Minimum % of load (e.g., 90)
    maxPercent: number;         // Maximum % of load (e.g., 140)
  };
  guidance: string[];           // Array of guidance messages
  warnings?: string[];          // Array of warning messages
}

interface SizingCondition {
  field: string;                // Field to check (e.g., 'sensibleHeatRatio')
  operator: 'lt' | 'lte' | 'gt' | 'gte' | 'eq' | 'between';
  value: number | [number, number];
}

// Example sizing rules data
const SIZING_RULES: SizingRule[] = [
  {
    equipmentType: EquipmentType.FURNACE,
    loadType: 'heating',
    conditions: [],
    sizeRange: { minPercent: 100, maxPercent: 140 },
    guidance: [
      "Furnace sized appropriately for heating load",
      "Verify blower capacity if pairing with AC"
    ]
  },
  {
    equipmentType: EquipmentType.HEAT_PUMP,
    loadType: 'cooling',
    conditions: [
      { field: 'sensibleHeatRatio', operator: 'lt', value: 0.95 },
      { field: 'heatingLoadRatio', operator: 'gt', value: 1.0 }
    ],
    sizeRange: { minPercent: 90, maxPercent: 120 },
    guidance: [
      "Heat pump sized for cooling load",
      "Backup heat required for heating load"
    ],
    warnings: [
      "Verify latent cooling capacity with OEM data"
    ]
  }
];
```

### Filter State Schema
Track user selections for equipment filtering:

```typescript
interface FilterState {
  // Load-based filters (always applied)
  loadRequirements: ManualJLoad;
  
  // User-selected filters (Phase 2)
  selectedBrands: string[];
  selectedEquipmentTypes: EquipmentType[];
  selectedStaging: StagingType[];
  selectedDistributionTypes: DistributionType[];
  
  // Heat pump specific (Phase 2)
  heatPumpSizingMode?: 'heating_load' | 'cooling_load';
  
  // Capacity filters (Phase 2)
  minAfue?: number;
  maxAfue?: number;
  minSeer?: number;
  maxSeer?: number;
}
```

### Equipment Recommendation Schema
Structure for displaying results to users:

```typescript
interface EquipmentRecommendation {
  equipment: Equipment;
  sizing: {
    matchType: 'excellent' | 'good' | 'acceptable' | 'oversized';
    loadCoverage: {
      heating?: {
        percentage: number;     // % of heating load covered
        adequacy: 'under' | 'adequate' | 'over';
      };
      cooling?: {
        percentage: number;     // % of cooling load covered  
        adequacy: 'under' | 'adequate' | 'over';
      };
    };
  };
  guidance: string[];           // Specific guidance for this equipment
  warnings: string[];           // Warnings or considerations
  priority: number;             // For sorting (lower = higher priority)
}
```

---

## ServiceTitan API Integration

### Mock Data Structure for Development
```typescript
// Mock pricebook data for development/testing
interface MockPricebook {
  contractorId: string;
  lastUpdated: string;
  equipment: Equipment[];
}

// Sample mock data with placeholder images
const MOCK_EQUIPMENT: Equipment[] = [
  {
    id: "carrier-58cva090",
    name: "Carrier 58CVA090",
    brand: "Carrier",
    model: "58CVA090",
    price: 2850,
    imageUrl: "https://via.placeholder.com/300x200/f0f0f0/606060?text=FURNACE",
    imageAlt: "Carrier 58CVA090 Gas Furnace",
    equipmentType: EquipmentType.FURNACE,
    systemType: SystemType.HEATING,
    distributionType: DistributionType.DUCTED,
    capacities: {
      heating: {
        inputBtu: 90000,
        outputBtu: 72000,
      }
    },
    specifications: {
      staging: StagingType.SINGLE_STAGE,
      afue: 80,
    }
  }
];

// API response structure
interface ServiceTitanResponse {
  success: boolean;
  data: Equipment[];
  pagination?: {
    page: number;
    total: number;
    hasMore: boolean;
  };
  error?: string;
}
```

### API Endpoints Expected
```typescript
// GET /api/equipment?contractorId={id}
// Returns: ServiceTitanResponse with equipment array

// POST /api/equipment/filter
// Body: FilterState
// Returns: ServiceTitanResponse with filtered equipment
```

---

## Technical Requirements

### Performance Requirements
- Initial load: < 2 seconds
- Equipment filtering: < 500ms for up to 1000 items
- Responsive design (mobile-first)
- Offline capability for core calculations (Phase 3)

### Data Validation
```typescript
// Input validation schemas
const LOAD_VALIDATION = {
  totalHeatingBtu: { min: 1000, max: 500000, required: true },
  totalCoolingBtu: { min: 1000, max: 500000, required: true },
  sensibleCoolingBtu: { min: 1000, max: 500000, required: true }
};

const DESIGN_TEMP_VALIDATION = {
  outdoorWinterDb: { min: -30, max: 70 },
  outdoorSummerDb: { min: 70, max: 130 },
  indoorTemps: { min: 65, max: 80 }
};
```

### Error Handling
- Graceful API failure handling
- Clear validation error messages
- Fallback calculations when data is missing
- User-friendly error states

---

## Design System

### Color Palette
**Primary Colors:**
- Carbon: #101010 (primary text, high contrast elements)
- Slate 1: #606060 (secondary text)
- Slate 2: #757575 (tertiary text)
- Dust 1: #EAEAEA (light backgrounds, dividers)
- Dust 2: #F0F0F0 (subtle backgrounds)
- Dust 3: #F6F6F6 (card backgrounds, input fields)
- White: #FFFFFF (main background, content areas)

**Accent Colors:**
- Electric Purple: #6834F4 (primary brand color, CTAs)
- Radiant Orange: #F7685A (warnings, alerts)

**Status Colors:**
- Success Green: #00CD42 (success states)
- Warning Orange: #FF8C80 (warning states)
- Error Red: #E84456 (error states)

### Typography
**Font Family:** Archivo

**Hierarchy:**
- Headline 1: SemiBold 32px/35px (main page titles)
- Headline 2: SemiBold 24px/28px (section headers)
- Headline 3: SemiBold 22px/25px (subsection headers)
- Body Regular: Regular 15px/20px (primary body text)
- Body Small: Regular 13px/15px (secondary text)
- Technical: Roboto Mono Medium 11.5px/15px (BTU values, model numbers)

### Component Guidelines
- **Cards:** White background, subtle Dust 1 borders, equipment photo prominently displayed
- **Equipment Images:** 300×200px, rounded corners, hover effects for interactivity
- **Buttons:** Electric Purple primary, standard secondary
- **Forms:** Dust 3 backgrounds, clear validation states
- **Equipment Lists:** Scannable cards with clear capacity display and visual equipment identification
- **Load Inputs:** Prominent, grouped logically
- **Guidance:** Distinct styling for tips vs. warnings

---

## Success Criteria

### Phase 1 Success Metrics
- Load input to equipment results in < 60 seconds
- Clear equipment recommendations with basic guidance
- Handles 100+ equipment items smoothly
- Mobile-responsive interface

### Phase 2 Success Metrics  
- Advanced filtering reduces results by 70%+ when applied
- Comprehensive guidance covers 90% of common scenarios
- Supports all major equipment types

### Phase 3 Success Metrics
- User satisfaction > 8/10 in usability testing
- 50%+ reduction in equipment selection time vs. current methods
- Reliable performance with 500+ equipment pricebooks
