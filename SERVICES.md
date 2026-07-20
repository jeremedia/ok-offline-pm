# OK-OFFLINE Services Documentation

> Historical 2025 service notes. Current 2026 service state and promotion gates are maintained in [docs/2026_RELEASE.md](docs/2026_RELEASE.md).

## Service Overview

The OK-OFFLINE ecosystem consists of multiple services working together to provide an offline-first experience for Burning Man participants. Each service has specific responsibilities and clear interfaces for communication.

## Service Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                 OK-OFFLINE Ecosystem                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ ┌─────────────────┐                                         │
│ │ Project Mgmt    │  ← You are here                        │
│ │ (ok-offline-pm) │                                         │
│ └─────────────────┘                                         │
│                                                             │
│ ┌─────────────────┐    ┌─────────────────┐                │
│ │ Frontend PWA    │◄──►│ Rails API       │                │
│ │ (okoffline-2025)│    │ (ok-offline-api)│                │
│ └─────────────────┘    └─────────────────┘                │
│                                                             │
│ ┌─────────────────┐    ┌─────────────────┐                │
│ │ Future: Mobile  │    │ Future: Admin   │                │
│ │ App (React RN)  │    │ Dashboard       │                │
│ └─────────────────┘    └─────────────────┘                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Frontend PWA Service

### Repository
**URL**: https://github.com/jeremedia/okoffline-2025  
**Technology**: Vue 3 + Vite  
**Status**: ✅ Production  
**Deployment**: https://offline.oknotok.com

### Purpose
Primary user interface providing offline-first access to Burning Man camps, art installations, events, and tools for schedule building and navigation.

### Core Architecture

#### Technology Stack
- **Vue 3** with Composition API for reactive components
- **Vite** for fast development and optimized production builds
- **Vue Router 4** for client-side navigation
- **Leaflet 1.9.3** for interactive maps with BRC coordinates
- **IndexedDB** for offline data persistence
- **Service Workers** for PWA functionality and caching

#### Key Features
```
User Features:
├── Offline Browsing - Camps, art, events without connectivity
├── Interactive Maps - BRC coordinate system with custom markers
├── Personal Schedule - Build burn schedule with conflict detection
├── Favorites System - Star camps/art/events across all views
├── Visit Tracking - Mark visited locations with notes
├── Global Search - Search across all content types
├── Emergency Info - Store medical info and contacts offline
├── Weather & Dust - Real-time conditions for Black Rock City
└── Smart Lists - Sortable, filterable views with grouping
```

#### Data Management
```javascript
// IndexedDB Structure
Database: bm2025-db
├── camps (store)
│   ├── Key: uid
│   └── Index: year
├── art (store) 
│   ├── Key: uid
│   └── Index: year
└── events (store)
    ├── Key: uid  
    └── Index: year

// LocalStorage Keys
├── selectedYear - Current year selection
├── favorites_[type] - User favorites by type
├── schedule - Personal event schedule
├── visits_[type] - Visit tracking with notes
├── emergency_contacts - Emergency contact info
└── weather_cache - Cached weather responses
```

#### Service Layer
```javascript
// Core Services
├── storage.js - IndexedDB operations and data management
├── dataSync.js - Sync with Burning Man API (static files)
├── weatherServiceCombined.js - Multi-source weather data
├── favorites.js - Favorites management across types
├── schedule.js - Personal schedule with conflict detection
├── visits.js - Visit tracking and notes
└── events.js - Event-specific operations and filtering
```

#### Component Architecture
```
Views:
├── MapView.vue - Interactive map with toggleable layers
├── ListView.vue - Sortable/filterable content lists
├── DetailView.vue - Individual item details with actions
├── SearchView.vue - Global search across all content
├── ScheduleView.vue - Personal schedule management
├── SettingsView.vue - Data sync and app configuration
├── EmergencyView.vue - Emergency contacts and medical info
└── DustForecastView.vue - Weather and dust conditions

Composables:
├── useGeolocation.js - Location services and distance calculation
├── useKeyboardShortcuts.js - Keyboard navigation shortcuts
├── useToast.js - User notification system
└── useAutoSync.js - Automatic data synchronization
```

#### PWA Features
- **Service Worker**: Caches app shell and static assets
- **Manifest**: Installable as standalone app
- **Offline Support**: Full functionality without connectivity
- **Background Sync**: Updates data when connectivity returns
- **Home Screen**: Custom icons and splash screens

### Development Setup

#### Prerequisites
- Node.js 18+
- Git

#### Quick Start
```bash
git clone https://github.com/jeremedia/okoffline-2025.git
cd okoffline-2025
npm install
npm run dev  # Runs on http://localhost:8000
```

#### Build Commands
```bash
npm run dev      # Development server with HMR
npm run build    # Production build
npm run preview  # Preview production build locally
```

#### Environment Configuration
```bash
# .env file
VITE_BM_API_KEY=your_burning_man_api_key
VITE_WEATHER_API_KEY=your_openweather_api_key
```

### API Integration

#### Current Integrations
- **Burning Man API**: Pre-enriched static data files
- **Weather Services**: Multi-source weather data via Rails API
- **OpenWeatherMap**: Primary weather service
- **Apple WeatherKit**: Fallback weather service with moon data

#### Request Patterns
```javascript
// Weather API Integration (via Rails API)
const getWeather = async () => {
  try {
    const response = await fetch('/api/v1/weather/current', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        latitude: 40.788645,
        longitude: -119.203018,
        dataSets: 'currentWeather,forecastDaily',
        timezone: 'America/Los_Angeles'
      })
    })
    return await response.json()
  } catch (error) {
    // Fallback to cached data
    return getCachedWeather()
  }
}
```

### Performance Optimizations

#### Bundle Optimization
- **Code Splitting**: Route-based chunks for optimal loading
- **Tree Shaking**: Remove unused code in production
- **Asset Optimization**: Minified CSS/JS, compressed images
- **Lazy Loading**: Heavy components loaded on demand

#### Caching Strategy
```javascript
// Service Worker Cache Strategy
const CACHE_STRATEGIES = {
  'app-shell': 'cache-first',        // Core app files
  'static-data': 'cache-first',      # BM data files  
  'api-responses': 'network-first',  // Weather data
  'user-data': 'cache-only'          // Local IndexedDB only
}
```

---

## Rails API Service

### Repository
**URL**: https://github.com/jeremedia/ok-offline-api  
**Technology**: Rails 8 API  
**Status**: 🔧 In Development  
**Deployment**: TBD

### Purpose
Backend API services providing weather data aggregation, CORS-compliant endpoints, and future Play Wisdom functionality.

### Core Architecture

#### Technology Stack
- **Rails 8** in API mode for lightweight, fast responses
- **PostgreSQL** for robust data persistence and JSON support
- **Rack CORS** for cross-origin resource sharing
- **JWT** for Apple WeatherKit authentication

#### Current Services

##### Weather Service
```ruby
# Primary endpoint for weather data
POST /api/v1/weather/current

# Service aggregates multiple sources:
├── OpenWeatherMap API (primary)
├── Apple WeatherKit API (fallback + moon data)
└── Cached responses (offline fallback)

# Response includes:
├── Temperature and feels-like
├── Wind speed/direction and pressure
├── Humidity and visibility
├── Dust level calculations
├── Moon phase data for navigation
└── Source attribution and timestamps
```

##### Authentication
```ruby
# Apple WeatherKit JWT Generation
class AppleWeatherService
  def generate_jwt
    # ES256 JWT with Apple Developer credentials
    # Includes team ID, service ID, and key ID
    # Expires after 1 hour for security
  end
end
```

#### API Architecture
```ruby
# Controller Structure
├── ApplicationController - Base controller with CORS
├── WeatherController - Weather data aggregation
└── HealthController - Service health checks

# Service Layer
├── WeatherAggregatorService - Multi-source weather data
├── AppleWeatherService - Apple WeatherKit integration
├── OpenWeatherService - OpenWeatherMap integration
└── WeatherCacheService - Response caching logic

# Models (Future)
├── WisdomEntry - Play wisdom content
├── User - User accounts and preferences
└── MediaUpload - File uploads and metadata
```

### Development Setup

#### Prerequisites
- Ruby 3.2+
- PostgreSQL 14+
- Git

#### Quick Start
```bash
git clone https://github.com/jeremedia/ok-offline-api.git
cd ok-offline-api
bundle install
rails db:setup
rails server  # Runs on http://localhost:3000
```

#### Environment Configuration
```bash
# .env file
OPENWEATHER_API_KEY=your_openweather_key
APPLE_WEATHER_TEAM_ID=your_apple_team_id
APPLE_WEATHER_SERVICE_ID=your_service_id
APPLE_WEATHER_KEY_ID=your_key_id
APPLE_WEATHER_PRIVATE_KEY=your_private_key
DATABASE_URL=postgresql://localhost/ok_offline_development
```

### CORS Configuration

#### Current Settings
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

#### Security Considerations
- **Rate Limiting**: Implement per-IP rate limiting
- **Input Validation**: Sanitize all request parameters
- **Error Handling**: Don't expose internal error details
- **API Versioning**: Support multiple API versions

### Deployment Planning

#### Docker Configuration
```dockerfile
# Planned Dockerfile
FROM ruby:3.2-alpine
RUN apk add --no-cache postgresql-dev build-base
WORKDIR /app
COPY Gemfile* ./
RUN bundle install
COPY . .
EXPOSE 3000
CMD ["rails", "server", "-b", "0.0.0.0"]
```

#### Production Environment
```yaml
# docker-compose.yml
version: '3.8'
services:
  api:
    build: .
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/ok_offline
      - RAILS_ENV=production
    ports:
      - "3000:3000"
    depends_on:
      - db
  
  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=ok_offline
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
```

### Future API Expansion

#### Play Wisdom Endpoints
```ruby
# Planned API endpoints for Phase 2
POST   /api/v1/wisdom           # Create wisdom entry
GET    /api/v1/wisdom           # List wisdom entries
GET    /api/v1/wisdom/:id       # Get specific entry
PUT    /api/v1/wisdom/:id       # Update entry
DELETE /api/v1/wisdom/:id       # Delete entry
POST   /api/v1/wisdom/:id/media # Upload media files
GET    /api/v1/wisdom/search    # Search entries
```

#### Authentication System
```ruby
# Future user authentication
POST   /api/v1/auth/login       # User login
POST   /api/v1/auth/register    # User registration  
POST   /api/v1/auth/refresh     # Refresh token
DELETE /api/v1/auth/logout      # User logout
GET    /api/v1/users/profile    # User profile
```

---

## Project Management Service

### Repository  
**URL**: https://github.com/jeremedia/ok-offline-pm  
**Technology**: Documentation + GitHub Issues  
**Status**: ✅ Active  
**Purpose**: Ecosystem coordination and documentation

### Role in Ecosystem
- **Cross-service coordination** for features spanning multiple services
- **Architecture documentation** and design decisions
- **Roadmap planning** and priority management
- **Developer onboarding** and ecosystem understanding

### Documentation Structure
```
Repository Contents:
├── README.md - Ecosystem overview and service status
├── CLAUDE.md - Instructions for Claude Code sessions
├── ARCHITECTURE.md - System architecture and decisions
├── ROADMAP.md - Development phases and priorities
├── SERVICES.md - This file - detailed service docs
└── docs/ - Additional specifications and guides
```

### Usage for Development
```bash
# For cross-service Claude Code sessions
git clone https://github.com/jeremedia/ok-offline-pm.git
cd ok-offline-pm
# Use as working directory for ecosystem-wide development
```

---

## Service Communication Patterns

### Frontend ↔ API Communication

#### Request/Response Flow
```
Frontend Request → CORS Preflight → API Processing → Response
     ↓                    ↓              ↓           ↓
Cache Check → Options Check → Service Logic → JSON Response
```

#### Error Handling Pattern
```javascript
// Robust error handling with fallbacks
async function apiRequest(endpoint, options) {
  try {
    // 1. Try API request
    const response = await fetch(endpoint, options)
    if (!response.ok) throw new Error(`HTTP ${response.status}`)
    return await response.json()
    
  } catch (apiError) {
    // 2. Try cached data
    const cached = getCachedData(endpoint)
    if (cached && !isExpired(cached)) return cached.data
    
    // 3. Use offline fallback
    return getOfflineFallback(endpoint)
  }
}
```

#### Data Synchronization
```javascript
// Bi-directional sync pattern
class DataSync {
  async syncToServer(localData) {
    // Upload local changes when online
  }
  
  async syncFromServer() {
    // Download server changes when online
  }
  
  resolveConflicts(localData, serverData) {
    // Handle conflicts with user input or automatic resolution
  }
}
```

### Future: Mobile ↔ API Communication

#### Shared API Strategy
```
Mobile Apps (iOS/Android)
     ↓
Shared API Layer (Rails)
     ↓  
Same endpoints as web PWA
     ↓
Consistent data models and caching
```

#### Mobile-Specific Considerations
- **Background Sync**: Sync data when app backgrounded
- **Push Notifications**: Real-time alerts and updates
- **Native Integration**: Camera, GPS, contacts, calendar
- **Performance**: Optimize for mobile networks and battery

---

## Development Workflows

### Cross-Service Feature Development

#### Planning Phase
1. **Specification in ok-offline-pm** - Document feature requirements
2. **Architecture Review** - Update ARCHITECTURE.md if needed
3. **API Design** - Define endpoints and data models
4. **Frontend Design** - Plan UI/UX and user flows

#### Implementation Phase
1. **API First** - Implement backend endpoints and tests
2. **Frontend Integration** - Build UI and integrate with API
3. **Cross-Service Testing** - Test full integration
4. **Documentation** - Update service docs and guides

#### Deployment Phase
1. **Staging Deployment** - Deploy to staging environment
2. **Integration Testing** - Test full system in staging
3. **Production Deployment** - Coordinated production deployment
4. **Monitoring** - Watch for issues and performance

### Service-Specific Development

#### Frontend PWA Development
```bash
# Typical development workflow
cd okoffline-2025
git checkout -b feature/new-feature
npm run dev
# Develop and test locally
npm run build && npm run preview
# Test production build

# IMPORTANT: Increment service worker cache version before release
# Edit public/sw.js: ok-offline-v4 → ok-offline-v5
# This forces Safari and other browsers to update cached assets

git commit && git push
# Deploy via GitHub Actions
```

#### Rails API Development  
```bash
# Typical development workflow
cd ok-offline-api
git checkout -b feature/new-endpoint
rails server
# Develop and test locally
rails test
# Run test suite
git commit && git push
# Deploy via Docker (future)
```

---

## Monitoring and Observability

### Frontend Monitoring
- **Performance**: Core Web Vitals, bundle sizes, load times
- **Usage**: Feature adoption, user flows, offline usage
- **Errors**: JavaScript errors, service worker issues
- **PWA Metrics**: Installation rates, engagement

### API Monitoring
- **Performance**: Response times, throughput, database queries
- **Reliability**: Error rates, uptime, service availability
- **Usage**: Endpoint popularity, rate limiting triggers
- **Resources**: CPU, memory, database performance

### Cross-Service Monitoring
- **Integration Health**: End-to-end request success rates
- **Data Consistency**: Sync success rates, conflict resolution
- **User Experience**: Complete user journey success
- **System Performance**: Overall ecosystem health

---

## Security Considerations

### Frontend Security
- **Content Security Policy**: Restrict resource loading
- **HTTPS Only**: Required for PWA and geolocation
- **Input Sanitization**: Clean all user inputs
- **Local Storage**: No sensitive data in browser storage

### API Security
- **CORS Configuration**: Proper origin restrictions
- **Rate Limiting**: Prevent abuse and DoS attacks
- **Input Validation**: Validate and sanitize all inputs
- **Authentication**: Secure JWT handling and storage

### Cross-Service Security
- **API Key Management**: Environment variables only
- **Request Signing**: HMAC signatures for sensitive operations
- **Audit Logging**: Log all security-relevant operations
- **Regular Updates**: Keep dependencies updated

---

## Troubleshooting Guide

### Common Issues

#### CORS Errors
```
Symptom: Browser blocks requests to API
Solution: 
1. Check CORS origins in rails config/initializers/cors.rb
2. Verify frontend URL matches allowed origins
3. Test with curl to isolate CORS vs API issues
```

#### Weather Data Not Loading
```
Symptom: Weather section shows loading or error state
Solution:
1. Check API key configuration in environment
2. Verify API endpoints are responding
3. Check browser network tab for failed requests
4. Verify fallback to cached data works
```

#### IndexedDB Issues
```
Symptom: Data not persisting or sync failures
Solution:
1. Check browser storage limits and usage
2. Clear IndexedDB and re-sync data
3. Check for schema version mismatches
4. Verify service worker registration
```

#### PWA Installation Problems
```
Symptom: Install prompt not appearing
Solution:
1. Verify HTTPS is working correctly
2. Check manifest.json is valid and served
3. Ensure service worker is registered
4. Check browser PWA installation criteria
```

### Debug Tools
- **Chrome DevTools**: Application tab for PWA inspection
- **Firefox PWA Tools**: PWA debugging and testing
- **Rails Console**: API debugging and data inspection
- **Network Tab**: Request/response inspection
- **Storage Inspector**: IndexedDB and cache inspection

---

*This documentation is maintained as part of the OK-OFFLINE ecosystem. For updates or corrections, please submit issues or pull requests to the respective repositories.*

**Created by**: Jeremy Roush  
**Brought to you by**: Mr. OK of OKNOTOK 🔥
