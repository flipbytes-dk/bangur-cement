# API Documentation - Home Construction Cost Calculator

## Overview
This document provides comprehensive API documentation for the home construction cost calculator, including endpoints, request/response formats, authentication, and integration guidelines.

## Base URL
```
Production: https://api.constructioncalculator.com/v1
Development: http://localhost:3000/api/v1
```

## Authentication
The API uses JWT-based authentication for admin endpoints and session-based tracking for calculator operations.

### Headers
```
Content-Type: application/json
X-Session-ID: [optional-session-identifier]
Authorization: Bearer [jwt-token] (for admin endpoints)
```

## 1. Calculator API Endpoints

### 1.1 Calculate Construction Estimate

**Endpoint:** `POST /calculator/estimate`

**Description:** Calculates comprehensive construction cost estimate based on project parameters.

#### Request Body
```json
{
  "plotArea": 1200,
  "floors": 2,
  "location": {
    "state": "MH",
    "city": "Pune",
    "pincode": "411001"
  },
  "rooms": {
    "bedrooms": 3,
    "bathrooms": 2,
    "kitchen": 1,
    "living": 1,
    "other": 0
  },
  "quality": "standard",
  "constructionType": "rcc_frame",
  "roofType": "flat",
  "specialFeatures": [
    "basement",
    "parking_covered",
    "solar_ready"
  ]
}
```

#### Response
```json
{
  "success": true,
  "sessionId": "550e8400-e29b-41d4-a716-446655440000",
  "data": {
    "totalCost": 2450000,
    "costPerSqFt": 1020.83,
    "breakdown": {
      "materials": {
        "structural": {
          "cement": 259500,
          "steel": 187500,
          "aggregates": 94500,
          "admixtures": 12000
        },
        "masonry": {
          "bricks": 165000,
          "mortarSand": 39000
        },
        "finishing": {
          "flooring": 145000,
          "tiles": 98000,
          "paint": 72000,
          "fixtures": 186200
        },
        "mep": {
          "electrical": 143500,
          "plumbing": 165500,
          "hvac": 0
        }
      },
      "labor": {
        "skilled": 245000,
        "semiskilled": 198000,
        "unskilled": 156000,
        "specialist": 87000
      },
      "equipment": {
        "machinery": 45000,
        "tools": 18000,
        "safety": 8500
      },
      "overhead": {
        "contractor": 147000,
        "supervision": 73500,
        "insurance": 24500,
        "utilities": 19600
      }
    },
    "materialBreakdown": {
      "cement": {
        "quantity": 518,
        "unit": "bags",
        "quality": "standard",
        "specifications": "OPC 53 Grade"
      },
      "steel": {
        "quantity": 3500,
        "unit": "kg",
        "quality": "standard",
        "specifications": "Fe 500 TMT"
      },
      "bricks": {
        "quantity": 95000,
        "unit": "pieces",
        "quality": "standard",
        "specifications": "Class A"
      }
    },
    "timeline": {
      "totalDuration": 247,
      "phases": [
        {
          "name": "Planning & Approvals",
          "duration": 46,
          "cost": 215000,
          "startDay": 1,
          "endDay": 46
        },
        {
          "name": "Foundation",
          "duration": 41,
          "cost": 786000,
          "startDay": 47,
          "endDay": 87
        },
        {
          "name": "Structure",
          "duration": 37,
          "cost": 438000,
          "startDay": 88,
          "endDay": 124
        },
        {
          "name": "Masonry",
          "duration": 25,
          "cost": 380000,
          "startDay": 125,
          "endDay": 149
        },
        {
          "name": "MEP Work",
          "duration": 30,
          "cost": 165500,
          "startDay": 150,
          "endDay": 179
        },
        {
          "name": "Finishing",
          "duration": 68,
          "cost": 465500,
          "startDay": 180,
          "endDay": 247
        }
      ]
    },
    "confidence": {
      "score": 87,
      "factors": {
        "dataCompleteness": "High",
        "regionalData": "High",
        "marketConditions": "Medium"
      },
      "reliability": "High",
      "variance": {
        "min": 2205000,
        "max": 2695000
      }
    },
    "cashFlow": [
      {
        "month": 1,
        "phase": "Planning & Approvals",
        "monthlyCost": 107500,
        "cumulativeCost": 107500,
        "percentComplete": 4.4
      }
    ],
    "recommendations": [
      {
        "type": "cost_optimization",
        "title": "Material Grade Optimization",
        "description": "Consider using medium grade materials for non-structural elements",
        "potential_savings": 125000
      }
    ],
    "calculationDate": "2025-08-26T10:30:00Z",
    "validityPeriod": 30,
    "disclaimer": "This estimate is for planning purposes only..."
  }
}
```

#### Error Response
```json
{
  "success": false,
  "errors": [
    {
      "field": "plotArea",
      "message": "Plot area must be between 100 and 50000 sq.ft",
      "code": "INVALID_PLOT_AREA"
    }
  ]
}
```

### 1.2 Get Detailed Breakdown

**Endpoint:** `GET /calculator/breakdown/{sessionId}`

**Description:** Retrieves detailed cost breakdown for a previous calculation.

#### Response
```json
{
  "success": true,
  "data": {
    "sessionId": "550e8400-e29b-41d4-a716-446655440000",
    "detailedBreakdown": {
      "materials": {
        "cement": {
          "breakdown": {
            "foundation": { "quantity": 156, "cost": 54600 },
            "columns": { "quantity": 124, "cost": 43400 },
            "beams": { "quantity": 93, "cost": 32550 },
            "slabs": { "quantity": 145, "cost": 50750 }
          }
        }
      },
      "workBreakdown": {
        "excavation": {
          "volume": 45.6,
          "unit": "cum",
          "rate": 120,
          "cost": 5472
        }
      }
    }
  }
}
```

### 1.3 Get Material Specifications

**Endpoint:** `GET /calculator/materials`

**Description:** Returns available material categories and their specifications.

#### Query Parameters
- `category` (optional): Filter by material category
- `quality` (optional): Filter by quality grade
- `location` (optional): State code for regional availability

#### Response
```json
{
  "success": true,
  "data": {
    "categories": [
      {
        "id": 1,
        "name": "cement",
        "displayName": "Cement",
        "unit": "bag",
        "basePrice": 350,
        "qualityGrades": [
          {
            "id": 1,
            "name": "basic",
            "displayName": "Basic Grade",
            "multiplier": 1.0,
            "brands": ["Local Brand A", "Local Brand B"],
            "specifications": "IS:12269"
          },
          {
            "id": 2,
            "name": "standard",
            "displayName": "Standard Grade",
            "multiplier": 1.3,
            "brands": ["ACC", "Ambuja", "JK Cement"],
            "specifications": "OPC 53 Grade"
          }
        ]
      }
    ]
  }
}
```

### 1.4 Location Data

**Endpoint:** `GET /calculator/locations`

**Description:** Returns available states and cities for location selection.

#### Response
```json
{
  "success": true,
  "data": {
    "states": [
      {
        "code": "MH",
        "name": "Maharashtra",
        "cities": [
          {
            "id": 1,
            "name": "Mumbai",
            "type": "metro",
            "pincodes": ["400001", "400002"]
          },
          {
            "id": 2,
            "name": "Pune",
            "type": "tier1",
            "pincodes": ["411001", "411002"]
          }
        ]
      }
    ]
  }
}
```

## 2. Admin API Endpoints

### 2.1 Regional Pricing Management

**Endpoint:** `GET /admin/pricing/regions`

**Authentication:** Required (Admin role)

**Description:** Retrieves all regional pricing data.

#### Response
```json
{
  "success": true,
  "data": {
    "regions": [
      {
        "stateCode": "MH",
        "stateName": "Maharashtra",
        "cities": [
          {
            "cityId": 1,
            "cityName": "Mumbai",
            "cityType": "metro",
            "materialMultipliers": {
              "cement": 1.15,
              "steel": 1.10,
              "bricks": 1.20
            },
            "laborMultipliers": {
              "skilled": 1.40,
              "semiskilled": 1.30,
              "unskilled": 1.25
            },
            "lastUpdated": "2025-08-20T12:00:00Z"
          }
        ]
      }
    ]
  }
}
```

### 2.2 Update City Pricing

**Endpoint:** `PUT /admin/pricing/cities/{cityId}`

**Authentication:** Required (Admin role)

#### Request Body
```json
{
  "materialMultipliers": {
    "cement": 1.18,
    "steel": 1.12,
    "bricks": 1.22
  },
  "laborMultipliers": {
    "skilled": 1.45,
    "semiskilled": 1.35,
    "unskilled": 1.28
  },
  "effectiveDate": "2025-08-26T00:00:00Z",
  "reason": "Market price increase due to monsoon season"
}
```

#### Response
```json
{
  "success": true,
  "data": {
    "cityId": 1,
    "updatedAt": "2025-08-26T10:30:00Z",
    "updatedBy": "admin@example.com",
    "changeHistory": [
      {
        "field": "materialMultipliers.cement",
        "oldValue": 1.15,
        "newValue": 1.18,
        "timestamp": "2025-08-26T10:30:00Z"
      }
    ]
  }
}
```

### 2.3 Material Management

**Endpoint:** `POST /admin/materials`

**Authentication:** Required (Admin role)

#### Request Body
```json
{
  "name": "reinforcement_steel",
  "displayName": "Reinforcement Steel",
  "category": "structural",
  "unit": "kg",
  "basePrice": 55,
  "wastagePercentage": 8,
  "qualityGrades": [
    {
      "name": "fe415",
      "displayName": "Fe 415 Grade",
      "multiplier": 1.0,
      "specifications": "IS 1786:2008"
    },
    {
      "name": "fe500",
      "displayName": "Fe 500 Grade", 
      "multiplier": 1.15,
      "specifications": "IS 1786:2008"
    }
  ],
  "calculationFormula": {
    "foundation": "volume * 80",
    "columns": "volume * 160",
    "beams": "volume * 120",
    "slabs": "volume * 100"
  }
}
```

### 2.4 Analytics Dashboard

**Endpoint:** `GET /admin/analytics/usage`

**Authentication:** Required (Admin role)

#### Query Parameters
- `startDate`: ISO date string
- `endDate`: ISO date string
- `groupBy`: day|week|month

#### Response
```json
{
  "success": true,
  "data": {
    "summary": {
      "totalCalculations": 15420,
      "uniqueUsers": 8932,
      "averageProjectSize": 1458,
      "popularLocations": [
        {"city": "Mumbai", "count": 2156},
        {"city": "Bangalore", "count": 1923},
        {"city": "Pune", "count": 1687}
      ]
    },
    "trends": [
      {
        "date": "2025-08-20",
        "calculations": 156,
        "averageCost": 2145000,
        "averageArea": 1234
      }
    ],
    "accuracy": {
      "averageVariance": 12.5,
      "confidenceScore": 84,
      "userFeedback": {
        "positive": 76,
        "neutral": 18,
        "negative": 6
      }
    }
  }
}
```

## 3. Webhooks

### 3.1 Calculation Webhook

**Description:** Triggered when a calculation is completed (for analytics/CRM integration).

#### Payload
```json
{
  "event": "calculation.completed",
  "timestamp": "2025-08-26T10:30:00Z",
  "data": {
    "sessionId": "550e8400-e29b-41d4-a716-446655440000",
    "totalCost": 2450000,
    "location": {
      "state": "MH",
      "city": "Pune"
    },
    "projectSize": 1200,
    "quality": "standard",
    "confidence": 87
  }
}
```

## 4. Error Codes

### 4.1 Standard Error Responses

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `INVALID_INPUT` | 400 | Request validation failed |
| `LOCATION_NOT_FOUND` | 400 | Invalid state/city combination |
| `CALCULATION_FAILED` | 500 | Internal calculation error |
| `SESSION_NOT_FOUND` | 404 | Invalid session ID |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `UNAUTHORIZED` | 401 | Invalid or missing authentication |
| `FORBIDDEN` | 403 | Insufficient permissions |

### 4.2 Validation Errors

```json
{
  "success": false,
  "error": {
    "code": "INVALID_INPUT",
    "message": "Validation failed",
    "details": [
      {
        "field": "plotArea",
        "message": "Must be between 100 and 50000",
        "code": "OUT_OF_RANGE"
      },
      {
        "field": "location.state",
        "message": "Invalid state code",
        "code": "INVALID_STATE"
      }
    ]
  }
}
```

## 5. Rate Limiting

### 5.1 Default Limits
- **Calculator API**: 100 requests per hour per IP
- **Admin API**: 1000 requests per hour per authenticated user
- **Bulk operations**: 10 requests per hour per authenticated user

### 5.2 Headers
Response includes rate limit information:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1693056000
```

## 6. SDK Examples

### 6.1 JavaScript/Node.js

```javascript
const ConstructionCalculator = require('@bangur/construction-calculator');

const client = new ConstructionCalculator({
  apiKey: 'your-api-key',
  baseUrl: 'https://api.constructioncalculator.com/v1'
});

// Calculate estimate
const estimate = await client.calculate({
  plotArea: 1200,
  floors: 2,
  location: { state: 'MH', city: 'Pune' },
  rooms: { bedrooms: 3, bathrooms: 2, kitchen: 1, living: 1 },
  quality: 'standard'
});

console.log(`Total cost: ₹${estimate.totalCost}`);
```

### 6.2 Python

```python
from bangur_calculator import ConstructionCalculator

client = ConstructionCalculator(
    api_key='your-api-key',
    base_url='https://api.constructioncalculator.com/v1'
)

estimate = client.calculate(
    plot_area=1200,
    floors=2,
    location={'state': 'MH', 'city': 'Pune'},
    rooms={'bedrooms': 3, 'bathrooms': 2, 'kitchen': 1, 'living': 1},
    quality='standard'
)

print(f"Total cost: ₹{estimate['total_cost']}")
```

### 6.3 cURL Examples

```bash
# Calculate estimate
curl -X POST https://api.constructioncalculator.com/v1/calculator/estimate \
  -H "Content-Type: application/json" \
  -H "X-Session-ID: my-session-123" \
  -d '{
    "plotArea": 1200,
    "floors": 2,
    "location": {"state": "MH", "city": "Pune"},
    "rooms": {"bedrooms": 3, "bathrooms": 2, "kitchen": 1, "living": 1},
    "quality": "standard"
  }'

# Get materials list
curl -X GET "https://api.constructioncalculator.com/v1/calculator/materials?category=structural" \
  -H "Content-Type: application/json"

# Update regional pricing (admin)
curl -X PUT https://api.constructioncalculator.com/v1/admin/pricing/cities/1 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-jwt-token" \
  -d '{
    "materialMultipliers": {"cement": 1.18, "steel": 1.12},
    "reason": "Market adjustment"
  }'
```

## 7. Integration Guidelines

### 7.1 Best Practices
1. **Session Management**: Use consistent session IDs for tracking user journeys
2. **Caching**: Cache material and location data locally, refresh daily
3. **Error Handling**: Implement exponential backoff for failed requests
4. **Validation**: Validate inputs client-side before API calls
5. **Security**: Never expose admin API keys in frontend code

### 7.2 Testing
Use the sandbox environment for development:
```
Sandbox URL: https://sandbox-api.constructioncalculator.com/v1
Test API Key: sk_test_123456789
```

### 7.3 Support
- **Documentation**: https://docs.constructioncalculator.com
- **Support Email**: support@bangurcement.com
- **Status Page**: https://status.constructioncalculator.com

This API documentation provides complete integration guidance for developers building applications with the construction cost calculator.