
# HVAC Equipment Selector - Product Requirements Document

## Executive Summary

The HVAC Equipment Selector is a web application that enables residential HVAC contractors to quickly match Manual J load calculations with appropriate equipment from their ServiceTitan pricebook. The tool prioritizes speed, accuracy, and ease of use to help contractors complete equipment selection in under 2 minutes while ensuring proper sizing and providing technical guidance.

## Product Overview

### Vision Statement
Streamline HVAC equipment selection for contractors by providing instant, accurate equipment recommendations based on Manual J calculations with comprehensive technical guidance.

### Key Value Propositions
- **Speed**: Complete equipment selection in under 2 minutes
- **Accuracy**: Precise sizing calculations with industry-standard algorithms
- **Guidance**: Clear technical instructions and warnings for each recommendation
- **Flexibility**: View all viable options (excluding grossly oversized equipment), not just "best" picks

## User Personas

### Primary User: Residential HVAC Contractor
- **Experience Level**: Basic to intermediate HVAC knowledge
- **Pain Points**: Time-consuming manual equipment sizing, uncertainty about equipment compatibility
- **Goals**: Quick equipment selection, confident recommendations to customers, avoid callbacks
- **Usage Context**: Field visits, office planning, customer consultations

## Functional Requirements

### 1. Load Input System

#### 1.1 Core Load Inputs
- **Total Heating Load** (BTU/hr): Range 0-500,000
- **Total Cooling Load** (BTU/hr): Range 0-500,000  
- **Sensible Cooling Load** (BTU/hr): Range 0-500,000
- **Auto-calculated values**: Latent cooling load, Sensible Heat Ratio (SHR)

#### 1.2 Advanced Load Inputs (Optional)
- **Outdoor Summer Design Temperature** (°F): Range -30 to 150
- **Outdoor Winter Design Temperature** (°F): Range -30 to 150
- **Elevation** (feet): Range -3,000 to 30,000
- **Indoor Humidity** (%): Range 0-100

#### 1.3 Input Validation Rules
- At least one load (heating or cooling) must be greater than zero
- Sensible cooling cannot exceed total cooling load
- Sensible Heat Ratio must be between 0.65 and 1.0
- Winter design temperature must be less than summer design temperature
- Real-time validation with immediate feedback

### 2. Equipment Type Selection

#### 2.1 Supported Equipment Types
- Furnaces (heating only)
- Air Conditioners (cooling only)
- Heat Pumps (heating & cooling)
- Boilers (heating only)
- Furnace + AC Combos (heating & cooling)

#### 2.2 Sizing Preferences
- **Size to Heating Load**: Prioritize heating capacity
- **Size to Cooling Load**: Prioritize cooling capacity
- **Smart Defaults**: Auto-suggest based on equipment type selection

### 3. Equipment Filtering System

#### 3.1 Basic Filters
- **Distribution Type**: Ducted, Ductless, Hydronic
- **Brand Filter**: Carrier, Trane, Lennox, Rheem, Goodman, American Standard
- **Unit Location**: Indoor, Outdoor, Split System
- **Staging**: Single Stage, Two Stage, Variable Speed

#### 3.2 Advanced Filters
- **Minimum AFUE**: Percentage-based efficiency filter
- **Maximum Price**: Dollar amount limit
- **Cold Climate Heat Pumps**: HSPF ≥ 9.5 filter

### 4. Sizing Algorithm Requirements

#### 4.1 Furnace Sizing Logic
- **Optimal Range**: 100-140% of heating load
- **Acceptable Range**: 90-100% (undersized) and 140-200% (oversized)
- **Elevation Derating**: Apply capacity reduction for elevations >2,000 ft
- **AFUE Consideration**: Use actual output capacity (input × AFUE)

#### 4.2 Air Conditioner Sizing Logic
- **Single Stage**: 90-120% for ≤24,000 BTU, 90-115% for >24,000 BTU
- **Two Stage**: 90-125% of cooling load
- **Variable Speed**: 90-130% of cooling load
- **Moisture Removal**: Special handling for high humidity environments (SHR <0.95)

#### 4.3 Heat Pump Sizing Logic
- **Complex Decision Tree** based on:
  - Load dominance (heating vs cooling)
  - Sensible Heat Ratio
  - User sizing preference
  - Equipment staging type
- **Backup Heat Calculation**: Required when heating capacity < heating load
- **Cold Climate Considerations**: Different rules for HSPF ≥9.5 units

#### 4.4 Boiler Sizing Logic
- **Optimal Range**: 100-125% of heating load
- **Acceptable Range**: 90-100% (undersized) and 125-150% (oversized)

### 5. Results Display System

#### 5.1 Equipment Cards
- **Equipment Image**: Realistic product photo
- **Key Specifications**: Capacity, SEER/AFUE/HSPF, staging, price
- **Sizing Status**: Optimal, Acceptable, Oversized, Undersized
- **Size Match Percentage**: Actual capacity vs required load

#### 5.2 Technical Details (Expandable)
- **Warnings**: Equipment-specific alerts and considerations
- **Instructions**: Installation and verification requirements
- **Backup Heat Requirements**: For heat pumps when applicable
- **CFM Recommendations**: For ducted systems

#### 5.3 Sorting and Organization
- **Primary Sort**: Size match quality (Optimal → Acceptable → Others)
- **Secondary Sort**: Proximity to 100% sizing
- **Alternative Sorts**: Price, Brand, Capacity

### 6. Validation and Error Handling

#### 6.1 Equipment Validation
- **Type Safety**: Verify equipment data integrity
- **Specification Validation**: Ensure required fields for each equipment type
- **Performance Bounds**: Check for reasonable capacity and efficiency values

#### 6.2 User Input Validation
- **Real-time Feedback**: Immediate validation on form inputs
- **Cross-field Validation**: Relationship checks between related inputs
- **Professional Messaging**: Contractor-friendly error descriptions

## Technical Requirements

### 1. Performance Specifications
- **Initial Page Load**: <2 seconds
- **Equipment Filtering**: <500ms response time
- **Form Validation**: <100ms immediate feedback
- **Equipment Database**: Support 50-500 equipment items efficiently

### 2. Data Management
- **Real-time Calculations**: No pre-computation required
- **Client-side Processing**: Instant filtering and sorting
- **Session-based**: No data persistence required
- **Equipment Database**: Structured TypeScript interfaces

### 3. Responsive Design
- **Mobile-first**: Touch-friendly controls
- **Tablet Optimization**: Field-use scenarios
- **Desktop Support**: Office environments
- **Cross-browser**: Modern browser compatibility

### 4. Error Handling Strategy
- **Graceful Degradation**: Fallbacks for calculation errors
- **User-friendly Messages**: Avoid technical jargon
- **Logging**: Track validation issues for monitoring
- **Recovery Options**: Clear paths to resolve issues

## Acceptance Criteria

### 1. Load Input Form
- [ ] All core load inputs accept valid numerical values within specified ranges
- [ ] Real-time validation prevents invalid combinations (e.g., sensible > total cooling)
- [ ] SHR automatically calculates and displays with 2 decimal precision
- [ ] Advanced inputs are collapsible and optional
- [ ] Form prevents submission when validation errors exist
- [ ] Error messages are contractor-friendly and actionable

### 2. Equipment Type Selection
- [ ] All 5 equipment types are selectable via checkboxes
- [ ] Smart unit location auto-selection based on equipment type
- [ ] Sizing preference appears when relevant equipment types selected
- [ ] At least one equipment type must be selected to proceed

### 3. Equipment Filtering
- [ ] All filters work independently and in combination
- [ ] Filters auto-adjust based on equipment type selection
- [ ] Cold climate filter only appears when heat pumps selected
- [ ] Filter state persists during session
- [ ] "Any" options reset filters to default state

### 4. Sizing Calculations
- [ ] Furnaces show elevation derating when elevation >2,000 ft
- [ ] Heat pumps calculate backup heat requirements accurately
- [ ] AC units provide moisture removal guidance for high humidity
- [ ] All equipment shows accurate size match percentages
- [ ] Sizing status categories (optimal/acceptable/oversized) are correct

### 5. Results Display
- [ ] Equipment sorted by size match quality first
- [ ] All required specifications display correctly
- [ ] Technical details expand/collapse properly
- [ ] Status badges show appropriate colors and icons
- [ ] Equipment images load properly with fallbacks
- [ ] Pricing displays in correct currency format

### 6. Technical Guidance
- [ ] Warnings appear for oversized cooling systems in high humidity
- [ ] Instructions provided for backup heat installation
- [ ] Elevation warnings display for furnaces at high altitude
- [ ] CFM recommendations calculate correctly for ducted systems
- [ ] Cold climate heat pump guidance appears appropriately

### 7. Edge Cases
- [ ] "No matches found" displays when no equipment meets criteria
- [ ] System handles missing optional equipment specifications
- [ ] Large equipment databases (500+ items) perform adequately
- [ ] Extreme load values (very high/low) handle gracefully
- [ ] Invalid equipment data is filtered out with appropriate logging

### 8. User Experience
- [ ] Complete workflow achievable in under 2 minutes
- [ ] Form auto-saves state during session
- [ ] Loading states provide appropriate feedback
- [ ] Mobile interface is fully functional and touch-friendly
- [ ] Keyboard navigation works for accessibility

### 9. Data Integrity
- [ ] Equipment validation catches type mismatches
- [ ] Cross-field validation prevents impossible combinations
- [ ] Calculation results are mathematically correct
- [ ] Price and capacity values display with proper formatting
- [ ] Equipment specifications match database requirements

### 10. Error Recovery
- [ ] Clear error messages guide users to solutions
- [ ] Invalid inputs highlight specific problems
- [ ] System recovers gracefully from calculation errors
- [ ] Network errors provide retry options
- [ ] Validation errors don't block other form interactions

## Success Metrics

### Primary KPIs
- **Time to Completion**: Average <2 minutes from load entry to equipment selection
- **User Satisfaction**: 90%+ of contractors find recommendations helpful
- **Accuracy Rate**: 95%+ of sizing calculations match industry standards
- **Error Rate**: <5% of sessions encounter blocking errors

### Secondary KPIs
- **Feature Adoption**: 80%+ usage of advanced filters
- **Equipment Coverage**: 95%+ of valid equipment passes validation
- **Performance**: 99%+ of operations complete within performance targets
- **Mobile Usage**: 60%+ of sessions on mobile/tablet devices

## Dependencies and Assumptions

### Technical Dependencies
- Modern web browser with JavaScript enabled
- Stable internet connection for initial load
- Equipment database in specified TypeScript format

### Business Assumptions
- Contractors have Manual J load calculations available
- Equipment specifications are accurate and complete
- Pricing data reflects current market values
- Users have basic HVAC knowledge

## Future Considerations

### Phase 2 Enhancements
- ServiceTitan integration for live pricebook data
- Equipment comparison features
- PDF report generation
- Advanced performance data integration

### Phase 3 Possibilities
- Multi-zone system support
- Custom equipment database uploads
- Integration with load calculation software
- Mobile app development

## Conclusion

This PRD defines a comprehensive HVAC equipment selection tool that balances speed, accuracy, and usability. The acceptance criteria ensure the delivered product meets contractor needs while maintaining technical excellence and providing reliable equipment recommendations.
