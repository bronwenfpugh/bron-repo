# Equipment Selection PRD
## Overview 
Create a webapp for residential HVAC contractors that matches users' ServiceTitan pricebook equipment to the home's Manual J load using equipment nominal output (in tons for ACs and heat pumps, and in BTUs for furnaces and boilers). Users can further filter their pricebook equipment by brand, equipment type (furnace, AC, heat pump, boiler, or furnace+AC), staging (single, two, variable), system type (heating, cooling, heating & cooling), distribution type (ducted, ductless, hydronic, mixed), AFUE for furnaces if available, and cabinet size if available.

## User Stories
- As a user, I want to filter the equipment in my ServiceTitan pricebook so that I see only my equipment that matches the job's Manual J load.
- As a user, I want simple guidance about how to adjust output sizes for my climate.

## Implementation Phases 
### Phase 1: Filter on Nominal tonnage
- Users can enter the home's Manual J inputs: Total heating BTU, Total cooling BTU, Sensible cooling BTU. The app should use those inputs to infer latent cooling BTU and sensible heat ratio. User can also enter design conditions: Outdoor winter design temperature in degrees F (_at_outdoor_winter_99_pct_db), outdoor summer design temperature in degrees F (_at_outdoor_summer_1_pct_db), indoor summer design temperature in degrees F (_indoor_design_cooling_db), indoor winter design temperature in degrees F (_indoor_design_heating_db), relative humidity (percentage), and elevation in feet (_elevation). Finally, they can indicate whether the system is zoned.
- The app shows all the users' pricebook equipment that meets the load
- Furnaces: List furnaces with 'actual output capacity' within 100-140% of heating load. Also show furnaces with 'actual output capacity' within 140-200% of heating load, but with a disclaimer: "This furnace is x% oversized. Use only if the AC system requires more blower power to accommodate the cooling load." Calculate 'actual output capacity' as 'heatingCapacityBtu' * 'afue'
- ACs: Use same rules as heat pump sized to cooling (below) - except remove the "backup heat needed" instruction
- Heat pumps: If the heating load is greater than the cooling load, and the load sensible heat ratio < 0.95, and the user selects "size to cooling load," list single_stage, two_stage, or variable_speed equipment sized to the cooling load. If the users selects single_stage equipment, if the total cooling load ≤ 24,000 BTU, list single_stage equipment within 90-120% of (total cool load/12,000); and if the total cooling load >24,000, list single_stage equipment within 90-115% of (total cool load/12,000). If the user selects two_stage equipment, list two_stage equipment within 90-125% of (total cool load/12,000). If the user selects variable_speed equipment, list variable_speed equipment within 90-130% of (total cool load/12,000). Also add this instruction: "Be sure to add backup heat. x kw of backup heat are required." (Calculate backup heat requirement in kilowatts as (totalheatingbtu-heatingcapacitybtu)/3412)). Also add this instruction: "Use OEM data to verify the system has x 'latentcoolingbtu' BTU min latent capacity."
- Heat pumps: If the heating load is greater than the cooling load, and the load sensible heat ratio < 0.95, and the user selects "size to heating load": List heat pump equipment within 100-150% of (heating load/12,000). Also add this instruction: "A system sized for heating will be oversized for cooling and struggle to remove moisture. Add a standalone dehumidifier and use OEM data to verify that the system turns down to <80% of total cooling load." If the system is zoned, 
- Heat pumps: If the heating load is greater than the cooling load, and the load sensible heat ratio is > 0.95, List heat pump equipment within 100-150% of (heating load/12,000). Also add this instruction: "Sizing the system to heating will oversize it for cooling. For optimal comfort and to avoid wear & tear on the system, use performance data to verify that it turns down to <80% of total cooling load."
- Heat pumps: If the heating load is less than the cooling load: Use same rules as heat pump sized to cooling (above) - except remove the "backup heat needed" instruction




### Phase 2: Additional User Filters
- Users can select additional filter criteria: Brand, equipment type (furnace, AC, heat pump, boiler, or AHRI-matched furnace + AC), staging (single, two, variable), equipment type (heating, cooling, heating & cooling), distribution type (ducted, ductless, hydronic, mixed), AFUE for furnaces if available, and cabinet size if available. For heat pumps, users can toggle select whether to size the equipment to the heating load or to the cooling load.
- Users can then view the filtered pricebook equipment
- If equipment type = "furnace" or "furnace + AC," distribution type will be automatically set to "ducted." If equipment type = "boiler," distribution type will automatically be set to "hydronic." If equipment type = "AC" or "heat pump," prompt user to select distribution type (ducted, ductless, hydronic). 
- If distribution type = ducted, include this instruction: "Remember to check whether the fan is powerful enough (static pressure) to move the air through the duct system. If not, consider modifying ductwork, increasing return size, adjusting airflow, or selecting a smaller system size."
- If "equipment type" = "furnace" or "boiler," then "system type" will be automatically set to "heating." If "equipment type" = "furnace + AC," "system type" will be automatically set to "heating & cooing"). If "euqipment type" = "AC" then "system type" will be automatically set to "cooling." IF "equipment type" = "heat pump" then "system type" will be automatically set to "heating & cooling."
- If system type = "cooling" and distribution = ducted, show this instruction: "verify that the exsting duct system is capable of handling at least x CFM." Calculate recommended CFM as (the closest standard tonnage that is > (total cool load/12,000)) * recommended CFM/ton. Standard tonnages are 1.5 ton, 2 ton, 2.5 ton, 3 ton, 4 ton and 5 ton. If the load SHR < 0.85, recommended CFM/ton = 350. If the load SHR is between 0.85 and 0.95, recommended CFM/ton = 400. If the load SHR > 0.95, recommended CFM/ton = 450.
- If "elevation" > 1,000 feet and "equipment type" = "furnace," give this instruction: "Cooling and fossil fuel heating capacity will  decrease (about 2–4% reduction in capacity per 1,000 feet above sea level)."
- If system type = cooling and outdoor summer design temp >95F, give this instruction: "High outdoor temperatures affect system capacity. Verify capacity for your area using performance data."
- If system type = cooling and SHR > 0.95, give this instruction: "Dry climates affect system capacity. Verify capacity for your area using performance data."
- If outdoor winter db > 30F and system type = heating and equipment type = HP, give this instruction: "Heat pumps lose capacity as outdoor temps drop. Use a cold climate heat pump and make sure that the system will meet the heating load at your outdoor winter design temp. (We recommend using a balance point chart.)"
## Design System
Color Palette
Primary Colors:

Carbon: #101010 (primary text, high contrast elements)
Slate 1: #606060 (secondary text)
Slate 2: #757575 (tertiary text)
Dust 1: #EAEAEA (light backgrounds, dividers)
Dust 2: #F0F0F0 (subtle backgrounds)
Dust 3: #F6F6F6 (card backgrounds, input fields)
White: #FFFFFF (main background, content areas)

Accent Colors:

Electric Purple: #683F4 (primary brand color, CTAs)
Radiant Orange: #F7685A (warnings, alerts)

Status Colors:

Warning Orange: #FF8C80 (warning states)
Success Green: #00CD42 (success states, available reports)
Error Red: #E84456 (error states, destructive actions)

Story Editor Colors (for data visualization):

Default Purple: #8B69C3 (primary charts/graphs)
Default Slate: #92AC97 (secondary data)
Variant Coral: #F18777 (highlighting, important data points)
Variant Teal: #709C91 (supplementary data)
Variant Magenta: #D279C0 (accent data)
Variant Gold: #DFC470 (special indicators)

Typography
Font Family: Archivo (primary font for all text)
Hierarchy:

Headline 1: SemiBold 32px/35px (main page titles)
Headline 2: SemiBold 24px/28px (section headers)
Headline 3: SemiBold 22px/25px (subsection headers)
Headline 4: SemiBold 20px/25px (card titles)
Headline 5: SemiBold 18px/20px (component headers)
Headline 6: SemiBold 16px/20px (small headers)

Body Text:

Button Large: SemiBold 15px/16px (primary CTAs)
Button Normal: Medium 15px/16px (secondary buttons)
Button Small: Medium 13px/15px (tertiary buttons)
Form Entry: Regular 15px/20px (input fields, form labels)
Description Large: Light 15px/20px (help text, descriptions)
Description Regular: Regular 13px/15px (body text)
Description Light: Light 13px/15px (secondary body text)

Micro Text:

Title Small: Regular/Light 13px/15px (labels, captions)
Title Micro: SemiBold/Regular/Light 10px/12px (tags, status indicators)
Status: Roboto Mono Medium 11.5px/15px (technical data, BTU values)

Component System
Button Hierarchy:

Primary: Electric Purple background, white text (main actions like "Filter Equipment", "Calculate Load")
Primary Outline: Electric Purple border, Electric Purple text (secondary actions)
Secondary: Carbon background, white text (standard actions)
Tertiary: Dust background, Carbon text (minimal actions)
Destructive: Error Red background, white text (delete, clear actions)

Button Sizes:

Large: Higher padding, 15px SemiBold text (primary page actions)
Normal: Standard padding, 15px Medium text (common actions)
Small: Compact padding, 13px Medium text (inline actions, tables)

Input Components:

Form fields use Dust 3 backgrounds with Carbon text
Labels use Description Regular styling
Error states show Error Red borders and text
Success states show Success Green borders
Focus states use Electric Purple borders

Card Components:

Background: White with subtle Dust 1 borders
Headers use Headline 4-6 depending on importance
Content uses Description Regular for body text
Actions placed in bottom-right or header-right areas

Navigation Elements:

Primary navigation uses Carbon text on White backgrounds
Active states use Electric Purple accents
Hover states use Dust 2 backgrounds
Icons from the provided icon set (construction, HVAC-related symbols)

Spacing System
Base Unit: 4px grid system
Common Spacing:

4px: Micro spacing (icon padding, tight elements)
8px: Small spacing (button padding, input padding)
16px: Medium spacing (card padding, section gaps)
24px: Large spacing (component separation)
32px: XL spacing (major section breaks)
48px: XXL spacing (page-level separation)

Layout Guidelines
Grid System:

12-column grid for main content areas
16px gutters between columns
Maximum content width: 1200px
Responsive breakpoints: Mobile (320px+), Tablet (768px+), Desktop (1024px+)

Content Organization:

Equipment filter controls in left sidebar or top panel
Main equipment results in center content area
Load calculation inputs in prominent top section
Use cards for individual equipment items
Implement clear visual hierarchy for BTU/tonnage values

Interactive States:

Hover: Dust 2 backgrounds, Electric Purple accents
Active: Electric Purple backgrounds for selections
Disabled: Dust colors with reduced opacity
Loading: Use skeleton screens with Dust 3 backgrounds


## Data Model
-----
Equipment Data Structure
Each equipment item in the pricebook should contain the following fields:

Core Equipment Properties:

id: Unique identifier (string)
name: Equipment model name/number (string)
brand: Manufacturer brand (string)
equipmentType: Equipment category (enum: "furnace", "ac", "heat_pump", "boiler", "furnace_ac_combo")
price: Equipment cost (number)

Capacity Properties:

heatingCapacityBtu: Heating output in BTUs (number, required for furnaces, heat pumps, boilers)
CapacityTons: Cooling output in tons (number, required for ACs, heat pumps)
coolingCapacityBtu: Total cooling output in BTUs (number, calculated from tons)

Technical Specifications:

staging: Equipment staging type (enum: "single_stage", "two_stage", "variable_speed")
distributionType: Compatible distribution system (enum: "ducted", "ductless", "mixed," "hydronic")
afue: Annual Fuel Utilization Efficiency percentage (number, applicable to furnaces/boilers)
cabinetSize: Physical cabinet dimensions (object, optional)
  length: Cabinet length in inches (number)
  width: Cabinet width in inches (number)
  height: Cabinet height in inches (number, optional)
refrigerant: Refrigerant type (string, optional)

AHRI Matching (for combination systems):

ahriMatched: Boolean indicating if furnace+AC is AHRI certified as a matched system
matchedIndoorModel: Indoor unit model for matched systems (string)
matchedOutdoorModel: Outdoor unit model for matched systems (string)
-------
Manual J Load Inputs
User-provided load calculation data:
Load Requirements:

totalHeatingBtu: Total heating load in BTUs (number, required)
totalCoolingBtu: Total cooling load in BTUs (number, required)
sensibleCoolingBtu: Sensible cooling load in BTUs (number, required)
latentCoolingBtu: Calculated latent cooling load (totalCooling - sensibleCooling)
sensibleHeatRatio: Calculated SHR (sensibleCooling / totalCooling)

--------
User Preferences
Filtering and sizing preferences:
Sizing Preferences:

heatPumpSizingMode: For heat pumps, size to heating or cooling load (enum: "heating_load", "cooling_load")

Filter Criteria:

selectedBrands: Array of selected brand names (string[])
selectedEquipmentTypes: Array of selected equipment types (string[])
selectedStaging: Array of selected staging options (string[])
selectedDistributionTypes: Array of selected distribution types (string[])
selectedCabinetSizes: Array of selected cabinet sizes (string[])


