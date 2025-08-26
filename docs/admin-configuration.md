# Admin Configuration System - Dynamic Backend Management

## Overview
This document outlines the administrative interface and configuration system that enables dynamic management of calculator parameters, regional pricing, and system behavior without code changes.

## 1. Admin Dashboard Architecture

### 1.1 Core Configuration Modules

#### Regional Pricing Management
```json
{
  "regionalPricing": {
    "states": [
      {
        "stateId": "MH",
        "stateName": "Maharashtra", 
        "baseCostMultiplier": 1.0,
        "cities": [
          {
            "cityId": "MH001",
            "cityName": "Mumbai",
            "cityType": "metro",
            "materialMultipliers": {
              "cement": 1.15,
              "steel": 1.10,
              "aggregates": 1.25,
              "bricks": 1.20,
              "sand": 1.30
            },
            "laborMultipliers": {
              "skilled": 1.40,
              "semiskilled": 1.30,
              "unskilled": 1.25
            },
            "lastUpdated": "2025-08-26T10:00:00Z"
          }
        ]
      }
    ]
  }
}
```

#### Material Database Configuration
```json
{
  "materials": {
    "cement": {
      "materialId": "MAT001",
      "name": "Cement OPC 53",
      "unit": "bag",
      "standardWeight": 50,
      "basePrice": 350,
      "qualityGrades": {
        "basic": {
          "multiplier": 1.0,
          "brands": ["Local Brand A", "Local Brand B"],
          "specifications": "IS:12269"
        },
        "standard": {
          "multiplier": 1.3,
          "brands": ["ACC", "Ambuja", "JK Cement"],
          "specifications": "OPC 53 Grade"
        },
        "premium": {
          "multiplier": 1.6,
          "brands": ["UltraTech", "Shree Cement"],
          "specifications": "OPC 53 Grade with additives"
        }
      },
      "wastagePercentage": 2,
      "storageRequirements": "Dry storage, 3 months shelf life",
      "transportationCost": 15,
      "isActive": true
    }
  }
}
```

### 1.2 Dynamic Formula Configuration

#### Calculation Rules Engine
```json
{
  "calculationRules": {
    "foundations": {
      "shallow": {
        "formula": "builtupArea * depth * 0.3",
        "parameters": {
          "depth": {
            "goodSoil": 1.5,
            "mediumSoil": 2.0,
            "poorSoil": 2.5,
            "unit": "meters"
          }
        },
        "applicability": {
          "floors": "<=3",
          "soilBearing": ">=150kN/m2"
        }
      },
      "deep": {
        "formula": "builtupArea * pileLength * pileCount * 0.25",
        "parameters": {
          "pileLength": {"min": 8, "max": 25},
          "pileCount": "builtupArea / 25",
          "unit": "meters"
        }
      }
    }
  }
}
```

## 2. Admin Interface Components

### 2.1 Regional Management Interface

#### State and City Management
```javascript
class RegionalManager {
    constructor() {
        this.statesList = [];
        this.citiesList = [];
    }
    
    // Add new state with base parameters
    addState(stateData) {
        const newState = {
            stateId: this.generateStateId(),
            stateName: stateData.name,
            baseCostMultiplier: stateData.baseCostMultiplier || 1.0,
            climateZone: stateData.climateZone,
            seismicZone: stateData.seismicZone,
            cities: [],
            isActive: true,
            createdDate: new Date().toISOString()
        };
        
        return this.saveState(newState);
    }
    
    // Bulk update pricing for multiple cities
    bulkUpdatePricing(stateId, priceAdjustments) {
        const updatePromises = [];
        
        priceAdjustments.forEach(adjustment => {
            updatePromises.push(
                this.updateCityPricing(stateId, adjustment.cityId, adjustment.multipliers)
            );
        });
        
        return Promise.all(updatePromises);
    }
}
```

### 2.2 Material Management System

#### Dynamic Material Configuration
```javascript
class MaterialManager {
    // Add new material category
    addMaterialCategory(categoryData) {
        const material = {
            materialId: this.generateMaterialId(),
            name: categoryData.name,
            category: categoryData.category,
            unit: categoryData.unit,
            basePrice: categoryData.basePrice,
            qualityGrades: this.createQualityGrades(categoryData.grades),
            calculationFactors: {
                wastagePercentage: categoryData.wastage || 5,
                transportMultiplier: categoryData.transport || 1.1,
                seasonalVariation: categoryData.seasonal || {}
            },
            suppliers: [],
            specifications: categoryData.specifications,
            isActive: true
        };
        
        return this.saveMaterial(material);
    }
    
    // Configure calculation formulas for materials
    setCalculationFormula(materialId, formula) {
        const formulaConfig = {
            materialId: materialId,
            formula: formula.expression,
            variables: formula.variables,
            conditions: formula.conditions,
            validationRules: formula.validation
        };
        
        return this.saveFormula(formulaConfig);
    }
}
```

### 2.3 Quality Grade Management

#### Dynamic Quality Configuration
```javascript
const qualityGradeManager = {
    // Define quality grades for any material
    defineQualityGrades(materialType, grades) {
        const gradeDefinition = {
            materialType: materialType,
            grades: grades.map(grade => ({
                gradeId: this.generateGradeId(),
                name: grade.name,
                multiplier: grade.multiplier,
                description: grade.description,
                specifications: grade.specifications,
                availableRegions: grade.regions || 'all',
                minimumQuantity: grade.minQuantity || 0,
                leadTime: grade.leadTime || 0
            }))
        };
        
        return this.saveGradeDefinition(gradeDefinition);
    },
    
    // Update pricing for specific grade
    updateGradePricing(gradeId, newMultiplier, effectiveDate) {
        return this.updateGradeData(gradeId, {
            multiplier: newMultiplier,
            effectiveDate: effectiveDate,
            updatedBy: this.getCurrentAdminId(),
            updateReason: 'Market price adjustment'
        });
    }
};
```

## 3. Configuration Management System

### 3.1 Version Control for Configurations

#### Configuration Versioning
```javascript
class ConfigVersionManager {
    createVersion(configType, changes, adminId) {
        const version = {
            versionId: this.generateVersionId(),
            configType: configType,
            changes: changes,
            previousVersion: this.getCurrentVersion(configType),
            createdBy: adminId,
            createdDate: new Date().toISOString(),
            status: 'draft',
            approvals: []
        };
        
        return this.saveVersion(version);
    }
    
    // Rollback to previous version
    rollbackToVersion(configType, versionId) {
        const targetVersion = this.getVersion(versionId);
        const rollbackRecord = {
            rollbackId: this.generateRollbackId(),
            fromVersion: this.getCurrentVersion(configType),
            toVersion: targetVersion,
            rollbackDate: new Date().toISOString(),
            reason: 'Admin rollback request'
        };
        
        return this.performRollback(rollbackRecord);
    }
}
```

### 3.2 Approval Workflow System

#### Multi-level Approval Process
```javascript
class ApprovalWorkflow {
    defineWorkflow(configType, approvalLevels) {
        const workflow = {
            configType: configType,
            levels: approvalLevels.map((level, index) => ({
                levelId: index + 1,
                name: level.name,
                requiredRole: level.role,
                requiredCount: level.count || 1,
                timeout: level.timeout || 72, // hours
                escalation: level.escalation
            })),
            isActive: true
        };
        
        return this.saveWorkflow(workflow);
    }
    
    // Submit configuration for approval
    submitForApproval(configId, submitterId) {
        const submission = {
            submissionId: this.generateSubmissionId(),
            configId: configId,
            submitterId: submitterId,
            currentLevel: 1,
            status: 'pending',
            submissionDate: new Date().toISOString(),
            approvalHistory: []
        };
        
        return this.processSubmission(submission);
    }
}
```

## 4. Real-time Configuration Updates

### 4.1 Live Configuration Sync

#### Real-time Update System
```javascript
class LiveConfigSync {
    constructor() {
        this.subscribers = new Map();
        this.updateQueue = [];
    }
    
    // Subscribe to configuration changes
    subscribe(configType, callback) {
        if (!this.subscribers.has(configType)) {
            this.subscribers.set(configType, []);
        }
        this.subscribers.get(configType).push(callback);
    }
    
    // Broadcast configuration updates
    broadcastUpdate(configType, updateData) {
        const subscribers = this.subscribers.get(configType) || [];
        
        subscribers.forEach(callback => {
            try {
                callback({
                    type: configType,
                    data: updateData,
                    timestamp: new Date().toISOString()
                });
            } catch (error) {
                console.error('Error notifying subscriber:', error);
            }
        });
    }
    
    // Handle bulk updates
    processBulkUpdate(updates) {
        const updatePromises = updates.map(update => 
            this.processIndividualUpdate(update)
        );
        
        return Promise.all(updatePromises);
    }
}
```

### 4.2 Cache Management

#### Smart Cache Invalidation
```javascript
class ConfigurationCache {
    constructor() {
        this.cache = new Map();
        this.dependencies = new Map();
    }
    
    // Set cache with dependencies
    set(key, value, dependencies = []) {
        this.cache.set(key, {
            value: value,
            timestamp: Date.now(),
            dependencies: dependencies
        });
        
        // Register reverse dependencies
        dependencies.forEach(dep => {
            if (!this.dependencies.has(dep)) {
                this.dependencies.set(dep, new Set());
            }
            this.dependencies.get(dep).add(key);
        });
    }
    
    // Invalidate cache based on configuration changes
    invalidateByDependency(dependency) {
        const dependentKeys = this.dependencies.get(dependency);
        
        if (dependentKeys) {
            dependentKeys.forEach(key => {
                this.cache.delete(key);
                console.log(`Invalidated cache for key: ${key}`);
            });
            
            this.dependencies.delete(dependency);
        }
    }
}
```

## 5. Analytics and Monitoring

### 5.1 Usage Analytics Dashboard

#### Configuration Usage Tracking
```javascript
class ConfigAnalytics {
    trackConfigUsage(configType, operation, metadata = {}) {
        const event = {
            eventId: this.generateEventId(),
            configType: configType,
            operation: operation, // 'create', 'update', 'delete', 'view'
            metadata: metadata,
            timestamp: new Date().toISOString(),
            adminId: this.getCurrentAdminId(),
            ipAddress: this.getClientIP()
        };
        
        return this.logEvent(event);
    }
    
    // Generate usage reports
    generateUsageReport(dateRange, configTypes) {
        const reportData = {
            period: dateRange,
            configTypes: configTypes,
            summary: {
                totalOperations: 0,
                byType: {},
                byAdmin: {},
                errorRate: 0
            },
            details: []
        };
        
        return this.buildReport(reportData);
    }
}
```

### 5.2 Performance Monitoring

#### Configuration Performance Metrics
```javascript
class ConfigPerformanceMonitor {
    measureOperationTime(operation, callback) {
        const startTime = performance.now();
        
        return callback().then(result => {
            const endTime = performance.now();
            const duration = endTime - startTime;
            
            this.recordMetric({
                operation: operation,
                duration: duration,
                timestamp: new Date().toISOString(),
                success: true
            });
            
            return result;
        }).catch(error => {
            const endTime = performance.now();
            const duration = endTime - startTime;
            
            this.recordMetric({
                operation: operation,
                duration: duration,
                timestamp: new Date().toISOString(),
                success: false,
                error: error.message
            });
            
            throw error;
        });
    }
}
```

## 6. Data Validation and Integrity

### 6.1 Configuration Validation Rules

#### Data Integrity Checks
```javascript
class ConfigValidator {
    validateMaterialConfig(materialData) {
        const rules = [
            {
                field: 'basePrice',
                validator: (value) => value > 0 && value < 100000,
                message: 'Base price must be between 0 and 100,000'
            },
            {
                field: 'wastagePercentage',
                validator: (value) => value >= 0 && value <= 50,
                message: 'Wastage percentage must be between 0% and 50%'
            },
            {
                field: 'qualityGrades',
                validator: (grades) => grades.length >= 2,
                message: 'At least 2 quality grades required'
            }
        ];
        
        return this.applyValidationRules(materialData, rules);
    }
    
    // Cross-reference validation
    validatePriceConsistency(stateData) {
        const inconsistencies = [];
        
        stateData.cities.forEach(city => {
            Object.keys(city.materialMultipliers).forEach(material => {
                const multiplier = city.materialMultipliers[material];
                if (multiplier < 0.5 || multiplier > 3.0) {
                    inconsistencies.push({
                        city: city.cityName,
                        material: material,
                        multiplier: multiplier,
                        issue: 'Multiplier outside expected range (0.5-3.0)'
                    });
                }
            });
        });
        
        return inconsistencies;
    }
}
```

This admin configuration system provides comprehensive control over all calculator parameters while maintaining data integrity and enabling real-time updates without system downtime.