# OK-OFFLINE Ecosystem Architecture

> Historical 2025 architecture notes. Current 2026 release architecture and proof boundaries are maintained in [docs/2026_RELEASE.md](docs/2026_RELEASE.md).

## System Overview

The OK-OFFLINE ecosystem is designed as a multi-service architecture optimized for offline-first usage in the challenging environment of Burning Man. The system prioritizes reliability, offline functionality, and progressive enhancement.

## Architecture Principles

### 1. Offline-First Design
- **Local Storage Priority**: All critical functionality works without connectivity
- **Progressive Sync**: Data syncs when connectivity is available
- **Graceful Degradation**: Features degrade gracefully when offline
- **Conflict Resolution**: Handle data conflicts when syncing

### 2. Service Independence
- **Loose Coupling**: Services can be developed and deployed independently
- **Clear Boundaries**: Well-defined interfaces between services
- **Fault Tolerance**: Service failures don't cascade
- **Technology Diversity**: Each service uses optimal technology stack

### 3. Progressive Enhancement
- **Core Functionality**: Essential features work in all conditions
- **Enhanced Features**: Additional features when connectivity allows
- **Performance**: Fast initial load, lazy loading for enhanced features
- **Accessibility**: Works across devices and network conditions

## Current Service Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    OK-OFFLINE Ecosystem                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐    ┌─────────────────┐                │
│  │   Frontend PWA  │    │   Rails API     │                │
│  │   (Vue 3)       │◄──►│   (Weather)     │                │
│  │                 │    │                 │                │
│  │  • Offline Maps │    │  • WeatherKit   │                │
│  │  • Local Cache  │    │  • OpenWeather  │                │
│  │  • PWA Features │    │  • CORS Config  │                │
│  │  • Service SW   │    │  • JWT Auth     │                │
│  └─────────────────┘    └─────────────────┘                │
│           │                        │                       │
│           ▼                        ▼                       │
│  ┌─────────────────┐    ┌─────────────────┐                │
│  │   IndexedDB     │    │   PostgreSQL    │                │
│  │   (Client)      │    │   (Server)      │                │
│  └─────────────────┘    └─────────────────┘                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Service Specifications

### Frontend PWA (Vue 3 + Vite)

**Purpose**: Primary user interface for Burning Man participants

**Key Components**:
- **Vue 3 Application**: Modern reactive framework with Composition API
- **Vite Build System**: Fast development with HMR and optimized production builds
- **Vue Router**: Client-side routing for SPA navigation
- **Leaflet Maps**: Interactive maps with BRC coordinate system
- **IndexedDB Storage**: Persistent offline data storage
- **Service Worker**: PWA functionality and caching strategy

**Data Flow**:
```
User Interaction → Vue Components → Services → IndexedDB/API
                                              ↓
                  UI Updates ← State Management ← Data Response
```

**Offline Strategy**:
- **Critical Data**: Camps, art, events stored in IndexedDB
- **User Data**: Favorites, schedule, visits stored locally
- **Map Tiles**: Cached via Service Worker
- **API Responses**: Cached with TTL for weather data

### Rails API (Weather Service)

**Purpose**: Backend services for weather data aggregation and future features

**Key Components**:
- **Rails 8 API Mode**: Lightweight API-only Rails application
- **CORS Configuration**: Proper cross-origin resource sharing
- **JWT Authentication**: For Apple WeatherKit integration
- **Multiple Weather Sources**: OpenWeatherMap + Apple WeatherKit

**API Design**:
```
POST /api/v1/weather/current
Content-Type: application/json

{
  "latitude": 40.788645,
  "longitude": -119.203018,
  "dataSets": "currentWeather,forecastDaily",
  "timezone": "America/Los_Angeles"
}

Response:
{
  "temperature": 85,
  "windSpeed": 15,
  "dustLevel": "moderate",
  "moonPhase": {
    "phase": 0.75,
    "phaseName": "Last Quarter",
    "phaseIcon": "🌗"
  },
  "source": "apple-api",
  "lastUpdated": "2025-01-26T20:30:00Z"
}
```

**Error Handling**:
- **Graceful Fallbacks**: Apple Weather → OpenWeather → Cached Data
- **Standard HTTP Codes**: Proper status codes for different scenarios
- **Error Context**: Detailed error messages for debugging

## Data Architecture

### Frontend Data Storage (IndexedDB)

```javascript
// Database: bm2025-db
{
  // Core event data
  camps: [
    {
      uid: "string",
      name: "string", 
      location_string: "string",
      description: "string",
      year: number
    }
  ],
  
  // User-generated data
  favorites_camp: ["uid1", "uid2"],
  schedule: [
    {
      eventId: "string",
      addedAt: "timestamp"
    }
  ],
  
  // Weather cache
  weather_cache: {
    current: { /* weather data */ },
    timestamp: number,
    ttl: 600000 // 10 minutes
  }
}
```

### API Data Models

```ruby
# Weather service doesn't persist data, acts as proxy
class WeatherService
  # Aggregates data from multiple sources
  # Handles authentication and rate limiting
  # Provides consistent response format
end

# Future: Play Wisdom models
class WisdomEntry
  # id, user_id, title, content, media_urls
  # location, tags, created_at, moderated
end
```

## Security Architecture

### Frontend Security

**Content Security Policy**:
```
default-src 'self'
script-src 'self' 'unsafe-inline'
style-src 'self' 'unsafe-inline'
img-src 'self' data: https:
connect-src 'self' https://api.openweathermap.org
```

**Data Protection**:
- **No Sensitive Storage**: No API keys or secrets in frontend
- **Local Data Only**: User data stays on device
- **HTTPS Required**: For PWA features and geolocation
- **Input Sanitization**: All user inputs sanitized

### API Security

**CORS Configuration**:
```ruby
# config/initializers/cors.rb
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins ['https://offline.oknotok.com', 'http://localhost:8000']
    resource '/api/*',
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head],
      credentials: false
  end
end
```

**Authentication**:
- **API Keys**: Environment variables for weather services
- **JWT Tokens**: Generated for Apple WeatherKit (ES256)
- **Rate Limiting**: Prevent abuse of weather endpoints
- **Input Validation**: Sanitize all parameters

## Communication Patterns

### Frontend ↔ API

**Request Pattern**:
```javascript
// Robust error handling with fallbacks
const fetchWeather = async () => {
  try {
    // Try API first
    const response = await fetch('/api/v1/weather/current', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ latitude, longitude })
    })
    
    if (!response.ok) throw new Error(`HTTP ${response.status}`)
    return await response.json()
    
  } catch (error) {
    // Fallback to cached data
    return getCachedWeather()
  }
}
```

**Response Format**:
```javascript
// Consistent response structure
{
  data: { /* actual response data */ },
  meta: {
    source: "api|cache|fallback",
    timestamp: "ISO string",
    ttl: number
  },
  errors: [
    { code: "string", message: "string" }
  ]
}
```

## Performance Architecture

### Frontend Performance

**Bundle Optimization**:
- **Code Splitting**: Route-based splitting for optimal loading
- **Tree Shaking**: Remove unused code in production builds  
- **Asset Optimization**: Minified CSS/JS, optimized images
- **Lazy Loading**: Maps and heavy components loaded on demand

**Caching Strategy**:
```javascript
// Service Worker caching
const CACHE_STRATEGY = {
  'static-assets': 'cache-first',    // CSS, JS, images
  'api-data': 'network-first',       // Weather, dynamic data
  'app-shell': 'cache-first',        // Core app structure
  'user-data': 'cache-only'          // IndexedDB only
}
```

**Data Loading**:
- **Progressive Loading**: Core data first, enhanced features later
- **Background Sync**: Update data when connectivity available
- **Intelligent Caching**: Cache based on usage patterns
- **Compression**: Gzip/Brotli for API responses

### API Performance

**Response Optimization**:
- **Efficient Queries**: Optimized database queries
- **Response Caching**: Cache responses at multiple levels
- **Compression**: Gzip responses automatically
- **Connection Pooling**: Efficient database connections

**Scaling Considerations**:
```ruby
# Future scaling architecture
class WeatherController
  before_action :rate_limit
  
  def current
    # Cache responses for 10 minutes
    Rails.cache.fetch("weather:#{cache_key}", expires_in: 10.minutes) do
      aggregate_weather_data
    end
  end
  
  private
  
  def rate_limit
    # Implement rate limiting per IP
  end
end
```

## Deployment Architecture

### Current Deployment

**Frontend (Static Hosting)**:
```
GitHub Actions → Build → Deploy to Caddy Server
├── Automatic deployment on push to main
├── Version management with semantic versioning
└── HTTPS with automatic certificate management
```

**API (Planned)**:
```
GitHub Actions → Docker Build → Container Deployment
├── PostgreSQL database
├── Environment-based configuration
├── Health checks and monitoring
└── Horizontal scaling capability
```

### Production Environment

```
Internet
    │
    ▼
┌─────────────────┐
│   Load Balancer │
│   (Caddy)       │
└─────────────────┘
    │
    ├── Static Assets ──► CDN/Cache
    │
    └── API Requests ──► Docker Container
                           │
                           ├── Rails API
                           └── PostgreSQL
```

## Monitoring and Observability

### Frontend Monitoring

**Metrics to Track**:
- **Performance**: Page load times, bundle sizes
- **Usage**: Feature adoption, offline usage patterns
- **Errors**: JavaScript errors, service worker failures
- **PWA**: Installation rates, offline usage

**Implementation**:
```javascript
// Error tracking
window.addEventListener('error', (event) => {
  // Log errors for analysis
  console.error('Application error:', event.error)
})

// Performance monitoring
const observer = new PerformanceObserver((list) => {
  // Track Core Web Vitals
})
```

### API Monitoring

**Metrics to Track**:
- **Performance**: Response times, throughput
- **Reliability**: Error rates, uptime
- **Usage**: Endpoint popularity, rate limiting
- **Resources**: Database performance, memory usage

**Health Checks**:
```ruby
# config/routes.rb
get '/health', to: 'health#check'

class HealthController < ApplicationController
  def check
    render json: {
      status: 'ok',
      timestamp: Time.current,
      services: {
        database: database_healthy?,
        weather_apis: weather_apis_healthy?
      }
    }
  end
end
```

## Future Architecture Evolution

### Phase 2: Play Wisdom Feature

**Additional Components**:
```
Current Architecture
    +
┌─────────────────┐    ┌─────────────────┐
│   Media Upload  │    │   Content API   │
│   (Frontend)    │◄──►│   (Rails)       │
└─────────────────┘    └─────────────────┘
         │                        │
         ▼                        ▼
┌─────────────────┐    ┌─────────────────┐
│   File Storage  │    │   Media DB      │
│   (AWS S3)      │    │   (PostgreSQL)  │
└─────────────────┘    └─────────────────┘
```

**New Data Flows**:
- **Media Capture**: Photos/audio captured offline
- **Sync Queue**: Uploads queued for connectivity
- **Content Pipeline**: Moderation and processing
- **Search Index**: Full-text search across wisdom

### Phase 3: Mobile App Integration

**Architecture Extension**:
```
Shared API Layer
├── Web PWA (existing)
├── iOS App (React Native)
├── Android App (React Native)
└── Admin Dashboard (Vue/React)
```

**Considerations**:
- **Shared Codebase**: React Native for mobile apps
- **Offline Sync**: Consistent strategy across platforms
- **Push Notifications**: Real-time updates
- **Native Integration**: Camera, GPS, device features

## Technology Decisions and Rationale

### Frontend Technology Choices

**Vue 3 vs React/Angular**:
- ✅ **Smaller bundle size** for offline performance
- ✅ **Excellent PWA ecosystem** with proven tools
- ✅ **Composition API** for better code organization
- ✅ **Strong TypeScript support** for maintainability

**Vite vs Webpack/Parcel**:
- ✅ **Faster development** with instant HMR
- ✅ **Optimized production builds** with Rollup
- ✅ **Modern ES modules** for better performance
- ✅ **Plugin ecosystem** for Vue.js integration

**IndexedDB vs localStorage/sessionStorage**:
- ✅ **Large storage capacity** for offline data
- ✅ **Structured queries** for complex data relationships
- ✅ **Asynchronous access** for better performance
- ✅ **Transaction support** for data consistency

### Backend Technology Choices

**Rails vs Node.js/Django**:
- ✅ **Rapid API development** with conventions
- ✅ **Mature ecosystem** for authentication and security
- ✅ **Strong PostgreSQL integration** for future features
- ✅ **API mode optimization** for performance

**PostgreSQL vs MySQL/MongoDB**:
- ✅ **JSON column support** for flexible data structures
- ✅ **Full-text search** for wisdom content
- ✅ **Strong consistency** for critical data
- ✅ **Advanced indexing** for performance optimization

## Conclusion

The OK-OFFLINE architecture is designed to provide a reliable, performant, and scalable platform for Burning Man participants. The offline-first approach ensures functionality in challenging connectivity conditions, while the multi-service architecture allows for independent development and deployment of features.

The current architecture successfully supports the core PWA functionality and weather services, with a clear path for evolution to support Play Wisdom and other community features. The technology choices prioritize performance, maintainability, and user experience in the unique environment of Black Rock City.

---

*Architecture designed by Jeremy Roush for the OK-OFFLINE ecosystem. Built with love for the Burning Man community. 🔥*
