# Construction Cost Calculator - Calculation Engine Specification

## Overview
This document defines the core calculation algorithms and formulas for the home construction cost calculator, providing precise mathematical models for cost estimation, material quantification, and timeline predictions.

## 1. Core Calculation Algorithms

### 1.1 Area and Volume Calculations

#### Built-up Area Algorithm
```javascript
function calculateBuiltupArea(plotArea, floors, plotType) {
    const utilizationFactors = {
        'small': 0.70,    // < 1000 sq.ft
        'medium': 0.75,   // 1000-2000 sq.ft  
        'large': 0.80     // > 2000 sq.ft
    };
    
    const factor = plotArea < 1000 ? utilizationFactors.small :
                   plotArea <= 2000 ? utilizationFactors.medium : 
                   utilizationFactors.large;
    
    return plotArea * factor * floors;
}
```

#### Structural Volume Calculations
```javascript
function calculateStructuralVolumes(builtupArea, floors, foundationType, soilType) {
    const volumes = {};
    
    // Foundation calculations based on soil type
    const foundationDepth = {
        'good': 1.5,      // meters - good bearing capacity
        'medium': 2.0,    // medium bearing capacity  
        'poor': 2.5       // poor bearing capacity
    };
    
    volumes.foundation = builtupArea * foundationDepth[soilType] * 0.3; // 30% of area for strip foundation
    volumes.columns = builtupArea * 0.04 * floors;  // 4% of area per floor
    volumes.beams = builtupArea * 0.03 * floors;    // 3% of area per floor
    volumes.slabs = builtupArea * 0.15 * floors;    // 150mm thick slab
    
    return volumes;
}
```

### 1.2 Material Quantity Algorithms

#### Cement Calculation Engine
```javascript
function calculateCementRequirement(concreteVolume, mortarVolume, concreteGrade) {
    const cementFactors = {
        'M15': 7.5,   // bags per m³
        'M20': 8.0,   // bags per m³
        'M25': 8.5    // bags per m³
    };
    
    const concreteCement = concreteVolume * cementFactors[concreteGrade];
    const mortarCement = mortarVolume * 5.5; // For 1:4 mortar
    
    return Math.ceil(concreteCement + mortarCement);
}
```

#### Steel Reinforcement Calculator
```javascript
function calculateSteelRequirement(structuralVolumes, seismicZone) {
    const baseFactors = {
        foundation: 80,   // kg per m³
        columns: 160,     // kg per m³
        beams: 120,       // kg per m³  
        slabs: 100        // kg per m³
    };
    
    // Seismic zone multipliers
    const seismicMultipliers = {
        'I': 1.0,
        'II': 1.1, 
        'III': 1.2,
        'IV': 1.3,
        'V': 1.4
    };
    
    let totalSteel = 0;
    Object.keys(baseFactors).forEach(element => {
        totalSteel += structuralVolumes[element] * baseFactors[element];
    });
    
    return totalSteel * seismicMultipliers[seismicZone];
}
```

#### Masonry Material Calculator
```javascript
function calculateMasonryMaterials(builtupArea, floors, wallConfig) {
    const calculations = {};
    
    // Wall area calculation
    const perimeter = Math.sqrt(builtupArea) * 4; // Approximate perimeter
    const wallHeight = 3.0; // meters
    const doorWindowArea = builtupArea * 0.20; // 20% for openings
    
    const totalWallArea = (perimeter * wallHeight * floors) - doorWindowArea;
    
    // Brick calculations (standard size 190x90x90mm)
    calculations.bricks = {
        external: Math.ceil(totalWallArea * 0.6 * 55), // 60% external walls, 55 bricks/m²
        internal: Math.ceil(totalWallArea * 0.4 * 28)  // 40% internal walls, 28 bricks/m²
    };
    
    // Sand calculation for mortar
    calculations.sand = totalWallArea * 0.03; // 30mm mortar thickness
    
    // Cement for brickwork mortar
    calculations.mortarCement = Math.ceil(calculations.sand * 5.5);
    
    return calculations;
}
```

### 1.3 Advanced Cost Modeling

#### Dynamic Pricing Algorithm
```javascript
function calculateDynamicCost(basePrice, locationFactor, qualityMultiplier, seasonFactor, demandIndex) {
    // Base cost with all modifying factors
    const modifiedPrice = basePrice * 
                         locationFactor * 
                         qualityMultiplier * 
                         seasonFactor * 
                         demandIndex;
    
    return {
        basePrice: basePrice,
        locationAdjustment: basePrice * (locationFactor - 1),
        qualityPremium: basePrice * locationFactor * (qualityMultiplier - 1),
        seasonalVariation: basePrice * locationFactor * qualityMultiplier * (seasonFactor - 1),
        demandAdjustment: basePrice * locationFactor * qualityMultiplier * seasonFactor * (demandIndex - 1),
        finalPrice: modifiedPrice
    };
}
```

#### Regional Cost Matrix
```javascript
const regionalFactors = {
    labor: {
        'metro': { skilled: 1.3, semiskilled: 1.2, unskilled: 1.15 },
        'tier1': { skilled: 1.0, semiskilled: 1.0, unskilled: 1.0 },
        'tier2': { skilled: 0.85, semiskilled: 0.9, unskilled: 0.95 },
        'rural': { skilled: 0.7, semiskilled: 0.75, unskilled: 0.85 }
    },
    materials: {
        'metro': { cement: 1.1, steel: 1.05, aggregates: 1.2, bricks: 1.15 },
        'tier1': { cement: 1.0, steel: 1.0, aggregates: 1.0, bricks: 1.0 },
        'tier2': { cement: 0.95, steel: 0.98, aggregates: 0.9, bricks: 0.95 },
        'rural': { cement: 0.9, steel: 0.95, aggregates: 0.8, bricks: 0.85 }
    }
};
```

## 2. Timeline Calculation Engine

### 2.1 Critical Path Analysis
```javascript
function calculateProjectTimeline(builtupArea, floors, complexity, weather, laborAvailability) {
    const baseDurations = {
        planning: 45,          // days
        foundation: Math.ceil(builtupArea / 40),   // sq.ft per day
        structure: Math.ceil(builtupArea / 35),    // sq.ft per day
        masonry: Math.ceil(builtupArea / 60),      // sq.ft per day
        roofing: Math.ceil(builtupArea / 80),      // sq.ft per day
        mepWork: 25,           // days (parallel with other work)
        plastering: Math.ceil(builtupArea / 100),  // sq.ft per day
        flooring: Math.ceil(builtupArea / 75),     // sq.ft per day
        finishing: Math.ceil(builtupArea / 120)    // sq.ft per day
    };
    
    // Apply multipliers
    const complexityFactor = {
        'simple': 1.0,
        'medium': 1.2,
        'complex': 1.4
    };
    
    const weatherFactor = {
        'favorable': 1.0,
        'monsoon': 1.3,
        'extreme': 1.5
    };
    
    const laborFactor = {
        'abundant': 0.9,
        'normal': 1.0,
        'scarce': 1.3
    };
    
    const totalMultiplier = complexityFactor[complexity] * 
                           weatherFactor[weather] * 
                           laborFactor[laborAvailability];
    
    const adjustedDurations = {};
    Object.keys(baseDurations).forEach(phase => {
        adjustedDurations[phase] = Math.ceil(baseDurations[phase] * totalMultiplier);
    });
    
    return adjustedDurations;
}
```

### 2.2 Resource Loading Algorithm
```javascript
function calculateResourceRequirements(timeline, builtupArea) {
    const resourceCalendar = {};
    
    Object.keys(timeline).forEach(phase => {
        const duration = timeline[phase];
        const dailyRequirements = calculateDailyResources(phase, builtupArea / duration);
        
        resourceCalendar[phase] = {
            duration: duration,
            dailyLabor: dailyRequirements.labor,
            dailyMaterials: dailyRequirements.materials,
            dailyEquipment: dailyRequirements.equipment
        };
    });
    
    return resourceCalendar;
}
```

## 3. Cost Breakdown Engine

### 3.1 Detailed Cost Categories
```javascript
function generateDetailedCostBreakdown(materials, labor, equipment, overhead) {
    const breakdown = {
        materials: {
            structural: {
                cement: materials.cement.cost,
                steel: materials.steel.cost,
                aggregates: materials.aggregates.cost,
                admixtures: materials.admixtures?.cost || 0
            },
            masonry: {
                bricks: materials.bricks.cost,
                blocks: materials.blocks?.cost || 0,
                mortarSand: materials.mortarSand.cost
            },
            finishing: {
                flooring: materials.flooring.cost,
                tiles: materials.tiles.cost,
                paint: materials.paint.cost,
                fixtures: materials.fixtures.cost
            },
            mep: {
                electrical: materials.electrical.cost,
                plumbing: materials.plumbing.cost,
                hvac: materials.hvac?.cost || 0
            }
        },
        labor: {
            skilled: labor.skilled.cost,
            semiskilled: labor.semiskilled.cost,
            unskilled: labor.unskilled.cost,
            specialist: labor.specialist.cost
        },
        equipment: {
            machinery: equipment.machinery.cost,
            tools: equipment.tools.cost,
            safety: equipment.safety.cost
        },
        overhead: {
            contractor: overhead.contractor,
            supervision: overhead.supervision,
            insurance: overhead.insurance,
            utilities: overhead.utilities
        }
    };
    
    // Calculate totals
    breakdown.totals = {
        materials: sumNestedObject(breakdown.materials),
        labor: sumNestedObject(breakdown.labor),
        equipment: sumNestedObject(breakdown.equipment),
        overhead: sumNestedObject(breakdown.overhead)
    };
    
    breakdown.grandTotal = Object.values(breakdown.totals).reduce((sum, val) => sum + val, 0);
    
    return breakdown;
}
```

### 3.2 Cash Flow Projection
```javascript
function generateCashFlowProjection(costBreakdown, timeline) {
    const monthlyFlow = [];
    let cumulativeCost = 0;
    
    const phases = Object.keys(timeline);
    phases.forEach((phase, index) => {
        const phaseDuration = timeline[phase];
        const phaseCost = calculatePhaseCost(phase, costBreakdown);
        const monthlyCost = phaseCost / Math.ceil(phaseDuration / 30);
        
        for (let month = 0; month < Math.ceil(phaseDuration / 30); month++) {
            cumulativeCost += monthlyCost;
            monthlyFlow.push({
                month: monthlyFlow.length + 1,
                phase: phase,
                monthlyCost: monthlyCost,
                cumulativeCost: cumulativeCost,
                percentComplete: (cumulativeCost / costBreakdown.grandTotal) * 100
            });
        }
    });
    
    return monthlyFlow;
}
```

## 4. Quality and Accuracy Algorithms

### 4.1 Confidence Scoring
```javascript
function calculateConfidenceScore(inputCompleteness, regionalDataQuality, marketVolatility) {
    const weights = {
        inputCompleteness: 0.4,
        regionalDataQuality: 0.3,
        marketVolatility: 0.3
    };
    
    const score = (inputCompleteness * weights.inputCompleteness) +
                  (regionalDataQuality * weights.regionalDataQuality) +
                  ((1 - marketVolatility) * weights.marketVolatility);
    
    return {
        score: Math.round(score * 100),
        range: calculateVarianceRange(score),
        reliability: score > 0.8 ? 'High' : score > 0.6 ? 'Medium' : 'Low'
    };
}
```

### 4.2 Error Correction Algorithms
```javascript
function applyErrorCorrection(calculatedValue, historicalData, marketConditions) {
    // Apply machine learning based corrections
    const historicalVariance = calculateHistoricalVariance(historicalData);
    const marketAdjustment = getMarketAdjustment(marketConditions);
    
    const correctedValue = calculatedValue * (1 + marketAdjustment) * 
                          (1 + (historicalVariance * 0.1)); // 10% weight to historical variance
    
    return {
        original: calculatedValue,
        corrected: correctedValue,
        adjustment: correctedValue - calculatedValue,
        confidence: calculateConfidenceScore(0.8, 0.7, marketConditions.volatility)
    };
}
```

## 5. Optimization Algorithms

### 5.1 Cost Optimization Engine
```javascript
function optimizeCosts(requirements, constraints, preferences) {
    const optimizations = [];
    
    // Material substitution analysis
    if (preferences.costOptimization === 'aggressive') {
        optimizations.push(...findMaterialSubstitutions(requirements.materials));
    }
    
    // Timeline optimization
    if (constraints.timeline === 'flexible') {
        optimizations.push(...findTimelineOptimizations(requirements.timeline));
    }
    
    // Quality balancing
    if (preferences.qualityFlexibility > 0) {
        optimizations.push(...findQualityBalancing(requirements.quality));
    }
    
    return {
        originalCost: requirements.totalCost,
        optimizedCost: calculateOptimizedCost(requirements, optimizations),
        savings: calculateSavings(optimizations),
        recommendations: optimizations
    };
}
```

## 6. Integration APIs

### 6.1 External Data Integration
```javascript
class ExternalDataIntegrator {
    async fetchMaterialPrices(location, materials) {
        // Integration with material supplier APIs
        const priceData = await Promise.all([
            this.fetchCementPrices(location),
            this.fetchSteelPrices(location),
            this.fetchAggregatePrice(location)
        ]);
        
        return this.normalizePrice(priceData);
    }
    
    async getRegionalFactors(pincode) {
        // Integration with economic data APIs
        const factors = await this.economicDataAPI.getRegionalIndices(pincode);
        return this.processRegionalFactors(factors);
    }
}
```

This calculation engine provides the mathematical foundation for accurate, reliable, and sophisticated cost estimation that will differentiate our calculator from existing market offerings.