# Testing Framework for Construction Cost Calculator

## Overview
This document outlines a comprehensive testing strategy to ensure accuracy, reliability, and usability of the construction cost calculator. The testing framework covers unit tests, integration tests, accuracy validation, and user acceptance testing.

## 1. Test Categories and Coverage

### 1.1 Unit Testing Framework

#### Calculation Function Tests
```javascript
describe('Construction Calculation Engine', () => {
    
    describe('Area Calculations', () => {
        test('should calculate built-up area correctly for small plots', () => {
            const result = calculateBuiltupArea(800, 2, 'residential');
            expect(result).toBe(800 * 0.70 * 2); // 1120 sq.ft
        });
        
        test('should apply correct utilization factors', () => {
            expect(calculateBuiltupArea(900, 1, 'residential')).toBe(630);   // 0.70 factor
            expect(calculateBuiltupArea(1500, 1, 'residential')).toBe(1125); // 0.75 factor
            expect(calculateBuiltupArea(2500, 1, 'residential')).toBe(2000); // 0.80 factor
        });
    });
    
    describe('Material Calculations', () => {
        test('should calculate cement bags correctly', () => {
            const concrete = 50; // m³
            const mortar = 20; // m³
            const result = calculateCementRequirement(concrete, mortar, 'M20');
            expect(result).toBe(Math.ceil(50 * 8.0 + 20 * 5.5)); // 510 bags
        });
        
        test('should calculate steel requirement with seismic factors', () => {
            const volumes = {
                foundation: 10,
                columns: 15,
                beams: 12,
                slabs: 25
            };
            const result = calculateSteelRequirement(volumes, 'III');
            const expected = ((10*80) + (15*160) + (12*120) + (25*100)) * 1.2;
            expect(result).toBeCloseTo(expected, 1);
        });
    });
    
    describe('Regional Price Adjustments', () => {
        test('should apply location factors correctly', () => {
            const basePrice = 1000;
            const result = calculateDynamicCost(basePrice, 1.2, 1.4, 1.1, 0.95);
            expect(result.finalPrice).toBeCloseTo(1753.2, 1);
        });
    });
});
```

#### Validation Tests
```javascript
describe('Input Validation', () => {
    test('should reject invalid plot areas', () => {
        expect(() => validatePlotArea(-100)).toThrow('Plot area must be positive');
        expect(() => validatePlotArea(0)).toThrow('Plot area must be positive');
        expect(() => validatePlotArea(100000)).toThrow('Plot area too large');
    });
    
    test('should validate room configurations', () => {
        const config = { bedrooms: 3, bathrooms: 2, kitchen: 1, living: 1 };
        expect(validateRoomConfiguration(config)).toBe(true);
        
        const invalidConfig = { bedrooms: 10, bathrooms: 1 };
        expect(() => validateRoomConfiguration(invalidConfig))
            .toThrow('Bathroom count insufficient for bedroom count');
    });
});
```

### 1.2 Integration Testing

#### API Integration Tests
```javascript
describe('Calculator API Integration', () => {
    let testServer;
    
    beforeAll(async () => {
        testServer = await setupTestServer();
    });
    
    test('should calculate complete estimate', async () => {
        const testInput = {
            plotArea: 1200,
            location: { state: 'MH', city: 'Pune' },
            floors: 2,
            rooms: { bedrooms: 3, bathrooms: 2, kitchen: 1, living: 1 },
            quality: 'standard'
        };
        
        const response = await request(testServer)
            .post('/api/calculate')
            .send(testInput)
            .expect(200);
        
        expect(response.body).toHaveProperty('totalCost');
        expect(response.body).toHaveProperty('breakdown');
        expect(response.body).toHaveProperty('timeline');
        expect(response.body.totalCost).toBeGreaterThan(0);
    });
    
    test('should handle weekly regional pricing updates', async () => {
        // Test with current week pricing
        const mumbaiInput = { plotArea: 1000, location: { state: 'MH', city: 'Mumbai' }};
        const puneInput = { plotArea: 1000, location: { state: 'MH', city: 'Pune' }};
        
        const [mumbaiResponse, puneResponse] = await Promise.all([
            request(testServer).post('/api/calculate').send(mumbaiInput),
            request(testServer).post('/api/calculate').send(puneInput)
        ]);
        
        // Mumbai should be more expensive than Pune
        expect(mumbaiResponse.body.totalCost).toBeGreaterThan(puneResponse.body.totalCost);
        
        // Verify pricing data freshness (within 7 days)
        expect(mumbaiResponse.body.pricingData.lastUpdated).toBeDefined();
        const lastUpdate = new Date(mumbaiResponse.body.pricingData.lastUpdated);
        const daysSinceUpdate = (Date.now() - lastUpdate.getTime()) / (1000 * 60 * 60 * 24);
        expect(daysSinceUpdate).toBeLessThan(7); // Updated within a week
    });
});
```

#### Database Integration Tests
```javascript
describe('Configuration Management', () => {
    test('should update regional pricing weekly', async () => {
        const originalPrice = await getPricingFactor('MH', 'Pune', 'cement');
        
        // Simulate weekly admin update
        await updatePricingFactor('MH', 'Pune', 'cement', 1.15, {
            updateType: 'weekly_admin_update',
            approvedBy: 'regional_manager_pune',
            effectiveDate: new Date().toISOString()
        });
        
        const updatedPrice = await getPricingFactor('MH', 'Pune', 'cement');
        
        expect(updatedPrice).toBe(1.15);
        expect(updatedPrice).not.toBe(originalPrice);
        
        // Verify update metadata
        const updateHistory = await getPricingUpdateHistory('MH', 'Pune', 'cement');
        expect(updateHistory.latest.updateType).toBe('weekly_admin_update');
        expect(updateHistory.latest.approvedBy).toBe('regional_manager_pune');
    });
    
    test('should maintain data consistency during updates', async () => {
        const batchUpdate = [
            { city: 'Mumbai', material: 'cement', factor: 1.2 },
            { city: 'Mumbai', material: 'steel', factor: 1.15 },
            { city: 'Mumbai', material: 'bricks', factor: 1.18 }
        ];
        
        await updateMultiplePricingFactors('MH', batchUpdate);
        
        const verification = await getAllPricingFactors('MH', 'Mumbai');
        expect(verification.cement).toBe(1.2);
        expect(verification.steel).toBe(1.15);
        expect(verification.bricks).toBe(1.18);
    });
});
```

## 2. Accuracy Validation Framework

### 2.1 Historical Data Validation

#### Benchmark Testing Against Known Projects
```javascript
class AccuracyValidator {
    constructor() {
        this.benchmarkProjects = this.loadBenchmarkData();
    }
    
    async validateAccuracy() {
        const results = [];
        
        for (const project of this.benchmarkProjects) {
            const calculatedCost = await this.calculateProjectCost(project.inputs);
            const actualCost = project.actualCost;
            const variance = Math.abs(calculatedCost - actualCost) / actualCost;
            
            results.push({
                projectId: project.id,
                calculated: calculatedCost,
                actual: actualCost,
                variance: variance,
                withinTolerance: variance <= 0.15 // 15% tolerance
            });
        }
        
        return this.analyzeResults(results);
    }
    
    loadBenchmarkData() {
        return [
            {
                id: 'PROJ001',
                inputs: {
                    plotArea: 1200,
                    location: { state: 'KA', city: 'Bangalore' },
                    floors: 2,
                    quality: 'standard'
                },
                actualCost: 2150000,
                completionDate: '2024-08-15'
            },
            {
                id: 'PROJ002', 
                inputs: {
                    plotArea: 800,
                    location: { state: 'TN', city: 'Chennai' },
                    floors: 1,
                    quality: 'premium'
                },
                actualCost: 1680000,
                completionDate: '2024-07-20'
            }
            // More benchmark projects...
        ];
    }
}
```

### 2.2 Cross-Validation Testing

#### Multi-Source Comparison
```javascript
class CrossValidator {
    async compareWithMultipleSources(inputs) {
        const sources = [
            this.calculateWithOurEngine(inputs),
            this.fetchCompetitorEstimate('ultratech', inputs),
            this.fetchCompetitorEstimate('jkcement', inputs),
            this.getContractorQuotes(inputs)
        ];
        
        const estimates = await Promise.all(sources);
        
        return {
            ourEstimate: estimates[0],
            competitorEstimates: estimates.slice(1, 3),
            contractorQuotes: estimates[3],
            variance: this.calculateVariance(estimates),
            recommendation: this.generateRecommendation(estimates)
        };
    }
    
    calculateVariance(estimates) {
        const mean = estimates.reduce((sum, est) => sum + est, 0) / estimates.length;
        const variance = estimates.reduce((sum, est) => sum + Math.pow(est - mean, 2), 0) / estimates.length;
        return Math.sqrt(variance) / mean; // Coefficient of variation
    }
}
```

### 2.3 Regional Accuracy Testing

#### Location-Specific Validation
```javascript
describe('Regional Accuracy Tests', () => {
    const testLocations = [
        { state: 'MH', city: 'Mumbai', expectedRange: [2000, 3500] },
        { state: 'KA', city: 'Bangalore', expectedRange: [1800, 3000] },
        { state: 'TN', city: 'Chennai', expectedRange: [1600, 2800] },
        { state: 'GJ', city: 'Ahmedabad', expectedRange: [1400, 2400] },
        { state: 'UP', city: 'Lucknow', expectedRange: [1200, 2000] }
    ];
    
    testLocations.forEach(location => {
        test(`should provide realistic estimates for ${location.city}`, async () => {
            const standardInput = {
                plotArea: 1000,
                floors: 2,
                quality: 'standard',
                location: location
            };
            
            const result = await calculateConstructionCost(standardInput);
            const costPerSqFt = result.totalCost / (standardInput.plotArea * 2);
            
            expect(costPerSqFt).toBeGreaterThanOrEqual(location.expectedRange[0]);
            expect(costPerSqFt).toBeLessThanOrEqual(location.expectedRange[1]);
        });
    });
});
```

## 3. Performance Testing Framework

### 3.1 Load Testing

#### Concurrent User Simulation
```javascript
describe('Performance Tests', () => {
    test('should handle 100 concurrent calculations', async () => {
        const startTime = Date.now();
        const promises = [];
        
        for (let i = 0; i < 100; i++) {
            promises.push(calculateConstructionCost({
                plotArea: 1000 + (i * 10),
                floors: 1 + (i % 3),
                location: { state: 'MH', city: 'Pune' },
                quality: 'standard'
            }));
        }
        
        const results = await Promise.all(promises);
        const endTime = Date.now();
        const duration = endTime - startTime;
        
        expect(results).toHaveLength(100);
        expect(duration).toBeLessThan(10000); // 10 seconds max
        results.forEach(result => {
            expect(result.totalCost).toBeGreaterThan(0);
        });
    });
    
    test('should maintain response time under load', async () => {
        const testRuns = 50;
        const responseTimes = [];
        
        for (let i = 0; i < testRuns; i++) {
            const start = Date.now();
            await calculateConstructionCost(standardTestInput);
            responseTimes.push(Date.now() - start);
        }
        
        const averageTime = responseTimes.reduce((sum, time) => sum + time, 0) / testRuns;
        expect(averageTime).toBeLessThan(2000); // 2 seconds average
    });
});
```

### 3.2 Memory and Resource Testing

```javascript
describe('Resource Usage Tests', () => {
    test('should not have memory leaks during repeated calculations', async () => {
        const initialMemory = process.memoryUsage().heapUsed;
        
        // Perform 1000 calculations
        for (let i = 0; i < 1000; i++) {
            await calculateConstructionCost(standardTestInput);
            
            // Force garbage collection every 100 iterations
            if (i % 100 === 0 && global.gc) {
                global.gc();
            }
        }
        
        const finalMemory = process.memoryUsage().heapUsed;
        const memoryIncrease = finalMemory - initialMemory;
        
        // Memory increase should be minimal
        expect(memoryIncrease).toBeLessThan(10 * 1024 * 1024); // 10MB max
    });
});
```

## 4. User Experience Testing

### 4.1 Usability Testing Framework

#### Input Validation and Error Handling
```javascript
describe('User Experience Tests', () => {
    test('should provide helpful error messages', async () => {
        const invalidInputs = [
            { plotArea: -100, expectedError: 'Plot area must be positive' },
            { plotArea: 0, expectedError: 'Plot area must be positive' },
            { floors: 0, expectedError: 'Number of floors must be at least 1' },
            { floors: 5, expectedError: 'Maximum 4 floors supported' }
        ];
        
        for (const testCase of invalidInputs) {
            try {
                await calculateConstructionCost(testCase);
                fail('Should have thrown error');
            } catch (error) {
                expect(error.message).toContain(testCase.expectedError);
            }
        }
    });
    
    test('should provide calculation confidence scores', async () => {
        const result = await calculateConstructionCost(standardTestInput);
        
        expect(result).toHaveProperty('confidence');
        expect(result.confidence.score).toBeGreaterThanOrEqual(0);
        expect(result.confidence.score).toBeLessThanOrEqual(100);
        expect(['Low', 'Medium', 'High']).toContain(result.confidence.reliability);
    });
});
```

### 4.2 Mobile Responsiveness Testing

```javascript
describe('Mobile Experience Tests', () => {
    test('should handle reduced precision inputs gracefully', async () => {
        const mobileInput = {
            plotArea: 1000, // User might input round numbers on mobile
            floors: 2,
            rooms: { bedrooms: 3, bathrooms: 2 }, // Simplified input
            location: { state: 'MH', city: 'Pune' },
            quality: 'standard'
        };
        
        const result = await calculateConstructionCost(mobileInput);
        expect(result.totalCost).toBeGreaterThan(0);
        expect(result.breakdown).toBeDefined();
    });
});
```

## 5. Automated Testing Pipeline

### 5.1 Continuous Integration Testing

```yaml
# .github/workflows/test.yml
name: Comprehensive Testing

on: [push, pull_request]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'
      - name: Install dependencies
        run: npm install
      - name: Run unit tests
        run: npm run test:unit
      - name: Upload coverage
        uses: codecov/codecov-action@v1

  accuracy-tests:
    runs-on: ubuntu-latest
    needs: unit-tests
    steps:
      - name: Run accuracy validation
        run: npm run test:accuracy
      - name: Generate accuracy report
        run: npm run generate:accuracy-report

  performance-tests:
    runs-on: ubuntu-latest
    needs: unit-tests
    steps:
      - name: Run performance tests
        run: npm run test:performance
      - name: Check performance thresholds
        run: npm run check:performance-thresholds
```

### 5.2 Test Data Management

```javascript
class TestDataManager {
    static generateTestCase(scenario) {
        const scenarios = {
            'small-house': {
                plotArea: 600,
                floors: 1,
                rooms: { bedrooms: 2, bathrooms: 1, kitchen: 1, living: 1 },
                quality: 'basic'
            },
            'medium-house': {
                plotArea: 1200,
                floors: 2,
                rooms: { bedrooms: 3, bathrooms: 2, kitchen: 1, living: 1 },
                quality: 'standard'
            },
            'large-house': {
                plotArea: 2000,
                floors: 2,
                rooms: { bedrooms: 4, bathrooms: 3, kitchen: 1, living: 2 },
                quality: 'premium'
            }
        };
        
        const base = scenarios[scenario];
        return {
            ...base,
            location: this.getRandomLocation(),
            testId: this.generateTestId(),
            timestamp: new Date().toISOString()
        };
    }
    
    static getRandomLocation() {
        const locations = [
            { state: 'MH', city: 'Mumbai' },
            { state: 'KA', city: 'Bangalore' },
            { state: 'TN', city: 'Chennai' },
            { state: 'DL', city: 'New Delhi' },
            { state: 'GJ', city: 'Ahmedabad' }
        ];
        
        return locations[Math.floor(Math.random() * locations.length)];
    }
}
```

## 6. Monitoring and Alerting

### 6.1 Production Testing

```javascript
class ProductionMonitor {
    async performHealthChecks() {
        const checks = [
            this.checkCalculationAccuracy(),
            this.checkResponseTimes(),
            this.checkDataConsistency(),
            this.checkRegionalPricing()
        ];
        
        const results = await Promise.all(checks);
        
        if (results.some(check => !check.passed)) {
            await this.triggerAlert(results);
        }
        
        return results;
    }
    
    async checkCalculationAccuracy() {
        const testCase = TestDataManager.generateTestCase('medium-house');
        const result = await calculateConstructionCost(testCase);
        
        // Validate result is within expected bounds
        const expectedRange = this.getExpectedRange(testCase);
        const withinBounds = result.totalCost >= expectedRange.min && 
                           result.totalCost <= expectedRange.max;
        
        return {
            check: 'calculation-accuracy',
            passed: withinBounds,
            details: { calculated: result.totalCost, expected: expectedRange }
        };
    }
}
```

This comprehensive testing framework ensures that the construction cost calculator delivers accurate, reliable, and user-friendly results across all scenarios and use cases.