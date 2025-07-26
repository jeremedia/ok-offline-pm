# OK-OFFLINE API Design Specification

## API Design Principles

### 1. Offline-First Compatibility
- **Cacheable Responses**: All responses include caching headers
- **Fallback Support**: Graceful degradation when services unavailable
- **Conflict Resolution**: Handle conflicts when syncing offline changes
- **Minimal Payloads**: Optimize for slow connections

### 2. RESTful Architecture
- **Resource-Based URLs**: Clear, predictable endpoint structure
- **HTTP Methods**: Proper use of GET, POST, PUT, DELETE
- **Status Codes**: Meaningful HTTP status codes for all responses
- **Stateless**: Each request contains all necessary information

### 3. Consistent Response Format
- **JSON API Standard**: Consistent structure across all endpoints
- **Error Handling**: Standardized error response format
- **Metadata**: Include timestamps, pagination, and source info
- **Versioning**: API versioning for backward compatibility

## Current API (Phase 1: Weather Service)

### Base Configuration
```
Base URL: https://api.offline.oknotok.com (planned)
Version: v1
Content-Type: application/json
CORS: Enabled for offline.oknotok.com
```

### Weather Endpoints

#### Get Current Weather
```http
POST /api/v1/weather/current
Content-Type: application/json

{
  "latitude": 40.788645,
  "longitude": -119.203018,
  "dataSets": "currentWeather,forecastDaily",
  "timezone": "America/Los_Angeles"
}
```

**Response Success (200)**:
```json
{
  "data": {
    "temperature": 85,
    "feelsLike": 88,
    "humidity": 15,
    "pressure": 1013,
    "windSpeed": 15,
    "windDirection": "SW",
    "windDegrees": 225,
    "visibility": 10,
    "description": "Clear sky",
    "icon": "01d",
    "dustLevel": "moderate",
    "dustLabel": "Moderate Dust",
    "recommendation": "Dust mask recommended. Secure loose items in camp.",
    "moonPhase": {
      "phase": 0.75,
      "phaseName": "Last Quarter",
      "phaseIcon": "🌗",
      "moonrise": "11:45 PM",
      "moonset": "12:30 PM"
    }
  },
  "meta": {
    "source": "apple-api",
    "lastUpdated": "2025-01-26T20:30:00Z",
    "ttl": 600,
    "coordinates": {
      "latitude": 40.788645,
      "longitude": -119.203018
    }
  }
}
```

**Response Error (400)**:
```json
{
  "error": {
    "code": "invalid_location",
    "message": "Latitude and longitude are required",
    "details": {
      "latitude": "must be between -90 and 90",
      "longitude": "must be between -180 and 180"
    }
  },
  "meta": {
    "timestamp": "2025-01-26T20:30:00Z",
    "requestId": "req_abc123"
  }
}
```

**Response Service Unavailable (503)**:
```json
{
  "error": {
    "code": "weather_service_unavailable",
    "message": "All weather services are currently unavailable"
  },
  "fallback": {
    "data": {
      "temperature": 85,
      "source": "cached_data",
      "lastUpdated": "2025-01-26T19:30:00Z"
    }
  },
  "meta": {
    "timestamp": "2025-01-26T20:30:00Z",
    "retryAfter": 300
  }
}
```

#### Get Weather Forecast
```http
POST /api/v1/weather/forecast
Content-Type: application/json

{
  "latitude": 40.788645,
  "longitude": -119.203018,
  "days": 5,
  "timezone": "America/Los_Angeles"
}
```

**Response Success (200)**:
```json
{
  "data": {
    "forecast": [
      {
        "date": "2025-08-26",
        "dayName": "Tuesday",
        "temperature": {
          "high": 95,
          "low": 65
        },
        "windSpeed": 12,
        "windDirection": "W",
        "humidity": 18,
        "dustLevel": "light",
        "dustLabel": "Light Dust",
        "description": "Partly cloudy"
      }
    ]
  },
  "meta": {
    "source": "openweather-api",
    "lastUpdated": "2025-01-26T20:30:00Z",
    "ttl": 3600,
    "coordinates": {
      "latitude": 40.788645,
      "longitude": -119.203018
    }
  }
}
```

### Health Check Endpoint
```http
GET /health
```

**Response Success (200)**:
```json
{
  "status": "ok",
  "timestamp": "2025-01-26T20:30:00Z",
  "services": {
    "database": "healthy",
    "openweather": "healthy",
    "apple_weather": "healthy"
  },
  "version": "1.0.0"
}
```

## Future API (Phase 2: Play Wisdom)

### Wisdom Endpoints

#### Create Wisdom Entry
```http
POST /api/v1/wisdom
Content-Type: application/json
Authorization: Bearer {jwt_token}

{
  "title": "Effective Shade Structure Setup",
  "content": "Use rebar and guy-lines at 45-degree angles for maximum wind resistance...",
  "category": "shelter_shade",
  "tags": ["shade", "setup", "wind-resistance"],
  "location": {
    "latitude": 40.788645,
    "longitude": -119.203018,
    "address": "7:30 & Esplanade"
  },
  "type": "text"
}
```

**Response Success (201)**:
```json
{
  "data": {
    "id": "wisdom_abc123",
    "title": "Effective Shade Structure Setup",
    "content": "Use rebar and guy-lines at 45-degree angles...",
    "category": "shelter_shade",
    "tags": ["shade", "setup", "wind-resistance"],
    "location": {
      "latitude": 40.788645,
      "longitude": -119.203018,
      "address": "7:30 & Esplanade"
    },
    "type": "text",
    "author": {
      "id": "user_def456",
      "username": "playa_veteran"
    },
    "status": "published",
    "moderationStatus": "approved",
    "createdAt": "2025-01-26T20:30:00Z",
    "updatedAt": "2025-01-26T20:30:00Z"
  },
  "meta": {
    "timestamp": "2025-01-26T20:30:00Z"
  }
}
```

#### List Wisdom Entries
```http
GET /api/v1/wisdom?category=shelter_shade&limit=20&offset=0&search=shade
```

**Response Success (200)**:
```json
{
  "data": [
    {
      "id": "wisdom_abc123",
      "title": "Effective Shade Structure Setup",
      "content": "Use rebar and guy-lines at 45-degree angles...",
      "category": "shelter_shade",
      "tags": ["shade", "setup", "wind-resistance"],
      "type": "text",
      "author": {
        "username": "playa_veteran"
      },
      "createdAt": "2025-01-26T20:30:00Z",
      "mediaCount": 0,
      "likesCount": 15,
      "commentsCount": 3
    }
  ],
  "meta": {
    "pagination": {
      "total": 150,
      "limit": 20,
      "offset": 0,
      "hasMore": true
    },
    "filters": {
      "category": "shelter_shade",
      "search": "shade"
    },
    "timestamp": "2025-01-26T20:30:00Z"
  }
}
```

#### Upload Media for Wisdom Entry
```http
POST /api/v1/wisdom/{id}/media
Content-Type: multipart/form-data
Authorization: Bearer {jwt_token}

{
  "file": [binary data],
  "type": "image",
  "caption": "Properly secured shade structure withstanding 30mph winds"
}
```

**Response Success (201)**:
```json
{
  "data": {
    "id": "media_xyz789",
    "type": "image",
    "url": "https://s3.amazonaws.com/ok-offline-media/wisdom/abc123/image1.jpg",
    "thumbnailUrl": "https://s3.amazonaws.com/ok-offline-media/wisdom/abc123/thumb_image1.jpg",
    "caption": "Properly secured shade structure withstanding 30mph winds",
    "size": 2048576,
    "dimensions": {
      "width": 1920,
      "height": 1080
    },
    "createdAt": "2025-01-26T20:30:00Z"
  },
  "meta": {
    "timestamp": "2025-01-26T20:30:00Z"
  }
}
```

#### Search Wisdom Entries
```http
POST /api/v1/wisdom/search
Content-Type: application/json

{
  "query": "shade setup wind",
  "filters": {
    "category": ["shelter_shade", "safety"],
    "tags": ["wind-resistance"],
    "hasMedia": true,
    "location": {
      "latitude": 40.788645,
      "longitude": -119.203018,
      "radius": 1000
    }
  },
  "sort": "relevance",
  "limit": 20,
  "offset": 0
}
```

**Response Success (200)**:
```json
{
  "data": {
    "results": [
      {
        "id": "wisdom_abc123",
        "title": "Effective Shade Structure Setup",
        "excerpt": "Use rebar and guy-lines at 45-degree angles for maximum wind resistance...",
        "score": 0.95,
        "category": "shelter_shade",
        "tags": ["shade", "setup", "wind-resistance"],
        "type": "text",
        "mediaCount": 2,
        "createdAt": "2025-01-26T20:30:00Z"
      }
    ],
    "suggestions": [
      "wind resistance",
      "shade structures",
      "playa setup"
    ]
  },
  "meta": {
    "query": "shade setup wind",
    "total": 25,
    "limit": 20,
    "offset": 0,
    "processingTime": 0.045,
    "timestamp": "2025-01-26T20:30:00Z"
  }
}
```

### User Management Endpoints

#### User Registration
```http
POST /api/v1/auth/register
Content-Type: application/json

{
  "username": "playa_newbie",
  "email": "newbie@example.com",
  "password": "secure_password",
  "profile": {
    "displayName": "Playa Newbie",
    "yearFirstBurn": 2025,
    "campAffiliation": "Camp Awesome"
  }
}
```

**Response Success (201)**:
```json
{
  "data": {
    "user": {
      "id": "user_ghi789",
      "username": "playa_newbie",
      "email": "newbie@example.com",
      "profile": {
        "displayName": "Playa Newbie",
        "yearFirstBurn": 2025,
        "campAffiliation": "Camp Awesome"
      },
      "createdAt": "2025-01-26T20:30:00Z"
    },
    "tokens": {
      "accessToken": "jwt_access_token",
      "refreshToken": "jwt_refresh_token",
      "expiresIn": 3600
    }
  },
  "meta": {
    "timestamp": "2025-01-26T20:30:00Z"
  }
}
```

#### User Login
```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "username": "playa_veteran",
  "password": "secure_password"
}
```

**Response Success (200)**:
```json
{
  "data": {
    "user": {
      "id": "user_def456",
      "username": "playa_veteran",
      "profile": {
        "displayName": "Playa Veteran",
        "yearFirstBurn": 2015
      }
    },
    "tokens": {
      "accessToken": "jwt_access_token",
      "refreshToken": "jwt_refresh_token",
      "expiresIn": 3600
    }
  },
  "meta": {
    "timestamp": "2025-01-26T20:30:00Z"
  }
}
```

## API Conventions

### Request Standards

#### Headers
```
Content-Type: application/json
Accept: application/json
Authorization: Bearer {jwt_token}  # For authenticated endpoints
User-Agent: OK-OFFLINE-PWA/1.3.0
X-Request-ID: unique_request_id    # For tracking
```

#### Query Parameters
```
# Pagination
?limit=20&offset=0

# Filtering  
?category=shelter_shade&tags=wind-resistance

# Sorting
?sort=created_at&order=desc

# Including related data
?include=media,comments

# Field selection
?fields=id,title,content
```

### Response Standards

#### Success Response Structure
```json
{
  "data": {
    // Response data here
  },
  "meta": {
    "timestamp": "2025-01-26T20:30:00Z",
    "requestId": "req_abc123",
    "pagination": {
      "total": 100,
      "limit": 20,
      "offset": 0,
      "hasMore": true
    }
  }
}
```

#### Error Response Structure
```json
{
  "error": {
    "code": "validation_error",
    "message": "Validation failed",
    "details": {
      "title": "is required",
      "content": "must be at least 10 characters"
    }
  },
  "meta": {
    "timestamp": "2025-01-26T20:30:00Z",
    "requestId": "req_abc123"
  }
}
```

### Status Codes

#### Success Codes
- **200 OK**: Successful GET, PUT requests
- **201 Created**: Successful POST requests
- **204 No Content**: Successful DELETE requests

#### Client Error Codes
- **400 Bad Request**: Invalid request parameters
- **401 Unauthorized**: Authentication required
- **403 Forbidden**: Access denied
- **404 Not Found**: Resource not found
- **409 Conflict**: Resource conflict (e.g., duplicate username)
- **422 Unprocessable Entity**: Validation errors

#### Server Error Codes
- **500 Internal Server Error**: Unexpected server error
- **502 Bad Gateway**: Upstream service error
- **503 Service Unavailable**: Service temporarily unavailable
- **504 Gateway Timeout**: Upstream service timeout

## Authentication and Authorization

### JWT Token Structure
```json
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "user_def456",
    "username": "playa_veteran",
    "role": "user",
    "iat": 1643235600,
    "exp": 1643239200,
    "permissions": ["wisdom:create", "wisdom:read", "wisdom:update"]
  }
}
```

### Permission System
```
Permissions:
├── wisdom:create - Create wisdom entries
├── wisdom:read - Read wisdom entries  
├── wisdom:update - Update own wisdom entries
├── wisdom:delete - Delete own wisdom entries
├── wisdom:moderate - Moderate other users' content
├── user:update - Update own profile
└── admin:* - Full administrative access
```

## Rate Limiting

### Current Limits
```
Weather API:
├── 60 requests per minute per IP
├── 1000 requests per hour per IP
└── Burst limit: 10 requests per 10 seconds

Wisdom API (Future):
├── 100 requests per minute per user
├── 2000 requests per hour per user
├── Upload limit: 10 files per minute
└── Search limit: 30 searches per minute
```

### Rate Limit Headers
```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1643235700
X-RateLimit-Retry-After: 60
```

## Caching Strategy

### Response Caching
```
Cache Headers:
├── Cache-Control: public, max-age=600    # Weather data: 10 minutes
├── Cache-Control: public, max-age=3600   # Wisdom content: 1 hour
├── Cache-Control: private, max-age=300   # User data: 5 minutes
└── ETag: "abc123def456"                  # Entity tags for validation
```

### Cache Invalidation
```
Invalidation Triggers:
├── Weather data: Time-based (10 minutes)
├── Wisdom content: Event-based (on update/delete)
├── User data: Event-based (on profile update)
└── Search results: Event-based (on content changes)
```

## Error Handling

### Error Categories
```
Client Errors (4xx):
├── validation_error - Invalid input data
├── authentication_required - Missing/invalid token
├── permission_denied - Insufficient permissions
├── resource_not_found - Requested resource doesn't exist
├── rate_limit_exceeded - Too many requests
└── conflict - Resource conflict

Server Errors (5xx):
├── internal_error - Unexpected server error
├── service_unavailable - Dependency service down
├── database_error - Database connection/query error
└── external_api_error - Third-party API failure
```

### Error Response Examples
```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded. Please try again later.",
    "details": {
      "limit": 60,
      "remaining": 0,
      "resetAt": "2025-01-26T20:35:00Z"
    }
  },
  "meta": {
    "timestamp": "2025-01-26T20:30:00Z",
    "requestId": "req_abc123"
  }
}
```

## API Versioning

### Versioning Strategy
- **URL Versioning**: `/api/v1/`, `/api/v2/`
- **Backward Compatibility**: Support previous version for 12 months
- **Deprecation Warnings**: Include deprecation headers when needed
- **Migration Guides**: Provide clear upgrade paths

### Version Headers
```
API-Version: 1.0
Deprecated: false
Sunset: 2026-01-26T00:00:00Z  # When version will be retired
```

---

*This API design specification is part of the OK-OFFLINE ecosystem documentation. For implementation details, see the individual service repositories.*

**Created by**: Jeremy Roush  
**Brought to you by**: Mr. OK of OKNOTOK 🔥