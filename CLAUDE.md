# CLAUDE.md - OK-OFFLINE Ecosystem

This file provides guidance to Claude Code when working with the OK-OFFLINE ecosystem from the top-level project management perspective.

## Project Overview

**OK-OFFLINE** is a multi-service ecosystem for Burning Man participants, providing offline-first tools for navigation, weather monitoring, and future playa wisdom sharing. This repository serves as the project management and coordination hub.

## Current Architecture

### Service Structure
```
OK-OFFLINE Ecosystem
├── Frontend PWA (okoffline-2025)      # Vue 3 + Vite
├── API Services (ok-offline-api)      # Rails 8 API
└── Project Management (ok-offline-pm) # This repository
```

### Repositories Status
- **Frontend PWA**: https://github.com/jeremedia/okoffline-2025 ✅ Production
- **Weather API**: https://github.com/jeremedia/ok-offline-api 🔧 In Development  
- **Project Hub**: https://github.com/jeremedia/ok-offline-pm 📋 Current

## Working with Claude Code in This Ecosystem

### This Repository's Purpose
- **Cross-service coordination** - Plan features spanning multiple services
- **Architecture documentation** - Document system design and decisions
- **Project management** - Track roadmap, issues, and milestones
- **Development planning** - Coordinate multi-service development

### When to Use This Repository
Use this repository as your Claude Code working directory when:
- Planning features that span multiple services (e.g., Play Wisdom)
- Documenting architecture decisions
- Coordinating between frontend and backend development
- Working on ecosystem-wide concerns (deployment, CI/CD, etc.)

### When to Switch to Service-Specific Repositories
Switch to individual service repositories for:
- **Frontend development**: Use `okoffline-2025` repository
- **API development**: Use `ok-offline-api` repository
- **Service-specific debugging or features**

## Current Development Priorities

### Phase 1: Weather Service Integration (In Progress)
**Priority**: HIGH - Complete the weather service integration

**Status**: 
- [x] Rails API service created
- [x] Apple WeatherKit JWT authentication implemented  
- [x] Frontend weather service integration completed
- [ ] CORS configuration between frontend and API
- [ ] Production deployment

**Next Actions**:
1. Complete CORS configuration in Rails API
2. Test cross-origin requests from Vue frontend
3. Deploy API service to production
4. Update frontend to use production API endpoints

### Phase 2: Play Wisdom Feature (Next)
**Priority**: MEDIUM - Design and implement the playa wisdom sharing system

**Concept**: Enable burners to capture and share practical knowledge through photos, audio recordings, and text notes that sync when connectivity is available.

**Key Requirements**:
- Offline-first capture (photos, audio, text)
- Sync when connectivity returns
- Community moderation system
- Search and discovery features
- Mobile-optimized media capture

**Technical Considerations**:
- File upload and storage strategy (AWS S3)
- Offline sync queue implementation
- Media compression and optimization
- Content moderation workflows

## Service Interaction Patterns

### Frontend ↔ API Communication
- **CORS**: Configured for localhost:8000 (dev) and offline.oknotok.com (prod)
- **Authentication**: API key for weather services, future JWT for user features
- **Data Format**: JSON API responses with standardized error handling
- **Caching**: Frontend caches API responses for offline functionality

### Future Service Integration
- **Mobile App** ↔ **API**: Same API endpoints, mobile-optimized responses
- **Admin Dashboard** ↔ **API**: Extended endpoints for content management
- **Real-time Features**: WebSocket connections for live updates during events

## Development Workflow

### Multi-Service Development
When working across services:

1. **Plan in this repository** - Document the feature or change
2. **Update architecture docs** - Keep ARCHITECTURE.md current
3. **Coordinate implementation** - Update both services simultaneously
4. **Test integration** - Verify cross-service functionality
5. **Deploy coordinated releases** - Ensure compatibility

### Environment Setup
```bash
# Clone all repositories
git clone https://github.com/jeremedia/ok-offline-pm.git
git clone https://github.com/jeremedia/okoffline-2025.git frontend
git clone https://github.com/jeremedia/ok-offline-api.git api

# Frontend development
cd frontend
npm install
npm run dev  # Port 8000

# API development  
cd api
bundle install
rails db:setup
rails server  # Port 3000
```

### API Integration Points

#### Weather Service
- **Endpoint**: `/api/v1/weather/current`
- **Method**: POST
- **Payload**: `{ latitude, longitude, dataSets, timezone }`
- **Response**: Weather data with moon phase information
- **CORS**: Must allow requests from offline.oknotok.com

#### Future Play Wisdom API
- **Base**: `/api/v1/wisdom`
- **Endpoints**: 
  - `POST /api/v1/wisdom` - Create new wisdom entry
  - `GET /api/v1/wisdom` - List wisdom entries
  - `PUT /api/v1/wisdom/:id` - Update entry
  - `POST /api/v1/wisdom/:id/media` - Upload media files

## Technology Stack Decisions

### Frontend (Vue.js PWA)
- **Why Vue 3**: Mature ecosystem, excellent PWA support, Composition API
- **Why Vite**: Fast HMR, modern build tooling, excellent dev experience
- **Why IndexedDB**: Robust offline storage, handles large datasets
- **Why Leaflet**: Mature mapping library, BRC coordinate support

### Backend (Rails API)
- **Why Rails 8**: API mode, mature ecosystem, rapid development
- **Why PostgreSQL**: Robust data types, JSON support, full-text search
- **Why JWT**: Stateless authentication, mobile app friendly
- **Why Rack CORS**: Standard CORS handling for Rails APIs

### Deployment Strategy
- **Frontend**: Static hosting with CDN (currently Caddy)
- **API**: Containerized deployment (Docker + PostgreSQL)
- **CI/CD**: GitHub Actions for automated deployment
- **Environment**: Separate staging and production environments

## Common Development Tasks

### Adding a New Feature Spanning Services

1. **Document in this repository**:
   ```bash
   # Create feature specification
   touch docs/feature-specs/new-feature.md
   ```

2. **Update API first**:
   ```bash
   cd api
   # Add routes, controllers, models
   # Add tests
   # Update API documentation
   ```

3. **Update frontend**:
   ```bash
   cd frontend  
   # Add service integration
   # Add UI components
   # Update user flows
   ```

4. **Test integration**:
   ```bash
   # Test cross-origin requests
   # Test offline/online behavior
   # Test error handling
   ```

### Debugging Cross-Service Issues

1. **CORS Problems**:
   - Check Rails `config/initializers/cors.rb`
   - Verify frontend request headers
   - Test with browser dev tools

2. **API Integration**:
   - Check API logs in `api/log/development.log`
   - Verify request/response format
   - Test API endpoints directly with curl/Postman

3. **Authentication Issues**:
   - Verify JWT generation (Apple WeatherKit)
   - Check API key configuration
   - Validate request headers

## Documentation Guidelines

### Keep Current
- Update README.md when adding services or major features
- Update ARCHITECTURE.md when making design decisions
- Update ROADMAP.md as priorities change
- Create feature specs in `docs/feature-specs/` for major features

### Cross-Reference
- Link between repositories in documentation
- Reference GitHub issues across repositories
- Maintain consistent terminology across services

## Deployment and Operations

### Current Infrastructure
- **Frontend**: https://offline.oknotok.com (Caddy server)
- **API**: TBD (planning Docker deployment)
- **Database**: PostgreSQL (to be deployed with API)

### Monitoring and Logging
- **Frontend**: Browser error reporting, PWA analytics
- **API**: Rails logs, performance monitoring
- **Infrastructure**: Server monitoring, uptime checks

### Backup Strategy
- **Frontend**: Git repository (stateless)
- **API**: Database backups, code repository
- **User Data**: Local storage (no user accounts currently)

## Security Considerations

### API Security
- **CORS**: Properly configured for production domains
- **Rate Limiting**: Implement for production API
- **Input Validation**: Sanitize all user inputs
- **Authentication**: Secure JWT handling for user features

### Frontend Security
- **CSP**: Content Security Policy for production
- **HTTPS**: Required for PWA features and geolocation
- **Local Storage**: No sensitive data in browser storage
- **API Keys**: Environment variables, not committed to git

## Future Considerations

### Scaling Strategy
- **Horizontal Scaling**: Load balancers for API
- **Caching**: Redis for API response caching
- **CDN**: Global distribution for frontend assets
- **Database**: Read replicas for scaling reads

### Mobile App Integration
- **Shared API**: Same endpoints for web and mobile
- **Offline Sync**: Consistent strategy across platforms
- **Push Notifications**: For event updates and reminders
- **Native Features**: Camera, GPS, offline maps

### Community Features
- **User Accounts**: Authentication and profiles
- **Content Moderation**: Automated and manual review
- **Social Features**: Sharing, commenting, following
- **Real-time Updates**: WebSocket integration

## Getting Help

### Development Issues
1. Check service-specific CLAUDE.md files
2. Review architecture documentation
3. Check GitHub issues across repositories
4. Create cross-service issues in this repository

### Architecture Questions
1. Review ARCHITECTURE.md
2. Check feature specifications in `docs/`
3. Consider impact on all services
4. Document decisions for future reference

---

**Remember**: This ecosystem is designed to work offline-first in the harsh conditions of Black Rock City. Every feature should enhance the burner experience while maintaining reliability without connectivity.

Created by Jeremy Roush and brought to you by Mr. OK of OKNOTOK 🔥